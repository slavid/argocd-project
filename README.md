# ArgoCD on Kubernetes (Minikube)

## Instalar e iniciar ArgoCD

Iniciar minikube (el servicio de Docker tiene que estar corriendo)
```
$ minikube start
```
(Optativo: `$ minikube start --driver docker` para asegurarse de usar docker)
Crear en namespace para argocd
```
$ kubectl create namespace argocd
```

Descargar y aplicar los manifestos para ArgoCD full (no la version core)
```
$ kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Verificar que todo está running:
```
$ kubectl get all -n argocd
```

Exponer el WebUI:
```
$ minikube service --namespace argocd argocd-server --url
```

Conseguir la contraseña del usuario admin:
```
$ kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

Si no se dispone de un decodificador base64 en la terminal, usar un decodificador base64 online, el comando entonces quedaría así:

```
$ kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | echo
```

Una vez hemos hecho login, cambiar la contraseña desde `User Info`.

## Configurar un repositorio

Settings -> Repositories -> Connect Repo:
```
Type: Git
Project: default
Repository URL: https://github.com/slavid/argocd-project.git
```

## Crear una aplicacion (en este ejemplo un servidor nginx)

Crear una carpeta: en mi caso he creado la carpeta `manifests`.\
Create application on the 3 dots on the right.

```
Application name:
Project name: default (escoger el creado en el paso anterior)
Sync Policy: Automatic
Repository URL: escoger la URL de github.
Path: el nombre de la carpeta donde están los manifests de esta aplicacion, en este caso: `manifests` es el nombre de la ruta.
Cluster URL: como estamos en local, escoger https://kubernetes.default.svc
Namespace: el namespace que hayas creado para esta aplicación.
```

Tras darle a create, si los datos introducidos son correctos y no hay problemas en la red deberias de ver la aplicación desplegada.

### Nginx manifest

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # tells deployment to run 2 pods matching the template
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

Tras guardarlo y subirlo a Github se debería de desplegar en Kubernetes y verlo de manera gráfica en ArgoCD. Cualquier cambio que pushees al repositorio actualizará el despliegue y habremos configurado un `pipeline de GitOps`.

Por ejemplo si cambiamos la linea de 

```yml
replicas: 2
```

a

```yml
replicas: 3
```

Se desplegará un pod más.
