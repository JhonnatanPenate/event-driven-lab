# Outputs del despliegue del Lab 9

Resultado comunicado por el estudiante desde su terminal:

```text
Apply complete! Resources: 12 added, 0 changed, 0 destroyed.
```

Región: `us-east-1`.

```text
event_bus_name = "devsecops-lab-orders-bus"
inventory_queue_url = "https://sqs.us-east-1.amazonaws.com/916207038368/devsecops-lab-inventory-queue"
notifications_queue_url = "https://sqs.us-east-1.amazonaws.com/916207038368/devsecops-lab-notifications-queue"
```

## Evidencias pendientes

- Captura de EventBridge con las dos reglas del bus en estado `Enabled`.
- Captura de terminal con la publicación y el procesamiento por ambos consumidores para la misma orden.

Este registro conserva los outputs proporcionados; no sustituye las capturas.
Conservar la infraestructura para el Laboratorio 10.
