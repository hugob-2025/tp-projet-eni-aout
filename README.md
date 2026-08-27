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


