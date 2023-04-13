# TP Kubernetes Ingress

## Author

- [@Enzo Pistre](https://github.com/DaoGod)
- [@Nicolas Seillé](https://github.com/Nicolas-3050)


## 1. Installer Kind et créer votre premier cluster. Suivre cette documentation : https://kind.sigs.k8s.io/docs/user/quick-start/

L'installation de Kind :

```bash
choco install kind
```

Création du cluster

```bash
Kind create cluster
```

## 2. Installer le Nginx ingress Controller. Suivre cette documentation : https://kind.sigs.k8s.io/docs/user/ingress/#ingress-nginx

Installation de Nginx ingress Controller

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

## 3. Compléter le schéma suivant avec des objets Kubernetes

Voici le schéma compléter:

![docker images](images\Tp_ingress.png)

## 4. Builder et publier (à partir de l’image nginx) sur le DockerHub, une image docker pour chacun des sites web présent sur le schéma précédent

On fait trois Dockerfile pour chacune des images que l'on veut créer ainsi que trois fichiers index.html :

```yaml
# burgeandacos/Dockerfile
FROM nginx

COPY index.html /usr/local/apache2/htdocs/
```

```html
<html>
<p>Burger and Tacos</p>
</html>
```

Le contenu des fichiers index.html est à modifier en fonction du nom du site web : mypizza, burgerandtacos (.com/.tacos)

Voici les commandes pour build les images correspondantes :

```bash
PS D:\Documents\Docker_Ynov_2023\TP-ingress\src\burgerandtacos> docker build -t burgerandtacos.com .
[+] Building 9.9s (7/7) FINISHED
```        
```bash
PS D:\Documents\Docker_Ynov_2023\TP-ingress\src\burgerandtacos.tacos> docker build -t burgerandtacos.tacos .
[+] Building 1.1s (7/7) FINISHED
```        
```bash
PS D:\Documents\Docker_Ynov_2023\TP-ingress\src\mypizza> docker build -t mypizza.eatsout.com . 
[+] Building 0.6s (7/7) FINISHED
```        

On peut vérifier que les images ont bien été créés :
```bash
PS D:\Documents\Docker_Ynov_2023\TP-ingress> docker images
REPOSITORY                                 TAG       IMAGE ID       CREATED          SIZE  
mypizza.eatsout.com                        latest    6399fb9fa83b   11 minutes ago   142MB 
burgerandtacos.tacos                       latest    109a57897283   12 minutes ago   142MB 
burgerandtacos.com                         latest    8d481f4c8c22   14 minutes ago   142MB 
```