# Monitoring Stack Demo

## Складові
- OpenTelemetry Collector
- Prometheus
- Fluentbit
- Grafana Loki
- Grafana

## Запуск

1. Встановіть Docker та Docker Compose.
2. Запустіть стек:
   ```bash
   docker-compose up -d
   ```
3. Grafana доступна на [http://localhost:3000](http://localhost:3000) (логін/пароль: admin/admin).
4. Prometheus доступний на [http://localhost:9090](http://localhost:9090).
5. Loki API: [http://localhost:3100](http://localhost:3100).

## Демо дашборд Grafana

1. Додайте джерела даних Prometheus та Loki у Grafana.
2. Імпортуйте демо-дешборд (ID: 1860 для Prometheus, ID: 13639 для Loki).
3. Приклад для метрик: [https://grafana.com/grafana/dashboards/1860](https://grafana.com/grafana/dashboards/1860)
4. Приклад для логів: [https://grafana.com/grafana/dashboards/13639](https://grafana.com/grafana/dashboards/13639)

> ![Demo Dashboard Prometheus](https://grafana.com/api/dashboards/1860/images/preview)
> ![Demo Dashboard Loki](https://grafana.com/api/dashboards/13639/images/preview)

## Проект інструментовано з наскрізним TraceID

- Всі сервіси інструментовані через OpenTelemetry SDK для передачі TraceID у метрики та логи.
- TraceID додається у кожен лог (наприклад, через middleware або логгер).
- Fluentbit парсить TraceID з логів та експортує у Grafana Loki.
- В Grafana можна шукати логи та метрики за TraceID для повного трейсингу запиту.

### Приклад для Python (FastAPI):
```python
from opentelemetry.instrumentation.fastapi import FastAPIInstrumentor
from opentelemetry.instrumentation.logging import LoggingInstrumentor
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
import logging

app = FastAPI()
FastAPIInstrumentor.instrument_app(app)
LoggingInstrumentor().instrument(set_logging_format=True)

# Додавайте TraceID у кожен лог
logging.basicConfig(format='%(asctime)s %(trace_id)s %(message)s')
```

### Приклад для Node.js (Express):
```js
const { NodeTracerProvider } = require('@opentelemetry/sdk-trace-node');
const { registerInstrumentations } = require('@opentelemetry/instrumentation');
const { ExpressInstrumentation } = require('@opentelemetry/instrumentation-express');
const { OTLPTraceExporter } = require('@opentelemetry/exporter-trace-otlp-grpc');
const { SimpleSpanProcessor } = require('@opentelemetry/sdk-trace-base');
const provider = new NodeTracerProvider();
provider.addSpanProcessor(new SimpleSpanProcessor(new OTLPTraceExporter()));
provider.register();
registerInstrumentations({ instrumentations: [new ExpressInstrumentation()] });

// Додавайте TraceID у кожен лог
app.use((req, res, next) => {
  const span = trace.getSpan(context.active());
  if (span) {
    req.traceId = span.spanContext().traceId;
  }
  next();
});
```

## Приклад метрик та логів
- Метрики збираються через OpenTelemetry Collector та Prometheus.
- Логи збираються Fluentbit та експортуються у Grafana Loki.

## Посилання
- [Офіційна документація Grafana](https://grafana.com/docs/)
- [Офіційна документація Prometheus](https://prometheus.io/docs/)
- [Офіційна документація Fluentbit](https://docs.fluentbit.io/manual/)
- [Офіційна документація Loki](https://grafana.com/docs/loki/latest/)
- [Офіційна документація OpenTelemetry](https://opentelemetry.io/docs/)
