# Provisioning the Server
now that we have a fresh installation of *Raspbian Stretch-lite* which we can ssh into, it is time to provision the pi. "but what could you possibly mean by provisioning?", you ask. well by provisioning we mean,
* **[changing `pi` user passwd](#change-pi-usr-passwd)**
* **[adding user accounts](#add-usr-accounts)**
* **[copying ssh public keys](#copy-ssh-public-keys)**
* **[changing the hostname](#change-hostname)**
* **[disabling passwd login](#disable-passwd-login)**
* **[adding a firewall](#add-a-firewall)**

## Change Pi Usr Passwd

## Add Usr Accounts

## Copy SSH Public Keys

## Change Hostname

## Disable Passwd Login
since it is way more secure to login using ssh public keys, we want to disable password login and only allow users to login who have their public keys on the server! **admin beware**, if you haven't put any ssh keys onto the server yet you could lock yourself out of the box in this step! you **must** be able to login with ssh keys before proceeding with this step!


  * `sudo emacs /etc/ssh/sshd_config`
  * set:
    ```
    PasswordAuthentication no
    ```
  * check:
    ```
    PubkeyAuthentication yes
    ChallengeResponseAuthentication no
    ```

## Add a Firewall
#### verify we are only listening on port 22
if we run `ss` (another utility to investigate sockets) on all processes listening over tcp,

```
$ ss -ltnp # --listen --tcp --numeric --processes
State       Recv-Q Send-Q              Local Address:Port                             Peer Address:Port
LISTEN      0      128                             *:22                                          *:*
LISTEN      0      128                            :::22                                         :::*
```
great, we are only listening on port 22 (ssh)

#### install `ufw`
we want to install *Uncomplicated Firewall*, or `ufw`,

```
$ sudo apt-get install ufw
```

#### default deny incoming and allow outgoing

```
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing
```

#### allow ssh, dhcp, dns, mdns, www, tls, (optionally tor)

```
$ sudo ufw allow in ssh  # 22/tcp for ssh
$ sudo ufw allow out ssh
$ sudo ufw allow in 68/udp # port for dhcp client
$ sudo ufw allow out 68/udp
$ sudo ufw allow in dns  # 53/tcp for dns
$ sudo ufw allow out dns
$ sudo ufw allow in mdns  # 5353/udp for multicast dns
$ sudo ufw allow out mdns
$ sudo ufw allow in www  # 80/tcp for standard http traffic
$ sudo ufw allow out www
$ sudo ufw allow in 443/tcp  # for serving tls certs
$ sudo ufw allow out 443/tcp

$ sudo ufw allow in 9050/tcp  # for tor proxy port (all traffic through the tor network starts here)
$ sudo ufw allow out 9050/tcp
$ sudo ufw allow in 9051/tcp  # for tor control port (any tor controlling application talks to tor through this listening port (note, it is unclear whether this ports needs to be exposed though))
$ sudo ufw allow out 9051/tcp
```

## Resources
* [UFW](https://www.digitalocean.com/community/tutorials/how-to-setup-a-firewall-with-ufw-on-an-ubuntu-and-debian-cloud-server)
* [SSH Setup](https://share.riseup.net/#YZBKJJesxwrTfCB1Bb3atQ)
* [More Debian Setup](https://share.riseup.net/#A8gDSxMgdIZMCkXUeQeC0A)
