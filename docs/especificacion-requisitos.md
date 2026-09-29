# Especificación de requisitos

**Sistema:** AwaLume

**Autor:** Marco Villegas Xanthakis

**Versión:** 0.3

**Fecha de la última actualización:** 2026-09-29

## 1. Propósito y alcance

**Propósito del documento:** Especificar los comportamientos y las condiciones de calidad que se comprobarán en AwaLume. Está dirigido al responsable del producto, al equipo de desarrollo y pruebas, a la dupla revisora y a los usuarios que participen en la validación. Contiene 22 requisitos funcionales y 10 no funcionales; describen objetivos de aceptación, no una declaración de que todos estén implementados o probados.

**Alcance del sistema:** Se retoma de la [Visión del producto](vision-del-producto.md), apartados 1, 3 y 4: monitoreo del flujo y consumo de agua; protección automática mediante límites de litros y tiempo de flujo continuo; control centralizado de varios dispositivos en distintas ubicaciones; gestión de usuarios; estadísticas por periodo; calibración por instalación; aplicación iOS y Android; control auxiliar local ante pérdida de Internet o del servicio remoto. La instalación y el soporte personalizados forman parte del servicio del producto; no se presupone un módulo de agenda, cobros o tickets. AWS, ESP32 y Flutter son decisiones de arquitectura ya documentadas y no se convierten aquí en requisitos funcionales.

**Fuera del alcance:** Instalación hidráulica completa por el cliente sin ayuda; localización física de una fuga o identificación de su causa; medición de precisión para facturación. El dispositivo mide y controla el agua que pasa por su punto de instalación antes de la cisterna: no mide por separado cada departamento conectado a una tubería común ni puede impedir que se pierda el agua ya almacenada. La modalidad de pago o membresía sigue como duda de la visión y no se resuelve mediante esta especificación.

**Condiciones de operación:** Las protecciones locales requieren alimentación eléctrica, sensor y válvula operativos. La válvula normalmente abierta puede dejar pasar agua durante un corte total de energía. Los requisitos de continuidad sin Internet no prometen continuidad sin energía. El estado de válvula que muestra la app corresponde al reportado por el dispositivo; no equivale a una medición independiente de la posición mecánica. Las pruebas físicas de cierre deben comprobar también la interrupción del paso de agua.

### Fuentes y tratamiento del origen

| Clave | Fuente | Uso y estado |
|---|---|---|
| DOC-VIS | [Visión del producto](vision-del-producto.md) | Documento del proyecto revisado el 2026-09-29; base del alcance y de las reglas de negocio. |
| DOC-GUION | [Guión de entrevista](guion-entrevista.md) | Entrevista realizada a la dupla para entender mejor al usuario, revisada el 2026-09-29. |
| DOC-DGM-CU | [Diagrama casos de uso](/diagramas/casos-de-uso.png) | Diagrama visual de los casos de uso documentados, generado el 2026-09-29. |

## 2. Usuarios y su contexto

| Usuario | Qué hace hoy sin el sistema | Qué espera del sistema |
|---|---|---|
| Propietario / dueño de casa | Se supone que revisa recibos o el medidor y cierra manualmente una llave cuando descubre una pérdida. La visión menciona alternativas industriales complejas y caras, pero no documenta una entrevista sobre su rutina. | Conocer el consumo de su vivienda y limitar pérdidas aunque no tenga el teléfono a mano. La protección es su necesidad principal. |
| Arrendador / dueño de departamentos | Se supone que compara consumos o recibos de sus inmuebles y solicita revisiones cuando detecta un gasto elevado; falta confirmar cómo lo hace y quién tiene acceso a las instalaciones. | Consultar varias instalaciones desde una cuenta y comparar sus consumos. Solo podrá distinguir departamentos si cada uno dispone de su propio punto de medición. |

**Conflictos identificados entre usuarios:**

- **Ubicación y protección:** Algunos usuarios priorizan conservar la presión instalando antes de la cisterna, mientras otros prefieren un cierre que reduzca más inmediatamente la pérdida dentro de la vivienda. Se conserva la ubicación descrita en la visión y se explicita su límite de cobertura.
- **Límites compartidos:** Un arrendador podría preferir un límite bajo y otro usuario de la misma instalación necesitar más agua.
- **Nombres personales:** Dos cuentas distintas pueden querer identificar el mismo dispositivo con nombres distintos, esto se resuelve con alias independientes por cuenta y desasociación individual.

## 3. Requisitos funcionales

Las prioridades se interpretan así: **imprescindible**, necesario para protección, acceso o uso básico; **importante**, necesario para una operación completa y comprensible; **deseable**, mejora que puede posponerse.

### 3.1 Resumen

| ID | Nombre | Prioridad | Origen |
|---|---|---|---|
| RF-001 | Registro de cuenta | Imprescindible | DOC-APP |
| RF-002 | Inicio de sesión | Imprescindible | DOC-APP, DOC-VIS |
| RF-003 | Asociación de dispositivo | Imprescindible | DOC-VIS, DOC-API |
| RF-004 | Directorio de dispositivos | Imprescindible | DOC-VIS, DOC-APP |
| RF-005 | Alias por cuenta | Importante | DOC-VIS, DOC-APP |
| RF-006 | Desasociación individual | Importante | DOC-VIS, DOC-API |
| RF-007 | Configuración de la red doméstica | Imprescindible | DOC-APP, DOC-FW |
| RF-008 | Cálculo del flujo | Imprescindible | DOC-VIS, DOC-FW |
| RF-009 | Acumulado diario | Imprescindible | DOC-VIS, DOC-API |
| RF-010 | Límite diario de consumo | Imprescindible | DOC-VIS, DOC-API |
| RF-011 | Límite de flujo continuo | Imprescindible | DOC-VIS, DOC-API |
| RF-012 | Cierre por consumo | Imprescindible | DOC-VIS, DOC-FW |
| RF-013 | Cierre por flujo continuo | Imprescindible | DOC-VIS, DOC-FW |
| RF-014 | Control remoto de válvula | Imprescindible | DOC-VIS, DOC-API |
| RF-015 | Control local de válvula | Imprescindible | DOC-VIS, DOC-FW, DOC-APP |
| RF-016 | Resultado de órdenes | Imprescindible | DOC-API, DOC-APP |
| RF-017 | Consulta de estadísticas | Importante | DOC-VIS, DOC-APP |
| RF-018 | Calibración por muestra | Importante | DOC-VIS, DOC-FW |
| RF-019 | Aviso de cambio de válvula | Importante | DOC-APP |
| RF-020 | Aviso de desconexión | Importante | DOC-APP, DOC-API |
| RF-021 | Aviso de consumo atípico | Importante | DOC-APP, SUP |
| RF-022 | Preferencias de avisos | Importante | DOC-APP |

### 3.2 Fichas

#### RF-001 · Registro de cuenta

| Campo | Contenido |
|---|---|
| Descripción | El sistema registra una cuenta con correo electrónico y contraseña, condicionada a la confirmación del correo. |
| Origen | DOC-APP, registro y confirmación por correo; derivado de documentación, no de entrevista. |
| Prioridad | Imprescindible |
| Criterio de aceptación | Con un correo no registrado y datos válidos, se crea una cuenta pendiente. Antes de confirmar el correo no se autoriza el acceso remoto; tras una confirmación válida, la cuenta puede iniciar sesión. Registrar de nuevo el mismo correo no crea una segunda cuenta. |
| Relacionado con | RF-002, RNF-SEG-001 |

#### RF-002 · Inicio de sesión

| Campo | Contenido |
|---|---|
| Descripción | El sistema autentica el acceso remoto mediante las credenciales de una cuenta confirmada. |
| Origen | DOC-APP, inicio de sesión; DOC-VIS, control y seguridad de usuarios. |
| Prioridad | Imprescindible |
| Criterio de aceptación | Una cuenta confirmada con credenciales válidas obtiene acceso a su directorio. Una contraseña incorrecta o una cuenta no confirmada no concede acceso a datos de dispositivos. |
| Relacionado con | RF-001, RF-004, RNF-SEG-001 |

#### RF-003 · Asociación de dispositivo

| Campo | Contenido |
|---|---|
| Descripción | El sistema asocia un dispositivo a una cuenta autenticada tras validar una autorización temporal obtenida mediante acceso local al dispositivo. |
| Origen | DOC-VIS, reglas 4 y 5; DOC-API, asociación sin propietario único. |
| Prioridad | Imprescindible |
| Criterio de aceptación | Una autorización válida crea una asociación visible en la cuenta. Conocer solo el identificador no basta; una autorización vencida o usada por otra cuenta se rechaza. Dos cuentas pueden asociar el mismo dispositivo con autorizaciones distintas. Repetir una solicitud ya completada por la misma cuenta no duplica la asociación. |
| Relacionado con | RF-002, RF-004, RF-006, RNF-SEG-001 |

#### RF-004 · Directorio de dispositivos

| Campo | Contenido |
|---|---|
| Descripción | El sistema muestra los dispositivos asociados a la cuenta para seleccionar la instalación que se desea consultar o controlar. |
| Origen | DOC-VIS, control centralizado de varias ubicaciones; DOC-APP, lista y selección. |
| Prioridad | Imprescindible |
| Criterio de aceptación | Una cuenta asociada a dos dispositivos muestra ambos, cada uno con su identificador y alias. Seleccionar uno abre sus datos y dirige las acciones a ese dispositivo. Una cuenta sin asociaciones muestra una lista vacía. |
| Relacionado con | RF-003, RF-005, RF-014, RF-017, RNF-SEG-001 |

#### RF-005 · Alias por cuenta

| Campo | Contenido |
|---|---|
| Descripción | El sistema guarda un alias de dispositivo independiente para cada cuenta asociada. |
| Origen | DOC-VIS, regla 4; DOC-APP, alias editable. |
| Prioridad | Importante |
| Criterio de aceptación | Dos cuentas asignan alias distintos al mismo dispositivo. Cambiar el alias en la primera modifica únicamente su directorio y el nombre se conserva al volver a iniciar sesión. |
| Relacionado con | RF-003, RF-004, RNF-SEG-001 |

#### RF-006 · Desasociación individual

| Campo | Contenido |
|---|---|
| Descripción | El sistema elimina la asociación del dispositivo con la cuenta que confirma su desasociación. |
| Origen | DOC-VIS, regla 4; DOC-API, eliminación de la relación usuario-dispositivo. |
| Prioridad | Importante |
| Criterio de aceptación | Tras confirmar, el dispositivo desaparece del directorio de esa cuenta y su acceso remoto se rechaza. El historial del dispositivo y el acceso de otra cuenta asociada permanecen. Cancelar la confirmación conserva la asociación. |
| Relacionado con | RF-003, RF-004, RNF-SEG-001 |

#### RF-007 · Configuración de la red doméstica

| Campo | Contenido |
|---|---|
| Descripción | El sistema configura la conexión del dispositivo a una red doméstica a partir de los datos proporcionados por un usuario con acceso local autorizado. |
| Origen | DOC-APP, alta y ajustes de Wi-Fi; DOC-FW, configuración local. |
| Prioridad | Imprescindible |
| Criterio de aceptación | Con datos correctos de una red compatible y disponible, el dispositivo logra conectarse. Con una contraseña incorrecta no informa éxito y conserva el acceso local para corregirla. Los datos sensibles no aparecen en bitácoras. |
| Relacionado con | RF-003, RF-015, RNF-SEG-002, RNF-USA-001 |

#### RF-008 · Cálculo del flujo

| Campo | Contenido |
|---|---|
| Descripción | El sistema calcula el flujo de agua en litros por minuto a partir de las lecturas del sensor y la calibración vigente. |
| Origen | DOC-VIS, descripción del sistema; DOC-FW, lectura del sensor. |
| Prioridad | Imprescindible |
| Criterio de aceptación | En una prueba con calibración de 450 pulsos por litro, 450 pulsos uniformes durante 60 segundos representan 1 L/min, con la tolerancia de redondeo de la presentación. Sin pulsos durante un intervalo completo de medición, el flujo calculado es cero. La prueba verifica la conversión, no una precisión física certificada. |
| Relacionado con | RF-009, RF-013, RF-018, RNF-CON-001 |

#### RF-009 · Acumulado diario

| Campo | Contenido |
|---|---|
| Descripción | El sistema acumula en litros el volumen medido por dispositivo durante cada día de la instalación. |
| Origen | DOC-VIS, consumo diario; DOC-API, agregados diarios y cambio de día. |
| Prioridad | Imprescindible |
| Criterio de aceptación | Con acumulado inicial cero y calibración de 450 pulsos/L, 4,500 pulsos suman 10 L. Al cambiar la fecha local, el acumulado del día nuevo empieza en cero y el total anterior queda consultable. El reinicio manual del contador, si se utiliza, no elimina el volumen ya contabilizado del día en el historial. |
| Relacionado con | RF-008, RF-012, RF-017, RNF-CON-002, RNF-CON-003 |

#### RF-010 · Límite diario de consumo

| Campo | Contenido |
|---|---|
| Descripción | El sistema configura por dispositivo el límite diario de consumo en litros, donde cero desactiva exclusivamente esta protección. |
| Origen | DOC-VIS, límites de consumo; DOC-API, configuración y valor cero. |
| Prioridad | Imprescindible |
| Criterio de aceptación | Al solicitar 1,000 L, el valor se presenta como pendiente hasta que el dispositivo confirme su aplicación. Tras confirmarlo, RF-012 utiliza ese límite. Un valor negativo se rechaza sin cambiar el límite anterior; cero no desactiva la protección de tiempo. |
| Relacionado con | RF-012, RF-016, RNF-CON-002 |

#### RF-011 · Límite de flujo continuo

| Campo | Contenido |
|---|---|
| Descripción | El sistema configura por dispositivo el tiempo máximo de flujo continuo en minutos, donde cero desactiva exclusivamente esta protección. |
| Origen | DOC-VIS, tiempo máximo de flujo; DOC-API, configuración y valor cero. |
| Prioridad | Imprescindible |
| Criterio de aceptación | Al solicitar 30 minutos, el valor queda pendiente hasta su confirmación por el dispositivo. Tras confirmarlo, RF-013 utiliza ese tiempo. Un valor negativo se rechaza sin cambiar el anterior; cero no desactiva la protección de litros. |
| Relacionado con | RF-013, RF-016, RNF-CON-002 |

#### RF-012 · Cierre por consumo

| Campo | Contenido |
|---|---|
| Descripción | El sistema cierra localmente la válvula cuando el acumulado diario supera el límite de consumo habilitado. |
| Origen | DOC-VIS, regla 1; DOC-FW, protección local. |
| Prioridad | Imprescindible |
| Criterio de aceptación | Con límite de 10 L y sin otra causa de cierre, 9 L no provoca cierre por consumo. Al superar 10 L, se acciona el cierre; en banco hidráulico se comprueba que cesa el paso de agua. El mismo resultado se obtiene sin Internet. |
| Relacionado con | RF-009, RF-010, RF-019, RNF-REN-001, RNF-CON-001, RNF-CON-005 |

#### RF-013 · Cierre por flujo continuo

| Campo | Contenido |
|---|---|
| Descripción | El sistema cierra localmente la válvula cuando la duración de un episodio de flujo continuo supera el límite de tiempo habilitado. |
| Origen | DOC-VIS, regla 1; DOC-FW, protección por tiempo. |
| Prioridad | Imprescindible |
| Criterio de aceptación | Con límite de 2 minutos y sin otra causa de cierre, un episodio de 1 minuto no cierra por tiempo y uno que supera 2 minutos sí. Un intervalo completo de medición sin flujo termina el episodio: el siguiente comienza con duración cero. Se verifica el cierre con agua y se repite sin Internet. |
| Relacionado con | RF-008, RF-011, RF-019, RNF-REN-001, RNF-CON-001, RNF-CON-005 |

#### RF-014 · Control remoto de válvula

| Campo | Contenido |
|---|---|
| Descripción | El sistema procesa órdenes remotas para establecer el estado abierto o cerrado de la válvula del dispositivo seleccionado. |
| Origen | DOC-VIS, reglas 1 y 2; DOC-API, comandos remotos. |
| Prioridad | Imprescindible |
| Criterio de aceptación | Una cuenta asociada solicita el cierre y el dispositivo conectado lo ejecuta. La apertura solo se admite con estado en línea, último reporte de antigüedad máxima de 7 minutos y ausencia de bloqueo local. Una apertura solicitada sin esas condiciones se rechaza y no queda pendiente para la reconexión. Cada orden remota de esta línea base vence a los 120 segundos de su emisión. |
| Relacionado con | RF-004, RF-016, RNF-SEG-001, RNF-CON-003, RNF-CON-005 |

#### RF-015 · Control local de válvula

| Campo | Contenido |
|---|---|
| Descripción | El sistema procesa órdenes de válvula mediante acceso local autorizado al dispositivo, incluso cuando el servicio remoto está inaccesible. |
| Origen | DOC-VIS, control auxiliar; DOC-FW y DOC-APP, acceso local independiente de la sesión remota. |
| Prioridad | Imprescindible |
| Criterio de aceptación | Sin Internet ni sesión remota disponible, el usuario conectado a la red local protegida del dispositivo puede cerrar la válvula. Puede abrirla solo si no hay un bloqueo local activo. Si la identidad del equipo conectado no coincide con la seleccionada, no se habilita el control. |
| Relacionado con | RF-016, RNF-SEG-001, RNF-CON-001, RNF-CON-005 |

#### RF-016 · Resultado de órdenes

| Campo | Contenido |
|---|---|
| Descripción | El sistema muestra el resultado de cada orden de válvula o cambio de límite según la confirmación del dispositivo destinatario. |
| Origen | DOC-APP, estados pendientes; DOC-API, confirmación del dispositivo. |
| Prioridad | Imprescindible |
| Criterio de aceptación | Que el servicio reciba una solicitud no basta para mostrarla como aplicada. Antes de la confirmación se muestra pendiente; después se muestra aplicada o rechazada según el resultado. Una orden vencida o sin confirmación se identifica como tal, sin atribuirle éxito. Una confirmación de otro dispositivo u otra orden no confirma la actual. |
| Relacionado con | RF-010, RF-011, RF-014, RF-015, RNF-CON-004 |

#### RF-017 · Consulta de estadísticas

| Campo | Contenido |
|---|---|
| Descripción | El sistema muestra las estadísticas de consumo y duración máxima de flujo continuo del dispositivo para el periodo seleccionado. |
| Origen | DOC-VIS, estadísticas por periodos; DOC-APP, filtros Hoy, 7 días, 30 días y 12 meses. |
| Prioridad | Importante |
| Criterio de aceptación | Con un historial conocido, cada filtro muestra solo los datos de su periodo y del dispositivo elegido, con litros y unidades de tiempo visibles. Las sumas coinciden con los registros de prueba. Si no hay registros, se indica ausencia de datos; no se presenta como consumo cero medido. |
| Relacionado con | RF-004, RF-008, RF-009, RNF-REN-002, RNF-SEG-001, RNF-CON-003, RNF-CON-004 |

#### RF-018 · Calibración por muestra

| Campo | Contenido |
|---|---|
| Descripción | El sistema ajusta la calibración del sensor mediante una sesión de medición y el volumen real de una muestra proporcionado por el usuario. |
| Origen | DOC-VIS, calibración por instalación; DOC-FW, procedimiento por muestra física. |
| Prioridad | Importante |
| Criterio de aceptación | En una sesión iniciada sin flujo, se hace pasar una muestra, se detiene el flujo y se registra su volumen. Con 4,500 pulsos y 10 L, el dispositivo confirma un factor de 450 pulsos/L. Un volumen cero, una sesión sin pulsos o una sesión interrumpida por reinicio se rechaza sin sustituir la calibración anterior. |
| Relacionado con | RF-008, RF-009, RNF-CON-002, RNF-CON-005 |

#### RF-019 · Aviso de cambio de válvula

| Campo | Contenido |
|---|---|
| Descripción | El sistema notifica los cambios de estado de la válvula a las cuentas asociadas que tengan habilitada la categoría correspondiente. |
| Origen | DOC-APP, avisos de válvula y cierres automáticos. |
| Prioridad | Importante |
| Criterio de aceptación | Con permisos del teléfono, entrega de avisos disponible y categoría habilitada, un cambio reportado genera un aviso que identifica el dispositivo y el nuevo estado; un cierre automático incluye su motivo. Repetir el mismo evento no genera un segundo aviso. Sin conexión remota, el cierre local no espera la entrega del aviso. |
| Relacionado con | RF-012, RF-013, RF-014, RF-015, RF-022, RNF-CON-003 |

#### RF-020 · Aviso de desconexión

| Campo | Contenido |
|---|---|
| Descripción | El sistema notifica la pérdida de conexión remota del dispositivo a las cuentas asociadas que tengan habilitado ese aviso. |
| Origen | DOC-APP, notificación de desconexión; DOC-API, presencia y antigüedad del último reporte. |
| Prioridad | Importante |
| Criterio de aceptación | Con entrega de avisos disponible, se genera un aviso cuando el servicio recibe una desconexión o deja de considerar reciente el último reporte conforme al límite de 7 minutos de RF-014. Se emite un aviso por episodio; una desconexión posterior a una reconexión confirmada permite uno nuevo. No se exige entrega al teléfono mientras este carezca de conexión. |
| Relacionado con | RF-014, RF-022, RNF-CON-003 |

#### RF-021 · Aviso de consumo atípico

| Campo | Contenido |
|---|---|
| Descripción | El sistema notifica cuando el consumo diario del dispositivo supera su promedio de los siete días completos anteriores. |
| Origen | DOC-APP, avisos de superación de promedios; SUP: exigir siete días completos como base de comparación para esta versión. |
| Prioridad | Importante |
| Criterio de aceptación | Con siete días completos de 100 L cada uno, un consumo actual de 101 L genera un aviso si la categoría está habilitada; 100 L no lo genera. Se emite como máximo uno por dispositivo, cuenta y día. Si faltan días de referencia, esta regla no emite aviso ni inventa valores para completar la media. |
| Relacionado con | RF-009, RF-017, RF-022, RNF-CON-003 |

#### RF-022 · Preferencias de avisos

| Campo | Contenido |
|---|---|
| Descripción | El sistema guarda por cuenta las preferencias de recepción de avisos de válvula, cierres automáticos, desconexión y consumo atípico. |
| Origen | DOC-APP, preferencias independientes por usuario. |
| Prioridad | Importante |
| Criterio de aceptación | Desactivar una categoría evita los avisos posteriores de esa categoría para la cuenta y el cambio persiste al volver a entrar. Otra cuenta asociada conserva sus preferencias. Desactivar avisos no desactiva los límites ni las protecciones del dispositivo. |
| Relacionado con | RF-019, RF-020, RF-021, RNF-SEG-001 |

## 4. Requisitos no funcionales

### 4.1 Resumen

| ID | Atributo | Nombre | Prioridad | Origen |
|---|---|---|---|---|
| RNF-REN-001 | Rendimiento | Respuesta del control protector | Imprescindible | DOC-VIS; SUP: 2 segundos |
| RNF-REN-002 | Rendimiento | Tiempo de consulta de estadísticas | Importante | DOC-VIS; SUP: 3 segundos y carga de prueba |
| RNF-SEG-001 | Seguridad | Aislamiento de acceso | Imprescindible | DOC-VIS, DOC-API, DOC-APP |
| RNF-SEG-002 | Seguridad | Ausencia de secretos en bitácoras | Imprescindible | DOC-VIS, DOC-API, DOC-FW |
| RNF-USA-001 | Usabilidad | Alta sin ayuda del equipo | Importante | DOC-VIS; SUP: muestra y meta de uso |
| RNF-CON-001 | Confiabilidad | Autonomía ante caída de red | Imprescindible | DOC-VIS, DOC-FW; SUP: duración del ensayo |
| RNF-CON-002 | Confiabilidad | Recuperación de configuración confirmada | Imprescindible | DOC-FW; SUP: cantidad de reinicios |
| RNF-CON-003 | Confiabilidad | Efectos únicos ante duplicados | Imprescindible | DOC-API, DOC-FW; SUP: repeticiones del ensayo |
| RNF-CON-004 | Confiabilidad | Consistencia ante reportes atrasados | Imprescindible | DOC-API |
| RNF-CON-005 | Confiabilidad | Prioridad de la protección local | Imprescindible | DOC-VIS, DOC-FW, DOC-API |

La seguridad funcional de DOC-VIS se concreta en RNF-CON-005 y RNF-REN-001; la disponibilidad y tolerancia a fallos, en RNF-CON-001; la confiabilidad e integridad, en RNF-CON-002 a RNF-CON-004; la seguridad y privacidad, en RNF-SEG-001 y RNF-SEG-002; y la usabilidad, en RNF-USA-001. Se usan las claves de atributos de la guía. No se añaden metas de escalabilidad o mantenibilidad sin una necesidad sustentada en el alcance.

### 4.2 Fichas

#### Rendimiento

##### RNF-REN-001 · Respuesta del control protector

| Campo | Contenido |
|---|---|
| Atributo de calidad | Rendimiento |
| Descripción | El dispositivo emite la acción de cierre en un máximo de 2 segundos desde que sus mediciones detectan la superación de un límite habilitado. |
| Métrica | Tiempo desde la detección del exceso hasta la señal de accionamiento de cierre, como máximo 2 s en cada uno de 20 ensayos por protección, con alimentación estable, tanto con Internet como sin él. No incluye el recorrido mecánico de la válvula, cuyo cierre físico se comprueba en RF-012 y RF-013. |
| Origen | DOC-VIS, sistema embebido y crítico; SUP: umbral de 2 s y número de ensayos propuestos, pendientes de validación en banco. |
| Prioridad | Imprescindible |
| Por qué importa | Retrasar la orden permite seguir acumulando pérdidas. La meta separa la respuesta del controlador del tiempo mecánico para poder medir ambas sin confundirlas. |
| Afecta a | RF-012, RF-013 |

##### RNF-REN-002 · Tiempo de consulta de estadísticas

| Campo | Contenido |
|---|---|
| Atributo de calidad | Rendimiento |
| Descripción | La app presenta una vista de estadísticas de hasta 365 resúmenes diarios en menos de 3 segundos en al menos el 95 % de las consultas bajo la carga definida. |
| Métrica | Desde seleccionar el periodo hasta visualizar la gráfica completa: menos de 3 s en al menos 95 de 100 consultas, con 10 cuentas consultando simultáneamente, conexión estable de al menos 10 Mbps y latencia de red de ida y vuelta no mayor a 100 ms. Se registra el teléfono y la versión utilizados. |
| Origen | DOC-VIS, sistema de información y análisis; SUP: tiempo, carga y condiciones de ensayo propuestos, no mediciones ya obtenidas. |
| Prioridad | Importante |
| Por qué importa | La consulta debe servir para comparar instalaciones sin esperas que dificulten el uso cotidiano. La carga acotada corresponde a una meta inicial por validar. |
| Afecta a | RF-017 |

#### Seguridad

##### RNF-SEG-001 · Aislamiento de acceso

| Campo | Contenido |
|---|---|
| Atributo de calidad | Seguridad |
| Descripción | El sistema rechaza el 100 % de los intentos de acceso a datos o acciones sin la autorización correspondiente al canal utilizado. |
| Métrica | En remoto, probar todas las operaciones de consulta y modificación con sesión ausente, inválida y de una cuenta no asociada: cero lecturas o cambios autorizados. Probar además que una cuenta no modifica alias, asociaciones o preferencias de otra. En local, sin acceso a la red protegida o con identidad de equipo distinta de la seleccionada: cero órdenes de válvula ejecutadas. La ausencia de sesión remota por sí sola no bloquea el acceso local autorizado. |
| Origen | DOC-VIS, seguridad y cuentas compartidas; DOC-API, autorización por asociación; DOC-APP, acceso local independiente. |
| Prioridad | Imprescindible |
| Por qué importa | El consumo revela hábitos de la vivienda y una orden no autorizada afecta el suministro. La condición local evita que una caída del servicio de identidad elimine el control de contingencia previsto. |
| Afecta a | RF-001, RF-002, RF-003, RF-004, RF-005, RF-006, RF-010, RF-011, RF-014, RF-015, RF-017, RF-018, RF-022 |

##### RNF-SEG-002 · Ausencia de secretos en bitácoras

| Campo | Contenido |
|---|---|
| Atributo de calidad | Seguridad |
| Descripción | Las bitácoras de la aplicación, del dispositivo y del servicio remoto contienen cero contraseñas, llaves privadas o credenciales temporales de acceso en claro. |
| Métrica | Ejecutar registro, acceso, asociación y configuración de red con credenciales de prueba identificables, incluyendo errores y reintentos. Buscar esas credenciales completas en las bitácoras de los tres componentes: cero coincidencias. |
| Origen | DOC-VIS, regla 5; DOC-API y DOC-FW, exclusión de secretos de bitácoras. |
| Prioridad | Imprescindible |
| Por qué importa | Los registros se utilizan para diagnóstico y no deben convertirse en un medio para entrar a cuentas o controlar instalaciones. |
| Afecta a | RF-001, RF-002, RF-003, RF-007 |

#### Usabilidad

##### RNF-USA-001 · Alta sin ayuda del equipo

| Campo | Contenido |
|---|---|
| Atributo de calidad | Usabilidad |
| Descripción | Al menos 8 de 10 usuarios nuevos completan la asociación y configuran la red del dispositivo en un máximo de 10 minutos sin asistencia verbal del equipo. |
| Métrica | Prueba con 10 personas sin experiencia previa en AwaLume, una cuenta ya confirmada, dispositivo instalado y alimentado, credenciales disponibles y red operativa. Medir desde abrir el alta hasta que el dispositivo aparece conectado en su cuenta; pueden usar únicamente las instrucciones de la app y la etiqueta del producto. Registrar teléfono y sistema operativo, incluyendo participantes con iOS y Android. |
| Origen | DOC-VIS, aplicación para usuarios no técnicos; SUP: tamaño de muestra, proporción y tiempo propuestos para validación. |
| Prioridad | Importante |
| Por qué importa | La instalación hidráulica requiere ayuda, pero el alta cotidiana debe poder completarse con las instrucciones del producto. La prueba no promete instalación de tubería sin un especialista. |
| Afecta a | RF-003, RF-007 |

#### Confiabilidad

##### RNF-CON-001 · Autonomía ante caída de red

| Campo | Contenido |
|---|---|
| Atributo de calidad | Confiabilidad |
| Descripción | El dispositivo mantiene la medición y el control protector durante la indisponibilidad de Internet y del servicio remoto mientras conserve alimentación. |
| Métrica | Durante un ensayo de 60 minutos sin conexión remota, comprobar acumulación con un volumen de prueba y provocar por separado ambos límites: cero cierres omitidos por falta de red. Comprobar también el control local autorizado. El ensayo de 60 minutos no establece una caducidad del funcionamiento autónomo. |
| Origen | DOC-VIS y DOC-FW, independencia de la nube; SUP: duración de 60 minutos para el ensayo inicial. |
| Prioridad | Imprescindible |
| Por qué importa | La pérdida de conexión no debe desactivar la protección de la instalación. Los avisos e históricos remotos pueden quedar indisponibles durante la caída. |
| Afecta a | RF-008, RF-009, RF-012, RF-013, RF-015 |

##### RNF-CON-002 · Recuperación de configuración confirmada

| Campo | Contenido |
|---|---|
| Atributo de calidad | Confiabilidad |
| Descripción | El dispositivo recupera el 100 % de los límites, la calibración y la orden de válvula cuya persistencia se había confirmado antes de un reinicio. |
| Métrica | Configurar valores conocidos, esperar confirmación y reiniciar 20 veces: cero pérdidas de límites o calibración. Tras cada arranque, un cierre persistido permanece ordenado; una sesión previamente abierta se restaura abierta si no hay bloqueo local. Se evalúa el estado posterior al arranque, no el suministro durante el corte eléctrico. |
| Origen | DOC-FW, persistencia y política de arranque; SUP: 20 reinicios como muestra inicial. |
| Prioridad | Imprescindible |
| Por qué importa | Volver a valores arbitrarios puede eliminar la protección. Esta garantía no se extiende al consumo ocurrido desde la última persistencia hasta el corte, cuya posible pérdida parcial reconoce DOC-FW. |
| Afecta a | RF-009, RF-010, RF-011, RF-014, RF-015, RF-018 |

##### RNF-CON-003 · Efectos únicos ante duplicados

| Campo | Contenido |
|---|---|
| Atributo de calidad | Confiabilidad |
| Descripción | El sistema produce como máximo un efecto por identificador de orden, muestra o evento aunque reciba duplicados. |
| Métrica | Reenviar 10 veces cada orden, muestra de consumo y evento de aviso con el mismo identificador: una ejecución como máximo, un registro de consumo y un aviso como máximo por destinatario elegible. Repetir una orden de válvula después de reiniciar no vuelve a ejecutarla si su resultado ya se confirmó. |
| Origen | DOC-API y DOC-FW, operaciones idempotentes y deduplicación; SUP: 10 repeticiones por ensayo. |
| Prioridad | Imprescindible |
| Por qué importa | Los reintentos de comunicación no deben inflar estadísticas, repetir movimientos de válvula o saturar de avisos a los usuarios. |
| Afecta a | RF-009, RF-014, RF-016, RF-017, RF-019, RF-020, RF-021 |

##### RNF-CON-004 · Consistencia ante reportes atrasados

| Campo | Contenido |
|---|---|
| Atributo de calidad | Confiabilidad |
| Descripción | El sistema conserva el estado más reciente de un dispositivo al recibir reportes anteriores a ese estado. |
| Métrica | Ingresar dos reportes con orden temporal conocido en ambas secuencias posibles: el estado actual final coincide con el más reciente en el 100 % de los casos de prueba. Una solicitud aún no confirmada no sustituye el estado reportado ni convierte por sí sola un equipo desconectado en conectado. |
| Origen | DOC-API, orden de ingesta y distinción entre solicitado y confirmado. |
| Prioridad | Imprescindible |
| Por qué importa | Presentar una lectura antigua como actual puede hacer que el usuario crea que una acción se ejecutó o que el dispositivo volvió a conectarse. |
| Afecta a | RF-004, RF-016, RF-017, RF-020 |

##### RNF-CON-005 · Prioridad de la protección local

| Campo | Contenido |
|---|---|
| Atributo de calidad | Confiabilidad, aplicada a seguridad funcional |
| Descripción | El dispositivo ejecuta cero aperturas que contradigan una protección local activa o una orden remota vencida. |
| Métrica | Probar apertura local y remota con exceso de litros, exceso de tiempo y fallo de almacenamiento, y entregar una orden remota después de sus 120 s de vigencia: cero aperturas. Repetir tras una reconexión y un reinicio. La apertura vuelve a ser admisible solo después de resolver el bloqueo y recibir una orden válida; la calibración no suspende esta condición. |
| Origen | DOC-VIS, reglas 1 a 3; DOC-FW y DOC-API, precedencia local y vigencia de comandos. |
| Prioridad | Imprescindible |
| Por qué importa | Una reconexión o un comando atrasado no debe reabrir el paso de agua contra una protección vigente. El criterio supone alimentación y actuador funcionales, como se delimita en el apartado 1. |
| Afecta a | RF-012, RF-013, RF-014, RF-015, RF-018 |

## 5. Casos de uso

La frontera considerada es AwaLume completo: dispositivo, aplicación y servicio remoto. El sensor y los componentes internos no se representan como actores externos. En CU-05 y CU-12 el usuario es el destinatario de un servicio iniciado por una condición o un evento; el cierre automático no requiere que intervenga. Los requisitos no funcionales indicados condicionan cada interacción y no son funciones adicionales.

Se conserva la numeración existente: CU-01 integra el acceso como parte de consultar las instalaciones; CU-03 se concreta en el alias y CU-10 en las preferencias. La desasociación y la recepción de avisos pasan a CU-11 y CU-12, respectivamente. Las rutas de la sección 6 permiten revisar los elementos del prototipo y no certifican que las pruebas ya se hayan satisfecho.

### CU-01 · Consultar las instalaciones asociadas

| Campo | Contenido |
|---|---|
| Actor principal | Propietario o arrendador. |
| Objetivo | Identificar las instalaciones a las que tiene acceso desde su cuenta. |
| Precondición | La aplicación está instalada y el servicio remoto está disponible. |
| Escenario principal | 1. El usuario solicita consultar sus instalaciones.<br>2. El sistema solicita las credenciales si no existe una sesión válida.<br>3. El usuario introduce sus credenciales.<br>4. El sistema autentica la cuenta confirmada y recupera sus asociaciones.<br>5. El sistema muestra las instalaciones con sus identificadores y alias para que el usuario las identifique. |
| Flujos alternos | 2a. Ya existe una sesión válida: el sistema continúa en el paso 4 sin solicitar credenciales.<br>3a. El usuario no tiene cuenta: solicita el registro, proporciona correo y contraseña, confirma el correo y vuelve al paso 3. Si el correo ya está registrado o la confirmación es inválida, el sistema informa el problema y no concede acceso.<br>4a. Las credenciales son incorrectas o la cuenta no está confirmada: el sistema rechaza el acceso; el usuario puede corregir la situación y volver al paso 3.<br>5a. La cuenta no tiene asociaciones: el sistema muestra una lista vacía y la opción de iniciar CU-02; no muestra dispositivos ajenos.<br>5b. Falla la consulta: el sistema informa el error y permite reintentar, sin presentar datos inventados. |
| Postcondición | El usuario dispone del directorio correspondiente a su cuenta o conoce que no tiene instalaciones asociadas. Si falla la autenticación o la consulta, no se concede acceso indebido ni se informa éxito. |
| Requisitos que realiza | RF-001, RF-002, RF-004, RNF-SEG-001, RNF-SEG-002, RNF-CON-004. |

### CU-02 · Dar de alta una instalación

| Campo | Contenido |
|---|---|
| Actor principal | Propietario o arrendador con acceso físico al dispositivo; el instalador puede apoyarlo. |
| Objetivo | Incorporar una instalación a su cuenta para consultarla y controlarla. |
| Precondición | El dispositivo está instalado, alimentado y accesible mediante su red local protegida. El usuario tiene una cuenta confirmada y los datos de la red doméstica. |
| Escenario principal | 1. El usuario se conecta a la red local protegida del dispositivo e inicia el alta.<br>2. El sistema solicita los datos de la red doméstica.<br>3. El usuario proporciona esos datos.<br>4. El sistema configura la conexión y comunica su resultado.<br>5. El usuario solicita asociar el dispositivo a su cuenta.<br>6. El sistema obtiene la autorización temporal local y, con acceso al servicio remoto y sesión válida, valida la solicitud de asociación.<br>7. El sistema incorpora el dispositivo y lo muestra en el directorio del usuario. |
| Flujos alternos | 4a. La conexión doméstica falla: el sistema informa el problema y conserva el acceso local; el usuario puede corregir los datos en el paso 3.<br>6a. La autorización es inválida, venció o fue utilizada por otra cuenta: el sistema rechaza la asociación y el usuario debe obtener una nueva mediante acceso local.<br>6b. No hay acceso al servicio remoto: la asociación no se presenta como completada; el usuario restablece la comunicación y reintenta con una autorización vigente.<br>7a. La misma solicitud ya fue completada para esa cuenta: el sistema devuelve la asociación existente sin duplicarla. |
| Postcondición | La instalación queda conectada y asociada a la cuenta, sin eliminar asociaciones de otros usuarios. Si la autorización falla, no se crea una nueva asociación. |
| Requisitos que realiza | RF-003, RF-004, RF-007, RNF-SEG-001, RNF-SEG-002, RNF-USA-001, RNF-CON-004. |

### CU-03 · Asignar un nombre personal a una instalación

| Campo | Contenido |
|---|---|
| Actor principal | Propietario o arrendador autenticado. |
| Objetivo | Reconocer una instalación mediante un alias propio en su directorio. |
| Precondición | El dispositivo está asociado a la cuenta y el servicio remoto está disponible. |
| Escenario principal | 1. El usuario selecciona la instalación en su directorio.<br>2. El sistema muestra el alias actual de esa cuenta.<br>3. El usuario introduce un nuevo alias y solicita guardarlo.<br>4. El sistema comprueba la asociación y guarda el alias para esa cuenta.<br>5. El sistema muestra el directorio con el nuevo nombre. |
| Flujos alternos | 3a. El usuario cancela la edición: el sistema conserva el alias anterior y el caso termina.<br>4a. La cuenta ya no está asociada: el sistema rechaza la modificación y no cambia el alias.<br>4b. No se confirma el guardado: el sistema informa el fallo y no presenta el nuevo alias como guardado; el usuario puede reintentar. |
| Postcondición | El alias queda actualizado para la cuenta solicitante y permanece al volver a iniciar sesión. Los alias de otras cuentas no cambian. |
| Requisitos que realiza | RF-004, RF-005, RNF-SEG-001, RNF-CON-004. |

### CU-04 · Establecer un límite de protección

| Campo | Contenido |
|---|---|
| Actor principal | Propietario o arrendador asociado al dispositivo. |
| Objetivo | Dejar aplicado el límite de consumo o de tiempo de flujo continuo elegido para una instalación. |
| Precondición | El usuario tiene una sesión válida y acceso al servicio remoto; la instalación está asociada a su cuenta. |
| Escenario principal | 1. El usuario selecciona la instalación y el límite que desea ajustar.<br>2. El sistema muestra el valor confirmado y su unidad: litros o minutos.<br>3. El usuario introduce el nuevo valor y solicita aplicarlo.<br>4. El sistema valida el valor y envía la solicitud al dispositivo, mostrándola como pendiente.<br>5. El dispositivo guarda el valor y confirma el cambio al sistema.<br>6. El sistema muestra el límite como aplicado. |
| Flujos alternos | 4a. El valor es negativo: el sistema lo rechaza y conserva el anterior; el usuario puede volver al paso 3.<br>4b. El valor es cero: el sistema tramita la desactivación de esa protección, sin modificar la otra, y continúa en el paso 5.<br>5a. El dispositivo rechaza el cambio: el sistema comunica el rechazo y no sustituye el valor confirmado.<br>5b. No llega confirmación del dispositivo: el sistema identifica el cambio como no confirmado; no lo presenta como aplicado. |
| Postcondición | El límite elegido queda confirmado en el dispositivo. Ante rechazo o falta de confirmación, el usuario conoce que el cambio no está confirmado. |
| Requisitos que realiza | RF-004, RF-010, RF-011, RF-016, RNF-SEG-001, RNF-CON-002, RNF-CON-004. |

### CU-05 · Limitar una pérdida de agua automáticamente

| Campo | Contenido |
|---|---|
| Actor principal | Propietario o arrendador que dejó habilitada la protección; recibe el servicio sin intervenir durante el cierre. |
| Objetivo | Detener el paso de agua por el dispositivo cuando se supere un límite habilitado, aunque el usuario no esté presente. |
| Precondición | El dispositivo tiene alimentación, sensor y válvula operativos y al menos un límite habilitado mediante CU-04. La superación de un límite desencadena este caso. |
| Escenario principal | 1. El sistema calcula el flujo y el acumulado a partir de las lecturas de la instalación.<br>2. El sistema detecta que se superó un límite habilitado.<br>3. El sistema acciona localmente el cierre de la válvula.<br>4. El sistema reporta el estado y el motivo del cierre cuando la comunicación está disponible.<br>5. El usuario recibe el aviso de cierre si tiene habilitada la categoría y la entrega está disponible. |
| Flujos alternos | 2a. No se supera ningún límite: el sistema continúa la medición del paso 1 sin cerrar por esa causa.<br>3a. No hay Internet o el servicio remoto está caído: el cierre se ejecuta localmente; la comunicación del paso 4 no se presenta como disponible.<br>3b. Se pierde totalmente la energía: deja de cumplirse la precondición y no se garantiza el cierre físico de la válvula normalmente abierta; aplica la limitación del apartado 1.<br>5a. La categoría está deshabilitada o no puede entregarse el aviso: el cierre conserva su efecto y no espera a la notificación. |
| Postcondición | Con las condiciones físicas de operación satisfechas, la válvula queda cerrada por el límite superado. La ausencia de aviso no implica ausencia de cierre. El agua ya almacenada en la cisterna queda fuera de este control. |
| Requisitos que realiza | RF-008, RF-009, RF-012, RF-013, RF-019, RNF-REN-001, RNF-CON-001, RNF-CON-002, RNF-CON-003, RNF-CON-005. |

### CU-06 · Cambiar el estado de la válvula a distancia

| Campo | Contenido |
|---|---|
| Actor principal | Propietario o arrendador asociado y autenticado. |
| Objetivo | Dejar la válvula de una instalación en el estado solicitado desde una ubicación remota. |
| Precondición | El usuario tiene acceso al servicio remoto y la instalación está asociada a su cuenta. |
| Escenario principal | 1. El usuario selecciona la instalación y solicita abrir o cerrar la válvula.<br>2. El sistema comprueba la autorización y, para una apertura, el estado en línea y la antigüedad máxima de 7 minutos del último reporte.<br>3. El sistema envía la orden con vigencia de 120 segundos y la muestra como pendiente.<br>4. El dispositivo comprueba la vigencia y las protecciones locales, ejecuta la orden válida y reporta su resultado.<br>5. El sistema muestra el resultado correspondiente a esa orden y, si hay cambio y avisos habilitados, notifica el nuevo estado. |
| Flujos alternos | 2a. La cuenta no está asociada: el sistema rechaza la solicitud y el caso termina.<br>2b. Se solicita abrir con estado desconectado, desconocido o antiguo: el sistema rechaza la apertura y no la deja pendiente para la reconexión.<br>4a. Una protección local impide abrir: el dispositivo rechaza la apertura y el sistema muestra el motivo.<br>4b. La orden vence antes de ejecutarse: no se ejecuta; el sistema la identifica como vencida cuando conoce el resultado.<br>4c. La válvula ya está en el estado solicitado: el dispositivo confirma que no se requiere cambio y el sistema muestra ese resultado.<br>5a. No llega confirmación: el sistema indica falta de confirmación sin atribuir éxito a la orden. |
| Postcondición | El dispositivo confirma el estado solicitado o el usuario recibe un resultado de rechazo, vencimiento o falta de confirmación. Una orden rechazada o vencida no se ejecuta al reconectar. |
| Requisitos que realiza | RF-004, RF-014, RF-016, RF-019, RNF-SEG-001, RNF-CON-002, RNF-CON-003, RNF-CON-004, RNF-CON-005. |

### CU-07 · Cambiar el estado de la válvula mediante acceso local

| Campo | Contenido |
|---|---|
| Actor principal | Propietario o arrendador con credenciales de acceso local al dispositivo. |
| Objetivo | Controlar la válvula estando junto a la instalación, incluso durante una caída del servicio remoto. |
| Precondición | El dispositivo está alimentado, el teléfono está al alcance de su red protegida y la identidad de la instalación es conocida por la app. No se requiere una sesión remota vigente. |
| Escenario principal | 1. El usuario conecta el teléfono a la red local protegida y selecciona la instalación conocida.<br>2. El sistema compara la identidad del equipo conectado con la instalación seleccionada.<br>3. El usuario solicita abrir o cerrar la válvula.<br>4. El dispositivo evalúa sus protecciones y ejecuta la orden local admisible.<br>5. El sistema muestra el resultado confirmado por el dispositivo. |
| Flujos alternos | 1a. No se obtiene acceso a la red protegida: el sistema no habilita órdenes locales y el usuario debe resolver el acceso.<br>2a. La identidad no coincide: el sistema bloquea el control de ese equipo y el caso termina.<br>4a. Existe una protección activa que impide abrir: el dispositivo rechaza la apertura y el sistema comunica el motivo.<br>5a. La conexión local se pierde antes de recibir el resultado: el sistema informa falta de confirmación; al reconectar, el usuario consulta el estado antes de decidir otra acción. |
| Postcondición | La orden local queda confirmada o se informa la causa de no aplicación o de falta de confirmación, sin depender de la autenticación remota. |
| Requisitos que realiza | RF-015, RF-016, RNF-SEG-001, RNF-CON-001, RNF-CON-002, RNF-CON-004, RNF-CON-005. |

### CU-08 · Consultar el consumo histórico de una instalación

| Campo | Contenido |
|---|---|
| Actor principal | Propietario o arrendador autenticado. |
| Objetivo | Conocer el consumo y la duración máxima de flujo continuo de una instalación en un periodo elegido. |
| Precondición | El dispositivo está asociado a la cuenta y el servicio remoto está disponible. |
| Escenario principal | 1. El usuario selecciona la instalación que desea consultar.<br>2. El sistema muestra sus estadísticas disponibles.<br>3. El usuario elige Hoy, 7 días, 30 días o 12 meses.<br>4. El sistema consulta los registros de esa instalación y del periodo solicitado.<br>5. El sistema presenta las gráficas y los valores con sus unidades para que el usuario consulte el historial. |
| Flujos alternos | 4a. La cuenta no tiene acceso al dispositivo: el sistema rechaza la consulta sin mostrar datos.<br>4b. El periodo no contiene registros: el sistema indica ausencia de datos y no los sustituye por consumo cero.<br>4c. Falla la comunicación: el sistema muestra el error y permite reintentar la consulta.<br>5a. Llegan datos repetidos o atrasados: el sistema evita duplicar el consumo y no sustituye el estado actual por uno anterior. |
| Postcondición | El usuario dispone de las estadísticas del dispositivo y periodo solicitados o de una indicación explícita de ausencia de datos o fallo. |
| Requisitos que realiza | RF-004, RF-017, RNF-REN-002, RNF-SEG-001, RNF-CON-003, RNF-CON-004. |

### CU-09 · Calibrar el sensor con una muestra real

| Campo | Contenido |
|---|---|
| Actor principal | Propietario o arrendador asociado, con apoyo del instalador si lo necesita. |
| Objetivo | Ajustar la medición del sensor al volumen real de una muestra en la instalación. |
| Precondición | El dispositivo está conectado, la válvula está abierta sin bloqueos activos, el flujo está detenido y hay un recipiente graduado disponible. |
| Escenario principal | 1. El usuario solicita iniciar una sesión de calibración.<br>2. El sistema inicia el registro de la muestra y confirma que la sesión está activa.<br>3. El usuario hace pasar agua por el dispositivo, mide el volumen y detiene el flujo.<br>4. El usuario introduce el volumen observado y solicita terminar la calibración.<br>5. El sistema valida la muestra, calcula el ajuste y lo guarda en el dispositivo.<br>6. El sistema comunica la confirmación de la nueva calibración. |
| Flujos alternos | 2a. No puede iniciarse la sesión: el sistema informa el fallo y no presenta la calibración como activa.<br>3a. Se activa una protección: el dispositivo mantiene su prioridad de cierre; la calibración no autoriza una apertura contra el bloqueo.<br>5a. El volumen es inválido, no hay pulsos o el factor resultante está fuera del rango aceptado: el sistema rechaza el ajuste y conserva la calibración anterior.<br>5b. Un reinicio interrumpió la sesión: el sistema la rechaza y el usuario debe iniciar una nueva desde el paso 1.<br>6a. No llega confirmación: el sistema no presenta el ajuste como aplicado. |
| Postcondición | La nueva calibración queda confirmada y persistida, o se conserva la anterior si la sesión fue rechazada. La falta de confirmación se comunica sin afirmar éxito. |
| Requisitos que realiza | RF-018, RNF-SEG-001, RNF-CON-002, RNF-CON-005. |

### CU-10 · Elegir los avisos que se desean recibir

| Campo | Contenido |
|---|---|
| Actor principal | Propietario o arrendador autenticado. |
| Objetivo | Dejar guardadas las categorías de avisos que desea recibir en su cuenta. |
| Precondición | El usuario tiene acceso al servicio remoto con una sesión válida. |
| Escenario principal | 1. El usuario solicita consultar sus preferencias de avisos.<br>2. El sistema muestra las categorías y sus valores actuales.<br>3. El usuario activa o desactiva las categorías de válvula, cierre automático, desconexión y consumo atípico.<br>4. El sistema guarda las preferencias para esa cuenta.<br>5. El sistema muestra los valores guardados. |
| Flujos alternos | 3a. El usuario cancela la modificación: conserva las preferencias anteriores.<br>4a. Falla el guardado: el sistema informa el error y no presenta el cambio como confirmado; el usuario puede reintentar.<br>4b. Se intenta modificar preferencias de otra cuenta: el sistema rechaza la solicitud. |
| Postcondición | Las preferencias quedan guardadas para esa cuenta y se conservan al volver a entrar. Las preferencias de otras cuentas y las protecciones del dispositivo no cambian. |
| Requisitos que realiza | RF-022, RNF-SEG-001. |

### CU-11 · Retirar una instalación del directorio personal

| Campo | Contenido |
|---|---|
| Actor principal | Propietario o arrendador autenticado. |
| Objetivo | Dejar de tener una instalación asociada a su cuenta sin afectar a los demás usuarios. |
| Precondición | El dispositivo está asociado a la cuenta y el servicio remoto está disponible. |
| Escenario principal | 1. El usuario selecciona la instalación que desea retirar.<br>2. El sistema muestra la instalación y solicita confirmar su desasociación.<br>3. El usuario confirma.<br>4. El sistema elimina exclusivamente la asociación de esa cuenta con el dispositivo.<br>5. El sistema actualiza el directorio y deja de autorizar el acceso remoto de esa cuenta a la instalación. |
| Flujos alternos | 3a. El usuario cancela: el sistema conserva la asociación y el caso termina.<br>4a. La solicitud no está autorizada: el sistema la rechaza sin modificar asociaciones.<br>4b. No se confirma la eliminación: el sistema informa el fallo y el usuario puede volver a consultar el directorio antes de reintentar. |
| Postcondición | El dispositivo deja de estar asociado a la cuenta solicitante. Su historial, registro y asociaciones de otras cuentas permanecen. |
| Requisitos que realiza | RF-004, RF-006, RNF-SEG-001, RNF-CON-004. |

### CU-12 · Recibir un aviso sobre una instalación

| Campo | Contenido |
|---|---|
| Actor principal | Propietario o arrendador asociado al dispositivo, destinatario del aviso. |
| Objetivo | Enterarse de un cambio de válvula, una desconexión o un consumo atípico en una instalación identificada. |
| Precondición | La cuenta está asociada al dispositivo y tiene habilitada la categoría pertinente; para la entrega, el teléfono tiene permisos y comunicación disponibles. Un evento reportado o una condición detectada inicia el caso. |
| Escenario principal | 1. El sistema detecta un cambio de válvula, una desconexión o la superación del promedio de consumo definido en RF-021.<br>2. El sistema identifica las cuentas asociadas elegibles y comprueba sus preferencias vigentes.<br>3. El sistema verifica que el aviso no duplique otro ya emitido para el mismo evento o episodio.<br>4. El sistema envía el aviso con la instalación y la situación identificadas; en un cierre automático incluye el motivo.<br>5. El usuario recibe el aviso y conoce qué ocurrió en la instalación. |
| Flujos alternos | 1a. Faltan siete días completos para la comparación de consumo: el sistema no emite el aviso de RF-021 ni inventa valores de referencia.<br>2a. La categoría fue deshabilitada o la cuenta se desasoció antes del envío: no se envía ese aviso a la cuenta.<br>3a. El aviso duplica el evento, episodio de desconexión o alerta diaria de consumo: se omite el envío repetido.<br>5a. El teléfono carece de conexión o permisos: no se garantiza la entrega; la protección local continúa sin esperar al aviso. |
| Postcondición | El usuario recibe un aviso identificable cuando se satisfacen las condiciones de entrega. No se generan avisos repetidos ni se modifican las protecciones por la ausencia de recepción. |
| Requisitos que realiza | RF-019, RF-020, RF-021, RNF-CON-003, RNF-CON-004. |

## 6. Trazabilidad

| Requisito | Origen | Caso de uso | Elemento del prototipo |
|---|---|---|---|
| RF-001 | DOC-APP | CU-01 | P-01 |
| RF-002 | DOC-APP, DOC-VIS | CU-01 | P-01, P-09 |
| RF-003 | DOC-VIS, DOC-API | CU-02 | P-03, P-08, P-09 |
| RF-004 | DOC-VIS, DOC-APP | CU-01, CU-02, CU-03, CU-04, CU-06, CU-08, CU-11 | P-02 |
| RF-005 | DOC-VIS, DOC-APP | CU-03 | P-02 |
| RF-006 | DOC-VIS, DOC-API | CU-11 | P-02, P-09 |
| RF-007 | DOC-APP, DOC-FW | CU-02 | P-03, P-05, P-08 |
| RF-008 | DOC-VIS, DOC-FW | CU-05 | P-04, P-08 |
| RF-009 | DOC-VIS, DOC-API | CU-05 | P-04, P-07, P-08 |
| RF-010 | DOC-VIS, DOC-API | CU-04 | P-04, P-08, P-09 |
| RF-011 | DOC-VIS, DOC-API | CU-04 | P-04, P-08, P-09 |
| RF-012 | DOC-VIS, DOC-FW | CU-05 | P-08, banco hidráulico |
| RF-013 | DOC-VIS, DOC-FW | CU-05 | P-08, banco hidráulico |
| RF-014 | DOC-VIS, DOC-API | CU-06 | P-04, P-08, P-09 |
| RF-015 | DOC-VIS, DOC-FW, DOC-APP | CU-07 | P-06, P-08 |
| RF-016 | DOC-API, DOC-APP | CU-04, CU-06, CU-07 | P-04, P-06, P-09 |
| RF-017 | DOC-VIS, DOC-APP | CU-08 | P-07 |
| RF-018 | DOC-VIS, DOC-FW | CU-09 | P-05, P-08, recipiente graduado |
| RF-019 | DOC-APP | CU-05, CU-06, CU-12 | P-10, aviso en el teléfono |
| RF-020 | DOC-APP, DOC-API | CU-12 | P-10, aviso en el teléfono |
| RF-021 | DOC-APP, SUP | CU-12 | P-10, aviso en el teléfono |
| RF-022 | DOC-APP | CU-10 | P-05, P-10 |
| RNF-REN-001 | DOC-VIS, SUP | CU-05 | P-08, medición en banco |
| RNF-REN-002 | DOC-VIS, SUP | CU-08 | P-07, medición de tiempos |
| RNF-SEG-001 | DOC-VIS, DOC-API, DOC-APP | CU-01, CU-02, CU-03, CU-04, CU-06, CU-07, CU-08, CU-09, CU-10, CU-11 | P-01 a P-07, P-09, pruebas de acceso |
| RNF-SEG-002 | DOC-VIS, DOC-API, DOC-FW | CU-01, CU-02 | P-01, P-03, P-08, P-09, revisión de bitácoras |
| RNF-USA-001 | DOC-VIS, SUP | CU-02 | P-03, prueba con usuarios |
| RNF-CON-001 | DOC-VIS, DOC-FW, SUP | CU-05, CU-07 | P-06, P-08, ensayo sin red |
| RNF-CON-002 | DOC-FW, SUP | CU-04, CU-05, CU-06, CU-07, CU-09 | P-08, ensayo de reinicios |
| RNF-CON-003 | DOC-API, DOC-FW, SUP | CU-05, CU-06, CU-08, CU-12 | P-08, P-09, P-10, inyección de duplicados |
| RNF-CON-004 | DOC-API | CU-01, CU-02, CU-03, CU-04, CU-06, CU-07, CU-08, CU-11, CU-12 | P-02, P-04, P-07, P-09, reportes fuera de orden |
| RNF-CON-005 | DOC-VIS, DOC-FW, DOC-API | CU-05, CU-06, CU-07, CU-09 | P-08, banco hidráulico |

## 7. Registro de cambios

| Fecha | Requisito | Qué cambió | Por qué |
|---|---|---|---|
| 2026-09-17 | RF-001 a RF-022; 10 RNF del apartado 4 | Versión 0.1: catálogo inicial, criterios de aceptación, métricas, 10 casos de uso y trazabilidad. Se mantienen las instrucciones y ejemplos originales separados del catálogo. | Documentar AwaLume con la estructura de la plantilla y las reglas de las guías, según la solicitud del autor. |
| 2026-09-22 | CU-01 a CU-12 y trazabilidad | Versión 0.2: se adopta el formato de siete campos del PDF, se numeran los escenarios y sus alternativas, se concretan los objetivos y se separan la desasociación (CU-11) y la recepción de avisos (CU-12). Se actualizan las relaciones con RF y RNF; se conservan sus fichas y los ejemplos de la plantilla. | Ajustar los casos de uso al material de elicitación del curso solicitado por el autor. |

