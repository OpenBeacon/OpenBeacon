# OpenBeacon

This documents does not define a protocol, but rather specifies some design
goals for an open protcol for beaconing. It is inspired by the
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

# Features
## Packet Format

* A defined message format, should include:
  - OpenBeacon Marker, 1 Octet, 0x0B
  - Refrence RSII @ 1m, 1 Octets
  - URI, Up To 26 Octets (Must Contain At Least One '.' and One '/')


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

## Other things to consider:
* Encode domain as UTF-8, then use a binary value as a path, base 64 encoded in
  HTTP for additional usable space.
  - '/' is used as terminator or use 4 bits for domain length minus 4.(min 4, 
    max 20).


# Going Further
Obviously this is from from a fully fleshed out spec, comments of all kinds are
welcome. For now, use the github issues tab to submit suggestions, questions,
and abuse.

I'll be adding more of my thoughts to this over the next few days.