1: *à quoi va servir ce pfsense.*
## 1. le but 
PfSense se comporte comme firewall et un routeur pour le LAB. Il gère:
	-le routage entre les VLANS et internet. 
	-la fonction de DHCP pour les VLANS
	-DNS forwarding 
	-les règles Firewall pour contrôler le flux de trafic dans le réseau. 

2: *Assigner les interfaces et les VLAN, et donner les ips, avec son resultat*
### 2. Configuration des interfaces et de VLAN

| Interface | Role                    | IP               |
| --------- | ----------------------- | ---------------- |
| em0       | WAN                     | DHCP(Vmware NAT) |
| em1       | LAN (VLAN10 - serveurs) | 192.168.10.1/24  |
| zm2       | OPT1 (VLAN20 - clients) | 192.168.20.1/24  |

==*A savoir*== : Vmware nécessite une carte réseau par VLAN, alors que VirtualBox n'a besoin d'une seule carte réseau pour gérer tous les vlan.  
#### Sur la console:
Option 1 - Assigner les interfaces
	-em0 > WAN
	-em1 > LAN (VLAN 10)
	-em2 > OPT1 (VLAN 20)
Option 2 - Assigner les IPs
	-em1 > 192.168.20.1/24 + SANS DHCP, car les serveurs auront besoins d'une IP fixe. 
	-em2 > 192.168.20.1/24 + DHCP

-placeholder: screen de la console de pfsense montront les interfaces. 


3: *se connecter à pfsense Web_GUI depuis une vm(ubuntu)*
## 3. Web_GUI pfsense
### Configuation d'ubuntu vm : 
		-192.168.10.20/24 static
		-gateway > 192.168.10.1
		-DNS > 192.168.10.1
La machine peut ping l'ip LAN de pfsense.
Depuis une page web on rentre : 
	http://192.168.10.1
	Configuration Wizard :
		Hostname = pfsense
		Domain = local.LAB1
		Primary DNS = 8.8.8.8
		Secondary DNS = 8.8.4.4
		NTP serer = 2.pfsense.pool.ntp.org
		Time Zone = Europe/Paris
placeholder : Webgui Setup page. 

4: *Les problèmes rencontrés, résoundre le problème*

## 4. Problème de DNS 
On a access à internet mais la resolution DNS ne fonctionne pas: 
	ping 8.8.8.8 > fonctionne.
	ping google.com > fonctionne pas 
La cause : 
Sur le vm ubuntu, on a systemd-resolved qui pointe vers 127.0.0.53 au lieu Gateway pfsense. 


5 : *Créer une règles pour voir si tout marche correctement*


