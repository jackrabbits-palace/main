# Commands for Manually Configuring TLS/Tor SASL

Since `./configure-certs` mysteriously won't work for freenode or oftc, we have provided the commands necesary to manually them it here.

## Freenode

Get password and cert fingerprint from a shell:

``` shell
$ jrp # or however you get to project root
$ cd bin/services/irc
$ source util
$ get-srv-cfg freenode password
  <prints your password -- copy it!>
$ openssl x509 -in ~/.weechat/certs/freenode.pem -outform der | sha1sum -b | cut -d' ' -f1
  <prints your fingerprint - copy it!>
```

Start weechat:

``` shell
$ weechat
```
Issue config commands inside weechat:

```
/connect freenode
/buffer freenode

/msg nickserv register ${password} ${email}
/msg nickserv identify ${password}
/msg nickserv cert add ${cert}

/set irc.server.freenode.ssl_cert %h/certs/freenode.pem
/set irc.server.freenode.sasl_mechanism external
/set irc.server.freenode.ssl_priorities 'NORMAL:-VERS-TLS-ALL:+VERS-TLS1.0:+VERS-SSL3.0:%COMPAT'
/set irc.server.freenode.ssl_verify off
/set irc.server.freenode.proxy 'tor'

/reconnect freenode
```

note, for v3 onions, you might need to use:

```
/set irc.server.freenode.addresses 'ajnvpgl6prmkb7yktvue6im5wiedlz2w32uhcwaamdiecdrfpwwgnlqd.onion/7000'
```

## OFTC

Get password and cert:

``` shell
$ get-srv-cfg oftc password
  <prints your password -- copy it!>
$ openssl x509 -in ~/.weechat/certs/oftc.pem -outform der | sha1sum -b | cut -d' ' -f1
  <prints your fingerprint - copy it!>
```

Start weechat:

``` shell
$ weechat
```

Issue config commands inside weechat:

```
/connect oftc
/buffer oftc
/msg nickserv identify ${password}
/msg nickserv cert add ${cert}

/set irc.server.oftc.ssl_cert %h/certs/oftc.pem
/set irc.server.oftc.sasl_mechanism external
/set irc.server.oftc.ssl_priorities 'NORMAL:-VERS-TLS-ALL:+VERS-TLS1.0:+VERS-SSL3.0:%COMPAT'
/set irc.server.oftc.proxy 'tor'

/reconnect oftc
```
