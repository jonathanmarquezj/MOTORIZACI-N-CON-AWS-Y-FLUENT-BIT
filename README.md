# MOTORIZACIÓN CON AWS Y FLUENT-BIT
En este ejemplo tendremos un escenario con varios servicios
- Drupal con PHP
- Nginx
- MariaDB
- Haproxy (Balanceador de carga)

## Necesitaremos los siguientes programas
- docker
- docker-compose

## Directorios necesarios dentro del proyecto
<pre>
mkdir volumen/mysql
mkdir volumen/files
</pre>

## Puesta en marcha

Lo primero es crear las imagenes de docker

<pre>
docker build -t "jonathan-nginx" ./nginx
docker build -t "jonathan-drupal" ./drupal
</pre>

Después solo tendremos que levantar el escenario

<pre>docker-compose up -d</pre>

## Explicación de la motorización
Lo que queremos es enviar el log de esos servicios que estan en contenedores al CloudWatch de AWS, para ello necesitaremos el contenedor con la imagen “amazon/aws-for-fluent-bit“, que sera el encargado de enviar los log al CloudWatch, tendremos que poner en docker-compose.yaml

<pre>
# FLUENT-BI
fluent:
    container_name: fluent-bit
    image: <b>amazon/aws-for-fluent-bit</b>
    volumes:
      - ./conf/fluent/fluent-bit.conf:/fluent-bit/etc/fluent-bit.conf
    restart: always
    networks:
      - red-entorno
    ports:
      - 24224:24224
      - 24224:24224/udp
    networks:
      - red-entorno
</pre>

En el cual le enviaremos el archivo de configuración “fluent-bit.conf” que es donde especificamos donde tiene que enviar los datos y donde recibe los datos, nos tendremos que fijar en las etiquetas “[INPUT]” y “[OUTPUT]”
<pre>
# ENTRADA DE DATOS
[INPUT]
    Name              forward
    Listen            0.0.0.0
    Port              24224
    Buffer_Chunk_Size 1M
    Buffer_Max_Size   6M

# MANDAMOS LOS DATOS A AWS
[OUTPUT]
    Name cloudwatch_logs
    Match   *
    region eu-west-1
    log_group_name fluent-bit-cloudwatch
    log_stream_prefix fluent-bit-cloudwatch-
    auto_create_group On
    workers 1
</pre>
En el “[INPUT]” podemos especificar varios parámetros:
- <b>Listen</b>: Es la IP de la cual entraran los datos
- <b>Port</b>: Es el puerto de escucha

En el caso de “[OUTPUT]”:
- <b>region</b>: La región del servidor
- <b>log_group_name</b>: Es el nombre del grupo del log que aparacera en AWS
- <b>log_stream_prefix</b>: Sera el nombre del flujo del servicio, en este caso le digo que empiece por “fluent-bit-cloudwatch-”
- <b>auto_create_group</b>: En el caso de que no este creado los grupos que los cree automáticamente

Ya solo queda decirle a los demás contenedores que envíen el log a fluent-bit, es tan fácil como ponerle lo siguiente a los contenedores:
<pre>
logging:
  driver: "fluentd"
  options:
    fluentd-address: localhost:24224
    tag: drupal.logs
</pre>
De tal manera que en un contenedor quedaría:
php:
<pre>
  container_name: drupal
  volumes:
    - type: volume
      source: staticfiles
      target: /opt/drupal/web
  image: jonathan-drupal
  restart: always
  hostname: drupal
  depends_on:
    - "fluent"
  networks:
    - red-entorno
  <b>logging:
    driver: "fluentd"
    options:
      fluentd-address: localhost:24224
      tag: drupal.logs</b>
</pre>

De tal manera que especificamos la ip del servidor, que en este caso es “localhost”, el puerto y por ultimo le ponemos el “tag” que sera lo que pondrá en el nombre del flujo