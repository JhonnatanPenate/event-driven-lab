# event-driven-lab

Proyecto de los laboratorios 9 a 15 de Desarrollo en la Nube II: arquitecturas
basadas en eventos y contenedores avanzados.

| Lab | Tema | Carpetas / archivos principales |
|-----|------|---------------------------------|
| 9 | EventBridge + colas SQS con consumidores independientes | `events/`, `publish-order-event.js`, `poll-*.js` |
| 10 | CQRS (modelo de lectura DynamoDB) y Saga con compensación | `events/read-model.tf`, `update-read-model.js`, `query-orders.js`, `saga-orchestrator.js` |
| 11 | API Gateway + Lambda + SQS (integración empresarial) | `api-gateway/main.tf`, `lambda/` |
| 12 | Fan-out con SNS hacia colas billing y shipping | `api-gateway/fanout.tf`, `sns-fanout-consumer.js` |
| 13 | Kubernetes local con kind: Kafka, Deployments, Services, Job y Service Discovery | `k8s/`, `k8s-app/` |
| 14 | Imagen multi-stage, probes, réplicas, Ingress (ingress-nginx) y escalado | `status-api/`, `k8s/status-api-*.yaml` |
| 15 | Auto-escalado (HPA), self-healing, rolling update y PodDisruptionBudget | `k8s/status-api-hpa.yaml`, `k8s/status-api-pdb.yaml` |

## Estado actual

Copia del código versionado de `devsecops-pipeline-lab`. La Parte A está preparada
en `events/`, siguiendo la guía. El estudiante confirmó el despliegue de 12 recursos;
sus outputs están guardados en `docs/lab9-outputs.md`. Los tres scripts están preparados
con las URLs reales. El estudiante confirmó el funcionamiento del flujo del Lab 9
y la preparación de sus evidencias.

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

El estudiante confirmó los pasos 1 a 5 del Lab 9.
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

## Laboratorio 10 — Partes A y B

`events/read-model.tf` agrega la tabla `devsecops-lab-orders-read-model`, el permiso
de DynamoDB y la espera de propagación. El estudiante confirmó el despliegue y
aportó una captura de la tabla activa con clave de partición `orderId` de tipo String.

`update-read-model.js` consume la cola de inventario, guarda cada orden en DynamoDB
con estado `received` y después elimina el mensaje de la cola.
`query-orders.js` consulta la tabla siguiendo el ejemplo de la guía.

Desde la raíz del repositorio, ejecutar uno por uno:

```powershell
node publish-order-event.js
Start-Sleep -Seconds 5
node update-read-model.js
node query-orders.js
```

Para esta prueba no ejecutar `poll-inventory.js`: compite por la misma cola y
podría consumir el mensaje antes que el modelo de lectura. Usar una orden nueva.
Si no llegan mensajes, esperar y repetir `update-read-model.js`. Si la escritura
terminó pero la consulta aún no muestra la orden, repetir `query-orders.js` tras
unos segundos.

Captura obligatoria: publicación, escritura y consulta con el mismo `orderId`
visible. Las capturas del código son respaldo adicional al repositorio.
La evidencia de CQRS está pendiente de revisión. La limpieza final (Parte D) está pendiente.

## Laboratorio 10 — Parte C: Saga

`saga-orchestrator.js` sigue la secuencia de la guía: reservar inventario, cobrar
y confirmar la orden. Si el pago falla después de reservar, libera la reserva.
Los servicios están simulados mediante mensajes de consola; esta demo no cobra
dinero ni modifica inventario real, y no utiliza las colas ni DynamoDB.

`run-saga-demo.js` reutiliza `tracing.js` y ejecuta cinco intentos con resultados
aleatorios. Los spans de OpenTelemetry se imprimen en consola al exportarse.
Se usa `finally` para cerrar el span y el SDK incluso ante errores inesperados.

```powershell
node run-saga-demo.js
```

Captura obligatoria: una ejecución que muestre al menos `SAGA COMPLETA` y
`SAGA COMPENSADA`. Para mostrar la compensación claramente, incluir también
el rechazo del pago y `[Compensación] Liberando reserva ...`. Si falta alguno
de los caminos, ejecutar la demo de nuevo. Un fallo por falta de stock no deja
una reserva que liberar, aunque el ejemplo de la guía también lo etiqueta
como `SAGA COMPENSADA`.

Guardar las evidencias de CQRS y Saga antes de realizar la limpieza final.

## Laboratorio 11 — API Gateway + Lambda + SQS

`api-gateway/main.tf` crea en `us-east-1`:

- La cola `devsecops-lab-orders-api-queue`.
- El rol `devsecops-lab-lambda-exec-role`, con logs básicos y permiso `sqs:SendMessage` solo sobre esa cola.
- Las funciones Lambda `devsecops-lab-create-order` y `devsecops-lab-health` (Node.js 20).
- Una HTTP API (`devsecops-lab-orders-api`) con las rutas `POST /orders` y `GET /health`.

`lambda/create-order.js` valida que el cuerpo traiga `productId` y `amount`, envía la orden
a SQS y responde `201`. Si el JSON es inválido o faltan campos, responde `400`.

```powershell
npm install
cd api-gateway
terraform init
terraform apply
$api = terraform output -raw api_url
Invoke-RestMethod "$($api)health"
Invoke-RestMethod -Method Post -Uri "$($api)orders" -ContentType "application/json" -Body '{"productId":"P-001","amount":25}'
cd ..
```

## Laboratorio 12 — Fan-out con SNS

`api-gateway/fanout.tf` agrega el tópico `devsecops-lab-order-events-topic` y las colas
`devsecops-lab-billing-queue` y `devsecops-lab-shipping-queue`, ambas suscritas al tópico.
Las políticas de cada cola solo aceptan mensajes enviados por SNS desde ese tópico.

`sns-fanout-consumer.js` lee los pedidos que dejó la API en `devsecops-lab-orders-api-queue`,
los republica en SNS y los elimina de la cola. SNS entrega una copia a cada cola suscrita.

```powershell
node sns-fanout-consumer.js
```

Verificación: en la consola de SQS, las colas billing y shipping reciben el mismo pedido.
Al terminar, si los recursos ya no se usan: `terraform destroy` dentro de `api-gateway/`.

## Laboratorio 13 — Kubernetes local con kind

Corre 100% en local (costo $0), con kind v0.33.0 (Kubernetes 1.37) sobre Docker Desktop.
Continúa en el Laboratorio 14.

- `k8s/kind-cluster-config.yaml`: clúster `devsecops-lab` de un nodo, con la etiqueta
  `ingress-ready=true` y los puertos 80/443 mapeados para el Ingress del Lab 14.
- `k8s/kafka-deployment.yaml`: Deployment de Kafka (`bitnamilegacy/kafka:3.7`, modo KRaft)
  y su Service `kafka`. Kafka se anuncia como `kafka:9092`, no como `localhost`.
- `k8s-app/`: productor y consumidor de Kafka (`kafkajs`) que se conectan a `kafka:9092`.
  El `Dockerfile` usa `node:20-alpine`, copia `package.json` antes del código para
  aprovechar el cache de capas y ejecuta la app con un usuario sin privilegios.
- `k8s/consumer-deployment.yaml`: Deployment `billing-consumer`, que escucha `order-events` sin detenerse.
- `k8s/producer-job.yaml`: Job `order-producer`, que publica una orden y termina.

```powershell
kind create cluster --config k8s/kind-cluster-config.yaml --name devsecops-lab
kubectl get nodes
kubectl apply -f k8s/kafka-deployment.yaml
kubectl get pods --watch
cd k8s-app
docker build -t devsecops-lab/k8s-app:1.0 .
cd ..
kind load docker-image devsecops-lab/k8s-app:1.0 --name devsecops-lab
kubectl apply -f k8s/consumer-deployment.yaml
kubectl apply -f k8s/producer-job.yaml
kubectl logs -l app=billing-consumer --tail=20
kubectl get pods
```

Resultado obtenido: `kafka` y `billing-consumer` en `Running`, `order-producer` en `Completed`,
y el consumidor muestra `[billing-service] Orden recibida desde la partición 0: KAFKA-...`.
La conexión se hizo solo por el nombre del Service (Service Discovery), sin IPs ni `localhost`.

El clúster se reutilizó en el Laboratorio 14.

## Laboratorio 14 — Imagen optimizada, probes e Ingress

Continúa sobre el clúster `devsecops-lab` del Laboratorio 13.

- `status-api/`: API Express con `GET /health` y `GET /`; la raíz devuelve el `hostname`
  del Pod y un contador de solicitudes propio de cada réplica.
- `status-api/Dockerfile`: construcción multi-stage. La etapa `deps` instala dependencias y
  la imagen final solo copia `node_modules`, `package.json` y `server.js`; corre como `appuser`.
- `k8s/status-api-deployment.yaml`: Deployment con 2 réplicas, `requests`/`limits` de CPU y
  memoria, `readinessProbe` y `livenessProbe` sobre `/health`, y Service `status-api` (80 → 3000).
- `k8s/status-api-ingress.yaml`: Ingress de clase `nginx` que envía `/` al Service `status-api`.

En PowerShell se usa `curl.exe`, porque `curl` es un alias de `Invoke-WebRequest`.

```powershell
cd status-api
docker build -t devsecops-lab/status-api:1.0 .
cd ..
kind load docker-image devsecops-lab/status-api:1.0 --name devsecops-lab
kubectl apply -f k8s/status-api-deployment.yaml
kubectl get pods -l app=status-api
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
kubectl wait --namespace ingress-nginx --for=condition=ready pod --selector=app.kubernetes.io/component=controller --timeout=120s
kubectl apply -f k8s/status-api-ingress.yaml
curl.exe http://localhost/health
curl.exe http://localhost/
kubectl scale deployment status-api --replicas=4
kubectl get pods
```

Resultado obtenido: las réplicas pasan de `0/1` a `1/1` cuando la readinessProbe responde;
las respuestas de `http://localhost/` alternan entre hostnames distintos (balanceo del Service)
y, tras escalar a 4 réplicas, aparecen nuevos hostnames sin modificar ningún YAML.

Limpieza final de los Labs 13 y 14:

```powershell
kind delete cluster --name devsecops-lab
docker rmi devsecops-lab/k8s-app:1.0 devsecops-lab/status-api:1.0
```

## Laboratorio 15 — Auto-escalado y resiliencia

Continúa sobre `status-api` en el clúster `devsecops-lab` (se recreó con la Parte B del Lab 13 y
las Partes A–D del Lab 14). Continúa en el Laboratorio 16 (chart de Helm).

- `k8s/metrics-server-kind-patch.json`: agrega `--kubelet-insecure-tls` a metrics-server, necesario
  en kind por los certificados autofirmados. Se aplica con `--patch-file` para evitar problemas de
  comillas en PowerShell.
- `k8s/status-api-hpa.yaml`: HPA de 2 a 6 réplicas con objetivo de 50 % de CPU promedio.
- `k8s/status-api-pdb.yaml`: PodDisruptionBudget con `minAvailable: 1`.
- `status-api/server.js`: el mensaje cambia a `DevSecOps Lab - Status API v2` para la imagen `2.0`.

```powershell
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
kubectl patch deployment metrics-server -n kube-system --type=json --patch-file k8s/metrics-server-kind-patch.json
kubectl top pods
kubectl apply -f k8s/status-api-hpa.yaml
kubectl run load-generator --image=busybox:1.36 --restart=Never -- /bin/sh -c "while true; do wget -q -O- http://status-api/ > /dev/null; done"
kubectl get hpa status-api-hpa --watch
kubectl delete pod load-generator
kubectl delete pod NOMBRE_DEL_POD
cd status-api
docker build -t devsecops-lab/status-api:2.0 .
cd ..
kind load docker-image devsecops-lab/status-api:2.0 --name devsecops-lab
1..90 | ForEach-Object { curl.exe -sS http://localhost/; ""; Start-Sleep -Seconds 1 }   # en otra terminal
kubectl set image deployment/status-api status-api=devsecops-lab/status-api:2.0
kubectl rollout status deployment/status-api
kubectl apply -f k8s/status-api-pdb.yaml
kubectl get pdb
```

Resultados obtenidos:

- Con carga, la CPU llegó a 171–279 % del objetivo y el HPA subió las réplicas de 2 a 4 y luego a 6.
- Al borrar un Pod, el Deployment creó uno nuevo automáticamente.
- Durante el rolling update las respuestas pasaron de `Status API` a `Status API v2` sin errores de conexión.
- `status-api-pdb` quedó con `MIN AVAILABLE 1` y `ALLOWED DISRUPTIONS 1`.

No destruir el clúster: el HPA, el PDB y `status-api` se reutilizan en el Laboratorio 16.
