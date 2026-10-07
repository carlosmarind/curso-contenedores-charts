# Jenkins

[Guía general y requisitos](../README.md)

Este chart instala Jenkins con almacenamiento persistente y los plugins configurados en sus valores.

## Dependencias

| Chart | Versión |
| --- | --- |
| jenkins | 5.9.67 |

## Despliegue de Jenkins

### 1) Entrar al chart

```bash
cd jenkins
```

### 2) Agregar repositorio de Jenkins (si no existe)

```bash
helm repo add jenkins https://charts.jenkins.io
helm repo update
```

### 3) Descargar dependencias del chart

```bash
helm dependency build .
```

### 4) Instalar o actualizar

```bash
helm upgrade --install jenkins . -n jenkins --create-namespace
```

Este comando crea el namespace `jenkins` si no existe e instala o actualiza la release `jenkins` usando la configuración definida en `values.yaml`.

### 5) Verificar recursos

```bash
kubectl get pods -n jenkins
kubectl get ingress -n jenkins
```

## Credenciales iniciales

El usuario inicial es `admin`. Para obtener la contraseña:

```bash
kubectl exec --namespace jenkins -it svc/jenkins -c jenkins -- /bin/cat /run/secrets/additional/chart-admin-password && echo
```

## Comandos útiles

```bash
helm uninstall jenkins -n jenkins
helm get values jenkins -n jenkins
```

## Configuración de Jenkins

La configuración principal está en [values.yaml](values.yaml).

Algunos valores relevantes son:

- `jenkins.controller.ingress.enabled: true`
- `jenkins.controller.ingress.hostName: jenkins.devops.cl`
- Lista de plugins preconfigurada en `jenkins.controller.installPlugins`

## Acceso local

Consulta las [URLs, dominios y configuración del archivo hosts](../README.md#dominios-y-archivo-hosts) de la guía general. Allí se explica también cómo eliminar políticas HSTS en Edge.


## Comprobar funcionamiento

Abre la [URL de Jenkins](../README.md#urls-de-acceso) y comprueba que aparece la página de acceso. También puedes comprobarla desde la terminal:

```bash
curl -fsS -o /dev/null http://jenkins.devops.cl/login
```
