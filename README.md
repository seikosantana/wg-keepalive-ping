# Wireguard KeepAlive PING Service
A simple __systemd__ service to PING a host to keep _connection_ alive.
  

#### Background
I've been tinkering around with my Wireguard installation on multiple hosts, on the cloud,  office and home servers, to keep them connected and accessible anywhere.

##### Conditions
- The cloud server is _not_ behind a NAT.
- All clients _is_ behind a NAT.
- All Wireguard interfaces is configured to be `wg0`
- All Wireguard interfaces is configured to __only tunnel non-local traffic__, allowing access to LAN and WAN.

##### Problems
Clients (or, peer in Wireguard context, but not the cloud that acts as the main Wireguard peer) are not able to _reconnect automatically_ when
- The Wireguard interface on the cloud gets restarted.
- The Wireguard interface on the _clients_ gets restarted.
- The clients lost connection to internet.
- The clients (apparently because being behind a NAT) reconnects after some unknown period of time, although KeepAlive is configured to be 25 seconds.

#### Investigation so far
According to my inspections so far after tinkering with these for 3 weeks, clients are never reachable unless it reaches first to the server which happens if:
- A traffic from the client is routed to the server which doesn't seem to happen after the kill-switch (to allow only VPN connections through the tunnel) is turned of by changing AllowedIPs.
- A ping to reach any host in the AllowedIPs is performed.

That should explain the workaround the service does.

##### Is this service required for Wireguard installations?
I don't think so. Wireguard peers seems to reconnect after some period of time if everything is tunneled through the VPN, which is _probably_ because network activities makes the host to reach the VPN peers.  
This might not be necessary unless you are facing disconnections like in my case.

I'm no expert but discussions are welcome. May this simple service help you.

## How to use?
Download the `wg-keepalive-ping.service` and install it into the systemd
```bash
wget https://raw.githubusercontent.com/seikosantana/wg-keepalive-ping/master/wg-keepalive-ping.service
sudo cp wg-keepalive-ping.service /etc/systemd/system/
```
Set the IP and the ping interval with
```bash
$ sudo systemctl edit wg-keepalive-ping
```
_Required_:
Add the environment variables of `IP` and `INTERVAL` as in the [example](https://github.com/seikosantana/wg-keepalive-ping/blob/master/sample-env-override.conf).
Put them in the file `systemctl` created for you.
```
[Service]
Environment="IP=10.66.66.1" #Any IP to ping, in this case the Wireguard peer
Environment="INTERVAL=15" #The ping interval.
```
Save the file and close the text editor.
Reload the daemon and restart the service.
```bash
$ sudo systemctl daemon-reload
$ sudo systemctl enable --now wg-keepalive-ping.service
```

To edit the variables again, use `systemctl edit wg-keepalive-ping` and run `systemctl daemon-reload`.


Any questions and suggestions are welcome :)