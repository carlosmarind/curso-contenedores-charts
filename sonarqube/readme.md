# SonarQube

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
