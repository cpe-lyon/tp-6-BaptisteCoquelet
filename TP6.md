
# TP 6 - Services réseau

  

## Exercice 1. Adressage IP (rappels)

 Vous administrez le réseau interne 172.16.0.0/23 d’une entreprise, et devez gérer un parc de 254 machines réparties en 7 sous-réseaux. La répartition des machines est la suivante :
52  000  172.16.0.0/26
35  001  172.16.0.64/26
37  010  172.16.0.128/26
35  011  172.16.0.192/26
35  100  172.16.1.0/26
35  101  172.16.1.64/26
25  110  172.16.1.128/27

nombre d'hôte / numéro réseau en binaire / adresse sous réseau

- Sous-réseau 1 : 38 machines
- Sous-réseau 2 : 33 machines
- Sous-réseau 3 : 52 machines
- Sous-réseau 4 : 35 machines
- Sous-réseau 5 : 34 machines
- Sous-réseau 6 : 37 machines
- Sous-réseau 7 : 25 machines

## Exercice 2. Préparation de l’environnement

2. Démarrez le serveur et vérifiez que les interfaces réseau sont bien présentes. A quoi correspond l’interface appelée lo ?
on vérifie que les interface sont ben présente grâce a la commande `ip a`
L'interface appelée **lo** correspond a l'interface **loopback**.
  
3.  Dans les versions récentes, Ubuntu installe d’office le paquet cloud-init lors de la configuration du système. Ce paquet permet la configuration et le déploiement de machines dans le cloud via un script au démarrage. Nous ne nous en servirons pas et sa présence interfère avec certains services (en particulier le changement de nom d’hôte) ; par ailleurs, vos machines démarreront plus rapidement. Désinstallez complètement ce paquet (il faudra penser à le faire également sur le client ensuite.).

On désinstalle le paquet **cloud-init** avec la commande :
```
sudo apt remove cloud-init
```
  
4. Les deux machines serveur et client se trouveront sur le domaine tpadmin.local. A l’aide de la commande hostnamectl renommez le serveur (le changement doit persister après redémarrage, donc cherchez les bonnes options dans le manuel !). On peut afficher le nom et le domaine d’une machine avec les commandes hostname et/ou dnsdomainname ou en affichant le contenu du fichier /etc/hostname

on peut renommez le serveur en **Server** avec la commande

```
sudo hostnamectl set-hostname Server
```
pour que le changement persiste on doit modifier le fichier **/etc/hosts** et y indiqué le nouveau nom du serveur :
```
sudo nano /etc/hosts
```

## Exercice 3. Installation du serveur DHCP

1. Sur le serveur, installez le paquet isc-dhcp-server. La commande systemctl status isc-dhcp-server devrait vous indiquer que le serveur n’a pas réussi à démarrer, ce qui est normal puisqu’il n’est pas encore configuré (en particulier, il n’a pas encore d’adresses IP à distribuer).

On installe le paquet **isc-dhcp-server** :

```
sudo apt install isc-dhcp-server
```
la commande `systemctl status isc-dhcp-server` indique bien que le serveur n'a pas réussie a démarrer.
  
2. Un serveur DHCP a besoin d’une IP statique. Attribuez de manière permanente l’adresse IP 192.168.100.1 à l’interface réseau du réseau interne. Vérifiez que la configuration est correcte.

On fait la commande suivante pour accédé au fichier **/etc/netplan/01-netcfg.yaml**
```
sudo nano /etc/netplan/01-netcfg.yaml
```
et on y indique :
```
network :
	version : 2
	renderer : networkd
	ethernets :
		ens192 :
			addresses :
			192.168.100.1/24
```

Enfin on fait les commandes suivante :

```
netplan try
netplan apply
systemctl restart systemd-networkd
```

 
3. La configuration du serveur DHCP se fait via le fichier /etc/dhcp/dhcpd.conf. Faites une sauvegarde du fichier existant sous le nom dhcpd.conf.bak puis éditez le fichier dhcpd.conf avec les informations suivantes :

```
sudo cp /etc/dhcp/dhcpd.conf /etc/dhcp/dhcpd.conf.bak
sudo nano /etc/dhcp/dhcpd.conf
```
  

**default-lease-time** définit le temps avant lequel les adresses IP sont considéré comme expiré par le serveur expirent.

**max-lease-time** définit le temps maximal qu'on peut attribué a une adresses IP avant qu'elle expirent.


4. Editez le fichier /etc/default/isc-dhcp-server afin de spécifier l’interface sur laquelle le serveur doit écouter
```
sudo nano /etc/default/isc-dhcp-server
INTERFACESv4="ens224"
```

5. Validez votre fichier de configuration avec la commande dhcpd -t puis redémarrez le serveur DHCP (avec la commande systemctl restart isc-dhcp-server) et vérifiez qu’il est actif.

On valide la configuration avec `dhcpd -t` :

```
Internet Systems Consortium DHCP Server 4.4.1
Copyright 2004-2018 Internet Systems Consortium.
All rights reserved.
For info, please visit https://www.isc.org/software/dhcp/
Config file: /etc/dhcp/dhcpd.conf
Database file: /var/lib/dhcp/dhcpd.leases
PID file: /var/run/dhcpd.pid
```

Aprés avoir redemarrer le serveur dhcp avec la commande `systemctl restart isc-dhcp-server` on vérifie le statut du serveur avec `systemctl status isc-dhcp-server` :

```
● isc-dhcp-server.service - ISC DHCP IPv4 server
Loaded: loaded (/lib/systemd/system/isc-dhcp-server.service; enabled; vendor preset: enabled)
Active: active (running) since Mon 2022-09-26 10:09:59 UTC; 1min 28s ago
Docs: man:dhcpd(8)
Main PID: 1873 (dhcpd)
Tasks: 4 (limit: 1631)
Memory: 4.6M
CPU: 25ms
CGroup: /system.slice/isc-dhcp-server.service
└─1873 dhcpd -user dhcpd -group dhcpd -f -4 -pf /run/dhcp-server/dhcpd.pid -cf /etc/dhcp/dhcpd.conf ens224
```



9. Vérifiez que les deux machines peuvent communiquer via leur adresse IP, à l’aide de la commande ping.
  ![](https://lh5.googleusercontent.com/RiOn77KPbB2Qjzpx-5jIf5iJ47aOF0981UdGwKmuEwykPHlK64mj_v4d4DIYb6J9xRwCAF95Sz_4fB9YtwqaBbXCXIF9IHo8pfm9TMxmiQToguyBBXIuyyv3Hzmkv3yJLWcwzaeHjAWXQO79UbNKiVtrmCLb2x7MIo3Ln5ap76dejRUWnmfC37GKdw)
  
![](https://lh6.googleusercontent.com/0xozsHCLFuJ4JbSR7RhOHo_VXnpX5K4hJMJaHW_avoTvkQ0ZHPaB8PZvd9TirjgJNK5gSLX0m9I5oinhTgqzGPHQTYYCrhJXgoVyThqsZeGV64x518c9HFWdkjyph2Rp3yqwQmD-d5O1-rvN8cZRzZMnU9baBTVTZi_B8z6PZ5TOm9cAlHvB-D2FdQ)

10. Modifiez la configuration du serveur pour que l’interface réseau du client reçoive l’IP statique 192.168.100.20
![](https://lh4.googleusercontent.com/LQcWLwSKLZJlm2wfcEc0ITPjuV1KXkOP2Rp4VTKZXDpliOR_sanKYx9BrWluki88D3H0MXw__ym1imUMR8JozNFHhZa5oQIlqFggsEIx-1qAVDev9DdyQYXG3N3cxoyNzP4ur7Ae7lgz5Jnxem5Wiu057r7o_g8I3pJk_tbe5YxB_ZFseqk6j0bHvQ)


## Exercice 4. Donner un accès à Internet au client
. Ensuite, il faut autoriser la traduction d’adresse source (masquerading) en ajoutant la règle iptables suivante : sudo iptables --table nat --append POSTROUTING --out-interface enp0s3 -j MASQUERADE Vérifiez à présent que vous arrivez à « pinguer » une adresse IP (par exemple 1.1.1.1) depuis le client. A ce stade, le client a désormais accès à Internet, mais il sera difficile de surfer : par exemple, il est même impossible de pinguer www.google.com. C’est parce que nous n’avons pas encore configuré de serveur DNS pour le client.
![](https://lh3.googleusercontent.com/rXcrY5jqxtYcc8nLaEVcaqnzuDDYcZ-bF6l3hlVMGFL8QO3dgMffN8pkHs3jS_5pBm2UI0tM9AAAy_QTHbYBRL0nHSNW4RrSxZMH8jdgL1pOMBaX4bV7O5UEKW_h8FTACz5ZIH5J1vxLsBEO_XvWq_VoUs5SWItRjeLOMEC5JR-EiUOS0gEznU5IHg)
  

## Exercice 5. Installation du serveur DNS
Sur le client, retentez un ping sur www.google.fr. Cette fois ça devrait marcher ! On valide ainsi la configuration du DHCP effectuée précédemment, puisque c’est grâce à elle que le client a trouvé son serveur DNS.
![](https://lh3.googleusercontent.com/rXcrY5jqxtYcc8nLaEVcaqnzuDDYcZ-bF6l3hlVMGFL8QO3dgMffN8pkHs3jS_5pBm2UI0tM9AAAy_QTHbYBRL0nHSNW4RrSxZMH8jdgL1pOMBaX4bV7O5UEKW_h8FTACz5ZIH5J1vxLsBEO_XvWq_VoUs5SWItRjeLOMEC5JR-EiUOS0gEznU5IHg)
. Sur le client, installez le navigateur en mode texte lynx et essayez de surfer sur fr.wikipedia.org (bienvenue dans le passé...)*
![](https://lh6.googleusercontent.com/V6BH-vJwVVF2VKIvxb1U4vTYS-pBIuYwI6vxCGrT7H2Fj94xlR_Di8oa7ZC5l8buW_LSQFyX85iKY31pijR4tTq2isVZX_KOkaC8yz2AxH6CDz3EYswPZQTosxW_mexRmIvNebkBscbop157lrP7Hp15CHoTu58ioBfIKNiJVYymKXOU6Xx8Owk3wA)
  

