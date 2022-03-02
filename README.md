# Linux CCNA - after  days linux
---

# Configure DHCP server, failover & relay 

- [Un simple serveur DHCP](#un-simple-serveur-dhcp)
- [Attribution d'une adresse ip statique](#attributionipstatic)
- [DHCP failover (redondance)](#dhcpfailover)
- [Remarks/Tips](#remarkstips)

## Un simple serveur DHCP

Installation du dhcp serveur

```
sudo apt update
sudo apt install isc-dhcp-server
```

ensuite **/etc/default/isc-dhcp-server**  
Indiquez votre carte réseau :  

```
INTERFACESv4="eno1"
INTERFACESv6=""
```

Ensuite la configuration du protocole **/etc/dhcp/dhcpd.conf**  

```
default-lease-time 86400;                       # Temps en secondes 86400 = 24h
max-lease-time 172800;                          # Temps en secondes 172800 = 48h / si la demande du client est plus elevée que le default
ddns-update-style none;
option domain-name-servers 1.1.1.1;             # Les DNS serveurs     
```
Et puis le subnet choisi:

```                failover peer "dhcp-failover";
subnet 192.168.1.0 netmask 255.255.255.0 {
        option routers 192.168.1.1;
        option subnet-mask 255.255.255.0;
        pool{
                range 192.168.1.50 192.168.1.250;
        }
}
```

On redémarre **isc-dhcp-server**

```
sudo systemctl restart isc-dhcp-server
```

<u>Et... C'est tout!</u>. Branchez un client sur le réseau.

---
## Attribution d'une adresse ip statique <a id="attributionipstatic"></a>

#use-host-decl-names on;
Toujours dans **dhcpd.conf** :

host station1
{
hardware ethernet 00:A0:45:65:E4:B0;
fixed-address 192.168.1.40;
}

## DHCP failover (redondance) <a id="dhcpfailover"></a>

On revient sur le premier serveur, et on rajoute la configuration failover dans le fichier **dhcpd.conf**  

```
failover peer "dhcp-failover" {
        primary;
        address 192.168.1.1;
        port 847;
        peer address 192.168.1.2;
        port 647;
        max-response-delay 60;
        max-unacked-updates 10;
        mclt 1800;
        split 128;  # 256 = no balancing / 128 = 50% loadbalancing
        load balance max seconds 5;
}
```
Rajouter une ligne dans le subnet :

>       failover peer "dhcp-failover";

Sur le second serveur, on va installer le daemon dhcp, pareil que pour le premier, et le configurer en serveur secondaire

```
sudo apt update
sudo apt install isc-dhcp-server
```

dans **/etc/default/isc-dhcp-server**    

```
INTERFACESv4="eno1"
INTERFACESv6=""
```
Dans **/etc/dhcp/dhcpd.conf**  

```
default-lease-time 86400;                       # Temps en secondes 86400 = 24h
max-lease-time 172800;                          # Temps en secondes 172800 = 48h / si la demande du client est plus elevée que le default
ddns-update-style none;
option domain-name-servers 1.1.1.1;             # Les DNS serveurs     

failover peer "dhcp-failover" {
        secondary;
        address 192.168.1.2;
        port 647;
        peer address 192.168.1.1;
        port 847;
        max-response-delay 60;
        max-unacked-updates 10;
        load balance max seconds 5;
}

subnet 192.168.1.0 netmask 255.255.255.0 {
        option routers 192.168.1.1;
        option subnet-mask 255.255.255.0;
        pool{
                range 192.168.1.50 192.168.1.250;
                failover peer "dhcp-failover";
        }
}
```
Redémarrer **isc-dhcp-server**

```
sudo systemctl restart isc-dhcp-server
```
Vérifier les logs et le journalctl pour voir ce qu'il se passe

## Remarks/Tips : <a id=remarkstips></a>

### On the DHCP server:

- Check the lease in CLI :

> sudo dhcp-lease-list

- Check leases in FILE :

> sudo vim /var/lib/dhcpd/dhcpd.leases

- Try/test the config file : 

> sudo dhcpd -t -cf /etc/dhcp/dhcpd.conf

How-To flush the DHCP server lease cache
How-To flush the DHCP server lease cache

Check your /var/lib/dhcpd/dhcpd.leases file and check for the entry. It contains the list of all dhcp leases.

Step 1:- Stop dhcp server

> service dhcpd stop

Step 2:- delete the temporary file dhcpd.leases~:

> rm dhcpd.leases~

Step 3:- flush the lease cache dhcpd.leases:

> echo "" > dhcpd.leases

Step4:- Start DHCP Server

> service dhcpd start


https://kb.isc.org/docs/isc-dhcp-41-manual-pages-dhcpdconf

sudo iptables -t nat -A POSTROUTING -o enx000ec6bc9ef6 -j MASQUERADE
sudo sysctl net.ipv4.ip_forward=1
sudo ip route add default via 192.168.1.254
- Clé OMAPI

---
# Linux_short Day2 ESSENTIAL
 
 **GUEST ADDitions** le plus gros interet pour nous :  
 - presse-papier partagé  
 - graphique amelioré (peut utiliser les drivers du PC, et avoir une full resolution)  
 
**Dynamic Kernel Module Support (DKMS)** est un programme/framework qui permet de générer des modules du noyau Linux dont les sources résident généralement en dehors de l'arbre des sources du noyau. Le concept est de reconstruire automatiquement les modules DKMS lorsqu'un nouveau noyau est installé.  

**uname (diminutif pour unix name)** est une commande Unix qui affiche les informations système sur la machine sur laquelle elle est exécutée.  
*Il est possible qu'on n'ait pas la mm version* www.kernel.org   

**Linux Headers**, ce sont des bibliothèques nécessaires pour développer et compiler des programmes.  

check if installed and all ok :
```
lsmod | grep vbox
sudo modinfo vboxguest
```
If issue:
```
sudo mount /dev/cdrom /mnt
sudo sh /mnt/VBoxLinuxAdditions.run
Reboot
```

-----------------------------------------------

# Command Line

## CLI vs. GUI

```
hawai@nitro5:~$

hawai = who
nitro5 = machine 
~ = where
$ user  # root

```
> date = when

```
whoami
who -> tty = teletype = console
dev/pts = pseudo-terminal
dev/pts/ptmx = the file /dev/ptmx (the pseudoterminal multiplexor device) 
if we do ls -l on /dev/tty* we can see a c

```

```
The file type is one of the following characters:
    -  regular file
    b  block special file
    c  character special file
    C  high performance ("contiguous data") file
    d  directory
    D  door (Solaris 2.5 and up)
    l  symbolic link
    M  off-line ("migrated") file (Cray DMF)
    n  network special file (HP-UX)
    p  FIFO (named pipe)
    P  port (Solaris 10 and up)
    s  socket
    ?  some other file type

```
    
> type / which / file

**ELF** : Executable and Linkable Format  
**interpreter /lib64/ld-linux-x86-64.so.2**: You have invoked ‘ld.so’, the helper program for shared library executables.  


man builtins  


**Everything is a file**  

https://en.wikipedia.org/wiki/Unix_filesystem#Conventional_directory_layout

man hier  

example : cd  

man  


```
echo (bash command/script)  

pwd  
ls  

cat  
tail/head  

touch  
mkdir  
cp  
mv  
ln -s  
rm  

```

-------

grep --> regex
find  


