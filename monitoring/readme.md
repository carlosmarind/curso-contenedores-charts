# Prometheus y Grafana

Este chart instala una plataforma de monitoreo y observabilidad compuesta por Prometheus, Grafana, Alertmanager, Loki y Alloy. Prometheus recopila métricas del clúster, Grafana permite visualizarlas, Loki almacena logs y Alloy los recolecta desde los pods de Kubernetes.

Loki usa una sola réplica y guarda sus logs mediante `filesystem` en el volumen persistente. MinIO está deshabilitado porque sus imágenes no pudieron descargarse durante la validación local. La configuración no necesita un servicio S3 y corresponde al entorno local del curso. Si se requiere un despliegue con varias réplicas o alta disponibilidad, revisa el uso de almacenamiento de objetos.

En una instalación nueva se crea el volumen persistente de Loki y no se crean recursos de MinIO.

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
helm dependency build .
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

### Acceso local por dominio

El archivo `values.yaml` configura ingresos para las interfaces web de monitoreo. Una vez desplegadas, quedarán disponibles en:

```text
http://grafana.devops.cl
http://prometheus.devops.cl
http://alertmanager.devops.cl
```

Para acceder desde el navegador, registra estos dominios en el archivo `hosts` de tu sistema operativo.

Primero identifica la IP o dirección de entrada de los ingress:

```bash
kubectl get ingress -n monitoring
```

Si estás trabajando en un entorno local, normalmente bastará con entradas como estas:

```text
127.0.0.1 grafana.devops.cl
127.0.0.1 prometheus.devops.cl
127.0.0.1 alertmanager.devops.cl
```

En Linux y macOS, edita `/etc/hosts`.

En Windows, edita `C:\Windows\System32\drivers\etc\hosts` con permisos de administrador.
