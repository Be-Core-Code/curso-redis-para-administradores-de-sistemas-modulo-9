### Latencia

^^^^^^

### Latencia: `intrinsic-latency`

[_Baseline latency_](https://redis.io/topics/latency#latency-baseline) es un test para probar el sistema en el que se 
ejecuta Redis para medir lo rápido que puede ejecutar una operación.

```bash
> redis-cli --intrinsic-latency 60
Max latency so far: 1 microseconds.
Max latency so far: 133 microseconds.
Max latency so far: 200 microseconds.
Max latency so far: 376 microseconds.
Max latency so far: 621 microseconds.
Max latency so far: 687 microseconds.
Max latency so far: 1051 microseconds.
Max latency so far: 1181 microseconds.

1181161216 total runs (avg latency: 0.0508 microseconds / 50.80 nanoseconds per run).
Worst run took 23249x longer than the average latency.
```
^^^^^^

#### Latencia: `intrinsic-latency`

Es una medida de la latencia mínima del sistema sobre el que se está ejecutando Redis.

Depende de:
* Hardware
* Servidor físico vs Servidor virtual: el primero siempre tendrá una latencia intrínseca inferior
  al segundo (la diferencia puede llegar a ser hasta de uno o dos órdenes de magnitud)

^^^^^^

### Latencia: Causas

La latencia puede ser causada por diferentes factores:

* Network round-trip
* Aumento en el número de conexiones de Red
* Comandos que tardan mucho tiempo en ejecutarse
* `fork`
* Procesos de persistencia en segundo plano
* Uso de SWAP
* Expiración de claves


^^^^^^

### Latencia: Network round-trip latency

Otra medida que podemos tomar de nuestro sistema es el tiempo que tarda nuestro redis en responder
a un comando `PING`. 

^^^^^^

#### Latencia: Network round-trip latency

Este test lo ejecutamos desde la máquina cliente:

```bash
> redis-cli --latency
min: 0, max: 4, avg: 1.26 (5560 samples)
```

^^^^^^

#### Latencia: Network round-trip latency

[Más información](https://redis.io/topics/latency#latency-induced-by-network-and-communication)


^^^^^^

### Latencia: Conexiones de red

Un aumento del número de conexiones de red puede generar un aumento en la latencia de Redis.

Debemos monitorizar el parámetro `total_connections_received`:

```bash
> redis-cli info STATS |grep total_connections_received
total_connections_received:1
```
^^^^^^

#### Latencia: Conexiones de red

Otro valor que debemos monitorizar es el número de conexiones rechazadas, que debería ser bajo o
directamente cero:

```bash
> redis-cli info STATS |grep rejected_connections
rejected_connections:1
```

^^^^^^

#### Latencia: Conexiones de red

Podemos utilizar el comando `ifstat` para evaluar y monitorizar el tráfico de red.

```bash
> ifstat
#4636.262836907 sampling_interval=1 time_const=60
Interface        RX Pkts/Rate    TX Pkts/Rate    RX Data/Rate    TX Data/Rate
                 RX Errs/Drop    TX Errs/Drop    RX Over/Rate    TX Coll/Rate
lo                     0 0             0 0             0 0             0 0
                       0 0             0 0             0 0             0 0
eth0                   8 2             7 1           570 163         602 166
                       0 0             0 0             0 0             0 0
```

^^^^^^

### Latencia: Comandos

El comando [`INFO COMMANDSTATS`](https://redis.io/commands/info) nos da información del
número de comandos procesados y su tiempo medio de ejecución:

```bash
# Commandstats
cmdstat_set:calls=1,usec=198,usec_per_call=198.00
cmdstat_del:calls=1,usec=4,usec_per_call=4.00
cmdstat_brpop:calls=4290706,usec=62985774,usec_per_call=14.68
cmdstat_zadd:calls=141,usec=2099,usec_per_call=14.89
cmdstat_zrem:calls=4094324,usec=36135489,usec_per_call=8.83
cmdstat_zrangebyscore:calls=4104229,usec=81289997,usec_per_call=19.81
cmdstat_ping:calls=17547,usec=13805,usec_per_call=0.79
cmdstat_multi:calls=8374568,usec=8144421,usec_per_call=0.97
cmdstat_exec:calls=21400525,usec=280823107,usec_per_call=13.12
cmdstat_info:calls=6,usec=242,usec_per_call=40.33
cmdstat_config:calls=1,usec=3,usec_per_call=3.00
cmdstat_command:calls=2,usec=2582,usec_per_call=1291.00
```

^^^^^^

#### Latencia: Comandos

Usamos [`SLOWLOG`](https://redis.io/commands/slowlog) para ver cuáles son las operaciones 
más lentas.

^^^^^^

#### Latencia: Comandos

Debemos monitorizar el uso de CPU por parte del proceso Redis. 

Un uso alto de CPU por parte de este proceso con un tráfico de red normal o bajo suele ser una indicación
de que se están ejecutando comandos lentos.

^^^^^^

#### Latencia: Comandos

Posibles soluciones:

* Evitar la ejecución de estos comandos lentos
* Ejecutar estos comandos en una réplica
* Evaluar la posibilidad de cambiar las estructuras de datos para trabajar con conjuntos más pequeños
  o con comandos más eficientes
  
notes:

En la documentación de redis, todos los comandos suelen llevar asociada la complejidad
del algoritmo precisamente por este motivo.

^^^^^^

#### Latencia: Comandos

[Más información](https://redis.io/topics/latency#latency-generated-by-slow-commands)

^^^^^^

### Latencia: `fork`

La operación de `fork` que se realiza al ejecutar [`BGSAVE`](https://redis.io/commands/bgsave)
o [`BGREWRITEAOF`](https://redis.io/commands/bgrewriteaof) en un sistema Linux es cara.

Al hacer el fork, se deben copiar unos cuantos objetos relacionados con el proceso. El más relevante
es el _page table_ que se encarga de mapear las direcciones de memoria virtuales en direcciones
reales


notes:

En un sistema Linux de 64 Bits, el tamaño de las páginas es de 4k. Una instancia de Redis que esté utilizando
24GB de memoria requiere copiar 24GB / 4k * 8 = 48MB de memoria 

^^^^^^

#### Latencia: `fork`

Esta operación de `fork` introduce más latencia en una máquina virtual que en una máquian física.

Por ejemplo:
* El uso de Xen tiene un impacto sobre el tiempo de `fork` que puede ser hasta dos órdenes de magnitud mayor
* Si usas instancias de amazon, debes utilizar instancias basadas en HVM
* VMWare o KVM presentan rendimiento entre un 15 y 25% menor al sistema operativo nativo
 
^^^^^^

#### Latencia: `fork`

Debemos monitorizar el valor de `latest_fork_usec`:

```bash
> redis-cli INFO |grep latest_fork_usec
latest_fork_usec:257
``` 

^^^^^^

#### Latencia: `fork`

Recuerda que el uso del parámetro `transparent_huge_pages` del kernel debe de estar desactivado
si queremos que la latencia de `fork` sea lo más baja posible.

^^^^^^

#### Latencia: `fork`

* [Sobre latencia de fork](https://redis.io/topics/latency#latency-generated-by-fork)
* [Sobre `transparent_huge_pages`](https://redis.io/topics/latency#latency-induced-by-transparent-huge-pages)

^^^^^^

### Latencia: Persistencia

Como vimos en el módulo de Persistencia, es importante que el parámetro `appendfsync` sea `no` o `everysec`.

El uso de `always` será la opción más segura y la que más latencia generará en el nuestra instancia.

^^^^^^

#### Latencia: Persistencia

En este caso, podemos monitorizar los siguientes parámetros:

* `aof_delayed_fsync`: un aumento de este parámetro nos indica de veces que `fsync()` se tiene que
  posponer porque ya hay otro `fsync()` en proceso.
* `aof_pending_bio_fsync`: número de trabajos `fsync()` pendientes en la cola I/O

Un aumento en estos valores indica que el fichero AOF no se está escribiendo lo suficientemente rápido.

^^^^^^

#### Latencia: Persistencia

[Sobre latencia y disk /IO](https://redis.io/topics/latency#latency-due-to-aof-and-disk-io)

^^^^^^

### Latencia: SWAP

Como se indicó en el módulo 7, la configuración recomendada es eliminar el uso de la SWAP de aquellas
instancias en las que Redis se ejecute.

^^^^^^

#### Latencia: SWAP

Si esto no es posible, debemos mantener monitorizado el uso de SWAP en la máquina (`vmstat`, `iostat`) y del 
proceso de Redis.

^^^^^^

#### Latencia: SWAP

Si el sistema tiene un alto uso de SWAP:

* Aumentar memoria
* Reducir los procesos que consuman memoria o que generen paginación

^^^^^^

#### Latencia: SWAP
 
[Sobre SWAP](https://redis.io/topics/latency#latency-induced-by-swapping-operating-system-paging)
 
^^^^^^

### Latencia: Expiración de claves

Redis tiene dos mecanismos para expirar las claves:

* _lazy_: cuando se pide la clave, Redis comprueba que está expirada y la borra
* _activa_: expira algunas claves cada 100 milisegundos

^^^^^^

#### Latencia: Expiración de claves

Cada 100 milisegundos, Redis hace lo siguiente:

1. Selecciona 20 claves con expiración
1. Borra todas aquellas que hayan expirado
1. Si más de un 25% de las claves se borran, vuelve al paso 1

^^^^^^

#### Latencia: Expiración de claves

En el caso en el que tengamos muchas claves que expiran en el mismo segundo, este proceso
puede bloquear Redis hasta que el porcentaje de claves a expirar sea inferior al 25%. 

^^^^^^

### Latencia: otras herramientas

* `redis-cli --latency-dist`
* [_watchdog_](https://redis.io/topics/latency#redis-software-watchdog)
* 

^^^^^^

#### Latencia: `redis-cli --latency-dist`

![latency-dist](slides/images/latency-dist.png)<!-- .element: style="height: 60vh" -->

^^^^^^

#### Latencia: _watchdog_

Herramienta para depurar cuando con todos los demás métodos de diagnóstico no somos capaces de localizar el problema.

^^^^^^

#### Latencia: _watchdog_

* Usando [`CONFIG SET`](https://redis.io/commands/config-set) iniciamos _watchdog_
* Redis empieza a monitorizarse a sí mismo
* Si Redis detecta que se queda bloqueado en alguna operación que puede ser la causante del
  aumento de la latencia, genera un informe (dump)
* Como usuarios, enviamos este informe a los desarrolladores de Redis a través del grupo de usuarios de Redis en Google 


^^^^^^

#### Latencia: _watchdog_

Para activarlo:

```redis-cli
redis-cli > CONFIG SET watchdog-period 500
OK 
```

Para desactivarlo:

```redis-cli
redis-cli > CONFIG SET watchdog-period 0
OK 
```

^^^^^^

#### Latencia: _watchdog_

Este parámetro **no** se puede configurar a través del fichero `/etv/redis.conf`.

^^^^^^

#### Latencia: _Latency Monitoring Framework_

```redis-cli
redis-cli > CONFIG SET latency-monitor-threshold 100
OK 
```

Al activarlo, se guarda un log con los eventos que consumen más del tiempo configurado en el comando anterior.

^^^^^^
 
 #### Latencia: _Latency Monitoring Framework_

Se guarda una serie temporal que podemos consultar con los siguientes comandos:

* [LATENCY LATEST](https://redis.io/commands/latency-latest)
* [LATENCY HISTORY](https://redis.io/commands/latency-history)
* [LATENCY RESET](https://redis.io/commands/latency-reset)
* [LATENCY GRAPH](https://redis.io/commands/latency-graph)
* [LATENCY DOCTOR](https://redis.io/commands/latency-doctor)

^^^^^^

#### Latencia: _Latency Monitoring Framework_

[Más información](https://redis.io/topics/latency-monitor)