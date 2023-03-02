 # Week 2 — Distributed Tracing

 # Honeycomb

 • create an account: https://ui.honeycomb.io/ 
 
 • create an environment in Honeycomb and copy the api key and add in the Env Vars:

 ```bash
 export HONEYCOMB_API_KEY="f1OJ8uGau8KnatoEyMCuGE"
 gp env HONEYCOMB_API_KEY="f1OJ8uGau8KnatoEyMCuGE"
 export HONEYCOMB_SERVICE_NAME="Cruddur"
 gp env HONEYCOMB_SERVICE_NAME="Cruddur"
 ```
 ## Add Env Vars to `backend-flask` in ` docker-compose.yml`:

 ```yaml
 OTEL_SERVICE_NAME: "backend-flask"
 OTEL_EXPORTER_OTLP_ENDPOINT: "https://api.honeycomb.io"
 OTEL_EXPORTER_OTLP_HEADERS: "x-honeycomb-team=${HONEYCOMB_API_KEY}"
 ```
 
 ## Add files to ` requirements.txt`:
 
 ```
 opentelemetry-api 
 opentelemetry-sdk 
 opentelemetry-exporter-otlp-proto-http 
 opentelemetry-instrumentation-flask 
 opentelemetry-instrumentation-requests
 ```
 ## Install files:
 
 ```bash
 pip install -r requirements.txt
 ```
 
 I got back an error:
 
 ```bash
 ERROR: Could not open requirements file: [Errno 2] No such file or directory: 'requirements.txt'
 ```
 
 I was in the wrong directory:
 
 `cd backend-flask`
 
 Then ran `pip install -r requirements.txt` again.
 
 ## Add lines to `app.py`:
 
 ```python
 from opentelemetry import trace
 from opentelemetry.instrumentation.flask import FlaskInstrumentor
 from opentelemetry.instrumentation.requests import RequestsInstrumentor
 from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter
 from opentelemetry.sdk.trace import TracerProvider
 from opentelemetry.sdk.trace.export import BatchSpanProcessor
 ```

 Then add:
 
 ```python
 # Initialize tracing and an exporter that can send data to Honeycomb
 provider = TracerProvider()
 processor = BatchSpanProcessor(OTLPSpanExporter())
 provider.add_span_processor(processor)
 trace.set_tracer_provider(provider)
 tracer = trace.get_tracer(__name__)
 ```
 
 Then lastly add:
 
 ```python
# Initialize automatic instrumentation with Flask
app = Flask(__name__)
FlaskInstrumentor().instrument_app(app)
RequestsInstrumentor().instrument()
```

Documentation: https://docs.honeycomb.io/getting-data-in/opentelemetry/python/

## Add code to make ports open:

```yaml
ports:
  - name: frontend
    port: 3000
    onOpen: open-browser
    visibility: public
  - name: backend
    port: 4567
    visibility: public
  - name: xray-daemon
    port: 2000
    visibility: public
```


