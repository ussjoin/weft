# Ansible Configuration

## How to Initialize the Prime Node

* Grab the newest "Lite" image from <https://www.raspberrypi.org/software/operating-systems/#raspberry-pi-os-32-bit>.
* Flash it onto your MicroSD card using the tool of your choice. I like [Balena Etcher](https://www.balena.io/etcher/).
* After the flash is done, remove and re-insert the MicroSD card so your machine mounts it.
* Make a new file `wpa_supplicant.conf` in the `boot` volume of the MicroSD card, edit the contents below, and save them in the file:

```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=US

network={
 ssid="<Name of your wireless LAN>"
 psk="<Password for your wireless LAN>"
}

```

* Make a new file `ssh` in the `boot` volume. Doesn't need any contents.
* Eject the SD card, stick it in the Pi, and boot the Pi and figure out its IP address. How to do that is left as an exercise to the reader.
* On your working machine, install ansible, and edit `/etc/hosts` to specify the hostname `prime.weft` as the IP address you found.
* Use `ssh-keygen` to generate an SSH key on your working machine. I'll assume in the next line the public key is `id_rsa.pub`, but if not, just substitute in your own file.
* `cat ~/.ssh/id_rsa.pub | ssh pi@prime.weft "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >>  ~/.ssh/authorized_keys"` . Credit to <https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-2> for this excellent one-liner which sets up SSH public key authentication on the `prime.weft` all at once for you.
* Use the Ansible remote playbook to do the initial configuration. This will take a while. `ansible-playbook -u pi -i "prime.weft," initialize-prime-remote.yml`
* Once that playbook finishes (including the node reboot), SSH to the prime node and get ready for round two. `ssh pi@prime.weft`
* `cd weft/ansible`
* `ansible-playbook initialize-prime-local.yml`



## Notes for Future Upgrades

As of this writing (Feb 2021), Raspbian is packaging ansible 2.7 series. Accordingly, the OpenSSL module is https://docs.ansible.com/ansible/2.7/modules/openssl_certificate_module.html . As of Ansible 2.9, that module will be moved to its new home at https://docs.ansible.com/ansible/latest/collections/community/crypto/x509_certificate_module.html , and need to be installed separately via `ansible-galaxy collection install community.crypto` . So if you're getting odd errors that look like that's the issue, check what version of ansible is installed, and if this doc hasn't been updated and ansible 2.9+ is shipping in Raspbian, open an issue.

On the same note, currently the `initialize-prime-local.yml` calls `ssh-keygen` via shell. The correct way to do that is https://docs.ansible.com/ansible/latest/collections/community/crypto/openssh_keypair_module.html -- which, again, is in the community.crypto galaxy collection. That should be changed at the same time as the above.

Similarly, the "create directory" calls need to be modulename `ansible.builtin.file`, not just `file`, when that upgrade happens.

`blockinfile` needs to use `path`, not `dest`.