# Especificación: recordatorios personales por Telegram

## Contexto y objetivo

La persona usuaria necesita recordar pendientes en el momento adecuado. El objetivo del MVP es permitir que una única persona programe por Telegram recordatorios puntuales o recurrentes y que el bot los envíe a las horas indicadas, sin enviar recordatorios a otras personas.

## Usuarios

- **Persona usuaria autorizada:** única persona que puede solicitar y recibir recordatorios; se identifica mediante un identificador de cuenta de Telegram definido por ella.
- **Personas no autorizadas:** cualquier otra cuenta de Telegram, que no debe poder utilizar el bot ni recibir recordatorios.

## Historias de usuario

- Como persona usuaria autorizada, quiero pedir un recordatorio en lenguaje natural para recibirlo en la fecha y hora que indique.
- Como persona usuaria autorizada, quiero programar recordatorios recurrentes para no tener que volver a solicitarlos cada vez.
- Como persona usuaria autorizada, quiero que el bot me pregunte cuando mi solicitud sea ambigua para evitar guardar un recordatorio incorrecto.
- Como persona usuaria autorizada, quiero que mis recordatorios se conserven aunque el bot se reinicie.
- Como persona usuaria autorizada, quiero que el bot reintente la entrega si falla para no perder un recordatorio.

## Requisitos funcionales

### RF-1 — Acceso exclusivo

- **Criterio de aceptación (EARS):** Cuando el bot reciba una interacción, si procede de la cuenta de Telegram autorizada, el sistema deberá permitir su uso y dirigirle los recordatorios únicamente a esa cuenta.
- **Criterio de aceptación (EARS):** Cuando el bot reciba una interacción de otra cuenta, el sistema deberá impedir que esa cuenta use la funcionalidad de recordatorios y no deberá enviarle recordatorios.

### RF-2 — Crear recordatorio puntual

- **Criterio de aceptación (EARS):** Cuando la persona autorizada solicite en lenguaje natural un recordatorio con una tarea, fecha y hora interpretables, el sistema deberá guardarlo y programar su envío para esa fecha y hora.

### RF-3 — Crear recordatorio recurrente

- **Criterio de aceptación (EARS):** Cuando la persona autorizada solicite en lenguaje natural un recordatorio recurrente con una tarea, hora y pauta de repetición interpretables, el sistema deberá guardarlo y programar sus envíos conforme a la pauta indicada.

### RF-4 — Aclarar solicitudes incompletas o ambiguas

- **Criterio de aceptación (EARS):** Si falta o es ambigua información necesaria para identificar la tarea, fecha, hora o pauta de repetición, el sistema deberá preguntar qué dato debe aclararse y no deberá guardar ni programar el recordatorio hasta obtener una respuesta suficiente.

### RF-5 — Interpretar horarios

- **Criterio de aceptación (EARS):** Cuando el sistema interprete una fecha u hora solicitada, deberá hacerlo en UTC y comunicar en español las preguntas y respuestas del bot.

### RF-6 — Conservar recordatorios

- **Criterio de aceptación (EARS):** Cuando se reinicie el bot, el sistema deberá conservar los recordatorios guardados y su configuración para continuar con los envíos pendientes.

### RF-7 — Enviar y reintentar

- **Criterio de aceptación (EARS):** Cuando llegue la hora programada de un recordatorio, el sistema deberá enviar su mensaje a la cuenta autorizada.
- **Criterio de aceptación (EARS):** Cuando falle la entrega de un recordatorio, el sistema deberá reintentar su envío y avisar a la persona autorizada si no consigue entregarlo.

## Requisitos no funcionales

- Los mensajes del bot dirigidos a la persona usuaria deberán estar en español.
- Los recordatorios guardados deberán sobrevivir a los reinicios del bot.
- La funcionalidad de recordatorios deberá poder verificarse sin depender de una conversación real con Telegram.

## Casos límite

- Solicitud sin tarea, fecha u hora suficiente: preguntar por el dato faltante y no programar todavía.
- Solicitud que admite más de una interpretación: pedir aclaración y no elegir una interpretación por cuenta propia.
- Solicitud recurrente sin una pauta de repetición clara: preguntar antes de guardarla.
- Reinicio con recordatorios pendientes: conservarlos y mantener sus horarios.
- Fallo al enviar a la hora prevista: reintentar; el límite y la frecuencia de los reintentos aún requieren definición.
- Interacción desde una cuenta no autorizada: denegar el uso y no enviarle recordatorios.

## Fuera de alcance

- Enviar recordatorios a cuentas distintas de la única cuenta autorizada.
- Permitir que otras personas utilicen el bot para gestionar sus propios recordatorios.
- Interacción basada principalmente en comandos o botones en lugar de solicitudes en lenguaje natural.

## Criterios de finalización

- Se cumplen los criterios de aceptación de todos los requisitos funcionales.
- Una solicitud puntual válida crea un recordatorio que se entrega a la cuenta autorizada en el horario indicado en UTC.
- Una solicitud recurrente válida genera envíos conforme a la pauta indicada.
- Las solicitudes ambiguas no crean recordatorios antes de ser aclaradas.
- Los recordatorios pendientes se conservan después de un reinicio.
- Los fallos de entrega provocan reintentos; si no se logra entregar, se avisa a la persona autorizada.
- Las cuentas no autorizadas no pueden utilizar la funcionalidad ni recibir recordatorios.
- Los casos de aceptación pueden verificarse sin depender de una conversación real con Telegram.

## Dudas abiertas

- [NECESITA ACLARACIÓN] ¿Cómo se define y vincula el identificador de la única cuenta autorizada?
- [NECESITA ACLARACIÓN] ¿Qué pautas recurrentes deben admitirse (por ejemplo, días de la semana, intervalos, fecha de fin)?
- [NECESITA ACLARACIÓN] ¿Qué debe ocurrir si se solicita una fecha pasada o una hora que ya pasó ese mismo día?
- [NECESITA ACLARACIÓN] ¿Cuántos reintentos deben realizarse y con qué frecuencia? ¿Cuándo se considera que no se logró entregar?
- [NECESITA ACLARACIÓN] ¿El MVP debe permitir consultar, editar, cancelar o pausar recordatorios ya guardados? ¿Cómo se solicitarán esas acciones en lenguaje natural?
- [NECESITA ACLARACIÓN] ¿El bot debe confirmar que un recordatorio se guardó y que cada envío se entregó?
- [NECESITA ACLARACIÓN] ¿El contenido del recordatorio debe conservarse exactamente como se escribió, aunque no esté en español?
