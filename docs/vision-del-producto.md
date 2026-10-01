# Visión del producto

**Autor: Marco Villegas Xanthakis
**Fecha de la última versión: 9/30/2026
**Repositorio: proyecto-ingenieria-software
**Fecha de la última revisión por dupla: 9/30/2026

\---

## 1\. Descripción del sistema

**Nombre del sistema:** AwaLume

**Descripción:** Sistema inteligente para control de agua que muestra estadisticas y datos relevantes al usuario mediante una app. El sistema previene perdidas catastroficas de agua, como podria suceder en casos de una fuga o una llave que quedo abierta accidentalmente. Esto se logra mediante una instalacion fisica de un dispositivo en la tuberia principal, el cual cuenta con un sensor de flujo y valvula eletrica. El sistema calcula datos como el gasto diario de agua en litros, flujo constante de agua, y promedios de estos datos en distintos intervalos de tiempo. Permite establecer limites que se ejecutan automaticamente por el sistema, tal que se pueden prevenir perdidas sin una interaccion directa del usuario. 

\---

## 2\. Problema y usuarios

**El problema: No hay buenas opciones viables para** tener un control real sobre el consumo de agua.

**Cómo se resuelve hoy sin el sistema: Con sistemas muy complejos y caros que aplican mayormente a casos industriales**

**Usuarios del sistema:**

|Tipo de usuario|Qué necesita del sistema|Qué le preocupa|
|-|-|-|
|Propietario / Dueño de casa|Tener control y estadisticas relevantes sobre su consumo de agua|Perdidas catastroficas de agua|
|Arrendador / Dueño de departamentos|Tener control y estadisticas relevantes sobre el consumo de agua de sus inquilinos|Gastos excesivos de agua ocasionado por un inquilino|
||||

**Un conflicto entre usuarios:**

El producto esta pensado para instalarse antes de una cisterna, de tal manera que no afecte la presion del agua que llega a las instalaciones. La desventaja de esto es que se puede perder una cisterna de agua tras una fuga o llave abierta. El sistema solo controla el agua que entra a la cisterna, no la que sale. Algunos clientes pueden preferir sacrificar presion para tener un control mas inmediato sobre el agua. 
\--- 

## 2.1\. Huecos y dudas mencionadas por mi dupla

* La aplicación te dice cuanto vas a pagar de agua o te dice cuanto fue lo desperdiciado?
* Como se realizaría la instalación, a que nivel debo ponerlo para que funcione correctamente
* Es un solo pago o funcionaria por membresías?

## 3\. Alcance

### Dentro del alcance

* Control centralizado de varios dispositivos en distintas ubicaciones.
* AWS para control y seguridad de usuarios, bases de datos de estadisticas y dispositivos. 
* Calibracion para mejorar precision directa con cada instalacion.
* Application iOS y Android pensada para el usuario (facil de usar y entender).
* Estadisticas detalladas sobre el consumo de agua en distinos periodos de tiempo.
* Instalacion y soporte personalizado para cada cliente.
* Control auxiliar directo/local en caso de perdida de conexion con internet o caida de AWS.

### Explícitamente fuera del alcance

* Instalacion completa realizada por el cliente sin ayuda externa. 
* Decirle al usuario donde se encuentra la fuga o causa de perdida. 
* Medicion super precisa de flujo de agua.

**Por qué queda fuera:**

* Instalacion completa sin ayuda realizada por el cliente - Para instalar el dispositivo fisico se necesitan conocimientos basicos de plomeria. El setup del dispositivo con la app si podria ser realizado por el cliente, pero asumir que tiene las habilidades para manejar tuberia no es viable.
\--- 

## 4\. Tipo de sistema y restricciones

**Tipo de sistema:** Sistema híbrido embebido y crítico, complementado por una plataforma Web/SaaS de información, control y análisis de datos.

**Por qué es de ese tipo:** El núcleo de AwaLume se ejecuta en una ESP32 conectada a un sensor de flujo y a una válvula eléctrica, por lo que interactúa directamente con una instalación hidráulica y debe tomar decisiones locales aun cuando no haya Internet. Es crítico porque una lectura, un cierre o una apertura incorrectos pueden provocar desperdicio de agua, daños materiales o dejar una vivienda sin suministro. A la vez, la app móvil y los servicios de AWS administran usuarios y dispositivos, permiten el control remoto y convierten la telemetría en estado, historial, estadísticas y alertas. No es un sistema de seguridad de vida ni un medidor con precisión de facturación, pero sus funciones de protección hidráulica requieren un tratamiento más riguroso que el de una aplicación informativa común.

**Restricciones principales:**

* El dispositivo se instala en la tubería principal, antes de la cisterna, y utiliza una válvula normalmente abierta; por ello, el software no puede garantizar el cierre si se pierde totalmente la energía.
* La protección no puede depender de la app, de Internet ni de AWS. La ESP32 debe medir, aplicar límites, accionar la válvula y recuperar su estado de forma local.
* La medición sirve para monitoreo y detección de consumo anormal, no para facturación ni para localizar físicamente una fuga.
* La instalación hidráulica requiere apoyo de una persona con conocimientos de plomería, aunque la configuración y el uso cotidiano deben ser accesibles para un usuario no técnico.
* La solución debe operar con los recursos limitados de la ESP32, redes Wi-Fi de 2.4 GHz y conectividad intermitente, además de mantener compatibilidad con iOS, Android y los servicios definidos de AWS.

**Atributos de calidad que impone:**

|Atributo|Por qué importa en mi caso|Qué pasa si no se cumple|
|-|-|-|
|Seguridad funcional|AwaLume acciona una válvula real y debe dar prioridad a los límites locales de consumo y tiempo de flujo.|Una orden errónea o tardía podría permitir una pérdida de agua, cerrar el suministro sin motivo o anular una protección activa.|
|Disponibilidad y tolerancia a fallos|Las fugas y consumos anormales pueden ocurrir cuando la app, Internet o AWS no están disponibles.|El usuario perdería la protección precisamente durante una caída de red o de la nube.|
|Confiabilidad e integridad de datos|El estado de la válvula, los límites, los comandos y el historial deben sobrevivir reinicios y no duplicarse ni retroceder por mensajes atrasados.|La app mostraría información incorrecta y el dispositivo podría repetir una acción o tomar decisiones con un estado obsoleto.|
|Seguridad y privacidad|El sistema permite consultar consumos y abrir o cerrar la válvula de forma remota para dispositivos asociados a una cuenta.|Una persona no autorizada podría conocer hábitos de consumo, modificar límites o controlar la válvula.|
|Usabilidad|El alta del dispositivo, la lectura de estadísticas y la reacción ante una alerta están dirigidas a usuarios no técnicos.|Una configuración confusa puede dejar el dispositivo sin conectar o hacer que el usuario ignore una alerta importante.|

**Reglas de negocio que ya identifiqué:**

1. Las protecciones locales siempre tienen prioridad sobre la nube y la app. Si se supera el límite diario de litros, el tiempo máximo de flujo continuo o existe un fallo de almacenamiento, la ESP32 debe cerrar la válvula y puede rechazar una apertura remota.
2. Una orden remota de apertura solo se acepta si el dispositivo está en línea y su último reporte es reciente. Toda orden tiene identificador y vigencia; una apertura vencida o enviada mientras el equipo está desconectado no puede ejecutarse cuando vuelva la conexión.
3. Después de un cierre automático, la válvula permanece cerrada hasta que exista un restablecimiento manual o una orden válida que no contradiga una protección local activa.
4. Un dispositivo puede estar asociado con varias cuentas autorizadas. El alias y la desasociación pertenecen a cada cuenta; quitar el dispositivo de una cuenta no borra su historial ni las asociaciones de las demás.
5. Asociar una cuenta nueva requiere acceso físico al punto de acceso protegido del dispositivo y un token efímero de un solo uso. Las contraseñas, llaves privadas y el token en claro no deben enviarse a la nube ni quedar registrados en bitácoras.

\---

## 5\. Ciclo de vida elegido

**Modelo elegido:** Espiral, aplicado de forma incremental y ligera, con prácticas ágiles y de DevOps.

**Por qué le conviene a este proyecto:**

El alcance general está definido, pero los requisitos detallados todavía evolucionan cuando se prueba el sistema completo. En el desarrollo ya fue necesario ajustar decisiones como usar solo DynamoDB para el MVP, mantener el punto de acceso local siempre activo, permitir asociaciones multiusuario y tratar los comandos de válvula como operaciones con identificador, caducidad y confirmación. Por ello no sería realista comprometerse a que el alcance y el diseño no cambiarán.

El mayor riesgo es técnico y de integración. Comprobar que el sensor, la válvula, la memoria local, Wi-Fi, MQTT/TLS, AWS y la app funcionen juntos y mantengan las protecciones ante reinicios, pérdida de red o mensajes atrasados. El modelo Espiral permite que cada vuelta identifique el riesgo principal, construya una solución o prototipo para reducirlo, lo valide en el dispositivo y planifique la siguiente vuelta con evidencia. Así se pueden abordar primero la protección local y la recuperación de estado, y después la comunicación segura y la identidad. Luego el historial, el onboarding, los comandos remotos, las notificaciones y el endurecimiento.

El responsable del producto y los usuarios piloto pueden revisar incrementos funcionales por etapas en el teléfono y en el prototipo físico, pero no se puede asumir la presencia continua de todos los futuros dueños de vivienda, arrendadores e instaladores. El MVP académico tampoco tiene actualmente una certificación externa definida. Antes de una instalación comercial deberán identificarse las normas aplicables y conservar evidencia verificable de las pruebas de las funciones críticas.

Como el equipo es pequeño, la Espiral se aplicará sin la carga documental de un proyecto grande, como vueltas cortas, objetivos y riesgos explícitos, un incremento integrado y criterios de aceptación medibles. Las prácticas ágiles aportan retroalimentación frecuente y software funcionando, las prácticas DevOps aportan control de versiones de firmware del dispositivo ioT y software del app, pruebas automáticas, infraestructura como código, despliegues repetibles y observabilidad para los componentes que permanecen en línea.

### Alternativas descartadas

**Alternativa 1:** Ágil como modelo principal.

Los requisitos cambiantes y las entregas frecuentes sí favorecen un enfoque ágil, y se conservarán esas prácticas. Sin embargo, en AwaLume el riesgo dominante durante el MVP es demostrar la viabilidad y la seguridad técnica de la integración física, no solamente reaccionar al valor percibido por el usuario. Además, los usuarios finales e instaladores no estarán disponibles todo el tiempo. La Espiral hace explícito que cada ciclo debe reducir primero un riesgo técnico concreto.

**Alternativa 2:** Modelo V.

Es una alternativa razonable por las consecuencias de un fallo en la válvula y por su énfasis en verificación, pero presupone requisitos estables y verificables desde el inicio. AwaLume aún modifica requisitos a partir de pruebas de hardware, restricciones de la nube y retroalimentación de uso, y el MVP no tiene una obligación de certificación formal definida. Se adoptará su disciplina de relacionar requisitos críticos con pruebas y evidencia, especialmente antes de una instalación real, pero no su secuencia como ciclo de vida principal en esta etapa.
\---

