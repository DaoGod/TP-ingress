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

## 5. Ecrire les fichiers yaml vous permettant de déployer sur votre cluster kind installé en local les composants décrits sur le schéma de la question 3 et les images crées à la question 4

J'ai créé quatres fichiers yaml. Il y en a un pour chaque services et un pour ingress.

burger.yaml :
```yaml
apiVersion: v1
kind: Service
metadata:
  name: burger-service
spec:
  selector:
    app: burger
  ports:
    - name: http
      port: 80
      targetPort: 80

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: burger-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: burger
  template:
    metadata:
      labels:
        app: burger
    spec:
      containers:
        - name: burger
          image: nicolassss/burger.eatsout.com
          ports:
            - containerPort: 80
```
On spécifie dans kind que l'on veut créée un service avec le nom burger-service 
Les pods sélectionnés dans le service sont ceux avec le label app=burger
Le service est accessible sur le port 80 et il dirige vers les pods avec le label indiqué.

On veut aussi créer un deployment (indiqué dans kind) avec le nom burger-deployment
Comme indiqué dans le schéma, le replicaset est à 1 pour mypizza et burger, et à 3 pour tacos.
Il y aura donc ici 1 pod avec le label burger de créée au moment du deploiement

On indique que le conteneur du pod utilise l'image nicolassss/burger.eatsout.com, créée précédement. Il est exposé sur le port 80.

Le contenu des fichiers mypizza.yaml et tacos.yaml est très similaire. Les changements sont le nom des services, des pods, des déploiements et le replica qui est à 3 pour tacos.

J'ai ensuite créé un fichier ingress.yaml :

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: eatsout-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: mypizza.eatsout.com
    http:
      paths:
      - path: /mypizza
        pathType: Prefix
        backend:
          service:
            name: mypizza-service
            port:
              number: 80
  - host: tacos.eatsout.com
    http:
      paths:
      - path: /tacos
        pathType: Prefix
        backend:
          service:
            name: tacos-service
            port:
              number: 80
  - host: burger.eatsout.com
    http:
      paths:
      - path: /burger
        pathType: Prefix
        backend:
          service:
            name: burger-service
            port:
              number: 80
```

Le fichier Ingress.yaml permet de faire fonctionner et de définir les ressource du cluster Kubernetes.
Il permet d'exposer nos services définis précédement à l'extérieur du cluster.

Les requêtes de mypizza.eatsout.com/mypizza, tacos.eatsout.com/tacos et burger.eatsout.com/burger sont dirigées vers les services mypizza-service, tacos-service et burger-service sur le port 80. Le path correspond au chemin d'accès de la requête dans l'URL.

Ici, si une requête arrive avec un chemin d'accès commençant par /burger, elle doit être routée vers le service burger-service sur le port 80.

On doit ensuite déployer nos différents services : 

```bash
PS D:\Documents\Docker_Ynov_2023\TP-ingress\src\tacos> kubectl apply -f tacos.yaml
service/tacos-service created
deployment.apps/tacos-deployment created
```

```bash
PS D:\Documents\Docker_Ynov_2023\TP-ingress\src\burger> kubectl apply -f burger.yaml
service/burger-service created
deployment.apps/burger-deployment created
```

```bash
PS D:\Documents\Docker_Ynov_2023\TP-ingress\src\mypizza> kubectl apply -f mypizza.yaml
service/mypizza-service created
deployment.apps/mypizza-deployment created
```

```bash
PS D:\Documents\Docker_Ynov_2023\TP-ingress\src> kubectl apply -f ingress.yaml
ingress.networking.k8s.io/eatsout-ingress created
```

On peut vérifier que les pods, les deployments et les services ont bien été créée avec ces commandes :

```bash
PS D:\Documents\Docker_Ynov_2023\TP-ingress\src> kubectl get service
NAME              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
burger-service    ClusterIP   10.111.202.248   <none>        80/TCP    4m9s
kubernetes        ClusterIP   10.96.0.1        <none>        443/TCP   27d
mypizza-service   ClusterIP   10.106.110.42    <none>        80/TCP    3m6s
tacos-service     ClusterIP   10.100.119.116   <none>        80/TCP    4m48s
```

```bash
PS D:\Documents\Docker_Ynov_2023\TP-ingress\src> kubectl get deployments
NAME                 READY   UP-TO-DATE   AVAILABLE   AGE
burger-deployment    3/3     3            3           5m28s
mypizza-deployment   3/3     3            3           4m25s
tacos-deployment     3/3     3            3           6m7s
```bash

```bash
PS D:\Documents\Docker_Ynov_2023\TP-ingress\src> kubectl get pod        
NAME                                  READY   STATUS    RESTARTS   AGE
burger-deployment-6fd5dcc984-fltl6    1/1     Running   0          5m40s
mypizza-deployment-86dff76457-7bvct   1/1     Running   0          4m36s
tacos-deployment-68957776f9-6l8vk     1/1     Running   0          6m19s
tacos-deployment-68957776f9-h7j4v     1/1     Running   0          6m19s
tacos-deployment-68957776f9-qgwjj     1/1     Running   0          6m19s
```

J'ai aussi créé des fichiers default.conf pour permettre au serveur de traiter les requêtes pour /mypizza, /tacos, /burger (chaque service).
Le serveur doit écouter sur le port 80.

```bash
server {
    listen       80;
    server_name  localhost;

    location /mypizza {
        alias   /usr/share/nginx/html/mypizza/;
        index  index.html;
        try_files $uri $uri/ /index.html;
    }

    location /tacos {
        alias   /usr/share/nginx/html/tacos/;
        index  index.html;
        try_files $uri $uri/ /index.html;
    }

    location /burger {
        alias   /usr/share/nginx/html/burger/;
        index  index.html;
        try_files $uri $uri/ /index.html;
    }
}
```

Le fichier default.conf est copié dans le Dockerfile.
J'ai aussi modifié le dossier où est copié index.html (/usr/local/apache2/htdocs/), car Nginx s'attend à ce qu'il soit dans /usr/share/nginx/html/

On peut maintenant faire cette commande pour exposer temporairement le service mypizza-service sur notre machine au port 8080:80 :
kubectl port-forward svc/mypizza-service 8080:80

On peut y accéder maintenant depuis : http://localhost:8080/mypizza

Pour que cela fonctionne, on peut aussi définir les routes dans le fichier Windows/System32/drivers/etc/hosts :
127.0.0.1 mypizza.eatsout.com
127.0.0.1 burger.eatsout.com
127.0.0.1 tacos.eatsout.com

## 6. Il va vous falloir gérer une charge importante sur le Service de commande des tacos (3 fois plus de commandes). Comment gérer cela ? Comment vérifier que les requêtes sont bien réparties (avec quelle commande kubectl ?) ?

Pour gérer cette charge de commande plus importante dans notre service Tacos, nous pouvons augmenter le nombre de replica de pods dans le deploiement. Cela permettre de répartir la charge entre plusieurs instance du service tacos. D'après le schéma définis lors de la question 3, le replica était à 3, nous pouvons l'augmenter à 9. 

La commande kubectl get pods, permet de vérifier que le nombre de pods voulu a bien été créé :

```bash
PS D:\Documents\Docker_Ynov_2023\TP-ingress\src> kubectl get pods -l app=tacos
NAME                                  READY   STATUS    RESTARTS   AGE
tacos-deployment-68957776f9-5jmqv     1/1     Running   0          2m16s
tacos-deployment-68957776f9-bszv8     1/1     Running   0          2m16s
tacos-deployment-68957776f9-k5vnw     1/1     Running   0          2m16s
tacos-deployment-68957776f9-ks5w4     1/1     Running   0          2m16s
tacos-deployment-68957776f9-ktdgg     1/1     Running   0          2m16s
tacos-deployment-68957776f9-lpx8b     1/1     Running   0          2m16s
tacos-deployment-68957776f9-mkwm6     1/1     Running   0          2m16s
tacos-deployment-68957776f9-wgf4k     1/1     Running   0          2m16s
```

On peut utiliser la commande kubectl logs <nom du pod> pour voir le nombre de requêtes traitée par le pod. Si la charge est équitablement répartie entre les pods, le nombre de requêtes traitées par pods devrait être à peu près égale. 

Exemple : 
```bash
kubectl logs tacos-deployment-68957776f9-k5vnw
```

Si les requêtes http n'apparaissent pas dans les logs des pods, on peut configurer le serveur Nginx pour enregistrer les requêtes en rajoutant la ligne 
access_log /var/log/nginx/access.log; dans les fichiers default.conf.