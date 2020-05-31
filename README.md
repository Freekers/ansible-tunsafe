# Ansible-TunSafe
Ansible Playbook to setup a dual stack* (IPv4/IPv6) TunSafe VPN with obfuscation.   

TunSafe is a fast and modern VPN based on the WireGuard protocol. It is a point-to-point VPN, which means it does not have a client-server architecture, but peers, and does not rely on a PKI, unlike OpenVPN.   

Please note that TunSafe is not affiliated with the WireGuard project. No support from the WireGuard project will be available and no statements about security from the official WireGuard project apply to tunsafe, nor will it receive (security) updates from WireGuard's upstream. [TunSafe is developed by Ludvig "Ludde" Strigeus, best known for developing software such as the BitTorrent client μTorrent, OpenTTD, and Spotify.](https://en.wikipedia.org/wiki/Ludvig_Strigeus)

*The playbook will try to detect if IPv6 connectivity is available and if so enable it for use with TunSafe. Peers/clients will connect via IPv4.

## Why TunSafe?
The WireGuard protocol is encrypted but not obfuscated. If you're behind a (corporate) firewall that does Deep Packet Inspection (DPI), the WireGuard protocol is very easy to detect and block.  
The Wireguard Project states that ['obfuscation should happen at a layer above WireGuard'](https://www.wireguard.com/known-limitations/) but does not offer any examples/solutions to do so.

TunSafe has incorporated obfuscation in it's latest Release Candidate.
[The default setting is to just make everything look totally random. It can also be set to `tls-chrome` or `tls-firefox` to make the traffic look like HTTPS traffic.](https://tunsafe.com/downloads/ChangeLog.txt).  In this particular playbook, I've chosen to use `tls-firefox`, but you change it if you like.

[Additionally, TunSafe supports two-factor authentication out of the box.](https://tunsafe.com/user-guide/2fa)

## Installation Instructions
1. Install Ansible using `sudo apt install ansible` on the machine that will initiate the playbook.
2. Clone repository using `git clone https://github.com/Freekers/ansible-tunsafe.git`
3. Edit `hosts` to reflect your setup, i.e. change IP, ports etc. `playbook.yml` does NOT need to be changed!
4. Start playbook using `ansible-playbook playbook.yml --ask-become-pass`   

Supported distros:
- Ubuntu 18.04
- Ubuntu 20.04
- Debian 9
- Debian 10

Supported virtualization architectures:
- OpenVZ
- KVM
- Xen
- LXC

## Usage instructions

After installation, copy the TCP & UDP configs to your client. They are located in the `home` directory of the `ansible_user` as indicated in the post install message.

Note that while the syntax between WireGuard and TunSafe is the same, TunSafe and WireGuard clients are not interchangeable because of the obfuscation used in TunSafe. Hence you _need_ to download TunSafe 1.5 RC-2 or later in order to connect to your TunSafe instance.

- For Windows 7 or later: https://tunsafe.com/download
- For Mac OS X, Linux or FreeBSD: https://tunsafe.com/download
- For Android: https://tunsafe.com/user-guide/android  
Note: the version in the Play Store does not support obfuscation. You need to download the experimental APK from the link listed above.
- For iOS: Not available. The version in the Appstore does not support obfuscation.

Server commands:
- Start TunSafe: `sudo service tunsafe start`
- Stop TunSafe: `sudo service tunsafe stop`
- Show Status: `sudo tunsafe show`

## Adding extra peers/clients
On the server:
- Stop TunSafe service
- Edit /opt/tunsafe/TunSafe.conf
```
[Peer]
PublicKey = <client 2 public key>
PresharedKey = <client 2 preshared key - OPTIONAL>
AllowedIPs = 10.66.66.3/24,fd42:42:42::3/120
```
- Start TunSafe service

On the client
- Create TCP Config as follows:
```
[Interface]
Address = 10.100.100.3/24,fd42:42:42::3/120
ObfuscateKey = 1337
PrivateKey = <client 2 private key>
DNS = 1.1.1.1,2606:4700:4700::1001
ObfuscateTCP = tls-firefox

[Peer]
AllowedIPs = 0.0.0.0/0,::/0
PersistentKeepalive = 25
AllowMulticast = false
PublicKey = <server public key>
PresharedKey = <client 2 preshared key - OPTIONAL>
Endpoint = tcp://<server public IP>:<server TCP port>
```
- Create UDP Config as follows:
```
[Interface]
Address = 10.100.100.3/24,fd42:42:42::3/120
ObfuscateKey = 1337
PrivateKey = <client 2 private key>
DNS = 1.1.1.1,2606:4700:4700::1001

[Peer]
AllowedIPs = 0.0.0.0/0,::/0
PersistentKeepalive = 25
AllowMulticast = false
PublicKey = <server public key>
PresharedKey = <client 2 preshared key - OPTIONAL>
Endpoint = <server public IP>:<server UDP port>
```

## License
Unless otherwise specified, all code in this repository is released under the GNU Affero General Public License v3.0. See the [repository's `LICENSE` file](https://github.com/Freekers/ansible-tunsafe/blob/master/LICENSE) for details.