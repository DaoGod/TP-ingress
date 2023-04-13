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
# burger/Dockerfile
FROM nginx

COPY index.html /usr/local/apache2/htdocs/
```

```html
<html>
<p>Burger</p>
</html>
```

Le contenu des fichiers index.html est à modifier en fonction du nom du site web : mypizza, burger et tacos

Voici les commandes pour build les images correspondantes :

```bash
PS D:\Documents\Docker_Ynov_2023\TP-ingress\src\burger> docker build -t nicolassss/burger.eatsout.com .
[+] Building 9.9s (7/7) FINISHED
```        
```bash
PS D:\Documents\Docker_Ynov_2023\TP-ingress\src\tacos> docker build -t nicolassss/tacos.eatsout.com .
[+] Building 1.1s (7/7) FINISHED
```        
```bash
PS D:\Documents\Docker_Ynov_2023\TP-ingress\src\mypizza> docker build -t nicolassss/mypizza.eatsout.com . 
[+] Building 0.6s (7/7) FINISHED
```        

On peut vérifier que les images ont bien été créés :
```bash
PS D:\Documents\Docker_Ynov_2023\TP-ingress> docker images
REPOSITORY                                 TAG       IMAGE ID       CREATED          SIZE  
nicolassss/mypizza.eatsout.com                        latest    6399fb9fa83b   11 minutes ago   142MB 
nicolassss/tacos.eatsout.com                          latest    109a57897283   12 minutes ago   142MB 
nicolassss/burger.eatsout.com                         latest    8d481f4c8c22   14 minutes ago   142MB 
```

J'ai renseigné mon nom d'utilisateur DockerHub dans le nom de mon image afin que celle-ci puisse être publié sur DockerHub. Ici c'est nicolassss.

Je me connecte ensuite à mon compte DockerHub :

 ```bash
 PS D:\Documents\Docker_Ynov_2023\TP-ingress\src\tacos> docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: nicolassss
Password:
Login Succeeded
```

Puis je push les images sur un repository DockerHub :

```bash
PS D:\Documents\Docker_Ynov_2023\TP-ingress\src\tacos> docker push nicolassss/tacos.eatsout.com
Using default tag: latest
9d907f11dc74: Mounted from library/nginx
79974a1a12aa: Mounted from library/nginx
f12d4345b7f3: Mounted from library/nginx
935b5bd454e1: Mounted from library/nginx
fb6d57d46ad5: Mounted from library/nginx
ed7b0ef3bf5b: Mounted from library/nginx
latest: digest: sha256:98d0e11ceac884f50434d2e0ae543ab890f04e8811af960e43d2651d94dece72 size: 1777
```

J'ai exécuté cette commande avec mes trois images. 

![docker images](images\dockerhub.png)