✅ Pasos para levantar y validar el entorno

1. 🧰 Instalación de Kind y kubectl

Kind (Kubernetes in Docker):
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.22.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

kubectl (CLI de Kubernetes):
curl -LO "https://dl.k8s.io/release/$(curl -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

Asegúrate de tener Docker en ejecución.

2. 🚀 Cómo levantar el entorno local con Kind

- Crear el clúster:
kind create cluster --name devops-challenge

- Verificar acceso al clúster:
kubectl cluster-info --context kind-devops-challenge

- Crear el namespace:
kubectl create namespace devops-challenge

- Crear el Secret base64:
kubectl apply -f k8s/secret.yaml

- Crear el Secret de autenticación al GHCR (opcional):
kubectl create secret docker-registry ghcr-secret   --docker-server=ghcr.io   --docker-username=<TU_USUARIO>   --docker-password=<TU_TOKEN>   --namespace=devops-challenge

- Aplicar los recursos Kubernetes:
kubectl apply -f k8s/

3. 🤖 Cómo ejecutar la pipeline

Opción A – GitHub Actions (recomendado):
Solo haz push a la rama main y GitHub ejecutará automáticamente el flujo CI/CD descrito en .github/workflows/deploy.yml.

Opción B – Manual (localmente en Kind):
docker build -t node-app:local .
kind load docker-image node-app:local --name devops-challenge

Luego edita el deployment:
kubectl edit deployment app -n devops-challenge
Cambia la imagen a: image: node-app:local

4. 🔍 Comandos de validación

- Verificar estado de pods:
kubectl get pods -n devops-challenge

- Verificar logs del contenedor:
kubectl logs -n devops-challenge deploy/app

- Probar el endpoint /health:
kubectl port-forward svc/app-service 8080:80 -n devops-challenge

Abre en tu navegador:
http://localhost:8080/health

Deberías ver:
{ "status": "ok" }
