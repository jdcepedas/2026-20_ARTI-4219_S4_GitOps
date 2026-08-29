# =====================================================================
# Análisis del Provider PostgreSQL
# =====================================================================

## Provider: tages/provider-postgresql v0.1.0

### 1. Managed Resources disponibles

El provider `tages/provider-postgresql v0.1.0` expone **17 Managed Resources**, todos con alcance **Cluster Scoped** (0 Namespace Scoped):

| Kind | Group | Version | Función |
|---|---|---|---|
| Database | `postgresql.postgresql.upbound.io` | v1alpha1 | Crea y administra bases de datos dentro del servidor PostgreSQL. |
| Extension | `postgresql.postgresql.upbound.io` | v1alpha1 | Instala y gestiona extensiones de PostgreSQL (p. ej. `pgcrypto`, `uuid-ossp`) en una base de datos. |
| Function | `postgresql.postgresql.upbound.io` | v1alpha1 | Define y administra funciones (stored functions) dentro de una base de datos. |
| Grant | `postgresql.postgresql.upbound.io` | v1alpha1 | Otorga privilegios (SELECT, INSERT, USAGE, etc.) sobre objetos de base de datos a un rol. |
| Mapping | `user.postgresql.upbound.io` | v1alpha1 | Administra *user mappings* usados por Foreign Data Wrappers (FDW) para asociar un usuario local con credenciales de un servidor remoto. |
| Privileges | `default.postgresql.upbound.io` | v1alpha1 | Configura privilegios por defecto (`ALTER DEFAULT PRIVILEGES`) que se aplican a objetos creados a futuro por un rol. |
| ProviderConfig | `postgresql.upbound.io` | v1beta1 | Define la configuración de conexión (credenciales, host, puerto) que usan los demás Managed Resources para conectarse al servidor. |
| ProviderConfigUsage | `postgresql.upbound.io` | v1beta1 | Recurso interno que rastrea qué Managed Resources están usando un `ProviderConfig` determinado. |
| Publication | `postgresql.postgresql.upbound.io` | v1alpha1 | Crea y administra publicaciones de replicación lógica (`CREATE PUBLICATION`). |
| ReplicationSlot | `physical.postgresql.upbound.io` | v1alpha1 | Administra *replication slots* físicos, usados en replicación física entre servidores. |
| Role (membership) | `grant.postgresql.upbound.io` | v1alpha1 | Otorga membresía de un rol dentro de otro rol (equivalente a `GRANT rol_a TO rol_b`). |
| Role | `postgresql.postgresql.upbound.io` | v1alpha1 | Crea y administra roles/usuarios de PostgreSQL (login, contraseña, atributos como `SUPERUSER`, `CREATEDB`, etc.). |
| Schema | `postgresql.postgresql.upbound.io` | v1alpha1 | Crea y administra esquemas dentro de una base de datos. |
| Server | `postgresql.postgresql.upbound.io` | v1alpha1 | Define un servidor foráneo (Foreign Server) para su uso con Foreign Data Wrappers. |
| Slot | `replication.postgresql.upbound.io` | v1alpha1 | Administra *replication slots* lógicos, usados en replicación lógica. |
| StoreConfig | `postgresql.upbound.io` | v1alpha1 | Configura el almacén externo de secretos (external secret store) donde se pueden publicar las credenciales generadas. |
| Subscription | `postgresql.postgresql.upbound.io` | v1alpha1 | Crea y administra suscripciones de replicación lógica que consumen una `Publication` (`CREATE SUBSCRIPTION`). |

> Fuente: [Upbound Marketplace – tages/provider-postgresql v0.1.0](https://marketplace.upbound.io/providers/tages/provider-postgresql/v0.1.0?tab=managedResources)

### 2. Campos requeridos del recurso Database

`Database` es el Schema para la API de Databases: crea y administra una base de datos en un servidor PostgreSQL.

**Campos de nivel superior (estructura estándar de un Managed Resource de Crossplane):**

| Campo | Tipo | Requerido | Descripción |
|---|---|---|---|
| `apiVersion` | String | Sí | `postgresql.postgresql.upbound.io/v1alpha1` |
| `kind` | String | Sí | `Database` |
| `metadata.name` | String | Sí | Nombre del recurso Crossplane (no necesariamente el nombre de la base de datos). |
| `spec.forProvider` | Object | Sí | Contiene los parámetros reales de la base de datos a crear (ver tabla siguiente). |
| `spec.providerConfigRef.name` | String | Sí | Referencia al `ProviderConfig` que indica a qué servidor PostgreSQL conectarse. |

**Campos dentro de `spec.forProvider`** (la API no marca ninguno individualmente como obligatorio a nivel de esquema, pero en la práctica se recomienda al menos definir `name`, ya que sin él Terraform/el provider usaría el nombre del recurso; el resto tiene valores por defecto en PostgreSQL):

| Campo | Tipo | Descripción |
|---|---|---|
| `name` | String | Nombre de la base de datos a crear en el servidor. |
| `owner` | String | Rol propietario de la base de datos. |
| `encoding` | String | Codificación de caracteres (p. ej. `UTF8`). |
| `lcCollate` | String | Configuración regional de ordenamiento (`LC_COLLATE`). |
| `lcCtype` | String | Configuración regional de clasificación de caracteres (`LC_CTYPE`). |
| `tablespaceName` | String | Tablespace donde se almacenará la base de datos. |
| `template` | String | Base de datos plantilla a partir de la cual se crea la nueva (p. ej. `template0`). |
| `isTemplate` | Boolean | Indica si la base de datos puede usarse como plantilla para otras. |
| `allowConnections` | Boolean | Permite o bloquea nuevas conexiones a la base de datos. |
| `connectionLimit` | Number | Número máximo de conexiones concurrentes permitidas (`-1` = sin límite). |

**Otros campos opcionales relevantes en `spec`:**
- `deletionPolicy`: define si al borrar el recurso Crossplane también se elimina (`Delete`) o no (`Orphan`) la base de datos real.
- `managementPolicies`: controla qué acciones (Create, Update, Delete, Observe, LateInitialize) puede ejecutar Crossplane sobre el recurso.
- `writeConnectionSecretToRef` / `publishConnectionDetailsTo`: indican dónde publicar el Secret con los detalles de conexión.

> Fuente: [Upbound Marketplace – Database v1alpha1](https://marketplace.upbound.io/providers/tages/provider-postgresql/v0.1.0/resources/postgresql.postgresql.upbound.io/Database/v1alpha1)

### 3. Información requerida por el ProviderConfig

`ProviderConfig` configura la conexión que usará el provider de PostgreSQL para autenticarse contra el servidor. Su único campo funcional es `spec.credentials`, obligatorio. Esto puede observarse en la hoja de trabajo 4 como ejemplo en la configuración provider-config.yaml.


**Campos de nivel superior:**

| Campo | Tipo | Requerido | Descripción |
|---|---|---|---|
| `apiVersion` | String | Sí | `postgresql.upbound.io/v1beta1` |
| `kind` | String | Sí | `ProviderConfig` |
| `metadata.name` | String | Sí | Nombre del ProviderConfig, referenciado por los Managed Resources en `spec.providerConfigRef.name`. |
| `spec.credentials` | Object | Sí | Credenciales necesarias para autenticarse contra el servidor PostgreSQL. |

**Dentro de `spec.credentials`:**

| Campo | Tipo | Requerido | Descripción |
|---|---|---|---|
| `source` | String | Sí | Indica el origen de las credenciales. Valores típicos: `Secret`, `Environment` (`env`) o `Filesystem` (`fs`). |
| `secretRef` | Object | Condicional (si `source: Secret`) | Referencia a un `Secret` de Kubernetes que contiene las credenciales de conexión. |
| `secretRef.name` | String | Sí (dentro de `secretRef`) | Nombre del Secret. |
| `secretRef.namespace` | String | Sí (dentro de `secretRef`) | Namespace donde vive el Secret. |
| `secretRef.key` | String | Sí (dentro de `secretRef`) | Clave dentro del Secret que contiene el valor de las credenciales (usualmente un string de conexión). |
| `env.name` | String | Condicional (si `source: Environment`) | Nombre de la variable de entorno que contiene las credenciales. |
| `fs.path` | String | Condicional (si `source: Filesystem`) | Ruta en el filesystem del pod del provider donde están las credenciales. |

**Notas:**
- La opción más común en un despliegue GitOps es `source: Secret`, apuntando a un `Secret` de Kubernetes (`secretRef`) que contiene el string de conexión al servidor PostgreSQL (host, puerto, usuario, contraseña, `sslmode`, etc., como un DSN/URI), ya que el schema del ProviderConfig no expone campos separados de `host`/`port`/`username` — todo va empaquetado en el valor referenciado por `secretRef.key`.
- El `status` del ProviderConfig es de solo lectura y expone `conditions` (estado de salud) y `users` (cantidad de Managed Resources que actualmente lo referencian).

En el caso de la presenta actividad se utilizó un credentials de tipo Secret y se referencian los secretos creados en el paso 4.

kubectl create secret generic postgresql-credentials \
  --namespace crossplane-system \
  --from-literal=connection='{"host":"postgresql.postgresql.svc.cluster.local","port":"5432","username":"postgres","password":"platform123","database":"postgres","sslmode":"disable"}'

> Fuente: [Upbound Marketplace – ProviderConfig v1beta1](https://marketplace.upbound.io/providers/tages/provider-postgresql/v0.1.0/resources/postgresql.upbound.io/ProviderConfig/v1beta1)
