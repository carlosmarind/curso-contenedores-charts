# SonarQube

[Guía general y requisitos](../README.md)

SonarQube permite analizar la calidad del código. Este ejemplo habilita la edición Community y se conecta a la base `db-sonar` del clúster CloudNativePG `main-postgres`. La dependencia oficial ya no incluye PostgreSQL.

## Dependencias

| Chart | Versión |
| --- | --- |
| sonarqube | 2026.5.1002 |

## Despliegue de SonarQube

### 1) Entrar al chart

```bash
cd sonarqube
```

### 2) Agregar el repositorio oficial (si no existe)

```bash
helm repo add sonarqube https://SonarSource.github.io/helm-chart-sonarqube
helm repo update
```

### 3) Descargar dependencias del chart

```bash
helm dependency build .
```

Antes de instalar SonarQube, completa la [guía de CloudNativePG](../cloudnative-pg/readme.md#crear-el-clúster-postgresql-de-ejemplo) y crea el clúster `main-postgres`, el rol `sonar-user` y la base `db-sonar`. Comprueba que las tres instancias estén disponibles y que la base esté aplicada:

```bash
kubectl wait cluster/main-postgres -n databases --for=condition=Ready --timeout=600s
kubectl get database db-sonar -n databases
```

Después, crea el Secret del passcode de monitoreo. Este passcode se utiliza en las sondas de SonarQube y es distinto de la contraseña del usuario web. Ejecuta estos comandos en Bash:

```bash
kubectl create namespace sonarqube --dry-run=client -o yaml | kubectl apply -f -
read -r -s -p "Passcode de monitoreo (no vacío): " SONAR_MONITORING_PASSCODE; echo
kubectl create secret generic sonarqube-monitoring -n sonarqube \
  --from-literal=passcode="$SONAR_MONITORING_PASSCODE" \
  --dry-run=client -o yaml | kubectl apply -f -
unset SONAR_MONITORING_PASSCODE
```

El Secret de la contraseña de PostgreSQL debe estar en el mismo namespace que SonarQube. Copia la contraseña del rol `sonar-user` sin escribirla en los valores del chart ni mostrarla en pantalla:

```bash
SONAR_JDBC_PASSWORD=$(kubectl get secret db-sonar-secret -n databases -o jsonpath='{.data.password}' | base64 --decode)
kubectl create secret generic sonarqube-jdbc -n sonarqube \
  --from-literal=password="$SONAR_JDBC_PASSWORD" \
  --dry-run=client -o yaml | kubectl apply -f -
unset SONAR_JDBC_PASSWORD
```

### 4) Instalar o actualizar

```bash
helm upgrade --install sonarqube . -n sonarqube --create-namespace
```

Este comando instala o actualiza la release `sonarqube` en el namespace `sonarqube` usando la configuración definida en `values.yaml`.

### 5) Verificar recursos

```bash
kubectl get pods -n sonarqube
kubectl get ingress -n sonarqube
```

## Comandos útiles

```bash
helm uninstall sonarqube -n sonarqube
helm get values sonarqube -n sonarqube
```

## Credenciales iniciales

En una instalación nueva, el usuario y la contraseña iniciales son `admin`. SonarQube solicitará cambiar la contraseña al iniciar sesión.

## Configuración de SonarQube

- `sonarqube.community.enabled: true`
- Ingress de clase `nginx` con dominio `sonar.devops.cl` y sin TLS.
- Passcode tomado del Secret `sonarqube-monitoring`, clave `passcode`.
- PostgreSQL externo en `main-postgres-rw.databases.svc.cluster.local:5432`, base `db-sonar` y usuario `sonar-user`.
- Conexión JDBC con `sslmode=require`; la contraseña se toma del Secret `sonarqube-jdbc`, clave `password`.
- Los datos se conservan en los volúmenes del clúster CloudNativePG. SonarQube no crea una PostgreSQL ni un volumen de base de datos propios.
- Los índices de búsqueda y archivos locales de SonarQube no tienen persistencia habilitada; los índices se reconstruyen desde PostgreSQL. Los logs locales y las extensiones instaladas manualmente pueden perderse al recrear el pod.

Al actualizar una instalación anterior que usaba la PostgreSQL incluida, respalda y migra sus datos antes de desinstalarla. Cambiar la URL JDBC no copia los datos automáticamente.

## Acceso local

Consulta las [URLs, dominios y configuración del archivo hosts](../README.md#dominios-y-archivo-hosts) de la guía general. Allí se explica también cómo eliminar políticas HSTS en Edge.


## Comprobar funcionamiento

Consulta el estado de la aplicación:

```bash
curl -fsS http://sonar.devops.cl/api/system/status
```

El campo `status` debe indicar `UP`. La primera inicialización puede tardar mientras se crea el esquema de la base de datos.
