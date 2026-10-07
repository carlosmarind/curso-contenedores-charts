# Repositorio de charts

Este repositorio reúne los charts Helm usados en el curso de contenedores.

## Objetivo

Centralizar configuraciones de despliegue sobre Kubernetes usando Helm. El repositorio incluye Ingress NGINX, Metrics Server, Jenkins, monitoreo con Prometheus, Grafana, Loki y Alloy, Argo CD, Harbor, CloudNativePG, SonarQube y cert-manager a partir de dependencias oficiales.

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
- Una StorageClass predeterminada con aprovisionamiento de volúmenes para los charts con persistencia.
- Recursos de CPU, memoria y almacenamiento suficientes; los ejemplos no requieren instalar todos los servicios a la vez.
- Bash para los pasos que solicitan contraseñas con `read -s`.

## Estructura

```text
.
├── README.md
├── ingress-nginx/
├── metrics-server/
├── jenkins/
├── monitoring/
├── argocd/
├── harbor/
├── cloudnative-pg/
│   └── postgres/
│       ├── cluster.yaml
│       └── db-sonar.yaml
├── sonarqube/
└── cert-manager/
```

Cada chart contiene `Chart.yaml`, `Chart.lock`, `values.yaml` y las dependencias empaquetadas en `charts/`. Los ejemplos de `cloudnative-pg/postgres/` se aplican por separado. Las instrucciones particulares están también en el `readme.md` de los charts nuevos y de monitoreo.

Argo CD, Harbor, CloudNativePG, SonarQube y cert-manager se incorporaron desde [carlosmarind/helm-charts](https://github.com/carlosmarind/helm-charts/tree/434d6aa063a1fda2cd99bd458836539b83cf1148), conservando sus versiones y adaptando las configuraciones al curso.

## Orden de instalación

1. Ingress NGINX, antes de los servicios con ingreso web.
2. Metrics Server, para las métricas de recursos.
3. Jenkins y monitoreo, según los temas del curso.
4. Argo CD, Harbor y SonarQube, según los temas del curso.
5. CloudNativePG, antes de crear los clústeres PostgreSQL de ejemplo.
6. cert-manager, cuando se trabaje con TLS y antes de crear emisores o certificados.

No es obligatorio instalar todos los charts. Harbor y SonarQube usan sus propias bases de datos; no requieren CloudNativePG. Los servicios web usan HTTP y subdominios de `devops.cl`, sin depender de cert-manager.

## Dominios y archivo hosts

Antes de acceder a los servicios, agrega estos nombres al archivo `hosts` del equipo desde el que usarás el navegador. Todas las entradas deben apuntar a la IP de entrada de Ingress NGINX.

En Docker Desktop, si el ingress está expuesto en tu equipo, usa:

```text
127.0.0.1 jenkins.devops.cl grafana.devops.cl prometheus.devops.cl alertmanager.devops.cl argocd.devops.cl harbor.devops.cl sonar.devops.cl
```

Si el ingress usa otra IP, reemplaza `127.0.0.1` por esa dirección. Puedes consultar su entrada con:

```bash
kubectl get svc ingress-nginx-controller -n ingress-nginx
```

En Linux y macOS, edita `/etc/hosts`. En Windows, edita `C:\Windows\System32\drivers\etc\hosts` con permisos de administrador. Reemplaza las entradas anteriores de los servicios del curso por estos nombres. El archivo `hosts` no admite comodines: registra cada subdominio; una entrada para `devops.cl` no cubre sus subdominios.

Los nombres internos de Kubernetes, como `*.svc.cluster.local`, se conservan. Si habilitas Thanos Ruler, su dominio configurado es `thanos.devops.cl` y también debes agregarlo al archivo `hosts`.

### URLs de acceso

Los servicios están configurados para HTTP. Después de guardar el archivo `hosts`, abre estas URLs:

| Servicio | URL |
| --- | --- |
| Jenkins | http://jenkins.devops.cl |
| Grafana | http://grafana.devops.cl |
| Prometheus | http://prometheus.devops.cl |
| Alertmanager | http://alertmanager.devops.cl |
| Argo CD | http://argocd.devops.cl |
| Harbor | http://harbor.devops.cl |
| SonarQube | http://sonar.devops.cl |

### Eliminar una política HSTS en Microsoft Edge

Si Edge recuerda una política HSTS para un dominio, puede cambiar automáticamente una URL de HTTP a HTTPS. Para eliminar la política dinámica guardada durante pruebas anteriores:

1. Abre `edge://net-internals/#hsts` en la barra de direcciones de Edge.
2. En **Query HSTS/PKP domain**, escribe el dominio afectado, por ejemplo `jenkins.devops.cl`, sin `http://`, puerto ni ruta, y pulsa **Query**.
3. En **Delete domain security policies**, escribe ese mismo dominio y pulsa **Delete**.
4. Si la política se hereda de `devops.cl` mediante `includeSubDomains`, consulta y elimina también la política dinámica de `devops.cl`.
5. Repite la consulta para comprobar el resultado y vuelve a abrir la URL explícitamente con `http://`.

La eliminación afecta al navegador de ese equipo. No elimina políticas precargadas o estáticas; si el servidor vuelve a enviar HSTS por HTTPS, Edge puede guardar nuevamente la política. Si no hay una política HSTS y continúa el cambio a HTTPS, revisa la opción de HTTPS automático del navegador y posibles redirecciones del servidor.

## Uso de las instrucciones

Cada sección parte desde la raíz de este repositorio. Si terminaste dentro de un chart, vuelve con `cd ..` antes de comenzar la siguiente sección.

`helm dependency build .` reconstruye las dependencias con las versiones del `Chart.lock`. Si ya están empaquetadas en `charts/`, puedes instalar directamente. Usa `helm dependency update .` cuando quieras recalcular las dependencias y actualizar el lock después de cambiar `Chart.yaml`.

## Validación local de los charts

Desde la raíz, puedes comprobar los charts sin desplegarlos:

```bash
(
  for chart in ingress-nginx metrics-server jenkins monitoring argocd harbor cloudnative-pg sonarqube cert-manager; do
    helm lint "$chart" || exit 1
    helm template "$chart" "./$chart" > /dev/null || exit 1
  done
)
```

`helm template` renderiza los manifiestos localmente. No comprueba la existencia de Secrets, el almacenamiento disponible, la descarga de imágenes ni la compatibilidad de las API con tu clúster. La comprobación de funcionamiento se realiza después de desplegar cada servicio.

## Validación en el clúster local

El 7 de octubre de 2026 se reinstalaron los nueve charts desde un clúster vacío en el contexto `clase-contenedores`, con Kubernetes 1.36.4, cuatro nodos y StorageClass predeterminada `standard` (`rancher.io/local-path`). Todas las releases quedaron `deployed`, los pods disponibles y los volúmenes persistentes `Bound`.

Se comprobaron las rutas HTTP de los siete subdominios de `devops.cl` a través de Ingress NGINX, publicado en `127.0.0.1:80`. Para este entorno, las entradas del archivo `hosts` indicadas arriba pueden usar `127.0.0.1`.

| Chart | Resultado observado |
| --- | --- |
| Jenkins | Página de acceso HTTP 200 después de inicializar los plugins desde cero. |
| Argo CD | Pods disponibles; API de versión HTTP 200. |
| Harbor | Todos los componentes sanos; consulta autenticada de proyectos HTTP 200. |
| CloudNativePG | Operador disponible, tres instancias PostgreSQL sanas y base `db-sonar` aplicada; conexión SQL autenticada como `sonar-user`. |
| SonarQube | PostgreSQL 15 y SonarQube disponibles; API de estado `UP`. |
| cert-manager | Emitió un certificado autofirmado de prueba; se retiró el namespace temporal de validación. |
| Ingress NGINX | Enruta correctamente las siete interfaces y API comprobadas usando los dominios `devops.cl`. |
| Metrics Server | Devuelve métricas CPU y memoria de los cuatro nodos usando la imagen oficial `v0.9.0`. |
| Monitoreo | MinIO deshabilitado. Grafana informa base de datos `ok`; Prometheus registra los cuatro nodos; Alertmanager disponible. Loki recibió y devolvió un log de prueba, escribió chunks en su volumen persistente y devolvió logs de Kubernetes recolectados por Alloy. |

Durante la inicialización simultánea se observaron descargas de imágenes lentas, demoras de `etcd` y reintentos de componentes dependientes de bases de datos. Al finalizar, la API del clúster respondió como disponible y las comprobaciones indicadas pasaron.

La instalación nueva usa los comandos normales de Helm; no requiere los flags de transferencia de propiedad utilizados en la migración anterior. Se conserva la imagen PostgreSQL disponible de SonarQube y Loki usa almacenamiento `filesystem` sobre su volumen persistente, sin MinIO.

Las pruebas comprueban arranque, acceso y las operaciones indicadas en la tabla; no incluyen pipelines Jenkins, sincronizaciones GitOps ni análisis de código.

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
helm dependency build .
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
helm dependency build .
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
helm dependency build .
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
- `jenkins.controller.ingress.hostName: jenkins.devops.cl`
- Lista de plugins preconfigurada en `jenkins.controller.installPlugins`

### Acceso local por dominio

El archivo `values.yaml` configura el ingreso con el dominio `jenkins.devops.cl`.

Una vez desplegado, Jenkins quedará disponible en:

```text
http://jenkins.devops.cl
```

Para acceder desde el navegador, registra ese dominio en el archivo `hosts` de tu sistema operativo.

Primero identifica la IP o dirección de entrada del ingress:

```bash
kubectl get ingress -n jenkins
```

Si estás trabajando en un entorno local, normalmente bastará con una entrada como esta:

```text
127.0.0.1 jenkins.devops.cl
```

En Linux y macOS, edita `/etc/hosts`.

En Windows, edita `C:\Windows\System32\drivers\etc\hosts` con permisos de administrador.

## Prometheus y Grafana

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

## Argo CD

Argo CD permite sincronizar aplicaciones de Kubernetes con los manifiestos definidos en un repositorio Git.

### Despliegue de Argo CD

#### 1) Entrar al chart

```bash
cd argocd
```

#### 2) Agregar el repositorio oficial (si no existe)

```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
```

#### 3) Descargar dependencias del chart

```bash
helm dependency build .
```

#### 4) Instalar o actualizar

```bash
helm upgrade --install argocd . -n argocd --create-namespace
```

Este comando instala o actualiza la release `argocd` en el namespace `argocd` usando la configuración definida en `values.yaml`.

#### 5) Verificar recursos

```bash
kubectl get pods -n argocd
kubectl get ingress -n argocd
```

### Comandos útiles

```bash
helm uninstall argocd -n argocd
helm get values argocd -n argocd
```

### Credenciales iniciales

El usuario inicial es `admin`. Para obtener la contraseña inicial:

```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 --decode && echo
```

Ese Secret contiene la contraseña inicial; después de cambiarla, utiliza la nueva contraseña.

### Configuración de Argo CD

- `argo-cd.global.domain: argocd.devops.cl`
- `argo-cd.server.ingress.ingressClassName: nginx`
- `argo-cd.configs.params.server.insecure: true`, para servir HTTP detrás del ingress local.
- Redirección HTTPS deshabilitada y sin configuración TLS adicional.

### Acceso local por dominio

El ingreso usa `argocd.devops.cl`. Después del despliegue, la interfaz quedará disponible en `http://argocd.devops.cl`.

Primero identifica la IP o dirección de entrada:

```bash
kubectl get ingress -n argocd
```

Registra esa IP y `argocd.devops.cl` en el archivo `hosts`. Si tu entorno local expone el ingress en el equipo, la entrada será:

```text
127.0.0.1 argocd.devops.cl
```

En Linux y macOS, edita `/etc/hosts`. En Windows, edita `C:\Windows\System32\drivers\etc\hosts` con permisos de administrador. Si el ingress tiene otra IP, utiliza esa dirección.


## Harbor

Harbor es un registro de imágenes de contenedores. Este ejemplo usa almacenamiento persistente y acceso HTTP para el entorno local del curso.

### Despliegue de Harbor

#### 1) Entrar al chart

```bash
cd harbor
```

#### 2) Agregar el repositorio oficial (si no existe)

```bash
helm repo add harbor https://helm.goharbor.io
helm repo update
```

#### 3) Descargar dependencias del chart

```bash
helm dependency build .
```

#### 4) Instalar o actualizar

```bash
helm upgrade --install harbor . -n harbor --create-namespace
```

Este comando instala o actualiza la release `harbor` en el namespace `harbor` usando la configuración definida en `values.yaml`.

#### 5) Verificar recursos

```bash
kubectl get pods -n harbor
kubectl get ingress -n harbor
kubectl get pvc -n harbor
```

### Comandos útiles

```bash
helm uninstall harbor -n harbor
helm get values harbor -n harbor
```

### Credenciales iniciales

El usuario inicial es `admin`. Para obtener la contraseña configurada:

```bash
kubectl get secret harbor-core -n harbor -o jsonpath="{.data.HARBOR_ADMIN_PASSWORD}" | base64 --decode && echo
```

### Configuración de Harbor

- `harbor.expose.ingress.hosts.core: harbor.devops.cl`
- `harbor.externalURL: http://harbor.devops.cl`
- `harbor.expose.tls.enabled: false`
- Persistencia habilitada: registro de 50Gi, base de datos de 5Gi, Redis de 1Gi, Trivy de 5Gi y logs de Jobservice de 1Gi.
- PostgreSQL y Redis internos; no depende del clúster de CloudNativePG.

### Acceso local por dominio

El ingreso usa `harbor.devops.cl`. Después del despliegue, la interfaz quedará disponible en `http://harbor.devops.cl`.

Primero identifica la IP o dirección de entrada:

```bash
kubectl get ingress -n harbor
```

Registra esa IP y `harbor.devops.cl` en el archivo `hosts`. Si tu entorno local expone el ingress en el equipo, la entrada será:

```text
127.0.0.1 harbor.devops.cl
```

En Linux y macOS, edita `/etc/hosts`. En Windows, edita `C:\Windows\System32\drivers\etc\hosts` con permisos de administrador. Si el ingress tiene otra IP, utiliza esa dirección.

### Usar Harbor desde Docker

Como este ejemplo usa HTTP, agrega `harbor.devops.cl` a los registros sin TLS del motor Docker. En Linux, incorpora esta propiedad a `/etc/docker/daemon.json`, conservando las demás propiedades existentes:

```json
{
  "insecure-registries": ["harbor.devops.cl"]
}
```

Si la lista ya existe, agrega el dominio sin eliminar los registros actuales. Reinicia Docker para aplicar el cambio. En Docker Desktop, hazlo en **Settings > Docker Engine** y aplica los cambios. Esta configuración corresponde al entorno local del curso; para exponer Harbor fuera de ese entorno, configura HTTPS y un certificado válido.

Inicia sesión introduciendo la contraseña cuando Docker la solicite:

```bash
docker login harbor.devops.cl
```

Desde la interfaz de Harbor, crea un proyecto llamado `curso`. Para publicar una imagen local que ya tengas:

```bash
docker tag mi-app:latest harbor.devops.cl/curso/mi-app:latest
docker push harbor.devops.cl/curso/mi-app:latest
```

Esto configura el cliente Docker. Si los nodos Kubernetes van a descargar imágenes de este registro HTTP, configura también su runtime y la resolución de `harbor.devops.cl` en cada nodo; el archivo `hosts` de tu equipo no configura los nodos.


## CloudNativePG

CloudNativePG instala un operador para gestionar clústeres PostgreSQL. La configuración incluye dos réplicas del operador, versión 1.27.0 y creación de RBAC.

### Despliegue de CloudNativePG

#### 1) Entrar al chart

```bash
cd cloudnative-pg
```

#### 2) Agregar el repositorio oficial (si no existe)

```bash
helm repo add cnpg https://cloudnative-pg.io/charts/
helm repo update
```

#### 3) Descargar dependencias del chart

```bash
helm dependency build .
```

#### 4) Instalar o actualizar

```bash
helm upgrade --install cloudnative-pg . -n cnpg-system --create-namespace
```

Este comando instala o actualiza la release `cloudnative-pg` en el namespace `cnpg-system` usando la configuración definida en `values.yaml`.

#### 5) Verificar recursos

```bash
kubectl get pods -n cnpg-system
kubectl get crd clusters.postgresql.cnpg.io databases.postgresql.cnpg.io
```

### Comandos útiles

```bash
helm uninstall cloudnative-pg -n cnpg-system
helm get values cloudnative-pg -n cnpg-system
```

### Crear el clúster PostgreSQL de ejemplo

El chart instala el operador. Los archivos de `postgres/` se aplican por separado y no forman parte de la instalación Helm.

Primero espera al operador y crea el namespace:

```bash
kubectl rollout status deployment -n cnpg-system -l app.kubernetes.io/instance=cloudnative-pg --timeout=180s
kubectl create namespace databases --dry-run=client -o yaml | kubectl apply -f -
```

Crea los Secrets antes de aplicar el clúster. Estos comandos se ejecutan en Bash y solicitan las contraseñas sin mostrarlas ni guardarlas en los archivos del repositorio:

```bash
read -r -s -p "Contraseña del propietario admin: " PG_ADMIN_PASSWORD; echo
kubectl create secret generic admin-secret -n databases \
  --type=kubernetes.io/basic-auth \
  --from-literal=username=admin \
  --from-literal=password="$PG_ADMIN_PASSWORD" \
  --dry-run=client -o yaml | kubectl apply -f -
unset PG_ADMIN_PASSWORD

read -r -s -p "Contraseña del usuario sonar-user: " PG_SONAR_PASSWORD; echo
kubectl create secret generic db-sonar-secret -n databases \
  --type=kubernetes.io/basic-auth \
  --from-literal=username=sonar-user \
  --from-literal=password="$PG_SONAR_PASSWORD" \
  --dry-run=client -o yaml | kubectl apply -f -
unset PG_SONAR_PASSWORD
```

Introduce contraseñas no vacías. Instala el clúster y espera a que esté disponible:

```bash
kubectl apply -f postgres/cluster.yaml
kubectl wait --for=condition=Ready cluster/main-postgres -n databases --timeout=300s
kubectl get cluster main-postgres -n databases -o yaml
```

El clúster crea tres instancias PostgreSQL 15 con volúmenes de 20Gi por instancia y gestiona el rol `sonar-user` mediante `spec.managed.roles`. Antes de crear la base, verifica en `status.managedRolesStatus` que el operador haya reconciliado ese rol sin errores.

Después crea la base de datos cuyo propietario es `sonar-user`:

```bash
kubectl apply -f postgres/db-sonar.yaml
kubectl get clusters,databases -n databases
kubectl get pods,svc,pvc -n databases
kubectl get database db-sonar -n databases -o yaml
```

Comprueba en el estado de `db-sonar` que la base se haya creado correctamente. El servicio de escritura es `main-postgres-rw.databases.svc.cluster.local:5432`.

`monitoring.enablePodMonitor` está deshabilitado para que el ejemplo pueda instalarse sin el stack de monitoreo. Después de instalar `monitoring`, puedes habilitarlo en `postgres/cluster.yaml` y volver a aplicar el archivo.

El ejemplo `db-sonar` permite practicar la creación de bases. SonarQube conserva su PostgreSQL incluida en el chart y no se conecta automáticamente a esta base.

Desinstalar el operador no equivale a eliminar los clústeres PostgreSQL ni sus datos. Revisa los recursos y respaldos antes de retirar el operador o eliminar recursos persistentes.


## SonarQube

SonarQube permite analizar la calidad del código. Este ejemplo habilita la edición Community y usa la PostgreSQL incluida en la dependencia oficial.

### Despliegue de SonarQube

#### 1) Entrar al chart

```bash
cd sonarqube
```

#### 2) Agregar el repositorio oficial (si no existe)

```bash
helm repo add sonarqube https://SonarSource.github.io/helm-chart-sonarqube
helm repo update
```

#### 3) Descargar dependencias del chart

```bash
helm dependency build .
```

Antes de instalar, crea el Secret del passcode de monitoreo. Este passcode se utiliza en las sondas de SonarQube y es distinto de la contraseña del usuario web. Ejecuta estos comandos en Bash:

```bash
kubectl create namespace sonarqube --dry-run=client -o yaml | kubectl apply -f -
read -r -s -p "Passcode de monitoreo (no vacío): " SONAR_MONITORING_PASSCODE; echo
kubectl create secret generic sonarqube-monitoring -n sonarqube \
  --from-literal=passcode="$SONAR_MONITORING_PASSCODE" \
  --dry-run=client -o yaml | kubectl apply -f -
unset SONAR_MONITORING_PASSCODE
```

#### 4) Instalar o actualizar

```bash
helm upgrade --install sonarqube . -n sonarqube --create-namespace
```

Este comando instala o actualiza la release `sonarqube` en el namespace `sonarqube` usando la configuración definida en `values.yaml`.

#### 5) Verificar recursos

```bash
kubectl get pods -n sonarqube
kubectl get ingress -n sonarqube
kubectl get pvc -n sonarqube
```

### Comandos útiles

```bash
helm uninstall sonarqube -n sonarqube
helm get values sonarqube -n sonarqube
```

### Credenciales iniciales

En una instalación nueva, el usuario y la contraseña iniciales son `admin`. SonarQube solicitará cambiar la contraseña al iniciar sesión.

### Configuración de SonarQube

- `sonarqube.community.enabled: true`
- Ingress de clase `nginx` con dominio `sonar.devops.cl` y sin TLS.
- Passcode tomado del Secret `sonarqube-monitoring`, clave `passcode`.
- PostgreSQL incluida en la dependencia, con persistencia habilitada por defecto y volumen de 20Gi. Se fija la imagen `bitnamilegacy/postgresql:15.9.0-debian-12-r0` porque la etiqueta PostgreSQL 11 predeterminada ya no está disponible.
- La persistencia del almacenamiento de SonarQube está deshabilitada por defecto; la base de datos guarda sus datos. Revisa los valores de la dependencia si necesitas persistir también ese almacenamiento.

Esta configuración PostgreSQL 15 corresponde a instalaciones nuevas. Si ya tienes datos de otra versión mayor de PostgreSQL, prepara su respaldo y migración antes de cambiar la imagen; Helm no realiza esa migración.

La instalación de CloudNativePG es independiente de SonarQube.

### Acceso local por dominio

El ingreso usa `sonar.devops.cl`. Después del despliegue, la interfaz quedará disponible en `http://sonar.devops.cl`.

Primero identifica la IP o dirección de entrada:

```bash
kubectl get ingress -n sonarqube
```

Registra esa IP y `sonar.devops.cl` en el archivo `hosts`. Si tu entorno local expone el ingress en el equipo, la entrada será:

```text
127.0.0.1 sonar.devops.cl
```

En Linux y macOS, edita `/etc/hosts`. En Windows, edita `C:\Windows\System32\drivers\etc\hosts` con permisos de administrador. Si el ingress tiene otra IP, utiliza esa dirección.


## cert-manager

cert-manager gestiona certificados TLS en Kubernetes. Es opcional para los ejemplos locales por HTTP y debe instalarse antes de aplicar recursos `Issuer`, `ClusterIssuer` o `Certificate`.

### Despliegue de cert-manager

#### 1) Entrar al chart

```bash
cd cert-manager
```

#### 2) Agregar el repositorio oficial (si no existe)

```bash
helm repo add jetstack https://charts.jetstack.io
helm repo update
```

#### 3) Descargar dependencias del chart

```bash
helm dependency build .
```

#### 4) Instalar o actualizar

```bash
helm upgrade --install cert-manager . -n cert-manager --create-namespace
```

Este comando instala o actualiza la release `cert-manager` en el namespace `cert-manager` usando la configuración definida en `values.yaml`.

#### 5) Verificar recursos

```bash
kubectl get pods -n cert-manager
kubectl get crd certificates.cert-manager.io issuers.cert-manager.io clusterissuers.cert-manager.io
```

### Comandos útiles

```bash
helm uninstall cert-manager -n cert-manager
helm get values cert-manager -n cert-manager
```

### Configuración de cert-manager

- `cert-manager.crds.enabled: true`, para instalar sus definiciones de recursos.
- `cert-manager.config.featureGates.ACMEHTTP01IngressPathTypeExact: false`, conservado del chart de origen.

Instalar el chart no crea emisores ni certificados. Para practicar TLS con los subdominios de `devops.cl`, configura un emisor interno o un emisor público con un método de validación autorizado para ese dominio. Las entradas del archivo `hosts` sólo configuran la resolución de nombres en el cliente; no demuestran el control del dominio ante una autoridad pública.

Este repositorio no incluye los recursos de Google Cloud DNS ni el Secret de cuenta de servicio del Ingress remoto.
