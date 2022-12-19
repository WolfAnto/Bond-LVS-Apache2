# Bond-LVS-Apache2
Agrégation de lien, avec répartition de charge du service Apache2

#### Agrégation de lien

- Installer le paquet ifsenslave
```bash
apt-get install ifenslave
```
- Configuration du Bond0
```bash
nano /etc/modprobe.d/alias-bond.conf
alias bond0 bonding
options bonding mode=1 primary=eth0 fail_over_mac=1
```
Configuration de la carte réseau bond0
```bash
nano /etc/network/interfaces
auto bond0
iface bond0 inet static
bond-mode 1
bond-slaves eth0 eth1
address 192.168.20.1
netmask 255.255.255.0
```
```bash
rmmod bonding
```
- Rédémarrage du service
```bash
/etc/init.d/networking restart
```
- Vérification
```bash
cat /proc/net/bonding/bond0
ip -br ad
```
