# Bond-LVS-Apache2
Agrégation de lien, avec répartition de charge du service Apache2

#### Agrégation de lien

- Installer le paquet ifsenslave
```bash
apt-get install ifenslave
```
- Configuration du Bond0 (Adapter le nom de la carte réseau)
```bash
nano /etc/modprobe.d/alias-bond.conf
alias bond0 bonding
options bonding mode=1 primary=eth0 fail_over_mac=1
```
Configuration de la carte réseau bond0 (Adapter le nom des cartes réseau)
```bash
nano /etc/network/interfaces
auto bond0
iface bond0 inet static
bond-mode 1
bond-slaves eth0 eth1
address 192.168.20.1
netmask 255.255.255.0
```
- Déchargement du module
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

#### Répartition de charge LVS/Apache2
- Installation des paquets
```bash
apt install iptables
apt install ipvsadm
```
- Créer un service virtuel
```bash
ipvsadm -A -t 192.168.XX.XXX:80 -s rr
ipvsadm -a -t 192.168.XX.XXX:80  -r 192.168.YY.YYY -m
ipvsadm -a -t 192.168.XX.XXX:80 -r 192.168.ZZ.ZZZ -m
ipvsadm -L
```
- Vérification
```bash
apt install curl
curl 192.168.XX.XXX
```
#### Stockage RAID 1
