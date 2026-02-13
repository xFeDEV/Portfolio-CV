# Feexel - Sistema de Gestión de Talento (ID System)

Sistema web para la gestión de colaboradores y generación de carnets corporativos, desarrollado por **Feexel - Software a la Medida**.

![Python](https://img.shields.io/badge/Python-3.12-blue)
![FastAPI](https://img.shields.io/badge/FastAPI-0.100+-green)
![Next.js](https://img.shields.io/badge/Next.js-15+-black)
![MariaDB](https://img.shields.io/badge/MariaDB-LTS-brown)
![Docker](https://img.shields.io/badge/Docker-Compose-blue)

---

## Descripción General

Este sistema permite:

- **Gestionar Colaboradores**: Registro completo con foto (subida o captura por cámara web), datos personales, cargo y tipo de sangre (RH)
- **Gestionar Contactos de Emergencia**: Registro de personas de contacto en caso de emergencia
- **Gestionar Áreas / Squads**: Organización por áreas (Backend, Frontend, Mobile, DevOps, etc.)
- **Generar Carnets PDF**: Carnets corporativos con QR, foto del colaborador y datos de contacto de emergencia
- **Módulo de Emergencia**: Página pública accesible vía QR con información de contacto de emergencia
- **Configuración Global**: Datos del reverso del carnet (vencimiento, texto legal, branding)
- **Sistema de Autenticación**: Login con JWT, roles de usuario (admin/operador)

---

## Arquitectura del Sistema

```
┌─────────────────────────────────────────────────────────────────┐
│                         DOCKER COMPOSE                          │
├─────────────────┬─────────────────────┬─────────────────────────┤
│                 │                     │                         │
│   ┌─────────┐   │   ┌─────────────┐   │   ┌─────────────────┐   │
│   │ MariaDB │   │   │   FastAPI   │   │   │    Next.js      │   │
│   │  :3306  │◄──┼───┤    :8000    │◄──┼───┤     :3000       │   │
│   │         │   │   │             │   │   │                 │   │
│   │ Database│   │   │  Backend    │   │   │   Frontend      │   │
│   └─────────┘   │   │  + Static   │   │   │   (React 19)    │   │
│                 │   │    Files    │   │   │                 │   │
│                 │   └─────────────┘   │   └─────────────────┘   │
│                 │         │           │           │             │
│   Volume:       │   Volume:           │                         │
│   mariadb_data  │   uploads_data      │                         │
│                 │   (/static/fotos)   │                         │
└─────────────────┴─────────────────────┴─────────────────────────┘
```

---

## Modelo de Datos

```
┌─────────────────────────┐       ┌─────────────────────────┐
│  CONTACTOS_EMERGENCIA   │       │          AREAS          │
├─────────────────────────┤       ├─────────────────────────┤
│ PK identificacion       │       │ PK id_area (AUTO)       │
│    _contacto            │       │    nombre               │
│    nombre_contacto      │       │                         │
│    celular_1            │       │ UNIQUE(nombre)          │
│    celular_2?           │       │                         │
│    email?               │       │                         │
└──────────┬──────────────┘       └───────────┬─────────────┘
           │                                  │
           │ 1:N                              │ 1:N
           │                                  │
           ▼                                  ▼
┌─────────────────────────────────────────────────────────────┐
│                      COLABORADORES                           │
├─────────────────────────────────────────────────────────────┤
│ PK identificacion_colaborador (VARCHAR)                     │
│    nombres                                                  │
│    apellidos                                                │
│    rh (tipo de sangre)                                      │
│    cargo (título del puesto)                                │
│    foto_archivo? (nombre del archivo)                       │
│ FK id_area → areas.id_area                                  │
│ FK identificacion_contacto → contactos_emergencia           │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                 CONFIGURACION_CARNET                          │
├─────────────────────────────────────────────────────────────┤
│ PK id (DEFAULT 1, singleton)                                │
│    fecha_vencimiento                                        │
│    texto_legal                                              │
│    empresa_nombre                                           │
│    empresa_slogan                                           │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                       USUARIOS                               │
├─────────────────────────────────────────────────────────────┤
│ PK id (AUTO_INCREMENT)                                      │
│    username (UNIQUE)                                        │
│    password_hash                                            │
│    nombre                                                   │
│    rol (ENUM: 'admin', 'operador')                          │
│    activo (BOOLEAN)                                         │
│    created_at (TIMESTAMP)                                   │
│    updated_at (TIMESTAMP)                                   │
└─────────────────────────────────────────────────────────────┘
```

---

## API Backend

### Base URL

```
http://localhost:8000/api
```

### Endpoints

#### Colaboradores (`/api/colaboradores`)

| Método | Endpoint                          | Descripción                                   |
| ------ | --------------------------------- | --------------------------------------------- |
| GET    | `/api/colaboradores/`             | Listar paginado (skip, limit, id_area, search) |
| GET    | `/api/colaboradores/{id}/`        | Obtener uno por identificación                |
| POST   | `/api/colaboradores/`             | Crear (JSON, sin foto)                        |
| POST   | `/api/colaboradores/con-foto`     | Crear con foto (multipart/form-data)          |
| PUT    | `/api/colaboradores/{id}/`        | Actualizar datos                              |
| POST   | `/api/colaboradores/{id}/foto/`   | Subir/actualizar foto                         |
| DELETE | `/api/colaboradores/{id}/`        | Eliminar                                      |

#### Contactos de Emergencia (`/api/contactos-emergencia`)

| Método | Endpoint                           | Descripción                                   |
| ------ | ---------------------------------- | --------------------------------------------- |
| GET    | `/api/contactos-emergencia/`       | Listar paginado (skip, limit, id_area, search) |
| GET    | `/api/contactos-emergencia/{id}/`  | Obtener uno                                   |
| POST   | `/api/contactos-emergencia/`       | Crear                                         |
| PUT    | `/api/contactos-emergencia/{id}/`  | Actualizar                                    |
| DELETE | `/api/contactos-emergencia/{id}/`  | Eliminar                                      |

#### Áreas (`/api/areas`)

| Método | Endpoint            | Descripción  |
| ------ | ------------------- | ------------ |
| GET    | `/api/areas/`       | Listar todos |
| GET    | `/api/areas/{id}/`  | Obtener uno  |
| POST   | `/api/areas/`       | Crear        |
| PUT    | `/api/areas/{id}/`  | Actualizar   |
| DELETE | `/api/areas/{id}/`  | Eliminar     |

#### Configuración (`/api/configuracion`)

| Método | Endpoint              | Descripción           |
| ------ | --------------------- | --------------------- |
| GET    | `/api/configuracion/` | Obtener config actual |
| PUT    | `/api/configuracion/` | Actualizar config     |

#### Impresión (`/api/impresion`)

| Método | Endpoint                 | Descripción                         |
| ------ | ------------------------ | ----------------------------------- |
| GET    | `/api/impresion/carnets` | Obtener datos para imprimir carnets |

#### Emergencia (`/api/emergencia`)

| Método | Endpoint               | Descripción                           |
| ------ | ---------------------- | ------------------------------------- |
| GET    | `/api/emergencia/{id}` | Obtener datos de emergencia (PÚBLICO) |

#### Autenticación (`/api/auth`)

| Método | Endpoint               | Descripción                                    | Requiere Auth |
| ------ | ---------------------- | ---------------------------------------------- | ------------- |
| POST   | `/api/auth/login`      | Login (form-data)                              | No            |
| POST   | `/api/auth/login/json` | Login con JSON body                            | No            |
| POST   | `/api/auth/setup`      | Crear primer admin (solo si no hay usuarios)   | No            |
| GET    | `/api/auth/me`         | Obtener usuario actual                         | Sí            |
| PUT    | `/api/auth/me`         | Actualizar perfil propio                       | Sí            |
| POST   | `/api/auth/register`   | Registrar nuevo usuario                        | Sí (Admin)    |
| GET    | `/api/auth/users`      | Listar todos los usuarios                      | Sí (Admin)    |

---

## Frontend

### Rutas de la Aplicación

| Ruta                     | Descripción                            | Requiere Auth |
| ------------------------ | -------------------------------------- | ------------- |
| `/login`                 | Página de inicio de sesión             | No            |
| `/`                      | Dashboard con estadísticas             | Sí            |
| `/colaboradores`         | CRUD con paginación y búsqueda         | Sí            |
| `/contactos-emergencia`  | CRUD con paginación y búsqueda         | Sí            |
| `/areas`                 | CRUD de áreas                          | Sí            |
| `/carnets`               | Generación de carnets PDF              | Sí            |
| `/configuracion`         | Configuración global                   | Sí            |
| `/emergencia/[id]`       | Página pública de emergencia           | No            |

---

## Instalación y Configuración

### Requisitos Previos

- Docker y Docker Compose
- Git

### Pasos

1. **Clonar el repositorio**

```bash
git clone https://github.com/xFeDEV/sistema-carnetizacion.git
cd sistema-carnetizacion
```

2. **Iniciar los servicios**

```bash
docker compose up --build -d
```

3. **Verificar que todo esté corriendo**

```bash
docker compose ps
```

Deberías ver 3 contenedores:

- `feexel_db` (MariaDB) - Puerto 3306
- `feexel_backend` (FastAPI) - Puerto 8000
- `feexel_frontend` (Next.js) - Puerto 3000

4. **Acceder a la aplicación**

- Frontend: http://localhost:3000
- API Docs: http://localhost:8000/docs

### Comandos Útiles

```bash
# Ver logs de todos los servicios
docker compose logs -f

# Reiniciar un servicio
docker compose restart frontend

# Reconstruir un servicio
docker compose build --no-cache backend
docker compose up -d backend

# Detener todo
docker compose down

# Detener y eliminar volúmenes (¡BORRA DATOS!)
docker compose down -v
```

---

## Tecnologías

### Backend
| Tecnología | Propósito                     |
| ---------- | ----------------------------- |
| Python 3.12| Lenguaje principal            |
| FastAPI    | Framework API REST            |
| SQLAlchemy | ORM para base de datos        |
| Pydantic v2| Validación de datos           |
| PyMySQL    | Conector MySQL/MariaDB        |

### Frontend
| Tecnología       | Propósito                   |
| ---------------- | --------------------------- |
| Next.js 15+      | Framework React             |
| React 19         | Librería UI                 |
| TypeScript       | Tipado estático             |
| TanStack Query v5| Gestión de estado servidor  |
| Shadcn/UI        | Componentes UI              |
| Tailwind CSS v4  | Estilos                     |
| jsPDF            | Generación de PDF           |

---

## Licencia

Proyecto desarrollado por **Feexel - Software a la Medida**.

---

## Contribuidores

- Desarrollado por el equipo de Feexel