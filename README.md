# Linux CCNA courses  
Trainer : Nico  
Location: IRISIB  

---

# Configure DHCP server, failover 

- [Un simple serveur DHCP](#un-simple-serveur-dhcp)
- [Attribution d'une adresse ip statique via DHCP](#attributionipstatic)
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

> sudo vim /var/lib/dhcp/dhcpd.leases

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

