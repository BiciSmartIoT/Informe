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

1. **IAM** — registro y validación de cuentas (Estudiante UPC / Arrendador).
2. **Providing** — publicación de bicicletas, tarifas y zonas operativas.
3. **IoT & Device Control** — comandos al hardware (cerradura, GPS, OTA).
4. **Renting** — orquestador del alquiler end-to-end.
5. **Tracking & Monitoring** — telemetría GPS, geofencing y anti-robo.
6. **Payments** — autorización y procesamiento de pagos.
7. **Notifications** — entrega multicanal (push, email, SMS).

![Paso 7 — Agregados y Bounded Contexts](assets/images/Chapter-4/Agregados%20_Bounded_Contexts.png)

---

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

Este diagrama UML representa la arquitectura del flujo de alquiler de bicicletas inteligentes. Se basa en principios de diseño orientado a objetos y sigue el enfoque CQRS (Command Query Responsibility Segregation).

Las entidades principales son Rental y Reservation, que gestionan el ciclo completo del alquiler.

Se interactúa con otros bounded contexts como:

* Providing (bicicletas)
* Payments (pagos)
* IoT & Device Control (inicio y fin físico del alquiler)

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




### 4.2.2. Bounded Context: **Payments**

#### 4.2.2.1. Domain Layer

**Agregados y Entidades**

* **PaymentMethod** *(Aggregate Root)*
  Atributos: `PaymentMethodId`, `UserId`, `Type{card|yape|plin}`, `Status{Pending|Verified|Failed|Disabled}`, `PspTokenRef`, `Brand?`, `Last4?`, `IsDefault`, `CreatedAt`.
  Invariantes:

  * Un usuario puede marcar **un** método por defecto.
  * `Status=Verified` exige token válido del PSP.
    Operaciones: `verify(pspToken)`, `setDefault()`, `disable()`.

* **Authorization** *(Aggregate Root)*
  Atributos: `AuthorizationId`, `UserId`, `ReservationId?`, `RentalId?`, `AmountEstimate(Money)`, `Currency`, `Status{Created|Authorized|Failed|Voided}`, `HoldExpiresAt?`, `PspAuthRef`.
  Invariantes:

  * Solo se puede **capturar** si `Status=Authorized`.
  * Una reserva/alquiler tiene a lo sumo **una** autorización activa.
    Operaciones: `markAuthorized(pspRef, hold)`, `fail(reason)`, `void()`.

* **Charge** *(Aggregate Root)*
  Atributos: `ChargeId`, `UserId`, `RentalId`, `AuthorizationId?`, `AmountFinal(Money)`, `Currency`, `Status{Captured|Failed|Refunded}`, `Breakdown{unlock, perMinute, penalties?}`, `PspChargeRef`, `CreatedAt`.
  Invariantes:

  * `Captured` requiere confirmación PSP o política de “pending\_capture” con conciliación.
    Operaciones: `capture(amount)`, `refund(partial?)`.

* **Penalty** *(Entidad ligada a Charge/Authorization)*
  Atributos: `PenaltyId`, `RentalId`, `Type{overdue|out_of_zone|damage}`, `Amount(Money)`, `Status{Pending|Charged|Failed}`, `Reason?`.

* **Payout** *(Aggregate Root)*
  Atributos: `PayoutId`, `ProviderId`, `Period{start,end}`, `Amount(Money)`, `Status{Scheduled|Processing|Paid|Failed}`, `PspPayoutRef?`, `CreatedAt`, `PaidAt?`.
  Invariantes:

  * Un período y proveedor generan **un único** payout (idempotencia por `ProviderId+Period`).
    Operaciones: `schedule()`, `markPaid(ref)`, `fail(reason)`.

**Value Objects**

* `Money{amount, currency}` (inmut.)
* `FeeBreakdown{unlock, perMinute, perKm?, penalties}`
* `WalletId/ExternalRef` (cuando aplique)
* `PspError(code,message)` (mapea errores externos a internos)

**Servicios de Dominio**

* **FeeCalculatorService**: calcula totales según tarifas vigentes.
* **AntiFraudPolicy** (básica S1): verificación mínima de riesgo (monto, historial de fallas).
* **PayoutPolicy**: define frecuencia (S1 semanal), mínimos y retenciones.

**Repositorios (interfaces)**

* `PaymentMethodRepository`, `AuthorizationRepository`, `ChargeRepository`, `PenaltyRepository`, `PayoutRepository`.

**Eventos publicados**

* `PaymentMethodVerified {userId, methodId, type}`
* `PaymentAuthorized {authorizationId, userId, rentalId?, reservationId?, amount, currency, holdExpiresAt}`
* `PaymentCaptured {chargeId, userId, rentalId, amount, currency}`
* `PaymentFailed {context, id, reason}`
* `PenaltyCharged {penaltyId, rentalId, amount, type}`
* `RefundProcessed {chargeId, amount}`
* `PayoutSettled {payoutId, providerId, amount, period}`

**Suscripciones (entrantes)**

* De **Renting**:

  * `ReservationCreated` *(opcional si se preautoriza en reserva)*
  * `RentalStarted` → **Authorize**
  * `RentalFinished` → **Capture**
  * `PenaltyApplied` → **ChargePenalty**
* De **Providing**:

  * `ProviderVerified` (checklist de payout)
* De **IAM**:

  * `UserSuspended` (bloquear cargos nuevos)

**Políticas clave**

* Autorización **previa** al inicio; captura **al finalizar**.
* Reintentos con backoff en fallas PSP; idempotencia por `Idempotency-Key`.
* No se expone **datos sensibles** (solo `PspTokenRef`).

---

#### 4.2.2.2. Interface Layer

**Base path:** `/api/v1/payments` · **Auth:** Bearer · **Formato:** JSON

**Métodos de pago (User)**

* `POST /methods` → alta/verify de método
  Body:

  ```
  { "type":"card|yape|plin", "pspToken":"tok_…" , "setDefault":true|false }
  ```

  201:

  ```
  { "methodId":"pm_…", "status":"Verified", "brand":"VISA", "last4":"1234", "isDefault":true }
  ```
* `GET /methods` → listar propios
* `POST /methods/{id}:default` → marcar por defecto
* `POST /methods/{id}:disable` → deshabilitar

**Autorización/Captura (desde Renting o app del usuario)**

* `POST /authorizations`
  Body:

  ```
  { "reservationId":"res_…", "rentalId":null, "amount":"12.50", "currency":"PEN", "methodId":"pm_…" }
  ```

  201:

  ```
  { "authorizationId":"auth_…", "status":"Authorized", "holdExpiresAt":"…" }
  ```
* `POST /charges` *(captura)*
  Body:

  ```
  { "rentalId":"rent_…", "authorizationId":"auth_…", "amount":"18.20", "currency":"PEN", "breakdown":{ "unlock":"1.50","perMinute":"16.70" } }
  ```

  201:

  ```
  { "chargeId":"ch_…", "status":"Captured", "receiptId":"inv_…" }
  ```

**Penalidades y reembolsos**

* `POST /penalties`
  Body:

  ```
  { "rentalId":"rent_…", "type":"overdue|out_of_zone|damage", "amount":"5.00", "currency":"PEN" }
  ```

  201:

  ```
  { "penaltyId":"pen_…", "status":"Charged" }
  ```
* `POST /charges/{id}:refund`
  Body: `{ "amount":"3.00" }` → 200 `{ "status":"Refunded" }`

**Payouts (Proveedor/Admin)**

* `GET /payouts?mine=true` → listar del proveedor
* `POST /payouts:simulate` *(preview)*
  Body: `{ "periodStart":"YYYY-MM-DD", "periodEnd":"YYYY-MM-DD" }`
* `POST /payouts:run` *(admin/job manual)* → crea `Payout(Scheduled)`
* `GET /payouts/{id}` → estado del payout

**Historial**

* `GET /users/me/charges?from=&to=&status=`
* `GET /providers/me/payouts?from=&to=&status=`

**Webhooks**

* `POST /webhooks/psp` *(firma HMAC/JWK)* → recibe `authorized|captured|failed|payout.paid|charge.refunded`.

**Errores comunes**

* `402 PAYMENT_REQUIRED` (AUTH\_DECLINED, CAPTURE\_FAILED)
* `409 INVALID_STATE` (capturar sin auth)
* `422 METHOD_NOT_VERIFIED`, `422 INVALID_AMOUNT`
* `503 PSP_UNAVAILABLE`

**Trazabilidad con US**
US20/US21/US22 (métodos, pagar), US23 (penalidades), US24 (historial), US25 (payouts).

---

#### 4.2.2.3. Application Layer

**Use Cases / Command Handlers**

* `AddPaymentMethod(cmd)` → `PaymentMethod.verify(pspToken)` via `PspClient.tokenVerify()` → guardar → `PaymentMethodVerified`.
* `AuthorizePayment(cmd)` → valida método por defecto o `methodId` → `AntiFraudPolicy.check()` → `PspClient.authorize()` → `Authorization.markAuthorized(pspRef, hold)` → `PaymentAuthorized`.
* `CapturePayment(cmd)` → busca `Authorization(Authorized)` → `PspClient.capture()` → crear `Charge(Captured)` con `Breakdown` → `PaymentCaptured`.
* `ChargePenalty(cmd)` → `PspClient.charge(amount)` → `Penalty.Charged` → `PenaltyCharged`.
* `RefundCharge(cmd)` → `PspClient.refund()` → `RefundProcessed`.
* `SchedulePayoutsJob()` → agrega `Payout(Scheduled)` por proveedor/periodo → `ProcessPayout(cmd)` → `PspClient.payout()` → `PayoutSettled`.

**Event Handlers**

* `OnRentalStarted` ← Renting → `AuthorizePayment(reservationId/rentalId, estimate)` (si el flujo es asíncrono).
* `OnRentalFinished` ← Renting → `CapturePayment(rentalId, total)` (asíncrono).
* `OnPenaltyApplied` ← Renting → `ChargePenalty(rentalId,type,amount)`.

**Puertos (Ports)**

* `PspClient` (ACL a la pasarela: Stripe/Yape/Plin/Agregador)

  * `tokenVerify(pspToken)`, `authorize(amount,currency,methodRef, idempotencyKey)`, `capture(pspAuthRef, amount, key)`, `charge(amount, methodRef, key)`, `refund(pspChargeRef, amount?, key)`, `payout(providerExternalRef, amount, key)`
* `EventPublisherPort` (outbox → `payments.events.*`)
* `ClockPort`, `IdempotencyStorePort` (Redis), `ConfigPort` (fees/currency)

**Consistencia e Idempotencia**

* **Transactional Outbox** para todos los eventos.
* Idempotency-Key = `contextId` (`reservationId`/`rentalId`/`payoutPeriod+providerId`).
* Retries con backoff; DLQ para errores PSP.

**Métricas S1**

* Tasa de **éxito** `authorize/capture`.
* GMV por día/periodo; contracargos (si aplica).
* Tiempo promedio de **payout**.

**Reglas de seguridad**

* Nunca loguear `pspToken` ni PAN; enmascarar `last4/brand`.
* Validar **webhook signature**; tolerar *replay* con nonce/ts.

---

#### 4.2.2.4. Infrastructure Layer

**Adaptadores**

* **Repos (MySQL/JPA)**: `SqlPaymentMethodRepository`, `SqlAuthorizationRepository`, `SqlChargeRepository`, `SqlPenaltyRepository`, `SqlPayoutRepository`.
* **PSP Client (HTTP)**: `StripeAdapter` / `YapePlinAdapter` (timeout, retries, circuit breaker).
* **Mensajería**: `OutboxPublisher` → `payments.events.*`; `WebhookHandler` firmado.
* **Idempotencia/Caché**: Redis (`idemp:{key}` con TTL), locks para evitar *double-capture*.
* **Clock/Config**: adaptadores simples.

**Esquema SQL mínimo**

```
CREATE TABLE pay_methods(
  id BIGINT PRIMARY KEY,
  user_id BIGINT NOT NULL,
  type VARCHAR(10) NOT NULL,             -- card|yape|plin
  status VARCHAR(12) NOT NULL,           -- Pending|Verified|Failed|Disabled
  psp_token_ref VARCHAR(120) NOT NULL,
  brand VARCHAR(20), last4 CHAR(4),
  is_default BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP NOT NULL,
  UNIQUE(user_id, is_default) WHERE is_default = TRUE
);

CREATE TABLE pay_authorizations(
  id BIGINT PRIMARY KEY,
  user_id BIGINT NOT NULL,
  reservation_id BIGINT NULL,
  rental_id BIGINT NULL,
  amount DECIMAL(10,2) NOT NULL,
  currency CHAR(3) NOT NULL,
  status VARCHAR(12) NOT NULL,           -- Created|Authorized|Failed|Voided
  psp_auth_ref VARCHAR(120),
  hold_expires_at TIMESTAMP NULL,
  created_at TIMESTAMP NOT NULL,
  UNIQUE(reservation_id),
  UNIQUE(rental_id)
);

CREATE TABLE pay_charges(
  id BIGINT PRIMARY KEY,
  user_id BIGINT NOT NULL,
  rental_id BIGINT NOT NULL,
  authorization_id BIGINT NULL,
  amount DECIMAL(10,2) NOT NULL,
  currency CHAR(3) NOT NULL,
  status VARCHAR(12) NOT NULL,           -- Captured|Failed|Refunded
  breakdown JSON,
  psp_charge_ref VARCHAR(120),
  receipt_id VARCHAR(60),
  created_at TIMESTAMP NOT NULL,
  INDEX idx_user (user_id),
  UNIQUE(rental_id)
);

CREATE TABLE pay_penalties(
  id BIGINT PRIMARY KEY,
  rental_id BIGINT NOT NULL,
  type VARCHAR(20) NOT NULL,             -- overdue|out_of_zone|damage
  amount DECIMAL(10,2) NOT NULL,
  currency CHAR(3) NOT NULL,
  status VARCHAR(12) NOT NULL,           -- Pending|Charged|Failed
  reason VARCHAR(200),
  created_at TIMESTAMP NOT NULL,
  INDEX idx_rental (rental_id)
);

CREATE TABLE pay_payouts(
  id BIGINT PRIMARY KEY,
  provider_id BIGINT NOT NULL,
  period_start DATE NOT NULL,
  period_end DATE NOT NULL,
  amount DECIMAL(10,2) NOT NULL,
  status VARCHAR(12) NOT NULL,           -- Scheduled|Processing|Paid|Failed
  psp_payout_ref VARCHAR(120),
  created_at TIMESTAMP NOT NULL,
  paid_at TIMESTAMP NULL,
  UNIQUE(provider_id, period_start, period_end)
);

CREATE TABLE payments_outbox(
  id BIGINT PRIMARY KEY,
  aggregate_id BIGINT NOT NULL,
  event_type VARCHAR(80) NOT NULL,
  payload JSON NOT NULL,
  status VARCHAR(12) NOT NULL DEFAULT 'PENDING',
  attempts INT NOT NULL DEFAULT 0,
  created_at TIMESTAMP NOT NULL,
  published_at TIMESTAMP NULL
);
```

**Topología de eventos**

* **Salida:** `payments.method.verified`, `payments.authorized`, `payments.captured`, `payments.failed`, `payments.penalty.charged`, `payments.payout.settled`, `payments.refund.processed`.
* **Entrada:** `renting.rental.started`, `renting.rental.finished`, `renting.penalty.applied`, `providing.provider.verified`.

**Operación y observabilidad**

* **Logs** estructurados sin PII/PCI.
* **Métricas**: `payments_authorize_success_total`, `payments_capture_success_total`, `payments_payout_paid_total`, `psp_latency_seconds`.
* **Alertas**: tasa de fallo PSP > umbral; backlog de outbox.

**Seguridad**

* TLS, secretos en **KeyVault**.
* Webhooks con validación de firma y ventana de tiempo.
* Cumplimiento PCI-DSS (tokenización vía PSP; no almacenamos PAN/CVV).

#### 4.2.2.5. Bounded Context Software Architecture Component Level Diagrams 
Este diagrama representa la descomposición interna del container IAM Application, correspondiente al bounded context de identidad y autenticación (IAM) dentro de la plataforma de bicicletas. Se trata de un backend desarrollado bajo los principios de Clean Architecture y Domain-Driven Design (DDD), ilustrado en el Nivel 3 del C4 Model (Component Diagram).

<img src="/assets/images/Chapter-4/bdc1.png" alt="bdc1" width=auto>

Este diagrama muestra la descomposición interna del container Renting Application.

<img src="/assets/images/Chapter-4/bdc2.png" alt="bdc1" width=auto>

El Providing Bounded Context se centra en la gestión de los vehículos que los proveedores ponen a disposición de los usuarios.

<img src="/assets/images/Chapter-4/bdc3.png" alt="bdc1" width=auto>

Dominio Vehicles:

<img src="/assets/images/Chapter-4/dc2.png" alt="bdc1" width=auto>

#### 4.2.2.6. Bounded Context Software Architecture Code Level Diagrams 
##### 4.2.2.6.1. Bounded Context Domain Layer Class Diagrams 
Este diagrama de clases ilustra la capa de dominio del bounded context IAM, con sus Agregados, Entidades y Value Objects.

<img src="/assets/images/Chapter-4/uml1.png" alt="bdc1" width=auto>

Diagrama del dominio Renting:

<img src="/assets/images/Chapter-4/uml3.png" alt="bdc1" width=auto>

Diagrama del dominio Providing:

<img src="/assets/images/Chapter-4/uml5.png" alt="bdc1" width=auto>

Diagrama del dominio Vehicles:

<img src="/assets/images/Chapter-4/dc1.png" alt="bdc1" width=auto>

##### 4.2.2.6.2. Bounded Context Database Design Diagram
El siguiente diagrama muestra el diseño de la base de datos relacional para el contexto IAM, incluyendo las tablas principales relacionadas con usuarios, credenciales y verificaciones.

<img src="/assets/images/Chapter-4/uml2.png" alt="bdc1" width=auto>

Tabla: users
| Nombre           | Descripción                                                         |
|------------------|---------------------------------------------------------------------|
| id               | Identificador único del usuario (UUID, PK).                          |
| full_name        | Nombre completo del ciclista/proveedor.                              |
| username         | Nombre de usuario único (opcional, para login/display).              |
| email            | Correo electrónico único del usuario (identificador de login).       |
| status           | Estado del usuario: Pending, Active, Suspended.                      |
| reputation_avg   | Promedio de calificaciones recibidas por el usuario (0.00–5.00).     |
| reputation_count | Cantidad de calificaciones recibidas.                                |
| avatar_url       | URL de la foto de perfil (opcional).                                |
| created_at       | Fecha y hora de creación del registro.                              |
| updated_at       | Fecha y hora de la última actualización.                            |

Tabla: credentials
| Nombre              | Descripción                                                         |
|---------------------|---------------------------------------------------------------------|
| id                  | Identificador único de la credencial (UUID, PK).                    |
| user_id             | Referencia al usuario propietario (FK → users.id).                  |
| password_hash       | Hash de la contraseña (Argon2/BCrypt).                              |
| password_salt       | Salt usado en el hash (si aplica).                                  |
| mfa_enabled         | Booleano: indica si MFA/TOTP está activado.                         |
| failed_attempts     | Contador de intentos fallidos de login.                             |
| locked_until        | Timestamp hasta el cual la cuenta está bloqueada.                   |
| last_login_at       | Fecha y hora del último inicio de sesión exitoso.                   |
| password_changed_at | Fecha y hora de la última modificación de contraseña.               |

Tabla: verifications
| Nombre            | Descripción                                                           |
|-------------------|-----------------------------------------------------------------------|
| id                | Identificador único de la verificación (UUID, PK).                    |
| user_id           | Usuario relacionado (FK → users.id).                                  |
| token             | Token de verificación único enviado por email.                        |
| issued_at         | Fecha y hora en que se emitió el token.                               |
| expires_at        | Fecha y hora de expiración del token.                                 |
| used_at           | Fecha y hora en que el token fue usado (null si no usado).            |
| type              | Tipo de verificación (email, university_domain, etc.).                |
| university_domain | Dominio universitario validado (ej. `uni.edu`) — opcional.            |

Tabla: roles
| Nombre      | Descripción                                               |
|-------------|-----------------------------------------------------------|
| id          | Identificador único del rol (UUID, PK).                   |
| name        | Nombre del rol (User, Provider, Admin, etc.).             |
| grants      | Conjunto de permisos/alcances del rol (JSON / array).     |
| description | Descripción breve del propósito del rol.                  |

Tabla: user_roles
| Nombre      | Descripción                                               |
|-------------|-----------------------------------------------------------|
| user_id     | Referencia al usuario (FK → users.id).                    |
| role_id     | Referencia al rol (FK → roles.id).                        |
| assigned_at | Fecha y hora en que se asignó el rol.                     |
| granted_by  | (Opcional) ID del admin o proceso que asignó el rol.      |

Tabla: refresh_tokens (opcional, para sesiones seguras)
| Nombre      | Descripción                                               |
|-------------|-----------------------------------------------------------|
| id          | Identificador único del refresh token (UUID, PK).         |
| user_id     | Usuario asociado (FK → users.id).                         |
| token_hash  | Hash del refresh token (no se guarda en texto plano).     |
| issued_at   | Fecha y hora de emisión.                                  |
| expires_at  | Fecha y hora de expiración.                               |
| revoked     | Booleano: true si fue revocado.                           |
| revoked_at  | Fecha y hora de revocación (si aplica).                   |
| device_info | Metadata del dispositivo/navegador (opcional).            |


Contexto Renting:

<img src="/assets/images/Chapter-4/uml4.png" alt="bdc1" width=auto>

Tabla: rentals  
| Nombre         | Descripción                                                                 |
|----------------|-----------------------------------------------------------------------------|
| id             | Identificador único del alquiler (UUID, PK).                                |
| user_id        | Identificador del usuario que alquila (FK → users en IAM).                  |
| bicycle_id     | Identificador de la bicicleta alquilada (FK → bicycles en Inventory).       |
| station_start  | Estación donde inicia el alquiler (FK → stations).                         |
| station_end    | Estación donde termina el alquiler (FK → stations).                        |
| start_time     | Fecha y hora de inicio del alquiler.                                        |
| end_time       | Fecha y hora de fin del alquiler (puede ser NULL si está en curso).         |
| status         | Estado del alquiler: Active, Completed, Cancelled.                          |
| total_cost     | Costo total del alquiler calculado.                                         |
| created_at     | Fecha y hora de creación del registro.                                      |
| updated_at     | Fecha y hora de la última actualización.                                    |


Tabla: rental_details  
| Nombre        | Descripción                                                                 |
|---------------|-----------------------------------------------------------------------------|
| id            | Identificador único del detalle (UUID, PK).                                 |
| rental_id     | Identificador del alquiler asociado (FK → rentals).                         |
| segment_start | Punto de inicio del tramo (coordenadas GPS o estación).                     |
| segment_end   | Punto de fin del tramo (coordenadas GPS o estación).                        |
| distance_km   | Distancia recorrida en kilómetros en el tramo.                              |
| duration_min  | Duración del tramo en minutos.                                              |
| cost_segment  | Costo parcial asociado al tramo.                                            |
| created_at    | Fecha y hora de creación del registro.                                      |

Tabla: payments  
| Nombre        | Descripción                                                                 |
|---------------|-----------------------------------------------------------------------------|
| id            | Identificador único del pago (UUID, PK).                                    |
| rental_id     | Identificador del alquiler asociado (FK → rentals).                         |
| amount        | Monto pagado en la transacción.                                             |
| method        | Método de pago: CreditCard, DebitCard, Wallet, Cash.                        |
| status        | Estado del pago: Pending, Successful, Failed, Refunded.                     |
| transaction_at| Fecha y hora de la transacción.                                             |
| created_at    | Fecha y hora de creación del registro.                                      |


Tabla: stations  
| Nombre        | Descripción                                                                 |
|---------------|-----------------------------------------------------------------------------|
| id            | Identificador único de la estación (UUID, PK).                              |
| code          | Código único de la estación.                                                |
| name          | Nombre de la estación.                                                      |
| location      | Dirección o coordenadas de ubicación.                                       |
| capacity      | Número máximo de bicicletas que puede albergar.                             |
| available     | Cantidad de bicicletas disponibles en el momento.                           |
| created_at    | Fecha y hora de creación del registro.                                      |
| updated_at    | Fecha y hora de la última actualización.                                    |

Contexto Providing:
<img src="/assets/images/Chapter-4/uml6.png" alt="bdc1" width=auto>

Proveedor

| Nombre        | Descripción                                  |
|--------------|-----------------------------------------------|
| id_proveedor  | Identificador único del proveedor (PK).      |
| nombre       | Nombre o razón social del proveedor.        |
| email         | Correo electrónico del proveedor.                |
| telefono      | Número de contacto del proveedor.                |


Bicicleta

| Nombre        | Descripción                                              |
| ------------- | -------------------------------------------------------- |
| id\_vehiculo  | Identificador único del vehículo (PK).                   |
| tipo          | Tipo de vehículo (`bicicleta` o `scooter`).              |
| marca         | Marca del vehículo.                                      |
| modelo        | Modelo del vehículo.                                     |
| año           | Año de fabricación del vehículo.                         |
| id\_proveedor | Relación con el proveedor que registró el vehículo (FK). |
| id\_categoria | Relación con la categoría asignada (FK).                 |


Categoría

| Nombre        | Descripción                             |
|---------------|-----------------------------------------|
| id_categoria  | Identificador único de la categoría (PK). |
| nombre        | Nombre de la categoría (urbana, MTB, etc.). |
| descripcion   | Breve descripción de la categoría.       |


Historial

| Nombre        | Descripción                                          |
|---------------|------------------------------------------------------|
| id_historial  | Identificador único del registro en el historial (PK). |
| id_bicicleta  | Relación con la bicicleta registrada (FK).            |
| fecha         | Fecha y hora del cambio o evento.                    |
| estado        | Estado de la bicicleta (ej. activa, en reparación).  |
| comentario    | Observaciones o detalles adicionales.                |

Contexto Vehicles:
<img src="/assets/images/Chapter-4/er2.png" alt="bdc1" width=auto>

Usuario
| Nombre      | Descripción                          |
| ----------- | ------------------------------------ |
| id\_usuario | Identificador único del usuario (PK) |
| nombre      | Nombre completo                      |
| email       | Correo electrónico único             |
| telefono    | Número de contacto                   |
| created\_at | Fecha de creación                    |
| updated\_at | Fecha de última actualización        |

Vehiculo 
| Nombre         | Descripción                              |
| -------------- | ---------------------------------------- |
| id\_vehiculo   | Identificador único del vehículo (PK)    |
| tipo           | Tipo de vehículo (bicicleta o scooter)   |
| marca          | Marca del vehículo                       |
| modelo         | Modelo del vehículo                      |
| anio           | Año de fabricación                       |
| id\_proveedor  | FK al proveedor que registró el vehículo |
| id\_categoria  | FK a la categoría del vehículo           |
| serial\_number | Número de serie opcional                 |
| created\_at    | Fecha de creación                        |
| updated\_at    | Fecha de actualización                   |

Categoria
| Nombre        | Descripción                              |
| ------------- | ---------------------------------------- |
| id\_categoria | Identificador único de la categoría (PK) |
| nombre        | Nombre de la categoría                   |
| descripcion   | Breve descripción                        |
| created\_at   | Fecha de creación                        |
| updated\_at   | Fecha de última actualización            |

Historial de uso

| Nombre        | Descripción                                    |
| ------------- | ---------------------------------------------- |
| id\_historial | Identificador del registro de uso (PK)         |
| id\_vehiculo  | FK al vehículo usado                           |
| id\_usuario   | FK al usuario que usó el vehículo              |
| fecha\_inicio | Fecha y hora de inicio del uso                 |
| fecha\_fin    | Fecha y hora de fin del uso                    |
| estado        | Estado del uso (activo, finalizado, cancelado) |
| comentario    | Observaciones o notas                          |



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

(Insertar diagrama de componentes del contexto Providing)


<div id='4.2.3.1.'><h5>4.2.3.6. Bounded Context Software Architecture Code Level Diagram</h5></div>

<div id='4.2.3.1.'><h5>4.2.3.6.1 Bounded Context Domain Layer Class Diagrams</h5></div>

Este diagrama UML representa la arquitectura del contexto de gestión de bicicletas. Se organiza bajo principios de diseño orientado a objetos y sigue el enfoque CQRS.

La entidad principal es Bicycle, encargada de centralizar la información del vehículo.

Este contexto interactúa con:

* Renting (uso de bicicletas en alquiler)
* Tracking & Monitoring (ubicación y estado en tiempo real)
* IoT & Device Control (sensores y smart lock)
* IAM (propietario de la bicicleta)



<div id='4.2.3.1.'><h5>4.2.3.6.1 Bounded Context Domain Layer Class Diagrams</h5></div>

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

## 4.2.4. Bounded Context: Tracking & Monitoring  (OSCAR)

### 4.2.4.1. Domain Layer

### 4.2.4.2. Interface Layer

### 4.2.4.3. Application Layer

### 4.2.4.4. Infrastructure Layer

### 4.2.4.5. Bounded Context Software Architecture Component Level Diagrams

### 4.2.4.6. Bounded Context Software Architecture Code Level Diagrams

#### 4.2.4.6.1. Bounded Context Domain Layer Class Diagrams

#### 4.2.4.6.2. Bounded Context Database Design Diagram


  


  

