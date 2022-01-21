# LLMNR-attack
Découverte et exploitation du protocole LLMNR

>Link-Local Multicast Name Resolution (LLMNR) est un protocole capable d'effectuer la résolution de noms en l'absence d'un serveur DNS. Lorsqu'une demande est faite pour un partage tel que `\\Fileshares` et que l'utilisateur tape accidentellement `\\Filesharez`, ce qui rend le nom du partage incorrect, le DNS tente d'abord de résoudre le partage cible et s'il n'y parvient pas (car il n'existe pas), il se rabat sur LLMNR.
Une fois que le DNS n'a pas réussi à résoudre la requête et que LLMNR entre en jeu, la machine demandeuse envoie une diffusion sur le sous-réseau demandant si l'un des autres périphériques peut la connecter au partage. La machine attaquante sur le réseau répond à la requête en indiquant qu'elle peut la connecter au partage. À ce stade, la machine demandeuse (victime) envoie le nom d'utilisateur et le hachage NTLMv2 du compte demandant la ressource à la machine malveillante.

## NTML
>NTLM (NT Lan Manager) est un protocole d'authentification utilisé dans diverses implémentations des protocoles réseau Microsoft et pris en charge par le « NTLMSSP » (Fournisseur de support de sécurité NT LM). À l'origine utilisé pour une authentification et une négociation sécurisée, NTLM est aussi utilisé partout dans les systèmes de Microsoft comme un mécanisme d'authentification unique (single sign-on).
NTLM utilise un mécanisme de « stimulation/réponse » (« challenge-response ») pour l'authentification, dans laquelle les clients sont capables de prouver leurs identités sans envoyer un mot de passe au serveur. Cela consiste en trois messages, généralement mentionnés comme Type 1 (la négociation), Type 2 (la stimulation) et Type 3 (l'authentification). Il fonctionne essentiellement comme suit :

* **Le client envoie un message Type 1 au serveur. Celui-ci contient principalement une liste des fonctionnalités prises en charge par le client et demandées au serveur.**

* **Le serveur répond par un message Type 2. Celui-ci contient une liste de fonctionnalités prises en charge et accordées par le serveur. Le plus important cependant, il contient une « stimulation » produite par le serveur.**

* **Le client répond à la « stimulation » avec un message Type 3. Celui-ci contient plusieurs informations au sujet du client, comprenant le domaine et le nom d'utilisateur du client. Il contient également une ou plusieurs réponses à la « stimulation » Type 2.
Les réponses dans le message du type 3 sont la partie la plus critique, car elles prouvent au serveur que l'utilisateur client a la connaissance du mot de passe du compte.
L'authentification Stimulation/Réponse de Windows NT ne pouvant pas franchir un pare-feu, elle est surtout utile sur les intranets, où la communication intervient derrière le pare-feu d'une organisation. Elle est également souvent utilisée pour l'authentification des utilisateurs sur un serveur proxy.**

https://github.com/lgandx/Responder

https://www.cynet.com/attack-techniques-hands-on/llmnr-nbt-ns-poisoning-and-credential-access-using-responder/

![image](https://user-images.githubusercontent.com/83721477/150578911-fb890914-8b91-4a75-a363-eff19a8aa559.png)
![image](https://user-images.githubusercontent.com/83721477/150578947-38ab83a8-8906-48c2-9329-8530d78170ff.png)
![image](https://user-images.githubusercontent.com/83721477/150577784-f74b1e0c-ef6f-426e-be3d-e260ebb7878f.png)
![image](https://user-images.githubusercontent.com/83721477/150579004-d8fe516d-3de3-4fad-8250-cc297af2f491.png)
![Responder-Edit](https://user-images.githubusercontent.com/83721477/150579450-b9d6807d-8680-4ff9-be10-d91c02c7338b.gif)
