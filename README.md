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

J'ai créé quatres fichiers yaml.

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
  replicas: 3
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

Le contenu des fichiers mypizza.yaml et tacos.yaml est très similaire.

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

On peut vérifier que les pods, les deployments et les services ont bien été créé avec ces commandes :

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
burger-deployment-6fd5dcc984-gjhd5    1/1     Running   0          5m40s
burger-deployment-6fd5dcc984-pcsd8    1/1     Running   0          5m40s
mypizza-deployment-86dff76457-7bvct   1/1     Running   0          4m36s
mypizza-deployment-86dff76457-jwsjp   1/1     Running   0          4m36s
mypizza-deployment-86dff76457-qlw2s   1/1     Running   0          4m36s
tacos-deployment-68957776f9-6l8vk     1/1     Running   0          6m19s
tacos-deployment-68957776f9-h7j4v     1/1     Running   0          6m19s
tacos-deployment-68957776f9-qgwjj     1/1     Running   0          6m19s
```

On peut faire cette commande pour exposer temporairement le service mypizza-service sur notre machine au port 8080:80 :
kubectl port-forward svc/mypizza-service 8080:80

On peut y accéder maintenant depuis : http://localhost:8080/mypizza

Pour que cela fonctionne, il faut aussi penser à définir les routes dans le fichier Windows/System32/drivers/etc/hosts :
127.0.0.1 mypizza.eatsout.com
127.0.0.1 burgerandtacos.eatsout.com
127.0.0.1 burgerandtacos.eatsout.tacos