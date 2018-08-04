# BIND GeoIP
## ARCHIVE of code.google.com/p/bind-geoip

This patch implements support for the MaxMind® geo-location database (http://www.maxmind.com/app/ip-location) and uses the GeoIP C API. This patch was inspired by the patch that used to live at http://www.caraytech.com/geodns/.

This patch was contributed to and substantially incorporated into the ISC BIND 9.10 release.

**PLEASE NOTE**: With Google Code shutting down, these are the final versions of the patch that will be available. If you have an intractable issue or request for certain missing functionality to be forward-ported to 9.10+, consider emailing me on gmail as irridiakb. These patches have been downloaded many thousands of times, so I hope it's been useful! And RIP Slide.

1.4 has been released, which includes fixes for a memory leak in non-threaded mode, broken IPv6 functionality, as well as clean patches for contemporary BIND versions. Downloads for GeoIP are listed here, due to Google Code fail.

CAVEAT: While these pass the bundled bind tests (make test), I'm no longer able to test against significant production traffic. Reports of success or issues would be greatly appreciated.

* Clean patch against 9.8.7-P1: `bind-9.8.7-P1-geoip-1.4.patch`   MD5SUM: `6c078926ab69d71d924a6d1214c5e97a`
* Clean patch against 9.9.5-P1: `bind-9.9.5-P1-geoip-1.4.patch`   MD5SUM: `96fd02a9420681da2e49e0a5b6797aac`

The goals for this patch were to implement the City and other MaxMind® databases, and add thread safety.

Support is implemented for all relevant MaxMind® databases, including IPv6 Country.

The Proxy Detection database is not supported, since it requires the client's IP, not the IP address of the user-local DNS server (LDNS). The Accuracy Radius database is also not supported.

Both threaded and non-threaded operation have been tested. Trillions of production queries have passed through this patch with neither crashes nor leaks.

-- Ken Brownfield

Copyright© 2012 Slide, Inc.
