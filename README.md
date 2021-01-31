# Pi-hole TVbox bypass

Pi-hole offers network-level advertisement and Internet tracker protection by acting as a DNS sinkhole. You just change your DHCP server config (normally included in your ISP router) to used the Pi-hole as the resolver, which then filters requests acording to blacklists or forwards them to other open DNS providers.

## The problem
Some equipment requires specific DNS servers to work, one example are tv set top boxes from ISPs that access internal IPTV stream servers. This could be easily solved if these equipments were easy to configure - just set the DNS server to the required one - but they normally are not.

## The solution
One possible solution is to disable the DHCP server from the ISP router (with normally sucks too) and use Pi-hole as a DHCP server too. Then configure it to serve different settings for specific equipment.

To this end ssh to pi-hole and add a new config file under /etc/dnsmasq.q/, e.g., `sudo nano /etc/dnsmasq.q/vodafone-iptv-boxes.conf`:
```
# TAG IPTV boxes bated on their MAC address
dhcp-host=08:80:39:XX:XX:XX,set:tvbox

dhcp-range=set:tvbox,192.168.1.46,192.168.1.50,infinite
dhcp-option=tag:tvbox,option:dns-server,192.168.1.1
```
In brief you are tagging a specific mac with label _tvbox_. Next these hosts will have distinct settings such as a specific range (46 to 50) and use the router/gateway dns server (the ISP one, in order to work in my specific case).


If you happen to have more than one Pi-hole running you can add two DNS servers to the DHCP server using:
```
# X and Y are the primary and secondary DNS servers (pi-hole instances)
dhcp-option=6,192.168.1.X,192.168.1.Y
```
Please note that this is may not be standard and normally the secondary is not a backup but can be queried too.

Finally just activate the DHCP server under the Pi-hole admin interface and it should work.

### Pi-hole docker image (and Synology)
If you use docker to run Pi-hole you might need to use `--cap-add=NET_ADMIN` and probably run it with host networking mode (`--net=host`) so it can see network broadcasts (or configure a dhcp relay) In this case you might want to change de web interface port (-e WEB_PORT). To achieve this under Synology DMS you might need to export your container config (to .json), edit the file and import, since you cannot change the run command under the GUI:
```
*) export json settings of a standard (won’t restart) pihole 4.1 container
*) edit json file to include “cap_add” : [ “NET_ADMIN” ],
*) import edited json file to create new container with high privileges
```

## Sources
* https://serverfault.com/questions/509388/per-client-dns-servers-with-dnsmasq
* https://discourse.pi-hole.net/t/secondary-dns-server-for-dhcp/1874
* https://docs.pi-hole.net/docker/DHCP/
* https://github.com/pi-hole/docker-pi-hole/issues/404
