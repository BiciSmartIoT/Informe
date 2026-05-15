# 6.2. Landing Page, Services & Applications Implementation.
## 6.1. Software Configuration Management

Requirements Management

User Experience Design (UX/UI)

Project Managment

Discord y WhatsApp: Estas plataformas fueron esenciales para la comunicación interna del equipo, siendo WhatsApp especialmente útil por su facilidad para gestionar grupos de trabajo.

Trello: Se utilizó para planificar y dar seguimiento al avance del proyecto mediante tableros que representaban el backlog del producto y otras tareas organizativas.

Product UX/UI

Figma: Herramienta principal para el diseño de wireframes y prototipos, tanto en versiones de escritorio como móviles.

Miro: Apoyo en la creación de los escenarios mapping y escenario mapping en ambos casos para ambos segmentos del objetivo en el desarrollo del proyecto.

Software Development

Visual Studio Code: Editor principal utilizado para programar el landing page.

Github y Git bash: Se emplearon para el control de versiones y el desarrollo colaborativo del repositorio del proyecto.

3.HTML y CSS: Lenguajes fundamentales utilizados para la estructura (HTML) y el diseño visual (CSS) del landing page.

Software Documentation

Google Drive: Plataforma utilizada para el almacenamiento compartido de documentación e informes colaborativos.

Google Meets y Zoom: Se usó Google Meets más que nada para las videoconferencias de reunión del equipo y el Zoom para las grabaciones de las entrevistas, y las presentaciones del trabajo en el desarrollo de este.

LucidChart: Herramienta utilizada para diagramas de flujo y modelado visual del diseño de la aplicación, incluyendo diagramas de clases.

Structuriz: Permite la creación del modelo C4 en sus tres niveles (contexto, contenedores, componentes), también trabajado en conjunto con Visual Studio Code.

Vertabello: Se empleó para el diseño de la base de datos y sus respectivos diagramas lógicos.
## 6.2.1. Sprint 1
### 6.2.1.1. Sprint Planning 1.

El Sprint Planning 1 establece la organización inicial del equipo para
desarrollar el MVP de **BiciSmartIOT**. Se definen el objetivo del Sprint,
las User Stories que serán trabajadas, la capacidad del equipo y los
entregables esperados. Este proceso permite una planificación clara y el
alineamiento de todo el equipo hacia un mismo objetivo para la entrega de
valor.

| Campo | Información |
|---|---|
| **Sprint #** | Sprint 1 |
| **Date** | 2026-04-18 |
| **Time** | 05:00 PM |
| **Location** | Virtual (Discord y Zoom) |
| **Prepared By** | Oscar Espinoza, Stephano Landauri, Renzo Uribe, Cameron Bustamante, Fabrisio Belahoni |
| **Attendees (to planning meeting)** | Oscar Espinoza, Stephano Landauri, Renzo Uribe, Cameron Bustamante, Fabrisio Belahoni |
| **Sprint n − 1 Review Summary** | No aplica (Primer Sprint del proyecto BiciSmartIOT). |
| **Sprint n − 1 Retrospective Summary** | No aplica (Primer Sprint del proyecto BiciSmartIOT). |
| **Sprint n Goal** | Entregar la versión corregida y mejorada de los artefactos presentados previamente, e implementar y desplegar la primera versión del Landing Page y de las Frontend Web Applications.<br><br>**Sprint Goal:**<br>Our focus is on launching the first public deployment of BiciSmartIOT.<br>We believe it delivers the platform's first product visibility through a working Landing Page and a navigable Frontend Web Application.<br>This will be confirmed when visitors can browse the Landing Page, create an account, sign in, explore the bicycle catalog, view bicycle details, access their personal panel and consult the GPS tracking screen successfully. |
| **Sprint n Velocity** | 82 Story Points |
| **Sprint Goal & User Stories** | **User Stories:**<br>HU01, HU02, HU03, HU04, HU05, HU06, HU07, HU08, HU09, HU10, HU11, HU12, HU13, HU14, HU15, HU16, HU17, HU18, TS01, TS02, TS03, TS04, TS05, TS06, TS07, TS08 |
| **Sum of Story Points** | 82 Story Points |

### 6.2.1.2. Aspect Leaders and Collaborators.

| Team Member (Last Name, First Name) | GitHub Username | Landing Page | Web Application |
|---|---|---|---|
| Bustamante Leveau, Cameron Charlotte | Pendiente | C | C |
| Uribe Livia, Renzo Sebastián | Pendiente | C | C |
| Espinoza Quijandría, Oscar Leonardo | Pendiente | C | C |
| Landauri Preciado, Stephano Mayzron | Pendiente | C | C |
| Belahonia Miranda, Fabrisio | Pendiente | C | C |

### 6.2.1.3. Sprint Backlog 1.

## 5.2.1.3. Sprint Backlog 1

El Sprint Backlog 1 consolida todas las funcionalidades principales de
**BiciSmartIOT**, enfocándose en completar la primera experiencia de
usuario: registro, login, exploración del catálogo de bicicletas, detalle
de cada unidad, panel personal del usuario, seguimiento GPS en tiempo
real, dashboard del arrendador, sección de planes, testimonios y FAQ. En
paralelo, se consolida la versión corregida y mejorada de los artefactos
DDD presentados previamente.

| User Story | Work-Item / Task | Description | Estimation (Hours) | Assigned To | Status |
|---|---|---|---|---|---|
| HU01 | T1  | Crear sección Hero del Landing con CTA principal | 5 | Oscar (Landing) | Done |
| HU02 | T2  | Crear sección Planes de Renta con 3 cards | 4 | Oscar (Landing) | Done |
| HU03 | T3  | Crear sección Testimonios con carousel | 3 | Fabrisio (Animaciones) | Done |
| HU04 | T4  | Crear sección FAQ con acordeón | 3 | Renzo (Frontend) | Done |
| HU05 | T5  | Crear pantalla de Registro de cuenta | 5 | Cameron (Auth) | Done |
| HU06 | T6  | Crear pantalla de Iniciar Sesión | 5 | Cameron (Auth) | Done |
| HU07 | T7  | Crear pantalla de Catálogo de Bicicletas con grid | 6 | Renzo (Frontend) | Done |
| HU08 | T8  | Crear pantalla de Detalle de Bicicleta | 5 | Renzo (Frontend) | Done |
| HU09 | T9  | Crear Panel personal con métricas de actividad | 6 | Stephano (Dashboard) | Done |
| HU10 | T10 | Crear pantalla de Seguimiento GPS en tiempo real | 5 | Stephano (IoT) | Done |
| HU11 | T11 | Crear Dashboard de gestión del arrendador con gráficos | 6 | Stephano (Dashboard) | Done |
| HU12 | T12 | Crear prototipo Mobile responsive | 4 | Fabrisio (Animaciones) | Done |
| HU13 | T13 | Implementar Navbar y Footer globales | 3 | Renzo (Frontend) | Done |
| HU13 | T14 | Definir paleta de colores Kinetic y tipografía | 3 | Fabrisio (Diseño) | Done |
| HU13 | T15 | Crear biblioteca de componentes UI reutilizables | 5 | Fabrisio (Diseño) | Done |
| HU14 | T16 | Rehacer EventStorming en 7 pasos incrementales | 6 | Stephano (DDD) | Done |
| HU15 | T17 | Rehacer Candidate Context Discovery con 3 técnicas | 4 | Stephano (DDD) | Done |
| HU16 | T18 | Rehacer Domain Message Flows como Domain Storytelling | 5 | Stephano (DDD) | Done |
| HU17 | T19 | Crear 7 Bounded Context Canvases V5 | 6 | Stephano (DDD) | Done |
| HU18 | T20 | Documentar Context Mapping con patrones DDD | 4 | Oscar (DDD) | Done |
| TS01 | T21 | Configurar repositorio GitHub y branching strategy | 2 | Cameron (DevOps) | Done |
| TS02 | T22 | Configurar tokens de diseño y theme dark | 3 | Fabrisio (Diseño) | Done |
| TS03 | T23 | Configurar pipeline CI/CD con GitHub Actions | 4 | Cameron (DevOps) | Done |
| TS04 | T24 | Desplegar Landing Page en hosting | 2 | Cameron (DevOps) | Done |
| TS05 | T25 | Desplegar Frontend Web Application | 3 | Cameron (DevOps) | Done |
| TS06 | T26 | Configurar dominio + certificado SSL/HTTPS | 2 | Cameron (DevOps) | Done |
| TS07 | T27 | Documentar componentes UI en la guía de estilos | 4 | Fabrisio (Diseño) | Done |
| TS08 | T28 | Redactar documentación del Sprint 1 (informe) | 3 | Oscar (Documentación) | Done |

### 6.2.1.4. Development Evidence for Sprint Review.
### 6.2.1.5. Testing Suite Evidence for Sprint Review.
### 6.2.1.6. Execution Evidence for Sprint Review.

A continuación, se muestran las evidencias de ejecución de la landing page y de la primera versión de la aplicación web.

Landing Page:


Link de Video:https://youtu.be/g29aSgglfV0

App Web:

Link de Video:

### 6.2.1.7. Services Documentation Evidence for Sprint Review.

Para este sprint, no nos centramos en elaborar los servicios web, por lo tanto, no hay evidencia de documentación de dichos servicios.

### 6.2.1.8. Software Deployment Evidence for Sprint Review.

En este primer sprint, se desplegaron tanto la Landing Page como la primera versión de la aplicación web frontend utilizando Cloudflare Pages, un servicio de Cloudflare que permite publicar sitios web estáticos y aplicaciones frontend de manera rápida, segura y escalable. Para ambos despliegues, se vinculó el repositorio correspondiente desde GitHub hacia Cloudflare Pages, permitiendo que cada cambio realizado en la rama principal del proyecto pueda actualizarse automáticamente en el entorno desplegado. De esta manera, el equipo pudo contar con versiones públicas y accesibles del landing y del frontend para validar el avance del producto.

De manera general, el proceso de despliegue consiste en ingresar a Cloudflare, seleccionar la opción Pages, conectar la cuenta de GitHub y escoger el repositorio del proyecto. Luego, se configura la rama principal desde donde se realizará el despliegue, se define el comando de construcción correspondiente si el proyecto lo requiere, y se indica la carpeta de salida generada por el build. Finalmente, Cloudflare Pages ejecuta el proceso de despliegue y genera una URL pública para acceder al sitio. Este procedimiento fue aplicado tanto para la Landing Page como para la aplicación web frontend.

### 6.2.1.9. Team Collaboration Insights during Sprint.
