# SMB attack
Découverte et exploitation des protocoles LLMNR et NBT-NS

Lien Responder: https://github.com/lgandx/Responder

## Protocoles NBT-NS, LLMNR
>Link-Local Multicast Name Resolution (LLMNR) et Netbios Name Service (NBT-NS) sont deux composants présents en environnement Microsoft. LLMNR a été introduit dans Windows Vista et est le successeur de NBT-NS.

>Ces composants permettent d’aider à identifier des hôtes sur le même sous-réseau lorsque les services DNS centraux échouent. Ainsi, si une machine tente de résoudre un hôte particulier, mais que la résolution DNS échoue, la machine tentera alors de demander à toutes les autres machines du réseau local la bonne adresse via NBT-NS ou LLMNR :

>NBT-NS est basée sur l’identification par le nom NetBIOS – Utilise le port TCP 137
LLMNR est basé sur le format DNS (Domain Name System) – Utilise le port UDP 5355
Historiquement, Microsoft et Apple ont proposé comme standards leurs propres protocoles en se basant sur Multicast Domain Name Service: Microsoft a développé LLMNR et Apple mDNS. La différence majeure entre les deux protocoles se situe au niveau de la gestion des noms de domaine. Là où mDNS n’autorise que des noms de domaine sur l’espace « .local. », LLMNR ne filtre pas et autorise tous les noms de domaine (nous verrons par la suite que cela peut poser un réel problème de sécurité). LLMNR n’a pas réussi à s’imposer comme standard. Le protocole mDNS est beaucoup plus largement utilisés que LLMNR.

>Pourtant LLMNR ou NBT-NS (aussi obsolète) sont très utilisés dans Windows. mDNS a également été implémenté à partir de Windows 10, mais son utilisation est limitée à la découverte d’une imprimante en réseau.

## Recherche de noms sur le réseau
Quand un client recherche un nom sur le réseau il effectue sa recherche en plusieurs étapes et attend une réponse d’un service distant. Prenons un exemple, je recherche le partage réseau `\\srvinconnu`
Windows va effectuer les recherches via plusieurs mécanisme pour tenter de trouver `\\srvinconnu`

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

Exploitation des failles
Ces protocoles semblent inoffensifs, en théorie… Il ne sont là que pour faciliter la vie les utilisateurs et leur permettre d’accéder aux ressources sans configuration complexe. Malheureusement, cela ouvre une vulnérabilité majeure que des personnes mal intentionnées peuvent utiliser pour obtenir des informations d’identification complètes d’utilisateurs.
Un attaquant pourrait librement écouter sur un réseau les diffusions LLMNR (UDP / 5355) ou NBT-NS (UDP / 137) et y répondre, en prétendant que l’attaquant connaît l’emplacement de l’hôte demandé (Poisoning). Cette réponse contiendrait l’adresse IP d’un serveur malveillant qui disposerait d’une fonction de collecte de valeur d’identification (comme les hashs NTLMv1/v2  par exemple).

C’est justement ce scénario que nous allons maquetter ci après. Nous verrons aussi d’autres type d’attaque par ce mécanisme et comment s’en protéger.

Responder
L’outil utilisé pour ces attaques est Responder mais Metasploit de Rapid7 a aussi un module qui permet d’effectuer la même chose (auxiliary/spoof/llmnr/llmnr_response).

Responder est un outil « multithread » permettant de répondre à des requêtes IPv4 LLMNR et NBT-NS pour mener des opérations de poisoning de manière automatique. En plus de ces fonctions de poisoning, on y retrouve également des serveurs Rogue disposant d’une fonction de collecte des valeurs de hachage NTLMv1/v2 ou dans certain cas le mot de passe en clair.

Attaque SMB


 ![image](https://user-images.githubusercontent.com/83721477/150602762-3c8ff229-1a34-443f-bf06-3a0abc5252b0.png)


Ce type d’attaque consiste pour l’attaquant à écouter sur le réseau les diffusions LLMNR ou NBT-NS, et répondre quand un client essaye de se connecter à un serveur qui n’est pas connu du DNS. Cela signifie pour le client qu’il faut qu’il essaye de se connecter à un serveur inexistant ou pour lequel il se trompe d’orthographe. Par exemple `\\SVR01` au lieu de `\\SRV01` ou dans notre cas `\\SRVINCONNU`:

Vue coté attaquant (avec l’outil Responder en écoute)


 ![image](https://user-images.githubusercontent.com/83721477/150602769-36be0b4a-f397-451a-9fdb-10fb1a441fa3.png)


Vue coté poste client (Analyse de trame):

![image](https://user-images.githubusercontent.com/83721477/150602776-1105dd04-8c83-4e0d-b983-b5b4eb0d107e.png)


Nous pouvons constater ci dessus que le client (192.168.0.101) tente de résoudre par tout moyen `\\SRVINCONNU`, jusqu’à ce que Responder intercepte une requête et y réponde. Il va même y répondre avec les 2 protocoles LLMNR et NBTNS.  A présent notre Client sait que `\\SRVINCONNU` correspond à l’adresse IP 192.168.0.1 (notre serveur rogue). Ce dernier est en écoute sur le port TCP 445 (SMB) et va donc se faire passer pour un serveur légitime. Nous voyons dans la dernière trame ci dessus, que le client négocie le protocole d’authentification. Le serveur Rogue va lui indiquer qu’il ne supporte que des protocoles d’authentification « faible ». Le client étant autorisé à utiliser aussi ces protocoles d’authentification « faibles » va donc initier un processus d’authentification basé sur le protocole NTLM V2 et présenter au final son hash au serveur (« challenge-response » NTLM).

Attention: Il arrive souvent qu’il y ait des incompréhensions à ce sujet: 
Les hashs Net-NTLM (alias NTLM V2 dans l’exemple ci dessus) sont utilisés pour l’authentification. Ils sont dérivés d’un algorithme challenge / réponse et sont basés sur le hash NT de l’utilisateur. Le Hash NT de l’utilisateur se trouve dans la base NTDSI.dit présente sur les contrôleurs de domaine ou dans la base SAM d’un ordinateur client d’un domaine et chargé en mémoire par le processus LSASS.

Ce hash NTLM V2 est utilisable dans le cadre d’une attaque par Relay (via les scripts pythons ntlmrelayx.py ou MultiRelay.py qui est présent avec la suite Responder). Ce hash peut être aussi bruteforcé moyennant du temps processeur afin de trouver le mot de passe en clair.
Par contre, il ne pourra pas être utilisé dans des attaques Pass-The-Hash (Mimikatz) car c’est le hash NT qui est nécessaire dans ce cas).

L’exemple ci dessous montre une recherche du hash par dictionnaire en utilisant JohnTheRipper):
![image](https://user-images.githubusercontent.com/83721477/150602796-8bccf440-0419-4787-8eeb-8dae3c68a939.png)

A noter que le serveur SMB exécuté par Responder supporte l’ensemble des systèmes de type Microsoft Windows allant de la version NT4 jusqu’à Windows Server 2012 RC. Le système d’exploitation mac os x Mojave fait également partie de la liste des systèmes supportés par le serveur SMB de Responder, au même titre que tous les systèmes utilisant la solution Open Source Samba.

###### © 2022 Remi VERNIER, viperone, CYNET
