# okd4_files

#### Topology

Base domain: mycluster.demo

|   | Machine  | IP  |
|---|---|---|
| 1 | services | 192.168.1.210 |
| 2 | bootstrap | 192.168.1.200 |
| 3 | control-plane-1 | 192.168.1.201 |
| 4 | control-plane-2 | 192.168.1.202 |
| 5 | control-plane-3 | 192.168.1.203 |
| 6 | ccompute-1 | 192.168.1.204 |
| 7 | ccompute-2 | 192.168.1.205 |
| 8 | ccompute-3 | 192.168.1.206 |

### Services Build CentOS 8

#### Post-install

```
visudo
ALL            ALL = (ALL) NOPASSWD: ALL

sudo dnf install -y epel-release
sudo dnf update -y
sudo dnf -y install curl git bind bind-utils haproxy httpd nfs-utils 
cat <<EOF >> ~/.bashrc
PS1="\u@\h \w\$ "
HISTTIMEFORMAT="%m/%d/%y %T "
EOF
Gnome tweaks
Gnome extentions: dash to panel
```
#### JQ and Helm
```
wget -O jq https://github.com/stedolan/jq/releases/download/jq-1.6/jq-linux64
chmod +x jq
sudo mv jq /usr/local/bin/
jq --version

curl -L https://mirror.openshift.com/pub/openshift-v4/clients/helm/latest/helm-linux-amd64 -o /usr/local/bin/helm
sudo chmod +x /usr/local/bin/helm
helm version -c
```
#### Git clone
```
git clone https://github.com/aroute/okd4_files.git
cd okd4_files
git checkout test
```
#### DNS
```
sudo cp named.conf /etc/named.conf
sudo cp named.conf.local /etc/named/
sudo mkdir /etc/named/zones
sudo cp db* /etc/named/zones
sudo systemctl enable named
sudo systemctl start named
sudo firewall-cmd --permanent --add-port=53/udp
sudo firewall-cmd --reload
```
Reboot / configure skytap DNS for 192.168.1.210

#### Verify DNS
```
sudo systemctl status named
dig mycluster.demo
dig –x 192.168.1.210
```
#### NFS
```
sudo systemctl enable nfs-server rpcbind
sudo systemctl start nfs-server rpcbind
sudo mkdir -p /var/nfsshare/registry
sudo chmod -R 777 /var/nfsshare
sudo chown -R nobody:nobody /var/nfsshare
echo '/var/nfsshare 192.168.1.0/24(rw,sync,no_root_squash,no_all_squash,no_wdelay)' | sudo tee /etc/exports
sudo setsebool -P nfs_export_all_rw 1
sudo systemctl restart nfs-server
sudo firewall-cmd --permanent --zone=public --add-service mountd
sudo firewall-cmd --permanent --zone=public --add-service rpc-bind
sudo firewall-cmd --permanent --zone=public --add-service nfs
sudo firewall-cmd --reload
```
#### HA Proxy
```
cd okd4_files
sudo cp haproxy.cfg /etc/haproxy/haproxy.cfg
sudo setsebool -P haproxy_connect_any 1
sudo systemctl enable haproxy
sudo systemctl start haproxy
sudo firewall-cmd --permanent --add-port=6443/tcp
sudo firewall-cmd --permanent --add-port=22623/tcp
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
sudo sed -i 's/Listen 80/Listen 8080/' /etc/httpd/conf/httpd.conf
sudo setsebool -P httpd_read_user_content 1
sudo systemctl enable httpd
sudo systemctl start httpd
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload
```
#### Verify
```
sudo systemctl status haproxy
curl localhost:8080
```
#### OpenShift Installer
```
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/openshift-install-linux.tar.gz
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/openshift-client-linux.tar.gz
tar -zxvf 
tar -zxvf 
sudo mv kubectl oc openshift-install /usr/local/bin/
rm *.tar.gz
```
#### OKD Installer
```
wget https://github.com/openshift/okd/releases/download/4.5.0-0.okd-2020-06-29-110348-beta6/openshift-client-linux-4.5.0-0.okd-2020-06-29-110348-beta6.tar.gz
wget https://github.com/openshift/okd/releases/download/4.5.0-0.okd-2020-06-29-110348-beta6/openshift-install-linux-4.5.0-0.okd-2020-06-29-110348-beta6.tar.gz
tar -zxvf openshift-client-linux-4.5.0-0.okd-2020-06-29-110348-beta6.tar.gz
tar -zxvf openshift-install-linux-4.5.0-0.okd-2020-06-29-110348-beta6.tar.gz
sudo mv kubectl oc openshift-install /usr/local/bin/
rm *.tar.gz
```
#### Verify
```
oc version
openshift-install version
```
#### SSH Keygen
```
ssh-keygen
```
#### Installation directory
```
cd
mkdir install_dir
cp okd4_files/install-config.yaml ./install_dir
```

Copy id and pull key.

```
openshift-install create manifests --dir=install_dir/
sed -i 's/mastersSchedulable: true/mastersSchedulable: False/' install_dir/manifests/cluster-scheduler-02-config.yml
openshift-install create ignition-configs --dir=install_dir/
```
#### Move files to the web server
```
sudo mkdir /var/www/html/okd4
sudo cp -R install_dir/* /var/www/html/okd4/
sudo chown -R apache: /var/www/html/
sudo chmod -R 755 /var/www/html/
```
#### Verify
```
curl localhost:8080/okd4/metadata.json
```
#### CoreOS
```
cd /var/www/html/okd4/
sudo wget https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/latest/latest/rhcos-4.4.3-x86_64-metal.x86_64.raw.gz
sudo wget https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/latest/latest/rhcos-4.4.3-x86_64-metal.x86_64.raw.gz.sig
sudo mv ... cos.raw.xz
sudo mv ... cos.raw.xz.sig
sudo chown -R apache: /var/www/html/
sudo chmod -R 755 /var/www/html/
cd ~/
ls -l /var/www/html/okd4/
```
#### Fedora CoreOS
```
cd /var/www/html/okd4/
sudo wget https://builds.coreos.fedoraproject.org/prod/streams/stable/builds/32.20200615.3.0/x86_64/fedora-coreos-32.20200615.3.0-metal.x86_64.raw.xz
sudo wget https://builds.coreos.fedoraproject.org/prod/streams/stable/builds/32.20200615.3.0/x86_64/fedora-coreos-32.20200615.3.0-metal.x86_64.raw.xz.sig
sudo mv fedora-coreos-32.20200615.3.0-metal.x86_64.raw.xz fcos.raw.xz
sudo mv fedora-coreos-32.20200615.3.0-metal.x86_64.raw.xz.sig fcos.raw.xz.sig
sudo chown -R apache: /var/www/html/
sudo chmod -R 755 /var/www/html/
cd ~/
ls -l /var/www/html/okd4/
```
#### Download CoreOS ISO for Skytap Asset
```
cd ~/Downloads
wget https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/latest/latest/rhcos-4.4.3-x86_64-installer.x86_64.iso
```

#### Boot install
```
coreos.inst=yes
coreos.inst.install_dev=sda 
coreos.inst.image_url=http://192.168.1.210:8080/okd4/cos.raw.xz 
coreos.inst.ignition_url=http://192.168.1.210:8080/okd4/bootstrap.ign

coreos.inst=yes
coreos.inst.install_dev=sda 
coreos.inst.image_url=http://192.168.1.210:8080/okd4/cos.raw.xz 
coreos.inst.ignition_url=http://192.168.1.210:8080/okd4/master.ign

coreos.inst=yes
coreos.inst.install_dev=sda 
coreos.inst.image_url=http://192.168.1.210:8080/okd4/cos.raw.xz 
coreos.inst.ignition_url=http://192.168.1.210:8080/okd4/worker.ign
```