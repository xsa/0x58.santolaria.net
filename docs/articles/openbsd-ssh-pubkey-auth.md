# OpenBSD: SSH pubkey authentication

In this document only SSH protocol v2 key generation is described, hope it satisfies you.

## Software versions used.
- [OpenBSD 3.2-current](http://www.openbsd.org/), from 13 December 2002 snapshots
- [OpenSSH 3.5](http://www.openssh.com/) (being in the base system)

## Generating your keys
If you want to use the SSH pubkey authentication feature, you will, of course, have to create your public and private keys.
Let's see how to generate those for the SSH protocol v2 with a DSA encryption.
To accomplish this we will use [ssh-keygen(1)](http://www.openbsd.org/cgi-bin/man.cgi?query=ssh-keygen&sektion=1):

`$ ssh-keygen -t dsa`

Which will output:
```
Generating public/private dsa key pair.
Enter file in which to save the key (/home/xsa/.ssh/id_dsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/xsa/.ssh/id_dsa.
Your public key has been saved in /home/xsa/.ssh/id_dsa.pub.
The key fingerprint is:
6b:bf:7d:83:b3:87:22:56:55:13:d1:df:1b:4d:d2:0b xsa@core
 ```
When [ssh-keygen(1)](http://www.openbsd.org/cgi-bin/man.cgi?query=ssh-keygen&sektion=1) asks
you in which file you want it to save the key, just press *<return>* (we will stick with the default path).
Then when you'll be asked for a passphrase enter one, and confirm it by entering it again one step later.

**NOTE**: no passphrase can be entered by just pressing *<return>*.
This is often used when the concerned account is used for automated data, backups transfers.
  
Now you are done with generating your keys, let's have a closer look at them:

```
$ ls -l ~/.ssh/id_*
-rw-------  1 xsa  xsa  744 Dec 27 11:41 .ssh/id_dsa
-rw-r--r--  1 xsa  xsa  598 Dec 27 11:41 .ssh/id_dsa.pub
```

The `id_dsa` key is your **_private_** key, which means that **you should keep this file away from other people**,
also, be sure it's only readable/writable by you and _only_ you ([chmod(1)](http://www.openbsd.org/cgi-bin/man.cgi?query=chmod&sektion=1) 0600).

The `id_dsa.pub` key is your **_public_** key, which will be added on systems you want to
have access to. We will see how to add that key later.

**NOTE**: `~/.ssh/` should be [chmod](http://www.openbsd.org/cgi-bin/man.cgi?query=chmod&sektion=1)'d 0700, for security matters.

`drwx------  2 xsa   xsa     512 Dec 27 21:14 .ssh/`

## Placing the key on the remote server.
To be able to log into another system using your keys,
you will first have to place your public key on the remote machine in a file called
`authorized_keys` (for protocol v2) which is located in your `~/.ssh/` directory.

Do as follows:

`$ cat .ssh/id_dsa.pub | ssh newmachine "cat >> .ssh/authorized_keys"`

## Configure the OpenSSH SSH daemon (sshd).

To configure the [OpenSSH SSH daemon (sshd)](http://www.openssh.org/), you'll only have to edit
[`/etc/ssh/sshd_config`](http://www.openbsd.org/cgi-bin/man.cgi?query=sshd_config&sektion=5)
(since OpenBSD 3.1 that's the location of this file, [before](http://www.openbsd.org/faq/upgrade-minifaq.html#3.0.4) it was `/etc/sshd_config`).

Let's have a closer look at it, first [su](http://www.openbsd.org/cgi-bin/man.cgi?query=su&sektion=1) to root:

```
$ su -
Password:
#
```

Then, edit [`/etc/ssh/sshd_config`](http://www.openbsd.org/cgi-bin/man.cgi?query=sshd_config&sektion=5):

`# vi /etc/ssh/sshd_config`
  
and set the following options:

```
PubkeyAuthentication    yes
AuthorizedKeysFile      .ssh/authorized_keys
```

You also might want to disable the `PasswordAuthentication` option, so people can only login through key authentication:

  PasswordAuthentication no
  

Now you are done with the configuration, restart your [sshd](http://www.openbsd.org/cgi-bin/man.cgi?query=sshd&sektion=8):

`# kill -HUP $(/var/run/sshd.pid)`

## Let's test our setup.
To test what we have configured so far, the easiest way would be to connect to the remote machine:

```
$ ssh user@host
Enter passphrase for key '/home/user/.ssh/id_dsa':
```

Enter your passphrase and ... do whatever you need to on the machine :-)

**NOTE**: to increase verbosity during the connection to the host, add the `-v` option.
Multiple `-v` options increases the verbosity. Maximum is 3.

## Managing your keys with ssh-agent
[ssh-agent(1)](http://www.openbsd.org/cgi-bin/man.cgi?query=ssh-agent&sektion=1) is a
program to hold private keys used for public key authentication (RSA, DSA).
The idea is that `ssh-agent` is started in the beginning of an X-session or a login session,
and all other windows or programs are started as clients to the `ssh-agent` program.
Through use of environment variables the agent can be located and automatically
used for authentication when logging in to other machines using ssh(1).

Initially, the agent does not have any private keys. Keys are added using [ssh-add(1)](http://www.openbsd.org/cgi-bin/man.cgi?query=ssh-add&sektion=1).
When run without any arguments, it adds the default location files
(`~/.ssh/id_rsa`, `~/.ssh/id_dsa` and `~/.ssh/identity`).
If your private key is encrypted with a passphrase,
[ssh-add(1)](http://www.openbsd.org/cgi-bin/man.cgi?query=ssh-add&sektion=1) will prompt you for it:

```
$ eval $(ssh-agent)
$ ssh-add ~/.ssh/id_dsa
Enter passphrase for /home/user/.ssh/id_dsa:
Identity added: /home/user/.ssh/id_dsa (/home/user/.ssh/id_dsa)
$
```

It then sends the identity to the agent. The agent can manage several identities
and to have a look at the ones held by the agent, just run the `ssh-add -l` command.

For the future connections to the remote machine(s) having your key, you
will notice that you are not prompted anymore to enter your passphrase.
That's the use of [ssh-agent(1)](http://www.openbsd.org/cgi-bin/man.cgi?query=ssh-agent&sektion=1).

```
$ ssh -v user@host
[...]
$
```
... you are now on the remote host without entering any password/passphrase.

If you want to remove all the identities stored in the agent (because you feel quite unsafe),
just run the following command:

```
$ ssh-add -D
All identities removed.
$
```
  
... or read the [ssh-agent(1)](http://www.openbsd.org/cgi-bin/man.cgi?query=ssh-agent&sektion=1) and
[ssh-add(1)](http://www.openbsd.org/cgi-bin/man.cgi?query=ssh-add&sektion=1) manuals for more information.

**NOTE**: The use of [ssh-agent(1)](http://www.openbsd.org/cgi-bin/man.cgi?query=ssh-agent&sektion=1) might be very useful and practical but keep in mind
that if someone gains access to your computer, it will be easy for him to connect to
any of the remote machines you have configured with your keys without entering any password.
So think twice before using [ssh-agent(1)](http://www.openbsd.org/cgi-bin/man.cgi?query=ssh-agent&sektion=1). Indeed if you only have a few keys,
you would rather remember your passphrases and use the `-i` option with [`ssh(1)`](http://www.openbsd.org/cgi-bin/man.cgi?query=ssh&sektion=1). 
  

*Published on Oct. 31, 2003 on [open.bsdcow.net](https://github.com/xsa/open.bsdcow.net/)*
