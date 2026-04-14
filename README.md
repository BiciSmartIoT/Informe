# Informe

##Capítulo III: Requirements Specification. 

### 3.1. User Stories.
Epics & User Stories – Sistema de Bicicletas IoT
EP01: Como usuario quiero registrarme, iniciar sesión y gestionar mi perfil, para tener acceso seguro y personalizado a la aplicación.

User Story ID | Título
US01 Registro de usuario estudiante
US02 Registro de usuario arrendador
US03 Iniciar sesión en la aplicación
US04 Recuperar contraseña
US05 Editar información de perfil
US06 Cerrar sesión

EP02: Como arrendador quiero publicar, editar y administrar mis bicicletas inteligentes para ponerlas a disposición de los usuarios.

User Story ID | Título
US07 Registrar una bicicleta IoT en la app
US08 Editar información de una bicicleta
US09 Marcar bicicleta como disponible/no disponible
US10 Eliminar bicicleta de la aplicación
US11 Consultar historial de alquileres

EP03: Como usuario quiero buscar y reservar bicicletas inteligentes disponibles para usarlas en mis traslados.

User Story ID | Título
US12 Buscar bicicletas por cercanía (GPS)
US13 Filtrar bicicletas por tipo o características
US14 Ver información detallada de la bicicleta y arrendador
US15 Realizar reserva de bicicleta
US16 Cancelar reserva
US17 Confirmar inicio de alquiler
US18 Finalizar alquiler

EP04: Como usuario quiero realizar pagos digitales de forma segura y como arrendador quiero recibir mis ingresos sin complicaciones.

User Story ID | Título
US19 Vincular método de pago (Yape, Plin, tarjeta)
US20 Pagar alquiler automáticamente
US21 Recibir confirmación de pago
US22 Recibir notificación de penalización por tiempo excedido
US23 Consultar historial de pagos
US24 Arrendador recibe liquidación automática

EP05: Como usuario quiero controlar el alquiler mediante tecnología IoT para tener una experiencia automatizada y sin contacto.

User Story ID | Título
US25 Desbloquear bicicleta mediante app (Smart Lock)
US26 Bloquear bicicleta automáticamente al finalizar
US27 Inicio automático de alquiler mediante sensor
US28 Detección de bicicleta en uso (sensor de movimiento)

EP06: Como usuario quiero visualizar y monitorear información en tiempo real de la bicicleta para mayor control y seguridad.

User Story ID | Título
US29 Ver ubicación en tiempo real de la bicicleta (GPS)
US30 Seguimiento de recorrido durante el alquiler
US31 Visualizar estado de la bicicleta (disponible/en uso)
US32 Registro de historial de rutas

EP07: Como usuario quiero sentirme seguro utilizando bicicletas inteligentes dentro de la plataforma.

User Story ID | Título
US33 Detección de manipulación o intento de robo
US34 Alerta por salida de zona permitida (geocerca)
US35 Botón de emergencia (SOS)
US36 Notificaciones de seguridad en tiempo real

EP08: Como usuario quiero recibir notificaciones relevantes para estar informado sobre mis actividades.

User Story ID | Título
US37 Notificaciones de reserva
US38 Notificaciones de inicio y fin de alquiler
US39 Notificaciones de pago
US40 Notificaciones de incidencias o alertas

EP09: Como administrador quiero gestionar usuarios y bicicletas inteligentes para asegurar el correcto funcionamiento del sistema.

User Story ID | Título
US41 Acceder a panel de control con métricas
US42 Gestionar usuarios (activar, suspender, eliminar)
US43 Gestionar bicicletas registradas
US44 Monitorear bicicletas en tiempo real (IoT)

EP10: Como visitante del sitio quiero explorar la landing page para conocer la aplicación y decidir si registrarme.

User Story ID | Título
US45 Visualizar información general del servicio
US46 Ver características y beneficios
US47 Acceder a registro o inicio de sesión
US48 Descargar la aplicación

Technical Stories – Sistema de Bicicletas IoT
Story ID | User | Priority | Epic

TS01 Programador Alta EP01

Título:
Implementar endpoints de autenticación y gestión de usuarios (IAM)

Descripción:
Desarrollar endpoints RESTful para el registro, autenticación, recuperación de contraseña y gestión de perfiles de usuarios.

Criterios de Aceptación:
Escenario 1: Endpoint de registro
Dado que el sistema recibe datos válidos, cuando procesa la solicitud, entonces debe crear el usuario correctamente.

Escenario 2: Endpoint de login
Dado que el sistema recibe credenciales válidas, cuando autentica, entonces debe devolver un token JWT.

Escenario 3: Recuperación de contraseña
Dado que el usuario solicita recuperación, cuando el sistema procesa, entonces debe enviar enlace.

Escenario 4: Actualización de perfil
Dado que el usuario envía cambios, cuando el sistema procesa, entonces debe guardar correctamente.

TS02 Programador Alta EP02

Título:
Endpoints para gestión de bicicletas IoT (Providing)

Descripción:
Implementar endpoints para registrar, editar, eliminar y consultar bicicletas inteligentes con sus datos IoT asociados.

Criterios de Aceptación:
Escenario 1: Registro de bicicleta
Dado que el sistema recibe datos válidos, entonces crea la bicicleta con ID único.

Escenario 2: Edición
Dado que se actualizan datos, entonces guarda cambios correctamente.

Escenario 3: Eliminación
Dado que se elimina, entonces se realiza eliminación lógica.

Escenario 4: Consulta
Dado que se solicita listado, entonces devuelve bicicletas registradas.

TS03 Programador Media EP03

Título:
Endpoints de catálogo y búsqueda con geolocalización (Vehicles + GPS)

Descripción:
Crear endpoints para listar, filtrar y ubicar bicicletas usando coordenadas GPS.

Criterios de Aceptación:
Escenario 1: Listado general
Devuelve bicicletas disponibles.

Escenario 2: Filtro por ubicación
Devuelve bicicletas cercanas según coordenadas.

Escenario 3: Detalle
Devuelve información completa del vehículo.

TS04 Programador Alta EP03

Título:
Endpoints de reservas y alquiler inteligente (Renting + IoT)

Descripción:
Desarrollar endpoints para gestionar reservas y sincronizar el inicio/fin del alquiler con dispositivos IoT.

Criterios de Aceptación:
Escenario 1: Crear reserva
Se registra correctamente.

Escenario 2: Cancelar reserva
Actualiza estado.

Escenario 3: Inicio de alquiler
Se activa al desbloquear bicicleta.

Escenario 4: Finalizar alquiler
Se cierra correctamente.

TS05 Programador Alta EP04

Título:
Endpoints de pagos, penalizaciones y liquidaciones (Payments)

Descripción:
Implementar endpoints para pagos automáticos basados en tiempo de uso IoT.

Criterios de Aceptación:
Escenario 1: Registrar pago
Se guarda correctamente.

Escenario 2: Penalización
Se genera por tiempo excedido.

Escenario 3: Liquidación
Se transfiere al arrendador.

Escenario 4: Historial
Se consulta correctamente.

TS06 Programador Alta EP05

Título:
Integración con Smart Lock (candado inteligente)

Descripción:
Implementar comunicación entre backend y dispositivo IoT para bloquear/desbloquear bicicletas.

Criterios de Aceptación:
Escenario 1: Desbloqueo remoto
Dado que el usuario inicia alquiler, entonces el sistema envía señal al candado.

Escenario 2: Bloqueo automático
Dado que finaliza alquiler, entonces el candado se bloquea.

Escenario 3: Error de comunicación
Si falla, se registra evento y alerta.

TS07 Programador Alta EP06

Título:
Procesamiento de datos GPS en tiempo real

Descripción:
Implementar recepción, almacenamiento y consulta de datos de geolocalización.

Criterios de Aceptación:
Escenario 1: Recepción de coordenadas
El dispositivo envía datos y el sistema los guarda.

Escenario 2: Actualización
Se actualiza ubicación en tiempo real.

Escenario 3: Consulta
Se puede visualizar ubicación actual.

TS08 Programador Media EP06

Título:
Procesamiento de sensores de uso (IoT)

Descripción:
Implementar lógica para interpretar datos de sensores (movimiento, uso, tiempo).

Criterios de Aceptación:
Escenario 1: Detección de movimiento
El sistema identifica bicicleta en uso.

Escenario 2: Registro de tiempo
Se calcula duración automáticamente.

Escenario 3: Estado
Actualiza estado de bicicleta.

TS09 Programador Alta EP07

Título:
Sistema de alertas IoT y seguridad

Descripción:
Desarrollar lógica para detectar eventos de riesgo (robo, geocerca, manipulación).

Criterios de Aceptación:
Escenario 1: Manipulación
Se detecta intento de robo.

Escenario 2: Geocerca
Se detecta salida de zona.

Escenario 3: Notificación
Se envía alerta en tiempo real.

TS10 Programador Media EP08

Título:
Sistema de notificaciones (Push)

Descripción:
Implementar envío de notificaciones para eventos del sistema.

Criterios de Aceptación:
Escenario 1: Reserva
Notifica confirmación.

Escenario 2: Alquiler
Notifica inicio/fin.

Escenario 3: Alertas
Notifica incidencias IoT.

TS11 Programador Media EP09

Título:
Panel administrativo y monitoreo en tiempo real

Descripción:
Desarrollar dashboard para visualizar métricas y estado de bicicletas IoT.

Criterios de Aceptación:
Escenario 1: Métricas
Muestra usuarios, alquileres e ingresos.

Escenario 2: Monitoreo
Muestra bicicletas en tiempo real.

Escenario 3: Alertas
Visualiza incidencias.

Spike Stories – Sistema de Bicicletas IoT
Story ID | User | Priority | Epic

SPIKE01 Programador Media EP03

Título:
Geolocalización y visualización en mapas (GPS)

Descripción:
Evaluar la integración de servicios de mapas para mostrar bicicletas en tiempo real usando datos GPS de dispositivos IoT.

Criterios de Aceptación:
Escenario 1: Comparación de servicios de mapas
Dado que se evalúan Google Maps y Mapbox, cuando se analizan precisión, costos y facilidad de integración, entonces se obtiene un cuadro comparativo.

Escenario 2: Prototipo de visualización
Dado que se reciben coordenadas GPS de prueba, cuando se muestran en el mapa, entonces se valida la visualización en tiempo real.

Escenario 3: Selección de proveedor
Dado que se analizan límites y costos, cuando se toma una decisión, entonces se documenta la recomendación.

SPIKE02 Programador Media EP05

Título:
Comunicación con dispositivos IoT (Smart Lock)

Descripción:
Investigar protocolos de comunicación (MQTT, HTTP, WebSocket) para interactuar con el candado inteligente de las bicicletas.

Criterios de Aceptación:
Escenario 1: Evaluación de protocolos
Dado que se analizan MQTT, HTTP y WebSocket, cuando se comparan latencia y confiabilidad, entonces se documenta la mejor opción.

Escenario 2: Prueba de comunicación
Dado que se simula un dispositivo IoT, cuando se envía un comando de bloqueo/desbloqueo, entonces se valida la comunicación.

Escenario 3: Recomendación técnica
Dado que se analizan resultados, cuando se selecciona el protocolo, entonces se documenta la arquitectura recomendada.

SPIKE03 Programador Alta EP06

Título:
Procesamiento de datos en tiempo real (sensores IoT)

Descripción:
Evaluar cómo procesar datos de sensores (movimiento, tiempo de uso) en tiempo real.

Criterios de Aceptación:
Escenario 1: Prueba de ingestión de datos
Dado que el dispositivo envía datos continuamente, cuando el sistema los recibe, entonces se valida la captura en tiempo real.

Escenario 2: Evaluación de almacenamiento
Dado que se generan múltiples eventos, cuando se analizan bases de datos (SQL vs NoSQL), entonces se define la mejor opción.

Escenario 3: Conclusión de arquitectura
Dado que se evalúan resultados, cuando se define la solución, entonces se documenta la estrategia de procesamiento.

SPIKE04 Programador Alta EP07

Título:
Sistema de alertas y seguridad IoT

Descripción:
Investigar cómo detectar eventos de riesgo como robo, manipulación o salida de zona (geocerca).

Criterios de Aceptación:
Escenario 1: Evaluación de eventos de riesgo
Dado que se analizan posibles escenarios (robo, caída, manipulación), cuando se clasifican, entonces se obtiene un listado de eventos críticos.

Escenario 2: Prueba de alertas
Dado que se simulan eventos, cuando el sistema detecta anomalías, entonces se valida el envío de alertas.

Escenario 3: Definición de reglas
Dado que se analizan los resultados, cuando se establecen reglas, entonces se documenta el comportamiento del sistema.

SPIKE05 Programador Media EP08

Título:
Notificaciones en tiempo real (Push + eventos IoT)

Descripción:
Evaluar servicios para envío de notificaciones basadas en eventos del sistema e IoT.

Criterios de Aceptación:
Escenario 1: Comparación de servicios
Dado que se analizan Firebase y OneSignal, cuando se comparan características, entonces se documenta la mejor opción.

Escenario 2: Prueba de envío
Dado que ocurre un evento (inicio de alquiler), cuando se envía notificación, entonces se valida la entrega.

Escenario 3: Selección final
Dado que se evalúan resultados, cuando se elige solución, entonces se documenta la decisión.

SPIKE06 Programador Alta EP09

Título:
Escalabilidad de sistema IoT en la nube

Descripción:
Evaluar la infraestructura necesaria para soportar múltiples bicicletas conectadas en tiempo real.

Criterios de Aceptación:
Escenario 1: Pruebas de carga
Dado que múltiples dispositivos envían datos, cuando se simula tráfico, entonces se obtienen métricas.

Escenario 2: Evaluación de servicios cloud
Dado que se analizan AWS IoT, Azure IoT y Firebase, cuando se comparan, entonces se obtiene recomendación.

Escenario 3: Arquitectura propuesta
Dado que se analizan resultados, cuando se define solución, entonces se documenta arquitectura escalable.

SPIKE07 Programador Alta EP01

Título:
Seguridad y protección de datos

Descripción:
Evaluar mecanismos de seguridad para proteger datos de usuarios y dispositivos IoT.

Criterios de Aceptación:
Escenario 1: Análisis de normativas
Dado que se revisan estándares (ISO 27001, GDPR), cuando se analizan requisitos, entonces se documentan.

Escenario 2: Identificación de riesgos
Dado que se evalúa el sistema, cuando se detectan vulnerabilidades, entonces se listan.

Escenario 3: Propuesta de seguridad
Dado que se identifican riesgos, cuando se definen medidas, entonces se documenta solución.


### 3.2. Impact Mapping.
ya hay uimg
### 3.3. Product Backlog.

Product Backlog – Sistema de Bicicletas IoT (Completo)
Orden	ID	Tipo	Título	Story Points
1	US01	User	Registro de usuario estudiante	5
2	US02	User	Registro de usuario arrendador	5
3	TS01	Technical	Endpoints de autenticación (IAM)	8
4	US03	User	Iniciar sesión	3
5	US04	User	Recuperar contraseña	3
6	US05	User	Editar perfil	3
7	US06	User	Cerrar sesión	2
8	US07	User	Registrar bicicleta IoT	5
9	TS02	Technical	Endpoints de gestión de bicicletas	8
10	US08	User	Editar bicicleta	3
11	US09	User	Disponibilidad de bicicleta	2
12	US10	User	Eliminar bicicleta	2
13	US12	User	Buscar bicicletas por GPS	5
14	TS03	Technical	Endpoints de búsqueda con geolocalización	5
15	SPIKE01	Spike	Evaluación de mapas (Google Maps vs Mapbox)	3
16	US13	User	Filtrar bicicletas	3
17	US14	User	Ver detalles de bicicleta	3
18	US15	User	Reservar bicicleta	5
19	TS04	Technical	Endpoints de reservas y alquiler	8
20	US16	User	Cancelar reserva	3
21	US17	User	Confirmar inicio de alquiler	5
22	US18	User	Finalizar alquiler	5
23	US19	User	Vincular método de pago	3
24	TS05	Technical	Endpoints de pagos y liquidaciones	8
25	US20	User	Pago automático	5
26	US21	User	Confirmación de pago	2
27	US23	User	Historial de pagos	3
28	SPIKE02	Spike	Comunicación con dispositivos IoT (MQTT vs HTTP)	5
29	TS06	Technical	Integración con Smart Lock	13
30	US25	User	Desbloquear bicicleta (Smart Lock)	8
31	US26	User	Bloqueo automático	5
32	US27	User	Inicio automático (sensor)	5
33	TS08	Technical	Procesamiento de sensores IoT	8
34	US28	User	Detección de uso	3
35	SPIKE03	Spike	Procesamiento de datos en tiempo real	5
36	TS07	Technical	Procesamiento GPS en tiempo real	13
37	US29	User	Ubicación en tiempo real	8
38	US30	User	Seguimiento de recorrido	5
39	US31	User	Estado de bicicleta	3
40	US32	User	Historial de rutas	3
41	TS09	Technical	Sistema de alertas IoT	8
42	SPIKE04	Spike	Seguridad IoT y eventos de riesgo	5
43	US33	User	Detección de robo	8
44	US34	User	Geocerca	5
45	US35	User	Botón SOS	3
46	US36	User	Notificaciones de seguridad	3
47	TS10	Technical	Sistema de notificaciones push	5
48	SPIKE05	Spike	Evaluación de servicios de notificaciones	3
49	US37	User	Notificaciones de reserva	2
50	US38	User	Notificaciones de alquiler	2
51	US39	User	Notificaciones de pago	2
52	US40	User	Notificaciones de incidencias	3
53	TS11	Technical	Panel admin y monitoreo	5
54	SPIKE06	Spike	Escalabilidad en la nube (AWS/Azure)	5
55	US41	User	Panel de métricas	5
56	US42	User	Gestionar usuarios	3
57	US43	User	Gestionar bicicletas	3
58	US44	User	Monitoreo en tiempo real	8
59	SPIKE07	Spike	Seguridad y protección de datos	5
60	US45	User	Ver landing page	2
61	US46	User	Ver beneficios	2
62	US47	User	Acceso a login desde landing	2
63	US48	User	Descargar app	2
