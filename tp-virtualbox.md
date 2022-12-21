# Bond-LVS-Web-RAID1 Sous VirtualBox
Agrégation de lien, avec répartition de charge du service Monkey Web

- Prérequis :
- 3 cartes réseau (Bridge, Bridge, Lan Segment)
- 3 Disques (20Go, 1Go, 1Go)
- Un peu de neurones
- 2 machines DSL

#### Agrégation de lien

- Installer le paquet ifsenslave
```bash
apt-get install ifenslave
```
- Configuration du Bond0 (Adapter le nom de la carte réseau)
```bash
nano /etc/modprobe.d/alias-bond.conf
alias bond0 bonding
options bonding mode=1 primary=enp0s3 fail_over_mac=1
```
Configuration de la carte réseau bond0 (Adapter le nom des cartes réseau)
- Désactiver au préalable les autres cartes réseau
```bash
nano /etc/network/interfaces
auto bond0
iface bond0 inet dhcp
bond-mode 1
bond-slaves enp0s3 enp0s8
```
- Déchargement du module (Supprime l'ancien Bond, si il existe)
```bash
rmmod bonding
```
- Rédémarrage du service, puis rédemarrage machine
```bash
/etc/init.d/networking restart
reboot
```
- Vérification
```bash
cat /proc/net/bonding/bond0
ip -br ad
```

#### Répartition de charge LVS/Serveur Web
- Ajouter une troisieme carte réseau (en LAN Segment) (Adapter le nom de la carte réseau)
```bash
nano /etc/network/interfaces
auto enp0s9
iface enp0s9 inet static
address 192.168.56.1
netmask 255.255.255.0
'Cette IP est la passerelle pour les machines DSL'
```
- Rédémarrage du service
```bash
/etc/init.d/networking restart
```
- Définir les IPs sur les machine DSL

![image](https://user-images.githubusercontent.com/73076854/208649588-5a670e60-ac6e-46e7-bf1a-c74c293e7ea1.png)
![image](https://user-images.githubusercontent.com/73076854/208649638-3360df0e-f5e6-4c5b-b1ee-3cab56b60661.png)

- Lancer le service Monkey Web

![image](https://user-images.githubusercontent.com/73076854/208650237-a144f383-0dd0-43df-b6b5-a7b1e11f17b1.png)

- Installation des paquets
```bash
apt install iptables
apt install iptables-persistent
apt install ipvsadm
```
- Mettre en place le NAT sur les paquets sortants vers internet (Adapter selon votre carte réseau)
```bash
iptables -t nat -A POSTROUTING -o bond0 -j MASQUERADE
iptables-save > /etc/iptables/rules.v4
```
- Autorisation du port 80
```bash
iptables -I INPUT -p tcp -m tcp --dport 80 -j ACCEPT
iptables-save > /etc/iptables/rules.v4
```
- Activation du routage
```bash
nano /etc/sysctl.conf
net.ipv4.ip_forward=1
```
-Rédémarrer votre machine pour appliquer le routage
```bash
reboot
```
- Créer un service virtuel
```bash
'XX = Machine Hote, YY/ZZ = Machines DSL'
ipvsadm -A -t 192.168.XX.XXX:80 -s rr
ipvsadm -a -t 192.168.XX.XXX:80 -r 192.168.YY.YYY -m
ipvsadm -a -t 192.168.XX.XXX:80 -r 192.168.ZZ.ZZZ -m
ipvsadm -L
```
- Vérification
```bash
apt install curl
curl 192.168.XX.XXX
ou
http://192.168.XX.XXX
```

![image](https://user-images.githubusercontent.com/73076854/208654898-56884b7f-ccf9-4194-b065-5c00422ac0a4.png)

#### Stockage RAID 1 (NE FONCTIONNE PAS SOUS VMWARE)
#### NE SURTOUT PAS REBOOT LA MACHINE PENDANT LA MANIPULATION

-Il est nécéssaire que vos disques soit branchable à chaud

![image](https://user-images.githubusercontent.com/73076854/208864749-dc000b05-e5ea-4ba5-ab88-fb54c326eaa1.png)

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
mdadm --manage /dev/md127 --add /dev/sdc
mdadm --detail /dev/md127
```
- Si cela ne fonctionne pas
```bash
apt install parted
partprobe
fdisk -l
mdadm --stop /dev/md127
mdadm -A --force /dev/md127
mdadm --manage /dev/md1 --add /dev/sdc
mdadm --detail /dev/md127
```
