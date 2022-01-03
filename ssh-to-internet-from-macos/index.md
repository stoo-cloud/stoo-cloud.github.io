# Ssh to Internet From Macos


## Unable to SSH to the Internet 

The first time I try and finally look into it, I have the problem of not being able to SSH out to a host on the Internet from MacOS.  It has happened in 11.15 Catalina and 11.x Big Sur and now in 12.x Monterey versons of MacOS.  It might be my gateway router doing something tricky, or the SSH software installed on the Mac by default, and in the case of the destination tried below it could have been the GCP firewall rules on my VPC network (and sometimes is) with missing default rules perhaps.

Note: I can SSH to hosts on the local LAN just fine.

## Network test

To check TCP connectivity to the `External_HOST_IP` on port `22`, I use the `nc` (ncat ot netcat) command that is installed into MacOS.  It is kind of like telnet but so much more powerful. If you want a more up to date `nc` version then you can use `homebrew` to install it.

```Shell
username@M1pro .ssh % nc  External_HOST_IP 22                                                                      
SSH-2.0-OpenSSH_7.4
^C
```

If we can't get a connection and a banner like this, then there are other firewall or route related issues that needs to be troubleshooted.

In this case I have the `~/.ssh/id_rsa` keyfile and `~/.ssh/id_rsa.pub` public key previously set up using `ssh-keygen` program.

Turning on `-v` for verbose gives alot of nice output, to help determine where an previously issue occurs or what worked.

```Shell
username@M1pro .ssh % ssh -i ~/.ssh/id_rsa -l username  External_HOST_IP   -v
OpenSSH_8.6p1, LibreSSL 2.8.3
debug1: Reading configuration data /Users/username/.ssh/config
debug1: /Users/username/.ssh/config line 2: Applying options for *
debug1: Reading configuration data /etc/ssh/ssh_config
debug1: /etc/ssh/ssh_config line 21: include /etc/ssh/ssh_config.d/* matched no files
debug1: /etc/ssh/ssh_config line 54: Applying options for *
debug1: Authenticator provider $SSH_SK_PROVIDER did not resolve; disabling
debug1: Connecting to  External_HOST_IP   [ External_HOST_IP  ] port 22.
debug1: Connection established.
debug1: identity file id_rsa type 0
debug1: identity file id_rsa-cert type -1
debug1: Local version string SSH-2.0-OpenSSH_8.6
debug1: Remote protocol version 2.0, remote software version OpenSSH_7.4
debug1: compat_banner: match: OpenSSH_7.4 pat OpenSSH_7.0*,OpenSSH_7.1*,OpenSSH_7.2*,OpenSSH_7.3*,OpenSSH_7.4*,OpenSSH_7.5*,OpenSSH_7.6*,OpenSSH_7.7* compat 0x04000002
debug1: Authenticating to  External_HOST_IP  :22 as 'username'
debug1: load_hostkeys: fopen /Users/username/.ssh/known_hosts2: No such file or directory
debug1: load_hostkeys: fopen /etc/ssh/ssh_known_hosts: No such file or directory
debug1: load_hostkeys: fopen /etc/ssh/ssh_known_hosts2: No such file or directory
debug1: SSH2_MSG_KEXINIT sent
ssh_dispatch_run_fatal: Connection to  External_HOST_IP   port 22: Operation timed out
```
## ProxyCommand in the ssh config file

What I have come to realise and forget each time I go about setting up the new environment or machine, is that for some reason I need to push the SSH connection through a proxy tunnel that is handled by `nc`.

After setting up the users `~/.ssh/config` file with the below settings and the important `ProxyCommand` is all that is needed to get me working again.

```Shell
username@M1pro .ssh % cat ~/.ssh/config

Host *
  ServerAliveInterval=120
  TCPKeepAlive yes
  ProxyCommand nc %h %p

```

## Working connection

Trying the same ssh command again..

```Shell
username@M1pro .ssh % ssh -i id_rsa -l username  External_HOST_IP   -v 
OpenSSH_8.6p1, LibreSSL 2.8.3
debug1: Reading configuration data /Users/username/.ssh/config
debug1: /Users/username/.ssh/config line 2: Applying options for *
debug1: Reading configuration data /etc/ssh/ssh_config
debug1: /etc/ssh/ssh_config line 21: include /etc/ssh/ssh_config.d/* matched no files
debug1: /etc/ssh/ssh_config line 54: Applying options for *
debug1: Authenticator provider $SSH_SK_PROVIDER did not resolve; disabling
debug1: Executing proxy command: exec nc  External_HOST_IP   22
debug1: identity file id_rsa type 0
debug1: identity file id_rsa-cert type -1
debug1: Local version string SSH-2.0-OpenSSH_8.6
debug1: Remote protocol version 2.0, remote software version OpenSSH_7.4
debug1: compat_banner: match: OpenSSH_7.4 pat OpenSSH_7.0*,OpenSSH_7.1*,OpenSSH_7.2*,OpenSSH_7.3*,OpenSSH_7.4*,OpenSSH_7.5*,OpenSSH_7.6*,OpenSSH_7.7* compat 0x04000002
debug1: Authenticating to  External_HOST_IP  :22 as 'username'
debug1: load_hostkeys: fopen /Users/username/.ssh/known_hosts2: No such file or directory
debug1: load_hostkeys: fopen /etc/ssh/ssh_known_hosts: No such file or directory
debug1: load_hostkeys: fopen /etc/ssh/ssh_known_hosts2: No such file or directory
debug1: SSH2_MSG_KEXINIT sent
debug1: SSH2_MSG_KEXINIT received
debug1: kex: algorithm: curve25519-sha256
debug1: kex: host key algorithm: ssh-ed25519
debug1: kex: server->client cipher: chacha20-poly1305@openssh.com MAC: <implicit> compression: none
debug1: kex: client->server cipher: chacha20-poly1305@openssh.com MAC: <implicit> compression: none
debug1: expecting SSH2_MSG_KEX_ECDH_REPLY
debug1: SSH2_MSG_KEX_ECDH_REPLY received
debug1: Server host key: ssh-ed25519 SHA256:Pt+/4PLN4e87/CKUr6u3rfgxnFJ3c9J7I9QFn5jm+9E
debug1: load_hostkeys: fopen /Users/username/.ssh/known_hosts2: No such file or directory
debug1: load_hostkeys: fopen /etc/ssh/ssh_known_hosts: No such file or directory
debug1: load_hostkeys: fopen /etc/ssh/ssh_known_hosts2: No such file or directory
debug1: hostkeys_find_by_key_hostfile: hostkeys file /Users/username/.ssh/known_hosts2 does not exist
debug1: hostkeys_find_by_key_hostfile: hostkeys file /etc/ssh/ssh_known_hosts does not exist
debug1: hostkeys_find_by_key_hostfile: hostkeys file /etc/ssh/ssh_known_hosts2 does not exist
The authenticity of host ' External_HOST_IP   (<no hostip for proxy command>)' can't be established.
ED25519 key fingerprint is SHA256:Pt+/4PLN4e87/CKUr6u3rfgxnFJ3c9J7I9QFn5jm+9E.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added ' External_HOST_IP  ' (ED25519) to the list of known hosts.
debug1: rekey out after 134217728 blocks
debug1: SSH2_MSG_NEWKEYS sent
debug1: expecting SSH2_MSG_NEWKEYS
debug1: SSH2_MSG_NEWKEYS received
debug1: rekey in after 134217728 blocks
debug1: Will attempt key: id_rsa RSA SHA256:v8CIX2xe6JRaIpsJwrnW1StBu3Ab0F4Fzyf2mPZhAjw explicit
debug1: SSH2_MSG_EXT_INFO received
debug1: kex_input_ext_info: server-sig-algs=<rsa-sha2-256,rsa-sha2-512>
debug1: SSH2_MSG_SERVICE_ACCEPT received
debug1: Authentications that can continue: publickey,gssapi-keyex,gssapi-with-mic
debug1: Next authentication method: publickey
debug1: Offering public key: id_rsa RSA SHA256:v8CIX2xe6JRaIpsJwrnW1StBu3Ab0F4Fzyf2mPZhAjw explicit
debug1: Server accepts key: id_rsa RSA SHA256:v8CIX2xe6JRaIpsJwrnW1StBu3Ab0F4Fzyf2mPZhAjw explicit
Enter passphrase for key 'id_rsa': 
debug1: Authentication succeeded (publickey).
Authenticated to  External_HOST_IP   (via proxy).
debug1: channel 0: new [client-session]
debug1: Requesting no-more-sessions@openssh.com
debug1: Entering interactive session.
debug1: pledge: filesystem full
debug1: client_input_global_request: rtype hostkeys-00@openssh.com want_reply 0
debug1: client_input_hostkeys: searching /Users/username/.ssh/known_hosts for  External_HOST_IP   / (none)
debug1: client_input_hostkeys: searching /Users/username/.ssh/known_hosts2 for  External_HOST_IP   / (none)
debug1: client_input_hostkeys: hostkeys file /Users/username/.ssh/known_hosts2 does not exist
debug1: Sending environment.
debug1: channel 0: setting env LANG = "en_AU.UTF-8"
Learned new hostkey: RSA SHA256:+fsdo8GO+muA16NhRc71dBr7zOdkfRW9snYpdUASNeg
Learned new hostkey: ECDSA SHA256:h1wv9vTry84C1WZSkgNwuT2l22bWQcRoxHkq67OphJ0
Adding new key for  External_HOST_IP   to /Users/username/.ssh/known_hosts: ssh-rsa SHA256:+fsdo8GO+muA16NhRc71dBr7zOdkfRW9snYpdUASNeg
Adding new key for  External_HOST_IP   to /Users/username/.ssh/known_hosts: ecdsa-sha2-nistp256 SHA256:h1wv9vTry84C1WZSkgNwuT2l22bWQcRoxHkq67OphJ0
debug1: update_known_hosts: known hosts file /Users/username/.ssh/known_hosts2 does not exist


Last login: Mon Jan  3 04:12:06 2022 from x.y.w.z
[username@test ~]$ uptime
 04:13:02 up  1:34,  2 users,  load average: 0.00, 0.01, 0.04
[username@test ~]$ exitdebug1: client_input_channel_req: channel 0 rtype exit-status reply 0
debug1: client_input_channel_req: channel 0 rtype eow@openssh.com reply 0

logout
debug1: channel 0: free: client-session, nchannels 1
Connection to  External_HOST_IP   closed.
Transferred: sent 4316, received 3896 bytes, in 11.3 seconds
Bytes per second: sent 383.3, received 346.0
debug1: Exit status 0
username@M1pro .ssh % 
```

I could always SSH to local machines on my network just fine, and this proxy does not affect them after the change either.


