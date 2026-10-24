# Tablón de notas — despliegue

Aplicación Flask (`app.py`) que guarda notas en una lista de Redis (`notas`, máximo 50 entradas) y se sirve con Gunicorn en el puerto **8000**.

## Requisitos

- Docker con el plugin **Compose** (`docker compose version`)

No hace falta instalar Python ni Redis: la imagen los incluye.

## Variables de entorno

| Variable | Obligatoria | Valor por defecto | Descripción |
|---|---|---|---|
| `REDIS_URL` | Sí en contenedor | `redis://localhost:6379` | URL de Redis. El `docker-compose.yml` la fija en `redis://cache:6379` (nombre del servicio). |
| `TITULO_APP` | No | `Tablón de notas` | Título que aparece en la página. |

Se definen **solo** en `docker-compose.yml` (`environment`), nunca en el `Dockerfile`.

## Puesta en marcha

```bash
docker compose up -d --build
```

El servicio `cache` debe pasar su healthcheck (`redis-cli ping`) antes de arrancar `web` (`depends_on: condition: service_healthy`).

Comprobar estado:

```bash
docker compose ps
```

Ambos servicios deben figurar `healthy` y solo debe publicarse `0.0.0.0:8000->8000/tcp`.

## Pruebas de comprobación

1. Salud del servicio (Redis accesible):

```bash
curl http://localhost:8000/salud
# {"estado":"ok","notas":0}
```

2. Alta de una nota:

```bash
curl -i -X POST -d "texto=nota de prueba" http://localhost:8000/notas
# HTTP/1.1 302 ...  (redirección a /)
```

3. Verificación en la portada (la nota debe aparecer en la lista):

```bash
curl -s http://localhost:8000/ | grep "nota de prueba"
```

4. El contador de `/salud` refleja el alta (`"notas":1`):

```bash
curl http://localhost:8000/salud
```

## Dónde quedan los datos

En el volumen con nombre **`notas-data`**, montado en `/data` del servicio `cache`, con persistencia AOF activada (`--appendonly yes`).

```bash
docker volume inspect notas-data
```

Los datos sobreviven a `docker compose down` y a reinicios; se borran solo con `down -v`.

## Cómo ver los registros

```bash
docker compose logs -f web
docker compose logs -f cache
```

## Parar y limpiar

```bash
docker compose down          # detiene contenedores y red, conserva notas-data
docker compose down -v       # detiene y elimina el volumen notas-data (borra las notas)
```

Opcional: eliminar también la imagen `notas-web` construida en local.

```bash
docker image rm notas-web
```
