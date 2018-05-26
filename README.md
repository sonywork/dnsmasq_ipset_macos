# Add support for ipv4/ipv6 separate ipset
Just add some header file from [apple-opensource/xnu](https://github.com/apple-opensource/xnu)

Tested with `pf` in Mac OS 10.13.4


The diff:

in `Makefile`

```
top!=pwd
# GNU make way.
top?=$(CURDIR)

+ifeq ($(shell uname), Darwin)
+OSX_HEADERS_CFLAGS = -I$(top)/xnu/last
+endif
```

```
all : $(BUILDDIR)
	@cd $(BUILDDIR) && $(MAKE) \
 top="$(top)" \
- build_cflags="$(version) $(dbus_cflags) $(idn2_cflags) $(idn_cflags) $(ct_cflags) $(lua_cflags) $(nettle_cflags)" \
+ build_cflags="$(OSX_HEADERS_CFLAGS) $(version) $(dbus_cflags) $(idn2_cflags) $(idn_cflags) $(ct_cflags) $(lua_cflags) $(nettle_cflags)" \
```

in `config.h`

```
#elif defined(__APPLE__)
#define HAVE_BSD_NETWORK
#define HAVE_GETOPT_LONG
#define HAVE_SOCKADDR_SA_LEN
/* Define before sys/socket.h is included so we get socklen_t */
#define _BSD_SOCKLEN_T_
/* Select the RFC_3542 version of the IPv6 socket API. 
   Define before netinet6/in6.h is included. */
#define __APPLE_USE_RFC_3542 
+#define PRIVATE
-#define NO_IPSET
+// #define NO_IPSET
```

add files:

```
└── xnu
    └── last
        ├── libkern
        │   └── tree.h
        └── net
            ├── pfvar.h
            └── radix.h
```

build result:

```
$ otool -L src/dnsmasq
src/dnsmasq:
	/usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1252.50.4)
$ src/dnsmasq -v
Dnsmasq version 2.79  Copyright (c) 2000-2018 Simon Kelley
Compile time options: IPv6 GNU-getopt no-DBus no-i18n no-IDN DHCP DHCPv6 no-Lua TFTP no-conntrack ipset auth no-DNSSEC loop-detect no-inotify

This software comes with ABSOLUTELY NO WARRANTY.
Dnsmasq is free software, and you are welcome to redistribute it
under the terms of the GNU General Public License, version 2 or 3.

```

Enjoy!