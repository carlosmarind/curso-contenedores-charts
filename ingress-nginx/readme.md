# Ingress NGINX

[Guía general y requisitos](../README.md)

Este chart instala el controlador Ingress NGINX para enrutar las solicitudes HTTP a los servicios del clúster.

## Dependencias

| Chart | Versión |
| --- | --- |
| ingress-nginx | 4.15.1 |

## Despliegue de Ingress NGINX

### 1) Entrar al chart

```bash
cd ingress-nginx
```

### 2) Agregar el repositorio oficial (si no existe)

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
```

### 3) Descargar dependencias del chart

```bash
helm dependency build .
```

### 4) Instalar o actualizar

```bash
helm upgrade --install ingress-nginx . -n ingress-nginx --create-namespace
```

Este comando instala o actualiza la release `ingress-nginx` en el namespace `ingress-nginx` usando la configuración definida en `values.yaml`.

### 5) Verificar recursos

```bash
kubectl get pods -n ingress-nginx
kubectl get svc -n ingress-nginx
kubectl get ingressclass
```

## Comandos útiles

```bash
helm uninstall ingress-nginx -n ingress-nginx
helm get values ingress-nginx -n ingress-nginx
kubectl describe ingressclass nginx
```

## Configuración de Ingress NGINX

La configuración principal está en [values.yaml](values.yaml).

Algunos valores relevantes son:

- `ingress-nginx.controller.ingressClass: nginx`
- `ingress-nginx.controller.ingressClassResource.name: nginx`
- `ingress-nginx.controller.ingressClassResource.default: true`
- `ingress-nginx.controller.service.type: LoadBalancer`

## Comprobar funcionamiento

El controlador debe estar disponible y el Service debe ofrecer una entrada accesible desde tu equipo. Usa una aplicación con ingress instalada para comprobar el enrutamiento, siguiendo la [guía de acceso general](../README.md#dominios-y-archivo-hosts).
