# ArgoCD on Kubernetes (Minikube)

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

