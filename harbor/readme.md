# Harbor

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
