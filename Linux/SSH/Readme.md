#Assumptions#
For the purposes of this document, we will assume:

* the SSH Server address is 10.0.0.2
* the username we're logging in as is 'clientuser0'
* and the group we want to create for allowing SSH access is 'sshusers'

#Installation#
This particular document is meant to be distro-agnostic, therefor I will not give any copy/paste commands that would be distro specific. First you need to install ssh-server (openSSH). If you are running a server oriented distro, this is more than likely already installed. If you are running an end-user oriented distro, it is more than likely not installed yet.

#Configuration Hardening#
What I like to do is specify settings in the config, even if they are the defaults. This is just in case the future changes the defaults. In addition, I like to disable root logins, and only allow key-based authentication, and only allow users to SSH in if they are a member of a certain group. Below is my personal go-to /etc/ssh/sshd_config
```
Port 22
Protocol 2
HostKey /etc/ssh/ssh_host_rsa_key
HostKey /etc/ssh/ssh_host_ecdsa_key

KexAlgorithms curve25519-sha256@libssh.org,ecdh-sha2-nistp521,ecdh-sha2-nistp384,ecdh-sha2-nistp256,diffie-hellman-group-exchange-sha256
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes192-ctr,aes128-ctr
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,umac-128-etm@openssh.com,hmac-sha2-512,hmac-sha2-256,umac-128@openssh.com

SyslogFacility AUTHPRIV
PermitRootLogin no
MaxSessions 2
RSAAuthentication yes
PubkeyAuthentication yes

AuthorizedKeysFile .ssh/authorized_keys
PermitEmptyPasswords no
#PasswordAuthentication no
ChallengeResponseAuthentication no

UsePAM yes
AllowGroups sshusers
X11Forwarding no

UsePrivilegeSeparation sandbox
UseDNS no

AcceptEnv LANG LC_CTYPE LC_NUMERIC LC_TIME LC_COLLATE LC_MONETARY LC_MESSAGES
AcceptEnv LC_PAPER LC_NAME LC_ADDRESS LC_TELEPHONE LC_MEASUREMENT
AcceptEnv LC_IDENTIFICATION LC_ALL LANGUAGE
AcceptEnv XMODIFIERS
Subsystem sftp /usr/libexec/openssh/sftp-server
```

The list of KeyAlgorithms/Ciphers/MACs I just copy and paste from https://wiki.mozilla.org/Security/Guidelines/OpenSSH#OpenSSH_server . After that I will run the following commands

```
ssh-keygen -A
#whatever command your distro uses to restart the sshd service
ssh-keygen -A
#whatever command your distro uses to restart the sshd service
```

#Creating group for SSH Users#
```
groupadd sshusers
```

Now to allow SSH access to a user, just add them to the sshusers group
```
usermod -a -G sshusers clientuser0
```

#Before you exit this console session (if you're doing this remotely)#
We need to do a key exchange between your user account on a client machine and this SSH server (because we will disable password based authentication once this works). From the ssh client run this
```
ssh-copy-id clientuser0@10.0.0.2
```

Now try to SSH to 10.0.0.2 as the clientuser0 user in another session and it should authenticate successfully without a password prompt. If this has happened, you are done. Now re-edit the /etc/ssh/sshd_config and uncomment the following line
```
#PasswordAuthentication no
```

For new users, they will need to generate their user ssh keys and give you their public key. Once you create their user account on the SSH server, add their public key to their ~/.ssh/authorized_keys file