# CloudNativePG

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
