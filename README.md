# event-driven-lab

Base para el Laboratorio 9: Implementación de Arquitecturas Basadas en Eventos.
Se continuará sobre este mismo proyecto en el Laboratorio 10 (CQRS y Saga).

## Estado actual

Copia del código versionado de `devsecops-pipeline-lab`. El Laboratorio 9 todavía
no está implementado ni desplegado. No se copiaron credenciales, dependencias
instaladas ni estados locales de Terraform.

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

Los pasos 2 a 5 están pendientes. Las evidencias deben corresponder a ejecuciones reales.
