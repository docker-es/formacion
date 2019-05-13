# ELK Introducción

ELK son las siglas de Elasticsearch, Logstash y Kibana y básicamente nos permiten tener un sistema de almacenamiento, gestión y visualización de los logs de nuestras aplicaciones. Hasta la aparición de este tipo de herramientas la explotación de la información que se volcaba en los logs podía ser bastante tediosa por lo que se hacía necesario un sistema que nos facilitase su organización y obtención así como una fácil, rápida y cómoda visualización.

Logstash es un software desarrollado por Elastic y básicamente permite recibir los logs de una fuente, procesarlos y enviarlos a Elasticsearch para su indexación.

Elasticsearch es otro software desarrollado por la gente de Elastic y del que ya hemos hablado en este blog en alguna ocasión, que permite la indexación de información pudiendo realizar búsquedas de textos muy efecientemente.

Por último tenemos a Kibana, desarroado también por nuestros amigos de Elastic y que básicamente permite obtener los datos indexados de Elasticsearch y mostrarlos en bonitas gráficas para facilitarlos la lectura y comprensión de la información.

Os podéis descargar el código de ejemplo de mi GitHub aquí.

Tecnologías empleadas:

    Java 8
    Gradle 3.1
    Junit 4.11
    Logback 1.2.2
    Logstash 2.0.0
    Elasticsearch 2.0.0
    Kibana 4.2.0

## Configuración y arranque del ELK

El fichero docker-compose.yml
```
elasticsearch:
  image: elasticsearch:2.0.0
  command: elasticsearch  -Des.network.host=0.0.0.0
  ports:
    - "9200:9200"
 
logstash:
  image: logstash:2.0.0
  volumes:
    - ./logstash/conf:/conf
    - ./logstash/logs:/logs
  links:
    - elasticsearch:elasticsearch
  command: logstash -f /conf/logspout.conf
  ports:
    - "9998:9998/udp"
 
kibana:
  image: kibana:4.2.0
  command: kibana -q
  links:
    - elasticsearch
  ports:
    - "5601:5601"
```
En el contenedor de logstash montaremos dos directorios.

    - En /conf montaremos el fichero de configuración de logstash
    - En /logs montaremos el fichero de log que utilizaremos para hacer pruebas

La idea consistirá en configurar logstash para que disponga de dos posibles fuentes de datos a través de los cuales ir suministrándole los logs. Por un lado tendremos un fichero de logs que se situará en **/logs/logging.log**, dentro del contenedor, y cuya información se indexará en elasticsearch en el índice indexfile. Por otro lado abriremos el puerto 9998 para que la información que llegue a través de udp sea indexada en el índice indexudp.

Para configurar la entrada de información utilizaremos la palabra reservada input
```
input{
    file{
       path => "/logs/logging.log"
        start_position => beginning
       type => "indexfile"
    }
    udp{
        port => "9998"
        type => "indexudp"
    }
}
```
Mientras que para la configuración de la salida utilizaremos la palabra reservada output
```
output {
  stdout{}
  if [type] == "indexfile"{
    elasticsearch {
       hosts => "elasticsearch:9200"
       index => "indexfile"
    }
  }
  if [type] == "indexudp"{
     elasticsearch {
       hosts => "elasticsearch:9200"
       index => "indexudp"
     }
  }
}
```
Sin embargo entre la entrada de información y su indexación en elasticsearch debe existir una transformación. Esto lo conseguiremos mediante el uso de grok que nos permitirá parsear las cadenas de logs en json que indexar.
```
filter {
  grok {
    match => { "message" => "%{IP:client} %{WORD:method} %{URIPATHPARAM:request} %{GREEDYDATA:loggingmessage}" }
  }
}
```
Se muestra todo junto en el fichero de configuración logstash.conf
```
input{
  file{
    path => "/logs/logging.log"
    start_position => beginning
    type => "indexfile"
  }
  udp{
    port => "9998"
    type => "indexudp"
  }
}
 
filter {
  grok {
    match => { "message" => "%{IP:client} %{WORD:method} %{URIPATHPARAM:request} %{GREEDYDATA:loggingmessage}" }
  }
}
 
output {
  stdout{}
  if [type] == "indexfile"{
    elasticsearch {
       hosts => "elasticsearch:9200"
       index => "indexfile"
    }
  }
  if [type] == "indexudp"{
     elasticsearch {
       hosts => "elasticsearch:9200"
       index => "indexudp"
     }
  }
}
```
Para arrancar el ELK basca con hacer:
	
    $ docker-compose up -d 

Verificamos que ha arrancado entrando en [http://localhost:5601](http://localhost:5601)

## Indexando logs desde fichero

Escribimos la siguiente línea en nuestro fichero ./logstash/logs/logging.log situado en nuestro filesystem
logging.log
	
192.168.1.1 GET /index.html This is the message from file

Damos de alta el índice indexfile en Kibana en **Settings > Índices**
Vamos a **Discovery** y vemos nuestro log indexado.

##  Indexado logs desde puerto udp

Para indexar la información a través del puerto 9998 de udp ejecutaremos un test configurando un appender de logback a través de la clase SyslogAppender.

build.gradle
```
group 'com.jorgehernandezramirez.elk'
version '1.0-SNAPSHOT'
 
apply plugin: 'java'
 
sourceCompatibility = 1.5
 
repositories {
    mavenCentral()
}
 
dependencies {
    testCompile group: 'junit', name: 'junit', version: '4.11'
    testCompile group: 'ch.qos.logback', name: 'logback-classic', version: '1.2.2'
}
```

logback-test.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
 
    <appender name="RSYSLOG" class="ch.qos.logback.classic.net.SyslogAppender">
        <syslogHost>localhost</syslogHost>
        <port>9998</port>
        <facility>LOCAL0</facility>
        <suffixPattern>%X{IPCLIENT:-192.168.1.1} %X{METHOD:-GET} %X{URIPATHPARAM:-localhost} %m%n</suffixPattern>
    </appender>
 
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <layout class="ch.qos.logback.classic.PatternLayout">
            <Pattern>
                %X{IPCLIENT:-192.168.1.1} %X{METHOD:-GET} %X{URIPATHPARAM:-index.html} %m%n
            </Pattern>
        </layout>
    </appender>
 
    <root level="info">
        <appender-ref ref="CONSOLE" />
        <appender-ref ref="RSYSLOG" />
    </root>
</configuration>
```

TestUdp.java
```
package com.jorgehernandezramirez.elk.testudp;
 
import org.junit.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.slf4j.MDC;
 
public class TestUdp {
 
    private static final Logger LOGGER = LoggerFactory.getLogger(TestUdp.class);
 
    @Test
    public void shouldIndexLogsIntoKibana(){
        LOGGER.info("This is the first message from junit test");
        MDC.put("IPCLIENT", "192.168.0.2");
        MDC.put("METHOD", "POST");
        MDC.put("URIPATHPARAM", "/index.html");
        LOGGER.info("This is the second message from junit test");
    }
}
```
Una vez ejecutado el test vamos a dar de alta el nuevo índice en **Settings > Índices**
Vamos a la sección **Discover** y vemos los mensajes indexados.