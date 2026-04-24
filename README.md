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

###  Opción 3 (RECOMENDADA)

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

<div id='4.2.1.1.'><h5>4.2.1.6. Bounded Context Software Architecture Code Level Diagram</h5></div>

<div id='4.2.1.1.'><h5>4.2.1.6.1 Bounded Context Domain Layer Class Diagrams</h5></div>

Este diagrama UML representa la arquitectura del flujo de alquiler de bicicletas inteligentes. Se basa en principios de diseño orientado a objetos y sigue el enfoque CQRS (Command Query Responsibility Segregation).

Las entidades principales son Rental y Reservation, que gestionan el ciclo completo del alquiler.

Se interactúa con otros bounded contexts como:

* Providing (bicicletas)
* Payments (pagos)
* IoT & Device Control (inicio y fin físico del alquiler)
<div id='4.2.1.1.'><h5>4.2.1.6.1 Bounded Context Domain Layer Class Diagrams</h5></div>

RENTALS

Propósito: Registro de alquileres realizados

| Campo      | Tipo      | Descripción                |
| ---------- | --------- | -------------------------- |
| id         | Long (PK) | Identificador del alquiler |
| user_id    | Long (FK) | Referencia a usuario       |
| bicycle_id | Long (FK) | Referencia a bicicleta     |
| start_time | datetime  | Fecha de inicio            |
| end_time   | datetime  | Fecha de fin               |
| status     | varchar   | Estado del alquiler        |
| total_cost | decimal   | Costo total                |
chequear esto

RESERVATIONS

Propósito: Registro de reservas

| Campo            | Tipo      | Descripción      |
| ---------------- | --------- | ---------------- |
| id               | Long (PK) | Identificador    |
| user_id          | Long (FK) | Usuario          |
| bicycle_id       | Long (FK) | Bicicleta        |
| reservation_time | datetime  | Fecha de reserva |
| expiration_time  | datetime  | Expiración       |
| status           | varchar   | Estado           |





---------------------------------------------------------------
<div id='4.2.3.'><h4>4.2.3. Bounded Context: Providing</h4></div>
<div id='4.2.3.1.'><h5>4.2.1.1. Domain Layer</h5></div>
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




  


  

