# � Glam 5.0 - Alquiler de Vestidos en Pereira

![Glam 5.0 Banner](/public/images/site-logo.png)

> **Objetivo 2026**: Posicionarnos como el **#1 en búsquedas de alquiler y venta de vestidos en Pereira**. Actualmente en el Top 8, ¡vamos por la cima! �

## � Descripción del Proyecto

Este repositorio aloja el sitio web moderno y optimizado de **Glam 5.0**, un negocio dedicado al alquiler y venta de vestidos exclusivos en Pereira, Risaralda. El proyecto ha sido desarrollado con un enfoque obsesivo en el **SEO (Search Engine Optimization)**, la velocidad de carga y una experiencia de usuario (UX) premium para captar clientes buscando vestidos de novia, quinceañera y grados.

Llevamos 1 año trabajando en esta plataforma para ofrecer la mejor vitrina digital de moda en la región.

## ✨ Características Principales

- **Catálogo Optimizado**: Secciones dedicadas para [Novias](/src/pages/vestidos/novia), [Quinceañeras](/src/pages/vestidos/quinceanera) y [Grados](/src/pages/vestidos/graduacion).
- **SEO Técnico Avanzado**:
    - Metaetiquetas dinámicas y optimizadas para CTR (`siteData.json.ts`).
    - Estructura semántica HTML5.
    - Sitemap XML y Robots.txt configurados automáticamente.
    - Open Graph para redes sociales.
- **Rendimiento Extremo**:
    - Construido con **Astro 5** para obtener puntuaciones 100/100 en Lighthouse.
    - Carga de imágenes optimizada.
    - Minificación de HTML, CSS y JS.
- **Diseño Responsive & Premium**:
    - Estilizado con **Tailwind CSS 4**.
    - Adaptable a todos los dispositivos móviles (Mobile First).

## �️ Stack Tecnológico

Este proyecto utiliza las tecnologías más modernas del desarrollo web para garantizar velocidad y escalabilidad:

- **Framework**: [Astro 5.3](https://astro.build/) (Enfoque en contenido y velocidad).
- **Estilos**: [Tailwind CSS 4](https://tailwindcss.com/) (Estilizado utilitario moderno).
- **Lenguaje**: [TypeScript](https://www.typescriptlang.org/) (Seguridad de tipos y mantenibilidad).
- **Despliegue**: [Vercel](https://vercel.com/) (Adapter Serverless para funciones dinámicas).
- **Iconos**: Astro Icon & Tabler Icons.

## � Estructura del Proyecto

```text
/
├── public/             # Archivos estáticos (imágenes, robots.txt)
├── src/
│   ├── assets/         # Assets procesados por Astro
│   ├── components/     # Componentes reutilizables (Hero, Galería, UI)
│   ├── config/         # Configuración global (Nav, SiteData SEO)
│   ├── data/           # Datos estáticos (Testimonios, Portafolio)
│   ├── layouts/        # Plantillas base (BaseLayout.astro)
│   ├── pages/          # Rutas del sitio (Inicio, Vestidos, Contacto)
│   └── styles/         # Estilos globales CSS
└── astro.config.mjs    # Configuración de Astro y Vercel
```

## � Instalación y Desarrollo

Sigue estos pasos para correr el proyecto localmente:

1.  **Clonar el repositorio:**
    ```bash
    git clone https://github.com/xFeDEV/vestidosGlam5pereira.git
    cd vestidosGlam5pereira
    ```

2.  **Instalar dependencias:**
    ```bash
    npm install
    # o si usas pnpm (recomendado)
    pnpm install
    ```

3.  **Iniciar servidor de desarrollo:**
    ```bash
    npm run dev
    ```
    El sitio estará disponible en `http://localhost:4321`.

## � Comandos Disponibles

| Comando | Acción |
| :--- | :--- |
| `npm run dev` | Inicia el servidor de desarrollo local. |
| `npm run build` | Compila el sitio para producción (SSR/Static). |
| `npm run preview` | Vista previa de la compilación local. |
| `npm run astro ...` | Ejecuta comandos del CLI de Astro (add, check, etc). |

## � Estrategia SEO (Road to Top 1)

Para lograr el objetivo de ser el #1 en Pereira, el código sigue estas directrices:
1.  **Palabras Clave**: Uso estratégico de "Alquiler de vestidos Pereira", "Vestidos de novia Risaralda", etc. en títulos y descripciones.
2.  **Velocidad**: Core Web Vitals optimizados para evitar penalizaciones de Google.
3.  **Datos Estructurados**: Implementación de Schema.org para negocios locales (pendiente/en proceso).
4.  **Accesibilidad**: Etiquetas ARIA y atributos alt en todas las imágenes.

---

Desarrollado con ❤️ para **Glam 5.0** - *La elegancia hecha vestido.*
