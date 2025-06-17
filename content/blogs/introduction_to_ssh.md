+++
date = '2025-04-27T09:01:55+05:30'
title = 'Introduction to SSH'
description = 'An overview of ssh and how to use it'
tags = ['unix']
+++

SSH is a word that you might often see thrown around when talking about servers and connections. It is sometimes used as a noun in sentences but also as a verb. Its colloquial and sometimes unconventional usage in certain contexts might leave you confused on what exactly it means. Is it a program or does it denote an action that we do? Well, its technically the former. SSH abbreviates to "Secure Shell" and it is a program written by Tatu Ylonen to serve as a replacement to the telnet program. Ever since the early days of computing, users often found it necessary to connect to another computer from their local machine. This computer, often called a server, had specific pathways or ports open for users to establish a connection if they had the proper credentials. Telnet was widely used in these early stages but one of its biggest problems was that it transmitted all data in clear text. 

This essentially meant that any malicious actor sniffing the traffic could not only intercept the credentials of the user but also any information that was being passed to and fro. Telent had to be relegated to the past primarily for this reason which made way for its faster and more secure replacement, SSH. The Secure Shell program brought in several mechanisms to address the limitations of telnet. It introduced asymmetric encryption so that all the traffic remained indecipherable. It also protected the integrity of the data that was being transmitted, while also providing alternate ways of authentication. Within a few years of its inception, ssh was adopted worldwide as the de facto standard to establish connections to servers. However unlike its daunting capabilities, using ssh is quite straightforward. 

If you have ssh installed on any operating system, you can access it by opening the terminal and simply typing "ssh". 
You would most likely see an output quite similar to this: 


``` 
usage: ssh [-46AaCfGgKkMNnqsTtVvXxYy] [-B bind_interface] [-b bind_address]
           [-c cipher_spec] [-D [bind_address:]port] [-E log_file]
           [-e escape_char] [-F configfile] [-I pkcs11] [-i identity_file]
           [-J destination] [-L address] [-l login_name] [-m mac_spec]
           [-O ctl_cmd] [-o option] [-P tag] [-p port] [-R address]
           [-S ctl_path] [-W host:port] [-w local_tun[:remote_tun]]
           destination [command [argument ...]]
       ssh [-Q query_option]
```

This enumerates all the different parameters that ssh provides. You would not be needing most of this when you simply want to connect to a server. The syntax for establishing an ssh connection in its generic sense is "ssh user_name@domain_name/ip". If for example I want to connect to the server jetbrains.org with the username martian the syntax would be: 

``` ssh martian@jetbrains.org ```

Depending on the server, this would prompt for a password or directly drop us into a shell if public key authentication is configured. The ip of the server can also be used instead of the domain. 

```ssh martian@100.54.11.24```

This is all the syntax you would need most of the time when connecting to a server. There can be instances where a server is not running its open ssh daemon (a background process listening for connections) on port 22 and the aforementioned commands assume this port when establishing a connection.  In cases where the server is listening for ssh connections in a different port the syntax will have to be slightly modified with the -p flag. 

```ssh martian@jetbrains.org -p 2121```

The flag slightly modifies the behaviour of the command so that it connects to the ssh daemon running on port 2121 in the server. Most users who regularly connect to servers through ssh find it tedious to enter their passwords each time. As I had fleetingly mentioned, SSH also allows for a form of authentication called Public Key and this is mostly what is used. This form of authentication works on the basis of assymetric encryption. On their local machines, users generate a key pair using the command 

```ssh-keygen -t ed25519```

The -t flag specifies the kind of keys you want to create which in this case is "ed25519" but ssh also provides other formats. After issuing the command, the user will be prompted to decide the location of the key and a passphrase. Although the passphrase can be left empty, using one provides better protection as it requires the user to enter the passphrase everytime the keys are used. After this stage, two keys will be generated with the designated name and one of the keys will have a ".pub" extension. For instance: 

```secure_ssh_key secure_ssh_key.pub```

Any kind of information encrypted by one key can only be decrypted by the other and this is the foundation of assymetric encryption. Once both the keys are generated the "ssh-copy-id" command can be used to copy the public key to the server. 

```ssh-copy-id -i secure_ssh_key.pub martian@jetbrains.org``` 

The -i flag denotes the path to the public key that is to be copied to the server. This command only works if the secure_ssh_key.pub file is in the current directory. If the keys are stored elswhere, change to that directory or simply provide the full path to the -i flag. Once the command is executed, the user will be able to authenticate to the server without a password using the command    

```ssh -i secure_ssh_key martian@jetbrains.org```

The -i flag denotes the path to the private key that was generated. If the name of the key was left to the default, the -i flag is most likely not required. However if the name has been changed, -i must be specified to instruct the command which private key to use. Some users might copy the same public key for all their server while others might generate a new one for each of them. Regardless of your choice, it might be wise to use the ~/.ssh/config file to manage all your different servers and keys. Each entry in the file takes the form of: 

```
Host jetb
	HostName jetbrains.org
	User martian
	Port 22
	IdentityFile ~/.ssh/secure_ssh_key
```

Host refers to the term you would use to connect to the server. Hostname is the actual domain or the ip of the server. User and Port are self explanatory and IdentityFile is the path to my private key. With this entry in ~/.ssh/config I can simply connect to jetbrains.org with the following command:

```ssh jetb``` 

This would automatically look for the host "jetb" in config file and apply all the specifications required to make the connection. Although programs like anydesk allow you to perform similar functions through a GUI, using ssh is certainly more immersive. With a few keystrokes you are literally inside a computer that could be millions of miles away and any initial experience of that is nothing less than magical.  

For a novice user ssh might be his first introduction into the world of UNIX so the syntax might seem a little cryptic at first. I think it is important to understand that each command and its syntax have a clear underlying logic to them that can be applied to other commands. The arguments and flags might vary but the essential structure itself remains the same. So this is partly not just an introduction to ssh but also to how commands themselves are in the UNIX world.  



