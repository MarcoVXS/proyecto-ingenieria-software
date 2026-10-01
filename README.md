# AwaLume

AwaLume es un sistema de monitoreo y control de agua compuesto por un
dispositivo AwaLume basado en ESP32, una app Flutter y un backend serverless en
AWS. El dispositivo conserva las protecciones y el control local aunque
Internet o AwaLume Cloud Service no estén disponibles.

## Documentación 

| Ruta | Contenido |
|---|---|
| `docs/vision-del-producto.md` | Visión del producto |
| `docs/guion-entrevista.md` | Entrevista dupla |
| `docs/especificacion-requisitos.md` | Especificación de requisitos |
| `docs/diagramas/casos-de-uso.png` | Diagrama de casos de uso |

# AwaLume App

Aplicación Flutter para iOS y Android. Esta base implementa el flujo de cuenta,
la lista de dispositivos AwaLume asociados al usuario y el detalle de cada uno
con tres áreas: **Estadísticas**, **Monitor** y **Ajustes**.

## Qué está implementado

- Tema claro/oscuro, fondos azulados y tarjetas neutras. Navegacion flotante y
  menus UIKit con Liquid Glass real en iOS 26 (compilado con Xcode 26+),
  alternativa compatible en iOS anterior y Material en Android.
- Registro, confirmación por correo, login, recuperación y logout mediante una
  abstracción de autenticación que usa Amazon Cognito al proporcionar la
  configuración AWS. Cuenta permite actualizar correo y cambiar la contraseña
  autenticada. Registro por teléfono permanece deshabilitado para el plan gratis.
- Lista de dispositivos AwaLume de la cuenta, selección, alta, alias editable y
  desasociación con confirmación. Eliminar uno de una cuenta no borra el
  dispositivo, sus datos ni las asociaciones de otros usuarios.
- Monitor con flujo, consumo, promedios, estado/antigüedad de la lectura,
  válvula, protecciones automáticas confirmadas y estado de comandos.
- Comandos de AwaLume Cloud Service modelados como `pending` hasta que una
  lectura confirme el resultado. Abrir queda bloqueado con estado offline,
  desconocido o antiguo; cerrar sigue disponible como acción fail-safe.
- Estadísticas con filtros Hoy, 7 días, 30 días y 12 meses, gráficas de consumo
  y flujo continuo, ejes adaptativos, lecturas intradía y agrupación mensual.
- Ajustes de Wi-Fi, Access Point, notificaciones, cuenta, información general e
  información del dispositivo. El cambio de red usa un selector generado por
  el escaneo del ESP32, con recarga y entrada manual para redes ocultas.
- Calibración guiada en un popup por muestra física: recomienda recolectar 10 L y al menos
  5 L para reducir el error, recibe el volumen observado y espera la
  confirmación del nuevo factor calculado por el dispositivo AwaLume.
- El intervalo de telemetría activa es un parámetro administrativo: no aparece
  en la app y el API de usuario rechaza intentos de modificarlo.
- Control local independiente de Cognito/API, disponible incluso desde login:
  valida exactamente el `thingName` cacheado antes de habilitar la válvula y
  usa `http://192.168.4.1` sin depender de AwaLume Cloud Service.
- Al regresar del Access Point a la app, la lista reintenta AwaLume Cloud
  Service automáticamente. Si el teléfono continúa unido al AP sin Internet, el
  error indica que debe volver al Wi-Fi de casa o a datos móviles; el directorio
  local sigue permitiendo entrar al control de contingencia.
- Repositorios demo para desarrollar la interfaz sin operar hardware real.
- Tests de navegación y contratos API/AP.
- Alta en tres pasos: Conectarse, Enviar y Guardar. Guardar autoriza localmente
  y asocia en la nube; conserva temporalmente la autorización durante un cambio
  de red. Reiniciar el acumulado diario ahora se encuentra en Ajustes.
- Notificaciones iOS nativas: solicita alertas/sonido/badge, registra el token
  APNs en SNS sin persistirlo en claro y permite preferencias independientes
  para válvula, cierres automáticos, desconexión y superación de promedios de
  consumo/flujo continuo de 7 días. iOS usa automáticamente el icono de la app.

  # Firmware AwaLume AWS MQTT/TLS

Este sketch es el firmware vigente. El firmware web original permanece
preservado como snapshot en `../legacy/ESP32_AwaLume_WebServer/`.
El historial de compilacion y las pruebas fisicas realizadas estan registrados
en `../../docs/phase-2-firmware-validation.md`. El identificador del source
vigente usa la nomenclatura `awalume-fw-0.5.0`; la imagen que ya estaba en el
prototipo antes de estos cambios se identificaba como `0.4.1-mqtt`.

Revision 2026-09-05: soporte de configuracion aislada por unidad y bloqueo de
MQTT si Thing y MAC STA fisica no coinciden. No cambia los pines, relay,
protecciones ni formato NVS. Para una segunda unidad (incluido el equipo web
instalado en casa), usar `../tools/Prepare-AwaLumeDevice.ps1`; conserva el header
del prototipo y genera el nuevo sketch dentro de `.local-secrets/devices/`.
Ver `../../docs/legacy-device-commissioning.md` para respaldo, permisos,
compilacion y puesta en marcha. La revision nueva aun requiere prueba fisica.

La validacion OTA usa ESP32 core 3.3.11, min_spiffs 4 MB con dos slots y rollback.
Descarga releases firmados desde S3 privado por orden administrativa en Shadow;
sin cambios Flutter. Requiere confianza publica y USB inicial. Ver
`../../docs/firmware-ota-operations.md`.

## Funciones incluidas

- Medicion local en GPIO 27 y control del relay en GPIO 26.
- Relay `HIGH` cierra la valvula normalmente abierta; relay `LOW` la abre.
- Persistencia compatible con el namespace NVS `awalume` del sketch anterior.
- Limites locales por litros y tiempo que funcionan aunque Wi-Fi/AWS fallen.
- MQTT mutual TLS nativo del core ESP32 3.3.10, puerto 8883 y QoS 1.
- Telemetria dirigida por cambios: inmediata al iniciar/detener flujo, al
  cambiar estado/configuracion y al generar eventos. Mientras hay flujo se
  repite cada 300 segundos por defecto; sin flujo estable no hay publicacion
  periodica de telemetria.
- `telemetryActiveIntervalSeconds` se guarda en NVS y un administrador puede
  cambiarlo directamente desde AWS en el named Shadow `config`, entre 15 y
  86400 segundos, sin recargar firmware. La API Cognito de usuario no permite
  modificarlo.
- Eventos inmediatos, `eventId` estable y cola NVS conservada hasta PUBACK.
- Auto-reconnect Wi-Fi del core con fallback explicito: cada intento recibe 30
  segundos y los reintentos posteriores usan backoff de 5 a 60 segundos.
- Auto-reconnect MQTT cada 15 segundos; al volver AWS se republica presencia,
  se vacian eventos pendientes y se resincroniza el named Shadow.
- Named Device Shadow `config`, con comandos idempotentes para valvula, reset y
  calibracion. `valveCommand` incluye `requestId` y vigencia explicita; el
  ultimo resultado con ID valido persiste en NVS y se refleja como
  `lastValveCommand`. Un comando mal formado sin ID se reporta durante la
  sesion, sin convertir una cadena vacia en un falso fallo NVS.
- `valveOpen` se conserva como contrato legacy y tambien se consume como
  comando one-shot; ninguna apertura remota vence un cierre local activo por
  consumo o tiempo.
- El firmware espera la respuesta correlacionada de AWS (`clientToken`) antes
  de confirmar cambios del Shadow, usa control optimista de `version` para
  rechazar updates obsoletos y resincroniza despues de desconexiones. Los
  mensajes `delta` disparan un GET completo; no accionan directamente el relay.
- Politica de arranque para valvula normalmente abierta: una sesion abierta se
  restaura abierta despues de un reinicio inesperado. Se conservan y restauran
  los cierres reales por limite de consumo, limite de tiempo, fallo NVS y orden
  manual. La razon heredada `unexpected_restart` de la version 0.2.0 se migra
  automaticamente a estado abierto, sin borrar NVS.
- Access Point local WPA2 siempre activo, incluso con Wi-Fi/AWS disponibles.
  Su identidad de fabrica es `AwaLume-<serial>`, donde `serial` es la MAC STA
  completa interpretada en orden canonico como decimal de 15 digitos. Permite
  setup, ajustes y control local sin depender de AWS.
- Password AP de fabrica de exactamente 8 digitos aleatorios, generado por
  unidad en el header secreto y destinado a una etiqueta/QR fisicos. El nombre
  y password activos se pueden personalizar y persistir en NVS.
- Asociacion multiusuario sin pairing code: el endpoint local crea un token
  efimero de 128 bits y solo publica su SHA-256 a AWS con TTL de 10 minutos.
- Migracion segura de un `dayKey` guardado como fecha UTC a la fecha local
  UTC-6 cuando esa firma es inequivoca. Conserva el acumulado y limita a una
  vez cada cinco minutos el aviso ante una fecha futura realmente desconocida.

# Backend AWS de AwaLume

Infraestructura CDK v2 y Lambdas para la aplicacion movil. El modo predeterminado
usa solamente servicios serverless compatibles con el nivel gratuito: Cognito,
HTTP API Gateway, Lambda, DynamoDB e IoT Core. Timestream permanece como una
extension opcional desactivada.

## Estado actual

- OTA administrativo: stack independiente `AwaLume-dev-OTA` con S3 privado,
  firmado por release/unidad desde herramientas de firmware, sin endpoints de
  app ni IoT Jobs. Ver `../docs/firmware-ota-operations.md`; requiere permisos
  complementarios `scripts/ota-publisher-policy.json` para el publicador.
- Cognito administra registro, confirmacion de email, inicio de sesion,
  recuperacion de cuenta, politica de contrasenas y MFA TOTP opcional.
- Todas las rutas de datos usan el `sub` inmutable de Cognito como `userId`.
- La API comprueba asociacion por `userId + deviceId` para cada lectura, ajuste o comando de
  un modulo. La app nunca recibe certificados IoT ni publica MQTT directamente.
- No hay pairing code ni owner unico. Cada cuenta puede asociar el mismo modulo
  con un token efimero generado por el ESP32 y atestado por su certificado IoT.
  Conocer o enumerar un serial no permite asociarlo.
- Telemetria y eventos se validan, se deduplican y se guardan con expiracion.
- Los reinicios diarios actualizan un agregado diario de forma idempotente.
- Estado online/offline se obtiene de `awalume/{thing}/status` y de los heartbeats
  `reported.lastSeenAt` del named Shadow `config`.
- API Gateway limita por defecto a 25 solicitudes/segundo y burst 50.
- Los logs estructurados nunca incluyen JWT, association tokens/hashes, certificados,
  tokens push ni cuerpos completos.

## Stacks

| Stack | Recursos principales |
|---|---|
| `Auth` | Cognito User Pool y cliente movil publico, sin client secret |
| `Data` | Registro, asociaciones multiusuario, tokens efimeros, estado, eventos, endpoints push, telemetria cruda y agregados diarios en DynamoDB |
| `IoT` | Policy por Thing, cinco Rules MQTT y Lambdas de ingestion |
| `Api` | HTTP API con authorizer Cognito, throttling y Lambda de negocio |


  
