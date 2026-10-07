# cert-manager

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
