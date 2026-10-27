# Sistemas distribuidos

Información (relevamiento) → Análisis → Propuesta

Se separa cada etapa para no confundir por ejemplo observación o relevamiento con análisis. 

**Afirmación:** En sistemas distribuidos, un observador remoto o externo no puede distinguir perfectamente entre una falla en el destino, en la comunicación, demora larga etc. **Hay incapacidad de saber qué pasó** y por lo tanto no tengo una visión perfecta. 

No hay sistemas perfectos, todo es trade off. 

**Los sistemas distribuidos** se ejecutan en máquinas distintas y se comunican por intercambio de mensajes. Las falacias de la red etc y no se puede garantizar el comportamiento. 

**El problema se vuelve distribuido**

- Modelo sincrónico
- Modelo asincrónico

FOTO FRAMEWORK DEL CURSO

### **Teorema CAP**

con estado compartido, hay 3 cosas que no pueden garantizarse en el mismo momento, **consistencia, disponibilidad y tolerancia a particiones** (cuando hay nodos hablando que no pueden hablar entre si, por ej. se cortó la red). 
Como no puedo evitar que haya problemas en la comunicación entre nodos, el teorema me lleva a pensar en los dos primeros. Eso pq no puedo garantizar que la red sea estable, por eso debo terminar eligiendo entre consistencia o disponibilidad. 

Si priorizo consistencia, digo no respondo hasta que los equipos sean consistentes. Ahora, si quiero disponibilidad debo responder cuando me preguntan (eso es disponibilidad) y si hago eso no estoy garantizando consistencia. 

**El pensamiento distribuido** consiste en decidir correctamente bajo incertidumbre. No se trata de encontrar una solución perfecta sino de explicar supuestos, elegir garantías y defender el trade-off que se acepta. 

## Clase 2

### Comunicar no es instantáneo

Una operación remota atraviesa varias capas y cada una puede responder distinto ante una falla.

Aplicación → RPC / cliente HTTP → Transporte → Red → Servidor

- Una capa puede reintentar mientras otra ya venció su timeout
- Una capa puede ocultar pérdida y otra observar demora.

### RPC (Remote Procedure Call)

**Abstracción:**

- Llamada local: función
- Llamada remota: red

A communication protocol that allows a computer program to run a function or subroutine on another computer across a network as if it were a local function.

### Timeout

**Timeout**: No implica falla, solo indica que no hubo respuesta dentro de una ventana. 
Alguien en todo este camino no pudo esperar lo suficiente. (A veces no es él mismo, sino algún otro punto de toda la cadena)

Hay que saber identificar que capa está fallando, hay que saber de quien es el reporte del error (Troubleshooting). Sabiendo que capa está fallando nos permite arreglar el problema y mejorar el sistema. 

### Retry

**¿Qué hacemos con esa falla?**

Retry

Cuando empiezan a fallar los sistemas y tienen reintentos pueden recargar los sistemas y hacer que el sistema se cargue. 

Retry introduce:

- Recuperación ante fallas transitorias
- Duplicación potencial
- Presión adicional sobre el sistema

**Ejemplo**: Pago duplicado si el primer cobro ocurrió y se perdió la respuesta.

Hay que entender el riesgo de mandar el mismo request varias veces. Puedo reintentar una, dos, tres veces, hay que diseñar bien esas políticas de retry y entender las consecuencias de hacerlo. 

### Idempotencia

Es la capacidad de darse cuenta que dos mensajes son iguales. Ya recibí ese mensaje 2 veces, el sistema se da cuenta y no lo procesa de nuevo. (Puede producir el mismo resultado)

Definición PPT: Aplicar la operación múltiples veces produce el mismo resultado observable.

Ejemplos: PUT idempotente, POST no necesariamente.

### Semánticas de entrega

- at-most-once: evita duplicación, puede perder ejecución.
- at-least-once: evita pérdida, puede duplicar.
- exactly-once: garantía lógica costosa, no magia de la red.

### Trade-off

- Confiabilidad versus complejidad.
- Latencia versus corrección.
- Retry versus duplicación.
- Idempotencia versus más estado.

Comunicar en sistemas distribuidos es decidir bajo incertidumbre y reconstruir qué pudo haber ocurrido en cada capa.

## Replicación

Replicación = múltiples nodos mantienen el mismo estado lógico.

Motivaciones:

- Disponibilidad
- Latencia (lecturas cercanas)
- Tolerancia a fallas

**Hecho estructural:** Sin coordinación perfecta, las réplicas **divergen**.

La divergencia falla cuando le puedo preguntar a cualquiera de las copias para cualquier información.

#### Por qué divergen?

- Latencias variables
- Pérdidas / Reintentos (Cualquier catástrofe puede pasar)
- Escrituras concurrentes
- Particiones

La divergencia no es un bug, es una consecuencia del modelo (de la vida según el profesor)

# Modelos de consistencia

### Consistencia define

Qué valores pueden observar los clientes y en qué orden.

## Linearizabilidad (consistencia fuerte)

- Equivalente a una sola copia
- Respeta orden de tiempo real

Cuando se escribe en un nodo, el nodo tiene que copiar en todo el resto de nodos para así poder devolver el okay para seguir con los otros users.

Intuición: Cada operación ocurre en un punto único entre invocación y respuesta.

## Causalidad (happens-before)

Si no están relacionados no me interesa

Relación a → b si:

- Mismo  proceso y a ocurre antes que b
- a envía mensaje que causa b
- La relación se propaga por transitividad

Consistencia causal: Respeta dependencias causales, no impone orden total, siempre mismo orden acciones que se relacionan.

## Tiempo físico no alcanza

Comparar timestamps de nodos distintos puede engañar: 

- Relojes locales derivan o se corrigen;
- La red reordena o demora mensajes;
- Una hora menor no prueba causalidad.

¿Qué evento pudo influir sobre qué otro evento?

## Relojes de Lamport

Cada proceso mantiene un contador lógico:

- Incrementa antes de cada evento local;
- Adjunta el contador al enviar mensajes;
- Al recibir, usa max(local, recibido) + 1

(faltan cosas)

## Eventual consistency

Eventual consistency is **a system design model where data updates propagate asynchronously across distributed nodes so that all replicas eventually match if no new updates are made.**

Git y los DNS trabajan así

- Sin nuevas escrituras, las réplicas convergen
- No hay garantías inmediatas de lectura

**Trade-off:**

- Fuerte: coordinación + latencia + menor disponibilidad
- Débil: disponibilidad + anomalías temporales

## Arquitecturas de replicación

### Leader-based

- Un líder serializa escrituras
- Followers replican (sync/async)

### Multi-leader

- Múltiples líderes
- Escritura local rápida
- Conflictos inevitables

### Leaderless (Dynamo)

- N réplicas
- Escribir a W, leer de R
- Condición clásica: R + W > N

La **arquitectura de replicación *leaderless*** (sin líder o descentralizada) es un **modelo de bases de datos distribuidas donde cualquier nodo puede aceptar operaciones de lectura y escritura, eliminando la figura del nodo maestro o líder coordinador**. 

#### **Funcionamiento Principal:**

**Escritura descentralizada:** El cliente envía los datos directamente a múltiples réplicas en paralelo.

**Lectura múltiple:** Para leer, el cliente también consulta a varios nodos para comparar versiones y obtener la información más actualizada.

**Ausencia de jerarquía:** No existe un único punto de entrada ni un registro centralizado que determine el orden absoluto de los eventos. 

#### **Mecanismos de Consistencia y Quórum**

Como no hay un líder que ordene los cambios, estos sistemas usan el **quórum** para asegurar que los datos sean confiables. Si tienes n réplicas totales: 

- Se define un cuórum de escritura w (cuántos nodos deben confirmar el cambio).
- Se define un cuórum de lectura r (cuántos nodos se deben consultar).

Si w + r > n, se garantiza obtener la versión más reciente del dato (lectura consistente), ya que los conjuntos de nodos de lectura y escritura se intersectan al menos en uno. 

#### **Sincronización y Reparación de Datos**

**Reparación en lectura (*Read repair*):** Cuando un cliente detecta una réplica desactualizada durante una lectura, envía el valor correcto a dicho nodo. 

**Anti-entropía:** Un proceso en segundo plano compara los datos entre nodos. Se suelen usar **árboles de Merkle** para detectar diferencias de manera eficiente sin transferir toda la base de datos.

#### **Ventajas y Desventajas**

**Ventajas:** Alta disponibilidad y tolerancia total a fallos; si un nodo cae, el sistema sigue operando sin interrupciones ni necesidad de elegir un nuevo líder. 

**Desventajas:** Mayor complejidad en la resolución de conflictos por escrituras concurrentes y tendencia a la consistencia eventual (los datos pueden estar desactualizados de manera temporal)

## Conflictos y convergencias

### Conflictos

- Escrituras concurrentes sobre un objeto
- a

### Resolución

- LWW (last-write-wins)
- Merge semántico
- Versionado (vector clocks)

## Vector clocks

Permiten comparar versiones:

- Si un vector domina a otro, hay descendencia causal;
- Si no son comparables, hay concurrencia.
    
    (2, 1, 0) || (1, 2, 0)
    

Uso: Detectar conflicto y forzar merge en vez de sobrescribir datos

**Estructura:** En un sistema con N procesos, cada nodo mantiene un vector de N enteros (un contador por cada nodo del sistema).

**Perspectiva local:** Cada entrada en el vector representa la estimación del nodo sobre la hora lógica de los otros procesos. 

#### **Reglas de actualización**

**Inicio:** Todos los contadores del vector empiezan en cero.

**Evento local:** El proceso incrementa su propio contador en su vector por uno.

**Enviar mensaje:** El proceso envía su propio vector actual junto con el mensaje.

**Recibir mensaje:** El nodo receptor actualiza cada elemento tomando el valor máximo entre su reloj local y el del mensaje recibido, y luego incrementa su propia posición en uno. 

#### **Comparación y Causalidad**

**Sucede antes (A → B):** Ocurre si cada valor del reloj A es menor o igual al de B, y al menos un valor es estrictamente menor.

**Concurrencia (A || B):** Ocurre si ningún reloj es menor o igual al otro; significa que los eventos ocurrieron de forma independiente o simultánea.

## Anti-entropy

- Sincronización periódica entre réplicas
- Gossip / read-reapair

## Modelo del curso (Aplicación de D = (P, F, G, S, T))

- P: Estado replicado
- F: Delay, pérdida, partición
- G: (linearizable / causal / eventual)
- S: leader, multi-leader, quorum
- T: Latencia, disponibilidad, complejidad

Replicar estado obliga a elegir qué inconsistencias aceptar y cuándo.

# Algoritmos de Consenso

Usas un algoritmo de consenso (como Raft o Paxos) **cuando necesitas que un sistema distribuido se comporte como si fuera una sola computadora**, garantizando que los datos sean idénticos en todos lados y que el sistema siga funcionando aunque se rompan algunos servidores o cables.

El consenso tiene un costo alto: **la velocidad**. Como el líder tiene que esperar el voto de la mitad más uno de los nodos para cada escritura, el sistema se vuelve más lento que una base de datos tradicional.

No usar si:

- **Tu aplicación tolera la "Consistencia Eventual":** Por ejemplo, si en una red social un *Like* tarda 2 segundos en aparecerle a otro usuario, no pasa nada. Herramientas como Cassandra o DynamoDB priorizan la velocidad sobre el consenso estricto.
- **Un único servidor es suficiente:** Si tu negocio funciona bien con una base de datos SQL tradicional (PostgreSQL, MySQL) bien respaldada, meter Raft o Paxos añade una complejidad técnica innecesaria.

## Raft

Raft es un [**algoritmo de consenso**](https://www.yugabyte.com/key-concepts/raft-consensus-algorithm/) diseñado para sistemas distribuidos que **permite a un grupo de servidores ponerse de acuerdo sobre un estado compartido o una secuencia de operaciones, incluso si algunos nodos fallan**. 

![image.png](Sistemas%20distribuidos/image.png)

(Donde n es el número total de nodos y los corchetes significan redondear hacia abajo).

**¿Cómo funciona Raft?**

Raft divide el problema del consenso en tres subproblemas principales para hacerlo más fácil de entender que otros protocolos como Paxos: 

- **Elección de líder:** Un servidor coordina todo el clúster.
- **Replicación de registros (logs):** El líder asegura que todos los demás tengan la misma información.
- **Seguridad:** Garantiza que ningún nodo aplique datos contradictorios.

**Los Estados de un Nodo**

En cualquier momento, un servidor en Raft se encuentra en uno de estos tres estados: 

- **Seguidor (*Follower*):** Recibe órdenes pasivamente del líder y responde a sus mensajes.
- **Candidato (*Candidate*):** Estado temporal al que pasa un seguidor si deja de recibir señales de vida y quiere postularse para líder.
- **Líder (*Leader*):** Gestiona todas las solicitudes de los clientes y coordina la replicación en el resto de los nodos.

**Fases del Algoritmo**

**1. Elección de Líder**

Los seguidores esperan mensajes periódicos de confirmación o latidos (*heartbeats*) del líder actual.

Si un seguidor no recibe latidos antes de que termine su temporizador aleatorio (*election timeout*), asume que el líder murió.

El nodo se vuelve **candidato**, incrementa el número de periodo (*term*) y vota por sí mismo pidiendo votos a los demás.

Si el candidato obtiene la mayoría de los votos de los servidores (**quórum**), se convierte en el nuevo **líder**. 

**2. Replicación de Registros (*Log Replication*)**

El cliente envía una petición o comando directamente al líder.

El líder añade esta orden a su propio registro de forma secuencial.

El líder envía la orden a los seguidores mediante mensajes (*AppendEntries*).

Cuando una mayoría de seguidores confirma que copió la entrada en su registro, el líder **confirma (*commits*)** la operación.

El líder aplica el cambio a su máquina de estados y le avisa al cliente; los seguidores también aplican el cambio en cuanto se enteran. 

**3. Tolerancia a Fallos y Consistencia**

Si un nodo se desconecta o la red falla, el líder reintenta enviar los mensajes hasta que se recupere.

Si el líder cae, los seguidores detectan la falta de latidos, inician una nueva elección y el nuevo líder sincroniza los registros rezagados de los demás nodos. 

**¿Por qué se necesita una mayoría absoluta?**

Imagina un clúster de **5 nodos**. El quórum es (5/2  + 1 = 3) nodos.

Si el líder quiere guardar un dato, no puede dar la operación por buena teniendo solo su propia confirmación. Debe enviarla a los demás y esperar a que al menos **3 de los 5 nodos** (incluyéndose él mismo) la hayan guardado de forma segura.

Una vez que alcanza ese número mágico, el dato está **consolidado (*committed*)** y ya no se puede perder.

**El beneficio: Solapamiento Garantizado**

La magia de la mayoría absoluta es geométrica: **dos mayorías de un mismo grupo siempre van a compartir, al menos, un nodo en común.**

Si tienes 5 nodos y el Líder A guardó un dato en los nodos {1, 2, 3}, y luego hay una falla de red y se intenta elegir un Líder B con los nodos {3, 4, 5}:
• El nodo **3** está en ambos grupos.
• Como el nodo 3 ya tiene el dato del Líder A, cuando el Líder B le pida su voto, el nodo 3 le dirá: *"Oye, mi registro es más reciente que el tuyo, no te voto"*.
• Esto garantiza que el nuevo líder **siempre** tendrá la información más actualizada del sistema.

**3. Evita el "Cerebro Dividido" (*Split-Brain*)**

¿Qué pasa si un cable se rompe y el clúster de 5 nodos se divide en dos mitades aisladas?

• **Lado Izquierdo (2 nodos):** Intentan elegir un líder. No pueden porque el mínimo para el quórum es 3. Este lado se queda bloqueado y no acepta escrituras.
• **Lado Derecho (3 nodos):** Tienen la mayoría. Detectan que el viejo líder no está, eligen uno nuevo (alcanzando los 3 votos) y siguen funcionando perfectamente.Si la regla fuera simplemente "cualquier grupo puede operar", ambos lados crearían líderes distintos, escribirían datos contradictorios y el sistema se rompería al volver a conectarse. Al exigir obligatoriamente la mitad más uno, **solo un lado de la red puede operar a la vez**.

## Paxos

Paxos es un algoritmo de consenso diseñado para sistemas distribuidos que permite a un grupo de servidores ponerse de acuerdo sobre un único valor o una secuencia de valores, incluso si algunos nodos o mensajes de red fallan. Fue el pionero en este campo matemático, diseñado originalmente por Leslie Lamport.

**¿Cómo funciona Paxos?**
A diferencia de Raft, que se basa en un líder fuerte y estados estrictos, Paxos aborda el consenso basándose en **roles**. Un mismo nodo físico suele jugar varios de estos roles al mismo tiempo:

- **Proponente (Proposer):** Recibe las solicitudes del cliente e intenta convencer a la red de que acepte su valor propuesto. Actúa como el coordinador de una votación.
- **Aceptador (Acceptor):** Actúa como votante. Escucha a los Proponentes, verifica las reglas matemáticas del algoritmo y decide si acepta o rechaza una propuesta. Memoria del sistema.
- **Aprendiz (Learner):** Se entera de qué valor fue finalmente aceptado por la mayoría y lo aplica o le avisa al cliente.

**Fases del Algoritmo**
Para ponerse de acuerdo en un dato (lo que se conoce como *Basic Paxos*), el algoritmo usa un sistema de votación en dos fases basado en números de propuesta progresivos:

**1. Fase de Preparación (Promesa)**

- Un Proponente crea un número de propuesta único `N` (siempre mayor a cualquier número que haya usado antes) y envía un mensaje `Prepare(N)` a los Aceptadores.
- Si un Aceptador recibe este `Prepare(N)` y el número `N` es el más alto que ha visto hasta ahora, responde con una **promesa**: "Prometo no aceptar ninguna propuesta futura que tenga un número menor a `N`".
- Si el Aceptador ya había aceptado un valor anteriormente, se lo envía de vuelta al Proponente en su respuesta.

**2. Fase de Aceptación (Consolidación)**

- Si el Proponente recibe promesas de la mayoría de los Aceptadores (quórum), avanza. Si los Aceptadores le devolvieron valores previos en la fase 1, el Proponente está obligado a descartar su propio valor y adoptar el valor del Aceptador que tenía el número de propuesta más alto.
- El Proponente envía un mensaje `Accept(N, Valor)` a los Aceptadores.
- Los Aceptadores reciben la petición y guardan el valor, **siempre y cuando** no hayan prometido responderle a otro Proponente con un número mayor en ese transcurso de tiempo.
- Cuando una mayoría absoluta acepta el valor, se considera consolidado (committed).

**¿Por qué se necesita una mayoría absoluta?**
Imagina un clúster de 5 nodos. El quórum es (5/2 + 1 = 3) nodos.
Un Proponente no puede dar un dato por válido por su cuenta. Debe asegurarse de que al menos 3 de los 5 Aceptadores (la mayoría) hayan guardado el valor. Una vez que alcanza este número, el dato es inmutable.

**El beneficio: Solapamiento Garantizado**
La magia geométrica de la mayoría absoluta es idéntica a la de Raft: dos mayorías de un mismo grupo siempre tendrán al menos un nodo en común.
Si tienes 5 nodos y el Proponente A logra que el valor "X" se guarde en los nodos {1, 2, 3}, y luego el Proponente B intenta proponer el valor "Y" en los nodos {3, 4, 5}:

- El nodo 3 está en ambos grupos.
- Cuando el Proponente B envíe su mensaje `Prepare(N)` al nodo 3, este le dirá: "Te prometo escucharte, pero ojo, yo ya acepté el valor 'X' anteriormente".
- Por las reglas de Paxos, el Proponente B se ve obligado a abandonar su valor "Y", adoptar "X" y proponerlo al resto.
- Esto garantiza que, una vez que la mayoría acepta un dato, es matemáticamente imposible que la red elija un dato distinto.

**Evita el "Cerebro Dividido" (Split-Brain)**
Si un problema de red divide el clúster de 5 nodos en dos partes aisladas:

- **Lado Izquierdo (2 nodos):** Si un Proponente intenta iniciar una votación, nunca conseguirá promesas de 3 nodos. Este lado de la red se bloquea de forma segura y rechaza operaciones.
- **Lado Derecho (3 nodos):** Al retener la mayoría absoluta, cualquier Proponente en esta mitad sí conseguirá el quórum de 3 votos. Este lado seguirá procesando información y guardando estado, impidiendo que ambas mitades tomen decisiones contradictorias mientras estén incomunicadas.

# Seriabilidad, anomalías y 2PC/3PC

## De orden global a correctitud

- Consenso ⇒ orden total de operaciones
- Falta: **correctitud lógica de operaciones compuestas**

¿Cómo ejecutar múltiples operaciones concurrentes sin violar invariantes?

**Una transacción ACID es una unidad de trabajo en una base de datos que garantiza la máxima fiabilidad e integridad mediante cuatro propiedades clave: Atomicidad, Consistencia, Aislamiento y Durabilidad**. O se ejecutan todas las operaciones por completo o no se realiza ninguna. 

**Propiedades de ACID**

**Atomicidad:** Cada instrucción se trata como una unidad indivisible ("todo o nada"). Si un paso falla, se revierten todos los cambios previos. 

**Consistencia:** La base de datos siempre pasa de un estado válido a otro estado válido, cumpliendo todas las reglas y restricciones del sistema. 

**Aislamiento:** Las operaciones simultáneas no interfieren entre sí; el resultado de ejecutar varias transacciones a la vez es el mismo que si se hicieran una tras otra. 

**Durabilidad:** Una vez que la transacción se confirma con éxito (*commit*), los datos se guardan de forma permanente, incluso si ocurre un corte de luz o un fallo del sistema. 

#### Idea central

Una ejecución concurrente es correcta si equivale a una ejecución secuencial.

## Serializables

Si el orden de ejecución de 2 transacciones no afecta al resultado, no son serializables.

## Dirty read

- T1 escribe
- T2 lee valor no confirmado
- T1 aborta

## Non-repeatable read

- T1 lee valor
- T2 modifica
- T1 vuelve a leer distinto valor

## Write skew

- Ambas transacciones leen mismo estado
- Ambas escriben sin conflicto directo

**Clave: Snapshot Isolation permite esto ⇒ No serializable**

## SQL Estándar

- Read Uncommitted
- Read Committed
- Repeatable Read
- Serializable

## Snapshot Isolation

- Snapshot consistente al inicio
- Evita dirty read y non-repearable read
- NO evita write skew

# Storage Distribuido

Significa almacenamiento distribuido: nodos y protocolos que convierten operaciones en estado permanente, recuperable y consultable aun cuando ocurren fallas.

- Conserva decisiones después de un reinicio.
- Coordina lecturas y escrituras simultáneas.
- Encuentra datos con latencia predecible.
- Absorbe carga sin acumular trabajo sin límite.

| Capa Distribuida | Motor local |
| --- | --- |
| Ubica y replica los datos; además, decide qué nodo tiene autoridad para confirmar ccada operación. | Convierte los bytes persistentes… |

El motor de almacenamiento es el software que traduce operaciones lógicas en bytes, versiones,…

### Caso conductor: CampusShop

Una estudiante compra la última unidad de un producto del campus. La API (interfaz de programación de aplicaciones) recibe la operación con identificador estable 9F2A y responde “compra confirmada”. Un segundo después, el nodo pierde energía.

- La compra debe existir exactamente una vez al reiniciar.
- La unidad descontada no debe reaparecer en el inventario.
- Un reporte largo debe conservar una vista coherente mientras entran nuevas compras.
- La consulta de loas últimas 20 compras debe evitar revisar toda la tabla-
- Una carga sostenida no debe crear una cola interna sin límite.

## Camino crítico y patrón de entrada/salida

| Camino crítico de una escritura | E/S aleatoria y secuencial |
| --- | --- |
| Secuencia de pasos que debe completarse antes de enviar la confirmación al cliente. El paso más lento de esa secuencia condiciona la latencia observada |  |

## Árbol B y LSM-tree: dos formas de organizar datos

Árbol B: Índice ordenado y balanceado cuyos nodos suelen corresponder a páginas. Permite localizar una clave o recorrer un rango con pocas lecturas de páginas- costo: Una escritura puede modificar

**LSM-tree: Log-Structured Merge-tree:** Acumula escrituras en memoria, las vuellca a archivos ordenados e inmutable y luego fusiona esos archivos. qué cambia??

## Vocabulario inicial: compactación, trabajo pendiente y pausa de escritura

- Compactación: Fusiona archivos ordenados, reduce el solapamiento y elimina versiones obsoletas cuando ya es seguro hacerlo.
- Backlog: Cantidad de bytes o archivos que estan pendientes a procesar.
- Regulación de escrituras: Mecanismo de admisión que retrasa, bloquea o limita nuevas solicitudes para que la cola interna no crezca sin límite.
- **Write stall:** Caso fuerte de regulación: el motor hace esperar a los escritores o deja de admitir temporalmente nuevas escrituras hasta recuperar capacidad.

Se regula la solicitud de escritura: el hilo puede esperar, recibir un retardo, consumirr menos permisos de admisión…

# WAL

## 1. Write again logging

- Llega una solicitud de escritura (crear, actualizar o borrar)
- Ingresa por la NIC como un bitstream o Data frame.
- Si la NIC determina que el destinatario del Data Frame es el nequipo receptor, almcaena vía DMA el Data frame en un buffer de memoria del kernel, y genera interrupción de hardware.
- El driver gestiona la interrupción (approach Top Half-Bottom Half)
- Top Half es la Gard IRQ; reconoce la interrupción y agenda la ejecución del Bottom Half
- El Bottom Half se ejecuta a continuación o vía ksoftirq.

Pasa por la ip

## 2. Escritura el en WAL

- El server escribe la transacción en forma secuencial en un archivo: el log.
- Utiliza la syscall write().
- El kernel normalmente copia el buffer de memoria de usuario que ofrece write() a un buffer en memoria de kernel, con el fin de liberar más rapidamente al proceso escritor (caché de disco)
- En segundo plano el kernel ejeccuta fsync() para aplicar a disco las modificaciones de este cache
- No obstante no queremos riesgos
- .

En este punto la transacción reccibida está salvada

## 3. Actualización en memoria

- Una vez asegurado el log, el cambio se aplica en las estructuras de datos que el motor de db mantiene en memoria de usuario.
- Se trata de una estructura interna de datos del motor, tipicamente llamada Buffer Pool o Caché de Páginas.
- Las bases de datos no manejan en memoria registros sueltos; los agrupan en bloques o páginas de tamaño fijo
- El motor busca en su caché de Páginas la página de datos donde debe ir ese registro (si no la encuentra, la trae del storage con el consiguiente retardo).
- La modifica insertando, actualizando, o borrando la información enviada por el cliente que ya está aplicado en el WAL.
- A partir de ese instante, la “página” en la RAM es marcado por el motor como “Modificada” (Dirty Page).

La copia de la página es más nueva que la versión que está guardada en la db en el storage

Acá se actualiza el modelo de datos organizado exitosamente en la db.

## 4. Respuesta al cliente

Se le confirma al usuario que la red fue exitosa.

## 5. Escritura en disco (flushing/checkpoint)

- Cuando sea oportuno en función de la carga de procesamiento del sistema, asincrónicamente, los datos del caché de páginas en memoria se vuelcamn de manera permanente a la db en el almacenamiento principal.
- El sistema toma esas “páginas sucias” que modificamos en la RAM durante el Paso 3 y ls escribe en el archivo de datos principal del storage

## Recuperación ante fallos

- A partir del paso 2 detallado anteriormente, si el servidor se apaga repentinamente tipicamente por un corte o falla de energía eléctrica, la información que hasta el momento…

## WAL: Alto rendimiento

- a
- a

## WAL: Otros beneficios

- Tolerancia a fallos
- Consistencia
