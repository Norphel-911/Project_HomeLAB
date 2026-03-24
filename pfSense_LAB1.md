# pfSense Configuration
## 1. Objectif
Configurer pfsense comme firewall et routeur pour le LAB avec segmentation et contrôle du trafic.    

-le routage entre les VLANS et internet     	
-la fonction de DHCP pour les VLANS  
-DNS forwarding   
-les règles Firewall pour contrôler le flux de trafic dans le réseau.  

## 2. Configuration réseau

| Interface | Rôle                    | IP               |
| --------- | ----------------------- | ---------------- |
| em0       | WAN                     | DHCP(Vmware NAT) |
| em1       | LAN (VLAN10 - serveurs) | 192.168.10.1/24  |
| zm2       | OPT1 (VLAN20 - clients) | 192.168.20.1/24  |

==*A savoir*== : Vmware nécessite une carte réseau par VLAN, alors que VirtualBox n'a besoin d'une seule carte réseau pour gérer tous les vlan.  
#### Sur la console:
Option 1 - Assigner les interfaces    
	WAN = em0   
	LAN (VLAN 10) = em1   
	OPT1 (VLAN 20) = em2   
Option 2 - Assigner les IPs      
	em1 = 192.168.10.1/24 + SANS DHCP, car les serveurs auront besoins d'une IP fixe.   
	em2 = 192.168.20.1/24 + DHCP  
 
![Console de mon pfsense après la config.](C:\Users\tenzi\Pictures\Numérisations/console-pfsense.png)

## 3. Web_GUI pfsense
### Configuation d'ubuntu vm : 

-address ip =192.168.10.20/24 static
-gateway = 192.168.10.1
-DNS = 192.168.10.1
----> ping vers pfsense ✅

Depuis une page web on rentre : 
	http://192.168.10.1
	Configuration Wizard :
		Hostname = pfsense
		Domain = local.LAB1
		Primary DNS = 8.8.8.8
		Secondary DNS = 8.8.4.4
		NTP serer = 2.pfsense.pool.ntp.org
		Time Zone = Europe/Paris

![Dashboard pfsense.](C:\Users\tenzi\Pictures\Numérisations/pfsense_dashboard1.png)

## 4. Problème de DNS 
Problème de résolution dns: 
	ping 8.8.8.8 = ✅
	ping google.com = ❌
### Cause : 
Le service systemd-resolved utilise un DNS local (127.0.0.53) au lieu Gateway pfsense. 
### Solution :
```
sudo systemctl disable systemd-resolved
sudo systemctl stop systemd-resolved 
sudo rm /etc/resolv.conf
echo "nameserver 192.168.10.1" | sudo tee /etc/resolv.conf
```
OU
On peut directement modifier le code source du service dans /etc/systemd/resolved.conf
on décommente la ligne du dns dans le fichier et ajoute dns = 192.168.20.1

Ainsi:
	ping 8.8.8.8 = ✅
	ping google.com = ✅

## 5. Création d'une règle firewall.

Blocage d'un site via règle firewall:
	On va utiliser nslookup pour repérer ip d'un site ou on peut utiliser DNS lookup dans pfsense.

1. On peut voir que tout fonctionne correctement. 
![Dashboard pfsense.](C:\Users\tenzi\Pictures\Numérisations/nslookup_ping.png)

2. Une règle crée qui bloque tous les requêtes envoyé par la machine(192.168.10.20) vers le site(192.54.115.242).
![Dashboard pfsense.](C:\Users\tenzi\Pictures\Numérisations/firewall_rule.png)

3. Une fois que la règle est mise en place, impossible d'accéder au site car les trafics sont capturé par la règle firewall et bloqués.
![Dashboard pfsense.](C:\Users\tenzi\Pictures\Numérisations/ping_after.png)
