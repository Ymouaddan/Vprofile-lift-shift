Ce projet vise à déployer l'application Vprofile, développée en Java, sur une plateforme cloud AWS. L'application comporte deux niveaux principaux :

![Diagram_architecture](https://github.com/user-attachments/assets/70fe1bba-443e-4daa-b49a-5d71c800eba0)


### Objectifs du déploiement cloud :

- **Héberger l'application Vprofile dans un environnement cloud scalable et fiable.**
- **Automatiser le processus de déploiement pour une livraison rapide et cohérente des mises à jour.**
- **Mettre en œuvre des mesures de sécurité pour protéger l'application contre les accès non autorisés et les menaces.**

Services Utulisés:

- **EC2** (Elastic Compute Cloud)** : Déploiement et gestion des instances.
- **Amazon S3 (Simple Storage Service)** : Stockage des artefacts et des fichiers.
- **IAM (Identity and Access Management)** : Gestion des permissions et des accès sécurisés.
- **VPC (Virtual Private Cloud)** : Configuration des réseaux et sous-réseaux.
- **Route 53** : Gestion des noms de domaine et mapping IP-nom.
- **Security Groups** : Définition des règles de trafic réseau.
- **ACM (AWS Certificate Manager)** : Gestion des certificats SSL pour sécuriser les connexions HTTPS.
- **ELB (Elastic Load Balancing)** : Équilibrage de la charge et gestion des connexions sécurisées.
- **Auto Scaling** : Gestion automatique de la scalabilité des instances.
- **Maven** : Outil de build pour créer l'artefact de l'application Java.
- **Tomcat** : Serveur d'applications pour déployer l'artefact Java.
- **GitHub** : Stockage et gestion du code source et des scripts.

###   Compétences et bénéfices acquis :

-**Gestion des services cloud** : Compétence en déploiement et gestion d'applications sur AWS.
-**Sécurité et contrôle d'accès** : Mise en œuvre de pratiques de sécurité robustes avec IAM et les groupes de sécurité.
-**Automatisation du déploiement** : Utilisation d'outils et de services pour automatiser les processus de déploiement.
-**Scalabilité et haute disponibilité** : Configuration des groupes d'auto-scaling et des load balancers pour assurer la disponibilité de l'application.
-**Gestion de noms de domaine et DNS** : Configuration et gestion des enregistrements DNS avec Route 53.
-**Certifications SSL** : Gestion et intégration des certificats SSL pour sécuriser les connexions.
-**Stockage et gestion des artefacts** : Utilisation d'Amazon S3 pour le stockage et la gestion des fichiers déployés.
-**Développement et build** : Utilisation de Maven pour compiler et construire des applications Java.
-**Déploiement d'applications web** : Compétence en déploiement d'applications sur Tomcat.
-**Collaboration et gestion du code** : Utilisation de GitHub pour la gestion du code source et des scripts de déploiement.



### Étapes de l’implémentation :

### 1. Créer une paire de clés pour se connecter aux instances EC2

![keypaire](https://github.com/user-attachments/assets/39ad9d34-de43-44c4-8e1f-561c7f6afa39)


### 2. Créer des groupes de sécurité pour définir les règles de trafic réseau

Pour cette partie, nous allons créer trois groupes de sécurité :

- **ALB-SG** : Autorise le trafic entrant sur les ports 80, 443.
- **my-app-sg** : Autorise le trafic entrant sur le port 8080 depuis la SG de l'ALB.
- **backend-vprofile-sg** : Autorise les ports 3306, 11211, 5800 depuis la SG de l'application pour assurer la communication entre l'application et ses composants en backend.

ALB-SG:

![ALB-SG](https://github.com/user-attachments/assets/01cbf9a6-1bde-4622-be9a-09a835fd4782)


my-app-sg : 

![my-app-sg](https://github.com/user-attachments/assets/ea1cb6af-a94c-43d6-836b-bcf4fb46297b)


backend-vprofile-sg: 

![backend-sg](https://github.com/user-attachments/assets/c0b6d7e9-44f6-44cc-baad-ab8fd0e67f73)


- 3. Lancer quatre instances EC2 avec USER DATA pour configurer l'application sur les instances.

![instances](https://github.com/user-attachments/assets/de200677-02ef-4112-8575-0b048ed68a74)


Tous les scripts se trouvent sur le répertoire GitHub : https://github.com/Ymouaddan/Vprofile-lift-shift.git

- **Mettre à jour la correspondance IP-nom** dans Route 53 pour l'accès aux instances.

Dans le code source nous utilison les nom de la machine pour la communication entre les diffirent instance eyet service pour ce raison nous avons cree une hosted zon DNS dans route 53 et nous avons ajouté des enregistrement de type A pour les 3 machine de backend 


![private_hosted_zone](https://github.com/user-attachments/assets/c8b63f91-ad53-41dd-8a17-2cef972f14e5)


Construire l'application en utilisant Maven pour créer l'artifact ROOT.war à partir du code source

![application_build](https://github.com/user-attachments/assets/00766202-447d-4525-a8bf-7fa0ba1950c1)


- 6. Configurer AWS CLI avec un compte ayant des accès sur S3 et uploader le fichier vprofile-v2.war

![s3](https://github.com/user-attachments/assets/c58b2187-058b-42e3-a2da-9677c270ba94)


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

![TG](https://github.com/user-attachments/assets/c6af0dd8-9928-4c4e-aef3-692ad38c7902)


ALB:

![ALB](https://github.com/user-attachments/assets/2af47973-6540-49fd-9870-b2c9396aeaef)



###  8. Ajouter un enregistrement de type CNAME qui mappe le nom de domaine vers le DNS de l'ALB dans Route 53
   ![CNAME_RECORDE](https://github.com/user-attachments/assets/98183536-6971-4e50-a98b-ae7ea27f8c08)

   
### 9. Créer une auto scaling group pour l'instance vprofile-app pour assurer la scalabilité et la disponibilité de l'application8. Ajouter un enregistrement de type CNAME qui mappe le nom de domaine vers le DNS de l'ALB dans Route 53

   

La première chose à faire est de créer une image AMI depuis notre instance vprofile-app.

![AMI_image](https://github.com/user-attachments/assets/0e08536c-bc7f-4408-9288-e758ee376a4b)


Ensuite, créer un launch template définissant l'image, les groupes de sécurité, les paires de clés et autres configurations à utiliser.

![lunch_template](https://github.com/user-attachments/assets/4e000817-01bc-4135-8e2b-99b07c1ba2ee)

Ensuite, nous avons créé un groupe d'auto-scaling dans lequel nous avons défini le nombre minimal et maximal d'instances à lancer, ainsi que le type de scaling sur lequel il sera basé.
![auto_scaling](https://github.com/user-attachments/assets/3bcb8289-72b5-4cea-945b-e76d54e5a797)

Ensuite, vérifiez le bon fonctionnement du site et notre configuration.
![vprofileapptest](https://github.com/user-attachments/assets/a6867846-b813-4b3f-8417-95ce5a710f74)


Fin
Merci
Yousssef MOU



