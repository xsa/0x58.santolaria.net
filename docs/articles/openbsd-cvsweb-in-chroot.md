# OpenBSD: Running *CVSweb* in chroot

## Software versions used

- [OpenBSD 3.3-stable](http://www.openbsd.org/)
- [Apache 1.3.27](http://www.apache.org/) (from the base install)
- [CVSweb 2.0.6](http://www.freebsd.org/projects/cvsweb.html)

## Installing *cvsweb*

```
$ cd /usr/ports/devel/cvsweb/
$ sudo make install clean
```
## Creating the directory structure and copying the required files into the chroot environment

In order to make CVSweb operate in a chroot environment, it is necessary to copy all of the
relevant tools, libraries, and perl modules that CVSweb employs into `/var/www`.
This is because `/var/www` becomes the root, or "`/`" directory for Apache and all the programs executed
from Apache, for instance, a module or CGI script like CVSweb.

First, we create the basic directory structure:
```
# cd /var/www
# mkdir {tmp,usr}
# chown www:www tmp     (needs to be writeable for the www user)

# cd /var/www/usr
# mkdir -p {bin,lib,libdata/perl5,libexec}

# cd /var/www/usr/libdata/perl5
# mkdir -p {File,IPC,Time,warnings,`machine`-openbsd/5.8.0}
```  

Now, the required binaries:
```
# cd /var/www/usr/bin
# cp -p /usr/bin/{co,cvs,diff,perl,rcsdiff,rlog,uname} .
```

Next, the libraries that the binaries are linked to:

**NOTE**: Wildcards will, of course, copy any old, unused libraries
that are lurking around (for example, from an upgrade). If you wish to diminish
the number of libraries required to be copied, there are several methods available (see below).

```
# cd /var/www/usr/lib
# cp -p /usr/lib/lib{asn1,c,crypto,des,gssapi,kafs,krb,krb5}.so* .
# cp -p /usr/lib/lib{m,perl,util,z}.so* .
```

1. If you have old dynamic libraries around and only want to copy the ones which will actually be used, run [ldd(1)](http://www.openbsd.org/cgi-bin/man.cgi?query=ldd&sektion=1) on every needed binary.
1. By default, cvs is built with Kerberos support. You can disable Kerberos support by adding the following lines to [mk.conf(5)](http://www.openbsd.org/cgi-bin/man.cgi?query=mk.conf&sektion=5):

```
KERBEROS=no
KERBEROS5=no
```

and then rebuilding the cvs program. This will reduce the number of required shared libraries by five.

Now, the run-time link-editor:
```
# cd /var/www/usr/libexec
# cp -p /usr/libexec/ld.so .
```
And finally, the perl modules:
```
# cd /var/www/usr/libdata/perl5
# cp -p /usr/libdata/perl5/{Carp,Exporter,Symbol,base,integer}.pm .
# cp -p /usr/libdata/perl5/{strict,warnings,vars}.pm .
# cp -p /usr/libdata/perl5/File/Basename.pm ./File/
# cp -p /usr/libdata/perl5/IPC/Open{2,3}.pm ./IPC/
# cp -p /usr/libdata/perl5/Time/Local.pm ./Time/
# cp -p /usr/libdata/perl5/warnings/register.pm ./warnings/

# cd /var/www/usr/libdata/perl5/`machine`-openbsd/5.8.0
# cp -p /usr/libdata/perl5/`machine`-openbsd/5.8.0/{Config,Cwd}.pm .
```
## Where to put my cvsroot now?

You will need to put your CVSROOT in the chroot environment just like what was done with
the binaries and libraries. The logical choice is to use `/var/www/cvs/`.
If you don't have enough space on your `/var/www` partition for your CVS repository,
you can always use an NFS share mounted over the loopback interface.

**NOTE**: The ownership of the CVS repository does **not** have to be changed. The files simply
have to be readable by the www user. If they were owned by the www user and a vulnerability
was found in CVSweb, the CVS repository could be compromised.

## Editing the cvsweb.conf

We'll need to edit the `/var/www/conf/cvsweb/cvsweb.conf` file so the CVSweb will find the correct path to our CVSROOT. Do as follows:

```# $EDITOR /var/www/conf/cvsweb/cvsweb.conf```

Then find the following (line 46):

```'local'   => ['Local Repository', '/home/cvs'],```
  
and replace it with:

```'local'   => ['Local Repository', '/cvs'],```

## Editing the CVSweb cgi script

We'll need to edit the `/var/www/cgi-bin/cvsweb` cgi script to make it find our config file. Do as follows:

```# $EDITOR /var/www/cgi-bin/cvsweb```

Then find the following (line 160):

```for ("$mydir/cvsweb.conf", '/var/www/conf/cvsweb/cvsweb.conf') {```
  
and replace it with:

```for ("$mydir/cvsweb.conf", '/conf/cvsweb/cvsweb.conf') {```

## Let's finish

Hopefully after following all the steps in this document, you can view your CVS repository
through the CVSweb interface with the a URL like `http://yoursite.com/cgi-bin/cvsweb/`

*Published on Jun. 01, 2006 on [open.bsdcow.net](https://github.com/xsa/open.bsdcow.net/)*
