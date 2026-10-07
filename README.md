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

## Guías de los charts

Elige los servicios que necesites y sigue su guía individual para instalar, configurar, verificar y desinstalar la release.

| Guía | Uso | Namespace |
| --- | --- | --- |
| [Ingress NGINX](ingress-nginx/readme.md) | Entrada HTTP al clúster | `ingress-nginx` |
| [Metrics Server](metrics-server/readme.md) | Métricas de CPU y memoria | `kube-system` |
| [Jenkins](jenkins/readme.md) | Automatización y pipelines | `jenkins` |
| [Prometheus y Grafana](monitoring/readme.md) | Métricas, alertas y logs | `monitoring` |
| [Argo CD](argocd/readme.md) | Despliegues desde Git | `argocd` |
| [Harbor](harbor/readme.md) | Registro de imágenes | `harbor` |
| [CloudNativePG](cloudnative-pg/readme.md) | Operador y clúster PostgreSQL | `cnpg-system / databases` |
| [SonarQube](sonarqube/readme.md) | Análisis de calidad de código | `sonarqube` |
| [cert-manager](cert-manager/readme.md) | Emisión de certificados TLS | `cert-manager` |

Cada carpeta contiene `Chart.yaml`, `Chart.lock`, `values.yaml`, las dependencias empaquetadas en `charts/` y su `readme.md`. Las versiones y configuraciones específicas se documentan en cada guía.

## Orden de instalación

1. Ingress NGINX, antes de los servicios con ingreso web.
2. Metrics Server, para las métricas de recursos.
3. Jenkins y monitoreo, según los temas del curso.
4. Argo CD y Harbor, según los temas del curso.
5. CloudNativePG y el clúster PostgreSQL de ejemplo con la base `db-sonar`, antes de instalar SonarQube.
6. SonarQube, que utiliza esa base externa.
7. cert-manager, cuando se trabaje con TLS y antes de crear emisores o certificados.

No es obligatorio instalar todos los charts. Harbor usa su propia base de datos. SonarQube utiliza la base `db-sonar` en CloudNativePG y requiere instalarla previamente. Los servicios web usan HTTP y subdominios de `devops.cl`, sin depender de cert-manager.

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

Los nombres internos de Kubernetes, como `*.svc.cluster.local`, se conservan. Si habilitas interfaces adicionales, agrega también sus dominios al archivo `hosts`, según la configuración de su chart.

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

Los comandos de las guías individuales parten desde la raíz de este repositorio; su primer paso entra en la carpeta del chart. Si terminaste dentro de un chart, vuelve con `cd ..` antes de comenzar otra guía.

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

## Actualizar dependencias

Antes de cambiar una versión, revisa las instrucciones de actualización de la dependencia y respalda los datos persistentes. Cambiar la versión de un chart no migra automáticamente bases de datos ni revierte sus datos.

## Verificar el despliegue

Después de instalar los charts que necesites, revisa las releases, los pods y el almacenamiento:

```bash
helm list -A
kubectl get pods -A
kubectl get pvc -A
```

Las releases deben aparecer como `deployed`. Los pods de los servicios deben estar `Running` con todos sus contenedores disponibles; los Jobs terminados pueden aparecer como `Completed`. Los volúmenes usados por los servicios deben estar `Bound`.

Para verificar métricas, endpoints de salud, bases de datos y otras funciones, sigue el apartado **Comprobar funcionamiento** de la guía individual correspondiente. Las [URLs de acceso](#urls-de-acceso) están centralizadas en esta guía.

Si un servicio no está disponible, consulta sus eventos y logs. Sustituye los nombres por el namespace y pod correspondientes:

```bash
kubectl get events -n <namespace> --sort-by=.metadata.creationTimestamp
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace> -c <contenedor> --tail=100
```

Los charts con almacenamiento persistente pueden conservar PVC después de `helm uninstall`. Consulta los volúmenes del namespace y respalda sus datos antes de eliminarlos. Las instrucciones de cada servicio incluyen los comandos para instalar, actualizar y desinstalar su release.
