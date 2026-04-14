<div align="center" style="display: flex; flex-direction: column; justify-content: center; align-items: center; height: 100vh; line-height: 1.8; font-family: 'Segoe UI', Arial, sans-serif;">


<img src="/assets/images/Chapter-1/logoupc.png" alt="Logo UPC" width="180">

### **Universidad Peruana de Ciencias Aplicadas**

### **Ingeniería de Software**

#### **Periodo 202610**

#### Desarollo De Soluciones IOT


#### **NRC:** 6766

#### **Docente:** LEON BACA,MARCO ANTONIO

---

#### “Informe de Trabajo Final”

**Nombre del StartUp:** *BiciSmartIOT*  
**Nombre del Producto:** *BiciSmartIOT*

---

#### Relación de Integrantes

U20231a804 — Bustamante Leveau, Cameron Charllotte   

U202311745 — Uribe Livia, Renzo Sebastián

U202311842 — Espinoza Quijandria, Oscar Leonardo

U202311828 — Landauri Preciado, Stephano Mayrzon

U202220219 — Belahonia Miranda, Fabrisio

---

Abril, 2026


<br><br>


</div>




## Registro de Versiones del Informe

|   Versión |   Fecha   |           Autor           |               Descripción  de modificación               |
|-----------|-----------|---------------------------|-------------------------------------------|
|   TB1     |     13/04/2026      |                           |                                           |
|   TP      |           |                           |                                           |
|   TB2     |           |                           |                                           |
|   TF      |           |                           |                                           |

<h2> Student Outcome</h2>

<table border="1" cellpadding="8" cellspacing="0" width="100%">
  <thead>
    <tr>
      <th>Criterio específico</th>
      <th>Acciones realizadas</th>
      <th>Conclusiones</th>
    </tr>
  </thead>
  <tbody>
    <tr>
    <tr>
      <td></td>
      <td>
      </td>
    </tr>
  </tbody>
</table>


## Project Report Collaboration Insights 

### **TB1**  

## Objetivos SMART



- [Capítulo I: Presentación](#capítulo-i-presentación)
  - [1.1. Startup Profile](#1.1.-startup-profile)
    - [1.1.1. Descripción de la Startup](#1.1.1.-descripción-de-la-startup)
    - [1.1.2. Perfiles de integrantes del equipo](#1.1.2.-perfiles-de-integrantes-del-equipo)
  - [1.2. Solution Profile](#1.2.-solution-profile)
    - [1.2.1. Antecedentes y problemática](#1.2.1.-antecedentes-y-problemática)
    - [1.2.2. Lean UX Process](#1.2.2.-lean-ux-process)
      - [1.2.2.1. Lean UX Problem Statements](#1.2.2.1.-lean-ux-problem-statements)
      - [1.2.2.2. Lean UX Assumptions](#1.2.2.2.-lean-ux-assumptions)
      - [1.2.2.3. Lean UX Hypothesis Statements](#1.2.2.3.-lean-ux-hypothesis-statements)
      - [1.2.2.4. Lean UX Canvas](#1.2.2.4.-lean-ux-canvas)
  - [1.3. Segmentos objetivo](#1.3.-segmentos-objetivo)
  



# Capítulo I: Presentación

## 1.1. Startup Profile

### 1.1.1. Descripción de la Startup

BiciSmartIOT es una startup tecnológica dedicada a la seguridad en el ámbito de la micromovilidad, enfocada principalmente en ciclistas urbanos y deportivos que buscan una forma precisa, segura y confiable de proteger sus bicicletas contra robos. A través de nuestra solución integral basada en el Internet de las Cosas (IoT), los usuarios pueden rastrear en tiempo real sus vehículos y recibir notificaciones ante movimientos anómalos o emergencias, generando confianza y protección de su inversión tanto en la ciudad como en viajes de ruta. De esta manera, promovemos un entorno de transporte más seguro, protegiendo un medio de transporte sustentable y mitigando la principal causa de desuso: el miedo al asalto a un medio desprotegido.

Misión: Facilitar la adopción de bicicletas como transporte sostenible mediante una plataforma y dispositivos de hardware IoT confiables, accesibles e inteligentes para la protección activa en la micromovilidad.

Visión: Convertirnos en la solución y aplicación líder de monitoreo y seguridad para micromovilidad en la región, reconocida por impulsar la confianza, la protección del patrimonio y la innovación.

Valores: Confiabilidad, Responsabilidad, Seguridad, Innovación y Prevención tecnológica.


### 1.1.2. Perfiles de integrantes del equipo

| Integrante | Descripción | Conocimiento |
|------------|-------------|--------------|
| U20231a804 — Bustamante Leveau, Cameron Charllotte ![Integrante-Cameron](/assets/images/chapter-I/foto_cameron.png) |  |
| U202311745 — Uribe Livia, Renzo Sebastián ![Integrante-Renzo](/assets/images/Chapter-1/Renzo.png) |Cuento con una sólida formación en desarrollo Full Stack, con dominio en tecnologías como React, Vue.js y Node.js, además de experiencia en bases de datos relacionales y no relacionales (SQL y MongoDB). Mi perfil técnico se complementa con habilidades en desarrollo móvil nativo usando Kotlin y Jetpack Compose. Me distingo por mi enfoque proactivo y mi capacidad para gestionar proyectos académicos con éxito, siempre orientado a crear soluciones digitales innovadoras en el ámbito de la inteligencia artificial y la productividad. |
| U202311842 — Espinoza Quijandria, Oscar Leonardo ![Integrante-Oscar](/assets/images/Chapter-1/Oscar.jpg) | |
| U202311828 — Landauri Preciado, Stephano Mayrzon ![Integrante-Stephano](/assets/images/Chapter-1/Stephano-Landauri.jpg) | |
| U202220219 — Belahonia Miranda, Fabrisio ![Integrante-Fabrisio](/assets/images/Chapter-1/fabrisio.png) | |

## 1.2. Solution Profile

### 1.2.1. Antecedentes y problemática

Para el análisis de los antecedentes y la problemática se ha empleado la técnica de las 5 'W's y 2 'H's:

*   **¿Quién? (Who):** Ciclistas urbanos y deportistas aficionados que utilizan sus bicicletas con frecuencia como medio de transporte principal, por salud o recreación.
*   **¿Qué ocurre? (What):** Incremento en el índice de robos de bicicletas estacionadas y una alta incidencia de accidentes o caídas en las que el ciclista no recibe asistencia inmediata por la falta de un sistema de prevención y alerta automática.
*   **¿Dónde? (Where):** Principalmente en vías metropolitanas con alto tráfico vehicular, estacionamientos en la vía pública frente a edificios o universidades, y durante rutas en carreteras o zonas remotas.
*   **¿Cuándo? (When):** Ocurre diariamente en todos los rangos horarios. En el día, el problema recae cuando estacionan su vehículo en un espacio público de forma temporal, mientras que de noche o en vías alejadas las condiciones favorecen atracos directos o accidentes no divisados por terceros.
*   **¿Por qué sucede? (Why):** Porque las medidas de seguridad tradicionales, tales como las cadenas y candados convencionales, son elementos vulnerables y pasivos. No advierten al dueño de la situación, y no existe una infraestructura estandarizada de alarmas o alertas inmediatas vinculadas a IoT que sean económicamente viables para este público.
*   **¿Cómo sucede? (How):** Generalmente, el robo ocurre de forma rápida vulnerando el control perimetral (cortando el candado de seguridad, forzándolo, etc.), para luego darse a la fuga sin que la víctima tenga un sistema rápido de geolocalización. En colisiones, ocurre de manera repentina contra un vehículo externo, perdiendo la posibilidad de solicitar auxilio inmediato si el usuario queda inhabilitado.
*   **¿Cuánto afecta? (How Much):** Produce un evidente daño económico frente a la pérdida total de la inversión para la máquina, e impacta física y mentalmente al ciclista, desincentivando directamente a los demás habitantes acerca de la viabilidad de la bicicleta como medio de transporte seguro.

### 1.2.2. Lean UX Process

#### 1.2.2.1. Lean UX Problem Statements
Hemos observado que los ciclistas urbanos y deportivos en la actualidad enfrentan altos niveles de inseguridad respecto a los robos sigilosos o violentos de sus bicicletas, y a los riesgos de accidentes viales fuera del radar de terceros. Los sistemas de resguardo con candados tradicionales carecen de capacidades de rastreo o notificación remota. 
¿Cómo podríamos crear un sistema inteligente, accesible y discreto, apoyado en la tecnología del Internet de las Cosas y de fácil integración con aplicativos móviles, que permita a los ciclistas rastrear sus vehículos 24/7 y que sea capaz de alertar a contactos de emergencia en caso de detección de anomalías repentinas en la integridad del equipo o manipulación sin autorización?

#### 1.2.2.2. Lean UX Assumptions
1.  **Creemos que** los usuarios dueños de bicicletas de mediano a alto valor adquisitivo estarán muy dispuestos a invertir en la compra de un dispositivo IoT de seguridad si se les garantiza la visibilidad de su posición de forma continua.
2.  **Creemos que** los ciclistas valorarán predominantemente recibir notificaciones instantáneas (push) cada vez que el microcontrolador de la bicicleta reporte un movimiento irregular.
3.  **Creemos que** es factible ingeniar una propuesta IoT en un tamaño reducido, con una batería prolongada para que se adhiera naturalmente en el diseño geométrico de la bicicleta (fácil fijamiento).
4.  **Creemos que** los canales comunicativos más eficientes son las tecnologías del ecosistema de área amplia (IoT/SIM) en conjunto a Low Energy Bluetooth para evitar desincronizaciones en un radio donde la persona pierda de vista la bicicleta.

#### 1.2.2.3. Lean UX Hypothesis Statements
*   **Creemos que** al brindar un dispositivo IoT con conectividad y rastreo integrativo que se enlazará a la plataforma móvil del usuario, **lograremos** reducir considerablemente la zozobra y la cifra de ciclistas que pierden sus bicicletas irreparablemente. **Lo sabremos cuando** veamos que al menos la mitad de nuestros usuarios del periodo Early Adopters declaren una mejora drástica en el índice de confianza durante nuestras primeras encuestas de seguimiento.
*   **Creemos que** al incorporar la habilidad de enviar señales de accidentes al microcontrolador a partir de su módulo analítico de movimiento, **lograremos** entregarle un segundo pilar de valor centrado en la protección humana. **Lo sabremos cuando** se registre un grado de retención igual o mayor al 80% después del tercer mes de uso, denotando el peso vital de las funciones preventivas de salud del hardware.

#### 1.2.2.4. Lean UX Canvas

| Componente | Descripción |
| :--- | :--- |
| **Business Problem** | Ausencia de plataformas unificadas o equipamientos de red inteligentes capaces y económicos orientados a reducir la frecuencia perjudicial en robos, siniestros y falta de socorro oportuno en la industria del ciclismo particular. |
| **Business Outcomes** | Convivencia sostenida y aumento progresivo en nuevos subscriptores a mensualidades para datos del sistema IoT (rentabilidad recurrente), con un SLA global del 98% de tiempo efectivo en reportes geolocalizados de advertencia temprana. |
| **Users** | Universitarios, trabajadores de urbes contemporáneas y atletas aficionados; principalmente un sector demográfico de 18 a 45 años, muy activos mediante smartphones y tecnología accesible. |
| **User Outcomes** | Confianza superlativa para recorrer rutas en solitario, seguridad psicológica garantizada cada vez que se deja la unidad amarrada en una calle, reducción contundente en el estrés antes de realizar largos tramos con alta dependencia a la asistencia médica. |
| **Solutions** | Un ecosistema digital holístico: módulo de hardware que equipa giroscopios, acelerómetros y GPS anidado a una aplicación nativa donde el individuo puede verificar el estado, controlar los permisos antirrobo, delimitar "GeoFences" como resguardo dinámico e ingresar sus contactos principales de auxilio (SOS). |
| **Hypotheses** | Si apostamos por construir una App que facilite las conexiones por "emparejamiento simple" y lectura veloz de códigos QR al hardware, entonces erradicaremos las barreras de entrada iniciales para el público menos orientado a la tecnología pura del hogar. |
| **What's the most important thing we need to learn first?** | Requerimos validar mediante entrevistas exploratorias de campo si la verdadera motivación de compra recae primero en el aspecto anti-ladrones del dispositivo, o si valoran casi igual de profundo las funciones paramédicas de salvataje SOS. |
| **What's the least amount of work we need to do to learn the next most important thing?** | Entablar dinámicas de descubrimiento de usuarios (Needfinding) abordando a un grupo focal pequeño para presentar los escenarios de uso teóricos, junto a encuestas simples de opción y la presentación de Mockups funcionales de la App, sin requerir todavía la construcción estricta del hardware o los microchips. |

## 1.3. Segmentos objetivo

A fines de centralizar nuestros esfuerzos en la propuesta de valor para *BiciSmartIOT*, hemos segmentado a nuestro público objetivo basándonos en las dos facetas fundamentales del dolor experimentado por nuestros prospectos:

**1. Ciclistas Urbanos (Commuters)**
*   **Perfil sociodemográfico:** Jóvenes adultos, estudiantes universitarios de pregrado y posgrado, así como profesionales de 18 a 35 años. Pertenecen usualmente a un nivel socioeconómico de clase media a media-alta. Viven alrededor del casco histórico o distritos clave, requiriendo desplazarse entre un alto tránsito durante la semana.
*   **Características e Intereses:** Adoptan la bicicleta en pos de evitar el pesado congestionamiento de tránsito en horas pico, por fuertes motivos ecologistas o por necesidad financiera al recortar gastos de servicios de movilidad. Manejan un uso fluido e intuitivo de las tecnologías portátiles, wearables y aplicativos de smartphone, por lo que esperan interfaces modernas y limpias.
*   **Necesidad principal:** Su temor primordial está radicado y enfocado en la experiencia de parqueo en infraestructuras y cicloparqueaderos públicos, semi-públicos (universidades) o privados. Requieren con suma urgencia un monitoreo pasivo eficaz para confirmar que su medio de traslado permanece inmutable e íntegro frente a terceros. Exigen ser alertados de forma síncrona ante cualquier forcejeo físico para reaccionar a la brevedad posible y evitar concretar un hurto.

**2. Ciclistas Deportivos y Aventureros (Hobbyists)**
*   **Perfil sociodemográfico:** Adultos entusiastas ubicados en el rango de 25 a 45 años. Destinan una gran proporción de liquidez adquisitiva e ingresos fijos a hobbies, mantenimiento de unidades o accesorios periféricos deportivos de muy buena gama. Tienen la flexibilidad de costear equipamiento adicional si el beneficio argumentado lo vale.
*   **Características e Intereses:** Realizan largas excursiones de trayectos interprovinciales por varios días en los fines de semana. Les interesan profundamente las métricas, la telemetría del entrenamiento y están muy acostumbrados a los aparatos sensoriales (frecuentan uso de relojes inteligentes, cámaras de acción, bandas torácicas, etc.). Tienen bicicletas con un costo altísimo de reposición.
*   **Necesidad principal:** A diferencia del primer segmento, su prioridad está subdividida. Por un lado, buscan resguardar la maquinaria costosa contra posibles asaltos violentos en carreteras solitarias (donde se hace indispensable el posicionamiento GPS continuo). Por otro lado, buscan la protección personal ante la latencia ininterrumpida de peligro de colisión contra vehículos automotores, o volcadores en calzadas irregulares, donde una alerta de caída o impacto a un contacto de auxilio marca toda la diferencia sobre el equipo de salvataje para su propia integridad.