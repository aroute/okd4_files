# okd4_files

### Services

#### Build CentOS 8

#### Post-install

```
visudo
ALL            ALL = (ALL) NOPASSWD: ALL
sudo dnf install -y epel-release
sudo dnf update -y
sudo dnf -y install curl git bind bind-utils haproxy httpd nfs-utils 
```

```
wget -O jq https://github.com/stedolan/jq/releases/download/jq-1.6/jq-linux64
chmod +x jq
sudo mv jq /usr/local/bin/
jq --version

curl -L https://mirror.openshift.com/pub/openshift-v4/clients/helm/latest/helm-linux-amd64 -o /usr/local/bin/helm
sudo chmod +x /usr/local/bin/helm
helm version -c
```

```
git clone https://github.com/aroute/okd4_files.git
cd okd4_files
```