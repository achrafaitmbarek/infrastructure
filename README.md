# Bank Platform - Infrastructure

C'est le repo GitOps pour le projet bancaire. Je l'ai mis en place pour gérer le pont entre GitLab et notre instance AWS EC2. L'objectif est simple : fini les déploiements SSH manuels. Vous poussez le code, le serveur se met à jour.

## Architecture
Tout est orchestré via Docker Compose. J'ai gardé la configuration la plus légère possible pour économiser la RAM sur AWS :

* **Keycloak** : Tourne sur le port `8180`. J'ai inclus le fichier `realm-export.json` pour l'import automatique au démarrage.
* **Postgres 15** : Une seule instance partagée entre Keycloak et le service d'authentification.
* **Kafka & Zookeeper** : Images Confluent standards pour le bus d'événements.
* **Bank-Auth-Service** : Notre image personnalisée récupérée depuis ECR.

## Flux de déploiement
Le pipeline défini dans `.gitlab-ci.yml` est direct mais strict. Il utilise un agent SSH pour se connecter à la VM, lance un `git pull`, puis exécute `docker-compose up -d`.

**Note sur la sécurité :** Le port 22 d'AWS doit rester ouvert pour les runners GitLab. Si le pipeline tombe en "timeout", vérifiez d'abord les règles de votre Security Group.

## Environnement GitLab
Vérifiez que ces variables sont bien configurées dans votre CI/CD, sinon le job va planter :
1. **SSH_PRIVATE_KEY** : Le contenu de votre fichier `.pem`. Utilisez impérativement le type **File** dans les paramètres GitLab.
2. **Identifiants AWS** : ID, Secret et Région pour la connexion à l'ECR.
3. **AWS_ECR_REGISTRY** : Votre URI de registre spécifique.

## Debug rapide
Si ça plante, ne redémarrez pas toute la VM. Regardez d'abord les logs :
`docker logs -f bank-auth-service`

Si vous avez besoin de forcer un état propre (re-création des conteneurs) :
`docker-compose -f docker-compose.aws.yml up -d --force-recreate`

## Auteur

Achraf Ait M'Barek — the-crazy-achraf@hotmail.fr