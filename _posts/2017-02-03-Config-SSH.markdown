---
layout: default
type: post
title: SSH Usage
excerpt: SSH is a power full tools for configuration management
---

# Introduction
SSH, which is an acronym for Secure SHell, was designed and created to provide the best security when accessing another computer remotely. Not only does it encrypt the session, it also provides better authentication facilities, as well as features like secure file transfer, X session forwarding, port forwarding and more so that you can increase the security of other protocols. It can use different forms of encryption ranging anywhere from 512 bit on up to as high as 32768 bits and includes ciphers like AES (Advanced Encryption Scheme), Triple DES, Blowfish, CAST128 or Arcfour. Of course, the higher the bits, the longer it will take to generate and use keys as well as the longer it will take to pass data over the connection.

# Negotiate Process
* The client begins by sending an ID for the key pair it would like to authenticate with to the server.
* The server check's the authorized_keys file of the account that the client is attempting to log into for the key ID.
* If a public key with matching ID is found in the file, the server generates a random number and uses the public key to encrypt the number.
* The server sends the client this encrypted message.
* If the client actually has the associated private key, it will be able to decrypt the message using that key, revealing the original number.
* The client combines the decrypted number with the shared session key that is being used to encrypt the communication, and calculates the MD5 hash of this value.
* The client then sends this MD5 hash back to the server as an answer to the encrypted number message.
* The server uses the same shared session key and the original number that it sent to the client to calculate the MD5 value on its own.It compares its own calculation to the one that the client sent back. If these two values match, it proves that the client was in possession of the private key and the client is authenticated.


# Basic Usage
### Generate
```
ssh-keygen -t dsa -b 4096
```
### Exchange Key
```
scp ~/.ssh/id_dsa.pub username@arvo.suso.org:.ssh/authorized_keys
```
If you have **ssh-copy-id** installed, you can just run
```
ssh-copy-id username@remote.host.name
```
### Login Remote Instance
After puting your public keys in remote instance, then you can run:
```
ssh username@remote.host.name
```
to login remote instance

### Excute Cammand On Remote Host
```
ssh username@remotehost.net ls -l /
```

### Using Scp
**Local to remote**
```
scp report.doc username@remote.host.net:
```
**Remote to local**
```
scp username@remote.host.net:report.doc report.doc
```

### Test Connection
```
ssh -T git@host -v
```
# Server Configurations
* 将PasswordAuthentication no加入server的/etc/ssh/sshd_config来阻止使用密码ssh登陆
* 通过`Port 4444` 来使用自定义端口
* 限定用户 `AllowUsers user1 user2`
* 限定用户组 `AllowGroups sshmembers`
* Disable root login `PermitRootLogin no`

# Client Configuration
Ssh client configuration lies in **~/.ssh/config**   
Example common settings like followings:
```
Host github-default
    HostName github.com
    User username
    IdentityFile ~/.ssh/gitkey_default
```
* Host   
Only the host of the ssh command match this value, will the following rule applied to this command
HostName
The real hostname the command will try to connect
* Username  
The real Username the command will use
* IdentityFile  
Location of the private key

For other configuration, please refer to [ssh man page](https://linux.die.net/man/5/ssh_config)
# Reuse Session & Tunel
```Shell
Host *
    ControlMaster auto
    ControlPath ~/.ssh/sockets/%r@%h-%p
    ControlPersist 600
```

**ControlMaster auto**

The ControlMaster option is one of SSH's best kept secrets. It instructs SSH to reuse an existing connection to the server if it already exists. This means that, if you run ssh example.com, open another terminal, and run ssh example.com there, the two sessions will be transported over the same underlying connection. The second session starts much more quickly because the SSH handshake has already been completed.

**ControlPath ~/.ssh/sockets/%r@%h-%p**

ControlPath is a specification of where to create the control socket on your filesystem. If you use the value provided here, make sure to manually mkdir ~/.ssh/sockets. At any time, you can manually remove the control socket (using plain-old rm), and the next ssh invocation will establish a new connection. This is especially useful if you've recently reopened your laptop and your SSH connections haven't yet figured out that the server terminated them.

**ControlPersist 600**

Without ControlPersist, once the first SSH session you open is closed, all other sessions on that connection are closed as well. This can lead to a variety of surprising behavior: if you exit the initial SSH shell while other sessions are sharing the connection, the process will just hang. If you send it a Ctrl-C, all of those other sessions will be abruptly terminated.

# Further Use
### Set Tunnel From Local To Remote Server
```
ssh -i key -f -N -L 8888:example.com:80 username@remote_host
```
Pattern
```
ssh -i key -f -N -L your_port:site_or_IP_to_access:site_port username@host
```
`-f` options let SSH to go into the background before executing    
`-N` options let SSH not open a shell or execute a program on the remote side

When you visit 127.0.0.1:8888 on your local computer, then you will go through the tunnel and ly reopened your laptop and your SSH connections haven't yeTuigured out that the server terminated them.
### Set Tunnel From Remote To Local
```
ssh -f -N -R 8888:example.com:80 username@remote_host
```
Pattern
```
ssh -R remote_port:site_or_IP_to_access:site_port username@host
```
When you visit 127.0.0.1:8888 on your remote computer, then you will go through the tunnel and access the example.com:80 via your local computer

### Set Dynamic Tunneling to a Remote Server
```
ssh -f -N -D 7777 username@remote_host
```
From here, you can start pointing your SOCKS-aware application (like a web browser), to the port you selected. The application will send its information into a socket associated with the port.

The method of directing traffic to the SOCKS port will differ depending on application. For instance, in Firefox, the general location is Preferences > Advanced > Settings > Manual proxy configurations. In Chrome, you can start the application with the --proxy-server= flag set. You will want to use the localhost interface and the port you forwarded.

# ProxyCommand
Some times you will protect your servers behind a bastion instance, or event 3 or 4 layers like following:

![image](https://cloud.google.com/solutions/images/bastion.png)
If you want ssh onto internal instances, you need jump from bastion and then do another ssh command.   
It's not fancy isn't it?
It`s time to embrace proxy command. You can just run command as followings:
```
ssh -o ProxyCommand='ssh -W 10.0.1.1:22 bastion-user@10.0.1.4' internal-user@10.0.1.1
```
It will setup a tunnel forward socket from local to internal via bastion, and then you `internal-user@10.0.1.1`, it will be forwarded to the internal instances.

Done.

Conclusion

Let just use the former problem as an example we can set our ssh configuration file like this
```
Host */*
    ControlMaster auto
    ControlPath ~/.ssh/sockets/%r@%h-%p
    ControlPersist 600
    ProxyCommand ssh %r@$(dirname %h) -W $(basename %h):%p
```

Then you can run `ssh user@bastion/internal` and it will directly ssh onto the internal instance
