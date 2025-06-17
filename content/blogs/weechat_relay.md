+++
date = '2024-10-24T09:01:16+05:30'
title = 'How to set up weechat relay'
description = 'A guide on how to set up weechat relay'
tags = ['unix']
+++

So weechat is one among many clients that is used to connect to an irc network. Unlike other messaging platforms where you recieve a message that was sent to you even if you are not online, IRC functions in way that requires your client to constantly stay connected to the server to recieve messages. So if your user disconnects, you won't be able to receive messages nor will you be able to access past messages that you sent in a particular channel. To circuvment this users often employ an irc bouncer like znc or soju that stays connected to an irc network and "bounces off" the messages it recieves when you connect to it using a client. Weechat affords its own kind of bouncer which is called a relay which provides similar functionalities. 


In order to set up a weechat relay, you first require a server that has weechat installed. You can create an account in any tilde community if you require a free shell account. Once you have access to a server, you simply open weechat and add the servers you would like to connect to. For instance if you would like to connect to libera: 

```/server add libera irc.libera.chat/6697 -tls```

After adding a server you can sometimes directly connect to it by executing: 

```/connect libera```

However, if Libera requires your server to authenticate via SASL you would need to set the sasl username and password using the following commands: 

```/set irc.server.libera.sasl_username "mynick```

```/set irc.server.libera.sasl_password "password```

SASL is a method through which a user can register their nick and password on a particular IRC network so if they perchance disconnect, they can still assume their nickname by sending the password they used to register the nick to NickServ in Libera. Users can message other users or bots using: 

```/query NickServ or /query any_use```

This will open a query window and if the user types help once they start a query with NickServ, they will be greeted with a list of commands they can use to register their nicknames. 

After a successful connection to a server, the user can start setting up the weechat relay using the following commands: 

```/relay add ssl.irc 700```

This creates a relay in our specificed port. It is usually recommended to use a port range that higher and unspecified.

```/secure set relay PASSWOR```
```/set relay.network.password "PASSWORD```

So far, the commands we have executed on the server side instance of weechat has established a connection to an irc network and set up a relay. After executing a "/save", the user can exit weechat and then generate an ssl certificate. SSL or its predecessor TLS, is a mechanism through which two clients establish end to end encryption. The protocol usually sits between the HTTP and TCP layer and facilitates most kinds of encrypted connections. In order for our client to securely connect to the relay in our server, we are required to generate a self signed certificate. In order to do that the user should cd into the weechat config directory which most likely would be "~/.config/weechat/". 

Depending on the weechat version, they should either create an "ssl" or "tls" directory within the config path and then generate a certificate with the name "relay.pem". (Older versions of weechat use the term "tls") 

```
export HOSTNAME=example.org
openssl req -x509 -nodes -newkey rsa:2048 -keyout relay.pem -extensions san_env \
  -subj "/O=WeeChat/CN=$HOSTNAME" \
  -config <(cat /etc/ssl/openssl.cnf <(printf "\n[ san_env ]\nsubjectAltName=DNS:\${ENV::HOSTNAME}")) \
  -days 365 -out relay.pem
```

If the server does not have a hostname, a certificate with jus the ip can also be generated 

```
export IP=192.168.1.2
openssl req -x509 -nodes -newkey rsa:2048 -keyout relay.pem \
  -subj "/O=weechat/CN=my-weechat" \
  -config <(cat /etc/ssl/openssl.cnf <(printf "\n[v3_ca]\nsubjectAltName = @alternate_names\n[alternate_names]\nIP.1 = \${ENV::IP}")) \
   -days 365 -out relay.pem
```


Once the certificate has been generated, the certificate can be loaded on to weechat by opening weechat and executing: 

```/relay sslcertke```

Afer this, the user can connect to the relay using any client which includes weechat as well. If the user chooses weechat as the client, then once again we are required to add a server using the /server command but this time we use the ip of our server where the weechat relay is running instead of the irc network. 

```/server add libera ip_of_host/port_number -ssl -autoconnect```

The user can also use a hostname of the server if it has a registered domain. After this the user can configure the local client to use a password to connect to the relay using: 

```/secure set relay PASSWORD (same as the one used on the server```

Before connecting to the relay, we are also required to set a password for the server using: 

```/set irc.server.libera.password "libera:PASSWORD```

The "libera" prefix before the password tells the relay which network we would like to connect to. We can simply add more networks on the server side by following the same steps and then modify the password according to the name of our network on the client. 

Most often, the server side time does not align with the time zone the client is connecting from. In order to display the time correctly we can set the option:

```/set irc.server_default.capabilities "server-time```

The final step is to ensure that the weechat on the client side accepts the certificate that the server provides. Most likely this will not be the default behaviour because the weechat client does not accept self signed certificate. In order to configure this we simply set: 

```/set weechat.network.gnutls_ca_user "path_to_certificate"```

This particular option should point to a local copy of certificate we generated on the server side. We can also toggle the "tls_verify" or the "ssl_verify" option for the particular network but that would invariably ignore the verification of all certificates. Once this is done the client should be able to connect to the weechat relay and join channels. 
