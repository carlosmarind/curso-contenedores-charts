# Metrics Server

[Guía general y requisitos](../README.md)

Este chart instala Metrics Server para consultar el uso de CPU y memoria de nodos y pods.

## Dependencias

| Chart | Versión |
| --- | --- |
| metrics-server | 7.4.12 |

## Despliegue de Metrics Server

### 1) Entrar al chart

```bash
cd metrics-server
```

### 2) Agregar el repositorio de Bitnami (si no existe)

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

### 3) Descargar dependencias del chart

```bash
helm dependency build .
```

### 4) Instalar o actualizar

```bash
helm upgrade --install metrics-server . -n kube-system --create-namespace
```

Este comando instala o actualiza la release `metrics-server` en el namespace `kube-system` usando la configuración definida en `values.yaml`.

### 5) Verificar recursos

```bash
kubectl get pods -n kube-system
kubectl get apiservice v1beta1.metrics.k8s.io
```

## Comandos útiles

```bash
helm uninstall metrics-server -n kube-system
helm get values metrics-server -n kube-system
```

## Configuración de Metrics Server

La configuración principal está en [values.yaml](values.yaml).

Algunos valores relevantes son:

- `metrics-server.image.registry: registry.k8s.io`
- `metrics-server.image.repository: metrics-server/metrics-server`
- `metrics-server.image.tag: v0.9.0`
- `metrics-server.command: /metrics-server`
- `metrics-server.global.security.allowInsecureImages: true`, necesario porque el chart de Bitnami valida sus propias imágenes y bloquea imágenes externas aunque sean oficiales.
- `metrics-server.args` con `--kubelet-insecure-tls`
- `metrics-server.args` con `--kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname`
- `metrics-server.args` con `--kubelet-use-node-status-port`
- `metrics-server.args` con `--secure-port=10250`
- `metrics-server.containerPorts.https: 10250`, para alinear el puerto del contenedor, el Service y las sondas.
- `metrics-server.args` con `--metric-resolution=15s`
- `metrics-server.apiService.create: true`
- `metrics-server.resources.requests.memory: 200Mi`
- `metrics-server.resources.requests.cpu: 100m`

Este chart usa la dependencia de Bitnami y la imagen oficial de Metrics Server de Kubernetes configurada en `values.yaml`.

## Comprobar funcionamiento

Comprueba que se muestran valores de CPU y memoria de los nodos y pods:

```bash
kubectl top nodes
kubectl top pods -A
```
