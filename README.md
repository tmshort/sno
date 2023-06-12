# sno

This is a set of script to set up SNO (Single Node Openshift) in a VM.

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
sudo dnf install qemu-kvm virt-manager libvirt
sudo systemctl enable --now libvirtd
sudo systemctl start libvirtd</dnf
sudo usermod -aG kvm $USER
sudo usermod -aG libvirt $USER
```

Files:

* `config.yaml` - this contains parameters for the install. You need to include a public SSH key and a `pull-secret.txt` file that you can get from Red Hat [console](https://console.redhat.com/openshift/install/pull-secret).
* `setup-sno` - this script downloads an install ISO and builds a custom ISO. It also creates a file that can be used with `oc` as KUBECONFIG.
* `setup-vm` - this script, which need to be run as *root* (via `sudo` or equivalent) and called  actually creates the the VM and starts it. Right now it opens up the console; I want it to start unattended. It also updates `/etc/hosts`.
* `delete-vm` - this script delete the VM (aka _domain_ and the disk volume)

The intent is for `setup-vm` to be started with the ISO.
```
emacs config.yaml
./setup-sno
sudo ./setup-vm work/rhcos-live.x86_64.iso
```

At this point, the VM will eventually shut itself down, and it will need to be restarted:
```
sudo virsh start SNO
```

You can connect to the console via:
```
sudo virsh --connect qemu:///system console SNO
```
But you won't get much out of it.

Note that `/etc/hosts` is modified to allow the install to occur, and so that you can access the base OS:
```
ssh -o StrictHostKeyChecking=no core@sno
```
The `journalctl` command will give a lot more information about installation status.

Eventually, you'll be able to run `oc` or `kubectl`
```
export KUBECONFIG=$(pwd)/work/ocp/auth/kubeconfig
oc get clusterversion
```

As of right now, I have the status of "the cluster operator ingress is degraded". Something to figure out.
