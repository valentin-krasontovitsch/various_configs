# email with CLI

## background

one may wish to use email to get notifications from scripts and whatnot,
running on a server or laptop.

setting this up turned out to be more complicated than expected.

installing a command line mail client was not enough for me - trying to send an
email to myself at gmail, i cot a connection refused or a timeout. after
googling a bit it turned out that my ISP (Bergen Fiber aka BKK fiber aka
altibox) is blocking traffic to the usual mail ports, like 25, 468, and 587.

but my isp simultaneously provides webmail, and also access to an smtp (and
imap) server. while i personally was not interested in imap, as i don't use
the altibox email to receive email, i only cared about the smtp server, and it
can be setup as a relay.

## setup

i used the `mail` client from the apt package `mailutils` and `postfix` as an
MTA (message transfer agent? don't really know much about the email stack).
Install them using

```
sudo apt-get install -y mailutils postifx
```

Googling I found a facebook discussion where people were asking about altibox,
and there some customer support rep commented that one should use on of the
higher ports, and TLS / SSL.

we setup postfix by editing the config file `/etc/postfix/main.cf` and adding
the following lines:

```
# that's your relay
relayhost = smtp.altibox.no:587

# auth info for the relay
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous

# make sure you don't need to supply a from sender
append_dot_mydomain = yes
# use anything "valid" here, like gmail.com, if you don't own a domain
mydomain = ranzig.freemyip.com
```

after setting up a mail address with password of choice on the web interface of
altibox's mail service, you setup a credentials file as referenced in the
config file above `/etc/postfix/sasl_passwd`, in my case:

```
smtp.altibox.no:587 myusername@altiboxmail.no:some-passw0rd
```

and aferwards run

```
sudo postmap /etc/postfix/sasl_passwd
```

which will do something and create `/etc/postfix/sasl_passwd.db`.

There are probably other ways of setting up authentication, I was lazy and used
these. Remember to alter the owner and access rights, for example as such:

```
sudo chown root:root /etc/postfix/sasl_passwd /etc/postfix/sasl_passwd.db
sudo chmod 0600 /etc/postfix/sasl_passwd /etc/postfix/sasl_passwd.db
```

this is not too restrictive, you can still send mail without sudo, at least on
ubuntu disco.

also remember to restart postfix for it to pickup config changes:

```
sudo service postfix restart
```

this should be enough to be able to send emails from `your_unix_user@hostname`
using

```
echo "hello!" | mail -s 'some subject' me@yahoo.com
```
