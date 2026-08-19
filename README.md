# Repositorio de charts

Este repositorio reúne los charts Helm usados en el curso de contenedores.

## Objetivo

Centralizar configuraciones de despliegue sobre Kubernetes usando Helm. Actualmente, el repositorio incluye charts para Ingress NGINX, Metrics Server, Jenkins y monitoreo con Prometheus, Grafana, Loki y Alloy a partir de dependencias oficiales.

## Helm

Helm es el gestor de paquetes de Kubernetes. Permite instalar, actualizar y mantener aplicaciones a partir de charts.

### Instalación de Helm

Si necesitas instalar Helm en Linux, puedes usar:

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-4 | bash
```

Una vez instalado, Helm se usa en este repositorio para gestionar los charts incluidos.

## Requisitos

- Un clúster de Kubernetes en funcionamiento.
- `kubectl` configurado contra ese clúster.
- Helm instalado.

## Estructura

```
.
├── README.md
├── ingress-nginx/
│   ├── Chart.yaml
│   ├── Chart.lock
│   ├── values.yaml
│   └── charts/
├── metrics-server/
│   ├── Chart.yaml
│   ├── Chart.lock
│   ├── values.yaml
│   └── charts/
├── jenkins/
│   ├── Chart.yaml
│   ├── Chart.lock
│   ├── values.yaml
│   └── charts/
└── monitoring/
    ├── Chart.yaml
    ├── Chart.lock
    ├── values.yaml
    └── charts/
```

## Orden de instalación

Los charts deben instalarse en este orden:

1. Ingress NGINX.
2. Metrics Server.
3. Jenkins.
4. Prometheus y Grafana.

Ingress NGINX debe instalarse primero porque Jenkins, Grafana, Prometheus y Alertmanager usan `ingressClassName: nginx` para exponer sus interfaces web.

## Ingress NGINX

### Despliegue de Ingress NGINX

#### 1) Entrar al chart

```bash
cd ingress-nginx
```

#### 2) Agregar el repositorio oficial (si no existe)

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
```

#### 3) Descargar dependencias del chart

```bash
helm dependency update .
```

#### 4) Instalar o actualizar

```bash
helm upgrade --install ingress-nginx . -n ingress-nginx --create-namespace
```

Este comando instala o actualiza la release `ingress-nginx` en el namespace `ingress-nginx` usando la configuración definida en `values.yaml`.

#### 5) Verificar recursos

```bash
kubectl get pods -n ingress-nginx
kubectl get svc -n ingress-nginx
kubectl get ingressclass
```

### Comandos útiles

```bash
helm uninstall ingress-nginx -n ingress-nginx
helm get values ingress-nginx -n ingress-nginx
kubectl describe ingressclass nginx
```

### Configuración de Ingress NGINX

La configuración principal del chart está en `ingress-nginx/values.yaml`.

Algunos valores relevantes del estado actual son:

- `ingress-nginx.controller.ingressClass: nginx`
- `ingress-nginx.controller.ingressClassResource.name: nginx`
- `ingress-nginx.controller.ingressClassResource.default: true`
- `ingress-nginx.controller.service.type: LoadBalancer`

## Metrics Server

### Despliegue de Metrics Server

#### 1) Entrar al chart

```bash
cd metrics-server
```

#### 2) Agregar el repositorio de Bitnami (si no existe)

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

#### 3) Descargar dependencias del chart

```bash
helm dependency update .
```

#### 4) Instalar o actualizar

```bash
helm upgrade --install metrics-server . -n kube-system --create-namespace
```

Este comando instala o actualiza la release `metrics-server` en el namespace `kube-system` usando la configuración definida en `values.yaml`.

#### 5) Verificar recursos

```bash
kubectl get pods -n kube-system
kubectl get apiservice v1beta1.metrics.k8s.io
```

### Comandos útiles

```bash
helm uninstall metrics-server -n kube-system
helm get values metrics-server -n kube-system
kubectl top nodes
kubectl top pods -A
```

### Configuración de Metrics Server

La configuración principal del chart está en `metrics-server/values.yaml`.

Algunos valores relevantes del estado actual son:

- `metrics-server.image.registry: registry.k8s.io`
- `metrics-server.image.repository: metrics-server/metrics-server`
- `metrics-server.image.tag: v0.8.0`
- `metrics-server.command: /metrics-server`
- `metrics-server.global.security.allowInsecureImages: true`, necesario porque el chart de Bitnami valida sus propias imágenes y bloquea imágenes externas aunque sean oficiales.
- `metrics-server.args` con `--kubelet-insecure-tls`
- `metrics-server.args` con `--kubelet-preferred-address-types=InternalIP`
- `metrics-server.args` con `--metric-resolution=15s`
- `metrics-server.apiService.create: true`
- `metrics-server.resources.requests.memory: 64Mi`
- `metrics-server.resources.requests.cpu: 100m`

Este chart usa la dependencia de Bitnami, pero reemplaza la imagen por la imagen oficial de Kubernetes para evitar fallas de descarga con la imagen `docker.io/bitnami/metrics-server:0.8.0-debian-12-r4`.

## Jenkins

### Despliegue de Jenkins

#### 1) Entrar al chart

```bash
cd jenkins
```

#### 2) Agregar repositorio de Jenkins (si no existe)

```bash
helm repo add jenkins https://charts.jenkins.io
helm repo update
```

#### 3) Descargar dependencias del chart

```bash
helm dependency update .
```

#### 4) Instalar o actualizar

```bash
helm upgrade --install jenkins . -n jenkins --create-namespace
```

Este comando crea el namespace `jenkins` si no existe e instala o actualiza la release `jenkins` usando la configuración definida en `values.yaml`.

#### 5) Verificar recursos

```bash
kubectl get pods -n jenkins
kubectl get ingress -n jenkins
```

### Credenciales iniciales

El usuario inicial es `admin`. Para obtener la contraseña:

```bash
kubectl exec --namespace jenkins -it svc/jenkins -c jenkins -- /bin/cat /run/secrets/additional/chart-admin-password && echo
```

### Comandos útiles

```bash
helm uninstall jenkins -n jenkins
helm get values jenkins -n jenkins
```

### Configuración de Jenkins

La configuración principal del chart está en `jenkins/values.yaml`.

Algunos valores relevantes del estado actual son:

- `jenkins.controller.ingress.enabled: true`
- `jenkins.controller.ingress.hostName: jenkins.local`
- Lista de plugins preconfigurada en `jenkins.controller.installPlugins`

### Acceso local por dominio

El archivo `values.yaml` configura el ingreso con el dominio `jenkins.local`.

Una vez desplegado, Jenkins quedará disponible en:

```text
http://jenkins.local
```

Para acceder desde el navegador, registra ese dominio en el archivo `hosts` de tu sistema operativo.

Primero identifica la IP o dirección de entrada del ingress:

```bash
kubectl get ingress -n jenkins
```

Si estás trabajando en un entorno local, normalmente bastará con una entrada como esta:

```text
127.0.0.1 jenkins.local
```

En Linux y macOS, edita `/etc/hosts`.

En Windows, edita `C:\Windows\System32\drivers\etc\hosts` con permisos de administrador.

## Prometheus y Grafana

Este chart instala una plataforma de monitoreo y observabilidad compuesta por Prometheus, Grafana, Alertmanager, Loki y Alloy. Prometheus recopila métricas del clúster, Grafana permite visualizarlas, Loki almacena logs y Alloy los recolecta desde los pods de Kubernetes.

### Despliegue de Prometheus y Grafana

#### 1) Entrar al chart

```bash
cd monitoring
```

#### 2) Agregar los repositorios oficiales (si no existen)

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

#### 3) Descargar dependencias del chart

```bash
helm dependency update .
```

#### 4) Instalar o actualizar

```bash
helm upgrade --install monitoring . -n monitoring --create-namespace
```

Este comando crea el namespace `monitoring` si no existe e instala o actualiza la release `monitoring` usando la configuración definida en `values.yaml`.

#### 5) Verificar recursos

```bash
kubectl get pods -n monitoring
kubectl get ingress -n monitoring
```

### Credenciales iniciales de Grafana

El usuario predeterminado de Grafana es:

```text
admin
```

Para obtener la contraseña inicial:

```bash
kubectl get secret monitoring-grafana -n monitoring -o jsonpath="{.data.admin-password}" | base64 --decode && echo
```

### Comandos útiles

```bash
helm uninstall monitoring -n monitoring
helm get values monitoring -n monitoring
kubectl get prometheus -n monitoring
kubectl get alertmanager -n monitoring
```

### Configuración de Prometheus y Grafana

La configuración principal del chart está en `monitoring/values.yaml`.

Algunos valores relevantes del estado actual son:

- `kube-prometheus-stack.grafana.ingress.enabled: true`
- `kube-prometheus-stack.grafana.ingress.hosts` con `grafana.local`
- `kube-prometheus-stack.prometheus.ingress.enabled: true`
- `kube-prometheus-stack.prometheus.ingress.hosts` con `prometheus.local`
- `kube-prometheus-stack.alertmanager.ingress.hosts` con `alertmanager.local`
- Loki configurado en modo `SingleBinary` con una réplica.
- MinIO habilitado como almacenamiento de objetos para Loki.
- Alloy configurado para recolectar los logs de los pods y enviarlos a Loki.
- Loki agregado como fuente de datos adicional en Grafana.

### Acceso local por dominio

El archivo `values.yaml` configura ingresos para las interfaces web de monitoreo. Una vez desplegadas, quedarán disponibles en:

```text
http://grafana.local
http://prometheus.local
http://alertmanager.local
```

Para acceder desde el navegador, registra estos dominios en el archivo `hosts` de tu sistema operativo.

Primero identifica la IP o dirección de entrada de los ingress:

```bash
kubectl get ingress -n monitoring
```

Si estás trabajando en un entorno local, normalmente bastará con entradas como estas:

```text
127.0.0.1 grafana.local
127.0.0.1 prometheus.local
127.0.0.1 alertmanager.local
```

En Linux y macOS, edita `/etc/hosts`.

En Windows, edita `C:\Windows\System32\drivers\etc\hosts` con permisos de administrador.
