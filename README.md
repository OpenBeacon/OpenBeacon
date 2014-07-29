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

# Going Further
Obviously this is from from a fully fleshed out spec, comments of all kinds are
welcome. For now, use the github issues tab to submit suggestions, questions,
and abuse.

I'll be adding more of my thoughts to this over the next few days.