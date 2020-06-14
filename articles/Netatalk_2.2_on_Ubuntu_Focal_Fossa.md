# Using Netatalk 2.2 on Ubuntu Focal Fossa

## Why?

**TL;DR: It makes no sense at all unless you're forced to use proprietary software that can only make use of the old Netatalk 2.x metadata scheme.**

Ubuntu 20.04 LTS finally ships with Netatalk 3.1.12 so unless you're in that bad situation explained above simply enjoy Netatalk 3 via an ''apt install netatalk''. In our case we need Netatalk 2.x since we're using a commercial backup/sync suite called [Archiware P5](https://p5.archiware.com) which is only compatible with the old Netatalk metadata format. We use P5 Synchronize to sync Mac fileserver contents hourly from Helios EtherShare/PCShare servers to be available through free and open Netatalk as a means of 'permanent backup' on another machine (no, you can't do this with `rsync` or `zfs send|receive` since metadata conversion and encodings translation is needed).

## The challenge ##

Netatalk 2.2.6 from 2017-07-09 is only available as source tarball and was linked against OpenSSL 1.0.x back then. Ubuntu 20.04 uses OpenSSL 1.1.x by default and as such patches are needed to make it work on Focal Fossa.

## The details ##

Install the prerequisits, grab the [Netatalk 2.2.6 tarball](https://sourceforge.net/projects/netatalk/files/netatalk/2.2.6/) and untar it below `/usr/local/src`.

    sudo apt install build-essential libssl-dev libgcrypt20-dev libkrb5-dev libpam0g-dev libwrap0-dev libdb-dev libtdb-dev libacl1-dev avahi-daemon libavahi-client-dev libcrack2-dev
    cd /usr/local/src
    tar xf /path/to/netatalk-2.2.6.tar.bz2

Now grab the patches from NetBSD project that make Netatalk 2.2.6 work with OpenSSL 1.1.x:

    wget https://github.com/obache/lpt1/raw/master/net/netatalk22/patches/patch-etc_uams_uams__randnum.c
    wget https://github.com/obache/lpt1/raw/master/net/netatalk22/patches/patch-etc_uams_uams__dhx__passwd.c
    wget https://github.com/obache/lpt1/raw/master/net/netatalk22/patches/patch-etc_uams_uams__dhx__pam.c
    cd netatalk-2.2.6
    patch -p1 <../patch-etc_uams_uams__dhx__pam.c
    patch -p1 <../patch-etc_uams_uams__dhx__passwd.c 
    patch -p1 <../patch-etc_uams_uams__randnum.c 

Then configure the software for Ubuntu 20.04 (`--enable-systemd`). It should look like this at the end:

    ./configure --enable-systemd --disable-ddp --disable-cups --without-ldap --without-acls
    ...
    Using libraries:
        LIBS = -lpthread  -L$(top_srcdir)/libatalk
        CFLAGS = -I$(top_srcdir)/include -D_U_="__attribute__((unused))" -g -O2 -I$(top_srcdir)/sys
        SSL:
            LIBS   =  -L/usr/lib64 -lcrypto
            CFLAGS =  -I/usr/include/openssl
        LIBGCRYPT:
            LIBS   = -L/usr/lib/x86_64-linux-gnu -lgcrypt
            CFLAGS = 
        PAM:
            LIBS   =  -lpam
            CFLAGS = 
        WRAP:
            LIBS   = -lwrap
            CFLAGS = 
        BDB:
            LIBS   =  -L/usr/lib64 -ldb-5.3
            CFLAGS = 
        ZEROCONF:
            LIBS   =  -lavahi-common -lavahi-client
            CFLAGS =  -D_REENTRANT
    Configure summary:
        Install style:
             systemd
        AFP:
             Large file support (>2GB) for AFP3: yes
             Extended Attributes: ad | sys
        CNID:
             backends:  dbd last tdb
        UAMS:
             DHX     (PAM SHADOW)
             DHX2    (PAM SHADOW)
             RANDNUM (afppasswd)
             clrtxt  (PAM SHADOW)
             guest
        Options:
             DDP (AppleTalk) support: no
             SLP support:             no
             Zeroconf support:        yes
             tcp wrapper support:     yes
             quota support:           yes
             admin group support:     yes
             valid shell check:       yes
             cracklib support:        no
             dropbox kludge:          no
             force volume uid/gid:    no
             ACL support:             no
             LDAP support:            no

Finally it's a `make`, `sudo make install` for your Netatalk 2.x installation being available below `/usr/local/`.

## Configure P5 to work with your self-built Netatalk 2

(TBD)