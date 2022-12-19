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
- Installation des paquets
```bash
apt update
apt install mdadm lvm2
```
- Création du RAID 1
```bash
mdadm --create /dev/md127 --level=1 --raid-devices=2 /dev/sdb /dev/sdc
mdadm --detail /dev/md127
mdadm --examine --scan --verbose
```
- Rebooter et verifier avec la commande 
```bash
cat /proc/mdstat 
```
- Créer un Volume Physique
```bash
pvcreate /dev/md127
pvdisplay /dev/md127
```
- Groupe de Volume
```bash
vgcreate vdisk1 /dev/md127
vgdisplay vdisk1
```
- Automatisation pour le démarrage une fois RAID et LVM configuré : 
```bash
vgchange -ay vdisk1
```
• Création des disques virtuels
```bash
lvcreate -L 100M -n lvol0 vdisk1
lvcreate -L 200M -n lvol1 vdisk1
lvscan
```
• Création des systèmes de fichiers
```bash
apt install ntfs-3g
mkfs.ext4 /dev/vdisk1/lvol0
mkfs.ntfs /dev/vdisk1/lvol1
```
• Monter les partitions
```bash
mkdir /mnt/linux
mkdir /mnt/windows
mount /dev/vdisk1/lvol0 /mnt/linux 
mount /dev/vdisk1/lvol0 /mnt/windows
```
- Créer un fichier dans ce repertoire
- Dans la configuration de la VM, supprimer un disque du RAID
- Constater : 
```bash
mdadm --detail /dev/md127
fdisk -l
mount
```
- Changement du Disque du Raid
- Recréer un disque de 1Go
```bash
apt install parted
partprobe
fdisk -l
mdadm --manage /dev/md1 --add /dev/sdc pour ajouter le disque au raid
mdadm --detail /dev/md127
```
