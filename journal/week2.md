# Week 2 — Distributed Tracing

## HoneyComb
You'll need to grab the API key from your honeycomb account:
```
export HONEYCOMB_API_KEY=""
gp env HONEYCOMB_API_KEY=""
```

Add teh following Env Vars to backend-flask in docker compose
```
OTEL_EXPORTER_OTLP_ENDPOINT: "https://api.honeycomb.io"
OTEL_EXPORTER_OTLP_HEADERS: "x-honeycomb-team=${HONEYCOMB_API_KEY}"
OTEL_SERVICE_NAME: "${HONEYCOMB_SERVICE_NAME}"
```
![otel](assets/addotl.PNG)

We'll add the following files to our ``requirements.txt``
```
opentelemetry-api 
opentelemetry-sdk 
opentelemetry-exporter-otlp-proto-http 
opentelemetry-instrumentation-flask 
opentelemetry-instrumentation-requests
```
![requirment](assets/requirment.PNG)


We'll install these dependencies:
```
pip install -r requirements.txt
```
Add to the ``app.py``
```
from opentelemetry import trace
from opentelemetry.instrumentation.flask import FlaskInstrumentor
from opentelemetry.instrumentation.requests import RequestsInstrumentor
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
```
![app](assets/addapp.PNG)

```
# Initialize tracing and an exporter that can send data to Honeycomb
provider = TracerProvider()
processor = BatchSpanProcessor(OTLPSpanExporter())
provider.add_span_processor(processor)
trace.set_tracer_provider(provider)
tracer = trace.get_tracer(__name__)
```
```
# Initialize automatic instrumentation with Flask
app = Flask(__name__)
FlaskInstrumentor().instrument_app(app)
RequestsInstrumentor().instrument()
```

![honeycomb](assets/honeyc.PNG)

![honey](assets/honey2.PNG)

## X-Ray
Instrument AWS X-Ray for Flask
```
export AWS_REGION="ca-central-1"
gp env AWS_REGION="ca-central-1"
```
Add to the requirements.txt
```
aws-xray-sdk
```
Install pythonpendencies
```
pip install -r requirements.txt
```
Add to app.py
```py
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.ext.flask.middleware import XRayMiddleware

xray_url = os.getenv("AWS_XRAY_URL")
xray_recorder.configure(service='Cruddur', dynamic_naming=xray_url)
XRayMiddleware(app, xray_recorder)
```
### Setup AWS X-Ray Resources
Add ``aws/json/xray.json``
```json
{
  "SamplingRule": {
      "RuleName": "Cruddur",
      "ResourceARN": "*",
      "Priority": 9000,
      "FixedRate": 0.1,
      "ReservoirSize": 5,
      "ServiceName": "Cruddur",
      "ServiceType": "*",
      "Host": "*",
      "HTTPMethod": "*",
      "URLPath": "*",
      "Version": 1
  }
}
```

```
aws xray create-group \
   --group-name "Cruddur" \
   --filter-expression "service(\"$FLASK_ADDRESS\")
```
![xray json](assets/addxrayjson.PNG)

![group name](assets/groupname.PNG)
```
aws xray create-sampling-rule --cli-input-json file://aws/json/xray.json
```
### Add Deamon Service to Docker Compose
```
  xray-daemon:
    image: "amazon/aws-xray-daemon"
    environment:
      AWS_ACCESS_KEY_ID: "${AWS_ACCESS_KEY_ID}"
      AWS_SECRET_ACCESS_KEY: "${AWS_SECRET_ACCESS_KEY}"
      AWS_REGION: "us-east-1"
    command:
      - "xray -o -b xray-daemon:2000"
    ports:
      - 2000:2000/udp
```
![daemon](assets/daemon.PNG)

We need to add these two env vars to our backend-flask in our ``docker-compose.yml`` file
```
      AWS_XRAY_URL: "*4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}*"
      AWS_XRAY_DAEMON_ADDRESS: "xray-daemon:2000"
```

![xray](assets/xrayaws.PNG)

![trace consol](assets/traceconsol.PNG)

![test log](assets/testlog.PNG)


### AWS X-Ray subsegment

![AWS X-Ray subsegment](assets/subsegment.jpg)

## Rollbar

Create a new project in Rollbar called ``Cruddur``

Add to ``requirements.txt``
```
blinker
rollbar
```
Install deps
```
pip install -r requirements.txt
```
We need to set our access token
```
export ROLLBAR_ACCESS_TOKEN=""
gp env ROLLBAR_ACCESS_TOKEN=""
```
Add to backend-flask for ``docker-compose.yml``
```
ROLLBAR_ACCESS_TOKEN: "${ROLLBAR_ACCESS_TOKEN}"
```
Import for Rollbar
```
import rollbar
import rollbar.contrib.flask
from flask import got_request_exception
```
![import](assets/importrollbar.PNG)
```
rollbar_access_token = os.getenv('ROLLBAR_ACCESS_TOKEN')
@app.before_first_request
def init_rollbar():
    """init rollbar module"""
    rollbar.init(
        # access token
        rollbar_access_token,
        # environment name
        'production',
        # server root directory, makes tracebacks prettier
        root=os.path.dirname(os.path.realpath(__file__)),
        # flask already sets up logging
        allow_logging_basic_config=False)

    # send exceptions from `app` to rollbar, using flask's signal system.
    got_request_exception.connect(rollbar.contrib.flask.report_exception, app)
```
We'll add an endpoint just for testing rollbar to ``app.py``
```
@app.route('/rollbar/test')
def rollbar_test():
    rollbar.report_message('Hello World!', 'warning')
    return "Hello World!"
```

![test](assets/helloword.PNG)

![rollbar](assets/rollbar.PNG)

![rol](assets/rl.PNG)
