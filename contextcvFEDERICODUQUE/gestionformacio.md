# Gestión Formación Contenerización

Este proyecto es un sistema integral para la gestión de la formación, centros, programas y horarios, diseñado con una arquitectura moderna y contenerizada. Utiliza **FastAPI** para el backend, **React** con **Tailwind CSS** para el frontend, y **MariaDB** como base de datos, todo orquestado mediante **Docker Compose**.

## 🚀 Tecnologías

El proyecto está construido sobre las siguientes tecnologías:

- **Frontend**: 
  - [React 19](https://react.dev/)
  - [Vite](https://vitejs.dev/)
  - [Tailwind CSS](https://tailwindcss.com/)
  - [TailAdmin Template](https://tailadmin.com/)
- **Backend**: 
  - [FastAPI](https://fastapi.tiangolo.com/) (Python)
- **Base de Datos**: 
  - [MariaDB 11.7](https://mariadb.org/)
- **Infraestructura**: 
  - [Docker](https://www.docker.com/)
  - [Docker Compose](https://docs.docker.com/compose/)

## 📋 Funcionalidades Principales

El sistema permite la administración y gestión de:

- **Regionales y Centros de Formación**: Gestión de sedes y ubicaciones.
- **Programas de Formación**: Administración de currículos y competencias.
- **Usuarios y Roles**: Sistema de roles (Superadmin, Admin, Instructor) con autenticación JWT.
- **Ambientes de Formación**: Control de aulas y espacios físicos.
- **Grupos (Fichas)**: Gestión de grupos de aprendices.
- **Programación**: Asignación de horarios e instructores.

## 🛠️ Instalación y Despliegue

### Prerrequisitos

Asegúrate de tener instalados:
- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)

### Pasos para ejecutar

1. **Clonar el repositorio**:
   ```bash
   git clone <URL_DEL_REPOSITORIO>
   cd GestionFormacionContenerizacion
   ```

2. **Configurar variables de entorno**:
   Crea un archivo `.env` en la raíz del proyecto basado en `.env.example`. Asegúrate de definir las credenciales de base de datos y JWT.

   Ejemplo básico de `.env`:
   ```env
   DB_USER=usuario_gestion
   DB_PASSWORD=tu_contraseña_segura
   DB_NAME=gestion_formacion
   DB_PORT=3306
   MARIADB_ROOT_PASSWORD=root_password
   
   JWT_SECRET=tu_secreto_super_seguro
   JWT_ALGORITHM=HS256
   JWT_ACCESS_TOKEN_EXPIRE_MINUTES=30
   
   FRONTEND_URL=http://localhost
   ```

3. **Levantar los contenedores**:
   Ejecuta el siguiente comando para construir e iniciar los servicios:
   ```bash
   docker-compose up -d --build
   ```

4. **Verificar estado**:
   Asegúrate de que los contenedores `gestion_formacion_db`, `gestion_formacion_backend` y `gestion_formacion_frontend` estén corriendo:
   ```bash
   docker-compose ps
   ```

## 🌐 Acceso a la Aplicación

Una vez desplegado el sistema, puedes acceder a través de:

- **Frontend (Aplicación Web)**: [http://localhost:80](http://localhost:80)
- **Backend (Documentación API - Swagger)**: [http://localhost:8001/docs](http://localhost:8001/docs)
- **Backend (Documentación API - Redoc)**: [http://localhost:8001/redoc](http://localhost:8001/redoc)

## 🔐 Credenciales por Defecto

La base de datos se inicializa con los siguientes usuarios (ver `db/init.sql`):

| Rol | Correo | Contraseña |
| --- | --- | --- |
| **Super Admin** | `super@example.com` | `123456789` (hash bcrypt) |
| **Admin** | `admin@example.com` | `123456780` (hash bcrypt) |
| **Instructor** | `instru@example.com` | `123456781` (hash bcrypt) |
| **Super Admin Docker** | `superadmin@gestion.com` | `Admin123*` |

> **Nota**: Las contraseñas en la base de datos están hashbreadas. Para el usuario "Super Admin Docker", la contraseña en texto plano es `Admin123*`.

## 📂 Estructura del Proyecto

```
GestionFormacionContenerizacion/
├── .env                  # Variables de entorno globales
├── docker-compose.yml    # Orquestación de servicios Docker
├── db/
│   ├── init.sql          # Script de inicialización SQL (Schema y Seed)
│   └── ...
├── GestionFormacion/     # Código fuente del Backend (FastAPI)
│   ├── app/              # Lógica de la aplicación
│   ├── Dockerfile        # Configuración de imagen Backend
│   └── ...
└── GestionFormacionFrontEnd/ # Código fuente del Frontend (React)
    ├── src/              # Componentes y páginas React
    ├── Dockerfile        # Configuración de imagen Frontend
    └── ...
```

## 🤝 Contribución

Si deseas contribuir, por favor crea un *Fork* del repositorio y envía un *Pull Request* con tus mejoras.

## 📄 Licencia

Este proyecto es para fines educativos y de demostración.
