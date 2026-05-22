
**Laboratorio Post-Contenido 1 — Unidad 12: Despliegue y CI/CD**  
Programación Web · Ingeniería de Sistemas · UDES 2026

---

##  URL en Railway

> **https://jaimes-post1-u12.up.railway.app**

---

##  Estructura del proyecto

```
garcia-post1-u12/
├── Dockerfile                          # Multi-stage: JDK builder + JRE producción
├── .dockerignore
├── docker-compose.yml                  # Orquestación local: app + PostgreSQL
├── pom.xml
├── src/
│   └── main/
│       ├── java/com/ejemplo/app/
│       │   ├── MiAppApplication.java
│       │   ├── model/
│       │   │   ├── Producto.java
│       │   │   └── ProductoRepository.java
│       │   └── controller/
│       │       └── ProductoController.java
│       └── resources/
│           ├── application.properties          # Perfil desarrollo (H2)
│           └── application-prod.properties     # Perfil producción (PostgreSQL)
└── README.md
```

---

##  Variables de entorno requeridas

| Variable | Descripción | Ejemplo |
|---|---|---|
| `SPRING_PROFILES_ACTIVE` | Perfil activo | `prod` |
| `DATABASE_URL` | URL JDBC de PostgreSQL | `jdbc:postgresql://host:5432/appdb` |
| `DB_USER` | Usuario de la base de datos | `appuser` |
| `DB_PASS` | Contraseña de la base de datos | `apppass` |

---

##  Construir y ejecutar localmente con Docker

### 1. Construir la imagen Docker

```bash
docker build -t mi-app:local .
```

Verificar tamaño (debe ser < 300 MB):

```bash
docker images mi-app:local
```

### 2. Levantar el stack completo (app + PostgreSQL)

```bash
docker compose up -d --build
```

Verificar que ambos contenedores estén sanos:

```bash
docker compose ps
```

### 3. Probar los endpoints

```bash
# Health check
curl http://localhost:8080/actuator/health

# Listar productos
curl http://localhost:8080/api/productos

# Crear un producto
curl -X POST http://localhost:8080/api/productos \
  -H "Content-Type: application/json" \
  -d '{"nombre":"Laptop","precio":1500.00,"descripcion":"Laptop gaming","stock":10}'

# Obtener por ID
curl http://localhost:8080/api/productos/1

# Actualizar
curl -X PUT http://localhost:8080/api/productos/1 \
  -H "Content-Type: application/json" \
  -d '{"nombre":"Laptop Pro","precio":1800.00,"descripcion":"Laptop gaming Pro","stock":5}'

# Eliminar
curl -X DELETE http://localhost:8080/api/productos/1
```

### 4. Detener los contenedores

```bash
docker compose down
```

---

## Despliegue en Railway

### Pasos realizados

1. Acceder a [railway.app](https://railway.app) e iniciar sesión con GitHub.
2. Crear nuevo proyecto → **"Deploy from GitHub repo"** → seleccionar `garcia-post1-u12`.
3. Railway detecta el `Dockerfile` automáticamente y ejecuta el primer build (~3-5 min).
4. Agregar servicio de base de datos: **"+ New" → "Database" → "Add PostgreSQL"**.
5. En el servicio de la aplicación, ir a **"Variables"** y configurar:

```
SPRING_PROFILES_ACTIVE  = prod
DATABASE_URL            = ${{Postgres.DATABASE_URL}}
DB_USER                 = ${{Postgres.PGUSER}}
DB_PASS                 = ${{Postgres.PGPASSWORD}}
```

6. Generar dominio público: **"Settings" → "Networking" → "Generate Domain"**.

### Verificación del despliegue

```bash
# Health check en Railway
curl https://garcia-post1-u12.up.railway.app/actuator/health
# Respuesta esperada: {"status":"UP","components":{"db":{"status":"UP"},...}}

# Listar productos
curl https://garcia-post1-u12.up.railway.app/api/productos

# Crear producto
curl -X POST https://garcia-post1-u12.up.railway.app/api/productos \
  -H "Content-Type: application/json" \
  -d '{"nombre":"Teclado","precio":75.00,"descripcion":"Mecánico RGB","stock":20}'
```

---

##  Endpoints REST disponibles

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| `GET` | `/api/productos` | Listar todos los productos |
| `GET` | `/api/productos/{id}` | Obtener producto por ID |
| `POST` | `/api/productos` | Crear nuevo producto |
| `PUT` | `/api/productos/{id}` | Actualizar producto |
| `DELETE` | `/api/productos/{id}` | Eliminar producto |
| `GET` | `/actuator/health` | Estado de la aplicación y BD |



*Universidad de Santander (UDES) · Programación Web 2026*
