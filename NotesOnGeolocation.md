# Notes On Geo-location

Geo-location provides a fairly accurate geographic location of a specified IP address. Specifically, when used in conjunction with DNS, it provides the geographic location of the DNS client that issued the request directly to your DNS server. This "local" DNS client is called an LDNS server.

The LDNS server is typically part of a large set of DNS servers used by ISPs to provide DNS service to their customers (your visitors). However, the LDNS IP address could be that of an arbitrarily chosen DNS server, or the DNS client on the visitor's computer itself.

The accuracy of geo-location DNS relies on the geographic correlation between LDNS servers and visitors. Thus, the location that you will receive from geo-location will typically only be accurate within a certain radius (perhaps a mile, perhaps much more). For reference, this accuracy radius is estimated by the MaxMind™ Accuracy Radius database.

According to a recent test we ran on social network web traffic, the number of unique visitor IPs was roughly 200 times the number of unique LDNS IP addresses over the same period. In other words, there was 1 LDNS server for every 200 visitor IP addresses. These IP addresses could also be NAT IPs, so the exact relationship between IP addresses and unique visitors is somewhat opaque.

Also, some large IP ranges are located entirely at a predefined central spot for the country in which that IP range is located. For example, many large US IP ranges are almost entirely located at 38,-97 with no other information (besides country) provided. 38,-97 is in the middle of a small field near Potwin, Kansas&mdash;the geographic center of the continental United States. Subsets of these large (end-user) ranges are broken out to have enhanced accuracy, but the database isn't perfect. This implies two things:

1. Anyone in the USA (for example) using these large ranges (millions of IPs) could show up as coming from Potwin, KS (or rather, 38,-97).
2. Using geo-location radiuses, you could get very surprising jumps in traffic. For example, including 38,-97 in your radius would cause a large jump.

These accuracy caveats are primarily applicable to a database that provides longitude/latitude data, or city granularity (e.g., MaxMind™ City). For other databases, the data provided is at a much larger granularity, and will be much less likely to be affected by the above accuracy issues.

Geo-location does provide a good estimate of your visitors' geographic location based on the generally good correlation between the location of a visitor and their LDNS server.

Geo-location does not provide a direct, accurate location of a specific visitor. The accuracy radii of both the LDNS location and a visitor's location with respect to their LDNS server can vary widely, and in rare cases can be as inaccurate as half a country away.

It's important to keep the above limitations in mind, especially the distinction between LDNS servers and visitors.
