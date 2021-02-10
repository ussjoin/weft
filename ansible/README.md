# Ansible Configuration

## Initialization

### How to Flash the Prime Node

* Grab the newest "Lite" image from <https://www.raspberrypi.org/software/operating-systems/#raspberry-pi-os-32-bit>.
* Flash it onto your MicroSD card using the tool of your choice. I like [Balena Etcher](https://www.balena.io/etcher/).
* After the flash is done, remove and re-insert the MicroSD card so your machine mounts it.
* Make a new file `ssh` in the `boot` volume. Doesn't need any contents. (<https://www.raspberrypi.org/documentation/remote-access/ssh/README.md> for the explanation; see #3.)
* Make a new file `wpa_supplicant.conf` in the `boot` volume of the MicroSD card. (See <https://www.raspberrypi.org/documentation/configuration/wireless/headless.md> for the explanation.) Edit the contents below, and save them in the file:

```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=US

network={
 ssid="<Name of your wireless LAN>"
 psk="<Password for your wireless LAN>"
}

```

* Eject the SD card and stick it in the Pi.

### How to Flash Worker Nodes

Same as the Prime node, except that you don't need to do the `wpa_supplicant.conf` step; you do, however, need to create the `ssh` file.


### How to Initialize the Prime Node

* boot the Pi and figure out its IP address. How to do that is left as an exercise to the reader.
* On your working machine, install Ansible.
* On your working machine, edit `/etc/hosts` to specify the hostname `prime.weft` as the IP address you found.
* Use `ssh-keygen` to generate an SSH key on your working machine. I'll assume in the next line the public key is `id_rsa.pub`, but if not, just substitute in your own file.
* `cat ~/.ssh/id_rsa.pub | ssh pi@prime.weft "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >>  ~/.ssh/authorized_keys"` . Credit to <https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-2> for this excellent one-liner which sets up SSH public key authentication on the `prime.weft` all at once for you. (Ansible doesn't play nicely with anything other than public key auth for SSH.)
* CD into the ansible folder, then use the Ansible remote playbook to do the initial configuration. `ansible-playbook -u pi -i "prime.weft," initialize-prime-remote.yml` This will take a while, and involve two reboots: one near the beginning, and one at the end. (If you upgrade the kernel then try to install docker without a reboot, docker screams bloody murder. See <https://bbs.archlinux.org/viewtopic.php?id=254759> for the explanation.)
* Once that playbook finishes (including the node reboots), SSH to the prime node and get ready for round two. `ssh pi@prime.weft`
* `cd weft/ansible` (The initial ansible playbook checked out this repo for you.)
* `ansible-playbook initialize-prime-local.yml`

### How to Add Worker Nodes

Decide on a worker node naming scheme; it's fine to call them worker1, worker2, etc., or name them on some other basis. (For small clusters, I often name them based on color of Ethernet cables.)

You need to decide on three items for each worker node:

* IP address: it'll be `10.10.220.x` where `x` is a number between 3 and 50. (You can change this in `dnsmasq.conf` if you wish). You just specify the `x` below.
* Hostname: as discussed above.
* MAC address: of the worker node, because this is how DNSMasq will identify it. You want it in lowercase, colon-delimited format, e.g., `aa:bb:cc:dd:ee:ff`.

Then for each worker, run (on the Prime node):

```
ansible-playbook add-worker.yml --extra-vars "fourth_octet=3 mac_address=aa:bb:cc:dd:ee:ff hostname=blue"
ansible-playbook initialize-worker.yml --extra-vars "hostname=blue"
```

The `add-worker` playbook has a 120-second pause; this is because DNSMasq can't give nodes a DHCP lease of less than 120 seconds, so it takes that long for the node to release its old IP address and pick up the new one. After that, you will likely get an SSH prompt to accept a host key and then enter a password; if you're using default Raspbian, the password is `raspberry`. This is the end of the Ansible script, where it is pushing the `prime.weft` SSH public key to the nodes.

The two workbooks are separate because Ansible targets commands by hostname; the first one runs almost exclusively on (and every actual system command does run from) Prime, whereas the second one actually runs on the new worker node. You can run the two in parallel for *separate* systems (i.e., `add-worker` blue, `initialize-worker` green, if green was already added.)


## Notes for Future Upgrades

As of this writing (Feb 2021), Raspbian is packaging ansible 2.7 series. Accordingly, the OpenSSL module is https://docs.ansible.com/ansible/2.7/modules/openssl_certificate_module.html . As of Ansible 2.9, that module will be moved to its new home at https://docs.ansible.com/ansible/latest/collections/community/crypto/x509_certificate_module.html , and need to be installed separately via `ansible-galaxy collection install community.crypto` . So if you're getting odd errors that look like that's the issue, check what version of ansible is installed, and if this doc hasn't been updated and ansible 2.9+ is shipping in Raspbian, open an issue.

On the same note, currently the `initialize-prime-local.yml` calls `ssh-keygen` via shell. The correct way to do that is https://docs.ansible.com/ansible/latest/collections/community/crypto/openssh_keypair_module.html -- which, again, is in the community.crypto galaxy collection. That should be changed at the same time as the above.

Similarly, the "create directory" calls need to be modulename `ansible.builtin.file`, not just `file`, when that upgrade happens.

`blockinfile` needs to use `path`, not `dest`.
