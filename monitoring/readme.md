# Prometheus y Grafana

[Guía general y requisitos](../README.md)

Este chart instala una plataforma de monitoreo y observabilidad compuesta por Prometheus, Grafana, Alertmanager, Loki y Alloy. Prometheus recopila métricas del clúster, Grafana permite visualizarlas, Loki almacena logs y Alloy los recolecta desde los pods de Kubernetes.

Loki usa una sola réplica y guarda sus logs mediante `filesystem` en el volumen persistente. MinIO está deshabilitado. La configuración no necesita un servicio S3 y corresponde al entorno local del curso. Si se requiere un despliegue con varias réplicas o alta disponibilidad, revisa el uso de almacenamiento de objetos.

En una instalación nueva se crea el volumen persistente de Loki y no se crean recursos de MinIO.

## Dependencias

| Chart | Versión |
| --- | --- |
| kube-prometheus-stack | 92.1.0 |
| loki | 7.3.0 |
| alloy | 1.13.0 |

## Despliegue de Prometheus y Grafana

### 1) Entrar al chart

```bash
cd monitoring
```

### 2) Agregar los repositorios oficiales (si no existen)

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

### 3) Descargar dependencias del chart

```bash
helm dependency build .
```

### 4) Instalar o actualizar

```bash
helm upgrade --install monitoring . -n monitoring --create-namespace
```

Este comando crea el namespace `monitoring` si no existe e instala o actualiza la release `monitoring` usando la configuración definida en `values.yaml`.

### 5) Verificar recursos

```bash
kubectl get pods -n monitoring
kubectl get ingress -n monitoring
```

## Credenciales iniciales de Grafana

El usuario predeterminado de Grafana es:

```text
admin
```

Para obtener la contraseña inicial:

```bash
kubectl get secret monitoring-grafana -n monitoring -o jsonpath="{.data.admin-password}" | base64 --decode && echo
```

## Comandos útiles

```bash
helm uninstall monitoring -n monitoring
helm get values monitoring -n monitoring
kubectl get prometheus -n monitoring
kubectl get alertmanager -n monitoring
```

## Configuración de Prometheus y Grafana

La configuración principal está en [values.yaml](values.yaml).

Algunos valores relevantes son:

- `kube-prometheus-stack.grafana.ingress.enabled: true`
- `kube-prometheus-stack.grafana.ingress.hosts` con `grafana.devops.cl`
- `kube-prometheus-stack.prometheus.ingress.enabled: true`
- `kube-prometheus-stack.prometheus.ingress.hosts` con `prometheus.devops.cl`
- `kube-prometheus-stack.alertmanager.ingress.hosts` con `alertmanager.devops.cl`
- Loki configurado en modo `SingleBinary` con una réplica.
- Loki usa almacenamiento `filesystem` en su volumen persistente de 10Gi.
- MinIO está deshabilitado.
- `loki.singleBinary.persistence.enableStatefulSetAutoDeletePVC: false`, para conservar el volumen al retirar el StatefulSet.
- Alloy configurado para recolectar los logs de los pods y enviarlos a Loki.
- Loki agregado como fuente de datos adicional en Grafana.

## Acceso local

Consulta las [URLs, dominios y configuración del archivo hosts](../README.md#dominios-y-archivo-hosts) de la guía general. Allí se explica también cómo eliminar políticas HSTS en Edge.


## Comprobar funcionamiento

Comprueba las interfaces de monitoreo:

```bash
curl -fsS http://grafana.devops.cl/api/health
curl -fsS http://prometheus.devops.cl/-/ready
curl -fsS http://alertmanager.devops.cl/-/ready
```

Grafana debe indicar `database: ok`; Prometheus y Alertmanager deben responder HTTP 200. En Grafana, usa la fuente de datos Loki en **Explore** y consulta `{namespace="monitoring"}` para comprobar que Alloy entrega logs.
