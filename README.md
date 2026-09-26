# event-driven-lab

Base para el Laboratorio 9: Implementación de Arquitecturas Basadas en Eventos.
Se continuará sobre este mismo proyecto en el Laboratorio 10 (CQRS y Saga).

## Estado actual

Copia del código versionado de `devsecops-pipeline-lab`. La Parte A está preparada
en `events/`, siguiendo la guía. El estudiante confirmó el despliegue de 12 recursos;
sus outputs están guardados en `docs/lab9-outputs.md`. Los tres scripts están preparados
con las URLs reales. Está pendiente verificar el flujo en AWS y reunir las capturas.

El pipeline anterior se conserva en `docs/previous-labs/deploy.yml.disabled` como
referencia. No se ejecuta automáticamente en este repositorio. El código previo
de SonarQube, OpenTelemetry e infraestructura se conserva como base histórica.

## Requisitos y entregables de la guía

- Haber completado los laboratorios 3 a 8 y disponer de Node.js, npm y acceso a AWS.
- Crear `events/main.tf`, `events/variables.tf` y `events/outputs.tf` con un bus de
  EventBridge, dos colas SQS y dos reglas independientes para `OrderCreated`.
- Gestionar los permisos nuevos mediante Terraform y esperar su propagación.
- Crear `publish-order-event.js`, `poll-notifications.js` y `poll-inventory.js`.
- Capturar la terminal con la publicación y el procesamiento por ambos
  consumidores de la misma orden.
- Capturar EventBridge con las dos reglas activas en `devsecops-lab-orders-bus`.
- Conservar el bus y las colas para el Laboratorio 10; destruirlos al terminar ese laboratorio.

La guía extiende el repositorio anterior y no exige renombrarlo ni crear otro.
Esta copia separada se preparó por decisión del estudiante. SonarQube y
OpenTelemetry no aparecen en la lista de entregables del Laboratorio 9.

## Trabajo por pasos

1. Preparar esta copia y publicarla en GitHub.
2. Crear y revisar la infraestructura de `events/`.
3. Desplegarla y registrar los outputs reales.
4. Crear el publicador y los dos consumidores.
5. Ejecutar el flujo y obtener las dos capturas solicitadas.

Los pasos 1 a 4 están preparados. La ejecución del flujo y las evidencias están pendientes.
Las evidencias deben corresponder a ejecuciones reales.

## Ejecutar el flujo en PowerShell

Desde la raíz de `event-driven-lab`, con las credenciales de AWS disponibles:

```powershell
npm install
node publish-order-event.js
Start-Sleep -Seconds 5
node poll-notifications.js
node poll-inventory.js
```

Ejecutar los comandos uno por uno y detenerse si aparece un error. Capturar la
terminal cuando ambos consumidores hayan procesado la misma orden publicada.
Si una cola no tiene mensajes, esperar y volver a ejecutar ese consumidor.
Para repetir una prueba ya consumida, publicar una orden nueva y ejecutar ambos consumidores.

Los scripts siguen la guía y simulan el correo y la reserva de inventario mediante
mensajes en consola. El publicador además comprueba `FailedEntryCount` para detectar
errores por evento devueltos por EventBridge.

## Cuándo tomar las capturas

- Ahora, en el editor: una captura de la carpeta `events/` y su código es opcional;
  no sustituye las dos capturas obligatorias.
- Después del despliegue: guardar los outputs de Terraform. Una captura de
  `Apply complete!` y los outputs es útil como respaldo, pero no es un entregable exigido.
- Después de ejecutar el publicador y ambos consumidores: tomar la captura
  obligatoria de terminal con el mismo `orderId` visible en los tres resultados.
- Después del despliegue, en AWS, región `us-east-1`: abrir EventBridge → Event buses
  → `devsecops-lab-orders-bus` → Rules y tomar la captura obligatoria de ambas reglas
  con estado `Enabled`.

No destruir el bus ni las colas al terminar el Lab 9; se reutilizan en el Lab 10.
