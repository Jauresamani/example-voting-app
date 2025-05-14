Nom : AMANI Kouakou Jaures De Sales

ESGI 5SRC2

# Projet Kubernetes - Déploiement de l'application Example Voting App

## Contexte
Dans le cadre de l’amélioration d’une application de vote existante, une entreprise souhaite s’inspirer du projet open-source Example Voting App. L’objectif est de le déployer dans un environnement Kubernetes pour en étudier le fonctionnement, valider son interactivité en temps réel et concevoir une architecture claire mettant en évidence les objets Kubernetes utilisés ainsi que leurs interactions.

 
## Objectif
Déployer l’application [Example Voting App](https://github.com/dockersamples/example-voting-app) dans un cluster Kubernetes local (Minikube) à l’aide des manifestes YAML fournis.
Accéder aux interfaces vote et result et tester que les votes apparaissent immédiatement dans les résultats des votes en temps réel.
Documenter l’ensemble du processus dans un README explicite :
Toutes les commandes utilisées
Captures d’écran des étapes clés
Schéma architectural clair des objets Kubernetes déployés et leurs relations (réalisé avec un outil comme Draw.io)

---
 
## Prérequis
 
- Un cluster fonctionnel (Minikube)
- kubectl installé et configuré
- Git installé
 
---

# Infrastructure mise en place

![archi](https://github.com/Jauresamani/example-voting-app/blob/main/screenREADME/Archi.png)  
Architecture mise en place pour notre application avec les differents commposants kubernetes, services, pods, ressources et liaisons.  


L'application de vote fonctionne comme un système distribué composé de plusieurs services interconnectés. L'utilisateur interagit d'abord avec une application web frontend, qui lui permet de voter entre deux options. Lorsqu’un vote est soumis, il est envoyé à un serveur Redis, qui joue le rôle de file d’attente pour stocker temporairement les votes. Ensuite, un worker interroge Redis pour récupérer les nouveaux votes, qui sont ensuite enregistrés dans une base de données PostgreSQL. Enfin, une application web se connecte à cette base pour afficher les résultats en temps réel.

## Clonage du dépôt
git clone https://github.com/dockersamples/example-voting-app.git
cd example-voting-app/k8s-specifications

# Déploiement des ressources Kubernetes

bash
## Création d'un namespace pour isoler nos ressources
Un namespace dédié permet d’isoler les ressources de l’application pour faciliter leur gestion

kubectl create ns voting-app  


## Déploiement des services de données


kubectl apply -f redis-deployment.yaml -n voting-app  
kubectl apply -f redis-service.yaml -n voting-app  
kubectl apply -f db-deployment.yaml -n voting-app  
kubectl apply -f db-service.yaml -n voting-app  

![image alt](https://github.com/Jauresamani/example-voting-app/blob/main/screenREADME/1De%CC%81ploiement%20des%20services%20de%20donne%CC%81es.png) 
 
## Déploiement du worker
kubectl apply -f worker-deployment.yaml -n voting-app

 
## Déploiement des interfaces utilisateur (vote et résultats)
kubectl apply -f vote-deployment.yaml -n voting-app  
kubectl apply -f vote-service.yaml -n voting-app  
kubectl apply -f result-deployment.yaml -n voting-app  
kubectl apply -f result-service.yaml -n voting-app  

![image alt](https://github.com/Jauresamani/example-voting-app/blob/main/screenREADME/2De%CC%81ploiement%20du%20worker.png)

## Vérification
Après le déploiement, il est essentiel de s’assurer que tous les composants sont bien créés et fonctionnent

kubectl get pod -n voting-app  

![image alt](https://github.com/Jauresamani/example-voting-app/blob/main/screenREADME/3getpods.png)
Tous les pods sont lancés et en état "Running".

kubectl get deployment -n voting-app  

![image alt](https://github.com/Jauresamani/example-voting-app/blob/main/screenREADME/4getdeployem.png) 

Les déploiements sont opérationnels : tous les pods attendus sont bien créés et actifs

kubectl get rs -n voting-app

![image alt](https://github.com/Jauresamani/example-voting-app/blob/main/screenREADME/5getrs.png) 
Les ReplicaSets assurent la gestion du nombre de pods souhaité pour chaque service
  

kubectl get svc -n voting-app  

![image alt](https://github.com/Jauresamani/example-voting-app/blob/main/screenREADME/6getserv.png) 

Les services sont bien exposés avec les bons ports pour permettre la communication interne et l'accès externe.


## Vue d'ensemble des ressources

kubectl get all -n voting-app  

![image alt](https://github.com/Jauresamani/example-voting-app/blob/main/screenREADME/7get%20all.png)
On retrouve bien toutes nos ressources créées et utilisées par notre application !  
  
## Accéder et tester l'aplication
Récupération de l'adresse IP du node 

kubectl describe node  

Permet de connaître l’adresse IP du nœud pour accéder aux services exposés en NodePort

![image alt](https://github.com/Jauresamani/example-voting-app/blob/main/screenREADME/8descri.png)

Et ensuite accéder à l'interface de vote : http://192.168.49.2:31000  
![image alt](https://github.com/Jauresamani/example-voting-app/blob/main/screenREADME/9vote.png) 
  
Accéder à l'interface de resultat: http://192.168.49.2:31001  
![image alt](https://github.com/Jauresamani/example-voting-app/blob/main/screenREADME/10resul.png) 
  
## Résultats
Resultats en en temps réel :
![image alt](https://github.com/Jauresamani/example-voting-app/blob/main/screenREADME/12resultdirect.png)  
![image alt](https://github.com/Jauresamani/example-voting-app/blob/main/screenREADME/11votedirect.png)  



## Conclusion 
Notre application est désormais entièrement fonctionnelle dans Kubernetes. Ce projet nous a permis de mettre en œuvre une architecture microservices complète, de comprendre les interactions entre les composants, et de valider le fonctionnement d’un flux de traitement distribué.  
