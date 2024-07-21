# Guía del archivo docker-compose.yml para CnMaestro Reports

Este archivo `docker-compose.yml` se utiliza para configurar y ejecutar un contenedor Docker que genera reportes para un sistema CnMaestro on-premise.

## Estructura general

```yaml
version: '3.8'
services:
  cambium-report:
    # Configuración del servicio
networks:
  hostnet:
    # Configuración de la red
```

El archivo define un servicio llamado `cambium-report` y una red llamada `hostnet`.

## Configuración del servicio `cambium-report`

### Imagen Docker

```yaml
image: jbraconig/cambium-reports
```

Este servicio utiliza la imagen Docker `jbraconig/cambium-reports`.

### Volúmenes

```yaml
volumes:
  - /home/cambium/reports:/usr/src/app/data
  - /home/cambium/logs:/usr/src/app/logs
```

Se definen dos volúmenes:
1. `/home/cambium/reports` en el host se monta en `/usr/src/app/data` en el contenedor.
2. `/home/cambium/logs` en el host se monta en `/usr/src/app/logs` en el contenedor.

Esto permite persistir los reportes y logs generados fuera del contenedor.

### Variables de entorno

```yaml
environment:
  CAMBIUM_HOST: "cnmaestro"
  CAMBIUM_CLIENT_ID: ""
  CAMBIUM_CLIENT_SECRET: ""
  #CAMBIUM_DATA: ""
  CAMBIUM_INTERVAL: "300"
```

Se configuran las siguientes variables de entorno:
- `CAMBIUM_HOST`: Especifica el host de CnMaestro.
- `CAMBIUM_CLIENT_ID` y `CAMBIUM_CLIENT_SECRET`: Credenciales para autenticación (deben ser proporcionadas).
- `CAMBIUM_INTERVAL`: Intervalo de generación de reportes en segundos (300 = 5 minutos).

### Configuración de hosts

```yaml
extra_hosts:
  - "cnmaestro:127.0.0.1"
```

Añade una entrada al archivo `/etc/hosts` del contenedor, mapeando "cnmaestro" a la IP 127.0.0.1.

### Configuración de logs

```yaml
logging:
  driver: json-file
  options:
    max-size: 10m
```

Utiliza el driver de logging `json-file` con un tamaño máximo de archivo de 10MB.

### Configuración de despliegue

```yaml
deploy:
  replicas: 1
  restart_policy:
    condition: on-failure
```

Especifica que se debe ejecutar 1 réplica del servicio y que debe reiniciarse en caso de fallo.

### Configuración de red

```yaml
networks:
  hostnet: {}
```

El servicio se conecta a la red `hostnet`.

## Configuración de la red

```yaml
networks:
  hostnet:
    external: true
    name: host
```

Define una red externa llamada `hostnet` que utiliza la red `host` de Docker. Esto permite que el contenedor comparta la red del host.

## Notas importantes

1. Asegúrate de proporcionar valores válidos para `CAMBIUM_CLIENT_ID` y `CAMBIUM_CLIENT_SECRET`.
2. El intervalo de generación de reportes está configurado a 5 minutos (300 segundos). Ajusta según sea necesario.
3. Los reportes se almacenarán en `/home/cambium/reports` en el host.
4. Los logs se almacenarán en `/home/cambium/logs` en el host.
5. El servicio se reiniciará automáticamente en caso de fallo.

Para ejecutar este contenedor, asegúrate de tener Docker y Docker Compose instalados, y ejecuta `docker-compose up -d` en el directorio que contiene este archivo YAML.
