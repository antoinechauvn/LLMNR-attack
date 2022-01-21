# LLMNR-attack
Découverte et exploitation du protocole LLMNR

>Link-Local Multicast Name Resolution (LLMNR) est un protocole capable d'effectuer la résolution de noms en l'absence d'un serveur DNS. Lorsqu'une demande est faite pour un partage tel que `\\Fileshares` et que l'utilisateur tape accidentellement `\\Filesharez`, ce qui rend le nom du partage incorrect, le DNS tente d'abord de résoudre le partage cible et s'il n'y parvient pas (car il n'existe pas), il se rabat sur LLMNR.
Une fois que le DNS n'a pas réussi à résoudre la requête et que LLMNR entre en jeu, la machine demandeuse envoie une diffusion sur le sous-réseau demandant si l'un des autres périphériques peut la connecter au partage. La machine attaquante sur le réseau répond à la requête en indiquant qu'elle peut la connecter au partage. À ce stade, la machine demandeuse (victime) envoie le nom d'utilisateur et le hachage NTLMv2 du compte demandant la ressource à la machine malveillante.

