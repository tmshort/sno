# SNO

This is a set of scripts to set up SNO (Single Node Openshift) in a VM.

## Requirements

It makes _LOTS_ of assumptions. I have only tested it on my Fedora 37 machine.

You will need:
* `oc`
* `podman`
* `curl`
* `jq` and `yq`
* `qemu-kvm`
* `virt-manager`
* `libvirt`
* `virt-install`
* `virsh`

This also assumes that you've done all the proper startup and config for libvirtd:
```
sudo dnf install qemu virt-manager virt-install libvirt
sudo systemctl enable --now libvirtd
sudo systemctl start libvirtd
sudo usermod -aG kvm $USER
sudo usermod -aG libvirt $USER
```
(There might be more required.)

## Files

* `config.yaml` - this contains parameters for the install. You need to include a public SSH key and a `pull-secret.txt` file that you can get from Red Hat [console](https://console.redhat.com/openshift/install/pull-secret).
* `setup-sno` - this script downloads an install ISO and builds a custom ISO. It also creates a file that can be used with `oc` as KUBECONFIG.
* `setup-vm` - this script actually creates the the VM and starts it. It also updates `/etc/hosts`, and needs to run `sudo` to do that.
* `delete-vm` - this script delete the VM (aka _domain_ and the disk volume). It will also update `/etc/hosts`, and needs to run `sudo` to do that.

## How To Use

The intent is for `setup-vm` to be started with the ISO.
```
emacs config.yaml
./setup-sno
./setup-vm work/sno.x86_64.iso
```
Ths ISO file is optional (if the default temp directory is used).


The VM will eventually restart itself, and then continue to install SNO.

You can connect to the console via:
```
virsh --connect qemu:///system console sno
```
But you won't get much out of it. You can run other `virsh` commands; you might need to set the following environment variable:
```
export LIBVIRT_DEFAULT_URI=qemu:///system
```

Note that `/etc/hosts` is modified to allow the install to occur, and so that you can access the base OS:
```
ssh -o StrictHostKeyChecking=no core@sno
```
(If you are creating and deleting many VMs, the host keys change, but the IP/DNS won't, the `StrictHostKeyChecking` option avoids this.)

The `journalctl` command will give a lot more information about installation status.
```
journalctl -b -f -u release-image.service -u bootkube.service
journalctl -f
```

Eventually, you'll be able to run `oc` or `kubectl`
```
export KUBECONFIG=$(pwd)/work/ocp/auth/kubeconfig
oc get clusterversion
```

## Notes

* The generated ISO has certificates that are only good for a few days. You should run `setup-sno` again in order to refresh the ISO certificates.
* The temporary download directory is named `work`, and will be cleaned up with `git clean -fdx`.
