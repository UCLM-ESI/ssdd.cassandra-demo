# Replicación y Tolerancia a Fallos con Apache Cassandra

Esta demostración proporciona una guía paso a paso y comandos de consola para mostrar cómo
Apache Cassandra maneja la replicación de datos y garantiza la tolerancia a fallos en un
entorno distribuido.

## Demo

### Creación de un KeySpace y una Tabla

Inicia `cqlsh`, la consola de Cassandra:

```bash
CREATE KEYSPACE demo_keyspace WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 3};
```

Este comando crea un keyspace llamado "DemoKeyspace" con una estrategia de replicación
simple y un factor de replicación de 3. Esto significa que los datos se replicarán en tres
nodos.

### Crea una tabla en el keyspace

```bash
USE demo_keyspace;
CREATE TABLE demo_table (id UUID PRIMARY KEY, name TEXT, age INT);
```

### Añade y consultar datos

```bash
INSERT INTO demo_table (id, name, age) VALUES (uuid(), 'John Doe', 25);
SELECT * FROM demo_table;
```

### Verifica la replicación

```bash
CONSISTENCY QUORUM;
SELECT * FROM demo_table WHERE id = <id>;
```

Al establecer la consistencia en QUORUM, estamos asegurando que al menos la mayoría de los
nodos respondan antes de considerar la lectura o escritura como exitosa. Puedes ejecutar
consultas SELECT para verificar que los datos se lean correctamente desde varios nodos.


### Simulación de un fallo

Detén un nodo de Cassandra mediante:

```bash
$ docker compose stop <node>
```


### Reactivación del nodo parado

```bash
$ nodetool resumehandoff <node_id>
```

Puedes monitorizar el proceso de sincronización utilizando `nodetool`:

```bash
$ nodetool status
```


## Niveles de consistencia

Los niveles de consistencia controlan la forma en que se garantiza la coherencia de los datos entre los nodos del clúster. Hay varios niveles de consistencia disponibles, y la elección de un nivel específico afecta el equilibrio entre consistencia y disponibilidad. Aquí están los niveles de consistencia principales en Cassandra:

ANY: Este nivel de consistencia indica que se considera suficiente que la escritura o la lectura se realice en al menos un nodo. No hay garantía de que los datos se replicarán inmediatamente a otros nodos.

ONE: La operación se considera exitosa si se realiza en al menos un nodo responsable de la partición. Este nivel de consistencia proporciona una baja latencia y es adecuado para situaciones en las que la consistencia inmediata no es crítica.

TWO, THREE, QUORUM: Estos niveles requieren que la operación se realice en un número específico de nodos que poseen la partición. En el caso de QUORUM, se requiere la mayoría de los nodos (número de nodos / 2 + 1). Estos niveles ofrecen un equilibrio entre consistencia y disponibilidad.

ALL: La operación se considera exitosa solo si se realiza en todos los nodos responsables de la partición. Proporciona la mayor consistencia, pero puede resultar en una mayor latencia y menor disponibilidad, ya que todos los nodos deben estar disponibles para completar la operación.

LOCAL_ONE, LOCAL_QUORUM, EACH_QUORUM: Estos niveles se utilizan en clústeres distribuidos en múltiples centros de datos (datacenters). Permiten especificar niveles de consistencia para operaciones que afectan a nodos locales o requieren la participación de nodos en diferentes centros de datos.

SERIAL y LOCAL_SERIAL: Estos niveles se utilizan en combinación con operaciones de lectura y escritura de tipo "lightweight transactions" (LWT), que proporcionan un mecanismo para garantizar la consistencia a través de operaciones de tipo transacción.

La elección del nivel de consistencia depende de los requisitos específicos de la aplicación, considerando factores como la consistencia deseada, la latencia tolerada y la disponibilidad. Es importante comprender las implicaciones de rendimiento y latencia al seleccionar un nivel de consistencia particular, ya que niveles más altos de consistencia pueden resultar en una menor disponibilidad y mayores tiempos de respuesta.
