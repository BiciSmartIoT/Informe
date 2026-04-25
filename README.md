# 4.1. Strategic-Level Domain-Driven Design

En esta sección se aborda la perspectiva estratégica del enfoque Domain-Driven Design (DDD), la cual se centra en definir los límites del dominio y establecer una visión clara de cómo las diferentes partes del sistema interactúan entre sí. A través de técnicas como Event Storming, Context Mapping y la definición de una arquitectura de software adecuada, se busca garantizar que el diseño del sistema esté alineado con los objetivos del negocio y las necesidades de los usuarios.

## 4.1.1. Design-Level EventStorming

Para descubrir el dominio de **BiciSmartIOT** realizamos un *Design-Level
EventStorming* en una sesión virtual de Miro. A diferencia del Big Picture,
esta variante se enfoca en agregados, comandos, políticas y read models, por
lo que construimos el board de manera **incremental en 7 pasos**: cada paso
añade un nuevo tipo de sticky para enriquecer el modelo sin saturarlo, y al
final converge en los Bounded Contexts candidatos del sistema.

### Convenciones de color (notación clásica de EventStorming)

| Color    | Significado                                                       |
|----------|-------------------------------------------------------------------|
| Naranja  | **Evento de Dominio** — un hecho de negocio en pasado.            |
| Azul     | **Comando** — una intención que dispara un evento.                |
| Amarillo | **Actor** — quién dispara el comando (Estudiante, Arrendador, Sistema, Dispositivo IoT). |
| Lavanda  | **Agregado** — el objeto que custodia el invariante.              |
| Púrpura  | **Política** — regla reactiva del tipo "cuando ocurre X entonces Y". |
| Verde    | **Read Model** — vista de lectura que un actor consulta.          |
| Rosa     | **Sistema externo** — Yape, Plin, Niubiz, Google Maps, Firebase.  |

---

### Paso 1 — Pivotal Events

Comenzamos identificando los **eventos pivotales**: hitos de negocio que
articulan el dominio y ordenan cronológicamente toda la historia. Cada sticky
naranja corresponde a un hecho de negocio en pasado (Cuenta Creada, Bicicleta
Desbloqueada, Alquiler Iniciado, Robo Potencial Detectado, Bicicleta
Liberada, etc.).

![Paso 1 — Pivotal Events](assets/images/Chapter-4/Pivot.png)


---

### Paso 2 — Agregamos los Comandos

Sobre cada evento colocamos el **comando** (azul) que lo provocó. Los
comandos son intenciones explícitas: *Reservar Bicicleta*, *Desbloquear
Bicicleta*, *Autorizar Pago*. Esta etapa expone qué decisiones toma el
sistema y permite separar lo que el usuario pide de lo que el sistema hace.

![Paso 2 — Comandos](assets/images/Chapter-4/Comandos.png)


---

### Paso 3 — Agregamos los Actores

Identificamos **quién dispara cada comando**. Aparecen cuatro tipos de
actores en el dominio: el **Estudiante** (alquila), el **Arrendador**
(publica), el **Sistema** (orquestador automatizado) y el **Dispositivo
IoT** (cerradura + GPS). Esto nos permite distinguir qué partes del flujo
son automáticas y cuáles las inicia una persona.

![Paso 3 — Actores](assets/images/Chapter-4/Actores.png)


---

### Paso 4 — Agregamos las Políticas

Las **políticas** (púrpura) son reglas reactivas del tipo *"cuando ocurre X
entonces Y"*. Encadenan eventos con nuevos comandos:

- *Si stock < 5 → notificar arrendador.*
- *Al iniciar alquiler → activar tracking.*
- *Si fuera de zona segura → emitir alerta.*
- *Patrón sospechoso → alerta crítica.*
- *Pago OK → liberar bicicleta.*

Hacer las políticas explícitas es lo que convierte un EventStorming en una
verdadera radiografía del dominio.

![Paso 4 — Políticas](assets/images/Chapter-4/politicas.png)


---

### Paso 5 — Agregamos los Read Models

Sumamos las **vistas de lectura** (verde) que los actores consultan
**antes** de disparar comandos:

- **Catálogo de Bicicletas** — el estudiante busca antes de reservar.
- **Mapa en Tiempo Real** — el estudiante encuentra la bici más cercana.
- **Historial de Viajes** — el estudiante revisa sus alquileres pasados.

Distinguir read models del resto evita confundir lectura con escritura, una
fuente común de mal diseño.

![Paso 5 — Read Models](assets/images/Chapter-4/Read_Models.png)


---

### Paso 6 — Agregamos los Sistemas Externos

Los stickies rosa marcan las **dependencias externas** del dominio:

- **Yape / Plin / Visa Niubiz** — pasarelas de pago peruanas.
- **Google Maps / GPS Satélites** — geolocalización.
- **Firebase Cloud Messaging** — notificaciones push.

Documentar estas integraciones desde temprano nos evita acoplar el modelo
de dominio con detalles de infraestructura.

![Paso 6 — Sistemas Externos](assets/images/Chapter-4/Sistemas_Externos.png)


---

### Paso 7 — Agregados y Bounded Contexts

Por último agrupamos los stickies según el **agregado** (lavanda) que los
encapsula y trazamos las fronteras de los **Bounded Contexts** candidatos.
El board final muestra cómo el dominio se divide naturalmente en **7
contextos especializados**:

1. **IAM** — registro y validación de cuentas (Estudiante / Arrendador).
2. **Providing** — publicación de bicicletas, tarifas y zonas operativas.
3. **IoT & Device Control** — comandos al hardware (cerradura, GPS, OTA).
4. **Renting** — orquestador del alquiler end-to-end.
5. **Tracking & Monitoring** — telemetría GPS, geofencing y anti-robo.
6. **Payments** — autorización y procesamiento de pagos.
7. **Notifications** — entrega multicanal (push, email, SMS).

![Paso 7 — Agregados y Bounded Contexts](assets/images/Chapter-4/Agregados%20_Bounded_Contexts.png)

---
## 4.1.1.1 Candidate Context Discovery

En esta sección se describe el proceso seguido por el equipo para descubrir
los *bounded contexts* candidatos de **BiciSmartIOT** a partir del
Design-Level EventStorming. El objetivo fue identificar límites naturales
del dominio de alquiler de bicicletas con IoT integrado, priorizar las
partes con mayor valor estratégico para el negocio y preparar una base
clara para el modelado posterior (Domain Message Flows y Bounded Context
Canvases).

La sesión se realizó en Miro con una duración aproximada de **1 hora 50
minutos** (dentro del límite recomendado de 2 horas). Como insumos se
utilizaron la línea de tiempo de eventos pivotales, los comandos asociados,
los actores identificados y los puntos de dolor detectados en la sesión de
EventStorming anterior.

### Técnica aplicada: Start-with-Value (complementada con Look-for-Pivotal-Events)

Se priorizó **Start-with-Value** para ubicar primero los flujos que
diferencian a BiciSmartIOT en el mercado peruano: alquiler self-service
por minuto, protección anti-robo en tiempo real basada en GPS y
geofencing, y un marketplace que conecta a estudiantes con
arrendadores independientes. De forma complementaria, se revisaron
**eventos pivote** para detectar cambios de estado entre subdominios (por
ejemplo: pago autorizado, desbloqueo de bicicleta, salida de zona segura,
robo potencial detectado, devolución).

El proceso se organizó en tres pasos documentados:

1. **Identificación de valor estratégico:** cada miembro del equipo
   respondió a la pregunta: *"¿Qué parte del sistema genera directamente
   valor para los usuarios y diferencia la solución de otras similares?"*.

   ![Identificación de valor — post-its de valor estratégico](assets/images/Chapter-4/Value_Identification.png)

2. **Agrupación de eventos en torno al valor:** luego se agruparon los
   eventos por afinidad de negocio y cohesión funcional, emergiendo
   clústeres naturales alrededor de identidad y acceso, publicación de
   bicicletas, control de dispositivos IoT, alquiler, tracking
   anti-robo, pagos y notificaciones.

   ![Agrupación de eventos en clústeres por candidate context](assets/images/Chapter-4/Agregados%20_Bounded_Contexts.png)

3. **Clasificación Core, Supporting y Generic:** con los clústeres ya
   estabilizados, se clasificaron los *candidate contexts* según su aporte
   de diferenciación al negocio y su rol dentro del ecosistema. En esta
   fase se distinguieron los contextos núcleo que sostienen la propuesta
   de valor de BiciSmartIOT (alquiler self-service + anti-robo IoT), los
   contextos de soporte operacional y los contextos genéricos de
   plataforma.

   ![Matriz Core / Supporting / Generic](assets/images/Chapter-4/Core_Generic_Supporting.png)

Con esta tercera iteración, el equipo dejó una base consistente para pasar
al detalle de *candidate contexts* y su justificación estratégica en la
siguiente subsección.

### Candidate Contexts identificados

Como resultado de la sesión, se identificaron **siete bounded contexts
candidatos** para BiciSmartIOT:

| Candidate Context | Eventos clave asociados | Clasificación | Descripción | Justificación |
|---|---|---|---|---|
| **IAM** | `User Created`, `Email Verified`, `Renter Approved`, `Session Started` | Generic | Gestión de identidad, autenticación y acceso por rol (Estudiante / Arrendador). | Es transversal y necesario para operar, pero no representa diferenciación de negocio en el mercado de alquiler. |
| **Providing** | `Bicycle Published`, `Fare Defined`, `Operating Zone Set`, `Bicycle Paused` | Supporting | Publicación y mantenimiento del catálogo de bicicletas por parte de los arrendadores. | Habilita la oferta del marketplace, pero la lógica es estándar de e-commerce y no constituye el núcleo diferenciador. |
| **IoT & Device Control** | `Device Registered`, `Device Paired`, `Bicycle Unlocked`, `Bicycle Released`, `Bicycle Locked`, `Device Low Battery` | Core | Comandos al hardware (cerradura + GPS) y emparejamiento con la bicicleta vía MQTT/BLE. | Es indispensable para el modelo *self-service* y la calidad de captura de telemetría que sustenta toda la operación. |
| **Renting** | `Bicycle Reserved`, `Rental Started`, `Rental Finished`, `Reservation Expired` | Core | Orquestación del ciclo de vida del alquiler end-to-end. | Contiene la propuesta de valor principal de BiciSmartIOT: alquilar una bicicleta en segundos sin intervención humana. |
| **Tracking & Monitoring** | `Location Updated`, `Left Safe Zone`, `Suspicious Movement Detected`, `Theft Potential Detected` | Core | Procesamiento de telemetría GPS, geofencing y detección de patrones anómalos. | Es lo que diferencia a BiciSmartIOT del resto: protección anti-robo en tiempo real basada en datos. |
| **Payments** | `Payment Authorized`, `Payment Processed`, `Payment Failed`, `Refund Issued` | Supporting | Cobros y conexión con pasarelas peruanas (Yape, Plin, Visa Niubiz). | Es clave para monetización y activación del servicio, aunque no constituye el núcleo diferencial de la plataforma. |
| **Notifications** | `Notification Sent`, `Notification Confirmed`, `False Alarm Marked`, `Delivery Failed` | Generic | Mensajería multicanal: push, email y SMS para alertas e información al usuario. | Capacidad genérica de plataforma resuelta con servicios estándar (FCM, Twilio); no aporta diferenciación al negocio. |

### Clasificación estratégica

Como parte del análisis Start-with-Value, se representó gráficamente la
clasificación de los *bounded contexts* en una matriz de dos ejes:

- **Business Differentiation** (eje X): grado en que el contexto aporta
  valor estratégico o diferenciación frente a otras soluciones del
  mercado.
- **Model Complexity** (eje Y): nivel de complejidad requerido para
  implementar y mantener el contexto.

En esta matriz de clasificación de *bounded contexts*, se distribuyeron
los siete contextos en los tres tipos:

- **Core:** `Renting`, `IoT & Device Control`, `Tracking & Monitoring`.
- **Supporting:** `Providing`, `Payments`.
- **Generic:** `IAM`, `Notifications`.

![Clasificación estratégica de los Bounded Contexts](assets/images/Chapter-4/Classification_Matrix.png)

### Resultados

Se definieron **siete bounded contexts candidatos**, de los cuales:

- **3 Core** (`Renting`, `IoT & Device Control`, `Tracking & Monitoring`).
- **2 Supporting** (`Providing`, `Payments`).
- **2 Generic** (`IAM`, `Notifications`).

La aplicación de la técnica Start-with-Value permitió asegurar que la
atención principal del diseño táctico se concentre en `Renting`,
`IoT & Device Control` y `Tracking & Monitoring`, dado que allí reside la
propuesta de valor diferenciadora de BiciSmartIOT: alquiler self-service
de bicicletas con protección anti-robo en tiempo real.

El resto de *contexts* se modelan en las siguientes secciones mediante
**Domain Message Flows** y **Bounded Context Canvases**, garantizando
consistencia y claridad en la arquitectura estratégica.

## 4.1.1.2 Domain Message Flows Modeling

El **Domain Storytelling** es una técnica visual y colaborativa que facilita
la exploración del conocimiento dentro del dominio del negocio, cuyo
propósito principal es generar una comprensión común sobre lo que se
desarrolla en un proceso específico, involucrando tanto a los expertos del
negocio como a los equipos técnicos.

En este sentido, elaboramos los *domain storytelling* tomando como
referencia las interacciones entre los *bounded contexts* identificados en
**BiciSmartIOT**, con el fin de analizar y comprender de manera más clara
la lógica del negocio: alquiler self-service de bicicletas, integración con
hardware IoT y protección anti-robo en tiempo real.

> **Notación utilizada en todos los escenarios.**
> - **Persona** → actor (Estudiante, Arrendador, Administrador).
> - **Nube morada** → Bounded Context (Renting, Payments, IoT & Device
>   Control, Tracking, Notifications, IAM, Providing).
> - **Engranaje** → System (aplicación móvil, web, dispositivo físico,
>   pasarela externa).
> - **Flecha punteada numerada** → mensaje en orden cronológico (de emisor
>   a receptor).
> - **Sticky azul** → Command · **Naranja** → Event · **Verde** → Query.
> - **Caja amarilla** → contenido del mensaje (campos relevantes).

---

### Escenario 1: Arrendador se registra y publica su primera bicicleta

**Objetivo:** un dueño de flota se registra en la plataforma, valida su RUC,
publica una bicicleta y la enlaza con un dispositivo IoT para activar la
oferta en el marketplace.

![Escenario 1 — Arrendador se registra y publica](assets/images/Chapter-4/DS_01_arrendador_publica.png)

---

### Escenario 2: Estudiante se registra con correo  y recibe la bienvenida

**Objetivo:** un estudiante crea su cuenta usando su correo institucional, el sistema valida el dominio y se le envía una notificación de
bienvenida vía Firebase Cloud Messaging.

![Escenario 2 — Estudiante se registra con correo](assets/images/Chapter-4/DS_02_estudiante_registro.png)

---

### Escenario 3: Estudiante alquila una bicicleta end-to-end

**Objetivo:** el estudiante busca una bicicleta cercana, la reserva,
autoriza el pago en Yape/Plin/Visa Niubiz y el sistema desbloquea
automáticamente la cerradura IoT para iniciar el alquiler.

![Escenario 3 — Alquiler end-to-end](assets/images/Chapter-4/DS_03_alquiler_e2e.png)

---

### Escenario 4: Verificación de estado del dispositivo IoT antes del alquiler

**Objetivo:** antes de confirmar la reserva, el sistema asegura que el
dispositivo IoT acoplado a la bicicleta tiene batería suficiente y conexión
estable, garantizando la captura de datos en tiempo real durante el viaje.

![Escenario 4 — Verificación del dispositivo IoT](assets/images/Chapter-4/DS_04_verificacion_dispositivo.png)

---

### Escenario 5: Detección de robo en tiempo real y alerta crítica

**Objetivo:** el sistema procesa los pings GPS, detecta que la bicicleta
salió de la zona segura con un patrón sospechoso, dispara una alerta crítica
al arrendador y, tras la confirmación, bloquea remotamente la cerradura.

![Escenario 5 — Detección de robo en tiempo real](assets/images/Chapter-4/DS_05_deteccion_robo.png)

---

### Escenario 6: Finalización del alquiler, cobro final y devolución

**Objetivo:** el estudiante finaliza el alquiler, el dispositivo IoT bloquea
la bicicleta, *Payments* calcula el monto final, *Notifications* envía el
recibo y la plataforma libera la bicicleta para el siguiente alquiler.

![Escenario 6 — Finalización del alquiler](assets/images/Chapter-4/DS_06_finalizacion_alquiler.png)

---

> **Cierre.** Estos seis escenarios cubren los flujos críticos del dominio
> de BiciSmartIOT y ejercitan la totalidad de los siete *bounded contexts*
> identificados en la sección 4.1.1.1. La consistencia en la notación
> permite que esta sección sirva como insumo directo para los **Bounded
> Context Canvases** de la sección siguiente, donde cada contexto se
> documenta en detalle.

## 4.1.1.3 Bounded Context Canvases

En esta sección el equipo presenta sus *Bounded Context Canvases*,
empezando por los importantes. Cada canvas sigue el **template oficial V1
de DDD-Crew (Annegret Junker, codecentric AG)** y resume — en una sola
página — el propósito, la clasificación estratégica, los roles, los
mensajes que entran y salen, el lenguaje ubicuo, las decisiones de negocio,
los supuestos, las métricas de verificación y las preguntas abiertas de
cada *bounded context*.

---

### IAM

Contexto **genérico** para la identificación y acceso de los usuarios
(estudiantes y arrendadores).

![Bounded Context Canvas — IAM](assets/images/Chapter-4/BCC_iam.png)

---

### Providing

Contexto **supporting** que administra el catálogo de bicicletas, las
tarifas y las zonas operativas definidas por los arrendadores.

![Bounded Context Canvas — Providing](assets/images/Chapter-4/BCC_providing.png)

---

### IoT & Device Control

Contexto **core** que registra y opera los dispositivos IoT (cerradura +
GPS) acoplados a cada bicicleta vía MQTT/BLE.

![Bounded Context Canvas — IoT & Device Control](assets/images/Chapter-4/BCC_iot_device_control.png)

---

### Renting

Contexto **core** que orquesta el ciclo de vida del alquiler — buscar,
reservar, iniciar, monitorear y finalizar — y constituye el flujo
principal de valor de la plataforma.

![Bounded Context Canvas — Renting](assets/images/Chapter-4/BCC_renting.png)

---

### Tracking & Monitoring

Contexto **core** que procesa la telemetría GPS, evalúa geofences y
detecta patrones de movimiento sospechoso para disparar alertas anti-robo.

![Bounded Context Canvas — Tracking & Monitoring](assets/images/Chapter-4/BCC_tracking_monitoring.png)

---

### Payments

Contexto **supporting** que calcula cobros, autoriza pagos y procesa
reembolsos integrándose con pasarelas peruanas (Yape, Plin, Visa Niubiz).

![Bounded Context Canvas — Payments](assets/images/Chapter-4/BCC_payments.png)

---

### Notifications

Contexto **genérico** que entrega mensajes oportunos al estudiante y al
arrendador a través de push, SMS y email, gestionando confirmaciones y
falsas alarmas.

![Bounded Context Canvas — Notifications](assets/images/Chapter-4/BCC_notifications.png)

## 4.1.2. Context Mapping

En esta sección desarrollamos un conjunto de context maps para visualizar las relaciones entre los bounded contexts del sistema **BiciSmartIOT**. A partir de la información recolectada (user stories, funcionalidades IoT, pagos, reservas, etc.), exploramos distintas alternativas de diseño, evaluando cómo cambiaría la arquitectura al dividir, agrupar o reorganizar capacidades del sistema.

Durante el análisis se consideraron preguntas clave como:

- ¿Qué sucede si movemos funcionalidades IoT a otro contexto?
- ¿Qué pasa si unimos pagos con reservas?
- ¿Cómo evitar dependencias fuertes entre módulos?
- ¿Qué pasa si duplicamos modelos o usamos un shared service?

Finalmente, evaluamos cada alternativa usando patrones de Domain-Driven Design como:

- Customer/Supplier
- Conformist
- Shared Kernel
- Anti-Corruption Layer (ACL)

Con el objetivo de definir la mejor arquitectura del dominio.

---

###  Opción 1

En esta estructura se mantienen los bounded contexts completamente separados, cada uno enfocado en una funcionalidad específica del sistema.

#### Bounded Contexts
- Authentication & User Management
- Bicycle Management
- Booking & Reservation
- Payments & Billing
- IoT Monitoring

#### Relaciones
- User Management → Booking *(Customer/Supplier)*
- Bicycle Management → Booking *(Customer/Supplier)*
- Booking → Payments *(Customer/Supplier)*
- IoT Monitoring → Bicycle Management *(ACL)*

#### Ventajas
- Alta separación de responsabilidades  
- Escalabilidad independiente  
- Bajo acoplamiento  

#### Desventajas
- Mayor complejidad de integración  
- Sincronización entre contextos más difícil  
- Mayor overhead en comunicación  

---
![Opción 1](assets/images/Chapter-4/opcion_1.png)

### Opción 2

Esta alternativa propone unir los contextos de Bicycle Management + IoT Monitoring en un solo bounded context.

#### Nuevo Contexto
**Smart Bicycle Management (Bici + IoT)**

#### Relaciones
- User Management → Booking *(Customer/Supplier)*
- Smart Bicycle Management → Booking *(Customer/Supplier)*
- Booking → Payments *(Customer/Supplier)*

#### Ventajas
- Simplificación de arquitectura  
- Comunicación directa entre sensores IoT y bicicletas  
- Menor latencia en monitoreo  

#### Desventajas
- Mezcla de responsabilidades (hardware + lógica de negocio)  
- Mayor complejidad interna  
- Riesgo de crear un “mega contexto” difícil de mantener  

---
![Opción 2](assets/images/Chapter-4/opcion_2.png)

###  Opción 3 

Esta alternativa propone una arquitectura equilibrada con bounded contexts bien definidos y relaciones claras, separando correctamente responsabilidades críticas del sistema.

#### Bounded Contexts definidos

- **User Management**  
  Registro, login, perfil  

- **Bicycle Management**  
  Registro de bicicletas, disponibilidad  

- **Booking Management**  
  Reservas, inicio y fin de alquiler  

- **Payment Management**  
  Pagos digitales (Yape, Plin, tarjeta)  

- **IoT Monitoring**  
  Sensores, GPS, smart lock  

---
![Opción 3](assets/images/Chapter-4/opcion_3.png)

#### Relaciones entre contextos

 **1. User → Booking**  
 *Customer/Supplier*  
User provee información de usuarios y Booking la consume.

 **2. Bicycle → Booking**  
 *Shared Kernel*  
Comparten el modelo de bicicleta y disponibilidad, evitando inconsistencias.

 **3. Booking → Payment**  
 *Customer/Supplier*  
Booking envía información de alquiler para que Payment procese el cobro.

 **4. Booking → Bicycle**  
 *Conformist*  
Booking usa el modelo de Bicycle sin modificarlo.

 **5. IoT Monitoring → Bicycle**  
 *Anti-Corruption Layer (ACL)*  
Se usa una capa intermedia para traducir datos IoT (sensores, GPS) a un formato entendible por el sistema.

 Esto evita que la lógica de negocio dependa directamente del hardware.

---

#### Ventajas
- Separación clara de responsabilidades  
- Escalabilidad por módulos  
- Protección del dominio ante IoT (ACL)  
- Arquitectura mantenible y flexible  

#### Desventajas
- Requiere diseño inicial más cuidadoso  
- Más contratos entre contextos  

---

###  Elección

Elegimos la **Opción 3**, ya que proporciona el mejor equilibrio entre:

- Separación de responsabilidades  
- Escalabilidad del sistema  
- Facilidad de mantenimiento  
- Integración con IoT sin acoplar el dominio  

Al utilizar patrones como **Shared Kernel** y **Anti-Corruption Layer**, se logra una arquitectura robusta donde el sistema puede evolucionar sin afectar otros módulos.

Esto permite que **BiciSmartIOT** sea:

- Escalable  
- Seguro  
- Modular  
- Preparado para crecimiento futuro  

---

## 4.1.3. Software Architecture

### 4.1.3.1. Software Architecture System Landscape Diagram

Este diagrama muestra que BiciSmartIOT opera dentro de un ecosistema compuesto por tres tipos principales de usuarios: estudiantes, arrendadores y administradores. Los estudiantes interactúan con la aplicación para buscar bicicletas cercanas, realizar reservas, iniciar y finalizar alquileres, consultar rutas y realizar pagos digitales. Los arrendadores registran y administran sus bicicletas inteligentes, revisan disponibilidad, historial de alquileres y liquidaciones. Por otro lado, los administradores supervisan usuarios, bicicletas registradas, métricas del sistema y monitoreo IoT.

El sistema principal BiciSmartIOT App se conecta con servicios externos como Google Maps API para geolocalización y rutas, una pasarela de pagos para procesar Yape, Plin o tarjetas, y dispositivos IoT instalados en las bicicletas para obtener ubicación GPS, estado del smart lock, sensores de movimiento y telemetría en tiempo real.

<p align="center">
  <img src="assets/images/Chapter-4/system_landscape_diagram.png" width="700"/>
</p>

### 4.1.3.2. Software Architecture Context Level Diagram

Este diagrama muestra que el sistema BiciSmartIOT, representado como una única entidad, interactúa con tres tipos principales de usuarios: estudiantes, que utilizan la aplicación para buscar bicicletas cercanas, reservar, desbloquear y finalizar alquileres; arrendadores, que registran y administran sus bicicletas inteligentes; y administradores, que gestionan usuarios, bicicletas registradas y el monitoreo general del sistema. 

Además, BiciSmartIOT se comunica con sistemas externos como Google Maps API para mostrar ubicaciones y rutas, una pasarela de pagos para procesar pagos digitales mediante Yape, Plin o tarjeta, y dispositivos IoT instalados en las bicicletas para enviar datos en tiempo real como ubicación GPS, estado del candado inteligente y movimiento.

<p align="center">
  <img src="assets/images/Chapter-4/c4_context.png" width="700"/>
</p>

### 4.1.3.3. Software Architecture Container Level Diagram

Este diagrama muestra que el sistema BiciSmartIOT está compuesto por varios contenedores principales: una aplicación móvil utilizada por estudiantes y arrendadores, una aplicación web administrativa para la gestión del sistema, una API en la nube que centraliza la lógica de negocio, una base de datos que almacena usuarios, bicicletas, reservas, pagos y rutas, y un módulo IoT encargado de procesar la información proveniente de los dispositivos instalados en las bicicletas.

Además, la API se conecta con servicios externos como Google Maps API para mostrar ubicaciones, rutas y bicicletas cercanas, así como con una pasarela de pagos para procesar cobros digitales mediante Yape, Plin o tarjetas. Por otro lado, los dispositivos IoT se comunican mediante un broker MQTT para enviar datos de GPS, sensores de movimiento y estado del smart lock en tiempo real.

<p align="center">
  <img src="assets/images/Chapter-4/c4_container.png" width="700"/>
</p>

### 4.1.3.4. Software Architecture Deployment Diagram

Este diagrama muestra que el sistema BiciSmartIOT se despliega en tres entornos principales: Microsoft Azure Cloud, dispositivos cliente y bicicletas físicas con dispositivos IoT. En Azure, el sistema utiliza App Service para alojar la API principal desarrollada en Spring Boot/Java, una base de datos en Azure Database for MySQL para almacenar usuarios, bicicletas, reservas, pagos, rutas y eventos IoT, además de un servicio de documentación Swagger para probar los endpoints de la API.

Los usuarios acceden al sistema mediante una aplicación móvil Android, mientras que los administradores pueden acceder desde una aplicación web administrativa. Las bicicletas cuentan con dispositivos IoT integrados con GPS, sensores de movimiento y smart lock, los cuales envían datos en tiempo real mediante un broker MQTT o Azure IoT Hub. Todas las comunicaciones entre componentes utilizan protocolos seguros como HTTPS y MQTT/TLS.

<p align="center">
  <img src="assets/images/Chapter-4/c4_deploy.png" width="700"/>
</p>

<div id='4.2.'><h3>4.2. Tactical-Level Domain-Driven Design</h3></div>
<div id='4.2.1.'><h4>4.2.1. Bounded Context: Notifications-Encargado de alertas e información al usuario</h4></div>
<div id='4.2.1.1.'><h5>4.2.1.1. Domain Layer</h5></div>

| Tipo | Nombre | Descripción | Responsabilidad Principal | Relación con otros elementos |
|------|--------|-------------|--------------------------|------------------------------|
| Aggregate | `Notification` | Clase que representa una alerta generada hacia un usuario del sistema. | Ser el punto de entrada para crear, enviar y mantener la integridad del ciclo de vida de una notificación. | Relacionado con `UserNotificationProfile`, `AlertRule` y los demás bounded contexts que producen eventos IoT. |
| Aggregate | `UserNotificationProfile` | Clase que agrupa las preferencias y el historial de notificaciones de un usuario específico. | Gestionar y mantener la coherencia de las configuraciones de alerta y el registro de notificaciones de cada usuario. | Relacionado con `Notification` y `NotificationPreference`. |
| Entity | `NotificationPreference` | Entidad que almacena la configuración de alertas de un usuario (canales habilitados, horario silencioso, tipos activos). | Representar y validar las preferencias del usuario sobre cómo y cuándo recibir notificaciones. | Usada en `UserNotificationProfile` y por `NotificationDispatchService`. |
| Entity | `AlertRule` | Entidad que define bajo qué condición IoT se debe disparar una notificación (ej: movimiento superior a umbral). | Encapsular la lógica de evaluación de eventos IoT para determinar si se genera una alerta. | Usada por `AlertEvaluationService`; se activa a partir de eventos de los bounded contexts de IoT y Tracking. |
| Command | `SendNotificationCommand` | Comando para enviar una notificación a un usuario. | Representar la intención de despachar una alerta por el canal correspondiente. | Usado en `SendNotificationCommandHandler` dentro de la capa de aplicación. |
| Command | `MarkNotificationAsReadCommand` | Comando para marcar una notificación como leída. | Representar la intención del usuario de confirmar que vio la alerta. | Usado en `MarkNotificationAsReadCommandHandler` dentro de la capa de aplicación. |
| Command | `UpdateNotificationPreferencesCommand` | Comando para actualizar las preferencias de notificación de un usuario. | Representar la intención de modificar los canales, horarios y tipos de alerta habilitados. | Usado en `UpdateNotificationPreferencesCommandHandler` dentro de la capa de aplicación. |
| Command | `TriggerAlertCommand` | Comando que se genera al recibir un evento IoT y desencadena la evaluación y creación de una alerta. | Representar la intención de procesar un evento externo y convertirlo en una notificación si corresponde. | Consume eventos de los bounded contexts de IoT Device Management y Tracking & Security. |
| Value Object | `NotificationId` | Identificador único inmutable de una notificación. | Garantizar la unicidad de cada notificación dentro del sistema. | Usado en el aggregate `Notification`. |
| Value Object | `NotificationType` | Enum que define los tipos de alerta posibles: `THEFT_ALERT`, `GEOFENCE_BREACH`, `SOS_SIGNAL`, `LOW_BATTERY`, `INFO`. | Clasificar la naturaleza de la notificación para determinar su urgencia y canal de envío. | Usado en `Notification`, `AlertRule` y `NotificationDispatchService`. |
| Value Object | `NotificationChannel` | Enum que representa el canal de envío: `PUSH`, `SMS`, `IN_APP`, `EMAIL`. | Determinar el medio de comunicación por el que se entrega la alerta al usuario. | Usado en `Notification` y `NotificationPreference`. |
| Value Object | `NotificationStatus` | Enum del estado del ciclo de vida de una notificación: `PENDING`, `SENT`, `DELIVERED`, `READ`, `FAILED`. | Representar el estado actual de una notificación para controlar reintentos y confirmaciones. | Usado en `Notification`. |
| Value Object | `AlertSeverity` | Enum del nivel de urgencia de una alerta: `CRITICAL`, `HIGH`, `MEDIUM`, `LOW`. | Priorizar el canal de envío y la urgencia de la notificación según la severidad del evento IoT. | Usado en `AlertRule`, `Notification` y `NotificationDispatchService`. |
| Value Object | `RecipientInfo` | Objeto que agrupa los datos del destinatario: userId, deviceToken y phoneNumber. | Encapsular de forma inmutable los datos necesarios para identificar y contactar al usuario receptor. | Usado en `Notification` y por los adaptadores de infraestructura (FCM, Twilio). |
| Domain Event | `AlertTriggered` | Evento publicado cuando una `AlertRule` es activada por un evento IoT externo. | Notificar al sistema que se debe iniciar el proceso de envío de una alerta. | Consumido por `AlertProcessingService` en la capa de aplicación. |
| Domain Event | `NotificationSent` | Evento publicado cuando una notificación es enviada exitosamente al canal seleccionado. | Registrar el éxito del envío para auditoría y confirmación del flujo. | Puede ser consumido por otros bounded contexts para trazabilidad. |
| Domain Event | `NotificationFailed` | Evento publicado cuando el envío falla tras los reintentos configurados. | Registrar el fallo para activar lógica de reintento o escalación. | Consumido por la infraestructura de reintentos y logging. |
| Domain Event | `UserAlertAcknowledged` | Evento publicado cuando el usuario confirma haber visto la alerta desde la app. | Cerrar el ciclo de vida de la notificación y actualizar su estado a `READ`. | Consumido por `MarkNotificationAsReadCommandHandler`. |
| Domain Service | `AlertEvaluationService` | Servicio que evalúa los eventos IoT entrantes contra las `AlertRule` definidas. | Determinar si un evento IoT debe generar una notificación y de qué tipo y severidad. | Usa `AlertRule`; es invocado por `TriggerAlertCommandHandler`. |
| Domain Service | `NotificationDispatchService` | Servicio que selecciona el canal de envío óptimo y construye el mensaje de la notificación. | Decidir el canal correcto según la `AlertSeverity` y las `NotificationPreference` del usuario. | Usa `NotificationPreference`, `AlertSeverity` y `NotificationChannel`; invocado por `SendNotificationCommandHandler`. |

<div id='4.2.1.2.'><h5>4.2.1.2. Interface Layer</h5></div>

| Tipo | Nombre | Descripción | Responsabilidad Principal | Relación con otros elementos |
|------|--------|-------------|--------------------------|------------------------------|
| Controller | `NotificationController` | Controlador REST que expone los endpoints del historial de notificaciones del usuario. | Recibir y responder las peticiones HTTP del cliente móvil sobre notificaciones. | Delega en `NotificationApplicationService`; expone `GET /api/v1/notifications` y `GET /api/v1/notifications/{id}`. |
| Controller | `NotificationPreferenceController` | Controlador REST que gestiona las preferencias de notificación del usuario. | Permitir al usuario consultar y actualizar sus configuraciones de alerta. | Delega en `NotificationApplicationService`; expone `GET` y `PUT /api/v1/notifications/preferences`. |
| Event Consumer | `IoTMotionEventConsumer` | Suscriptor al evento `MotionDetected` publicado por el bounded context de dispositivos IoT. | Recibir el evento de movimiento sospechoso y despachar el `TriggerAlertCommand`. | Origen: Bounded Context IoT Device Management. Publica hacia la capa de aplicación. |
| Event Consumer | `GeofenceBreachConsumer` | Suscriptor al evento `GeofenceBreach` publicado por el bounded context de Tracking & Security. | Recibir el evento de salida de geocerca y despachar el `TriggerAlertCommand`. | Origen: Bounded Context Tracking & Security. Publica hacia la capa de aplicación. |
| Event Consumer | `SOSSignalConsumer` | Suscriptor al evento `SOSActivated` publicado por el dispositivo IoT ante detección de impacto o caída. | Recibir la señal de emergencia y despachar el `TriggerAlertCommand` con severidad `CRITICAL`. | Origen: Bounded Context IoT Device Management. Publica hacia la capa de aplicación. |
| Event Consumer | `LowBatteryConsumer` | Suscriptor al evento `LowBatteryDetected` publicado por el dispositivo IoT. | Recibir el aviso de batería baja y generar una notificación informativa al usuario. | Origen: Bounded Context IoT Device Management. Publica hacia la capa de aplicación. |
| DTO | `NotificationResponse` | Objeto de transferencia con los datos de una notificación: id, tipo, mensaje, canal, estado y timestamp. | Serializar la información de una notificación para enviarla como respuesta al cliente. | Usado en `NotificationController`. |
| DTO | `NotificationListResponse` | Objeto de transferencia con la lista paginada de notificaciones y su metadata. | Encapsular la respuesta del historial de notificaciones del usuario. | Usado en `NotificationController`. |
| DTO | `NotificationPreferenceRequest` | Payload de entrada para actualizar preferencias: canales habilitados, modo silencioso y tipos de alerta activos. | Transportar los datos de configuración del usuario hacia la capa de aplicación. | Usado en `NotificationPreferenceController`. |

<div id='4.2.1.3.'><h5>4.2.1.3. Application Layer</h5></div>

| Tipo | Nombre | Descripción | Responsabilidad Principal | Relación con otros elementos |
|------|--------|-------------|--------------------------|------------------------------|
| Application Service | `NotificationApplicationService` | Servicio de aplicación que actúa como punto de entrada principal para los casos de uso del bounded context. | Orquestar la ejecución de commands y queries coordinando el dominio con la infraestructura. | Invocado por los controllers de la Interface Layer; delega en los command y query handlers. |
| Application Service | `AlertProcessingService` | Servicio de aplicación que recibe eventos IoT y coordina su evaluación y conversión en notificaciones. | Procesar eventos externos, invocar `AlertEvaluationService` y despachar el `SendNotificationCommand` si aplica. | Invocado por los Event Consumers; usa `AlertEvaluationService` del Domain Layer. |
| Command Handler | `SendNotificationCommandHandler` | Handler que orquesta la creación y envío de una notificación. | Invocar `NotificationDispatchService`, persistir la `Notification` y publicar el evento `NotificationSent`. | Usa `NotificationDispatchService`, `NotificationRepository` y el publicador de eventos. |
| Command Handler | `MarkNotificationAsReadCommandHandler` | Handler que actualiza el estado de una notificación a `READ`. | Recuperar la notificación, actualizarla y persistir el cambio, publicando `UserAlertAcknowledged`. | Usa `NotificationRepository` y el publicador de eventos. |
| Command Handler | `UpdateNotificationPreferencesCommandHandler` | Handler que persiste las nuevas preferencias de notificación del usuario. | Recuperar el `UserNotificationProfile`, actualizar las preferencias y persistirlas. | Usa `NotificationPreferenceRepository`. |
| Command Handler | `TriggerAlertCommandHandler` | Handler que recibe un evento IoT, lo evalúa y, si corresponde, despacha el `SendNotificationCommand`. | Coordinar la evaluación de la alerta con el dominio y delegar el envío al handler correspondiente. | Usa `AlertEvaluationService`, `AlertRuleRepository` y despacha `SendNotificationCommand`. |
| Query Handler | `GetNotificationHistoryQueryHandler` | Handler que recupera el historial paginado de notificaciones de un usuario. | Consultar el repositorio y devolver la lista de notificaciones formateada. | Usa `NotificationRepository`. |
| Query Handler | `GetNotificationPreferencesQueryHandler` | Handler que recupera las preferencias de notificación del usuario autenticado. | Consultar el repositorio de preferencias y devolver la configuración activa. | Usa `NotificationPreferenceRepository`. |
| Query Handler | `GetNotificationDetailQueryHandler` | Handler que recupera el detalle completo de una notificación específica por su id. | Buscar la notificación en el repositorio y devolverla como respuesta. | Usa `NotificationRepository`. |

<div id='4.2.1.4.'><h5>4.2.1.4. Infrastructure Layer</h5></div>

| Tipo | Nombre | Descripción | Responsabilidad Principal | Relación con otros elementos |
|------|--------|-------------|--------------------------|------------------------------|
| Repository | `NotificationRepositoryImpl` | Implementación del repositorio de notificaciones usando PostgreSQL y JPA. | Persistir, actualizar y recuperar entidades `Notification` de la base de datos. | Implementa la interfaz del dominio; usado por los command y query handlers. |
| Repository | `NotificationPreferenceRepositoryImpl` | Implementación del repositorio de preferencias de notificación usando PostgreSQL y JPA. | Persistir y recuperar entidades `NotificationPreference` de la base de datos. | Implementa la interfaz del dominio; usado por `UpdateNotificationPreferencesCommandHandler`. |
| Repository | `AlertRuleRepositoryImpl` | Implementación del repositorio de reglas de alerta usando PostgreSQL y JPA. | Persistir y recuperar entidades `AlertRule` para la evaluación de eventos IoT. | Implementa la interfaz del dominio; usado por `AlertEvaluationService`. |
| External Adapter | `FirebasePushNotificationAdapter` | Adaptador que integra Firebase Cloud Messaging (FCM) para el envío de notificaciones push. | Enviar notificaciones push a dispositivos Android e iOS mediante FCM. | Usado por `NotificationDispatchService` cuando el canal es `PUSH`. |
| External Adapter | `TwilioSmsAdapter` | Adaptador que integra la API de Twilio para el envío de alertas por SMS. | Enviar mensajes de texto ante alertas críticas como `THEFT_ALERT` o `SOS_SIGNAL`. | Usado por `NotificationDispatchService` cuando el canal es `SMS`. |
| External Adapter | `SendGridEmailAdapter` | Adaptador que integra SendGrid para el envío de notificaciones por correo electrónico. | Enviar resúmenes o reportes de actividad al usuario por email. | Usado por `NotificationDispatchService` cuando el canal es `EMAIL`. |
| Message Consumer | `IoTEventMessageConsumer` | Componente suscriptor al broker de mensajería (Kafka/RabbitMQ) para recibir eventos IoT. | Consumir los topics de eventos provenientes del bounded context de dispositivos y enrutar a los consumers de la Interface Layer. | Se conecta al broker externo; alimenta a `IoTMotionEventConsumer`, `SOSSignalConsumer` y `LowBatteryConsumer`. |
| Message Publisher | `NotificationEventPublisher` | Componente publicador al broker de mensajería que emite domain events del bounded context. | Publicar eventos como `NotificationSent` y `AlertTriggered` para que otros bounded contexts los consuman. | Usado por los command handlers tras completar una operación; publica al broker Kafka/RabbitMQ. |

<div id='4.2.1.5.'><h5>4.2.1.5. Bounded Context Software Architecture Component Level Diagrams</h5></div>

<div id='4.2.1.6.'><h5>4.2.1.6. Bounded Context Software Architecture Code Level Diagrams</h5></div>

<div align="center">
<img src="assets/images/Chapter-4/Component-Diagram-BoundedNotifications.jpeg">
</div>

<div id='4.2.1.6.1.'><h6>4.2.1.6.1. Bounded Context Domain Layer Class Diagrams</h6></div>

<div align="center">
<img src="assets/images/Chapter-4/Domain-Layer-Class-Diagram-BoundedNotifications.jpeg">
</div>

<div id='4.2.1.6.2.'><h6>4.2.1.6.2. Bounded Context Database Design Diagram</h6></div>

<div align="center">
<img src="assets/images/Chapter-4/Database-Design-Diagram-BoundedNotifications.jpeg">
</div>


<div id='4.2.1.'><h4>4.2.1. Bounded Context: Renting</h4></div>
<div id='4.2.1.1.'><h5>4.2.1.1. Domain Layer</h5></div>
Subcapamodel<br>

| Tipo         | Nombre                    | Descripción                                                  | Responsabilidad Principal                             | Relación con otros elementos                                |
| ------------ | ------------------------- | ------------------------------------------------------------ | ----------------------------------------------------- | ----------------------------------------------------------- |
| Aggregate    | Rental                    | Representa un alquiler activo o finalizado de una bicicleta. | Gestionar el ciclo de vida del alquiler.              | Relacionado con Reservation, Payment y Bicycle (Providing). |
| Aggregate    | Reservation               | Representa la reserva previa de una bicicleta.               | Bloquear temporalmente una bicicleta para un usuario. | Relacionado con Rental y User (IAM).                        |
| Value Object | RentalTime                | Intervalo de tiempo del alquiler.                            | Calcular duración del alquiler.                       | Asociado a Rental.                                          |
| Value Object | RentalStatus              | Estado del alquiler (activo, finalizado, cancelado).         | Controlar el flujo del alquiler.                      | Asociado a Rental.                                          |
| Value Object | ReservationStatus         | Estado de la reserva.                                        | Controlar disponibilidad previa.                      | Asociado a Reservation.                                     |
| Command      | CreateReservationCommand  | Crear una reserva de bicicleta.                              | Registrar una nueva reserva.                          | Usa Reservation y User.                                     |
| Command      | CancelReservationCommand  | Cancelar una reserva.                                        | Liberar bicicleta reservada.                          | Usa Reservation.                                            |
| Command      | StartRentalCommand        | Iniciar alquiler.                                            | Activar el conteo de tiempo.                          | Usa Rental y IoT.                                           |
| Command      | EndRentalCommand          | Finalizar alquiler.                                          | Cerrar alquiler y calcular costo.                     | Usa Rental y Payments.                                      |
| Query        | GetAvailableBicyclesQuery | Obtener bicicletas disponibles.                              | Mostrar opciones al usuario.                          | Consulta Providing.                                         |
| Query        | GetUserRentalsQuery       | Obtener historial de alquileres.                             | Consultar uso del usuario.                            | Consulta Rental.                                            |
| Query        | GetReservationDetailQuery | Obtener detalle de reserva.                                  | Mostrar información específica.                       | Consulta Reservation.                                       |

Sub-capa Services<br>

| Tipo      | Nombre                    | Descripción                                        | Responsabilidad Principal                            | Relación con otros elementos                                              |
| --------- | ------------------------- | -------------------------------------------------- | ---------------------------------------------------- | ------------------------------------------------------------------------- |
| Interface | RentalCommandService      | Servicio para comandos relacionados con alquileres | Declarar métodos para iniciar y finalizar alquileres | Implementado por RentalCommandServiceImpl. Usado en capa Application      |
| Interface | ReservationCommandService | Servicio para comandos relacionados con reservas   | Declarar métodos para crear y cancelar reservas      | Implementado por ReservationCommandServiceImpl. Usado en capa Application |
| Interface | RentalQueryService        | Servicio para consultas de alquileres              | Declarar métodos para obtener alquileres             | Implementado por RentalQueryServiceImpl. Usado en capa Application        |
| Interface | ReservationQueryService   | Servicio para consultas de reservas                | Declarar métodos para obtener reservas               | Implementado por ReservationQueryServiceImpl. Usado en capa Application   |

<div id='4.2.1.1.'><h5>4.2.1.2. Interface Layer</h5></div>

Sub-capa REST<br>

| Tipo       | Nombre                                        | Descripción                                                  | Responsabilidad Principal                                                      | Relación con otros elementos                                       |
| ---------- | --------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------------------------ | ------------------------------------------------------------------ |
| Controller | RentalController                              | Controlador REST para gestionar alquileres                   | Recibe solicitudes del cliente relacionadas con alquileres y coordina comandos | Utiliza RentalRequestResource, RentalResponseResource y assemblers |
| Controller | ReservationController                         | Controlador REST para gestionar reservas                     | Maneja solicitudes de creación y cancelación de reservas                       | Utiliza ReservationRequestResource y ReservationResponseResource   |
| Resource   | RentalRequestResource                         | Estructura de una petición para iniciar o finalizar alquiler | Representa datos de entrada del cliente                                        | Usado por RentalController                                         |
| Resource   | RentalResponseResource                        | Estructura de respuesta de alquiler                          | Devuelve datos del alquiler al cliente                                         | Usado por RentalController                                         |
| Resource   | ReservationRequestResource                    | Estructura de petición para crear reserva                    | Representa datos de entrada                                                    | Usado por ReservationController                                    |
| Resource   | ReservationResponseResource                   | Estructura de respuesta de reserva                           | Devuelve datos al cliente                                                      | Usado por ReservationController                                    |
| Assembler  | CreateReservationCommandFromResourceAssembler | Convierte request en comando de reserva                      | Traducir entrada a comando de dominio                                          | Usado por ReservationController                                    |
| Assembler  | RentalResourceFromEntityAssembler             | Convierte entidad Rental en respuesta                        | Traducir dominio a respuesta                                                   | Usado por RentalController                                         |

<div id='4.2.1.1.'><h5>4.2.1.3. Aplications Layer</h5></div>

Sub-capa Internal

| Tipo    | Nombre                        | Descripción                                             | Responsabilidad Principal                   | Relación con otros elementos                                  |
| ------- | ----------------------------- | ------------------------------------------------------- | ------------------------------------------- | ------------------------------------------------------------- |
| Service | RentalCommandServiceImpl      | Implementación del servicio de comandos para alquileres | Ejecutar lógica de inicio y fin de alquiler | Implementa RentalCommandService. Usa entidades y repositorios |
| Service | ReservationCommandServiceImpl | Implementación del servicio de reservas                 | Ejecutar lógica de creación y cancelación   | Implementa ReservationCommandService                          |
| Service | RentalQueryServiceImpl        | Implementación de consultas de alquiler                 | Obtener historial y datos                   | Implementa RentalQueryService                                 |
| Service | ReservationQueryServiceImpl   | Implementación de consultas de reservas                 | Obtener información de reservas             | Implementa ReservationQueryService                            |

<div id='4.2.1.1.'><h5>4.2.1.4. Infrastructure Layer</h5></div>


Sub-capa Infrastructure <br>

| Tipo       | Nombre                | Descripción                           | Responsabilidad Principal         | Relación con otros elementos |
| ---------- | --------------------- | ------------------------------------- | --------------------------------- | ---------------------------- |
| Repository | RentalRepository      | Repositorio para gestionar alquileres | Persistencia de datos de alquiler | Relacionado con Rental       |
| Repository | ReservationRepository | Repositorio para gestionar reservas   | Persistencia de reservas          | Relacionado con Reservation  |


<div id='4.2.1.1.'><h5>4.2.1.5. Bounded Context Software Architecture Component Level Diagrams</h5></div>
<div align="center">
<img src="assets/images/Chapter-4/Bounded Context Software Architecture Component Level Diagrams_renting.jpeg">
</div>


<div id='4.2.1.1.'><h5>4.2.1.6. Bounded Context Software Architecture Code Level Diagram</h5></div>

<div id='4.2.1.1.'><h5>4.2.1.6.1 Bounded Context Domain Layer Class Diagrams</h5></div>

Este diagrama UML representa la arquitectura del Bounded Context Renting, centrado en la gestión del proceso de alquiler de bicicletas. El modelo sigue principios de diseño orientado a objetos y adopta un enfoque basado en DDD (Domain-Driven Design) junto con CQRS, separando las operaciones de comandos y consultas.

En el núcleo del dominio se encuentran los agregados principales como Rental y Reservation, responsables de gestionar el ciclo de vida del alquiler y las reservas previas. Además, se utilizan Value Objects como RentalTime, RentalStatus y ReservationStatus, que permiten representar el estado y duración del alquiler de forma clara y consistente.

<div align="center">
<img src="assets/images/Chapter-4/Bounded Context Domain Layer Class Diagrams_renting.png">
</div>

<div id='4.2.1.1.'><h5>4.2.1.6.2 Bounded Context Database Design Diagram</h5></div>

<div align="center">
<img src="assets/images/Chapter-4/Bounded Context Database Design Diagram_renting.jpeg">
</div>

### RENTALS

**Propósito:** Registro de alquileres de bicicletas

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | Long (PK) | Identificador único del alquiler |
| `user_id` | Long (FK) | Usuario que realiza el alquiler (IAM) |
| `bicycle_id` | Long (FK) | Bicicleta alquilada (Providing) |
| `start_time` | datetime | Inicio del alquiler |
| `end_time` | datetime | Fin del alquiler |
| `status` | varchar | Estado (`ACTIVE`, `FINISHED`, `CANCELLED`) |
| `total_cost` | decimal | Costo total calculado |

**Relaciones:**
- `N:1` con **USERS** (IAM)
- `N:1` con **BICYCLES** (Providing)
- `1:1` opcional con **RESERVATIONS**


### RESERVATIONS


**Propósito:** Reservas previas de bicicletas

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | Long (PK) | Identificador único |
| `user_id` | Long (FK) | Usuario que reserva |
| `bicycle_id` | Long (FK) | Bicicleta reservada |
| `status` | varchar | Estado (`ACTIVE`, `CANCELLED`, `EXPIRED`) |
| `reserved_at` | datetime | Fecha de reserva |
| `expiration_time` | datetime | Tiempo límite de la reserva |

**Relaciones:**
- `N:1` con **USERS** (IAM)
- `N:1` con **BICYCLES** (Providing)
- `1:1` con **RENTALS** (cuando se convierte)



###  PAYMENTS

**Propósito:** Procesamiento de pagos de alquileres

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | Long (PK) | Identificador |
| `rental_id` | Long (FK) | Alquiler asociado |
| `amount` | decimal | Monto |
| `status` | varchar | Estado del pago |
| `payment_date` | datetime | Fecha |

**Relaciones:**
- `1:1` con **RENTALS**

----------------------------------------------------------------
<div id='4.2.2.'><h4>4.2.2. Bounded Context: Payments</h4></div>

<div id='4.2.2.1.'><h5>4.2.2.1. Domain Layer</h5></div>

Subcapamodel<br>

| Tipo         | Nombre                  | Descripción                                      | Responsabilidad Principal                         | Relación con otros elementos          |
| ------------ | ----------------------- | ------------------------------------------------ | ------------------------------------------------- | ------------------------------------- |
| Aggregate    | Payment                 | Representa un pago de un alquiler                | Gestionar el proceso de pago                      | Relacionado con Rental (Renting)      |
| Aggregate    | PaymentMethod           | Método de pago del usuario                       | Gestionar métodos de pago                         | Relacionado con User (IAM)            |
| Value Object | PaymentAmount           | Monto del pago                                  | Representar valor monetario                       | Asociado a Payment                    |
| Value Object | PaymentStatus           | Estado del pago                                 | Controlar flujo del pago                          | Asociado a Payment                    |
| Value Object | PaymentType             | Tipo de método de pago                          | Clasificar método                                 | Asociado a PaymentMethod              |
| Command      | CreatePaymentCommand    | Crear pago                                      | Registrar un nuevo pago                           | Usa Payment y Rental                  |
| Command      | ProcessPaymentCommand   | Procesar pago                                   | Validar y confirmar pago                          | Usa PaymentMethod                     |
| Query        | GetUserPaymentsQuery    | Obtener pagos del usuario                       | Consultar historial                               | Consulta Payment                      |
| Query        | GetPaymentDetailQuery   | Obtener detalle de pago                         | Mostrar información específica                    | Consulta Payment                      |

---

Sub-capa Services<br>

| Tipo      | Nombre                | Descripción                           | Responsabilidad Principal                      | Relación con otros elementos             |
| --------- | --------------------- | ------------------------------------- | ---------------------------------------------- | ---------------------------------------- |
| Interface | PaymentCommandService | Servicio de comandos de pagos         | Declarar operaciones de creación y proceso     | Implementado por PaymentCommandServiceImpl |
| Interface | PaymentQueryService   | Servicio de consultas de pagos        | Declarar operaciones de consulta               | Implementado por PaymentQueryServiceImpl |

---

<div id='4.2.2.2.'><h5>4.2.2.2. Interface Layer</h5></div>

Sub-capa REST<br>

| Tipo       | Nombre                               | Descripción                                | Responsabilidad Principal                        | Relación con otros elementos           |
| ---------- | ------------------------------------ | ------------------------------------------ | ------------------------------------------------ | -------------------------------------- |
| Controller | PaymentController                    | Controlador REST de pagos                  | Gestionar solicitudes de pagos                   | Usa services y resources               |
| Resource   | PaymentRequestResource               | Petición de pago                           | Representar datos de entrada                     | Usado por PaymentController            |
| Resource   | PaymentResponseResource              | Respuesta de pago                          | Devolver información al cliente                  | Usado por PaymentController            |
| Resource   | PaymentMethodRequestResource         | Petición de método de pago                 | Representar datos de entrada                     | Usado por PaymentController            |
| Resource   | PaymentMethodResponseResource        | Respuesta de método de pago                | Devolver datos al cliente                        | Usado por PaymentController            |
| Assembler  | PaymentCommandFromResourceAssembler  | Convierte request a comando                | Traducir entrada a dominio                       | Usado por PaymentController            |
| Assembler  | PaymentResourceFromEntityAssembler   | Convierte entidad a response               | Traducir dominio a salida                        | Usado por PaymentController            |

---

<div id='4.2.2.3.'><h5>4.2.2.3. Application Layer</h5></div>

Sub-capa Internal<br>

| Tipo    | Nombre                    | Descripción                                 | Responsabilidad Principal                 | Relación con otros elementos              |
| ------- | ------------------------- | ------------------------------------------- | ----------------------------------------- | ----------------------------------------- |
| Service | PaymentCommandServiceImpl | Implementación de comandos de pago          | Ejecutar lógica de creación y procesamiento | Implementa PaymentCommandService          |
| Service | PaymentQueryServiceImpl   | Implementación de consultas de pagos        | Obtener historial y detalle               | Implementa PaymentQueryService            |

---

<div id='4.2.2.4.'><h5>4.2.2.4. Infrastructure Layer</h5></div>

Sub-capa Infrastructure<br>

| Tipo       | Nombre                    | Descripción                           | Responsabilidad Principal        | Relación con otros elementos |
| ---------- | ------------------------- | ------------------------------------- | -------------------------------- | ---------------------------- |
| Repository | PaymentRepository         | Repositorio de pagos                  | Persistencia de pagos            | Relacionado con Payment      |
| Repository | PaymentMethodRepository   | Repositorio de métodos de pago        | Persistencia de métodos          | Relacionado con PaymentMethod|

---

<div id='4.2.2.5.'><h5>4.2.2.5. Bounded Context Software Architecture Component Level Diagrams</h5></div>

Este diagrama representa la arquitectura del Bounded Context Payments, mostrando la interacción entre la capa de dominio, aplicación, interfaz e infraestructura para la gestión de pagos.

<div align="center">
<img src="assets/images/Chapter-4/bdc1.png">
</div>

---

<div id='4.2.2.6.'><h5>4.2.2.6. Bounded Context Software Architecture Code Level Diagram</h5></div>

<div id='4.2.2.6.1'><h5>4.2.2.6.1 Bounded Context Domain Layer Class Diagrams</h5></div>

Este diagrama UML representa la estructura del dominio Payments, mostrando los agregados principales como Payment y PaymentMethod, así como los Value Objects que controlan el estado y tipo de pago.

<div align="center">
<img src="assets/images/Chapter-4/uml1.png">
</div>

---

<div id='4.2.2.6.2'><h5>4.2.2.6.2 Bounded Context Database Design Diagram</h5></div>

<div align="center">
<img src="assets/images/Chapter-4/uml2.png">
</div>

---

### PAYMENTS

**Propósito:** Registro de pagos de alquileres

| Campo          | Tipo        | Descripción                              |
| -------------- | ----------- | ---------------------------------------- |
| `id`           | Long (PK)   | Identificador del pago                   |
| `rental_id`    | Long (FK)   | Alquiler asociado (Renting)              |
| `amount`       | decimal     | Monto total                             |
| `status`       | varchar     | Estado (`PENDING`, `SUCCESS`, `FAILED`) |
| `method`       | varchar     | Método (`CARD`, `YAPE`, `PLIN`)         |
| `payment_date` | datetime    | Fecha del pago                          |

**Relaciones:**
- `1:1` con **RENTALS**

---

### PAYMENT_METHODS

**Propósito:** Métodos de pago de usuarios

| Campo        | Tipo      | Descripción                          |
| ------------ | --------- | ------------------------------------ |
| `id`         | Long (PK) | Identificador                        |
| `user_id`    | Long (FK) | Usuario (IAM)                        |
| `type`       | varchar   | Tipo (`CARD`, `YAPE`, `PLIN`)        |
| `status`     | varchar   | Estado (`ACTIVE`, `DISABLED`)        |
| `created_at` | datetime  | Fecha de registro                    |

**Relaciones:**
- `N:1` con **USERS** (IAM)


---------------------------------------------------------------
<div id='4.2.3.'><h4>4.2.3. Bounded Context: Providing</h4></div>
<div id='4.2.3.1.'><h5>4.2.3.1. Domain Layer</h5></div>
Subcapamodel<br>

| Tipo         | Nombre                     | Descripción                                                 | Responsabilidad Principal                | Relación con otros elementos                   |
| ------------ | -------------------------- | ----------------------------------------------------------- | ---------------------------------------- | ---------------------------------------------- |
| Aggregate    | Bicycle                    | Representa una bicicleta registrada en la plataforma.       | Centralizar la información del vehículo. | Relacionado con Rental (Renting) y User (IAM). |
| Aggregate    | BicycleCatalog             | Representa el conjunto de bicicletas de un arrendador.      | Gestionar el listado de bicicletas.      | Relacionado con Bicycle.                       |
| Value Object | BicycleStatus              | Estado de la bicicleta (disponible, no disponible, en uso). | Controlar disponibilidad del vehículo.   | Asociado a Bicycle.                            |
| Value Object | BicycleType                | Tipo de bicicleta (eléctrica, montaña, urbana).             | Clasificar bicicletas.                   | Asociado a Bicycle.                            |
| Value Object | Location                   | Ubicación de la bicicleta.                                  | Representar coordenadas geográficas.     | Asociado a Bicycle y Tracking.                 |
| Value Object | Price                      | Tarifa de alquiler.                                         | Definir costo por uso.                   | Asociado a Bicycle.                            |
| Command      | RegisterBicycleCommand     | Registrar una nueva bicicleta.                              | Crear una bicicleta en el sistema.       | Usa Bicycle y User.                            |
| Command      | UpdateBicycleCommand       | Actualizar datos de bicicleta.                              | Modificar información del vehículo.      | Usa Bicycle.                                   |
| Command      | DeleteBicycleCommand       | Eliminar bicicleta.                                         | Dar de baja una bicicleta.               | Usa Bicycle.                                   |
| Command      | ChangeBicycleStatusCommand | Cambiar estado de disponibilidad.                           | Controlar visibilidad en búsquedas.      | Usa Bicycle.                                   |
| Query        | GetBicyclesByOwnerQuery    | Obtener bicicletas de un arrendador.                        | Listar bicicletas propias.               | Consulta Bicycle.                              |
| Query        | GetBicycleDetailQuery      | Obtener detalle de bicicleta.                               | Mostrar información completa.            | Consulta Bicycle.                              |
| Query        | GetAvailableBicyclesQuery  | Obtener bicicletas disponibles.                             | Mostrar bicicletas habilitadas.          | Consulta Bicycle.                              |



Sub-capa Services<br>

| Tipo      | Nombre                | Descripción                           | Responsabilidad Principal    | Relación con otros elementos                                     |
| --------- | --------------------- | ------------------------------------- | ---------------------------- | ---------------------------------------------------------------- |
| Interface | BicycleCommandService | Servicio para comandos de bicicletas  | Declarar métodos de gestión  | Implementado por BicycleCommandServiceImpl. Usado en Application |
| Interface | BicycleQueryService   | Servicio para consultas de bicicletas | Declarar métodos de consulta | Implementado por BicycleQueryServiceImpl. Usado en Application   |


<div id='4.2.3.1.'><h5>4.2.3.2. Interface Layer</h5></div>

Sub-capa REST <br>

| Tipo       | Nombre                                    | Descripción                          | Responsabilidad Principal               | Relación con otros elementos |
| ---------- | ----------------------------------------- | ------------------------------------ | --------------------------------------- | ---------------------------- |
| Controller | BicycleController                         | Controlador REST para bicicletas     | Gestiona operaciones CRUD de bicicletas | Usa services y resources     |
| Resource   | BicycleRequestResource                    | Estructura de petición de bicicleta  | Representa datos de entrada             | Usado por BicycleController  |
| Resource   | BicycleResponseResource                   | Estructura de respuesta de bicicleta | Devuelve datos al cliente               | Usado por BicycleController  |
| Assembler  | CreateBicycleCommandFromResourceAssembler | Convierte request en comando         | Traducir entrada a dominio              | Usado por BicycleController  |
| Assembler  | BicycleResourceFromEntityAssembler        | Convierte entidad a response         | Traducir dominio a salida               | Usado por BicycleController  |


<div id='4.2.3.1.'><h5>4.2.3.3. Aplications Layer</h5></div>

Sub-capa Internal<br>

| Tipo    | Nombre                    | Descripción                              | Responsabilidad Principal                          | Relación con otros elementos     |
| ------- | ------------------------- | ---------------------------------------- | -------------------------------------------------- | -------------------------------- |
| Service | BicycleCommandServiceImpl | Implementación de comandos de bicicletas | Ejecutar lógica de registro, edición y eliminación | Implementa BicycleCommandService |
| Service | BicycleQueryServiceImpl   | Implementación de consultas              | Obtener datos de bicicletas                        | Implementa BicycleQueryService   |


<div id='4.2.3.1.'><h5>4.2.3.4. Infrastructure Layer</h5></div>


Sub-capa Infrastructure <br>

| Tipo       | Nombre            | Descripción               | Responsabilidad Principal | Relación con otros elementos |
| ---------- | ----------------- | ------------------------- | ------------------------- | ---------------------------- |
| Repository | BicycleRepository | Repositorio de bicicletas | Persistencia de datos     | Relacionado con Bicycle      |


<div id='4.2.3.1.'><h5>4.2.3.5. Bounded Context Software Architecture Component Level Diagrams</h5></div>

<div align="center">
<img src="assets/images/Chapter-4/Bounded Context Software Architecture Component Level Diagrams_proving.jpeg">
</div>

<div id='4.2.3.1.'><h5>4.2.3.6. Bounded Context Software Architecture Code Level Diagram</h5></div>

<div id='4.2.3.1.'><h5>4.2.3.6.1 Bounded Context Domain Layer Class Diagrams</h5></div>

Este diagrama UML representa la arquitectura del Bounded Context Providing, enfocado en la gestión de bicicletas dentro de la plataforma. El modelo sigue principios de diseño orientado a objetos y adopta un enfoque basado en DDD (Domain-Driven Design) junto con CQRS, separando las operaciones de comandos y consultas.

En el núcleo del dominio se encuentran los agregados principales como Bicycle y BicycleCatalog, responsables de gestionar la información y organización de las bicicletas. Además, se utilizan Value Objects como BicycleStatus, BicycleType, Location y Price, que permiten modelar el dominio de forma más clara y consistente, evitando el uso de tipos genéricos sin contexto.


<div align="center">
<img src="assets/images/Chapter-4/Bounded Context Domain Layer Class Diagrams_proving.png">
</div>


<div id='4.2.3.1.'><h5>4.2.3.6.2 Bounded Context Database Design Diagram</h5></div>


<div align="center">
<img src="assets/images/Chapter-4/Bounded Context Database Design Diagram_proving.jpeg">
</div>

BICYCLES

Propósito: Registro de bicicletas del sistema

| Campo      | Tipo      | Descripción                   |
| ---------- | --------- | ----------------------------- |
| id         | Long (PK) | Identificador de la bicicleta |
| owner_id   | Long (FK) | Arrendador                    |
| type       | varchar   | Tipo de bicicleta             |
| status     | varchar   | Estado                        |
| price      | decimal   | Tarifa                        |
| latitude   | decimal   | Ubicación                     |
| longitude  | decimal   | Ubicación                     |
| created_at | datetime  | Fecha de registro             |

BICYCLE_HISTORY

Propósito: Historial de cambios de estado y uso

| Campo      | Tipo      | Descripción      |
| ---------- | --------- | ---------------- |
| id         | Long (PK) | Identificador    |
| bicycle_id | Long (FK) | Bicicleta        |
| status     | varchar   | Estado           |
| timestamp  | datetime  | Fecha del cambio |

## 4.2.4. Bounded Context: Tracking & Monitoring

El bounded context Tracking & Monitoring está encargado del seguimiento en tiempo real de las bicicletas, permitiendo registrar ubicación GPS, visualizar rutas recorridas, controlar el estado actual de la bicicleta y monitorear eventos generados por sensores IoT. Este contexto se relaciona con las user stories de ubicación en tiempo real, seguimiento de recorrido, estado de bicicleta e historial de rutas. Además, se conecta con el contexto de Providing, ya que la ubicación de la bicicleta está asociada al registro de bicicletas y al tracking del sistema.

### 4.2.4.1. Domain Layer

En esta capa se definen las entidades principales, agregados, value objects, comandos, queries, eventos de dominio y servicios del sistema encargados del seguimiento en tiempo real de bicicletas.

| Tipo           | Nombre                          | Descripción                                                                 | Responsabilidad Principal                                                           | Relación con otros elementos                            |
| -------------- | ------------------------------- | --------------------------------------------------------------------------- | ----------------------------------------------------------------------------------- | ------------------------------------------------------- |
| Aggregate      | TrackingSession                 | Representa una sesión activa de seguimiento de una bicicleta.               | Controlar el inicio, actualización y finalización del seguimiento en tiempo real.   | Relacionado con Bicycle, RouteHistory y LocationUpdate. |
| Aggregate      | RouteHistory                    | Representa el historial de recorridos realizados por una bicicleta.         | Guardar las rutas recorridas para consultas posteriores.                            | Relacionado con TrackingSession y Bicycle.              |
| Entity         | LocationUpdate                  | Registra una actualización puntual de ubicación GPS.                        | Almacenar latitud, longitud, precisión y fecha de actualización.                    | Pertenece a TrackingSession.                            |
| Entity         | BicycleMonitoringStatus         | Representa el estado operativo actual de una bicicleta.                     | Determinar si la bicicleta está disponible, en uso, detenida, offline o con alerta. | Se relaciona con Bicycle y eventos IoT.                 |
| Value Object   | Coordinates                     | Objeto que contiene latitud y longitud.                                     | Representar una ubicación geográfica inmutable.                                     | Usado por LocationUpdate y RouteHistory.                |
| Value Object   | GPSAccuracy                     | Representa el nivel de precisión de la señal GPS.                           | Validar si una ubicación puede ser usada correctamente.                             | Asociado a LocationUpdate.                              |
| Value Object   | TrackingStatus                  | Estado del seguimiento: ACTIVE, PAUSED, FINISHED.                           | Controlar el ciclo de vida del tracking.                                            | Asociado a TrackingSession.                             |
| Value Object   | MonitoringStatus                | Estado de bicicleta: AVAILABLE, IN_USE, STOPPED, OFFLINE, ALERT.            | Representar el estado actual de la bicicleta.                                       | Asociado a BicycleMonitoringStatus.                     |
| Command        | StartTrackingCommand            | Comando para iniciar el seguimiento de una bicicleta.                       | Activar una nueva sesión de tracking.                                               | Usa TrackingSession y Bicycle.                          |
| Command        | UpdateLocationCommand           | Comando para registrar una nueva ubicación GPS.                             | Guardar una actualización de coordenadas en tiempo real.                            | Usa LocationUpdate.                                     |
| Command        | StopTrackingCommand             | Comando para finalizar el seguimiento.                                      | Cerrar la sesión y guardar la ruta final.                                           | Usa TrackingSession y RouteHistory.                     |
| Command        | UpdateMonitoringStatusCommand   | Comando para actualizar el estado de una bicicleta.                         | Cambiar el estado actual de la bicicleta según eventos IoT.                         | Usa BicycleMonitoringStatus.                            |
| Query          | GetCurrentLocationQuery         | Consulta para obtener la ubicación actual de una bicicleta.                 | Mostrar la posición actual en el mapa.                                              | Consulta LocationUpdate.                                |
| Query          | GetRouteHistoryQuery            | Consulta para obtener rutas anteriores.                                     | Recuperar el historial de recorridos.                                               | Consulta RouteHistory.                                  |
| Query          | GetBicycleMonitoringStatusQuery | Consulta para obtener el estado actual de una bicicleta.                    | Mostrar disponibilidad, conexión o uso actual.                                      | Consulta BicycleMonitoringStatus.                       |
| Domain Event   | LocationUpdated                 | Evento generado cuando se actualiza la ubicación GPS.                       | Notificar que existe una nueva posición de bicicleta.                               | Consumido por mapas, monitoreo y notificaciones.        |
| Domain Event   | TrackingStarted                 | Evento generado al iniciar el seguimiento.                                  | Registrar el inicio del monitoreo.                                                  | Relacionado con Renting e IoT.                          |
| Domain Event   | TrackingFinished                | Evento generado al finalizar el seguimiento.                                | Cerrar la ruta recorrida.                                                           | Relacionado con RouteHistory.                           |
| Domain Event   | BicycleStatusChanged            | Evento generado cuando cambia el estado de una bicicleta.                   | Comunicar cambios de disponibilidad o uso.                                          | Relacionado con Providing y Notifications.              |
| Domain Service | RouteCalculationService         | Servicio que calcula distancia y duración de rutas.                         | Procesar coordenadas para obtener recorrido, distancia y tiempo.                    | Usa LocationUpdate y RouteHistory.                      |
| Domain Service | MonitoringStatusService         | Servicio que evalúa eventos IoT para determinar el estado de una bicicleta. | Detectar bicicleta en uso, detenida, offline o con alerta.                          | Usa BicycleMonitoringStatus y eventos IoT.              |

### 4.2.4.2. Interface Layer

| Tipo           | Nombre                                     | Descripción                                              | Responsabilidad Principal                                        | Relación con otros elementos                     |
| -------------- | ------------------------------------------ | -------------------------------------------------------- | ---------------------------------------------------------------- | ------------------------------------------------ |
| Controller     | TrackingController                         | Controlador REST para operaciones de seguimiento.        | Exponer endpoints para iniciar, actualizar y finalizar tracking. | Usa TrackingApplicationService.                  |
| Controller     | MonitoringController                       | Controlador REST para consultar estados de bicicletas.   | Permitir al cliente consultar estado y ubicación actual.         | Usa MonitoringApplicationService.                |
| Event Consumer | GPSLocationEventConsumer                   | Consumidor de eventos GPS enviados por dispositivos IoT. | Recibir coordenadas y enviar UpdateLocationCommand.              | Consume eventos del contexto IoT Device Control. |
| Event Consumer | MotionDetectedEventConsumer                | Consumidor de eventos de movimiento.                     | Detectar si una bicicleta está en uso o detenida.                | Relacionado con sensores IoT.                    |
| DTO            | TrackingSessionResponse                    | Respuesta con datos de una sesión de tracking.           | Enviar información de la sesión al cliente.                      | Usado por TrackingController.                    |
| DTO            | LocationUpdateRequest                      | Payload con latitud, longitud, precisión y timestamp.    | Transportar datos GPS hacia la aplicación.                       | Usado por TrackingController.                    |
| DTO            | CurrentLocationResponse                    | Respuesta con la ubicación actual de la bicicleta.       | Mostrar la posición actual en el mapa.                           | Usado por MonitoringController.                  |
| DTO            | RouteHistoryResponse                       | Respuesta con el recorrido histórico.                    | Mostrar rutas anteriores al usuario.                             | Usado por TrackingController.                    |
| DTO            | BicycleMonitoringStatusResponse            | Respuesta con el estado actual de la bicicleta.          | Mostrar disponibilidad, uso o alerta.                            | Usado por MonitoringController.                  |
| Assembler      | TrackingSessionResourceFromEntityAssembler | Convierte entidad TrackingSession a respuesta.           | Traducir dominio a DTO.                                          | Usado por TrackingController.                    |
| Assembler      | LocationUpdateCommandFromResourceAssembler | Convierte request en comando.                            | Traducir entrada HTTP a comando de aplicación.                   | Usado por TrackingController.                    |

### 4.2.4.3. Application Layer

| Tipo                | Nombre                                 | Descripción                                       | Responsabilidad Principal                            | Relación con otros elementos                            |
| ------------------- | -------------------------------------- | ------------------------------------------------- | ---------------------------------------------------- | ------------------------------------------------------- |
| Application Service | TrackingApplicationService             | Servicio principal para casos de uso de tracking. | Orquestar comandos y consultas de seguimiento.       | Invocado por TrackingController.                        |
| Application Service | MonitoringApplicationService           | Servicio principal para monitoreo de bicicletas.  | Coordinar consultas de estado y ubicación.           | Invocado por MonitoringController.                      |
| Command Handler     | StartTrackingCommandHandler            | Handler para iniciar una sesión de seguimiento.   | Crear TrackingSession y publicar TrackingStarted.    | Usa TrackingSessionRepository.                          |
| Command Handler     | UpdateLocationCommandHandler           | Handler para registrar coordenadas GPS.           | Guardar LocationUpdate y publicar LocationUpdated.   | Usa LocationUpdateRepository.                           |
| Command Handler     | StopTrackingCommandHandler             | Handler para finalizar tracking.                  | Cerrar sesión y generar RouteHistory.                | Usa TrackingSessionRepository y RouteHistoryRepository. |
| Command Handler     | UpdateMonitoringStatusCommandHandler   | Handler para actualizar estado de bicicleta.      | Modificar BicycleMonitoringStatus según eventos IoT. | Usa MonitoringStatusService.                            |
| Query Handler       | GetCurrentLocationQueryHandler         | Handler para obtener ubicación actual.            | Recuperar la última ubicación GPS registrada.        | Usa LocationUpdateRepository.                           |
| Query Handler       | GetRouteHistoryQueryHandler            | Handler para consultar historial de rutas.        | Obtener rutas previas por usuario o bicicleta.       | Usa RouteHistoryRepository.                             |
| Query Handler       | GetBicycleMonitoringStatusQueryHandler | Handler para consultar estado actual.             | Recuperar disponibilidad, conexión o alerta.         | Usa BicycleMonitoringStatusRepository.                  |

### 4.2.4.4. Infrastructure Layer

| Tipo              | Nombre                            | Descripción                          | Responsabilidad Principal                                       | Relación con otros elementos                      |
| ----------------- | --------------------------------- | ------------------------------------ | --------------------------------------------------------------- | ------------------------------------------------- |
| Repository        | TrackingSessionRepository         | Repositorio de sesiones de tracking. | Persistir sesiones activas y finalizadas.                       | Relacionado con TrackingSession.                  |
| Repository        | LocationUpdateRepository          | Repositorio de ubicaciones GPS.      | Guardar coordenadas recibidas en tiempo real.                   | Relacionado con LocationUpdate.                   |
| Repository        | RouteHistoryRepository            | Repositorio de historial de rutas.   | Almacenar rutas finalizadas.                                    | Relacionado con RouteHistory.                     |
| Repository        | BicycleMonitoringStatusRepository | Repositorio de estados de bicicleta. | Persistir estado actual de monitoreo.                           | Relacionado con BicycleMonitoringStatus.          |
| External Adapter  | GoogleMapsAdapter                 | Adaptador para servicios de mapas.   | Mostrar ubicaciones y rutas en mapa.                            | Usado por consultas de ubicación.                 |
| Message Consumer  | IoTTrackingMessageConsumer        | Consumidor de mensajes IoT.          | Recibir GPS, movimiento y estado desde MQTT/IoT Hub.            | Alimenta los event consumers.                     |
| Message Publisher | TrackingEventPublisher            | Publicador de eventos del contexto.  | Emitir LocationUpdated, TrackingStarted y BicycleStatusChanged. | Consumido por Notifications, Providing y Renting. |

### 4.2.4.5. Bounded Context Software Architecture Component Level Diagrams

Este diagrama representa la arquitectura interna del bounded context **Tracking & Monitoring**, mostrando cómo se organizan sus principales componentes por capas. En la capa de interfaz se encuentran los controladores encargados de recibir solicitudes desde la aplicación móvil y el panel administrativo, así como los consumidores de eventos provenientes de dispositivos IoT.

La capa de aplicación coordina los casos de uso principales, como iniciar el seguimiento de una bicicleta, actualizar su ubicación, consultar su estado actual y recuperar el historial de rutas. La capa de dominio contiene los elementos centrales del negocio, como **TrackingSession**, **LocationUpdate**, **RouteHistory** y **BicycleMonitoringStatus**, los cuales permiten controlar el seguimiento en tiempo real y el estado operativo de cada bicicleta.

Finalmente, la capa de infraestructura se encarga de la persistencia de datos, la integración con Google Maps y la recepción de eventos IoT. Este diseño permite separar responsabilidades, facilitar el mantenimiento del sistema y asegurar que el monitoreo de bicicletas pueda evolucionar sin afectar otros bounded contexts.

<p align="center">
  <img src="assets/images/Chapter-4/cld_1.png" width="700"/>
</p>

### 4.2.4.6. Bounded Context Software Architecture Code Level Diagrams

Este diagrama muestra con mayor detalle la estructura de código del bounded context **Tracking & Monitoring**, representando las principales clases, entidades, value objects, comandos, consultas, eventos de dominio y servicios de dominio utilizados en el seguimiento de bicicletas.

El agregado principal es **TrackingSession**, encargado de gestionar el ciclo de vida del seguimiento, desde su inicio hasta su finalización. La entidad **LocationUpdate** registra cada actualización GPS recibida desde los dispositivos IoT, mientras que **RouteHistory** conserva el resumen del recorrido realizado. Además, **BicycleMonitoringStatus** permite conocer el estado actual de la bicicleta, como disponible, en uso, detenida, sin conexión o en alerta.

El diseño sigue el enfoque **CQRS** (separación entre comandos y consultas), donde los comandos modifican el estado del sistema y las consultas recuperan información sin alterarla. También se utilizan eventos de dominio como **LocationUpdated**, **TrackingStarted** y **TrackingFinished**, que permiten comunicar cambios importantes a otros contextos como Notifications, Renting y Providing.

<p align="center">
  <img src="assets/images/Chapter-4/code_d.png" width="700"/>
</p>

#### 4.2.4.6.1. Bounded Context Domain Layer Class Diagrams

Este diagrama UML representa la arquitectura del bounded context Tracking & Monitoring. Se organiza bajo principios de diseño orientado a objetos y sigue el enfoque CQRS, separando comandos para modificar el estado del sistema y consultas para obtener información de monitoreo.

El agregado principal es TrackingSession, encargado de controlar el ciclo de vida del seguimiento de una bicicleta. LocationUpdate registra las coordenadas GPS recibidas desde los dispositivos IoT, mientras que RouteHistory almacena el recorrido finalizado. Además, BicycleMonitoringStatus permite conocer el estado actual de la bicicleta, como disponible, en uso, detenida, offline o en alerta.

Este contexto interactúa con:
- Providing, para actualizar la ubicación y estado de las bicicletas.
- Renting, para iniciar o finalizar el seguimiento durante un alquiler.
- IoT & Device Control, para recibir datos GPS, movimiento y conexión.
- Notifications, para generar alertas ante eventos importantes.

<p align="center">
  <img src="assets/images/Chapter-4/class_diagram.png" width="700"/>
</p>

#### 4.2.4.6.2. Bounded Context Database Design Diagram

La base de datos del bounded context Tracking & Monitoring permite almacenar sesiones de seguimiento, actualizaciones GPS, historial de rutas, estado actual de monitoreo y eventos generados durante el recorrido.

La tabla tracking_sessions registra cada sesión activa o finalizada. La tabla location_updates almacena las coordenadas recibidas periódicamente. La tabla route_histories conserva el resumen de los recorridos realizados. La tabla bicycle_monitoring_status mantiene el estado actual de cada bicicleta, mientras que tracking_events registra eventos relevantes como pérdida de conexión, movimiento sospechoso o cambios de estado.

<p align="center">
  <img src="assets/images/Chapter-4/data_base.png" width="700"/>
</p>

## 4.2.5. Bounded Context: IoT & Device Control

Este *bounded context* es **core** y se encarga de la **interacción con el
hardware y los sensores**: registra los dispositivos IoT acoplados a cada
bicicleta, los empareja con su bicicleta correspondiente, ejecuta comandos
físicos (unlock / lock / lockdown) vía MQTT y publica la telemetría que
alimenta a *Tracking & Monitoring*.

---

### 4.2.5.1. Domain Layer

#### Sub-capa model

| Tipo | Nombre | Descripción | Responsabilidad Principal | Relación con otros elementos |
|---|---|---|---|---|
| Aggregate | **Device** | Representa un dispositivo IoT (cerradura + GPS) acoplado a una bicicleta. | Gestionar el ciclo de vida del dispositivo y su estado físico. | Relacionado con Pairing, Bicycle (Providing) y TelemetryReading. |
| Aggregate | **Pairing** | Representa el vínculo seguro entre un Device y una Bicycle. | Mantener la relación device ↔ bicicleta y validar tokens. | Relacionado con Device y Bicycle (Providing). |
| Aggregate | **TelemetryReading** | Representa una lectura de telemetría (batería, conexión, GPS). | Almacenar la telemetría histórica del dispositivo. | Asociado a Device. Consumido por Tracking. |
| Value Object | **LockState** | Estado físico de la cerradura (`LOCKED` / `UNLOCKED` / `LOCKDOWN`). | Controlar el estado actual de la cerradura. | Asociado a Device. |
| Value Object | **DeviceStatus** | Estado operativo (`ONLINE` / `OFFLINE` / `LOW_BATTERY`). | Gestionar disponibilidad para alquiler. | Asociado a Device. |
| Value Object | **FirmwareVersion** | Versión semántica del firmware OTA (major.minor.patch). | Controlar las actualizaciones OTA. | Asociado a Device. |
| Value Object | **BatteryLevel** | Nivel de batería en porcentaje (0-100). | Decidir cuándo pausar el dispositivo. | Asociado a Device y TelemetryReading. |
| Value Object | **PairingToken** | Token de un solo uso con expiración. | Asegurar que el pairing sea único e irrepetible. | Asociado a Pairing. |
| Command | **RegisterDeviceCommand** | Registrar un nuevo dispositivo IoT en la plataforma. | Crear la entidad Device. | Usa Device. |
| Command | **PairDeviceCommand** | Emparejar un Device con una Bicycle. | Crear un Pairing válido. | Usa Pairing y Device. |
| Command | **UnlockBicycleCommand** | Enviar comando MQTT de desbloqueo a la cerradura. | Activar la cerradura. | Usa Device. |
| Command | **ReleaseBicycleCommand** | Devolver la bicicleta al cierre del alquiler. | Liberar la cerradura. | Usa Device. |
| Command | **ActivateLockdownCommand** | Activar modo bloqueo anti-robo de forma remota. | Bloquear remotamente. | Usa Device. |
| Command | **UpdateFirmwareCommand** | Lanzar una actualización OTA al dispositivo. | Actualizar versión de firmware. | Usa Device. |
| Query | **GetDeviceStatusQuery** | Obtener el estado actual del dispositivo. | Mostrar estado al sistema. | Consulta Device. |
| Query | **GetPairingByBicycleQuery** | Obtener el pairing por bicicleta. | Encontrar el Device asociado. | Consulta Pairing. |
| Query | **GetTelemetryHistoryQuery** | Obtener el historial de telemetría de un dispositivo. | Análisis y diagnósticos. | Consulta TelemetryReading. |

#### Sub-capa Services

| Tipo | Nombre | Descripción | Responsabilidad Principal | Relación con otros elementos |
|---|---|---|---|---|
| Interface | **DeviceCommandService** | Servicio para comandos relacionados con dispositivos. | Declarar métodos para registrar, desbloquear, bloquear y actualizar firmware. | Implementado por `DeviceCommandServiceImpl`. Usado en capa Application. |
| Interface | **PairingCommandService** | Servicio para comandos relacionados con emparejamientos. | Declarar métodos para emparejar y desemparejar dispositivos. | Implementado por `PairingCommandServiceImpl`. Usado en capa Application. |
| Interface | **DeviceQueryService** | Servicio para consultas de dispositivos. | Declarar métodos para obtener estado y configuración. | Implementado por `DeviceQueryServiceImpl`. Usado en capa Application. |
| Interface | **TelemetryQueryService** | Servicio para consultas de telemetría. | Declarar métodos para obtener historial y agregaciones. | Implementado por `TelemetryQueryServiceImpl`. Usado en capa Application. |

---

### 4.2.5.2. Interface Layer

#### Sub-capa REST

| Tipo | Nombre | Descripción | Responsabilidad Principal | Relación con otros elementos |
|---|---|---|---|---|
| Controller | **DeviceController** | Controlador REST para gestionar dispositivos. | Recibe solicitudes del cliente sobre registro, unlock, lock y firmware. | Utiliza `DeviceCommandService`, `DeviceQueryService` y los assemblers de Device. |
| Controller | **PairingController** | Controlador REST para gestionar emparejamientos. | Maneja solicitudes de creación y cancelación de pairings. | Utiliza `PairingCommandService` y los assemblers de Pairing. |
| Resource | **DeviceRequestResource** | Estructura de petición para registro o comando físico. | Representa los datos de entrada del cliente. | Usado por `DeviceController`. |
| Resource | **DeviceResponseResource** | Estructura de respuesta del dispositivo. | Devuelve el estado del Device al cliente. | Usado por `DeviceController`. |
| Resource | **PairingRequestResource** | Estructura de petición para crear un pairing. | Representa los datos de entrada del cliente. | Usado por `PairingController`. |
| Resource | **PairingResponseResource** | Estructura de respuesta de pairing. | Devuelve los datos al cliente. | Usado por `PairingController`. |
| Assembler | **RegisterDeviceCommandFromResourceAssembler** | Convierte el request en comando de dominio. | Traducir la entrada a `RegisterDeviceCommand`. | Usado por `DeviceController`. |
| Assembler | **DeviceResourceFromEntityAssembler** | Convierte la entidad Device en respuesta. | Traducir el dominio a la respuesta REST. | Usado por `DeviceController`. |

---

### 4.2.5.3. Application Layer

#### Sub-capa Internal

| Tipo | Nombre | Descripción | Responsabilidad Principal | Relación con otros elementos |
|---|---|---|---|---|
| Service | **DeviceCommandServiceImpl** | Implementación del servicio de comandos para dispositivos. | Ejecutar la lógica de registro, unlock, lock y actualización OTA. | Implementa `DeviceCommandService`. Usa repositorios y `MqttBrokerAdapter`. |
| Service | **PairingCommandServiceImpl** | Implementación del servicio de pairing. | Ejecutar la lógica de emparejamiento y validación de tokens. | Implementa `PairingCommandService`. Usa `PairingRepository` y `BleProvisioningAdapter`. |
| Service | **DeviceQueryServiceImpl** | Implementación de consultas de dispositivos. | Obtener estado y configuración. | Implementa `DeviceQueryService`. |
| Service | **TelemetryQueryServiceImpl** | Implementación de consultas de telemetría. | Obtener historial y agregaciones. | Implementa `TelemetryQueryService`. |

---

### 4.2.5.4. Infrastructure Layer

#### Sub-capa Infrastructure

| Tipo | Nombre | Descripción | Responsabilidad Principal | Relación con otros elementos |
|---|---|---|---|---|
| Repository | **DeviceRepository** | Repositorio para gestionar dispositivos. | Persistencia de datos del Device. | Relacionado con Device. |
| Repository | **PairingRepository** | Repositorio para gestionar emparejamientos. | Persistencia de pairings. | Relacionado con Pairing. |
| Repository | **TelemetryRepository** | Repositorio para gestionar lecturas de telemetría. | Persistencia de TelemetryReading. | Relacionado con TelemetryReading. |
| Adapter | **MqttBrokerAdapter** | Cliente MQTT para enviar comandos al hardware. | Publicar comandos en `/bicismart/{deviceId}/cmd` con QoS 2 y mTLS. | Conecta con AWS IoT Core. |
| Adapter | **BleProvisioningAdapter** | Cliente BLE para emparejamiento inicial. | Provisionar credenciales del Device vía Bluetooth Low Energy. | Conecta con la app móvil del arrendador. |
| Adapter | **OtaUpdateAdapter** | Adaptador de actualizaciones OTA. | Empujar firmware al dispositivo de forma segura. | Conecta con bucket S3 de firmware. |

---

### 4.2.5.5. Bounded Context Software Architecture Component Level Diagrams

![Component Level Diagram — IoT & Device Control](assets/images/Chapter-4/IoT_Component_Level.png)

> *Figura 4.2.5.1 — Diagrama de componentes (C4) del Bounded Context IoT & Device Control. Se observan los controladores REST, los servicios de comandos y queries, los aggregates del dominio, los repositorios JPA y el adaptador MQTT que conecta con AWS IoT Core y los dispositivos físicos.*

---

### 4.2.5.6. Bounded Context Software Architecture Code Level Diagrams

#### 4.2.5.6.1 Bounded Context Domain Layer Class Diagrams

Este diagrama UML representa la arquitectura del flujo de control de
dispositivos IoT. Se basa en principios de diseño orientado a objetos y
sigue el enfoque **CQRS** (Command Query Responsibility Segregation).

Las entidades principales son **Device**, **Pairing** y **TelemetryReading**,
que gestionan el ciclo de vida del hardware acoplado a cada bicicleta.

Se interactúa con otros *bounded contexts* como:

- **Providing** (relación con Bicycle al emparejar)
- **Renting** (recibe los comandos de unlock/release durante el alquiler)
- **Tracking & Monitoring** (consume los pings GPS publicados)

![UML Class Diagram — IoT & Device Control](assets/images/Chapter-4/IoT_Class_Diagram.png)

> *Figura 4.2.5.2 — Diagrama UML del Domain Layer del BC IoT & Device Control con sus aggregates, commands, queries, services y value objects.*

---

#### 4.2.5.6.2 Bounded Context Database Design Diagram

![Database Design Diagram — IoT & Device Control](assets/images/Chapter-4/IoT_Database_Design.png)

> *Figura 4.2.5.3 — Diseño de base de datos del BC IoT & Device Control. La entidad central `devices` se relaciona con `pairings` (emparejamientos device ↔ bicicleta), `telemetry_readings` (telemetría histórica) y `firmware_updates` (gestión de OTA).*

##### DEVICES

**Propósito:** Registro maestro de dispositivos IoT acoplados a las bicicletas.

| Campo | Tipo | Descripción |
|---|---|---|
| `id` | Long (PK) | Identificador único del dispositivo. |
| `mac_address` | varchar | Dirección MAC física del módulo IoT. |
| `firmware_version` | varchar | Versión semántica del firmware (`x.y.z`). |
| `battery_level` | int | Nivel de batería 0-100. |
| `lock_state` | varchar | Estado (`LOCKED`, `UNLOCKED`, `LOCKDOWN`). |
| `device_status` | varchar | Estado (`ONLINE`, `OFFLINE`, `LOW_BATTERY`). |
| `registered_at` | datetime | Fecha de registro inicial. |

**Relaciones:**

- `1:N` con **PAIRINGS**
- `1:N` con **TELEMETRY_READINGS**
- `1:N` con **FIRMWARE_UPDATES**

##### PAIRINGS

**Propósito:** Vínculo seguro entre un Device y una Bicycle.

| Campo | Tipo | Descripción |
|---|---|---|
| `id` | Long (PK) | Identificador único. |
| `device_id` | Long (FK) | Dispositivo emparejado. |
| `bicycle_id` | Long (FK) | Bicicleta emparejada (Providing). |
| `pairing_token` | varchar | Token de un solo uso. |
| `paired_at` | datetime | Fecha de emparejamiento. |
| `status` | varchar | Estado (`ACTIVE`, `REVOKED`). |

**Relaciones:**

- `N:1` con **DEVICES**
- `N:1` con **BICYCLES** (Providing)

##### TELEMETRY_READINGS

**Propósito:** Lecturas históricas de telemetría del dispositivo.

| Campo | Tipo | Descripción |
|---|---|---|
| `id` | Long (PK) | Identificador único. |
| `device_id` | Long (FK) | Dispositivo emisor. |
| `battery_level` | int | Nivel de batería en el momento. |
| `is_online` | bool | Estado de conexión. |
| `lat` | double | Latitud GPS. |
| `lon` | double | Longitud GPS. |
| `recorded_at` | datetime | Timestamp de la lectura. |

**Relaciones:**

- `N:1` con **DEVICES**

##### FIRMWARE_UPDATES

**Propósito:** Bitácora de actualizaciones OTA aplicadas a cada dispositivo.

| Campo | Tipo | Descripción |
|---|---|---|
| `id` | Long (PK) | Identificador. |
| `device_id` | Long (FK) | Dispositivo actualizado. |
| `from_version` | varchar | Versión anterior. |
| `to_version` | varchar | Versión nueva. |
| `requested_at` | datetime | Inicio de la actualización. |
| `completed_at` | datetime | Fin de la actualización (nullable). |
| `status` | varchar | Estado (`PENDING`, `IN_PROGRESS`, `COMPLETED`, `FAILED`). |

**Relaciones:**

- `N:1` con **DEVICES**


  


  

