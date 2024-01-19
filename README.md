<p align="center">
  <a href="https://github.com/mvdiogo/observability">Observability com OpenTelemetry, Python, Jaeger, FastAPI</a>
  <h3 align="center">Observability</h3>
  <h4 align="center">Buscando uma prova de conceito, vamos analisar um estudo de caso, em que, uma aplicação, rodando um servidor Backend com FastAPI possa monitorar o comportamento da aplicação no Jaeger.</h4>
</p>

<p align="center">
  <a href="#instalação">Instalação</a> |
  <a href="#rodando-a-aplicação">Rodando a aplicação</a> |
  <a href="#links-de-referência">Links de referência</a> |
</p>

# Instalação

### Instale e configure docker em sua máquina

[Instalando docker](https://docs.docker.com/get-docker/)

### Verifique se as portas listadas abaixo estão livres em sua máquina local. Se sim instale o container Jaeger.

```

docker run -d --name jaeger \
  -e COLLECTOR_ZIPKIN_HOST_PORT=9411 \
  -e COLLECTOR_OTLP_ENABLED=true \
  -e OTEL_METRICS_EXPORTER='none' \
  -p 4317:4317 \
  -p 4318:4318 \
  -p 5775:5775/udp \
  -p 6831:6831/udp \
  -p 6832:6832/udp \
  -p 5778:5778 \
  -p 16686:16686 \
  -p 14268:14268 \
  -p 14269:14269 \
  -p 14250:14250 \
  -p 9411:9411 \
  jaegertracing/all-in-one:1.53 

```

### Crie um arquivo chamado 'requirements.txt' com o seguinte conteúdo:
```

fastapi==0.85.0
uvicorn==0.18.3
pydantic==1.10.2
opentelemetry-distro==0.33b0
opentelemetry-exporter-otlp-proto-grpc==1.12.0
opentelemetry-instrumentation-fastapi==0.33b0

```

### Crie um arquivo chamado 'app.py' com o seguinte conteúdo:

```

from fastapi import FastAPI
import time
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider

provider = TracerProvider()

trace.set_tracer_provider(provider)
tracer = trace.get_tracer(__name__)

app = FastAPI()

@app.get("/")
def get_homepage():
	count = 1
	while count <= 3:
		with tracer.start_as_current_span(f"loop-count-{count}") as span:
			print(f"Loop {count}")
			count += 1
			time.sleep(1)
	return {
		"status":"ok",
		"foo":"bar"
	}

```

### Crie e ativo o ambiente virtual em sua máquina. Depois instale as dependências:


```

python -m venv venv
source ./venv/bin/activate
pip3 install -r requirements.txt

```

# Rodando a aplicação

### Ative o container criado no docker

```

docker ps a
docker rm jaeger
docker start jaeger
docker ps

```
### Rode a aplicação criada

```

    opentelemetry-instrument --service_name my.first.api uvicorn app:app

```

### Entre no site abaixo para ver a aplicação rodando

```

http://127.0.0.1:16686/

```


# Links de referência

[cookbook](https://opentelemetry.io/docs/languages/python/cookbook/)

[Blog no signoz.io](https://signoz.io/blog/opentelemetry-fastapi/)

[Docker](https://docs.docker.com/build/building/opentelemetry/)

[Jaegertracing.io](https://www.jaegertracing.io/docs/1.17/getting-started/#all-in-one)

