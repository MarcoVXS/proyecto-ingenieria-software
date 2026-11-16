# SC-001: Exportar el consumo diario de una instalación a una hoja de cálculo

## Estado

Propuesta - 2 de noviembre de 2026.

## Quién la pide

Dupla de revisión, considerando las necesidades del arrendador descrito en la entrevista. 

## Qué se pide

“Quiero poder guardar el consumo diario de uno de mis edificios durante el periodo que estoy consultando en un archivo que pueda abrir en mi hoja de cálculo. Necesito que incluya las fechas, los litros registrados y la identificación de la instalación para compararlo con mis recibos y notas de ocupación sin copiar los valores a mano. Cuando falten datos, quiero poder distinguirlos de un día en el que realmente no se consumió agua.”

## Por qué

En la ficha de dominio y las respuestas de la entrevista, el arrendador administra dos edificios, registra gastos en una hoja de cálculo y compara recibos con notas de ocupación. Una exportación le permitiría incorporar las mediciones de AwaLume a ese trabajo, conservar una copia y comparar periodos sin transcribir manualmente los valores de las gráficas.

La documentación vigente contempla consultar estadísticas, pero no establece una función de exportación para el usuario. Por ello, se propone una mejora del uso de la información existente, no la corrección de un incumplimiento demostrado.

## Qué pasa si no se hace

El arrendador podrá seguir consultando las estadísticas en la aplicación, pero tendrá que transcribir los consumos o guardar capturas para incorporarlos a sus registros. Esto exige trabajo adicional y puede introducir errores al copiar fechas, cantidades o identificar el edificio correspondiente.

No se pierde la capacidad actual de medición, control o protección automática. La consecuencia es principalmente de comodidad y aprovechamiento de los datos, por lo que la mejora puede programarse después de las funciones críticas.

## Tipo de mantenimiento

- [ ] **Correctivo** · algo está mal y hay que repararlo.
- [ ] **Adaptativo** · el entorno cambió y hay que ajustarse.
- [x] **Perfectivo** · funciona, pero puede ser mejor.
- [ ] **Preventivo** · todavía no falla, pero va a fallar.

**Justificación** 
amplía la utilidad de las estadísticas para una tarea documentada del usuario, sin responder a un defecto demostrado ni a un cambio impuesto por un proveedor o por el entorno.

## Criterio de aceptación

El cambio se considerará satisfactorio cuando se cumplan todos estos resultados observables:

1. **Obtención del archivo:** una cuenta autenticada y asociada a una instalación puede guardar un archivo CSV del periodo seleccionado (Hoy, 7 días, 30 días o 12 meses) e importarlo en una hoja de cálculo con fechas y cantidades reconocibles en columnas separadas. Se comprueba desde las versiones de iOS y Android admitidas por AwaLume.
2. **Contenido identificable:** el archivo identifica el dispositivo y su alias para la cuenta solicitante, el periodo cubierto y la fecha y hora de generación. Incluye la fecha local de cada día y su consumo en litros; las unidades quedan explícitas. El día actual se identifica como parcial, con su hora de corte.
3. **Correspondencia con los datos:** con registros de prueba de 100, 150 y 200 litros en tres días completos, se obtienen esas tres cantidades y su suma es 450 litros. El archivo contiene únicamente fechas del periodo y datos de la instalación seleccionada, sin duplicar días.
4. **Ausencia de registros:** si dentro del periodo falta un día, se identifica como “sin datos” y su consumo queda vacío; un día con consumo cero medido aparece como 0. Si todo el periodo carece de registros, se informa que no hay datos para exportar y no se presenta un archivo como una medición exitosa.
5. **Acceso autorizado:** una cuenta sin asociación vigente no puede obtener una nueva exportación de esa instalación, aunque conozca su identificador. Dos cuentas asociadas ven sus respectivos alias, sin recibir datos personales de la otra. La desasociación no pretende borrar copias que el usuario haya guardado previamente.
6. **Resultado claro ante fallos:** si la obtención de los datos falla, se informa el problema y se permite reintentar; no se presenta como completo un archivo con datos recuperados solo parcialmente por ese fallo.
7. **Alcance de la medición:** el archivo indica que los valores corresponden al punto de instalación y sirven para monitoreo, no para facturación. No reparte automáticamente el consumo entre departamentos ni calcula importes de cobro.

CSV se especifica como formato de entrega que el usuario puede abrir, sin prescribir bibliotecas, almacenamiento ni componentes internos.

## Prioridad que sugiere el solicitante

**Media.** Resuelve una necesidad concreta del arrendador y evita trabajo manual repetitivo. Sin embargo, la consulta actual permite continuar trabajando y la exportación no es necesaria para cerrar la válvula ni para mantener las protecciones locales; no debe desplazar correcciones de seguridad, confiabilidad o control.

---

*Esta solicitud describe la necesidad y los resultados esperados. La solución técnica, la estimación de esfuerzo y la recomendación de aceptar, rechazar o aplazar corresponden al análisis de impacto posterior.*
