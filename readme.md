# Utilisation d'oauth2 avec NGinx,oauth2-proxy, Azure AD pour donner l'accès à une application web


Repo pour le billet de blog : [https://blog.alterway.fr/utilisation-doauth2-avec-nginx-oauth2-proxy-azure-ad-pour-donner-un-acces-securise-a-une-application-web.html](https://blog.alterway.fr/utilisation-doauth2-avec-nginx-oauth2-proxy-azure-ad-pour-donner-un-acces-securise-a-une-application-web.html)


## Pré-requis

- Un cluster kubernetes
- Un ingress controller basé sur nginx
- Un cert-manager pour sécuriser les flux des ingress et générer des certificats ssl
- Une souscription Azure avec un compte administrateur pouvant créer des applications et leurs permissions
- les outils classiques pour interagir avec kubernetes (kubectl, krew, kustomize)

## Principe et processus de l'authentification oauth2

L'authentification OAuth2 permet à une application tierce d'accéder aux données d'un utilisateur sur un service tiers de manière sécurisée et sans avoir besoin de connaître les identifiants de l'utilisateur.

```bash
        +--------+                                    +---------------+
        |        |--(A)- Authorization Request ->     |               |
        |        |         Authorization Server       |               |
        |        |                                    |  Utilisateur  |
        |        |<-(B)-- Authorization Grant ------- |               |
        | Client |                                    |               |
        |        |--(C)-- Authorization Grant ------->|               |
        |        |          Resource Server           |               |
        |        |                                    |               |
        |        |<-(D)------ Access Token -----------|               |
        |        |                                    |               |
        |        |--(E)------ Access Token----------->|               |
        |        |            Protected Resource      |               |
        |        |                                    |               |
        |        |<-(F)------- Ressource demandée-----|               |
        |        |                                    |               |
        |        |--(G)------- Ressource demandée---->|               |
        |        |            Utilisateur             |               |
        +--------+                                    +---------------+
        

```


- L'application tierce demande l'autorisation d'accéder aux données de l'utilisateur en envoyant une requête d'autorisation au service tiers.
- Le service tiers redirige l'utilisateur vers sa page d'authentification où il doit entrer ses identifiants et autoriser l'application tierce.
- Si l'utilisateur autorise l'application tierce, le service tiers génère un jeton d'accès et le renvoie à l'application tierce.
- L'application tierce utilise le jeton d'accès pour accéder aux données de l'utilisateur auprès du service tiers.


Pour configurer l'authentification OAuth2 avec Microsoft AD, l'administrateur doit créer une application Azure AD, configurer les autorisations requises et fournir à l'application cliente les identifiants de l'application (ID client et clé secrète). L'application cliente peut ensuite utiliser ces identifiants pour demander des jetons d'accès OAuth2 au serveur d'autorisation.


L'authentification OAuth2 avec Azure AD offre plusieurs avantages, notamment :

- La sécurité : l'utilisateur n'a pas besoin de fournir ses identifiants de connexion à l'application tierce, ce qui réduit le risque de compromettre ses informations d'identification.
- La simplicité : l'application tierce n'a pas besoin de stocker les informations d'identification de l'utilisateur, ce qui simplifie la gestion des informations d'identification.
- L'évolutivité : l'authentification OAuth2 peut être utilisée pour autoriser l'accès à plusieurs applications tierces, ce qui permet une évolutivité facile et une gestion centralisée des autorisations.


Pour NGinx, nous allons déléguer l'authentification  a oauth2-proxy qui fera tout le travail.


## Proxy oauth2

Un proxy OAuth2 est un serveur intermédiaire qui permet à une application d'obtenir un jeton d'accès OAuth2 pour accéder à une ressource protégée par un fournisseur d'identité OAuth2.

Plus précisément, lorsqu'une application souhaite accéder à une API ou à une ressource protégée, elle doit obtenir un jeton d'accès valide auprès du fournisseur d'identité OAuth2. Le processus d'obtention d'un jeton d'accès peut être complexe et nécessite une communication entre l'application, le fournisseur d'identité et l'utilisateur final.

Le proxy OAuth2 agit comme un intermédiaire entre l'application et le fournisseur d'identité OAuth2, en gérant le processus d'authentification et d'autorisation à la place de l'application. En conséquence, l'application peut simplement envoyer une requête au proxy OAuth2 pour obtenir un jeton d'accès valide sans avoir à gérer le processus d'authentification complexe.


## Le projet oauth2-proxy

https://github.com/oauth2-proxy/oauth2-proxy

Le projet OAuth2 Proxy prend en charge plusieurs fournisseurs d'identité OAuth2 tels que Google, GitHub, Okta, Keycloak, Azure Active Directory, etc. Il offre également la possibilité de personnaliser l'apparence et le comportement du proxy en utilisant des fichiers de configuration. 
En résumé : OAuth2-Proxy est un projet open-source qui fournit une solution facile à utiliser pour sécuriser les ressources web avec l'authentification et l'autorisation basées sur OAuth2.

En outre, le projet OAuth2 Proxy offre des fonctionnalités de gestion des sessions, de gestion des cookies, de réécriture des URL et de journalisation des événements. Il est facile à déployer, extensible et peut être utilisé avec n'importe quelle application web qui prend en charge le protocole OAuth2.

