### Memoria

En esta sección vamos a ver varias métricas que conviene monitorizar para detectar problemas
en la memoria de nuestra instancia Redis.

^^^^^^

#### Memoria

Como se comentó en la sección anterior, monitorizar el la memoria residente y uso de swap del
proceso redis a nivel del sistema operativo:

```bash 
> redis-cli INFO MEMORY |grep used_memory_rss_human
used_memory_rss_human: 1.03G
    
> redis-cli INFO|grep process_id
process_id: 9909
> awk '/VmSwap/{print $2 " " $3}' /proc/9909/status
0 kB 
```

^^^^^^

#### Memoria: ¿Qué es qué?

* `used_memory` Memoria asignada (allocated) por Redis usando llamadas a malloc (o a cualquier otra alternativa)
* `used_memory_rss` Memoria asignada (allocated) a Redis desde el punto de vista del sistema operativo (Resident Set Size)
* `maxmemory` Tamaño máximo de los datos
* `used_memory_dataset`: Tamaño de los datos
* `used_memory_overhead`: La suma de la memoria extra (_overhead_) que el servidor necesita para gestionar sus estructuras de datos internamente

Ver todos los parámetros en la documentación del comando [`INFO`](https://redis.io/commands/info)

^^^^^^
#### Memoria

La primera métrica que debemos monitorizar es `used_memory_human` sea inferior a `maxmemory_human`. 

```bash 
> redis-cli INFO MEMORY|egrep "used_memory_human|maxmemory_human"
used_memory_human:1016.45M
maxmemory_human:1.00G
```


^^^^^^

#### Memoria

Trazar el comportamiento de `evicted_keys`:

```bash 
> redis-cli INFO Stats|grep evicted_keys
evicted_keys:382901 
```

^^^^^^

#### Memoria

Debemos monitorizar el tamaño total que ocupan los datos:

```bash
bin/redis-cli INFO MEMORY  | egrep "used_memory_dataset|used_memory_dataset_perc"
used_memory_dataset:980896614
used_memory_dataset_perc:91.43%
```

En caso de que veamos un consumo alto de memoria, esto nos permitirá saber si la memoria la están
ocupando los datos o se trata de memoria interna utilizada por Redis

^^^^^^

#### Memoria

* [Optimización de memoria](https://redis.io/topics/memory-optimization)
* [Más sobre optimización de memoria](https://github.com/sripathikrishnan/redis-rdb-tools/wiki/Redis-Memory-Optimization)

