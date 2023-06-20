# OpenBSD: Using Procmail and Mutt to filter mailing lists

In this tutorial we'll see how to use filters to filter (d'oh) mails coming from
various mailing lists. Have you ever thought about not subcribing to mailing lists
because it would flood your INBOX? Here is one of the solutions available. Enjoy!

## Software versions used

- [OpenBSD 3.2-STABLE](http://www.openbsd.org/32.html)
- [Procmail 3.22](http://www.openbsd.org/cgi-bin/cvsweb/ports/mail/procmail/)
- [Mutt 1.4i](http://www.openbsd.org/cgi-bin/cvsweb/ports/mail/mutt/)

## Installing and configuring Procmail

First of all, what is Procmail? Procmail can be used to create mail-servers, mailing lists, sort your incoming
mail into separate folders/files, preprocess your mail, start any programs upon mail arrival or selectively
forward certain incoming mail automatically to someone.

Let's install it now: change directory to the procmail location in the ports tree, build and install it:

```
# cd /usr/ports/mail/procmail
# make install clean
```

**NOTE**: if you do not have enough diskspace to install from the ports tree, use the
[pkg_add(1)](http://www.openbsd.org/cgi-bin/man.cgi?query=pkg_add&sektion=1) way.

Now that `procmail` is installed, let's configure it to suit our needs.

Create a configuration file in your home directory. Note that your home directory should
have the execute/search bit (o+x) set and `.procmailrc` should be world readable (o+r):

```$ $EDITOR ~/.procmailrc```

**NOTE**: the `$EDITOR` variable must be set to your favorite text editor.
For instance, export `EDITOR=vi` for Bourne Shells users and `setenv EDITOR` vi for C Shells users.

Now edit it with the following content:
```
VERBOSE=off

# Mutt and Elm use 'Mail'; Pine uses 'mail'
MAILDIR=$HOME/Mail/
  
# Directory for storing procmail log and rc files
PMDIR=$HOME/.procmail	# or $HOME/.Procmail for easier TAB-completion

LOGFILE=$PMDIR/log
INCLUDERC=$PMDIR/rc.lists
```

Now we got our own configuration file, let's see about our `rc.lists` file that will contain the actions we
want *procmail* to take when we receive a mail that matches one of our rules:

```$ $EDITOR ~/.procmail/rc.lists```
 
As a first example, we will redirect mails sent by `owner-misc@OpenBSD.org` to the `~/Mail/IN.OpenBSD-misc` mailbox.
Mails that do not match this rule are still delivered to the user's default mailbox (`/var/mail/$LOGNAME`).

```
:0
* ^Sender:.*owner-misc@OpenBSD.org
IN.OpenBSD-misc
```
 
You can, of course, do the same for the any other [OpenBSD.org](http://www.openbsd.org/mail.html) mailing list.

## Installing and configuring mutt

You might already be accustomed to installing OpenBSD ports or packages so we'll do it quick:
```
# cd /usr/ports/mail/mutt
# make install clean
```
Once you got [*Mutt*](http://www.mutt.org/) installed, get you a [default dot-muttrc configuration](http://www.mutt.org/links.html#config)
and let's start editing it so *Mutt* will inform you when new mails have arrived in your different mailboxes.

```$ $EDITOR ~/.muttrc```
 
**NOTE**: `~/.mutt/muttrc` is also a valid location for the configuration file.

and add the following into it:
```
mailboxes ! =IN.OpenBSD-misc =IN.OpenBSD-tech =IN.OpenBSD-ports \
            =IN.OpenBSD-sources-changes =IN.OpenBSD-hppa =IN.OpenBSD-sparc
```
 
This way, *Mutt* will notify you when new mails have arrived in the above mailboxes.
When starting *Mutt*, you'll see the following message in the bottom of your screen:

```- Mutt 1.4i [2] (moo:/var/mail/xsa) 2 more to go.```
 

Now press the `c` key and Mutt will ask you if you want to open the mailbox `IN.OpenBSD-misc` (assuming new mails have entered this mailbox).

```Open mailbox ('?' for list): =IN.OpenBSD-misc```
 
press `Enter` and it will proceed it. If you want to have a look at the other filled mailboxes, just
press the `c` key again and repeat the operation.


## Quick Sendmail configuration

By default, `/etc/mail/localhost.cf` and `/etc/mail/sendmail.cf` do not have the Procmail feature enabled.
To do so we'll have to do a quick hack in the configuration file located in `/usr/share/sendmail/cf/openbsd-localhost.mc`
then rebuild the `localhost.cf` we use:

```
# cd /usr/share/sendmail/cf
# $EDITOR openbsd-localhost.mc
```

In this file, add:

```FEATURE(local_procmail)```
 
and replace:

```MAILER(local)dnl```
 
by

```MAILER(procmail)dnl```
 
Now generate a new `localhost.mc` with the following commands (assuming you are still in `/usr/share/sendmail/cf/`):

```# m4 ../m4/cf.m4 openbsd-localhost.mc > localhost.cf```

Then backup your existing configuration, replace it with the new one and restart [Sendmail](http://www.sendmail.org/):

```
# cp /etc/mail/localhost.cf /etc/mail/localhost.cf.orig
# cp localhost.cf /etc/mail/localhost.cf
# kill -HUP $(head -n1 /var/run/sendmail.pid)
```

Now you have finished with the various configurations,
[subscribe to the OpenBSD.org mailing lists](http://www.openbsd.org/mail.html) and give it a try :-)

## Related links

- [Procmail official homepage](http://www.procmail.org/)
- [Mutt official homepage](http://www.mutt.org/)
- [Han Boetes' Mutt configuration file](http://www.xs4all.nl/~hanb/configs/dot-mutt/muttrc)
- [Sendmail official homepage](http://www.sendmail.org/)
    
*Published on May 21, 2003 on [open.bsdcow.net](https://github.com/xsa/open.bsdcow.net/)*
