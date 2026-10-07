# Argo CD

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
