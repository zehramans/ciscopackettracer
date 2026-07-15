# Configuring SSH Authentication with AAA

In this lab, SSH was configured so that the router could be accessed securely instead of using Telnet. An AAA server was used to authenticate users before allowing them to log in.

The first step was enabling AAA on the router with the command:

```cisco
aaa new-model
```

This activates the AAA system. Even if user accounts already exist on the AAA server, the router will not use them until AAA is enabled.

Next, the router was configured to communicate with the RADIUS server by specifying the server's IP address and a shared secret. This shared secret must be identical on both the router and the AAA server or authentication will fail.

```cisco
radius-server host 192.168.1.10 key Cisco123
```

The following command was then used to create the authentication method:

```cisco
aaa authentication login default group radius local
```

This tells the router to authenticate login requests using the RADIUS server first. If the server cannot be reached, the router falls back to its own local user database. The word **default** means this authentication method is used automatically unless another method list is configured.

The VTY lines were then configured for remote access.

```cisco
line vty 0 4
 login authentication default
 transport input ssh
```

The command `login authentication default` tells the VTY lines to use the AAA authentication method that was created earlier. The command `transport input ssh` disables Telnet and only allows SSH connections.

Before SSH can work, the router needs a hostname and a domain name because they are required when generating RSA keys.

```cisco
hostname R1
ip domain-name lab.local
```

The RSA keys were generated using:

```cisco
crypto key generate rsa
```

These keys are used only for SSH. They allow the router to encrypt SSH sessions and identify itself to clients. They are different from the key configured for the RADIUS server. The RADIUS key is simply a shared password that allows the router and the AAA server to trust each other, while the RSA keys are used to secure the SSH connection.

Finally, SSH version 2 was enabled.

```cisco
ip ssh version 2
```

After completing the configuration, the router could be accessed securely with:

```bash
ssh -l ccna 192.168.1.1
```

When the connection is made, the router sends the username and password to the RADIUS server. If the credentials are correct, access is granted. If the RADIUS server is unavailable and local authentication has been configured as a backup, the router checks its local user database instead.
