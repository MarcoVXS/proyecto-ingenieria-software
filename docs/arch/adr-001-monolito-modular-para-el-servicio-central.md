# ADR-001: Usar un monolito modular para el servicio central de AwaLume

## Estado

Aceptada para el diseño académico, 29 de octubre de 2026. Registra la elección de la Actividad 2 del 20 de octubre de 2026; no declara terminada una refactorización de la implementación.

## Contexto

AwaLume es desarrollado por una persona, Marco Villegas Xanthakis. Combina una app móvil, un servicio central y un dispositivo que mide agua y acciona una válvula. En la Actividad 2 se identificaron cuatro módulos: Cuentas y acceso, Medición, Control y protección, e Información y avisos. Sus fronteras todavía pueden ajustarse durante la integración.

El [README](../../README.md) describe Flutter, ESP32 y servicios administrados de AWS, con varias funciones Lambda y stacks. La elección arquitectónica concierne a la organización y entrega del núcleo de negocio central, no a convertir la app, el firmware y la base de datos en un único proceso ni a afirmar que el despliegue actual ya está consolidado.

La [especificación](../especificacion-requisitos.md) exige autorización por asociación (RNF-SEG-001), protección sin conexión remota (RNF-CON-001) y prioridad de los bloqueos locales sobre órdenes de apertura (RNF-CON-005). No se dispone de una medición que justifique escalar cada módulo central por separado. Las 10 cuentas simultáneas de RNF-REN-002 son una carga de prueba, no una cantidad comprobada de clientes.

Se consideraron un monolito sin fronteras internas explícitas, que facilita atajos y mezcla de reglas, y microservicios, que permitirían despliegues independientes a costa de más contratos de red, observabilidad y coordinación para una sola persona.

## Decisión

Organizamos el servicio central como un monolito modular: los cuatro módulos se versionan y entregan juntos, colaboran por interfaces internas y cada uno accede exclusivamente a sus datos mediante repositorios propios. Mantenemos la app y el firmware como unidades independientes, con las protecciones y el control local ejecutándose en el dispositivo sin depender del servicio central.

## Consecuencias

- **Beneficio:** una entrega coordinada del núcleo reduce el trabajo operativo y permite ajustar fronteras sin introducir contratos de red entre los cuatro módulos. Los servicios administrados de AWS siguen siendo infraestructura, no microservicios propios por módulo.
- **Beneficio:** las interfaces permiten probar reglas por separado y conservar el Adapter de notificaciones; la autonomía del firmware evita que una caída de la nube desactive las protecciones.
- **Costo:** los módulos centrales no se despliegan ni escalan de forma independiente; una falla o un consumo excesivo de recursos puede afectar a otros módulos del servicio remoto.
- **Costo y riesgo:** separar carpetas no basta. Hay que impedir accesos a datos ajenos, mantener contratos y pruebas, y revisar dependencias en cada cambio. Alinear las funciones existentes con esta unidad de entrega puede requerir reorganización posterior; este ADR no la ejecuta.
- **Condición para revisar:** el primer candidato a extracción es Información y avisos. Se reconsiderará si incumple sostenidamente RNF-REN-002 —menos de 3 segundos en al menos el 95 % de consultas, bajo las condiciones de prueba especificadas— y las mediciones atribuyen el problema a competencia por recursos con Control. Primero se optimizan consultas y procesamiento; solo se separa si requiere escalar independientemente.
- **Registro futuro:** si cambia la decisión, este archivo se conserva y su estado se marca como reemplazado por el nuevo ADR. Los [tres diagramas](diagramas.md) muestran las fronteras de esta decisión.
