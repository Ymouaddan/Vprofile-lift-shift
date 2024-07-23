Ce projet vise à déployer l'application Vprofile, développée en Java, sur une plateforme cloud AWS. L'application comporte deux niveaux principaux :

![Diagram_architecture](https://github.com/user-attachments/assets/70fe1bba-443e-4daa-b49a-5d71c800eba0)


### Objectifs du déploiement cloud :

- **Héberger l'application Vprofile dans un environnement cloud scalable et fiable.**
- **Automatiser le processus de déploiement pour une livraison rapide et cohérente des mises à jour.**
- **Mettre en œuvre des mesures de sécurité pour protéger l'application contre les accès non autorisés et les menaces.**

### Étapes de l’implémentation :

### 1. Créer une paire de clés pour se connecter aux instances EC2

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/13adfb3a-db8c-42bd-9f9c-1548801e3170/1dd46321-50d8-4fbc-98c4-c400c02efd31/Untitled.png)

### 2. Créer des groupes de sécurité pour définir les règles de trafic réseau

Pour cette partie, nous allons créer trois groupes de sécurité :

- **ALB-SG** : Autorise le trafic entrant sur les ports 80, 443.
- **my-app-sg** : Autorise le trafic entrant sur le port 8080 depuis la SG de l'ALB.
- **backend-vprofile-sg** : Autorise les ports 3306, 11211, 5800 depuis la SG de l'application pour assurer la communication entre l'application et ses composants en backend.

ALB-SG:

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/13adfb3a-db8c-42bd-9f9c-1548801e3170/adcd7c66-2bd9-49e0-a526-2b4784d75d91/Untitled.png)

my-app-sg : 

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/13adfb3a-db8c-42bd-9f9c-1548801e3170/83829f28-f7ec-4dce-b15b-b3099b90f179/Untitled.png)

backend-vprofile-sg: 

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/13adfb3a-db8c-42bd-9f9c-1548801e3170/6d85730c-1aa7-4be1-bf8a-bc0a866893de/Untitled.png)

- 3. Lancer quatre instances EC2 avec USER DATA pour configurer l'application sur les instances.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/13adfb3a-db8c-42bd-9f9c-1548801e3170/71b7045a-835f-4f3f-a225-b46939da2728/Untitled.png)

Tous les scripts se trouvent sur le répertoire GitHub : https://github.com/Ymouaddan/Vprofile-lift-shift.git

- **Mettre à jour la correspondance IP-nom** dans Route 53 pour l'accès aux instances.

Dans le code source nous utilison les nom de la machine pour la communication entre les diffirent instance eyet service pour ce raison nous avons cree une hosted zon DNS dans route 53 et nous avons ajouté des enregistrement de type A pour les 3 machine de backend 

(ici il ya une photo private hosted zone )

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/13adfb3a-db8c-42bd-9f9c-1548801e3170/c280f93c-e034-4c81-b41f-938e09a35459/Untitled.png)

Construire l'application en utilisant Maven pour créer l'artifact ROOT.war à partir du code source

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/13adfb3a-db8c-42bd-9f9c-1548801e3170/49678da5-f9fd-46f8-b25e-4a07ccabad9d/Untitled.png)

- 6. Configurer AWS CLI avec un compte ayant des accès sur S3 et uploader le fichier vprofile-v2.war

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/13adfb3a-db8c-42bd-9f9c-1548801e3170/157603b9-545e-4bb0-868f-9ad978b5ba10/Untitled.png)

Ensuite, se connecter à l'instance vprofile-app, télécharger l'artifact, arrêter le service Tomcat, supprimer le dossier ROOT et le remplacer par vprofile-v2.war que nous avons renommé en ROOT.war.

########################################################

ubuntu@ip-172-31-18-15:~$ aws s3 cp s3://vprofileliftandshift/vprofile-v2.war /tmp/
download: s3://vprofileliftandshift/vprofile-v2.war to ../../tmp/vprofile-v2.war
ubuntu@ip-172-31-18-15:~$ ls /tmp
vprofile-v2.war
ubuntu@ip-172-31-18-15:~$ sudo systemctl stop tomcat9
ubuntu@ip-172-31-18-15:~$ sudo su
root@ip-172-31-18-15:/home/ubuntu# rm -rf /var/lib/tomcat9/webapps/ROOT/
root@ip-172-31-18-15:/home/ubuntu# cp /tmp/vprofile-v2.war /var/lib/tomcat9/webapps/ROOT.war
root@ip-172-31-18-15:/home/ubuntu# systemctl start tomcat9
root@ip-172-31-18-15:/home/ubuntu# ls /var/lib/tomcat9/webapps/ROOT
ROOT.war

########################################################

### 7. Configurer un ELB avec HTTPS pour équilibrer la charge et sécuriser la connexion

La première étape est de créer un groupe cible VPROFILE-APP-TG dans lequel nous ajoutons les instances de destination et le port destiné qui est 8080. Ensuite, nous créons un load balancer qui écoute sur les ports 80, 443 et redirige vers le port 8080 de la machine vprofile-app. Nous utilisons le service ACM (Amazon Certificate Manager) pour demander un certificat afin de sécuriser le trafic sur le port 443.

Target group:

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/13adfb3a-db8c-42bd-9f9c-1548801e3170/54ffd88a-2110-4497-b0ce-6edbcb961335/Untitled.png)

ALB:

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/13adfb3a-db8c-42bd-9f9c-1548801e3170/a1b5168f-ec09-42a3-8c10-88b9cba25a88/Untitled.png)

### 9. Créer une auto scaling group pour l'instance vprofile-app pour assurer la scalabilité et la disponibilité de l'application

La première chose à faire est de créer une image AMI depuis notre instance vprofile-app.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/13adfb3a-db8c-42bd-9f9c-1548801e3170/8131d610-221a-410f-8ce7-fb292482be09/Untitled.png)

la premiere chose a faire c’est de cree une image AMI depuis notre instance vprofile-app

(ici il ya une photo de image créé  )

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/13adfb3a-db8c-42bd-9f9c-1548801e3170/38be85f7-5e60-4090-948d-a5b312a3e6ad/Untitled.png)

Ensuite, créer un launch template définissant l'image, les groupes de sécurité, les paires de clés et autres configurations à utiliser.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/13adfb3a-db8c-42bd-9f9c-1548801e3170/68723667-a290-4b0b-ae5d-38c2551a29e3/Untitled.png)

Ensuite, nous avons créé un groupe d'auto-scaling dans lequel nous avons défini le nombre minimal et maximal d'instances à lancer, ainsi que le type de scaling sur lequel il sera basé.
