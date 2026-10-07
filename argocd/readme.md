# Argo CD

[Guía general y requisitos](../README.md)

Argo CD permite sincronizar aplicaciones de Kubernetes con los manifiestos definidos en un repositorio Git.

## Dependencias

| Chart | Versión |
| --- | --- |
| argo-cd | 10.10.0 |

## Despliegue de Argo CD

### 1) Entrar al chart

```bash
cd argocd
```

### 2) Agregar el repositorio oficial (si no existe)

```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
```

### 3) Descargar dependencias del chart

```bash
helm dependency build .
```

### 4) Instalar o actualizar

```bash
helm upgrade --install argocd . -n argocd --create-namespace
```

Este comando instala o actualiza la release `argocd` en el namespace `argocd` usando la configuración definida en `values.yaml`.

### 5) Verificar recursos

```bash
kubectl get pods -n argocd
kubectl get ingress -n argocd
```

## Comandos útiles

```bash
helm uninstall argocd -n argocd
helm get values argocd -n argocd
```

## Credenciales iniciales

El usuario inicial es `admin`. Para obtener la contraseña inicial:

```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 --decode && echo
```

Ese Secret contiene la contraseña inicial; después de cambiarla, utiliza la nueva contraseña.

## Configuración de Argo CD

- `argo-cd.global.domain: argocd.devops.cl`
- `argo-cd.server.ingress.ingressClassName: nginx`
- `argo-cd.configs.params.server.insecure: true`, para servir HTTP detrás del ingress local.
- Redirección HTTPS deshabilitada y sin configuración TLS adicional.

## Acceso local

Consulta las [URLs, dominios y configuración del archivo hosts](../README.md#dominios-y-archivo-hosts) de la guía general. Allí se explica también cómo eliminar políticas HSTS en Edge.


## Comprobar funcionamiento

Consulta la API de versión; debe devolver la versión de Argo CD:

```bash
curl -fsS http://argocd.devops.cl/api/version
```
