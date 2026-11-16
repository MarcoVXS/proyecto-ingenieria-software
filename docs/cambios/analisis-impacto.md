# Análisis de impacto de SC-001

**Sistema:** AwaLume  
**Responsable del análisis:** Marco Villegas Xanthakis
**Fecha:** 15 de noviembre de 2026  
**Estado:** análisis propuesto

## Solicitud que se analiza

SC-001: Exportar el consumo diario de una instalación a una hoja de cálculo, solicitada por la dupla de revisión el 2 de noviembre de 2026. 

Se solicita guardar un CSV por instalación y periodo (Hoy, 7 días, 30 días o 12 meses) con fechas, litros, identificación, alias personal y fecha de generación. Debe distinguir consumo cero, ausencia de registros y día actual parcial; respetar las asociaciones vigentes y comunicar los fallos sin entregar resultados incompletos como completos.

Es mantenimiento perfectivo: añade una utilidad a estadísticas existentes. La prioridad sugerida es media. 

Se consultaron la especificación, visión, entrevista, README, C4, ADR-001, y el modelo de clases. 

## Traducción a términos del sistema

La vista de estadísticas necesita una acción "Exportar consumo", conservando el dispositivo y periodo seleccionados. El módulo **Información y avisos** debe proporcionar una serie diaria autorizada, con fechas locales y unidades explícitas. **Cuentas y acceso** debe comprobar la asociación vigente y proporcionar el alias correspondiente a la cuenta solicitante.

Se propone obtener datos actualizados del servicio remoto para cada nueva exportación y generar el CSV en la aplicación. No basta con transformar los puntos visibles de una gráfica. El filtro de 12 meses podría mostrar agregados mensuales, mientras que la solicitud exige detalle diario. Debe comprobarse el contrato existente y ampliarlo si no entrega todos los días necesarios.

En el diseño de clases se proponen un servicio `ExportadorConsumoCSV` y un objeto de transferencia `ResumenConsumoDiario`, con fecha, litros opcionales, disponibilidad y condición parcial. Se reutilizan Cuenta, Asociacion y Dispositivo para las lecturas que originan los agregados, sin añadir responsabilidades de exportación al sensor o a la válvula.

El archivo utilizará columnas identificadas para dispositivo, alias, periodo, fecha de generación, fecha local, litros, estado del dato y hora de corte cuando corresponda. Se propone CSV UTF-8, escape de comillas y separadores, fechas inequívocas y números sin separadores de miles. El formato y las instrucciones de importación deben probarse en hojas de cálculo con configuración regional española.

El dato ausente se representa con litros vacíos y estado “sin datos”. Cero se reserva para consumo cero registrado. El día actual se marca parcial. Una descarga interrumpida no debe confundirse con un hueco real del historial. El archivo se ofrece únicamente tras recuperar y validar toda la respuesta correspondiente al periodo.

## Impacto directo

| Artefacto | Unidad | Qué hay que cambiar |
|---|---|---|
| Nuevo requisito RF-023, identificador propuesto | 2 | Especificar exportación diaria por instalación y periodo, incorporando los siete criterios de SC-001. Verificar que el identificador siga libre antes de incorporarlo. |
| RF-017 y RF-009 | 2 | Enlazar la exportación con las estadísticas y el acumulado diario; precisar que debe usar la misma fecha local y los mismos valores, sin cambiar cómo se mide el agua. |
| CU-08 y nuevo CU-13, identificador propuesto | 2 | Añadir acceso a “Exportar consumo” desde la consulta y documentar un caso específico: seleccionar, validar acceso, recuperar datos, generar y guardar; alternos de periodo vacío, fallo, cancelación y acceso revocado. |
| Prototipo y pantalla de estadísticas | 2 | Incorporar acción de exportación, progreso, ausencia de datos, error con reintento y salida al mecanismo de guardado del teléfono. No afirmar “guardado” si el usuario canceló. |
| Modelo de clases | 4 | Añadir ExportadorConsumoCSV y ResumenConsumoDiario; expresar sus dependencias con los datos autorizados, sin trasladar reglas de protección al exportador. Actualizar el modelo trabajado previamente, cuya publicación como archivo no se da por comprobada. |
| Diagrama de secuencia de exportación | 4 | Crear la secuencia usuario → app → consulta autorizada → datos diarios → generación → guardado, con alternativas de autorización, datos vacíos y error.  |
| App y servicio de Información y avisos | Implementación | Incorporar generación y guardado multiplataforma, y adaptar la consulta diaria si hace falta. Recuperar todas las páginas antes de producir un archivo exitoso. |
| Pruebas de aceptación | Validación | Verificar cada criterio de la solicitud y registrar el teléfono, sistema operativo y hoja de cálculo utilizados. |

## Impacto indirecto

| Artefacto | Unidad | Por qué se ve afectado |
|---|---|---|
| RNF-SEG-001; RF-003, RF-005 y RF-006 | 2 | La exportación crea otra salida de datos. Validar asociación en la nueva solicitud, evitar reutilizar datos de otra sesión y usar el alias de la cuenta correcta. Tras desasociación, impedir nuevas exportaciones; las copias ya guardadas quedan fuera del control de AwaLume. |
| RNF-CON-003 y RNF-CON-004 | 2 | Duplicados o datos atrasados también pueden contaminar el CSV. Reutilizar las reglas de integridad y probar que la exportación no altera el estado más reciente. |
| RNF-REN-002 | 2 | Más consultas podrían degradar las estadísticas. Mantener su prueba de menos de 3 segundos en al menos 95 de 100 consultas bajo la carga definida; ese umbral mide la gráfica, no el tiempo de interacción con el diálogo de guardado. Medir la exportación por separado antes de proponer un límite propio. |
| Periodos y fechas | 2 | “12 meses” no equivale siempre a 365 días. Mantener las fronteras usadas por la consulta y probar años bisiestos, cambio de mes y fecha local. El límite de 365 resúmenes del ensayo de RNF-REN-002 no debe truncar el periodo exportado. |
| Tabla de trazabilidad y diagrama de casos de uso | 2 | Relacionar RF-023 y CU-13 con Exportar consumo y sus pruebas; añadir el caso al dibujo para que no quede desactualizado. |
| Visión, README y ayuda | 1–2 | Describir la nueva capacidad y sus límites: monitoreo por instalación, sin facturación ni reparto por departamento. La entrevista histórica se conserva; la petición se registra como cambio posterior. |
| ADR-001 | 3 | **Sin impacto en la decisión:** la propuesta cabe en el monolito modular y mantiene firmware independiente. No se reemplaza el ADR ni se extrae un microservicio para esta función. |
| C4, contexto | 3 | **Sin impacto estructural:** no se incorpora un servicio externo obligatorio. Abrir el archivo fuera de AwaLume es una acción del usuario; puede ampliarse la descripción de su interacción. |
| C4, contenedores | 3 | **Sin impacto estructural:** se reutilizan app, servicio central y almacenamiento. No se proponen un servidor de exportaciones ni un repositorio permanente de archivos. |
| C4, componentes | 3 | Ampliar las responsabilidades de Información y avisos y de la consulta autorizada. El generador de CSV pertenece a la app en la alternativa elegida; no debe dibujarse como componente interno del servidor. |
| Máquinas de estados | 4 | **Sin impacto** en el ciclo de válvula, protecciones y órdenes. Para la exportación se propone documentar disponible → obteniendo → generando → listo, con error/cancelación y reintento. No se encontró un archivo independiente de estados en la documentación revisada; no se afirma haberlo actualizado. |
| Datos y almacenamiento | Diseño | No se prevé migración ni guardar CSV en la nube si los agregados diarios existentes bastan. Si faltan fechas locales, detalle diario o información de completitud, habrá que revisar el contrato y reestimar; no reconstruir datos inexistentes como si fueran mediciones. |
| Archivos y privacidad | Implementación | El historial sale de la app. Evitar incluir credenciales o datos de otras cuentas, eliminar temporales cuando ya no sean necesarios y neutralizar textos de alias que una hoja de cálculo pueda interpretar como fórmulas. |
| Pruebas existentes | Validación | Añadir regresión de consulta, alias por cuenta, desasociación y control remoto mientras se exporta; no asumir que generar un archivo solo afecta a la pantalla nueva. |

## Lo que queda sin tocar

- Medición física, calibración del sensor, límites y algoritmo de cierre automático.
- Firmware ESP32, válvula, control local sin Internet y prioridad de las protecciones.
- Estados, confirmación y caducidad de las órdenes de apertura/cierre.
- Entrega de notificaciones y preferencias de avisos; no se añade envío automático del CSV por correo o mensajería.
- Arquitectura de monolito modular, proveedores de identidad y reglas de asociación existentes.
- Datos históricos almacenados: exportar es una lectura, no una edición ni un reinicio del consumo.

## Riesgos

| Riesgo | Probabilidad | Consecuencia | Tratamiento |
|---|---|---|---|
| Confundir datos ausentes, día parcial y cero | Media | Comparaciones equivocadas del arrendador. | Probar los tres casos y fallos de descarga; etiquetar sin inventar cantidades. |
| API sin detalle diario suficiente para 12 meses | Media | La implementación excede la estimación o produce un CSV distinto a lo solicitado. | Verificar contrato y disponibilidad al inicio; ampliar consulta dentro del rango estimado o reestimar si exige reconstrucción histórica. |
| Exportación después de perder acceso o mezcla de cuentas | Media | Exposición de consumos o alias ajenos. | Consulta nueva con autorización vigente, invalidación al cambiar sesión y pruebas negativas tras desasociación. |
| CSV mal interpretado o alias tratado como fórmula | Media | Cantidades incorrectas o ejecución de fórmulas al abrirlo. | Definir formato, escapar contenido, neutralizar prefijos peligrosos y probar comas, comillas, acentos y configuración regional. |
| Guardado diferente entre iOS y Android | Media | Función aparentemente terminada que falla en una plataforma. | Pruebas reales de guardar, cancelar, falta de espacio y fallo del mecanismo de archivos. |
| Mayor carga sobre estadísticas y control remoto | Baja  | Lentitud en funciones existentes. | Limitar al periodo permitido, evitar solicitudes repetidas concurrentes y medir regresión de rendimiento. |
| Copias fuera de la app | Alta | El usuario puede compartir o conservar datos después de desasociarse. | Exportar solo lo solicitado y explicar que las copias guardadas no son revocables. |
| No realizar el cambio | Media | Continúa la transcripción manual y sus posibles errores. | Mantener la consulta actual; no hay pérdida adicional de protección hidráulica. |

## Alternativas consideradas

1. **Generar el CSV en la app con una consulta remota autorizada. (RECOMENDADA)** Reutiliza los datos diarios y las asociaciones existentes, añade un generador y el guardado nativo. Evita almacenamiento permanente de exportaciones y un servicio nuevo. Su costo principal es asegurar datos completos y comportamiento consistente en las dos plataformas. No permite nuevas exportaciones sin validar acceso remoto.
2. **Generar y devolver el CSV desde el servicio central.** Centraliza formato y validaciones, y reduce lógica de serialización en la app. Añade una operación de servidor, consumo de recursos y pruebas de respuesta/descarga; la app todavía debe guardar el archivo. Es razonable si aparecen varios clientes que requieran un formato idéntico, pero no ofrece una ventaja suficiente para esta primera versión.
3. **Conservar únicamente gráficas o capturas.** Tiene poco costo inmediato, pero no cumple el CSV editable ni evita transcribir valores. Sirve como situación provisional si se aplaza, no como entrega que satisfaga SC-001.

## Estimación de esfuerzo

| Concepto | Esfuerzo |
|---|---|
| Cambios de diseño y documentación | 6–8 horas: revisar contrato y periodos, concretar formato, requisitos, caso de uso, trazabilidad y modelos. |
| Implementación | 12–20 horas: consulta/autorización y completitud (4–8), generador y validaciones CSV (3–5), interfaz y guardado en iOS/Android (5–7). |
| Pruebas | 10–16 horas: integridad y formato (3–5), permisos y fallos (3–5), plataformas y regresión (4–6). |
| **Total** | **28–44 horas de trabajo efectivo de una persona.** |

**De dónde sale este número.** Es una estimación por descomposición de una ampliación de la consulta existente: una salida de datos, un formato de archivo y dos plataformas. No se atribuye a una experiencia previa cronometrada. Equivale aproximadamente a 7–11 jornadas de cuatro horas efectivas, sin incluir esperas por dispositivos o revisiones externas. 

**Supuestos:** existen agregados diarios accesibles para los periodos solicitados, una comprobación reutilizable de asociaciones y un proyecto móvil compilable. Se dispone de entorno de compilación de iOS, teléfonos de prueba y una forma compatible de guardar archivos. El rango incluye una adaptación acotada de la consulta, no rehacer la infraestructura ni recuperar registros perdidos.

**Fuera de la estimación:** cambios de firmware/hardware, migración o reconstrucción histórica masiva, nuevos servicios de exportación, informes programados, múltiples instalaciones en un mismo archivo, rangos arbitrarios, formatos PDF/XLSX y publicación en tiendas. No se calculan costos monetarios de nube sin mediciones de uso.

**Condición de reestimación:** al revisar contrato y pruebas iniciales, si falta detalle diario, no puede distinguirse información ausente, o el guardado exige cambios mayores del proyecto móvil, se detalla el trabajo adicional antes de comprometer la implementación. Tampoco se da por cumplido un objetivo nuevo de rendimiento de exportación sin acordarlo y medirlo.

### Comprobaciones incluidas en la estimación

| Criterio de SC-001 | Evidencia prevista |
|---|---|
| 1. Obtención del archivo | Guardar e importar CSV de los cuatro periodos en iOS y Android. |
| 2. Contenido identificable | Revisar columnas, alias por cuenta, fecha local y corte del día parcial. |
| 3. Correspondencia | Datos de 100, 150 y 200 L producen 450 L; sin duplicados ni otra instalación. |
| 4. Ausencia de registros | Distinguir hueco, cero y periodo totalmente vacío; incluir cambio de mes y año bisiesto. |
| 5. Acceso autorizado | Cuenta ajena, desasociación, cambio de sesión y dos alias para un mismo dispositivo. |
| 6. Fallos claros | Fallo en primera o última página de datos, cancelación y error de guardado; no anunciar éxito. |
| 7. Alcance | Verificar nota de monitoreo y ausencia de cobros o reparto por departamento. |

## Recomendación

**Yo, Marco Villegas, recomiendo aceptar el cambio con prioridad media**, utilizando generación del CSV en la app y una consulta remota autorizada. Resuelve una necesidad identificada del arrendador y puede aprovechar la arquitectura existente, con un esfuerzo preliminar de 28–44 horas.

Programarlo después de pendientes críticos de seguridad, confiabilidad y control. Si las comprobaciones invalidan los supuestos, actualizar la estimación y revisar la decisión antes de ampliar el alcance.
