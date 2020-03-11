### Slow Queries

Redis dispone de un sistema para guardar un log de los Queries más lentos.

^^^^^^

### Slow Queries: Configuración

* `slowlog-log-slower-than`: Activa el log de queries lentos 

```bash
# /etc/redis.conf
slowlog-log-slower-than [microsegundos]
```

* Posibles valores:
  * `<0`: Desactiva el log
  * `0`: Registra todos los comandos
  * `>0`: Duración del comando para que quede registrado

^^^^^^

#### Slow Queries: Configuración

* `slowlog-max-len`: Longitud (nùmero de elementos) de la estructura FIFO que en la que se almacenan
  los queries
  
  No hay límite de tamaño. Aunque debemos tener en cuenta que esta estructura ocupa memoria

^^^^^^

### Slow Queries: SLOWLOG

[`SLOWLOG GET`](https://redis.io/commands/slowlog) es el comando de redis que nos muestra los queries lentos.


^^^^^^

### Slow Queries

Redis sólo computa el tiempo que tarda en ejecutarse la instrucción para decidir si un comando debe
guardarse en el log o no. 

**No se tiene en cuenta el tiempo de acceso a disco ni el tiempo de transferencia de red**. 
  
^^^^^^

### Slow Queries

Puede darse el caso de que desde la aplicación veamos un aumento de la latencia en la respuesta de Redis
pero no observemos ningún query lento en el log.

En este caso, el problema no está en el proceso de las operaciones sino en otro sitio como puede ser la
red o problemas con el acceso a disco.

^^^^^^

### 💻️ Slow Queries: Práctica 1
  
Fijamos el valor de `slowlog-log-slower-than` a un valor artificialmente bajo para ilustrar su funcionamiento:

```redis-cli
redis-cli > CONFIG SET slowlog-log-slower-than 5 
OK
```

^^^^^^

#### 💻️ Slow Queries: Práctica 1
  
Ejecutamos algunos comandos

```redis-cli
redis-cli > SET nombre "Curso de Redis" 
OK
redis-cli > HMSET alumnos:1 nombre Fulanito email fulanito@email.com
OK
```   
^^^^^^

#### 💻️ Slow Queries: Práctica 1

Consultamos el log

```redis-cli
redis-cli >> SLOWLOG GET
1) 1) (integer) 7
  2) (integer) 1583957040
  3) (integer) 12
  4) 1) "HMSET"
     2) "alumnos:1"
     3) "nombre"
     4) "Fulanito"
     5) "email"
     6) "fulanito@email.com"
  5) "127.0.0.1:42568"
  6) ""
2) 1) (integer) 6
  2) (integer) 1583956986
  3) (integer) 11
  4) 1) "SET"
     2) "nombre"
     3) "Curso de Redis"
  5) "127.0.0.1:42568"
  6) "" 
```

^^^^^^

#### 💻️ Slow Queries: Práctica 1

Cada entrada del log consta de:

* Un identificador para cada entrada del log
* El timestamp en el que se registró el comando
* Microsegundos utilizados en la ejecución
* Un array con los argumentos del comando
* IP del cliente
* Nombre del cliente

^^^^^^

### Slow Queries: otros comandos

* [`SLOWLOG LEN`](https://redis.io/commands/slowlog): Número de elementos en el slowlog
* [`SLOWLOG RESET`](https://redis.io/commands/slowlog): Borra los elemetnos en el sloglog
