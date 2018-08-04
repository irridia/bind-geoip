# BIND GeoIP v1.4

This patch implements support for the MaxMind® geo-location database and uses the GeoIP C API. This patch was inspired by the patch at Caraytech, and was incorporated into ISC BIND 9.10 with different syntax.

The goals for this patch were to implement the City and other MaxMind® databases and add thread safety.

Support is implemented for all relevant MaxMind® databases:

* Country
* City
* Region
* ISP
* Organization
* AS number
* Netspeed
* Domain
* Country IPv6

The Proxy and Accuracy Radius databases are not supported, since they should be used with the client's IP, not the IP address of the user-local DNS server (LDNS).

Both threaded and non-threaded operation have been tested. >100B production queries have passed through this patch with neither crashes nor leaks.

## SUPPORTED BIND DISTRIBUTIONS

This patch applies cleanly to and has been heavily tested on BIND production releases 9.6.0-P1 and above. We plan to continue porting the patch to the latest 9.6 and higher BIND production releases, but back-ports for older BIND versions are not currently planned.

## TESTED UNIX DISTRIBUTIONS

This patch has been heavily tested on Ubuntu 8.04 LTS. Other Linux platforms should prove equally stable. While we put significant effort into the portability of this patch, it has not been tested on BSD-derivatives, Solaris, other Unixes, or AIX.

## INSTALLATION

To build BIND with GeoIP support, you will first need to build/install the GeoIP C API (linked above). We recommend 1.4.6 or later; 1.4.5 (at least) was observed to suffer from memory leaks.

After unpacking the BIND distribution, cd into the untarred directory and run:

  `patch -p0 -b < /path/to/this/patch/bind-9.9.5-P1-geoip-1.4.patch`

This will apply the required patches. You can ignore any conflict on version, but we recommend you change the `RELEASEVER` string to denote a patched version.

  `autoconf`

(NOTE: patching configure is much less portable than configure.in. If you do not have access to autoconf, please let us know and we may provide patches against configure.)

To add GeoIP support when running configure, there are two key options:

  `--with-geoip`

This enables support for GeoIP functionality. It looks for GeoIP in standard locations, and will automatically detect if IPv6 support is available.

You can also specify a path to the location of your GeoIP install (within which should exist "lib", "include", and "share" directories):

  `--with-geoip=/usr/local`
  
To enable debug output, use:

  `--with-geoip-debug`

Starting named with `-d3` or higher will provide logging for all attempted GeoIP matching. Adding these debug logging calls is theoretically a performance hit, but even in heavy production, CPU usage does not change in any perceivable way. Enabling this is recommended to debug unexpected behavior.

## CONFIGURATION

GeoIP matches are performed in the BIND `match-clients{}` clause in a BIND view. The match-client{} parameters follow this general pattern:

  `geoip_<DBTYPE>DB_<FIELD>_<VALUE>`

For values that contain spaces, the `_` character should be used instead. For timezone values that contain the `/` character, the `|` character should be used instead.

Note that `match-clients{}` options are ORed.

## EXAMPLES

### Backwards compatibility for Caraytech/geodns and derived patches:

  `country_US;`

### New syntax

```
geoip_countryDB_country_US;
geoip_cityDB_city_San_Francisco;
geoip_cityDB_timezone_America|Chicago;
geoip_cityDB_country3_JAP;
geoip_cityDB_regionname_California;
geoip_cityDB_postal_94118;
```

### "Square" latitude/longitude area

`geoip_cityDB_lat_41.1_lat_43.1_lon_-82.0_lon_-84.1;`

### Latitudinal "stripe" area

`geoip_cityDB_lat_10_lat_11;`

### Longitudinal "stripe" area

`geoip_cityDB_lon_20_lon_21;`

### Lat/lon radius in degrees (adjusted for tapering longitude at the poles)

`geoip_cityDB_lat_80_lon_83.97_radius_1de;`

### Lat/lon radius in miles (adjusted)

`geoip_cityDB_lat_80_lon_73.97_radius_500mi;`

### Lat/lon radius in kilometers (adjusted)

`geoip_cityDB_lat_80_lon_73.97_radius_100km; geoip_orgDB_name_Slide;`

### The mechanics of BIND views and other BIND details are beyond the scope of this document, but the following is an example:

Note this will match ANY city named Paris!

```
view "PARIS" {
  match-clients {
    geoip_cityDB_city_Paris;
  };
  zone "example.com" in {
    type master;
    file "paris.example.com.dns";
  };
};
view "FRANCE" {
  match-clients {
    geoip_cityDB_country_FR;
  };
  zone "example.com" in {
    type master;
    file "france.example.com.dns";
  };
};
view "GERMANY" {
  match-clients {
    geoip_cityDB_country_DE;
  };
  zone "example.com" in {
    type master;
    file "germany.example.com.dns";
  };
};
view "DEFAULT" {
  zone "example.com" in {
    type master;
    file "example.com.dns";
  };
};
```

Views have precedence -- the first matching view will be used. LDNS servers in Paris will see results from the first zone file. LDNS servers in all of FR and all of DE will then see their respective zones, and everyone else will see the final zone file.

## STARTUP

Once you install and start BIND, syslog will report something like the following:

```
Dec 18 17:00:11 u804 named[5162]: Initializing GeoIP Country DB
Dec 18 17:00:11 u804 named[5162]: GEO-106FREE 20090201 Build 1 Copyright (c) 2007 MaxMind LLC All Rights Reserved
Dec 18 17:00:11 u804 named[5162]: Initializing GeoIP City DB Revision 1
Dec 18 17:00:11 u804 named[5162]: GEO-133 20091215 Build 1 Copyright (c) 2009 MaxMind Inc All Rights Reserved
Dec 18 17:00:11 u804 named[5162]: GeoIP Region DB Revision 0 or 1 not available
Dec 18 17:00:11 u804 named[5162]: GeoIP ISP DB not available
Dec 18 17:00:11 u804 named[5162]: Initializing GeoIP Organization DB
Dec 18 17:00:11 u804 named[5162]: GEO-111 20091201 Build 1 Copyright (c) 2009 MaxMind Inc All Rights Reserved
Dec 18 17:00:11 u804 named[5162]: Initializing GeoIP AS DB
Dec 18 17:00:11 u804 named[5162]: GEO-117 20090321 Build 1 Copyright (c) 2007 MaxMind LLC All Rights Reserved
Dec 18 17:00:11 u804 named[5162]: GeoIP NetSpeed DB not available
Dec 18 17:00:11 u804 named[5162]: GeoIP Domain DB not available
Dec 18 17:00:11 u804 named[5162]: Initializing GeoIP Country DB IPv6
Dec 18 17:00:11 u804 named[5162]: GEO-106FREE 20091201 Build 1 Copyright (c) 2009 MaxMind Inc All Rights Reserved
```

GeoIP support was able to find the Country, City Rev1, Organization, AS, and Country IPv6 databases, but not the others.

If you see the `DB not available` message, it means that the GeoIP C API could not find the data file for that database.

Be aware that GeoIP expects a certain naming scheme for the database files. For example, the City database is expected to be named GeoIPCity.dat. See the MaxMind site for more details; particularly here and here.

Also, if you run named in a `chroot()` environment, you will need to make sure the database files are accessible from within the `chroot()` tree. For example, the default MaxMind data directory is `/usr/share/GeoIP`. In a named `chroot()` environment, these files will need to be available in /var/named/usr/share/GeoIP. This duplication can be managed, for example, by a wrapper around geoipupdate that will copy/rsync the files from /usr/share/GeoIP to /var/named/usr/share/GeoIP.

IMPORTANT NOTE: If you specify a geoip rule in `match-clients{}` that reflects an unavailable DB, it will be silently ignored.

If you see a syslog message like:

`error while loading shared libraries: libGeoIP.so.1: cannot open shared object file: No such file or directory`

This means BIND can't find the GeoIP C API. You may need to specify its location via `LD_LIBRARY_PATH`. Alternatively, passing an argument to the `--with-geoip=` configure option will set the runtime library path.

## UPDATING THE MAXMIND® DATABASES

If you purchase the commercial version of the MaxMind® databases (which we recommend) you can use the MaxMind® "geoipupdate" tool to run in-place updates of the databases. Alternatively, you can periodically download updated versions of the free databases.

Simply reload BIND (via `rndc reload` or `SIGHUP`) and the MaxMind® databases will be reloaded without client impact, and the above messages should be repeated.

We chose not to use `GEOIP_CHECK_CACHE` due to the performance hit of a stat() system call for every lookup. This also more closely follows the behavior of BIND, which does not auto-detect changes.

## CITY DATABASE OPTIMIZATION

Results from City database lookups are cached for the duration of a query. City results are represented by a data structure that contains many data points that could be reused by future City lookups. For example, the following would generate two separate GeoIP calls:

`match-clients { geoip_countryDB_country_US; geoip_countryDB_country_FR; };`

Whereas this will only generate one call:

`match-clients { geoip_cityDB_country_US; geoip_cityDB_country_FR; };`

`match-clients{}` clauses that contain multiple cityDB lookups are free from an API perspective. Also, subsequent sequential DNS requests from the same LDNS IP can reuse this structure, though this usage pattern is not common.

NOTE: This optimization will most likely not be implemented for a potential future City IPv6 database, since storing and comparing the necessary IPv6 IP struct (ip6_addr) in TLS will likely create more overhead than the cost of simply re-querying GeoIP.

Copyright (c) 2010, Slide, Inc.
