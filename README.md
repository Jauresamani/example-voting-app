# Projet Kubernetes - Déploiement de l'application Example Voting App
 
## Objectif
 
Mettre en place l’application open-source [Example Voting App](https://github.com/dockersamples/example-voting-app) dans un cluster Kubernetes, comprendre son architecture et tester le cycle de vote en temps réel.
 
---
 
## Prérequis
 
- Un cluster fonctionnel
- `kubectl` installé et configuré
- Git
 
---

# Infrastructure mise en place

![architecture](./images/architecture.JPG)  
Voici l'architecture mise en place pour notre application.  
On y retrouve nos différents pods et services ainsi que la communication entre eux.  

L'application de vote fonctionne comme un système distribué composé de plusieurs services interconnectés. L'utilisateur interagit d'abord avec une application web frontend, qui lui permet de voter entre deux options. Lorsqu’un vote est soumis, il est envoyé à un serveur Redis, qui joue le rôle de file d’attente pour stocker temporairement les votes de manière rapide et efficace. Ensuite, un worker interroge Redis pour récupérer les nouveaux votes. Chaque vote récupéré est ensuite enregistré dans une base de données PostgreSQL,  pour garantir que les données ne soient pas perdues en cas de redémarrage. Enfin, une application web se connecte à cette base de données pour afficher les résultats du vote en temps réel, en les mettant à jour dynamiquement afin que les utilisateurs puissent voir l’évolution des votes en direct.  
  
## Clonage du dépôt


git clone https://github.com/dockersamples/example-voting-app.git
cd example-voting-app/k8s-specifications

# Déploiement des ressources Kubernetes

```bash
## Création d'un namespace pour isoler nos ressources
kubectl create ns voting-app  
```


  
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

```bash
kubectl get pod -n voting-app  
```

![image alt](https://github.com/Jauresamani/example-voting-app/blob/main/screenREADME/3getpods.png)
On retrouve bien tous nos pods qui sont en état running.  

```bash
kubectl get deployment -n voting-app  
```
![image alt](https://github.com/Jauresamani/example-voting-app/blob/main/screenREADME/4getdeployem.png) 

On retrouve bien tous nos deployments comme on peut le voir dans la colonne READY.  

```bash
kubectl get rs -n voting-app
```
![image alt](https://github.com/Jauresamani/example-voting-app/blob/main/screenREADME/5getrs.png) 
On retrouve bien tous nos replicaSet comme on peut le voir dans la colonne DESIRED. 
  
```bash
kubectl get svc -n voting-app  
```
![image alt](https://github.com/Jauresamani/example-voting-app/blob/main/screenREADME/6getserv.png) 
On retrouve bien tous nos services avec les ports sur lesquels ont peut les joindre.
  
## Vue d'ensemble des ressources
```bash
kubectl get all -n voting-app  
```
![image alt](https://github.com/Jauresamani/example-voting-app/blob/main/screenREADME/7get%20all.png)
On retrouve bien toutes nos ressources créées et utilisées par notre application !  
  
## Accéder et tester l'aplication
On peut vérifier l'adresse ip du node avec la commande :  

kubectl describe node  

![image alt](https://github.com/Jauresamani/example-voting-app/blob/main/screenREADME/8descri.png)

Et ensuite accéder à l'interface de vote depuis le navigateur : http://192.168.49.2:31000  
![image alt](https://github.com/Jauresamani/example-voting-app/blob/main/screenREADME/9vote.png) 
  
Accéder à l'interface de resultat depuis le navigateur : http://192.168.49.2:31001  
![image alt](https://github.com/Jauresamani/example-voting-app/blob/main/screenREADME/10resul.png) 
  
## Résultats
Vérifier que les resultats s'actualisent en temps réel :
![image alt](https://github.com/Jauresamani/example-voting-app/blob/main/screenREADME/11votedirect.png)  
![image alt](https://github.com/Jauresamani/example-voting-app/blob/main/screenREADME/12resultdirect.png)  
  
Notre application est donc bien fonctionnelle !  
