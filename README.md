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
<div id='4.2.3.'><h4>4.2.3. Bounded Context: Notifications</h4></div>
<div id='4.2.3.1.'><h5>4.2.1.1. Domain Layer</h5></div>
Subcapamodel<br>


| Tipo         | Nombre                        | Descripción                                               | Responsabilidad Principal                             | Relación con otros elementos                   |
| ------------ | ----------------------------- | --------------------------------------------------------- | ----------------------------------------------------- | ---------------------------------------------- |
| Aggregate    | Notification                  | Representa una notificación enviada al usuario.           | Centralizar la gestión de notificaciones del sistema. | Relacionado con User (IAM), Rental y Payments. |
| Value Object | NotificationType              | Tipo de notificación (reserva, pago, alerta, incidencia). | Clasificar las notificaciones.                        | Asociado a Notification.                       |
| Value Object | NotificationStatus            | Estado de la notificación (enviada, leída, fallida).      | Controlar el estado de entrega.                       | Asociado a Notification.                       |
| Value Object | Message                       | Contenido del mensaje de la notificación.                 | Estandarizar el texto enviado al usuario.             | Asociado a Notification.                       |
| Command      | SendNotificationCommand       | Enviar una notificación.                                  | Crear y despachar una notificación al usuario.        | Usa Notification.                              |
| Command      | MarkAsReadNotificationCommand | Marcar notificación como leída.                           | Actualizar estado de lectura.                         | Usa Notification.                              |
| Command      | SendBulkNotificationCommand   | Enviar múltiples notificaciones.                          | Gestionar envío masivo.                               | Usa Notification.                              |
| Query        | GetUserNotificationsQuery     | Obtener notificaciones de un usuario.                     | Listar notificaciones del usuario.                    | Consulta Notification.                         |
| Query        | GetUnreadNotificationsQuery   | Obtener notificaciones no leídas.                         | Filtrar notificaciones pendientes.                    | Consulta Notification.                         |
| Query        | GetNotificationDetailQuery    | Obtener detalle de notificación.                          | Mostrar información específica.                       | Consulta Notification.                         |

Sub-capa Services<br>

| Tipo      | Nombre                     | Descripción                               | Responsabilidad Principal                   | Relación con otros elementos                                          |
| --------- | -------------------------- | ----------------------------------------- | ------------------------------------------- | --------------------------------------------------------------------- |
| Interface | NotificationCommandService | Servicio para comandos de notificaciones  | Declarar métodos para envío y actualización | Implementado por NotificationCommandServiceImpl. Usado en Application |
| Interface | NotificationQueryService   | Servicio para consultas de notificaciones | Declarar métodos de consulta                | Implementado por NotificationQueryServiceImpl. Usado en Application   |



<div id='4.2.3.1.'><h5>4.2.3.2. Interface Layer</h5></div>

Sub-capa REST<br>

| Tipo       | Nombre                                       | Descripción                             | Responsabilidad Principal              | Relación con otros elementos     |
| ---------- | -------------------------------------------- | --------------------------------------- | -------------------------------------- | -------------------------------- |
| Controller | NotificationController                       | Controlador REST para notificaciones    | Gestiona solicitudes de notificaciones | Usa services y resources         |
| Resource   | NotificationRequestResource                  | Estructura de petición de notificación  | Representa datos de entrada            | Usado por NotificationController |
| Resource   | NotificationResponseResource                 | Estructura de respuesta de notificación | Devuelve datos al cliente              | Usado por NotificationController |
| Assembler  | SendNotificationCommandFromResourceAssembler | Convierte request en comando            | Traducir entrada a dominio             | Usado por NotificationController |
| Assembler  | NotificationResourceFromEntityAssembler      | Convierte entidad a response            | Traducir dominio a salida              | Usado por NotificationController |



<div id='4.2.3.1.'><h5>4.2.3.3. Aplications Layer</h5></div>

Sub-capa Internal

| Tipo    | Nombre                         | Descripción                                  | Responsabilidad Principal      | Relación con otros elementos          |
| ------- | ------------------------------ | -------------------------------------------- | ------------------------------ | ------------------------------------- |
| Service | NotificationCommandServiceImpl | Implementación de comandos de notificaciones | Ejecutar envío y actualización | Implementa NotificationCommandService |
| Service | NotificationQueryServiceImpl   | Implementación de consultas                  | Obtener notificaciones         | Implementa NotificationQueryService   |



<div id='4.2.3.1.'><h5>4.2.3.4. Infrastructure Layer</h5></div>


Sub-capa Infrastructure <br>


| Tipo       | Nombre                 | Descripción                    | Responsabilidad Principal          | Relación con otros elementos |
| ---------- | ---------------------- | ------------------------------ | ---------------------------------- | ---------------------------- |
| Repository | NotificationRepository | Persistencia de notificaciones | Guardar y consultar notificaciones | Relacionado con Notification |


<div id='4.2.3.1.'><h5>4.2.3.5. Bounded Context Software Architecture Component Level Diagrams</h5></div>

(Insertar diagrama de componentes del contexto Notifications)


<div id='4.2.3.1.'><h5>4.2.3.6. Bounded Context Software Architecture Code Level Diagram</h5></div>

<div id='4.2.3.1.'><h5>4.2.3.6.1 Bounded Context Domain Layer Class Diagrams</h5></div>

Este diagrama UML representa la arquitectura del sistema de notificaciones. Se basa en el enfoque CQRS, separando comandos (envío y actualización de notificaciones) de consultas (visualización y filtrado).

La entidad principal es Notification, que gestiona toda la comunicación hacia el usuario.

Este contexto interactúa con:

Renting (eventos de reserva y alquiler)
Payments (confirmaciones y penalizaciones)
IoT & Device Control (alertas técnicas)
IAM (usuarios)

<div id='4.2.3.1.'><h5>4.2.3.6.1 Bounded Context Domain Layer Class Diagrams</h5></div>

NOTIFICATIONS

Propósito: Registro de notificaciones enviadas a usuarios

| Campo      | Tipo      | Descripción                      |
| ---------- | --------- | -------------------------------- |
| id         | Long (PK) | Identificador de la notificación |
| user_id    | Long (FK) | Usuario receptor                 |
| type       | varchar   | Tipo de notificación             |
| message    | varchar   | Contenido                        |
| status     | varchar   | Estado (enviada, leída)          |
| created_at | datetime  | Fecha de creación                |





  


  

