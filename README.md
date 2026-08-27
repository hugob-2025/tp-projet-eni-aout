Projet devops ENI

1)L’architecture de l’application

2)Les choix techniques

3)La démarche CI/CD

4)La configuration AKS

5)L’infrastructure déclarée avec Terraform

6)Le monitoring

7)Les difficultés rencontrées


1)L’architecture de l’application
L'ensemble de l'architecture est exécuté sur un cluster AKS. Il contiens plusieurs pods, notemment :

-mysql => pour le stockage des données de la todolist
-Backend => pour l'application todolist
-Frontend => pour l'application todolist
-Prometheus => pour la récupérations de métriques
-Grafana => pour afficher les métriques de prometheus via des dashboards
-kubeStateMetrics => pour la récupération de métriques des objets kubernetes
-NodeExporter => pour la récupération de métriques sur les nodes kubernetes
-nginx => pour NGINX Ingress, dans le but d'acceder a l'application todolist part internet

2)Les choix techniques
Organisation des namespaces : 

-Le namespace "default" contient les pods de l'application (frontend, backend, mysql)
-Le namespace "monitoring" contient les pods pour la partie monitoring (prometheus, grafana, kubestatemetrics, nodeExporter...)
-le namespace "app-routing-system" contient les pods pour l'accès a l'appli todolist par internet (nginx)

Conteneurisation avec docker : 

Le frontend et backend sont conteneurisé avec docker, et sont publié sous forme d'image fixe sur docker hub dans mon repository personnel.
Elles sont ensuite récupéré par kuberneters dans des fichiers .yaml pour créer des pods a partir de ces images. 

Utilisation de deployment pour la gestion des pods:

Les composants de l'application todolist (frontend, backend, mysql) sont déployés a l'aide d'un deployment, dans le but de pouvoir rajouter l'option "replicas", permettant de définir le nombre de pods qui doivent être maintenus.

3)La démarche CI/CD

J'utilise Github Actions pour mettre en place mon cicd. Mon workflow se déclenche lors d'un push ou d'un pull request sur la branche main.
Je réalise en premier le test du frontend et du backend. Lorsque ceux ci on reussi, je passe au build des deux images frontend et backend, puis, je les push sur docker hub.
Pour la partie nécessitant le déploiement automatique dans le cluster AKS, je n'ai pas réussi cette tache car je me suis retrouvé bloqué avec un manque de droit pour créer ma managed identity. J'avais une erreur me disant "RequestDisallowedByPolicy". J'ai donc choisi de passer cette étape.

4)La configuration AKS

L'application est hébergée sur un cluster AKS. Il contient deux nodes kubernetes, sur lesquels sont exécutés les différents pods. Les ressources Kubernetes de l'application sont définies à l'aide de fichiers YAML présents dans le dossier k8s/ . Ces fichiers permettent de créer les deployment, services, secrets, persistnetnvolumeclaim et regles nginx ingress pour le bon fonctionnement de l'application
La communication avec le cluster est réalisée avec l'aide de l'outil kubectl.

NGINX
NGINX est exécuté dans le namespace app-routing-system. Les règles de routage sont définies dans k8s/ingress.yaml. Deux chemins principaux sont configurés : / redirige les requêtes vers le Service frontend(port 80) /api redirige les requêtes vers le Service backend (port 3000). Son objectif est de permettre l'accès a l'application todolist par internet.

MYSQL
Pour mysql, sa configuration est séparée en quatre parties
Le premier fichier, mysql-pvc.yaml, demande a Kubernetes un espace de stockage persistant (PersitantVolumeClaim) pour mysql, configuré avec une taille de 5GiB.
Le deuxième fichier, mysql-secret.yaml contient un secret Kubernetes avec le mot de passe root mysql
Le troisième fichier, mysql-service.yaml, crée un service permettant aux autres pods de communiquer avec mysql via le port 3306
Enfin, le dernier fichier, mysql-deployment.yaml, décrit comment le pod mysql doit être lancé. En particulier, il n'a qu'un seul replica (un seul pod maintenu), il utilise la version de l'image mysql:8.0, utilise le port 3306, déclare les variables et utilise les variable de mysql-secret.yaml, et utilise le PVC mysql-pvc comme stockage persistant.

FRONTEND
Pour la frontend, sa configuration est en deux fichiers
le fichier frontend-service.yaml crée un service permettant au cluster de communiquer avec le pod frontend via le port 80
le fichier frontend-deployment.yaml décrit comment le pod frontend doit être lancé. En particulier, il n'a qu'un seul replica (un seul pod maintenu), il récupère l'image frontend de mon docker hub personnel, et il communiqueras avec les autres pods via le port 80

BACKEND
Enfin, la backend est configurée aveux deux fichiers 
le fichier backend-service.yaml crée un service permettant au cluster de communiquer avec le pod backend via le port 3000
le fichier backend-deployment.yaml décrit comment le pod backend doit être lancé. En particulier, il n'a qu'un seul replica (un seul pod maintenu), il récupère l'image backend de mon docker hub personnel, et il communiqueras avec les autres pods via le port 3000

5)L’infrastructure déclarée avec Terraform

L'infrastructure est déclarée dans le fichier main.tf.
J'y précise notamment la version de Terraform, mon node pool (2 nodes, de type Standard_B2ms), et mon ressource group (précisé comme déjà existant).
Un tag user = "HBaillet2025" est également défini sur les ressources créées par Terraform.

6)Le monitoring

Pour la partie monitoring, j'ai d'abord essayer une installation comme je l'ai fait pour la partie application au dessus, avec plusieurs fichier de config.yaml.
J'ai ensuite supprimé cette configuration, et je suis passé par helm pour réaliser la partie monitoring. J'ai donc créé un fichier values.yaml,  ou j'ai activé grafana (avec son login et mot de passe), prometheus, alertmanager, kubeStateMetrics et node Exporter.
Avec Helm, l'installation a été automatisée et la configuration de Grafana était déjà réalisée (data source déjà set, dashboards déjà créé). J'ai quand même néanmoins créé un dashboard manuellement pour tester la remontée de métriques.

7)Les difficultés rencontrées

Les principales difficultés que j'ai rencontrée sont :

D'abord pour le monitoring, la première version que j'ai mise en place était très longue a mettre en place, et ne récuperrais pas les bonnes métriques. L'installation via Helm m'a permis de gagner beaucoup de temps.

La plus grosse difficulté était de mettre en place le déploiement automatique vers AKS depuis Gtihub Actions. La création de la Managed Identity nécessaire à l'authentification entre GitHub Actions et Azure était bloquée par une politique Azure, avec l'erreur RequestDisallowedByPolicy. Je n'ai pas trouvé d'alternatives et j'ai donc décidé de sauter cette étape.
