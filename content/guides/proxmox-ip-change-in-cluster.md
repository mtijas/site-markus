---
Title: Changing the IP address of a Proxmox node
Description: Changing the IP address of a Proxmox node already in a cluster
Template: withsubmenu
---

# Changing the IP address of a Proxmox node already in a cluster

In this guide I'll assume basic proficiency in working in a shell as well as having static IP's set to each of the machines. You will probably break something on the first (and second) try so it is strongly suggested to test this in a local testing environment first.

1. I'll strongly suggest backing up the old configuration files `/etc/hosts`, `/etc/network/interfaces` and `/etc/pve/corosync.conf` in case something goes wrong.
2. Update your firewall rules (in Datacenter -> Firewall) to allow traffic from the new IP address.
3. Check that the rules are propagated through the cluster successfully by opening a shell and listing the rules with `iptables -L` on at least the node whose IP address you are going to change (just to make sure you will have access after the change). If you are using the IPSet functionality the addresses can be listed with shell command `ipset -L`.
4. On the target node change the IP address in `/etc/hosts` file. You might want to consider changing the IP address in every node's `/etc/hosts` file if you have added it there previously and are not using a DNS server to do the name resolution.
5. On the target node change the IP address in `/etc/network/interfaces` file (the actual network config).
6. Change the IP address in `/etc/pve/corosync.conf` file on one of the nodes to match the new one and increase the `config_version` value by one in that file too so the configuration gets read by Proxmox.
7. Reboot the target node.
8. Once you are connected to the shell of the target node (with browser to the new IP address) remember to update certificates to fix connecting to the node through other node's web interface. Updating can be done with command `pvecm updatecerts --force`.
9. Also remember to update DNS records if needed.

When everything is in working order feel free to repeat the process to all the other nodes you might want to have new IP addresses.

For more information on the topic I'd suggest to visit [the Proxmox wiki](https://pve.proxmox.com/wiki/Separate_Cluster_Network).

Last updated: 17.07.2021