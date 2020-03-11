### Health checking

Existen dos comandos que nos ayudarán en primera instancia a analizar la salud de nuestras instancias de redis:

* `redis-cli --stat`
* [`INFO`](https://redis.io/commands/info)

^^^^^^

### Health checking: `redis-cli --stat`

Muestra métricas básicas de la instancia de redis en tiempo real:

```bash
redis-cli --stat
keys       mem      clients blocked requests            connections
1003010    595.23M  1       0       1 (+0)              2
1003010    595.23M  1       0       2 (+1)              2
1003010    595.23M  1       0       3 (+1)              2
1003010    595.23M  1       0       4 (+1)              2
1003010    595.23M  1       0       5 (+1)              2
1003010    595.23M  1       0       6 (+1)              2
1003010    595.23M  1       0       7 (+1)              2
1003010    595.23M  1       0       8 (+1)              2
1003010    595.23M  1       0       9 (+1)              2
1003010    595.23M  1       0       10 (+1)             2
```

^^^^^^

### Health checking: `INFO`

Comando que hemos usado extensivamente en los módulos anteriores del curso.

Parámetros a los que debemos prestar atención:

* `total_connections_received`: Número de conexiones aceptadas por el servidor. Si este número
  aumenta rápidamente en un período corto de tiempo, podemos observar un aumento en el uso de CPU


^^^^^^

#### Health checking: `INFO`

* `instantaneous_ops_per_sec`: Número de comandos procesados por segundo
* `rejected_connections`: Número de conexiones rechazadas debido al límite impuesto por `maxclients`.
  Si este número es alto conviene revisar el uso de memoria  
   

^^^^^^

#### Health checking: `INFO`

* `sync_full`: Número de veces que las réplicas han hecho un `full resync` con el maestro
* `sync_partial_ok`: Número de veces que se ha completado un `partial resync` con el maestro
* `sync_partial_err`: Número de veces que el `partial resync` no se completó


^^^^^^

#### Health checking: `INFO`

* `evicted_keys`: Númoro de claves borradas debido al límite `maxmemory`
* `keyspace_misses`: Número de veces que no se encuentra una clave. Si esta métrica es alta,
  coviene revisar la aplicación para reducir su valor (no se está usando la cache adecuadamente)
* `latest_fork_usec`: Duración de la última operación que requirió un fork.

^^^^^^

#### Health checking: `INFO`

* `client_longest_output_list`: Si este parámetro aumenta, es una señal de que Redis no puede enviar
  a alguno de sus clientes. Suele ser por un problema en el propio cliente o en la red
  
  De forma orientativa, un valor de 100.000 se considera ya alto.
  

^^^^^^

#### Health checking: `INFO`

* `client_biggest_input_buf`: Buffer de entrada más grande entre las conexiones activas en ese momento.

  Si se superan lo 10MB, debemos revisar los clientes y ver quién nos está enviando un flujo tan alto de
  comandos que redis no los puede procesar
  

^^^^^^

#### Health checking: `INFO`

* `blocked_clients`: Número de clientes que están pendientes de operaciones con bloqueo como
  [`BLPOP`](https://redis.io/commands/blpop) o [`BRPOP`](https://redis.io/commands/brpop)
* `rdb_last_cow_size`: Tamaño en bytes de _copy-on-write allocations_ durante la generación del último
  fichero RDB
* `aof_last_cow_size`: Tamaño en bytes de _copy-on-write allocations_ durante la generación de la última
  reescritura del fichero AOF.


