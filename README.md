# OpenBeacon

This documents does not define a protocol, but rather specifies some design
goals for an open protocol for beaconing. It is inspired by the
[AltBeacon](http://altbeacon.org/) specification. I decided to make this because
I don't think it goes far enough. In my opinion an open protocol like this
should not depend on any one provider to provide a database of UUIDs.

The AltBeacon specification says that "For interoperability purposes, the first
16+ bytes of the beacon identifier should be unique to the advertiser's
organizational unit.". But, who will assign that unique identifier and why
should I trust them?

AltBeacon says that won't charge any royalties or fees, but then who will pay to
keep this central database up and running?

What we could really use is some sort of existing, fair system for identifying
organizations and then allow them define their own id allocation within their
organization identifier. How about domains and URIs?

Now we've got no central database that needs to be maintained and we
automatically get a way to get more information about that beacon.

# An Example
Lets say that MetroTransit decides that they want to stick an OpenBeacon on each
of their bus stop signs so that users standing near their stops can get notified
of when the next buses will be arriving at that stop.

They could setup beacons that broadcast the beacon "mt.obcn.io/S17948". When the
user's device sees this, it can make a request to that URL and get some
structured result that would then tell the device how to handle that data. It
could also make a request to something like "mt.ocbn.io/.info" to get general
information about that OpenBeacon Organization.

    OpenBeacon     <BLE>             Client          <HTTPS>           Server
      |                                 |                                 |
      |-------------------------------->|                                 |
      | 0x0B, -37, "mt.obcn.io/S17948"  |                                 |
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
      | 0x0B, -37, "mt.obcn.io/S17948"  |                                 |
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
  - Reference RSSI @ 1m, 1 Octets
    - I'm not sure how useful this field will be for a heterogeneous mixture of
      client devices. (Rather than just iPhone in iBeacon)
  - URI, Up To 30 Octets (Must Contain At Least One '.' and One '/')


     0               1               2               3
     0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |      0x0B     |    Ref RSSI   |               URI
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |   URI (Continued)
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |   URI (Continued)
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |   URI (Continued)
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |   URI (Continued)
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |   URI (Continued)
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |   URI (Continued)
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |   URI (Continued)
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

# Other things to consider:
* Encode domain as UTF-8, then use a binary value as a path, base 64 encoded in
  HTTP for additional usable space.
  - '/' is used as terminator or use 4 bits for domain length minus 4.(min 4, 
    max 20).

# Extended Information Requests
These are HTTP GET requests to the URI broadcast in the advertising packets.
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
            }
        },
        "required": ["name"]
    }

The response should include HTTP methods for cache control such as `Age`,
`Cache-Control`, `Expires`, and/or those relating to entity tags. Clients
should respect these headers when present.

In the case that an application has registered the domain, the extended
information request does not need to be performed directly and the URI should
be passed to the application.

# Security and Privacy Considerations
Beaconing has a high risk of abuse and the loss of user privacy should be
considered throughly. More will be added to this section as the protocol is
more fully fleshed-out.

## Ideas for tackling privacy.

* Requests must have an exact set of explicitly defined HTTP headers so that
  there is no variation between clients.
* All servers that service OpenBeacon requests must act as proxies to other
  OpenBeacon servers. Then a server will only ever know your location or your
  identity, but never both at the same time.

# Going Further
Obviously this is from from a fully fleshed out spec, comments of all kinds are
welcome. For now, use the github issues tab to submit suggestions, questions,
and abuse.

I'll be adding more of my thoughts to this over the next few days.
