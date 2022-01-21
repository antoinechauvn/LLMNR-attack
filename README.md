# LLMNR-attack
Découverte et exploitation du protocole LLMNR

## Protocoles NBT-NS, LLMNR?
>Link-Local Multicast Name Resolution (LLMNR) et Netbios Name Service (NBT-NS) sont deux composants présents en environnement Microsoft. LLMNR a été introduit dans Windows Vista et est le successeur de NBT-NS.

>Ces composants permettent d’aider à identifier des hôtes sur le même sous-réseau lorsque les services DNS centraux échouent. Ainsi, si une machine tente de résoudre un hôte particulier, mais que la résolution DNS échoue, la machine tentera alors de demander à toutes les autres machines du réseau local la bonne adresse via NBT-NS ou LLMNR :

>NBT-NS est basée sur l’identification par le nom NetBIOS – Utilise le port TCP 137
LLMNR est basé sur le format DNS (Domain Name System) – Utilise le port UDP 5355
Historiquement, Microsoft et Apple ont proposé comme standards leurs propres protocoles en se basant sur Multicast Domain Name Service: Microsoft a développé LLMNR et Apple mDNS. La différence majeure entre les deux protocoles se situe au niveau de la gestion des noms de domaine. Là où mDNS n’autorise que des noms de domaine sur l’espace « .local. », LLMNR ne filtre pas et autorise tous les noms de domaine (nous verrons par la suite que cela peut poser un réel problème de sécurité). LLMNR n’a pas réussi à s’imposer comme standard. Le protocole mDNS est beaucoup plus largement utilisés que LLMNR.

>Pourtant LLMNR ou NBT-NS (aussi obsolète) sont très utilisés dans Windows. mDNS a également été implémenté à partir de Windows 10, mais son utilisation est limitée à la découverte d’une imprimante en réseau.

## Recherche de noms sur le réseau
Quand un client recherche un nom sur le réseau il effectue sa recherche en plusieurs étapes et attend une réponse d’un service distant. Prenons un exemple, je recherche le partage réseau \\srvinconnu
Windows va effectuer les recherches via plusieurs mécanisme pour tenter de trouver \\srvinconnu

* Recherche en utilisant le HOST  local de la machine (C:\Windows\System32\drivers\etc\hosts);
* S’il ne trouve rien: recherche dans le cache DNS local (ipconfig /displaydns);
* S’il ne trouve rien: recherche dans le/les DNS configuré(s) dans la carte réseau;
* S’il ne trouve rien: recherche NetBIOS  en utilisant NBT-NS et en broadcastant le subnet;
* Si personne ne lui répond: recherche en utilisant LLMNR et en diffusant sur l’adresse multicast;
* Si personne ne lui répond: échec de la recherche. Partage réseau introuvable.

Nous pouvons voir les 3 étapes avec une analyse de trame ci dessous (les 2 premières étant locales, elles ne s’affichent bien évidement pas):
*  DNS (Requête vers le serveur DNS 192.168.0.104 : name srvinconnu.test.local)
*  NBTNS (Diffusion vers adresse de broadcast de mon sous réseau 192.168.0.255)
*  LLMNR (Diffusion vers les adresses de multicast IPv4: 224.0.0.252 et Ipv6:  FF02::1:3)

En complément d’information: LLMNR supporte IPv6; NBT-NS ne supporte que IPv4
