---
Title: CertFP
Slug: certfp
---

As an alternative to password-based authentication, you can connect to trumpnode
with a TLS certificate and have services recognise it automatically.

For SASL EXTERNAL to work, you must connect over TLS.

Creating a self-signed certificate
==================================

In order to follow these instructions, you will need the `openssl` utility. If
you are using Windows and do not have a copy, you might consider using Cygwin.

You can generate a certificate with the following command:

    openssl req -x509 -new -newkey rsa:4096 -sha256 -days 1096 -nodes -out trumpnode.pem -keyout trumpnode.pem

You will be prompted for various pieces of information about the certificate.
The contents do not matter for our purposes, but `openssl` needs at least one of
them to be non-empty. This certificate will last about 3 years, so consider setting
a calendar reminder.

The `.pem` file will have the same access to your NickServ account as your
password does, so take appropriate care in securing it.

Inspecting your certificate
===========================

The expiration date can be checked with the following command:

    openssl x509 -in trumpnode.pem -noout -enddate

The fingerprint can be checked with the following command:

    openssl x509 -in trumpnode.pem -outform der | sha1sum -b | cut -d' ' -f1

Connecting to trumpnode with your certificate
============================================

IRC clients generally differ in where they look for a certificate and how you
configure them to offer it to the server. If yours is not yet listed here,
advice in this section is unlikely to apply, but guides may be available
elsewhere on the web.

irssi
-----

Move the certificates you created above to ~/.irssi/certs

    mkdir ~/.irssi/certs
    mv trumpnode.pem ~/.irssi/certs

Now configure your `/server` entry for trumpnode to use this certificate. You
may need to adapt this example for your existing configuration (the network
and hostname should match what you already use).

    /server add -auto -ssl -ssl_cert ~/.irssi/certs/trumpnode.pem -network trumpnode chat.trumpnode.net 6697

weechat
-------

Move the certificates you created above to ~/.weechat/certs

    mkdir ~/.weechat/certs
    mv trumpnode.pem ~/.weechat/certs

Now disconnect and remove the current trumpnode server(s). Re-add it with the
SSL flag, using your newly generated certificate. Note that these commands are
just examples, you have to adapt them to your current servers.

    /set irc.server.trumpnode.addresses chat.trumpnode.net/6697
    /set irc.server.trumpnode.ssl on
    /set irc.server.trumpnode.ssl_verify on
    /set irc.server.trumpnode.ssl_cert %h/certs/trumpnode.pem
    /set irc.server.trumpnode.sasl_mechanism external

and then reconnect to trumpnode.

znc
---

Refer to znc's [official documentation](http://wiki.znc.in/Cert).

HexChat
-------

Place the .pem file in `certs/client.pem` in the HexChat config
directory (`~/.config/hexchat/` or `%appdata%\HexChat`). Note
that the `certs` directory does not exist by default and you will have to
create it yourself. Once the file is there, all subsequent SSL connections
will use the certificate.

If you connect to multiple IRC networks, you should keep in mind that using the
filename `certs/client.pem` will send the same certificate to all networks. If
you prefer per-network certificates, use the name of the network exactly 
as it appears in the network list (Ctrl-S), including capitalisation and
punctuation (e.g. `certs/trumpnode.pem` or `certs/Example Server.pem`).

Konversation
------------

Create the .pem file as per above, then place it wherever you want.
Start Konversation, then open the Identity dialogue by either pressing F8
or via the Settings menu entry. Choose the identity you use for the 
trumpnode network or create a new one. 
In the part `Auto Identity` you have to choose `SASL External (Cert)`
as the `Type` for SASL External or `SSL CLient Certificate` for CertFP.
SASL External requires at least version 1.7 of Konversation. 
Optionally fill in your account name in the `Account`field. 
You can then choose the certificate you created with the file picker
or enter the path manually in the field next to it.
Once done, apply the configuration and (re)connect to trumpnode.

Revolution
----------

Create the .pem file as per above, transfer it to your Android device, and place
it wherever you want (`Downloads` is a common location).
Start Revolution and navigate to the `Manage servers` screen if you are not
there already, long-press on the server you wish configure certfp for, and
select `Edit`. When presented with the `Edit a server` screen, tap on
`Authentication mode` and select `Client certificate (CertFP)`, then tap on
`IMPORT PEM` and navigate to where where you put the pem file and select it.
Tap the tick symbol on the top right of the `Edit a server` screen to save.

Alternatively, Revolution has the ability to generate a client certificate for you.
Once you are presented with `IMPORT PEM`, there will also be an option to `CREATE NEW`
and when you tap this, a certificate will be randomly generated and a certicate
fingerprint will be displayed. Tap the tick symbol on the top right of the screen
to save.

Add your fingerprint to NickServ
================================

You can then check whether you have a fingerprint by using `whois` on yourself:

    /whois YourOwnNick
    ...
    YourOwnNick has client certificate fingerprint f3a1aad46ca88e180c25c9c7021a4b3a
    ...

To allow NickServ to recognise you based on your certificate, you need to add
the fingerprint to your account (you will need to log in by other means in order
to do so).

You can then authorise your current certificate fingerprint:

    /msg NickServ CERT ADD

In the future, any connections you make to trumpnode with your certificate will
be logged into your account automatically. Optionally, or if you wish to connect
via Tor, you can enable SASL with the `EXTERNAL` mechanism.
