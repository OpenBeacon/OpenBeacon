# OpenBeacon

This document specifies some design goals and ideas for an open protocol for
beaconing. I decided to start defining this specification because I don't think
protocols like AltBeacon don't do enough to make beaconing truly open. In my
opinion an open protocol like this should not depend on any one provider to
provide a database of UUIDs and I think there are other very valuable features
that could be added without comprimising security or privacy.

Most existing beaconing protocols use an arbitrary UUID to identify a beacon,
some sperateing it into an organization and individual ID. But this lacks
discoverability without some certral database that assigns the organization ID
and an interal database or known structure of the internal IDs. This doesn't
seem like an ideal system to me.

What we could really use is some sort of existing, fair system for identifying
organizations and then allow them define their own id allocation within their
organization identifier. How about domains and URLs?

Now we've got no central database that needs to be maintained and we
automatically get a way to get more information about that beacon.

# An Example
Lets say that MetroTransit decides that they want to stick an OpenBeacon on each
of their bus stop signs so that users standing near their stops can get notified
of when the next buses will be arriving at that stop.

They could setup beacons that broadcast the beacon "mt.obcn.io/S17948". When the
user's device sees this, it can make a request to that URL and get some
structured result that would then tell the device how to handle that data. It
could also make a request to the base domain "mt.ocbn.io" to get general
information about that OpenBeacon Organization.

    OpenBeacon     <BLE>             Client          <HTTPS>           Server
      |                                 |                                 |
      |-------------------------------->|                                 |
      |       "mt.obcn.io/S17948"       |                                 |
      |                                 |-------------------------------->|
      |                                 |      GET mt.obcn.io/S17948      |
      |                                 |                                 |
      |                                 |<--------------------------------|
      |                                 |         application/json        |
      |                                 | {"location": "<loc>",           |
      |                                 |  "actions":                     |
      |                                 |   {0: { "href": "<URL>"},       |
      |                                 |    5: { "msg": "<MSG>"}}}       |
      |                                 |                                 |
      |-------------------------------->|                                 |
      |       "mt.obcn.io/S17948"       |                                 |
      |                                 |     <Already Seen Recently>     |
      |              ...                |           <No Action>           |

In the given example, the `href` action would be a link to a webpage with live
stop information and the `msg` action could be a simple message that says "The
4L northbound will be here in 5 minutes.". Other action options could be defined
actions such-as "transit-stop" which would directly list the upcoming routes
that will be coming to that location.

# Packet Format

The message format shown below is sent an a Bluetooth 4.1 Advertising PDU with a
type of `ADV_NONCONN_IND` (0010). The OpenBeacon message is sent as the
`AdvData` portion of the PDU payload.

* A defined message format, should include:
  - OpenBeacon Marker, 1 Octet, 0x0B
  - URL, Up To 26 Octets (Must Contain At Least One '.' and One '/')


     0               1               2               3
     0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |    Length     |     0xFF      |    Company Identifier Code
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |     0x0B      |     URL 
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
        URL (Continued)
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
        URL (Continued)
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
        URL (Continued)
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
        URL (Continued)
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
        URL (Continued)
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
        URL (Continued)                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    
# General Information Requests
This is the basic request that fetches general information about the
organization which lets the user say whether they would like to let their device
make requests to that domain. These requests will be made by the device only
once per domain and must be cached until the user Accepts the domain. The TLS
cert should be validated and any special features, such as an EV cert, should
be noted in the UI and items should be sorted based on level of authentication.

    {
        "$schema": "http://json-schema.org/draft-04/schema#",
        "title": "OpenBeacon Organization Information Response",
        "type": "object",
        "properties": {
            "name": {
                "description": "Name of Organization (Eg. Metro Transit, etc.)"
                "type": "string"
            },
            "description": {
                "description": "A human readable description of the organization.",
                "type": "object",
            },
            "type": {
                "description": "The well-known organization type.",
                "type": "string"
            },
            "beacon-list": {
                "description": "A list of beacons or beacon types.",
                "type": "string",
                "example": "https://moa.obcn.io/beacons
            },
            "map": {
                "description": "Link to more info about the area in geojson.",
                "type": "string",
                "example": "https://moa.obcn.io/geo"
            }
        },
        "required": ["name"]
    }

## Notes to Self
May want to limit the size of the response since we're caching everything we see,
may also want to let applicaions expire caches to prevent denial of service
attacks on the deivce by broadcasting thousands of domains at once. But then this
might start leaking more info than we want.

# Extended Information Requests
These are HTTP GET requests to the URL broadcast in the advertising packets.
They will return a json object containing at a minimum a `actions` member whose
value is an array of `<action>`s where the order defines the preference of
execution. `<action>` is an is a pair consisting of the well-known action name
and an action value where the action value is defined per action.

    {
        "$schema": "http://json-schema.org/draft-04/schema#",
        "title": "OpenBeacon Extended Information Response",
        "type": "object",
        "properties": {
            "name": {
                "type": "string"
            },
            "descriptor": {
                "description": "A static description of the beacon or location.",
                "type": "object",
                "properties": {
                    "description": {
                        "description": "Human readable description.",
                        "type": "string"
                    },
                    "type": {
                        "description": "The well-known descriptor type.",
                        "type": "string"
                    },
                    "value": {
                        "description": "The type-dependent object.",
                        "type": "object"
                    }
                }
            },
            "actions": {
                "description": "A prioritized list actions that may be taken.",
                "type": "array",
                "items": {
                    "type": "object",
                    "description" : "Value defined by the well-known name."
                }
            },
            "location": {
                "description": "Coordinates of the beacon.",
                "$ref": "http://json-schema.org/geo"
            },
            "map": {
                "description": "Link to more info about the area and surrounding beacons in geojson.",
                "type": "string",
                "example": "https://moa.obcn.io/.geo"
            }
        },
        "required": ["name"]
    }

The response should include HTTP methods for cache control such as `Age`,
`Cache-Control`, `Expires`, and/or those relating to entity tags. Clients
should respect these headers when present.

In the case that an application has registered the domain, the extended
information request does not need to be performed directly and the URL should
be passed to the application.

# Security and Privacy Considerations
Beaconing has a high risk of abuse and the loss of user privacy should be
considered throughly.

## HTTP Headers
Requests must have an exact set of explicitly defined HTTP headers so that
there is no variation between clients. Beacon servers must must respond with
a message format error if the headers differ.

A message format error is defined as either a 400 response code or any 3xx 
response code. This is to allow servers to redirect users hitting those URLs
with a browser to either an information page or the actual resource that
the beacon is representing.

## Proxied Requests
All requests must be proxied through another beacon server. This also means
that all beacon servers must act as a proxy to other beacon servers. This is a
measure to prevent any single server from knowing both who you are (your IP)
and where you are (the beacons that you see).

The beacon domain must use a SRV record to point to the host that will handle
responding to the extended information request, proxies must not proxy to a
domain that does not provide a SRV record in the form
 `_openbeacon._tcp.obcn.io. 86400 IN SRV 0 5 5060 serv.obcn.io.`


* A device may not make requests to beacon URLs until the domain of the beacon
  has been approved by the user. The deivce may only make one request to the
  root (`/`) URI once and must cache the response so as to not ever need to
  make that request again.
  - Also, the UI of the application should show the levels of trust based on
    the cert. That is, if it's an EV cert, hilight it and show the company info.
    May also want to show it higher in the list.

# Going Further
Obviously this is from from a fully fleshed out spec, comments of all kinds are
welcome. For now, use the github issues tab to submit suggestions, questions,
and abuse.

Another thing that would be nice, would be to not use the manufacturer specfic
data type, since it's ideally not manufaturer specic and that would actually
give us another three bytes to work with. If anyone has experiance working with
the Bluetooth SIG, please get in touch with me.

# Comparison with Other Options
* [AltBeacon](http://altbeacon.org/)
  - Basically iBeacon with the trademarks crossed out.
* [Physical Web](https://github.com/google/physical-web/)
  - Dev project from Google
  - Requires having a trusted server to proxy requests. (Or giving up privacy.) (Maybe, there's some discussion going on about how to handle this in their GH issues tracker.)
