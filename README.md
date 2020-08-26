# OKD 4 Automated Cluster Installation (on KVM) Script

Original Work: https://github.com/kxr/ocp4_setup_upi_kvm. 

We have modified shell script to work with OKD.

### Prerequistes:

- Internet connected physical host running a modern linux distribution
- Virtualization enabled and Libvirt/KVM setup
- DNS on the host managed by dnsmasq or NetworkManager/dnsmasq

## Installing OKD 4 Cluster


| Option  |Description   |
| :------------ | :------------ |
| -O, --okd-version VERSION | You can set this to a specific version like 4.5.0-0.okd-2020-08-12-020541 etc. More info on https://github.com/OKD/okd/releases.<br>Default: stable |
| -R, --rhcos-version VERSION | You can set a specific FCOS version to use. For example "32.20200809.3.0" etc. More info on https://getfedora.org/coreos/download?tab=metal_virtualized&stream=stable.<br>Default: 32.20200809.3.0  |
| -p, --pull-secret FILE | Location of the pull secret file<br>Default: /root/pull-secret |
| -c, --cluster-name NAME | OKD 4 cluster name<br>Default: okd4 |
| -d, --cluster-domain DOMAIN | OKD 4 cluster domain<br>Default: local |
| -m, --masters N | Number of masters to deploy<br>Default: 3 |
| -w, --worker N | Number of workers to deploy<br>Default: 2 |
| --master-cpu N | Number of CPUs for the master VM(s)<br>Default: 4 |
| --master-mem SIZE(MB) | RAM size (MB) of master VM(s)<br>Default: 16000 |
| --worker-cpu N | Number of CPUs for the worker VM(s)<br>Default: 4 |
| --worker-mem SIZE(MB) | RAM size (MB) of worker VM(s)<br>Default: 8000 |
| --bootstrap-cpu N | Number of CPUs for the bootstrap VM<br>Default: 4 |
| --bootstrap-mem SIZE(MB) | RAM size (MB) of bootstrap VM<br>Default: 16000 |
| --lb-cpu N | Number of CPUs for the load balancer VM<br>Default: 1 |
| --lb-mem SIZE(MB) | RAM size (MB) of load balancer VM<br>Default: 1024 |
| -n, --libvirt-network NETWORK | The libvirt network to use. Select this option if you want to use an existing libvirt network<br>The libvirt network should already exist. If you want the script to create a separate network for this installation see: -N, --libvirt-oct<br>Default: default |
| -N, --libvirt-oct OCTET | You can specify a 192.168.{OCTET}.0 subnet octet and this script will create a new libvirt network for the cluster<br>The network will be named okd-{OCTET}. If the libvirt network okd-{OCTET} already exists, it will be used.<br>Default: [not set] |
| -v, --vm-dir | The location where you want to store the VM Disks<br>Default: /var/lib/libvirt/images |
| -z, --dns-dir DIR | We expect the DNS on the host to be managed by dnsmasq. You can use NetworkMananger's built-in dnsmasq or use a separate dnsmasq running on the host. If you are running a separate dnsmasq on the host, set this to "/etc/dnsmasq.d"<br>Default: /etc/NetworkManager/dnsmasq.d |
| -s, --setup-dir DIR | The location where we the script keeps all the files related to the installation<br>Default: /root/okd4_setup_{CLUSTER_NAME} |
| -x, --cache-dir DIR | To avoid un-necessary downloads we download the OKD/RHCOS files to a cache directory and reuse the files if they exist<br>This way you only download a file once and reuse them for future installs<br>You can force the script to download a fresh copy by using -X, --fresh-download<br>Default: /root/okd4_downloads |
| -X, --fresh-download | Set this if you want to force the script to download a fresh copy of the files instead of reusing the existing ones in cache dir<br>Default: [not set] |
| -k, --keep-bootstrap | Set this if you want to keep the bootstrap VM. By default bootstrap VM is removed once the bootstraping is finished<br>Default: [not set] |
| --autostart-vms | Set this if you want to the cluster VMs to be set to auto-start on reboot<br> Default: [not set] |
| -y, --yes | Set this for the script to be non-interactive and continue with out asking for confirmation<br>Default: [not set] |
| --destroy | Set this if you want the script to destroy everything it has created<br>Use this option with the same options you used to install the cluster<br>Be carefull this deletes the VMs, DNS entries and the libvirt network (if created by the script)<br>Default: [not set] |


### Examples
    # Deploy OKD 4.5.0-0.okd-2020-08-12-020541 cluster
    ./okd4_setup_kvm.sh --okd-version 4.5.0-0.okd-2020-08-12-020541
    ./okd4_setup_kvm.sh -O 4.5.0-0.okd-2020-08-12-020541

    # Deploy OKD 4.5.0-0.okd-2020-08-12-020541 cluster with RHCOS 32.20200809.3.0
    ./okd4_setup_kvm.sh --okd-version 4.5.0-0.okd-2020-08-12-020541 --rhcos-version 32.20200809.3.0
    ./okd4_setup_kvm.sh -O 4.5.0-0.okd-2020-08-12-020541 -R 32.20200809.3.0

    # Deploy 4.5.0-0.okd-2020-08-12-020541 OKD version with pull secret from a custom location
    ./okd4_setup_kvm.sh --pull-secret ~/pull-secret --okd-version latest
    ./okd4_setup_kvm.sh -p ~/pull-secret -O 4.5.0-0.okd-2020-08-12-020541

    # Deploy OKD 4.5.0-0.okd-2020-08-12-020541 with custom cluster name and domain
    ./okd4_setup_kvm.sh --cluster-name okd45 --cluster-domain lab.test.com --okd-version 4.5.0-0.okd-2020-08-12-020541
    ./okd4_setup_kvm.sh -c okd45 -d lab.test.com -O 4.5.0-0.okd-2020-08-12-020541

    # Deploy OKD 4.5.0-0.okd-2020-08-12-020541 on new libvirt network (192.168.155.0/24)
    ./okd4_setup_kvm.sh --okd-version 4.5.0-0.okd-2020-08-12-020541 --libvirt-oct 155
    ./okd4_setup_kvm.sh -O 4.5.0-0.okd-2020-08-12-020541 -N 155

    # Destory the already installed cluster
    ./okd4_setup_kvm.sh --cluster-name okd45 --cluster-domain lab.test.com --destroy
    ./okd4_setup_kvm.sh -c okd45 -d lab.test.com --destroy
