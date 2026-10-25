# Resumen de Sistemas Distribuidos

> Resumen integrador de las 12 clases de la materia (Ingeniería Informática – Comisión A – Pilar, 2026). Está armado a partir de las presentaciones, los apuntes de cátedra y las notas de clase de un compañero. La idea no es repetir las diapositivas, sino explicar **qué problema resuelve cada concepto, por qué existe, qué cuesta y cuándo conviene (o no) usarlo**.

## Cómo usar este resumen

- Cada **Clase X** es una unidad. Dentro de cada una hay definiciones, explicación del "por qué", tablas comparativas y una sección de **trade-offs / cuándo usar y cuándo no**.
- Al final de cada clase hay un bloque **"Para el parcial"** con lo que hay que poder explicar o hacer.
- Las clases se encadenan: cada una resuelve un problema que la anterior dejó abierto. Leerlas en orden ayuda mucho.

### El hilo conductor de toda la materia

| Clase | Pregunta que responde | Idea en una línea |
| --- | --- | --- |
| 1 | ¿Qué hace "distribuido" a un problema? | No hay visión global: se decide bajo incertidumbre. |
| 2 | ¿Cómo hablo con otro nodo si no sé qué pasó? | Timeouts, retries e idempotencia. |
| 3 | ¿Qué pasa cuando tengo varias copias del dato? | Las réplicas divergen; la consistencia es un contrato. |
| 4 | ¿Cómo logro que todos acuerden un mismo orden? | Consenso (Raft/Paxos) con mayorías. |
| 5 | ¿Cómo hago operaciones compuestas correctas? | Transacciones, serializabilidad, 2PC. |
| 6 | ¿Qué tiene que pasar físicamente para que un "OK" sea verdad? | WAL, MVCC, índices, LSM. |
| 7 | ¿Cómo comparto hechos entre servicios sin esperarlos? | Logs distribuidos, particiones, offsets, outbox, streams. |
| 8 | ¿Cómo junto causalidad, consenso y consistencia en un caso real? | Entrada VIP, PACELC, Dynamo vs. Spanner. |
| 9 | ¿Dónde pongo las fronteras de la arquitectura? | Monolito modular vs. microservicios, sagas. |
| 10 | ¿Cómo veo qué pasa adentro? | Observabilidad: logs, métricas, trazas. |
| 11 | ¿Cómo evito que una falla local se vuelva global? | Timeouts, reintentos, circuit breaker, bulkheads, load shedding. |
| 12 | ¿Cuándo funciona "suficientemente bien"? | SLI/SLO, error budget, chaos, incidentes, postmortems. |

---

# Clase 1 — Pensamiento distribuido

## La idea más importante del curso

> **En un sistema distribuido, ningún observador tiene una visión global, completa, instantánea y confiable del estado del sistema.**

Esto no es un caso raro ni un defecto de implementación: es una consecuencia estructural de tres hechos:

1. Los componentes corren en **máquinas distintas**.
2. Se comunican **solo intercambiando mensajes**.
3. La red introduce **demora, pérdida, reordenamiento e incertidumbre**.

El problema es casi epistemológico: el sistema **no nos entrega conocimiento perfecto sobre su propio estado**. Por eso la materia no busca "adivinar el estado real", sino **garantizar un comportamiento razonable con información incompleta**.

## Qué es un sistema distribuido (y qué vuelve distribuido a un problema)

**Definición operativa:** un sistema distribuido es un conjunto de procesos o nodos que se ejecutan en máquinas distintas, se comunican por mensajes y cooperan para ofrecer una abstracción que el usuario percibe como **un solo sistema**.

Consecuencias directas: la comunicación **no es instantánea**, las fallas son **parciales** (una parte cae y el resto sigue) y el conocimiento del estado global es **limitado**.

No alcanza con decir "hay varias máquinas", "hay red" o "hay muchos usuarios". La definición que importa es:

> Un problema se vuelve distribuido cuando **su corrección depende de coordinar componentes que no comparten memoria, ni reloj global confiable, ni observación perfecta**, y que solo pueden intercambiar mensajes.

Eso cambia la pregunta de diseño. Ya no es *"¿cuál es el estado exacto del sistema?"* sino:

> *"¿Qué puede observar este componente, qué no puede saber, y qué garantía puedo ofrecer bajo esa incertidumbre?"*

## Ejemplo motivador: el pago que no responde

Un usuario hace clic en "pagar", el cliente envía la solicitud y la respuesta nunca llega. ¿Se cobró o no?

**Respuesta correcta: no lo sabemos, y en general no podemos saberlo con certeza en ese momento.**

### Indistinguibilidad observacional

Distintas historias internas producen **exactamente la misma observación** para el cliente:

| | Historia A | Historia B |
| --- | --- | --- |
| ¿Llegó el request? | No, se perdió en la red | Sí |
| ¿Se procesó el pago? | No | Sí, se cobró |
| ¿Qué pasó con la respuesta? | No hubo | Se perdió a la vuelta |
| **Lo que ve el cliente** | **Timeout** | **Timeout** |

Para el cliente, ambas historias son **idénticas**. La ausencia de respuesta **no identifica la causa**. Diseñar sistemas distribuidos es aprender a operar con **evidencia parcial**, no con conocimiento perfecto.

## Un timeout no es una prueba

Error clásico: *"hubo timeout, entonces el servidor se cayó"*. Falso. Un timeout solo significa que **no se observó una respuesta antes de cierto umbral**. Puede deberse a caída del servidor, red lenta, request perdido, respuesta perdida, sobrecarga, ejecución lenta o una combinación.

> **Un timeout es un hecho de observación, no una explicación causal.**

## Modelo síncrono vs. asíncrono

| | Modelo síncrono | Modelo asíncrono |
| --- | --- | --- |
| Cotas de tiempo | Existen límites **conocidos** para procesamiento, red y deriva de relojes | **No hay** cotas confiables |
| Interpretación de una demora | Si se pasa de la cota, se puede tratar como falla | Una demora arbitrariamente larga es **indistinguible** de una falla |
| Realismo | Más cómodo para razonar, poco realista | Más severo, pero se parece a Internet real |

**El curso razona con la intuición asíncrona**: como no puedo distinguir demora de falla, las soluciones tienen que ser robustas a esa ambigüedad.

## El framework del curso: D = (P, F, G, S, T)

Toda decisión técnica se ordena con este esquema. Sirve para **no saltar directo a una tecnología** sin haber modelado el problema.

| Letra | Significado | Pregunta |
| --- | --- | --- |
| **P** | Problema | ¿Cuál es el problema real? |
| **F** | Fallas asumidas | ¿Qué fallas estoy modelando? |
| **G** | Garantía requerida | ¿Qué propiedad tengo que preservar? |
| **S** | Solución elegida | ¿Qué mecanismo uso? |
| **T** | Trade-off aceptado | ¿Qué costo acepto pagar? |

**Ejemplo con el pago:**
- **P:** evitar pagos duplicados en un pago remoto con timeout.
- **F:** pérdida de respuesta, demora, retry del cliente.
- **G:** como mucho un débito efectivo.
- **S:** ID de request único + deduplicación + persistencia del resultado.
- **T:** más complejidad, más estado, más lógica de validación.

La solución no aparece en el vacío: es la respuesta a un problema, a un modelo de fallas y a una garantía.

### Relevar → Analizar → Proponer

Para estructurar y defender una propuesta (habilidad central del ingeniero), primero se separan las tres dimensiones para no mezclar observación, inferencia y decisión; después se conectan:

| Relevar | Analizar | Proponer |
| --- | --- | --- |
| Síntomas observables | Causas compatibles | Garantía buscada |
| Contexto y restricciones | Evidencia disponible | Solución elegida |
| Datos que faltan | Hipótesis descartadas | Trade-off aceptado |

## CAP bien entendido

**Enunciado:** en un sistema que replica estado no se puede garantizar al mismo tiempo:
- **C – Consistencia:** todas las respuestas observan una única versión correcta del estado.
- **A – Disponibilidad:** toda operación recibe respuesta.
- **P – Tolerancia a particiones:** el sistema sigue operando aunque haya nodos incomunicados.

**Partición:** dos partes del sistema siguen vivas pero no pueden comunicarse entre sí de forma confiable. Cada lado puede recibir pedidos y no hay coordinación para decidir globalmente.

**Lectura correcta (no el slogan "elegí 2 de 3"):** las particiones **no se eligen**, son un hecho de la vida. Entonces CAP es una restricción que aparece **cuando hay partición**, y la decisión real es:

| Si priorizo **Consistencia (CP)** | Si priorizo **Disponibilidad (AP)** |
| --- | --- |
| Rechazo (o demoro) operaciones hasta poder coordinar | Respondo igual con lo que tengo |
| Nunca muestro datos incorrectos | Acepto posibles inconsistencias temporales |
| Ej.: saldo bancario, stock de la última unidad | Ej.: likes, carrito, DNS |

> CAP no te dice qué arquitectura usar. Te obliga a **explicitar qué valorás más cuando la coordinación ya no es posible**.

## Seguridad vs. progreso

Dos tipos de propiedades que aparecen en toda la materia:
- **Seguridad (safety):** "nada malo ocurre" (ej.: nunca se cobra dos veces).
- **Progreso (liveness):** "algo bueno eventualmente ocurre" (ej.: el pago termina procesándose).

Muchas soluciones sacrifican progreso temporal para no violar seguridad.

## Errores conceptuales frecuentes

- Creer que timeout implica crash.
- Creer que la red es confiable salvo casos extremos.
- Asumir una visión global perfecta del sistema.
- Tratar CAP como slogan y no como restricción bajo partición.
- Pensar que una garantía importante se obtiene gratis, o que "más tecnología" es "mejor diseño".
- Afirmar "exactly-once", "alta disponibilidad" o "consistencia" sin definir el alcance observable.

> 📝 **Para el parcial:** explicar qué vuelve distribuido a un problema; construir dos historias indistinguibles para un cliente; aplicar D = (P, F, G, S, T) a un caso; explicar CAP como decisión bajo partición; distinguir seguridad de progreso.

---

# Clase 2 — RPC, timeouts, retries e idempotencia

## Comunicar no es instantáneo

> **Una llamada remota no es una función distribuida: es una interacción bajo incertidumbre.**

Entre "quiero invocar algo" y "recibí la respuesta" hay un camino con muchas capas: aplicación → librería RPC / cliente HTTP → serialización → transporte (TCP) → red → balanceadores → servidor → almacenamiento → y todo el camino de vuelta.

Cada capa reacciona distinto ante una falla:
- la aplicación puede abandonar y mostrar error;
- el cliente RPC puede reintentar automáticamente;
- TCP puede retransmitir sin que la aplicación se entere;
- un balanceador puede redirigir a otra instancia;
- **el servidor puede seguir procesando aunque el cliente ya venció su timeout**;
- la base puede confirmar o abortar independientemente de que la respuesta vuelva.

Esto genera una forma práctica de concurrencia: **mientras una capa ya se rindió, otra sigue trabajando**. Para diagnosticar no alcanza con "falló o no falló": hay que reconstruir **qué observó cada capa, qué hizo y qué efecto pudo producir**. Se razona con **líneas de tiempo** (lo que ve el cliente, lo que transporta la red, lo que ejecuta el servidor y lo que confirma el storage pueden no coincidir).

## RPC (Remote Procedure Call)

**Definición:** abstracción que permite invocar una función en otra máquina "como si fuera local".

**El problema:** la abstracción es incompleta. Una llamada local no falla parcialmente y tiene semántica clara; una remota sí puede fallar en cualquier etapa (envío, recepción, ejecución, envío de respuesta, recepción) y depende de la red.

> RPC intenta **ocultar** la red. En un sistema robusto la red **se gestiona**, no se oculta.

## Timeout

**Qué es:** el punto en el que el cliente decide dejar de esperar. Es una **decisión local, basada en tiempo, que introduce incertidumbre**. Al vencer, el cliente puede: seguir esperando, reintentar o informar error.

**Clave:** timeout no implica falla. "Alguien en la cadena no pudo esperar lo suficiente" (y a veces no es el propio cliente sino otra capa). Saber **qué capa reportó el error** es la base del troubleshooting.

**Deadline vs. timeout:** el timeout es "cuánto espero yo"; el *deadline* es un instante absoluto que se propaga en la cadena de llamadas (si al servicio C le llega un request cuyo deadline ya venció, no tiene sentido procesarlo).

| Timeout muy corto | Timeout muy largo |
| --- | --- |
| Falsos positivos: abandono operaciones que iban a terminar | Recursos (hilos, conexiones) bloqueados esperando |
| Más retries → más carga | Mala experiencia de usuario, fallas que se propagan lento |

## Retry (reintento)

**Qué aporta:** recuperación ante **fallas transitorias** (mejora el progreso/liveness).

**Qué introduce:**
- **Duplicación potencial:** no sé si la primera ejecución ocurrió, y reintento igual. Ejemplo: pago duplicado si el primer cobro se hizo y se perdió la respuesta.
- **Presión adicional:** cuando un sistema empieza a fallar, los retries de todos los clientes lo cargan más todavía y lo pueden terminar de tumbar (*retry storm*).

**Cómo reintentar bien:**
- **Backoff exponencial:** esperar cada vez más entre intentos (100 ms, 200 ms, 400 ms…).
- **Jitter:** agregar aleatoriedad para que todos los clientes no reintenten sincronizados.
- **Límite de intentos** y presupuesto de retries.
- **Solo reintentar operaciones seguras** (idempotentes o con clave de idempotencia).

## Idempotencia

**Definición:** una operación es idempotente si **aplicarla varias veces produce el mismo resultado observable** que aplicarla una vez. Formalmente: `op(op(s)) = op(s)`.

**Ejemplos:** `PUT /usuario/5 {nombre: "Ana"}` es idempotente (el estado final es el mismo). `POST /pagos` en general no (cada llamada crea un pago nuevo). "Sumar 100 al saldo" no es idempotente; "fijar saldo en 1100" sí.

**Cómo se implementa cuando la operación no lo es naturalmente:**
1. El cliente genera un **ID de request / clave de idempotencia** estable y lo reenvía en cada reintento.
2. El servidor guarda un **log de operaciones** o una **caché de resultados** por clave.
3. Si llega una clave ya vista, **no re-ejecuta**: devuelve la misma respuesta guardada.

En palabras simples: es la capacidad del sistema de **darse cuenta de que dos mensajes son el mismo** y no procesarlo dos veces.

**Costo:** más estado que guardar (¿cuánto tiempo retengo las claves?), más lógica, y la clave y el efecto deben persistirse **juntos** (ver Clase 6).

## Semánticas de entrega / ejecución

| Semántica | Qué garantiza | Riesgo | Cómo se logra |
| --- | --- | --- | --- |
| **At-most-once** (a lo sumo una vez) | Nunca duplica | Puede **perder** la ejecución | No reintentar |
| **At-least-once** (al menos una vez) | Nunca pierde | Puede **duplicar** | Reintentar hasta tener respuesta |
| **Exactly-once** (exactamente una vez) | Ni pierde ni duplica | Caro, requiere coordinación | En la práctica: **at-least-once + idempotencia** (o deduplicación) |

> Exactly-once **no es magia de la red**: es una garantía lógica que se construye del lado del receptor.

**Cuándo usar cada una:**
- *At-most-once:* métricas o notificaciones donde perder una es tolerable y duplicar molesta.
- *At-least-once:* la mayoría de los sistemas reales, combinada con consumidores idempotentes.
- *Exactly-once efectivo:* pagos, reservas, inventario: todo lo que tenga efecto de negocio irreversible.

## Trade-offs de la clase

| Decisión | Gano | Pago |
| --- | --- | --- |
| Reintentar | Robustez ante fallas transitorias | Duplicados y carga extra |
| Idempotencia | Reintentos seguros | Más estado y complejidad |
| Timeout | No quedarme colgado | Más incertidumbre (¿se hizo o no?) |
| Esperar más (latencia) | Más corrección | Peor experiencia |

> El diseño de una RPC consiste en elegir explícitamente: **semántica de ejecución, política de timeout, política de retry y mecanismo de idempotencia**.

> 📝 **Para el parcial:** dibujar el timeline de una llamada con request o respuesta perdida; explicar por qué un retry puede duplicar; proponer una clave de idempotencia y qué hay que persistir; distinguir las tres semánticas de entrega.

---

# Clase 3 — Consistencia y replicación

## Por qué replicar

**Replicación:** varios nodos mantienen el mismo estado lógico. Se hace por:
- **Disponibilidad:** si un nodo cae, otro responde.
- **Latencia:** leer de una copia cercana.
- **Tolerancia a fallas:** no perder datos por una sola máquina.

**El hecho estructural:** sin coordinación perfecta, **las réplicas divergen**. Por latencias variables, pérdidas y reintentos, escrituras concurrentes y particiones.

> La divergencia **no es un bug**, es una consecuencia del modelo. Lo que hay que decidir es **qué divergencia acepto y cuándo**.

## ¿Qué es "consistencia"?

> Un **modelo de consistencia** define **qué valores pueden observar los clientes y en qué orden**. Es un **contrato** entre el sistema y sus clientes, no una propiedad binaria.

### Linearizabilidad (consistencia fuerte)

- El sistema se comporta **como si hubiera una sola copia** del dato.
- Respeta el **orden de tiempo real**: si una escritura terminó antes de que empiece una lectura, la lectura la ve.
- Intuición: cada operación "ocurre" en un **punto único** entre su invocación y su respuesta (punto de linearización).

**Costo:** requiere coordinación (esperar a otras réplicas) → más latencia y menos disponibilidad bajo partición.

### Causalidad y consistencia causal

**Relación happens-before (a → b)** si:
1. a y b están en el mismo proceso y a ocurre antes que b;
2. a es el envío de un mensaje y b su recepción;
3. por transitividad (a → c y c → b ⇒ a → b).

Si no hay relación en ningún sentido, los eventos son **concurrentes** (a ∥ b).

**Consistencia causal:** si a → b, **todos** los procesos ven a antes que b. No impone un orden total: dos eventos no relacionados pueden verse en distinto orden en distintos nodos. Ejemplo: una respuesta en un foro nunca debe aparecer antes que la pregunta, pero dos posts independientes pueden verse en cualquier orden.

### Consistencia eventual

> Si **dejan de llegar escrituras**, eventualmente todas las réplicas **convergen** al mismo valor.

No garantiza leer lo último ni un orden. Ejemplos: DNS, Git, contadores de likes.

### Comparación

| Modelo | Qué garantiza | Costo | Ejemplo de uso |
| --- | --- | --- | --- |
| Linearizable | Como una sola copia, en tiempo real | Alta latencia, menos disponibilidad | Saldos, locks, líder único, stock |
| Causal | Respeta causa → efecto | Medio: hay que rastrear dependencias | Chats, comentarios, redes sociales |
| Eventual | Convergencia final | Bajo; anomalías temporales | DNS, caches, likes, carritos |

**Trade-off general:** fuerte = coordinación + latencia + menor disponibilidad; débil = disponibilidad + anomalías temporales.

## Tiempo físico vs. orden lógico

No existe un reloj global perfecto. Comparar timestamps de nodos distintos engaña porque:
- los relojes **derivan** y las sincronizaciones (NTP) los mueven hacia adelante o atrás;
- un mensaje enviado antes puede **llegar después**;
- una hora menor **no prueba** causalidad.

La pregunta correcta no es "qué hora marcaba el reloj" sino **"¿qué evento pudo influir sobre qué otro?"**.

### Relojes de Lamport

Cada proceso tiene un contador `L`:
1. Antes de cada evento local: `L = L + 1`.
2. Al enviar un mensaje, adjunta `L`.
3. Al recibir un mensaje con valor `t`: `L = max(L, t) + 1`.

**Propiedad:** si a → b entonces L(a) < L(b).
**Límite:** la inversa **no vale**. L(a) < L(b) no prueba que a causó b. Lamport ordena de forma compatible con la causalidad, pero **no detecta concurrencia**.

### Relojes vectoriales (vector clocks)

Cada proceso mantiene un **vector con un contador por cada proceso** del sistema.
- Evento local: incrementa **su propia** posición.
- Enviar: adjunta su vector.
- Recibir: toma el **máximo componente a componente** y luego incrementa su posición.

**Comparación:**
- V(a) < V(b) si todas las componentes de a son ≤ que las de b y al menos una es estrictamente menor ⇒ **a precede causalmente a b**.
- Si ninguno domina al otro ⇒ **concurrentes**. Ejemplo: (2,1,0) ∥ (1,2,0).

**Para qué sirven:** detectar **conflictos reales** (escrituras concurrentes) y forzar un merge en vez de sobrescribir ciegamente.
**Costo:** el vector crece con la cantidad de nodos/clientes.

| | Lamport | Vectorial |
| --- | --- | --- |
| Tamaño | Un entero | Un entero por proceso |
| a → b ⇒ | L(a) < L(b) | V(a) < V(b) |
| ¿Detecta concurrencia? | No | **Sí** |
| Uso | Orden total compatible con causalidad | Detección de conflictos y versionado |

## Arquitecturas de replicación

### Leader-based (líder único / primario-secundario)

Un **líder serializa todas las escrituras**; los followers replican.

| Replicación síncrona | Replicación asíncrona |
| --- | --- |
| El líder espera a los followers antes de confirmar | El líder confirma y replica después |
| Más consistencia, más latencia | Menos latencia |
| Si un follower se cuelga, se bloquean escrituras | Riesgo de **perder** escrituras confirmadas si el líder cae (failover) y de **lecturas viejas** (*stale reads*) en followers |

**Cuándo:** la opción por defecto para la mayoría de las bases (PostgreSQL, MySQL). Simple de razonar.
**Cuándo no:** escrituras desde muchas regiones con baja latencia, o si no se tolera el líder como cuello de botella.

### Multi-leader

Varios líderes (típicamente uno por región) aceptan escrituras localmente.
- ✅ Escrituras locales rápidas, tolera caída de una región.
- ❌ **Conflictos inevitables** por escrituras concurrentes en distintos líderes; hay que resolverlos.
- **Cuándo:** multi-datacenter, apps offline (cada dispositivo es un "líder"), edición colaborativa.

### Leaderless (estilo Dynamo / Cassandra)

Cualquier nodo acepta lecturas y escrituras. El cliente escribe y lee **en paralelo a varias réplicas**.

Parámetros:
- **N:** réplicas de cada dato.
- **W:** cuántas deben confirmar una escritura (quórum de escritura).
- **R:** cuántas se consultan en una lectura (quórum de lectura).

> **Si R + W > N**, el conjunto que leo y el que escribí **se intersectan en al menos un nodo**, así que alguna réplica consultada tiene la última versión.

Ejemplo: N=3, W=2, R=2 → 4 > 3 ✔. Ajustando W y R se mueve el trade-off: W=N, R=1 favorece lecturas rápidas; W=1, R=N favorece escrituras rápidas.

- ✅ Alta disponibilidad, no hay que elegir líder si un nodo cae.
- ❌ Conflictos por escrituras concurrentes, tendencia a consistencia eventual, más complejidad de reparación.

## Conflictos y convergencia

Dos escrituras concurrentes sobre el mismo objeto generan **versiones divergentes** sin un orden global definido. Formas de resolverlo:

| Estrategia | Cómo funciona | Ventaja | Riesgo |
| --- | --- | --- | --- |
| **LWW** (last-write-wins) | Gana el timestamp más alto | Muy simple | **Pierde datos** silenciosamente; depende de relojes |
| **Merge semántico** | Se combinan según el dominio (ej.: unir carritos) | No pierde información | Requiere lógica de negocio |
| **Versionado (vector clocks)** | Se detecta la concurrencia y se guardan ambas versiones | Detecta conflictos reales | Alguien (app o usuario) tiene que resolver |

## Anti-entropía (reparación de réplicas)

Mecanismos para que réplicas divergentes converjan:
- **Read repair:** al leer de varias réplicas, si una está atrasada se le envía el valor nuevo.
- **Gossip / anti-entropía en segundo plano:** los nodos comparan sus datos periódicamente (usando **árboles de Merkle** para encontrar diferencias sin transferir todo).
- **Hinted handoff:** si una réplica está caída, otro nodo guarda la escritura "en custodia" y se la entrega cuando vuelve.

## D = (P, F, G, S, T) aplicado

- **P:** mantener estado replicado. **F:** demora, pérdida, partición, failover.
- **G:** linearizable / causal / eventual. **S:** líder, multi-líder, quórum.
- **T:** latencia, disponibilidad, complejidad.

> 📝 **Para el parcial:** explicar por qué divergen las réplicas; distinguir los tres modelos de consistencia con ejemplo; aplicar reglas de Lamport y vectoriales y decidir si dos eventos son concurrentes; justificar R + W > N; comparar las tres arquitecturas de replicación.

---

# Clase 4 — Consenso distribuido

## El problema

La Clase 3 mostró que las réplicas divergen. Si necesito que **todos los nodos acuerden un único orden de operaciones** (o un único valor: quién es el líder, si se hizo commit), necesito **consenso**.

**Modelo de la clase:** n procesos, mensajes, red **asíncrona**, fallas por **crash** (el nodo se detiene, no miente). No hay relojes sincronizados ni tiempos máximos de entrega.

## Definición formal

Cada proceso propone un valor. El algoritmo debe garantizar:

| Propiedad | Significado | Tipo |
| --- | --- | --- |
| **Agreement (acuerdo)** | Si dos procesos deciden, deciden **lo mismo** | Seguridad |
| **Validity (validez)** | El valor decidido fue **propuesto** por alguien | Seguridad |
| **Termination (terminación)** | Todo proceso correcto **eventualmente decide** | Progreso |

## Teorema FLP (Fischer, Lynch, Paterson)

> En un sistema **completamente asíncrono** donde **al menos un proceso puede fallar**, **no existe** un algoritmo **determinista** que garantice consenso (en particular, que garantice terminación).

**Intuición:** el problema no es acuerdo ni validez, sino **terminación**. Como una demora es indistinguible de un crash, la red puede demorar justo los mensajes "decisivos" indefinidamente.

**Idea de la prueba:**
- Un estado es **bivalente** si el resultado todavía puede ser 0 o 1, y **univalente** si ya está determinado.
- Existe un estado inicial bivalente.
- Desde un estado bivalente siempre existe una transición que lo mantiene bivalente.
- Un "scheduler adversario" puede elegir siempre esas transiciones ⇒ ejecución infinita sin decidir.

**Cómo se vive con FLP en la práctica:** agregando supuestos: **timeouts**, **aleatorización** (timeouts de elección aleatorios), **sincronía parcial** (la red eventualmente se comporta). Los algoritmos reales **nunca violan seguridad**; lo que puede pasar es que **temporalmente no progresen**.

## Quórums de mayoría

Un **quórum** es un subconjunto de nodos con autoridad para decidir. En consenso: **|Q| > N/2** (mayoría estricta, o sea `⌊N/2⌋ + 1`).

**Propiedad clave: dos mayorías siempre se intersectan.** Si fueran disjuntas, |Q1| + |Q2| ≤ N; pero cada una es > N/2, así que suman > N. Contradicción.

Por qué importa: cualquier decisión tomada por una mayoría **la "recuerda"** al menos un nodo de cualquier mayoría futura.

| Nodos (N) | Mayoría | Fallas toleradas |
| --- | --- | --- |
| 3 | 2 | 1 |
| 4 | 3 | 1 |
| 5 | 3 | 2 |
| 7 | 4 | 3 |

> Para tolerar **f** fallas hacen falta **2f + 1** nodos. Por eso los clústeres suelen ser impares (4 nodos toleran lo mismo que 3, pero cuestan más).

### Split-brain (cerebro dividido)

Si un clúster de 5 se parte en {2} y {3}: el lado de 2 **no alcanza mayoría** → no elige líder ni acepta escrituras. El lado de 3 sí sigue funcionando. Si la regla fuera "cualquier grupo opera", ambos lados escribirían datos contradictorios. La mayoría garantiza que **solo un lado puede operar a la vez** (esto es elegir **C** en CAP).

## Raft: consenso diseñado para entenderse

Divide el problema en tres partes: **elección de líder**, **replicación del log** y **seguridad**.

**Estados de un nodo:**
- **Follower:** pasivo, responde al líder.
- **Candidate:** estado temporal cuando cree que el líder murió.
- **Leader:** recibe todos los pedidos de clientes y coordina la replicación.

**Términos (terms):** el tiempo se divide en períodos numerados y monótonos; cada término tiene a lo sumo un líder. Sirven para detectar líderes viejos.

### Elección de líder

1. Los followers esperan **heartbeats** del líder.
2. Si un follower no recibe nada antes de su **election timeout (aleatorio)**, asume que el líder murió.
3. Pasa a candidate, **incrementa el término**, se vota a sí mismo y pide votos.
4. Si obtiene **mayoría**, es líder. (Los timeouts aleatorios evitan que todos se postulen a la vez y se dividan los votos.)
5. Un nodo **no vota** a un candidato cuyo log está menos actualizado que el suyo.

### Replicación del log

1. El cliente manda el comando al líder.
2. El líder lo agrega a su log.
3. Lo replica a los followers (`AppendEntries`).
4. Cuando una **mayoría** lo guardó, la entrada está **committed** y ya no se puede perder.
5. El líder la aplica a su máquina de estados y responde; los followers la aplican cuando se enteran.

### Invariantes de seguridad

- **Log Matching:** si dos logs tienen una entrada con el mismo índice y término, **todo lo anterior es idéntico** (el líder solo agrega sobre prefijos consistentes; los followers rechazan inconsistencias).
- **Leader Completeness:** si una entrada está committed, **aparece en todos los líderes futuros** (commit requiere mayoría, elección requiere mayoría, se intersectan, y nadie vota a un log más viejo).

## Paxos: el clásico

Propuesto por Leslie Lamport. Se basa en **roles** (un nodo físico puede tener varios):
- **Proposer:** propone un valor, coordina la votación.
- **Acceptor:** vota; es la "memoria" del sistema.
- **Learner:** se entera del valor elegido.

**Fase 1 – Prepare/Promise:** el proposer elige un número de propuesta `n` (único y creciente) y envía `Prepare(n)`. Cada acceptor que no vio uno mayor **promete** no aceptar propuestas menores a `n` y devuelve el valor que ya hubiera aceptado.

**Fase 2 – Accept:** con promesas de una mayoría, el proposer envía `Accept(n, v)`. **Si algún acceptor ya había aceptado un valor, el proposer está obligado a proponer ese** (el de número más alto), no el suyo. Cuando una mayoría acepta, el valor queda elegido.

**Seguridad:** una vez elegido un valor, ningún otro puede elegirse (las nuevas propuestas consultan una mayoría, que intersecta con la que aceptó, y "heredan" el valor).

### Raft vs. Paxos

| | Raft | Paxos |
| --- | --- | --- |
| Enfoque | Líder fuerte, estados explícitos | Roles y rondas numeradas |
| Objetivo de diseño | Ser comprensible e implementable | Mínimo y general (decide un valor) |
| Para un log | Nativo | Requiere Multi-Paxos |
| Uso real | etcd, Consul, CockroachDB | Chubby (Google), Spanner |

## Consenso ⇒ máquina de estados replicada ⇒ linearizabilidad

Si todos los nodos aplican **las mismas operaciones en el mismo orden** sobre una máquina de estados determinista, todos llegan al mismo estado. **Consenso ⇒ orden total ⇒ linearizabilidad.** Es la forma de que un sistema distribuido "se comporte como una sola computadora".

## Costo y cuándo usarlo

**Costo:** varias rondas de mensajes, esperar a la mayoría en cada escritura (latencia), y **menor disponibilidad bajo partición** (el lado minoritario se bloquea).

| ✅ Usar consenso cuando… | ❌ No usarlo cuando… |
| --- | --- |
| Necesitás un único líder, locks o elección de coordinador | La app tolera consistencia eventual (likes, feeds) |
| Configuración/metadata crítica que todos deben ver igual | Un único servidor bien respaldado alcanza |
| Un log de operaciones que no puede divergir (ej.: base distribuida fuertemente consistente) | La latencia de mayoría entre regiones es inaceptable para el caso |

> El consenso **no elimina la incertidumbre: la controla**. Transforma un sistema distribuido en una máquina secuencial lógica, pero no es gratis y no siempre progresa.

> 📝 **Para el parcial:** enunciar las tres propiedades; explicar FLP y cómo se "esquiva"; demostrar que dos mayorías se intersectan; describir elección y replicación en Raft; explicar por qué una partición minoritaria no puede escribir; comparar Raft y Paxos.

---

# Clase 5 — Transacciones distribuidas

## Del orden global a la correctitud lógica

El consenso da un **orden total** de operaciones. Pero falta otra cosa: que las **operaciones compuestas** (varias lecturas y escrituras que juntas forman una acción de negocio) **no violen invariantes** cuando se ejecutan en concurrencia.

## Transacción y ACID

**Transacción:** unidad lógica de trabajo que debe ejecutarse de forma atómica y aislada.

| Propiedad | Qué significa | Qué mecanismo suele sostenerla |
| --- | --- | --- |
| **A**tomicidad | Todo o nada: si un paso falla, se deshace todo | Log (WAL), commit protocol |
| **C**onsistencia | Se pasa de un estado válido a otro; se preservan invariantes | Reglas de la app + restricciones |
| **I**solation (Aislamiento) | Cada transacción se ejecuta "como si estuviera sola" | Locks, MVCC, validación |
| **D**urabilidad | Tras el commit, sobrevive a fallas | WAL durable, réplicas |

## Serializabilidad

> Una ejecución concurrente es **correcta si es equivalente a alguna ejecución secuencial** (una transacción detrás de la otra).

Es el estándar de oro de corrección: el programador puede razonar como si no hubiera concurrencia.

**Conflictos:** dos operaciones de transacciones distintas sobre el mismo dato, donde al menos una escribe: **RW** (read-write), **WR** (write-read), **WW** (write-write).

**Grafo de precedencia:** nodos = transacciones; arista T1 → T2 si una operación de T1 entra en conflicto con una posterior de T2.

> **El schedule es serializable si y solo si el grafo es acíclico.** Si hay ciclo (T1 → T2 → T1), no existe un orden secuencial equivalente.

## Anomalías

| Anomalía | Qué pasa | Ejemplo |
| --- | --- | --- |
| **Dirty read** | T2 lee algo que T1 escribió **sin confirmar**; T1 aborta | Se muestra un saldo que nunca existió |
| **Non-repeatable read** | T1 lee x, T2 lo modifica y confirma, T1 relee y ve otro valor | Un reporte con números inconsistentes entre sí |
| **Phantom** | T1 lee un **conjunto** (ej.: WHERE), T2 inserta una fila que cumple la condición, T1 relee y aparece un "fantasma" | Contar reservas de un día y que cambie el total |
| **Write skew** | Ambas leen el mismo estado y escriben **filas distintas**; ninguna pisa a la otra pero juntas violan un invariante | Ver abajo |

**Ejemplo clásico de write skew (médicos de guardia):** invariante "siempre al menos un médico de guardia". Hay dos. Ambos leen "hay otro médico", ambos se retiran. Cada uno escribió **su propia fila** (no hubo conflicto directo), pero el resultado es **ningún médico**.

## Niveles de aislamiento

| Nivel | Dirty read | Non-repeatable read | Phantom | Write skew |
| --- | --- | --- | --- | --- |
| Read Uncommitted | Posible | Posible | Posible | Posible |
| Read Committed | Evita | Posible | Posible | Posible |
| Repeatable Read | Evita | Evita | Posible* | Posible |
| **Snapshot Isolation** | Evita | Evita | Evita (en lecturas) | **Posible** |
| **Serializable** | Evita | Evita | Evita | Evita |

\* Depende del motor.

**Snapshot Isolation (SI):** cada transacción ve una **foto consistente** tomada al inicio. Evita dirty y non-repeatable reads, y rechaza conflictos write-write directos, **pero no evita write skew** ⇒ **SI no es serializable**.

**Trade-off:** más aislamiento = más corrección, pero más bloqueos, más abortos y menos concurrencia. Por eso muchos motores usan por defecto Read Committed o SI.

**Cuándo exigir Serializable:** cuando hay invariantes que abarcan varias filas (cupos, guardias, saldos entre cuentas). **Cuándo no:** lecturas analíticas o datos donde una anomalía temporal no importa.

## Commit distribuido: 2PC (Two-Phase Commit)

**Problema:** una transacción que toca datos en **varios nodos** tiene que confirmarse **en todos o en ninguno**.

**Fase 1 — Prepare (votación):** el coordinador pregunta a cada participante "¿podés confirmar?". Cada uno deja todo listo de forma durable (estado *prepared*) y vota sí o no.

**Fase 2 — Commit/Abort (decisión):** si **todos** votaron sí ⇒ commit; si **alguno** dijo no ⇒ abort. El coordinador comunica la decisión.

**El gran problema: bloqueo.** Si el coordinador cae después de que los participantes votaron "sí", estos quedan **bloqueados**: no pueden confirmar (quizás otro votó no) ni abortar (quizás el coordinador ya decidió commit). Retienen locks hasta que el coordinador vuelva.

## 3PC (Three-Phase Commit)

Agrega una fase intermedia **pre-commit** para que los participantes sepan que "todos votaron sí" antes del commit final. Reduce el bloqueo, **pero requiere supuestos de sincronía** (cotas de tiempo) que en redes reales con particiones no se cumplen, así que se usa poco.

| | 2PC | 3PC |
| --- | --- | --- |
| Fases | Prepare, Commit | Prepare, Pre-commit, Commit |
| Bloqueo si cae el coordinador | Sí | Reducido |
| Supuesto de red | Asíncrona (pero puede bloquear) | Necesita sincronía parcial |
| Mensajes / latencia | Menos | Más |

**Alternativa en microservicios:** las **sagas** (Clase 8), que cambian atomicidad global por compensaciones.

## Relación con consenso

- Consenso ⇒ **orden global** de operaciones.
- Transacciones ⇒ **consistencia lógica** de operaciones compuestas.

> La serializabilidad garantiza correctitud, pero exige **coordinación fuerte y tiene costo**.

> 📝 **Para el parcial:** armar un grafo de precedencia y decidir si es serializable; identificar la anomalía en un caso; explicar por qué Snapshot Isolation permite write skew y cómo evitarlo; describir 2PC y su problema de bloqueo; comparar 2PC con 3PC.

---

# Clase 6 — Sistemas de almacenamiento (storage)

## Pregunta central

Las clases anteriores preguntaban **qué garantía queremos**. Esta pregunta **qué tiene que ocurrir físicamente** para que esa garantía siga siendo verdad **después de una falla**. Cuando una API devuelve `200 OK` o una base confirma `COMMIT`, hay una cadena de buffers, logs, páginas, versiones e índices detrás.

> El almacenamiento es el lugar donde **las promesas lógicas se convierten en órdenes de escritura, esperas, recuperación y riesgos medibles**.

**Storage distribuido:** nodos y protocolos que convierten operaciones en **estado permanente, recuperable y consultable** aun cuando ocurren fallas. Tiene dos capas:
- **Capa distribuida:** ubica (particiona), replica y decide qué nodo tiene **autoridad** para confirmar cada operación.
- **Motor de almacenamiento local:** traduce operaciones lógicas en bytes, versiones, índices y reglas de recuperación.

### Cómo se conecta con lo anterior

- **Clase 1–2:** si el cliente puede reintentar sin saber si se ejecutó, el servidor necesita **memoria durable** de la clave idempotente, el estado, el efecto y la respuesta, **guardados juntos**. Si la API responde antes de que eso sea durable, un crash puede convertir el siguiente retry en un segundo cobro.
- **Clase 3:** replicar **no es** persistir. Tres copias en RAM se pierden en un corte común de luz.
- **Clase 4:** Raft supone que votos y términos sobreviven reinicios. El consenso decide qué recordar; el storage determina si realmente se puede recordar.
- **Clase 5:** ACID es el contrato; WAL, MVCC e índices son parte de cómo se cumple físicamente.

### Caso conductor: CampusShop

Una estudiante compra la última unidad de un producto (operación con ID estable `9F2A`), recibe "compra confirmada" y un segundo después el nodo pierde energía. Se debe cumplir:
1. La compra existe **exactamente una vez** al reiniciar (→ WAL + idempotencia).
2. La unidad descontada **no reaparece** en el inventario.
3. Un reporte largo ve una **vista coherente** mientras entran compras (→ MVCC).
4. Mostrar las últimas 20 compras **sin recorrer toda la tabla** (→ índices).
5. Una carga sostenida **no crea una cola interna sin límite** (→ backpressure / write stall).

## La frontera de durabilidad

Entre la aplicación y el disco hay capas: buffer del proceso → **page cache del SO** → cache del controlador/dispositivo → medio persistente. Un `write()` puede volver cuando el kernel **solo copió los bytes a su caché**: un corte de luz los borra. Para forzar bajada a disco se usa **`fsync()`**.

Vocabulario que **no** es intercambiable:

| Término | Significa | Ojo |
| --- | --- | --- |
| **Aceptado** | Alguna capa recibió la operación | No dice cuánto sobrevive |
| **Escrito** | Los bytes pasaron a una capa inferior | Puede seguir siendo volátil |
| **Durable / flushed** | Cruzó la frontera de persistencia **para el modelo de falla declarado** | Siempre "durable frente a qué falla" |
| **Committed** | Se tomó la decisión final | Aclarar si la decisión es durable |
| **Aplicado** | La estructura principal refleja el cambio | Puede ocurrir **después** del commit |
| **Visible** | Una lectura puede verlo según su snapshot | Depende del aislamiento |
| **Replicado** | Otra copia lo recibió | ¿En memoria, en disco, en quórum? |

**No hay durabilidad absoluta:** un flush local tolera un crash del proceso pero no que se queme el disco; una mayoría durable tolera perder nodos pero se bloquea en partición. Toda promesa de durabilidad debe venir con:
- **RPO (Recovery Point Objective):** cuánta pérdida de datos reciente es aceptable.
- **RTO (Recovery Time Objective):** cuánto puede tardar la recuperación.

## WAL (Write-Ahead Logging)

**Problema:** escribir y forzar a disco cada página modificada antes de responder es caro (**E/S aleatoria**: páginas dispersas). Y si una transacción toca varias páginas y el sistema cae a mitad, queda una mezcla incoherente.

**Idea:** antes de tocar las páginas, anotar en un **log secuencial durable** (un "cuaderno de evidencia") cómo rehacer el cambio. Escribir al final de un archivo (**E/S secuencial**) es mucho más barato y se puede agrupar.

> **"Primero persisto la evidencia que me permite reconstruir; después materializo las páginas con más libertad."**

### Las dos reglas del WAL

| Regla | Orden | Qué falla evita |
| --- | --- | --- |
| **1. Log antes que página** | Registro WAL → flush durable → recién ahí la página puede ir a disco | Una página nueva en disco sin evidencia para interpretarla o deshacerla |
| **2. COMMIT antes que ACK** | Registro COMMIT → flush durable → ACK al cliente | Decirle "confirmada" al cliente y perder la decisión |

Esto permite la política **no-force**: las páginas **no** se escriben en cada commit; se drenan después en segundo plano (checkpoint).

### Timeline (el que hay que saber dibujar)

1. Llega la solicitud `9F2A`.
2. El motor prepara cambios en memoria.
3. Se escribe el registro WAL (aún en buffer).
4. Las páginas se modifican **en memoria** → quedan **sucias** (*dirty pages*).
5. Se agrega el registro COMMIT.
6. **Flush durable del WAL** hasta el COMMIT.
7. **ACK** al cliente ("confirmada").
8. Crash: se pierde la memoria.
9. **Recovery**: se lee el WAL y se hace **REDO** de lo confirmado que faltaba en las páginas.

> Las notas del compañero agregan el detalle de bajo nivel: el paquete entra por la NIC, se copia vía DMA a memoria del kernel, se atiende la interrupción (top half / bottom half), el servidor hace `write()` del log (que el kernel guarda en su caché) y luego hace falta `fsync()` para que realmente llegue a disco.

### ¿Qué pasa según dónde cae el crash?

| Momento del crash | Evidencia durable | Resultado correcto tras recovery |
| --- | --- | --- |
| Antes del COMMIT durable | Quizás registros de cambio, pero sin decisión | La transacción **no aparece**; se descarta o deshace (UNDO) |
| Después del COMMIT durable, antes de escribir páginas | Cambios + COMMIT en el WAL | **REDO** reconstruye pedido, stock e índice |
| Después de COMMIT y páginas | Todo | Recovery verifica; repetir REDO **no debe** duplicar el efecto |

**REDO debe ser repetible (idempotente):** recovery puede caerse y volver a empezar. Por eso no "suma 1500 otra vez" sino que lleva la página al estado descrito, usando el **LSN** (*Log Sequence Number*, posición en el WAL) guardado en cada página para saber si ya se aplicó.

**Políticas de buffer:** *STEAL/NO-STEAL* (¿puede ir a disco una página con cambios no confirmados?) y *FORCE/NO-FORCE* (¿hay que forzar todas las páginas al commit?). La combinación común **STEAL + NO-FORCE** es la más flexible pero exige poder hacer UNDO y REDO (ARIES es la referencia clásica).

### Group commit y commit asíncrono

- **Group commit:** varios COMMIT comparten **un solo fsync**. Más throughput; cada transacción puede esperar un poco más. **La garantía no se debilita** porque cada ACK sale después de que su COMMIT es durable.
- **Commit asíncrono:** se responde **antes** del flush. Menor latencia, pero hay una ventana en la que una transacción "confirmada" se puede perder. **No es la misma garantía más rápida: es otra garantía** y hay que declararla.

**Lo que WAL NO hace solo:** no da exactly-once de negocio (hace falta clave idempotente + restricción de unicidad + persistir efecto y resultado juntos), ni replicación, ni backups, ni consenso.

### Checkpoints

Si recovery tuviera que releer todo el WAL desde el principio, reiniciar tardaría cada vez más. Un **checkpoint** marca un punto durable desde el cual alcanza con reproducir el WAL.

| Checkpoints frecuentes | Checkpoints espaciados |
| --- | --- |
| Menos WAL que reproducir → **RTO menor** | Operación normal más suave |
| Más E/S y picos de escritura | Más WAL retenido y **recovery más largo** |

**Checkpoint ≠ backup ≠ réplica:** el checkpoint acota cuánto log releer; el backup recupera ante pérdida o corrupción amplia; la réplica mejora disponibilidad pero **también copia una corrupción lógica**. Ninguno reemplaza a los otros.

## MVCC (Multi-Version Concurrency Control)

**Problema:** un reporte tarda 40 segundos mientras entran 900 compras. Sin versiones, o **bloqueo a los escritores** todo ese tiempo, o el reporte **mezcla datos de momentos distintos**.

**Idea:** no borrar la versión vieja al escribir. Cada escritura crea una **versión nueva**; cada lectura usa un **snapshot** que define qué versiones puede ver. El reporte que empezó antes sigue viendo `v1 = 120` aunque ya exista `v2 = 135`; una consulta nueva ve `v2`.

**Regla de visibilidad (simplificada):** una versión es visible para un snapshot si quien la creó ya había confirmado para ese snapshot y su reemplazo todavía no.

**Ventaja:** **lectores y escritores no se bloquean entre sí.**

**Costo ("la historia también cuesta"):** más espacio, páginas menos densas, índices apuntando a versiones muertas, más E/S, necesidad de **garbage collection**. En PostgreSQL se llama **VACUUM**: libera versiones que ningún snapshot activo necesita. **Una transacción muy larga retiene versiones viejas** → *bloat* (tablas e índices inflados) → consultas más lentas.

> **MVCC es un mecanismo, el aislamiento es el contrato.** Con MVCC se puede implementar Read Committed, Snapshot Isolation o Serializable. MVCC **no** garantiza serializabilidad: con Snapshot Isolation sigue existiendo **write skew** (dos supervisores leen "hay dos activos", cada uno desactiva uno distinto, quedan cero). Soluciones: aislamiento serializable, lock explícito sobre un recurso común, reformular el invariante para que haya conflicto detectable, operación atómica especializada.

## Índices y EXPLAIN

**Índice:** estructura **redundante** que organiza claves para responder ciertas preguntas con menos trabajo (un "mapa adicional").

**B-tree:** árbol balanceado cuyos nodos son páginas. Cada página guarda cientos de punteros (**alto fan-out**), así que el árbol es **muy bajo**: millones de claves en 3–4 niveles. Sirve para igualdad, rangos, orden, "siguiente/anterior" y prefijos.

**Costo de cada índice:** encarece INSERT/UPDATE/DELETE, más WAL, más páginas modificadas, más espacio y cache, splits, vacuum. **Crear un índice es cambiar costo de lectura por costo de escritura y mantenimiento.**

### Ejemplo: últimas 20 compras de una estudiante

```sql
SELECT id, total, created_at FROM campus_purchases
WHERE student_id = 42 ORDER BY created_at DESC LIMIT 20;
```

- **Sin índice:** `Seq Scan` de toda la tabla, descarta ~1 millón de filas, ordena, se queda con 20 (≈ 86 ms).
- **Con índice compuesto `(student_id, created_at DESC)`:** `Index Scan` que entrega directo las filas de esa estudiante **ya ordenadas**, y el `LIMIT` corta a las 20 (≈ 0,07 ms).

**El orden de las columnas importa:** el índice sirve para los **prefijos**. `(student_id, created_at)` no es lo mismo que `(created_at, student_id)`.

**EXPLAIN** muestra el plan que eligió el optimizador (con estimaciones). **EXPLAIN ANALYZE** lo **ejecuta** y agrega filas y tiempos reales (cuidado con consultas que modifican datos). **BUFFERS** agrega E/S. Se lee de las hojas a la raíz. Qué mirar: tipo de scan, filas estimadas vs. reales (gran diferencia = estadísticas malas), filas descartadas por filtro, sorts, loops, buffers.

> No preguntes primero "¿cuántos ms tardó?" sino **"¿qué trabajo hizo el motor y era necesario?"**.

**Un seq scan puede ser correcto aunque haya índice:** si la consulta devuelve gran parte de la tabla, muchos accesos dispersos por índice salen más caros que leer todo secuencialmente. El planner **compara costos**, no "obedece" la existencia de un índice.

Variantes: **índice covering** (incluye columnas para no ir a la tabla: *index-only scan*) e **índice parcial** (solo un subconjunto, ej.: compras pendientes).

## LSM-tree (Log-Structured Merge-tree)

**Idea:** convertir muchas escrituras pequeñas y aleatorias en **appends y merges secuenciales**. Ideal para ingestión intensa (telemetría, series temporales, key-value).

**Componentes:** WAL, **memtable** ordenada en RAM, memtables inmutables pendientes de flush, **SSTables** (archivos ordenados e **inmutables** en disco) organizados en niveles, **compaction** en segundo plano.

**Escritura:** append al WAL → insertar en memtable → responder → cuando la memtable se llena, se congela y se vuelca a una SSTable en L0 → compaction la mueve a niveles inferiores. **El camino crítico es corto, pero se genera una deuda de organización que se paga después.**

**Lectura:** buscar en memtable, memtables inmutables, archivos L0 (pueden solaparse) y niveles. Para no leer todo se usan:
- **Bloom filters:** responden "**seguro que no está**" o "**quizás está**". No tienen falsos negativos; los falsos positivos cuestan E/S extra. Útiles para claves inexistentes, poco para rangos.
- Índices por archivo/bloque, block cache, metadata de rangos.

**Tombstones:** como las SSTables son inmutables, un borrado se escribe como una **marca**. No se puede descartar enseguida: si se elimina antes de que desaparezcan las versiones viejas en otros niveles, **el dato borrado "resucita"**.

### Compaction y amplificaciones

**Compaction:** fusiona SSTables, se queda con la versión vigente, propaga tombstones y elimina datos obsoletos. **No es cosmética:** consume CPU, E/S y espacio temporal, y compite con las lecturas y escrituras.

| Métrica | Fórmula / significado | Sube cuando… |
| --- | --- | --- |
| **Write amplification** | bytes físicos escritos / bytes lógicos ingresados | El dato se reescribe en WAL, L0 y cada compaction |
| **Read amplification** | Cuántos archivos/bloques hay que mirar para una lectura | Hay muchos runs solapados |
| **Space amplification** | espacio físico ocupado / datos vivos | Hay versiones viejas, tombstones, espacio temporal |

| Leveled compaction | Tiered / universal compaction |
| --- | --- |
| Niveles poco solapados | Muchos runs fusionados por grupos |
| Mejor lectura y espacio | Menos reescritura temprana |
| Más write amplification | Más read y space amplification |

Ninguna es mejor en absoluto: **redistribuyen el costo**.

### Backlog, backpressure y write stalls

Si entran **90 MB/s** y la compaction procesa **60 MB/s**, el **backlog** (trabajo pendiente) crece sin límite: es una **cola inestable**. El motor no puede "crear" capacidad, solo puede acumular, desacelerar, rechazar o redistribuir.

**Write stall:** el motor **frena o detiene las escrituras** a propósito para que el backlog no crezca sin fin. Es una forma de **backpressure** (regulación de admisión).
- Un stall **puntual** = autoprotección sana.
- Stalls **frecuentes** = la capacidad, la configuración o la carga no están alineadas. Hay que reducir ingreso, aumentar capacidad o cambiar la arquitectura; tunear solo mueve el problema.

Métricas para vigilar: bytes de compaction, pending compaction bytes, cantidad de archivos L0, tiempo en stall, write amplification, p95/p99, utilización del disco, throughput de compaction vs. ingreso.

## B-tree vs. LSM-tree

| Criterio | B-tree | LSM-tree |
| --- | --- | --- |
| Escritura | Actualiza páginas e índices en su lugar; más E/S aleatoria | WAL + memtable; trabajo diferido a flush y compaction |
| Lectura puntual | Pocas páginas; estable si la cache está sana | Puede mirar varios runs; Bloom filters y cache ayudan |
| Rangos | Naturales por el orden del árbol | Eficientes si hay pocos runs y buena compaction |
| Mantenimiento | Splits, vacuum, bloat, reindex | Tombstones, compaction, stalls, amplificaciones |
| Carga típica | OLTP equilibrado o con muchas lecturas | Ingestión sostenida, muchas escrituras |
| Pregunta crítica | ¿Qué consulta justifica cada índice? | ¿Puede la compaction sostener el ritmo de ingreso? |

> "B-tree para leer, LSM para escribir" es una **orientación inicial, no una ley**. Hardware, cache, tamaño de valores y patrón de acceso pueden cambiar el resultado.

**Ejemplos de decisión:**
- **Compras del campus** (transacciones cortas, idempotencia, p99 predecible) → motor transaccional con WAL + MVCC + B-tree (ej.: PostgreSQL). Validar con p99 de checkout, latencia de flush, tiempo de recovery, bloat.
- **Telemetría del campus** (decenas de miles de eventos/s, append, rangos temporales, tolera frescura eventual) → LSM con particionamiento temporal y TTL. Validar con throughput **sostenible**, backlog, amplificaciones, stalls.

## Método para decidir storage

1. **Describir la carga:** proporción lectura/escritura, tamaños, skew, crecimiento, rangos, concurrencia, retención.
2. **Declarar garantías:** durabilidad, RPO, RTO, aislamiento, frescura.
3. **Hipótesis de mecanismo:** WAL, MVCC, B-tree, LSM, particionamiento, replicación.
4. **Identificar el costo desplazado:** flush, vacuum, compaction, espacio, recovery.
5. **Definir métricas:** percentiles, throughput sostenible, amplificaciones, backlog.
6. **Diseñar un experimento que pueda refutar la elección**, no solo mostrar el caso feliz.

> "Usaremos RocksDB porque escribe rápido" **no es una defensa**. La defensa necesita un modelo de carga y un experimento.

## Errores conceptuales frecuentes

- Confundir `write()` exitoso con durabilidad.
- Creer que replicación sustituye a persistencia.
- Pensar que COMMIT implica páginas ya escritas.
- Creer que WAL solo implementa ACID completo, o que MVCC garantiza serializabilidad.
- Crear índices sin mirar consultas ni costo de escritura.
- Tratar compaction como gratis; mirar solo throughput medio e ignorar p99, backlog y recovery.

> 📝 **Para el parcial:** dibujar el timeline del WAL y ubicar el ACK; explicar las dos reglas del WAL y los tres puntos de crash; explicar por qué REDO debe ser repetible; explicar MVCC con un timeline de dos lectores y por qué una transacción larga genera bloat; leer un EXPLAIN antes y después de un índice; describir escritura y lectura en LSM, Bloom filters, tombstones, amplificaciones y write stalls; comparar B-tree y LSM según la carga.

---

# Clase 7 — Logs distribuidos y procesamiento de eventos

## Pregunta central

> ¿Cómo convertir hechos producidos por una aplicación en una **historia durable, ordenada por partes y reprocesable**, sin confundir **publicación, entrega, progreso y efecto de negocio**?

En la Clase 6 el log (WAL) era **evidencia interna** para reconstruir un nodo. Acá el log se vuelve una **historia compartida** que varias aplicaciones leen, cada una a su ritmo.

## Por qué un log: el problema del acoplamiento

En CampusShop, cuando se confirma una compra tienen que reaccionar **inventario, facturación, notificaciones y analítica**.

| Integración sincrónica (llamadas directas) | Integración por log de eventos |
| --- | --- |
| El checkout llama a los 4 servicios y espera | El checkout publica **un hecho durable** y sigue |
| Hereda la latencia del más lento | Solo espera la garantía de publicación |
| Si uno cae, cae la compra | Cada consumidor procesa después, con su propia política de error |
| Simple de razonar | La complejidad se **desplaza** a orden, particiones, progreso, reintentos e idempotencia |

> El log **no elimina** complejidad: la **desplaza**.

## Log vs. cola vs. base de estado

| | Cola de trabajo | Log retenido | Base de estado |
| --- | --- | --- | --- |
| Objeto central | Tarea pendiente | Historia ordenada | Valor vigente |
| Después de leer | Se retira | **Permanece** (según retención) | Se sigue consultando |
| Consumidores | **Compiten** por las tareas | Cada uno lleva **su propio progreso** | Hacen queries |
| Replay (releer) | No es natural | Parte del modelo | Requiere historial aparte |
| Uso típico | Repartir trabajo (ej.: enviar mails) | Integración, streams, auditoría | Servir el estado actual |

**Un log conviene cuando:** varios consumidores necesitan el mismo hecho, aparecen consumidores nuevos que deben reconstruir estado, se necesita auditoría o replay, el productor no debe esperar y el **orden por entidad** importa.
**Costo:** la retención ocupa espacio, los contratos de evento sobreviven a muchos deploys y cada consumidor gobierna su progreso.

## Vocabulario (modelo Kafka)

| Término | Qué es |
| --- | --- |
| **Evento** | Representación de un hecho **que ya ocurrió** (identidad, tipo, datos, tiempo) |
| **Productor / Consumidor** | Quien publica / quien lee y produce efectos |
| **Broker / Cluster** | Servidor que almacena y sirve particiones / conjunto de brokers |
| **Topic** | Categoría lógica de eventos, dividida en particiones |
| **Partición** | Secuencia **append-only ordenada**. Unidad de **orden** y de **paralelismo** |
| **Key** | Dato que decide a qué partición va el evento (y por lo tanto qué eventos comparten orden) |
| **Offset** | Posición de un registro **dentro de una partición** |
| **Consumer group** | Conjunto de instancias que forman una aplicación lógica y se reparten las particiones |
| **Lag** | Distancia entre el final de la partición y el progreso confirmado del consumidor |
| **Rebalance** | Reasignación de particiones cuando cambia el grupo |

### Tres identidades que no se deben confundir

| Identidad | Ejemplo | Para qué sirve |
| --- | --- | --- |
| `operation_id` | `9F2A` | Reconocer un **retry del cliente** como la misma compra |
| `event_id` | `evt-9F2A-1` | **Deduplicar** el procesamiento en consumidores |
| Partición + offset | `P2, offset 418` | Una **posición física**. El mismo offset existe en otras particiones: **no identifica el pedido** |

## Particiones y orden

> **El orden existe solo dentro de cada partición.** Un topic con varias particiones **no tiene orden global**.

Si `OrderCreated → PaymentAuthorized → OrderCancelled` deben verse en orden para un pedido, la key natural es `order_id`: todos sus eventos van a la misma partición.

**La key define dos cosas a la vez:** la unidad de orden **y** dónde se concentra la carga.

| Key | Orden que preserva | Riesgo |
| --- | --- | --- |
| `order_id` | Eventos de un pedido | Bajo |
| `student_id` | Compras de un estudiante | Usuario o bot "caliente" |
| `campus_id` | Toda una sede | Partición masiva |
| Sin key | Ninguno | Pierde relaciones |

**Hot partition:** una partición recibe mucho más que las demás. El promedio del topic puede verse sano mientras esa partición acumula todo el lag → **medir por partición**.

**¿Cuántas particiones?** Cota inicial: `P = ⌈pico / capacidad por consumidor⌉`. Ej.: 80.000 ev/s y 5.000 ev/s por consumidor → 16; con 30% de margen → 21. Más particiones dan más paralelismo pero suman metadata, conexiones, archivos y costo operativo.

## Publicación, réplicas y ACK

Cada partición tiene un **líder** (recibe las escrituras) y **seguidores** (copian). **ISR** (*In-Sync Replicas*) = réplicas suficientemente al día.

| Política de ACK | El broker responde cuando… | Riesgo / costo |
| --- | --- | --- |
| Sin ACK (`acks=0`) | No responde | No sé si llegó |
| ACK del líder (`acks=1`) | El líder lo agregó | Se pierde si el líder cae antes de replicar |
| ACK de réplicas (`acks=all` + `min.insync.replicas`) | Líder + réplicas en ISR lo tienen | Más latencia; si no hay suficientes réplicas, **rechaza** escrituras |

Configuración típica: **RF=3, min.insync.replicas=2, acks=all** → se sigue escribiendo con una réplica caída, pero si queda una sola se **rechaza** antes que confirmar con una única copia (se elige consistencia sobre disponibilidad del productor).

**Productor idempotente:** usa ID de productor + números de secuencia para que el broker descarte lotes reenviados por retries. **No** evita que la aplicación genere dos `event_id` para la misma intención ni que un consumidor repita un pago. Hay que distinguir **idempotencia de transporte** de **idempotencia de negocio**.

## Consumidores, grupos y paralelismo

Dentro de un grupo, **cada partición tiene un solo consumidor activo**. Otro grupo puede leer la misma partición con su propio progreso.

> **Paralelismo activo ≤ min(particiones, consumidores).** Con 3 particiones y 5 consumidores, 2 quedan ociosos. Y si el problema es una sola partición caliente, **agregar consumidores no ayuda**.

**Rebalance:** pausa, revocación, reasignación, restauración de estado local, calentamiento. Un autoscaler agresivo puede causar **oscilación** (agrega consumidores por lag → rebalance baja la capacidad → más lag → agrega más). Se necesita histéresis y ventanas de estabilización.

## Offset: progreso durable, no efecto durable

- **Position:** próximo registro que la instancia viva va a leer (en memoria).
- **Committed offset:** punto **durable** desde el que el grupo reinicia tras un crash.

Si leí hasta 100 pero confirmé hasta 81, tras un crash **releo 81–100**. La pregunta clave de la clase:

> **¿En qué orden se vuelven durables el efecto de negocio y el progreso del consumidor?**

| | At-most-once | At-least-once |
| --- | --- | --- |
| Orden | **Primero** confirmo el offset, **después** hago el efecto | **Primero** hago el efecto, **después** confirmo el offset |
| Si crashea en el medio | El efecto **se pierde** | El evento **reaparece** y el efecto **se repite** |
| Aceptable para | Métricas aproximadas | Casi todo, **si el efecto es idempotente** |

### Inbox idempotente (la solución robusta)

Guardar **en la misma transacción local** el `event_id` procesado y el efecto de negocio:

```sql
BEGIN;
INSERT INTO inbox(event_id) VALUES ('evt-9F2A-1');  -- con restricción UNIQUE
UPDATE inventory SET stock = stock - 1 WHERE product_id = 17;
COMMIT;
```

Si el evento vuelve, el INSERT falla por unicidad y el replay es un no-op. **La atomicidad es esencial:** si marca y efecto se guardan por separado, un crash entre ambos reabre la duplicación.

### "Procesado" significa tres cosas distintas

| Capa | Pregunta |
| --- | --- |
| Entrega | ¿Cuántas veces llegó el registro? |
| Estado interno | ¿Cuántas veces cambió el state store? |
| Efecto de negocio | ¿Cuántas veces pasó **afuera** (pago, mail, stock)? |

Un pipeline puede tener estado interno exactly-once y **mandar dos mails**.

## Exactly-once: siempre dentro de una frontera

En un pipeline **Kafka → Kafka**, una transacción puede coordinar offsets de entrada, cambios de estado y registros de salida: o se ven todos o ninguno. **Pero la garantía termina donde termina la transacción.** Un proveedor de mail, una API bancaria o una base externa **no participan**; para ellos hacen falta claves idempotentes, inbox/outbox o reconciliación.

> "Kafka garantiza exactly-once" es una frase incompleta. Hay que decir **qué resultado, dentro de qué frontera y ante qué fallas**.

## Dual write y Transactional Outbox

**Dual write:** un servicio escribe en su base **y** publica en el broker. No existe un orden que vuelva atómicas dos escrituras en sistemas independientes:
- **Base primero:** si cae antes de publicar → estado sin evento (nadie se entera de la compra).
- **Evento primero:** si la base aborta → evento de algo que nunca pasó.

**Transactional Outbox:** guardar el cambio de negocio **y** el evento pendiente (en una tabla `outbox`) **en la misma transacción local**. Un proceso aparte (**relay**) lee la outbox y publica. Alternativa: **CDC** (*Change Data Capture*), que lee el log transaccional de la base y lo convierte en eventos.

El relay puede publicar, perder el ACK y volver a publicar → **outbox garantiza que la intención no se pierde, pero no evita duplicados**. Por eso el consumidor necesita inbox o idempotencia.

> **Pipeline sólido:** transacción de negocio + outbox → relay repetible → log → inbox + efecto local.

## Procesamiento stateless vs. stateful

- **Stateless:** cada evento se procesa solo (validar, filtrar, proyectar). Fácil de escalar y recuperar.
- **Stateful:** depende de la historia (conteos, rankings, joins, detección de fraude, deduplicación). El estado local (**state store**) acelera, pero debe recuperarse **en forma compatible con el offset** (vía **changelog** y snapshots). Si se restauran de momentos distintos, aparecen omisiones o duplicados. Restaurar un estado de 24 GB a 200 MB/s tarda ~2 minutos: un autoscaler que rebalancea cada minuto nunca llega a régimen.

## El tiempo en streams

Un stream **no termina**, así que para "ventas cada 5 minutos" se necesita una **ventana**.

| Tiempo | Qué es |
| --- | --- |
| **Event time** | Cuándo ocurrió el hecho en la fuente |
| **Ingestion time** | Cuándo lo recibió el sistema |
| **Processing time** | Cuándo lo procesó el operador |

Un celular sin red puede mandar algo 7 minutos tarde: con processing time cae en la ventana equivocada. Si el dominio necesita "cuándo pasó", hay que usar **event time** (de una fuente confiable, no del cliente manipulable).

**Ventanas:** *tumbling* (intervalos fijos sin solapamiento) y *sliding* (se solapan y emiten cada paso).

**Watermark:** señal de "creo que ya llegaron los eventos hasta tal tiempo"; decide cuándo cerrar una ventana. **No** promete que no llegará nada más.

| Watermark largo (espero más) | Watermark corto (espero menos) |
| --- | --- |
| Resultados más completos | Resultados más rápidos |
| Más estado retenido y más latencia | Más correcciones o datos descartados |

**Evento tardío:** llega después del watermark. Opciones: descartarlo (y contarlo), mandarlo a un flujo de tardíos, corregir la salida, reabrir la ventana. Depende de qué cuesta más: esperar o responder incompleto. Ej.: un ranking visible de un juego puede corregirse; la entrega de premios necesita un cierre más fuerte.

## Operación del pipeline

Métricas: tasa por partición, **lag por partición**, **edad del evento más viejo pendiente**, latencia extremo a extremo hasta el efecto, retries, rebalances, tiempo de restore, tamaño de outbox/inbox, duplicados detectados, eventos tardíos.

**DLQ (Dead-Letter Queue):** cuarentena para eventos que no se pueden procesar (*poison pill*), así no bloquean toda la partición. **No debe ser un cementerio silencioso:** necesita dueño, alerta, retención y procedimiento de replay.

**El mismo lag puede tener causas distintas:** ingreso mayor a capacidad (crecen todas las particiones), hot partition, dependencia lenta, rebalances o poison pill. Agregar consumidores solo arregla el primero.

**Cuenta útil:** lag de 600.000 eventos, entran 30.000/s, proceso 50.000/s → me recupero a 20.000/s → **30 s**. Si consumo ≤ ingreso, **nunca** me pongo al día.

**Lag cero no prueba corrección:** puedo estar al día y haber duplicado o perdido efectos.

## Contrato mínimo de un evento

```json
{
  "event_id": "evt-9F2A-1",        // deduplicación
  "aggregate_id": "9F2A",          // agrupa la historia de una entidad
  "event_type": "OrderConfirmed",
  "schema_version": 2,             // evolución explícita
  "occurred_at": "2026-09-10T12:43:18Z",  // event time
  "producer": "campusshop-checkout",
  "trace_id": "tr-77A1",           // causalidad operativa (Clase 10)
  "payload": {"student_id": 8421, "product_id": 17}
}
```

## Cuándo usar (y no) un log de eventos

| ✅ Conviene | ❌ No conviene |
| --- | --- |
| Varios consumidores del mismo hecho | Un único consumidor que necesita respuesta inmediata |
| Desacoplar disponibilidad y latencia del productor | Operaciones que requieren confirmación sincrónica del resultado |
| Replay, auditoría, reconstruir vistas | Equipo sin capacidad de operar brokers, lag, DLQ |
| Ingesta masiva / streaming | Cuando "usaremos Kafka porque escala" es la única justificación |

> 📝 **Para el parcial:** diferenciar WAL local y log distribuido, y log de cola; explicar orden por partición y elección de key; calcular particiones y tiempo de catch-up; dibujar timelines at-most-once y at-least-once; diseñar una inbox; explicar dual write y outbox; distinguir event/ingestion/processing time y el trade-off del watermark.

---

# Clase 8 — Causalidad, consenso y consistencia distribuida (integración)

Esta clase **integra** las Clases 3, 4 y 7 alrededor de un caso y agrega conceptos nuevos: PACELC, distintos tipos de "disponibilidad", Dynamo y Spanner.

## Caso conductor: una entrada VIP y cinco réplicas

Hay **una sola** entrada `VIP-01` (stock = 1). Sofía compra desde Buenos Aires y Tomás desde Madrid, casi al mismo tiempo. Invariante: **reservas confirmadas ≤ 1**.

La dificultad no es guardar dos pedidos. Es **garantizar que una sola transición autoritativa** cambie el estado de `AVAILABLE` a un único dueño.

**Por qué Kafka no alcanza acá:** Kafka **preserva y distribuye** registros, pero no decide cuál de dos comandos incompatibles "gana" sobre un recurso indivisible. Conservar ambas solicitudes no sirve: hay que **elegir una**. Para eso se necesita consenso.

## Pregunta guía

> **¿Qué error es inaceptable para cada operación, y qué costo (latencia, indisponibilidad, coordinación o reparación) estoy dispuesto a pagar para evitarlo?**

| Operación | Error inaceptable | Estrategia razonable |
| --- | --- | --- |
| Entrada VIP / stock único | Vender dos veces | Consenso, rechazar en la minoría |
| Ledger bancario | Perder o duplicar movimientos | Transacciones fuertes |
| Carrito | Perder un ítem agregado | Disponibilidad + merge de versiones |
| Ranking de un juego | Que no se actualice | Eventual, se puede corregir |

## Modelo mínimo de réplica

Una réplica es: un proceso + estado en RAM + estado persistente + interfaz de red. Puede estar **sana y comunicada**, **sana pero aislada** (sigue ejecutando, no coordina) o **caída/pausada**. Un timeout es compatible con todas. Por esa indistinguibilidad existen los IDs idempotentes, los términos, las mayorías, el fencing y los logs durables.

## Causalidad aplicada

(Las reglas de happens-before, Lamport y vectoriales están en la Clase 3.) Lo nuevo:
- **Concurrencia no es simultaneidad física**: significa que ninguna rama tiene evidencia causal de la otra. El envío de Sofía y el de Tomás son concurrentes aunque uno haya sido "antes" en el reloj de pared.
- Lamport da etiquetas compatibles con causalidad; los vectores distinguen posterioridad de concurrencia; **ninguno decide quién se queda con la entrada**.

## Máquina de estados replicada

Una máquina de estados determinista aplica `(estado, comando) → (nuevo estado, respuesta)`. `AVAILABLE + Reserve(Sofía) → RESERVED_BY_SOFIA`; una segunda reserva se **rechaza**. Si todas las réplicas aplican **los mismos comandos en el mismo orden**, convergen.

**Total-order broadcast:** acordar el comando del índice 87, después el 88, el 89… construye un log con **orden total sin huecos**. Un timestamp de Lamport **no alcanza**: etiqueta eventos pero no garantiza que todas las réplicas entreguen el mismo conjunto.

### Estados de una entrada en Raft (no son sinónimos)

> **recibida ≠ agregada ≠ replicada ≠ committed ≠ aplicada ≠ confirmada al cliente**

Ejemplo con 5 nodos: N1 (líder) agrega la entrada → N2 la persiste (2/5, todavía no) → N3 la persiste (3/5 = **mayoría → committed**) → N1 la aplica y responde a Sofía → N2 y N3 la aplican al enterarse.

**Mover el crash:** si N1 cae cuando solo él tenía la entrada, no estaba committed y **puede desaparecer** (Sofía no debe haber recibido confirmación). Si cae después de la mayoría, cualquier mayoría futura la conserva.

### Pausas de GC y fencing tokens

Una pausa larga de *garbage collection* puede hacer que un líder **deje de mandar heartbeats sin estar muerto**. Se elige otro líder con término mayor. Cuando el viejo "despierta", sus mensajes internos se rechazan por **término obsoleto**.

Pero un **recurso externo** (un disco, una API) no conoce los términos de Raft. Para eso se usa un **fencing token**: número creciente que acompaña cada operación; el recurso rechaza cualquier orden con token menor al último visto. Protege contra un **líder zombi**.

### Paxos vs. Raft en un log

Paxos básico decide **un** valor; para un log se repite por posición (*slot*). **Multi-Paxos** estabiliza un líder para muchos slots. Raft habla de término, índice y followers; Multi-Paxos de *ballot*, *slot* y *acceptors*. Mismo objetivo: que dos valores incompatibles nunca queden elegidos para la misma posición.

## Tres logs que no hay que confundir

| Log | Pregunta que responde |
| --- | --- |
| Log de consenso (Raft) | ¿Qué comandos se acordaron y en qué orden autoritativo? |
| WAL local (Clase 6) | ¿Qué puede reconstruir **cada nodo** tras un crash? |
| Log de eventos (Kafka, Clase 7) | ¿Qué hechos pueden leer y reprocesar otros componentes? |

Que algo esté committed en Raft **no elimina** la necesidad de persistirlo bien localmente; publicar un evento **no reemplaza** el acuerdo sobre el estado autoritativo.

## CAP aplicado y tres "disponibilidades"

Sofía ya fue confirmada en {N1, N2, N3} y la red se parte: {N1, N2, N3} | {N4, N5}. Tomás llega a la minoría. Para N4 y N5 hay dos historias indistinguibles: "Sofía ya tiene la entrada pero el mensaje no cruza" y "nadie la compró". Si la minoría responde "éxito" para ser disponible, puede **vender dos veces**. Si espera o rechaza, preserva el invariante y **sacrifica disponibilidad CAP**.

| Concepto | Qué mide |
| --- | --- |
| **Uptime** | Fracción del tiempo en que el proceso/endpoint estuvo levantado y alcanzable |
| **Disponibilidad CAP** | Que toda solicitud a un nodo no caído reciba una respuesta **no errónea** |
| **Disponibilidad semántica** | Cuántas operaciones cumplen la promesa de negocio |

> Un **503 rápido** demuestra que el proceso está vivo (uptime), pero **no** es disponibilidad CAP para la compra.

## PACELC

CAP solo habla de **qué pasa durante una partición**. PACELC agrega la ruta normal:

> **Si hay Partición (P):** elegir entre **A**vailability y **C**onsistency. **Else (E)**, en operación normal: elegir entre **L**atency y **C**onsistency.

Aunque la red funcione, coordinar regiones cuesta. Una lectura local de 15 ms puede estar atrasada; una coordinada de 180 ms da una garantía más fuerte. El dominio decide cuándo vale pagar esa diferencia.

Ejemplos: Dynamo/Cassandra son **PA/EL** (disponibles y rápidos, consistencia eventual); Spanner es **PC/EC** (consistente siempre, paga latencia).

## Recencia, linearizabilidad, serializabilidad y consistencia externa

| Concepto | Qué exige |
| --- | --- |
| **Recencia** | Qué tan nuevo es lo que observo (se mide como antigüedad o lag) |
| **Linearizabilidad** | Cada operación parece atómica entre invocación y respuesta y respeta el **orden real** de operaciones no superpuestas. Habla de **operaciones individuales** |
| **Serializabilidad** | Varias **transacciones** equivalen a **algún** orden serial, **no necesariamente** el temporal |
| **Consistencia externa** | Serializabilidad **+** respeta el orden observado por clientes de afuera (= "strict serializability") |

## Mayoría de consenso vs. quórum estilo Dynamo

Ambos usan intersección, pero **no son lo mismo**. En consenso, la mayoría elige **una única entrada autoritativa** por índice. En Dynamo, `R + W > N` solo asegura que los conjuntos ideales de lectura y escritura se cruzan. **La intersección no crea un líder, ni un orden total, ni linearizabilidad.**

**Tres contraejemplos de "R + W > N ⇒ leo lo último":**
1. **Escrituras concurrentes:** el nodo de la intersección guarda dos versiones hermanas; sabe que hay conflicto pero no inventa un orden.
2. **Sloppy quorum:** si los nodos "dueños" no responden, la escritura se acepta en **nodos alternativos**; una lectura sobre los dueños puede no ver esa escritura hasta el handoff.
3. **Last-write-wins con relojes:** un reloj adelantado hace que una escritura causalmente anterior parezca más nueva y **pise** información válida.

## Version vectors y conflictos fusionables

Mismo criterio de dominancia que los vectores de la Clase 3. Lo importante:
- **Carrito:** `[1,0]` agregó un libro y `[0,1]` agregó auriculares → concurrentes → **se pueden fusionar** (carrito con ambos).
- **Entrada VIP:** `RESERVED_BY_SOFIA` y `RESERVED_BY_TOMAS` → concurrentes → **no hay merge válido**.

> **Detectar concurrencia no implica que exista un merge válido.** Depende del invariante.

## Dynamo: disponibilidad con deuda de reparación

**Flujo:** hash de la key → *preference list* de réplicas → aceptar con W respuestas → guardar versiones concurrentes si no puede ordenarlas → **sloppy quorum** si los dueños no responden → guardar un **hint** → **hinted handoff** cuando el dueño vuelve → **read repair** y **anti-entropía** (árboles de Merkle) para el resto.

**Deuda de reparación:** si se generan divergencias más rápido de lo que se reparan, la divergencia acumulada **crece**. "Eventual" **no significa que se arregla solo**: hay que medir tasa de divergencia, capacidad de reparación, edad máxima y un objetivo de convergencia.

**Costos:** versiones hermanas, hints pendientes, espacio, red e I/O de reparación, frescura variable y lógica de merge en la aplicación.

**Cuándo usarlo:** cuando el dominio admite reconciliación y prioriza seguir respondiendo (carritos, sesiones, preferencias). **Cuándo no:** recursos indivisibles o invariantes duros.

## Spanner y TrueTime: transacciones globales con orden externo

**Problema:** una compra toca el grupo de entradas, el de pagos y el de derechos del usuario, en distintas regiones; y si un cliente ya vio la confirmación en Buenos Aires, una operación posterior en Madrid tiene que "verla". Hace falta transacción multi-grupo **y** orden compatible con el tiempo real.

- **Consenso localizado:** los datos se dividen en grupos replicados con Paxos. Una transacción coordina **solo los grupos que toca** (con 2PC entre ellos), no una votación planetaria.
- **TrueTime:** en vez de un instante "exacto", devuelve un **intervalo** `[earliest, latest]` donde se sabe que está el tiempo real (gracias a GPS y relojes atómicos; el ancho refleja error de sincronización y deriva).
- **Commit wait:** el coordinador elige un timestamp `s` para el commit y **espera** hasta que `TT.now().earliest > s`, es decir, hasta estar **seguro** de que `s` ya pasó en todo el mundo. Recién ahí confirma.
- **Resultado — consistencia externa:** si T1 confirmó antes de que empiece T2, entonces `s(T1) < s(T2)` en cualquier serialización.

> TrueTime **no elimina** la incertidumbre: la **acota** y la convierte en una espera explícita. Cuanto más ancho el intervalo, más se espera.

| | Dynamo | Spanner |
| --- | --- | --- |
| Prioridad | Seguir respondiendo | Corrección global |
| Ante conflicto | Guarda versiones y repara después | Lo evita con consenso y locks |
| Costo principal | Deuda de reparación, merges | Latencia (consenso + commit wait), infraestructura de tiempo |
| PACELC | PA/EL | PC/EC |

## Mapa: qué pregunta responde cada mecanismo

| Mecanismo | Pregunta |
| --- | --- |
| Kafka | ¿Qué registros se preservan, particionan y releen? |
| Happens-before / Lamport / vectores | ¿Qué evento pudo influir en otro? ¿Es posterior o concurrente? |
| Raft / Paxos | ¿Qué comando se vuelve decisión autoritativa? |
| WAL | ¿Qué recuerda cada nodo al reiniciar? |
| CAP | ¿Qué sacrifico durante una partición? |
| PACELC | ¿Qué pago normalmente por coordinar? |
| Quórums Dynamo | ¿Con cuántas réplicas completo una operación? ¿Cómo reparo? |
| Spanner / TrueTime | ¿Cómo sostengo transacciones globales con orden externo? |

## Errores frecuentes

| Error | Corrección |
| --- | --- |
| "Timeout = servidor caído" | Armar al menos 4 historias compatibles |
| "Lamport menor implica causa" | Dos procesos sin mensajes pueden tener etiquetas ordenadas sin relación |
| "Tres copias = commit" | Exigir término, índice, regla de mayoría |
| "Un rechazo rápido es disponibilidad CAP" | Separar uptime de respuesta válida |
| "R + W > N ⇒ linearizable" | Probar escrituras concurrentes y sloppy quorum |
| "Eventual se arregla solo" | Medir tasa de divergencia y capacidad de reparación |
| "TrueTime elimina la incertidumbre" | Mostrar el intervalo y cómo crece la espera |

> 📝 **Para el parcial:** razonar el caso VIP con particiones; distinguir los estados de una entrada en Raft; explicar fencing tokens; diferenciar uptime / disponibilidad CAP / semántica; explicar PACELC; diferenciar linearizabilidad, serializabilidad y consistencia externa; explicar por qué R + W > N no da linearizabilidad; decidir si un conflicto es fusionable; explicar sloppy quorum, hinted handoff y TrueTime con commit wait.

---

# Clase 9 — Fronteras de arquitectura (monolito, microservicios y sagas)

## Pregunta central

> **¿Qué beneficio concreto obtiene una separación arquitectónica y qué nueva falla, latencia, duplicación de datos o responsabilidad operativa introduce?**

La clase no defiende monolitos ni microservicios "por moda". Enseña a identificar una **necesidad observable**, comparar una solución simple con una distribuida y **exigir evidencia** de que el costo extra de distribuir se justificó.

> Una arquitectura no es un diagrama de cajas: es una **distribución de autoridad, datos, fallas, comunicación y responsabilidad operativa**.

## Caso conductor: Asteria Online

En un juego multijugador, Sofía compra la pieza única `EspadaSolar#17`. Intervienen **Mercado** (oferta), **Inventario** (propiedad), **Pagos** (cobro) y después **Ranking** y **Notificaciones**.

| Reglas que **no** pueden romperse | Efectos que **pueden** demorarse |
| --- | --- |
| La espada tiene como máximo **un** propietario | Actualizar el ranking |
| El mismo intento no se cobra dos veces | Notificar al jugador |
| Una compra confirmada no desaparece tras un reinicio | Analítica, historial derivado |

Esta diferencia ya anticipa la decisión clave: **no todo trabajo pertenece al camino sincrónico** de la compra.

## Vocabulario fundamental

| Concepto | Definición | En Asteria |
| --- | --- | --- |
| **Dominio** | Parte de la realidad con reglas y lenguaje propios | Comercio de objetos |
| **Capacidad** | Responsabilidad estable del negocio (no un endpoint): agrupa reglas, datos y operaciones | "Gestionar la propiedad de objetos" |
| **Invariante** | Regla que debe ser verdadera en todo estado válido | propietarios(#17) ≤ 1 |
| **Frontera** | Límite explícito entre responsabilidades, reglas y datos, con "puertas" permitidas | Mercado no toca la tabla de propietarios |
| **Dato autoritativo** | El registro que es "la verdad oficial" para decidir | `owner(#17)` en Inventario |
| **Autoridad** | Componente con la última palabra sobre un hecho | Inventario decide el dueño; Pagos, el cobro |
| **Responsabilidad** | El equipo que cambia, opera, observa y recupera esa parte | Quién atiende un objeto duplicado |
| **Autonomía** | Cuánto puede una parte cambiar, publicarse, escalar o recuperarse **sin arrastrar al resto** | ¿Ranking puede deployar sin re-deployar Inventario? |

> **Separar procesos no crea autonomía automáticamente.** Una base compartida, contratos frágiles, versiones coordinadas o un equipo sin capacidad operativa conservan el acoplamiento.

**Vuelta atrás (rollback) de versión ≠ deshacer el negocio:** volver a un binario anterior **no deshace** pagos, eventos ni transferencias ya confirmadas. Eso requiere **compensaciones**.

## De regla local a protocolo distribuido

Si una sola autoridad tiene el dato, protege el invariante con **una transacción local** (leer, verificar disponibilidad, reservar, commit). Cuando la decisión cruza procesos, aparece un **protocolo**: mensajes, estados durables, IDs, timeouts, reintentos y reglas de recuperación.

> **Distribuir no elimina complejidad: cambia complejidad local por coordinación explícita, observabilidad y recuperación entre procesos.**

## Módulo, proceso, servicio, microservicio

| Término | Qué es |
| --- | --- |
| **Módulo** | Frontera **de código** dentro de una aplicación (interfaz interna, oculta detalles) |
| **Proceso** | Programa en ejecución con memoria, recursos y fallas propias |
| **Servicio** | Capacidad accesible mediante un **contrato**, normalmente por red o mensajería |
| **Microservicio** | Servicio enfocado en **una capacidad coherente**, con datos y responsabilidad claros e **independencia razonable** de cambio y operación |

Exponer una clase por HTTP **no** la convierte en microservicio.

## Monolito modular: una opción válida

Se publica como **una sola unidad**, pero internamente tiene **módulos con fronteras y dependencias explícitas**. Analogía: un edificio (la app) con habitaciones (módulos), puertas (APIs internas) y paredes (nadie entra a las tablas ajenas). Los "caminos entre edificios" serían la red.

| ✅ Ventajas | ⚠️ Riesgos |
| --- | --- |
| Llamadas locales rápidas y simples | Dependencias internas sin control |
| **Transacciones locales** para invariantes críticos | Acceso directo a tablas de otros módulos |
| Debugging y pruebas end-to-end más fáciles | Builds y tests cada vez más lentos |
| Un solo deploy y un solo rollback | Responsabilidad difusa ("todos cambian todo") |
| Menos superficie operativa | |
| Fácil mover fronteras mientras el dominio cambia | |

> Un monolito modular **no es una etapa vergonzosa**. Puede ser la solución correcta mientras la modularidad interna alcance.

## ¿Cuándo se justifica separar?

Hace falta una **necesidad concreta y medible**, no una "presión" difusa:

| Necesidad | Síntoma | Evidencia |
| --- | --- | --- |
| Carga diferente | Ranking consume CPU y sube el p99 de la compra | CPU, colas, p99 |
| Ritmos de cambio distintos | Ranking espera el deploy de todo Asteria | Tiempo hasta producción |
| Aislamiento de fallas | Un módulo secundario bloquea el checkout | Alcance del incidente |
| Seguridad / auditoría | Pagos necesita controles específicos | Auditorías, incidentes |
| Equipos | Todos dependen de todos | Tickets, bloqueos, versiones coordinadas |

**Antes de crear un servicio, probar lo simple:** mejor modularidad, índices, caché, colas, límites de recursos o un worker separado.

**Ejemplo de buena primera frontera — separar Ranking:** durante un torneo, Ranking satura CPU y base compartidas y sube el p99 de compras. Ranking ya trabaja "después" de la compra. Separarlo y alimentarlo con eventos `PurchaseConfirmed` (vía outbox) le da escala y deploy propios. **Costo aceptado:** entrega de eventos, vista atrasada, duplicados, otra guardia.

## Contextos delimitados (Bounded Contexts)

Ámbito donde las palabras y reglas tienen **un significado coherente**. "Objeto" significa cosas distintas según el contexto:
- **Catálogo:** tipo de pieza, rareza, atributos base.
- **Inventario:** instancia concreta, propietario, versión.
- **Mercado:** oferta, vendedor, precio.
- **Combate:** arma equipada, daño, durabilidad.

Forzar un "objeto universal" genera campos opcionales, permisos confusos y deploys coordinados. Mejor **traducir entre modelos** con contratos.

> Un bounded context **no obliga** a un microservicio: **la frontera semántica precede a la frontera de red**.

## Datos: database per service y consultas globales

**Database per service** no significa "un servidor por servicio", sino que **solo la autoridad modifica directamente su modelo**. Los demás consultan por API, consumen eventos o mantienen proyecciones.

| Alternativa para consulta global | Ventaja | Costo |
| --- | --- | --- |
| Composición de APIs | Datos frescos | Depende de que respondan todos |
| Vista materializada por eventos | Rápida y disponible | Puede estar atrasada |
| Almacén analítico | Ideal para reporting | No sirve para decisiones transaccionales |
| Índice de búsqueda | Optimizado para leer | Hay que reconstruirlo desde la fuente |

**Réplicas y cachés regionales** acercan lecturas (Sofía ve la oferta más rápido), pero **una oferta visible en caché no garantiza disponibilidad**: la reserva autoritativa decide.

## Sincrónico vs. asincrónico y el costo de distribuir

| Cadena sincrónica `A → B → C → D` | Continuación asincrónica `A → log ⇒ {C, D, E}` |
| --- | --- |
| A no termina hasta que responden todos | A no espera a los lectores |
| Latencias se **suman**, timeouts ambiguos | Atraso medible, duplicados posibles |
| Para pasos que **necesitan** el resultado (autorizar el pago) | Para ranking, notificaciones, analítica |

**Comando, evento y consulta:**
- **Comando:** pide a una autoridad que **intente** cambiar estado (`ReserveItem`). Puede ser rechazado.
- **Evento:** afirma un hecho **ya confirmado** (`ItemReserved`).
- **Consulta:** pide información sin modificar (`GetOwner`).

Un evento **no debería ser un comando disfrazado**.

**Lo que agrega pasar una llamada local a la red:** serialización y compatibilidad, latencia y variabilidad, timeouts y respuestas perdidas, reintentos e idempotencia, autenticación y secretos, trazas, deploys independientes, guardias, consistencia entre datos separados.

### Disponibilidad compuesta

Si un flujo depende **obligatoriamente** de 4 componentes con 99,9% cada uno: `0,999⁴ ≈ 0,996` (99,6%). Es optimista (supone independencia), pero muestra que **más procesos no implican más disponibilidad**: cada dependencia obligatoria puede bloquear el camino crítico. El objetivo es **sacar dependencias no críticas del camino** o permitir degradación.

## Antipatrón: el monolito distribuido

Varios procesos y red, pero se comporta como **una sola unidad inseparable**: misma base compartida, versiones que se publican juntas, cadenas RPC largas obligatorias, cambios simples que requieren varios equipos, una falla local que frena todo. **Se paga la latencia, las fallas parciales y la observabilidad sin obtener autonomía.** Un monolito modular sería más simple.

## RPC entre servicios, dual write y outbox

- **Timeout entre servicios:** si Mercado pide `Reserve(#17, Sofía)` y la respuesta se pierde, Mercado no sabe si se reservó. Solución (Clase 2): `request_id` estable; Inventario guarda durablemente identidad + efecto + resultado y ante un reintento **devuelve el mismo resultado**.
- **Dual write y transactional outbox:** idéntico a la Clase 7. Inventario actualiza el dueño **y** inserta `OwnershipTransferred` en la tabla `outbox` en el mismo COMMIT; un relay publica (al menos una vez). **Resuelve:** no hay estado sin intención de publicar. **Deja abierto:** duplicados del relay, atraso, limpieza de la tabla, orden entre entidades, consumidores no idempotentes.

**Contrato de evento:** identidad, significado, esquema, evolución, orden relevante y política de entrega. Se parece a una API, pero **queda almacenado**: consumidores nuevos pueden leer eventos de versiones viejas, así que la evolución del esquema importa más.

## Sagas

**Problema:** la compra atraviesa **tres autoridades independientes** (Mercado, Inventario, Pagos), cada una con su base. **No hay un commit global** (y 2PC entre servicios bloquea y acopla, Clase 5).

> **Saga:** proceso de negocio **durable** compuesto por **transacciones locales** coordinadas con comandos y eventos. Registra dónde quedó, qué respuesta recibió y qué paso puede ejecutar, reintentar, **compensar** o escalar a revisión manual.

**Estados felices:** `CREATED → RESERVED → PAYMENT_AUTHORIZED → OWNERSHIP_TRANSFERRED → COMPLETED`

**Estados inciertos (hay que modelarlos):** `PAYMENT_UNKNOWN`, `TRANSFER_PENDING`, `COMPENSATING`, `MANUAL_REVIEW`.

### Rollback vs. vuelta atrás vs. compensación

| Mecanismo | Qué hace |
| --- | --- |
| **Rollback transaccional** | Aborta cambios locales **aún no confirmados** |
| **Vuelta atrás de versión** | Devuelve tráfico/software a una versión anterior |
| **Compensación** | **Nueva acción de negocio** que repara consecuencias **ya confirmadas** |

Si Pagos ya cobró y la transferencia falla, la historia no se borra: se **reembolsa**, se libera la reserva, se restaura la oferta y, si no se puede, se escala a revisión manual. **Una compensación puede fallar y debe ser idempotente: no es un rollback perfecto.**

**Trade-off de la saga:** se gana disponibilidad y autonomía (sin locks globales), pero se pierde **aislamiento**: otros pueden ver estados intermedios (ej.: la espada "reservada" mientras se cobra).

### Coreografía vs. orquestación

| | **Coreografía** | **Orquestación** |
| --- | --- | --- |
| Idea | Cada servicio reacciona a eventos y emite el siguiente (como una danza) | Un **coordinador durable** guarda el estado y envía comandos |
| Flujo | `ItemReserved ⇒ PaymentAuthorized ⇒ OwnershipTransferred ⇒ PurchaseCompleted` | Orquestador → Inventario, Pagos, Mercado |
| ✅ Ventajas | Bajo acoplamiento, autonomía local | Flujo visible, timeouts, reintentos y compensaciones centralizados |
| ⚠️ Riesgos | Difícil de visualizar, ciclos, compensaciones dispersas | Concentra conocimiento; otro componente crítico que operar |
| Cuándo | Flujos cortos y estables | Pagos inciertos, compensaciones complejas, revisión humana |

## Migración gradual: Strangler Fig

Lo viejo y lo nuevo **coexisten** mientras un router desvía tráfico de a poco: **coexistir → comparar → desviar → retirar**.

Para Ranking: alimentar ambos cálculos con los mismos eventos, comparar exactitud/atraso/costo, mandar una fracción de lecturas al nuevo (**canary**), ampliar solo si las métricas cumplen, mantener la posibilidad de volver, y retirar lo viejo con evidencia. **Es reversible mientras ambos caminos puedan recibir tráfico.**

## Preparación operativa (readiness)

Separar implica **operar otro sistema**. Checklist mínima: CI y publicación de versiones, rollback **probado**, permisos y secretos, logs/métricas/trazas, timeouts/reintentos/idempotencia, **SLO y alertas**, **guardia (on-call) y runbooks**, backup y recuperación, pruebas de falla, capacidad y costos.

- **CI:** integrar seguido con build y tests automáticos.
- **CD** es ambiguo: *entrega continua* (siempre listo, promoción manual) vs. *despliegue continuo* (todo lo que pasa los controles va solo a producción).
- **SLI / SLO:** indicador medido y meta para ese indicador (se profundiza en la Clase 12).

**Métricas que demuestran autonomía** (con línea base previa): tiempo hasta producción, deploys sin coordinación, escala independiente, alcance de incidentes, p99 y errores, coordinaciones entre equipos.

## Método para decidir una frontera

1. **Problema y evidencia:** ¿qué necesidad persiste y cómo se mide?
2. **Regla, dato y autoridad:** ¿qué invariante protege la frontera y quién decide?
3. **Alternativa simple:** ¿por qué no alcanza mejorar la modularidad interna?
4. **Costo, falla y mecanismo:** ¿qué agrega la red y cómo se recupera?
5. **Métrica y revisión:** ¿qué resultado justificaría seguir o volver atrás?

## Errores frecuentes

| Error | Corrección |
| --- | --- |
| "Monolito = código desordenado" | Separar forma de deploy de calidad modular |
| "HTTP convierte una clase en microservicio" | Exigir capacidad, dato, autoridad y responsabilidad |
| "Más servicios = más disponibilidad" | Dibujar la cadena sincrónica y mover una falla |
| "Outbox publica una sola vez" | Mover el crash entre publicar y marcar la fila |
| "Saga = transacción distribuida" | Mostrar estados intermedios y compensaciones que fallan |
| "Rollback borra el pasado" | Distinguir versión, rollback local y compensación |
| "Separar crea autonomía" | Medir deploys independientes, base compartida, guardias |

> 📝 **Para el parcial:** definir capacidad, invariante, autoridad y autonomía; argumentar cuándo un monolito modular alcanza y cuándo separar; calcular disponibilidad compuesta; identificar un monolito distribuido; diseñar una saga con estados inciertos y compensaciones; comparar coreografía y orquestación; explicar Strangler Fig.

---

# Clase 10 — Observabilidad distribuida

## Por qué el diagnóstico "clásico" deja de funcionar

En una aplicación en **una sola máquina** el diagnóstico tiene garantías: un **stack trace completo**, un **reloj único** que ordena los eventos del log y llamadas en memoria de nanosegundos. Al distribuir, eso colapsa por cuatro razones:

1. **El stack trace se corta en la red.** El llamante solo ve un `500` o un `DeadlineExceeded`; no sabe qué falló cuatro saltos más abajo.
2. **El reloj ya no ordena.** Por *clock drift* (Clase 3), si mezclás logs de varios nodos por timestamp, una recepción puede aparecer **antes** que su envío.
3. **El tiempo "fuera del código" es invisible.** Mucha latencia ocurre esperando en colas (sockets, balanceadores, pools); el log local solo mide desde que el worker tomó el pedido.
4. **Fallas combinatorias y latencia de cola.** Las degradaciones rara vez son "up/down". Ejemplo del **"cliente elefante"**: un cliente corporativo con carritos de 30.000 ítems agrega micro-demoras en cada servicio; **todos los dashboards están en verde**, pero la suma supera el timeout del API Gateway y el usuario ve una falla total.

> La observabilidad **no es un agregado del final**: es una **propiedad arquitectónica** del sistema.

## ¿Por qué monitorear?

- **Detección temprana:** disco llenándose, certificado por vencer, degradación lenta (antes de que se quejen los usuarios).
- **Diagnóstico:** reconstruir qué pasó y por qué, no solo constatar que falló.
- **Decisiones informadas:** ¿agregar un nodo de cache mejoró el hit rate lo suficiente para justificar el costo?
- **Contención de daño:** detectar la degradación para activar circuit breakers, degradación o rate limiting (Clase 11).
- **Aprendizaje continuo:** alimenta postmortems y experimentos de caos (Clase 12).

## Monitoreo vs. observabilidad

**Definición (de la teoría de control, Kálmán 1960):** un sistema es **observable** si su estado interno puede inferirse a partir de sus salidas externas.

**En software:** un sistema es observable si permite **investigar y explicar cualquier estado**, incluso los **imprevistos** ("unknown-unknowns"), **haciendo preguntas nuevas sin tener que deployar código nuevo** para agregar logs.

| Monitoreo tradicional | Observabilidad |
| --- | --- |
| Responde preguntas **conocidas de antemano** (dashboards fijos) | Permite preguntas **nuevas** sobre fallas nunca vistas |
| "¿Está arriba el servicio?" | "¿Por qué este subconjunto de requests está lento?" |

Un buen monitor tiene que responder dos preguntas: **¿qué está roto?** ("pagos tiene p99 > 2 s", no "el sistema está lento") y **¿por qué?** (¿CPU? ¿dependencia caída? ¿deploy reciente? ¿un cliente saturando?).

## De los "tres pilares" a la "trenza única de datos"

Tradicionalmente se habla de **logs, métricas y trazas** como tres pilares. Si se implementan como **silos separados**, aparece el antipatrón de las **"tres pestañas del navegador"**: una con métricas (Grafana), otra con logs (ELK) y otra con trazas (Jaeger). En un incidente, el operador **correlaciona a mano**, copiando IDs y alineando horarios a ojo.

**Trenza única (*single braid*):** cada señal mantiene su propósito, pero todas están **entrelazadas por contexto compartido** (vía OpenTelemetry), así las herramientas correlacionan automáticamente.

### Hard context vs. soft context

| | Soft context (blando) | Hard context (duro) |
| --- | --- | --- |
| Qué es | Atributos que acompañan la medición: cliente, host, zona, timestamp | Enlaces **explícitos** inyectados por protocolo: `trace_id`, `span_id`, `parent_span_id` |
| Para qué | Definir **dimensiones** para cortar y agrupar ("el 98% de los fallos está en `us-east-1a`") | Definir la **forma causal** del sistema: qué se relaciona con qué |
| Debilidad | El más usado (el tiempo) es el más débil para correlacionar | Si un servicio no propaga las cabeceras, la cadena se corta |

> El hard context define la **forma** del sistema; el soft context, las **dimensiones** para explicarlo. Lo frágil es tener **soft sin hard**.

### OpenTelemetry (OTel)

- **Un único SDK** en la aplicación contra una API neutral (cambiar de proveedor no obliga a reescribir instrumentación).
- **OTLP** (OpenTelemetry Protocol): un solo protocolo para métricas, logs y trazas, en lotes (transporte recomendado `http/protobuf`; puertos 4317 gRPC y 4318 HTTP).
- **OTel Collector:** proceso intermedio (sidecar o agente por nodo) que recibe OTLP, filtra, muestrea y envía a varios backends sin tocar el código.

**Correlación automática:**
1. El `trace_id`/`span_id` activo vive en el contexto del hilo/corrutina.
2. **Log ↔ traza:** el SDK estampa `trace_id` y `span_id` en cada log, permitiendo un "JOIN" exacto entre un span y sus logs.
3. **Métrica ↔ traza (exemplars):** un pico del p99 en el histograma tiene enlazado el `trace_id` de una request concreta; un clic abre la traza culpable.
4. **Análisis diferencial (ej.: BubbleUp):** compara automáticamente todos los atributos entre requests que fallaron y que no, y señala la dimensión que más difiere.

## Las tres señales

| Señal | Pregunta que responde | Fortaleza | Cuidado |
| --- | --- | --- | --- |
| **Métricas** | ¿Cuál es la tendencia o salud agregada? (**qué** está roto) | Baratas, ideales para alertar | Sin alta cardinalidad |
| **Trazas** | ¿Cómo atravesó esta request los servicios? (conecta **qué** con **por qué**) | Muestran dónde se fue el tiempo | Volumen → sampling |
| **Logs / eventos estructurados** | ¿Qué pasó en este instante, con qué detalle? (**por qué**) | Detalle fino de dominio | Deben ser estructurados |

### Logs: de texto libre a eventos estructurados

Un log de texto libre (`"User 42 logged in from 10.0.0.1"`) es legible pero obliga a regex frágiles. En sistemas distribuidos se usan **logs estructurados (JSON)** con: timestamp UTC de alta precisión, `service_name` y versión, **`trace_id` y `span_id`**, severidad y atributos de dominio.

**Evento estructurado ancho (*wide event*):** en lugar de muchas líneas sueltas por request, se acumula un mapa durante la ejecución y se emite **un único evento rico** al final (usuario, tenant, cantidad de ítems, método de pago, tiempos, resultado…). Es la materia prima del análisis diferencial.

### Métricas: golden signals, percentiles y cardinalidad

**Las cuatro señales doradas (Google SRE):**

| Señal | Qué mide | Ejemplo |
| --- | --- | --- |
| **Latencia** | Cuánto tarda (separando éxito de error) | p99 de checkout |
| **Tráfico** | Demanda | Requests/s |
| **Errores** | Tasa de fallas, explícitas (500) o implícitas (respuesta incorrecta) | % de 5xx |
| **Saturación** | Qué tan lleno está el recurso más limitado | Pool de conexiones, CPU, memoria |

Sobre saturación: muchos sistemas se degradan **antes del 100%**, y **el aumento de latencia suele anticipar la saturación** (mirar p99 en ventanas cortas).

**La falacia del promedio:** si 99 usuarios tardan 10 ms y 1 tarda 10.000 ms, el promedio da 110 ms y **esconde** al usuario que sufrió. Hay que medir **percentiles**:
- **p50 (mediana):** experiencia típica.
- **p95 / p99:** la **cola larga**, donde aparecen contención, GC, colas.

**Explosión de cardinalidad:** una TSDB (Prometheus) crea **una serie temporal por cada combinación de etiquetas**: `series = métricas × endpoints × status × user_ids`. Meter `user_id` (10 millones) en una métrica multiplica la memoria hasta romper la base.

> **Regla:** en métricas, **solo etiquetas de baja cardinalidad** (`method`, `status_code`, `endpoint`). La alta cardinalidad (`user_id`, `order_id`) va en **spans y eventos estructurados**.

### Tracing distribuido

- **Trace:** el viaje completo de una request por el sistema, modelado como un **grafo acíclico dirigido**.
- **Span:** una unidad de trabajo con tiempo (una RPC, una query SQL, un cálculo). Tiene: Trace ID, Span ID, Parent Span ID (salvo el raíz), inicio y duración.

**SpanKind (rol del span):**

| Kind | Representa | Termina cuando… |
| --- | --- | --- |
| `CLIENT` | Llamada síncrona saliente (HTTP, gRPC, SQL) | Llega la respuesta |
| `SERVER` | Atención de una request entrante | Se despacha la respuesta |
| `PRODUCER` | Publicar un mensaje en un broker | El broker confirma (no espera al consumidor) |
| `CONSUMER` | Procesar un mensaje de una cola/topic | Termina el procesamiento (desacoplado en el tiempo) |
| `INTERNAL` | Operación interna sin red ni colas | Termina el cálculo |

**Truco del par CLIENT–SERVER:** `duración(CLIENT) − duración(SERVER)` = tiempo de **red de ida + red de vuelta + encolamiento** antes del handler. No permite separar los tres, pero **no necesita relojes sincronizados** (cada duración se mide con un solo reloj). Ojo: si el cliente abandonó por timeout, el SERVER puede seguir después de que el CLIENT cerró.

**Span links:** un consumidor que procesa un **lote** de mensajes de distintas trazas no puede tener varios padres; los *links* conectan causalmente con varias trazas sin forzar un árbol.

**Conexión con Lamport:** una traza **no** es una lista ordenada por hora. Es la **materialización de happens-before**: `parent_span_id` establece causalidad independientemente del desfase de relojes.

**Propagación de contexto:**
- **In-band:** el contexto viaja **dentro** de los mensajes (cabecera W3C `traceparent: 00-<trace_id>-<parent_id>-<flags>`, metadata gRPC). Si un servicio no reinyecta la cabecera al llamar hacia abajo, la traza se corta.
- **Out-of-band:** los spans terminados se envían **asíncronamente** al Collector para no sumar latencia al usuario.

**Sampling (muestreo):** no se pueden guardar miles de millones de trazas.
- **Head-based:** se decide al **inicio** (ej.: 5% de las requests). Simple y barato, pero puede descartar justo la request con error.
- **Tail-based:** se decide al **final**, cuando ya se sabe si hubo error o lentitud. Ej.: guardar **100% de errores y lentas** y 5% del resto. Mejor señal, más costo (hay que bufferear).

## Instrumentación en dos capas

1. **Capa de infraestructura (auto-instrumentación):** agentes/middlewares OTel capturan spans raíz, propagación de cabeceras y latencias de transporte **sin tocar la lógica de negocio**.
2. **Capa semántica de dominio (manual):** el desarrollador agrega atributos de negocio al span activo (`tenant.id`, `payment.method`, `cart.items`), para poder responder preguntas de negocio en producción.

## Debugging en producción: el Core Analysis Loop

**La causa raíz no coincide con el síntoma.** Cascada típica: (1) una base se degrada por contención de locks → (2) el servicio intermedio agota su pool de hilos → (3) los clientes **reintentan** y amplifican el tráfico → (4) el API Gateway devuelve **504**. El usuario y la alerta ven el paso 4; lo que hay que arreglar es el 1.

**Core Analysis Loop** (análisis por primeros principios, sin depender de la intuición):
1. **Observar la anomalía agregada (métrica):** saltó el p99. Sé que algo cambió, no qué.
2. **Diferenciar dimensiones (eventos anchos):** ¿qué atributo separa las requests anómalas de las normales? (ej.: `availability_zone=us-east-1a` en el 98% de las fallas vs. 15% normal).
3. **Confirmar con una traza individual:** abrir una request representativa y ver **en qué span se fue el tiempo**.
4. Si no alcanza, **filtrar por ese atributo y volver al paso 2**. Por eso es un *loop*.

## Trade-offs

| Gano | Pago |
| --- | --- |
| Reconstruir causalidad y encontrar causas raíz en minutos | Overhead de CPU y red por telemetría |
| Responder preguntas nuevas sin deployar | Costo de almacenamiento (sobre todo alta cardinalidad) |
| Correlación automática | Complejidad de configuración, necesidad de sampling |

**D = (P, F, G, S, T):** P: diagnosticar fallas imprevistas en producción. F: clock drift, cascadas, latencia variable, cardinalidad. G: reconstruir causalidad y aislar la causa raíz. S: trenza única (OTel/OTLP) + hard context W3C + instrumentación en 2 capas + core analysis loop. T: overhead, retención, sampling.

> Observabilidad **no es "mirar dashboards ni abrir tres pestañas"**: es diseñar el software para que **transporte contexto causal y pueda explicarse a sí mismo** cuando falle.

> 📝 **Para el parcial:** explicar por qué se pierde el stack trace y el orden temporal; diferenciar monitoreo y observabilidad; hard vs. soft context; las golden signals; por qué percentiles y no promedio; qué es la explosión de cardinalidad y dónde va la alta cardinalidad; span, trace, SpanKind y la resta CLIENT − SERVER; propagación in-band/out-of-band; head vs. tail sampling; aplicar el core analysis loop a un caso.

---

# Clase 11 — Resiliencia distribuida

## Pregunta central

En un sistema distribuido **la falla parcial es la condición normal**: siempre hay alguna dependencia lenta, algún nodo reiniciándose, alguna conexión cortada. La Clase 10 enseñó a **ver** las fallas; esta clase trata de **qué hace el sistema mientras ocurren**.

> La pregunta no es cómo evitar toda falla (imposible), sino **cómo impedir que una falla local se convierta en una falla global**: dado que una dependencia va a fallar, **¿en qué punto de la cadena corto la propagación y a qué costo?**

**Resiliencia** (en este curso): capacidad de seguir funcionando ante fallas parciales, aunque sea **degradado**. Una dependencia lenta o caída debe afectar **solo a la parte que la necesita, el menor tiempo posible y de forma entendible** para usuario y operador. No es lo mismo que disponibilidad total (que nada falle) ni que redundancia (tener una réplica): es la disciplina de **contener la propagación**.

## Falta, error y falla

| Término | Qué es | Ejemplo (caso aerolínea) |
| --- | --- | --- |
| **Fault (falta)** | Condición que crea un estado interno incorrecto (defecto latente que se dispara) | Tras conmutar la base, cerrar una consulta lanza una excepción no manejada y la conexión no vuelve al pool |
| **Error** | Comportamiento **visiblemente** incorrecto | Los primeros llamantes reciben excepciones en vez de vuelos |
| **Failure (falla)** | El sistema **no responde** | Pool agotado: la búsqueda de vuelos y los quioscos de check-in dejan de responder |

- **"La falla está en el ojo del observador":** una máquina puede estar encendida y no responder nada. En la aerolínea, el monitoreo miraba una página de estado y mostraba todo **sano** mientras todos los hilos estaban bloqueados. → La salud se mide **desde quien usa el servicio** (Clase 10).
- **La grieta (crack):** falta → error → falla. En cada eslabón puede acelerar o frenarse; **el acoplamiento fuerte la acelera**. Disparador + forma de propagación + daño = **modo de falla**.
- **Crackstoppers:** protecciones construidas a propósito para que el daño no avance, como las **zonas de deformación** de un auto. Si el diseño no define cómo falla el sistema, falla **como emerja**, que suele ser lo peor.

## El ejemplo recurrente: el generador de contraseñas

Un servicio arma contraseñas llamando a **cuatro servicios hijos** (mayúsculas, minúsculas, dígitos, especiales), con **5 llamadas a cada hijo por pedido** (*fan-out*): cualquier política por llamada **se multiplica por pedido**. Además hay una **dependencia oculta**: minúsculas consulta internamente a dígitos (no figura en el diagrama). Se retoma en la Clase 12.

## Anatomía de una falla en cascada

**Los puntos de integración son "el asesino número uno" de los sistemas**: cada conexión con otro sistema es una oportunidad de que su problema se vuelva tuyo. Descomponer en microservicios e integrar proveedores **multiplica** esos puntos.

**Falla en cascada:** una grieta en una capa provoca una grieta en la capa que la llama. Necesita un **mecanismo de transmisión**, casi siempre **un pool de recursos que se agota** por una falla de abajo. **Una llamada sin timeout es la forma segura de provocarla.** Los servicios con muchos llamantes (*alto fan-in*) propagan más lejos sus problemas.

### Falla rápida vs. falla lenta

Una conexión rechazada vuelve en milisegundos; un paquete perdido puede dejar un hilo **bloqueado minutos**. El hilo bloqueado no atiende a nadie; si se bloquean todos, el servidor está caído "a todo fin práctico".

> **Una dependencia lenta es más peligrosa que una caída**: la caída libera rápido los recursos del llamante; la lentitud los **retiene**. *"A slow response is a lot worse than no response."*

### Hilos bloqueados: el sistema no "se cae", "se cuelga"

La mayoría de las caídas reales son **procesos vivos con todos sus hilos esperando**. Por eso, con el sistema saturado, **la CPU está baja** (los hilos esperan E/S, no calculan). Un panel de CPU en verde es compatible con un servicio totalmente bloqueado → la saturación se mide sobre **el recurso que se agota** (hilos ocupados / disponibles).

### Otros dos mecanismos de propagación

- **Reacción en cadena:** 5 servidores iguales al 20% cada uno; cae uno → los otros 4 pasan al 25% (**+25% de carga cada uno**). Si la caída fue por una condición de carga (fuga de memoria, race condition), los sobrevivientes ahora son **más** propensos a caer: 33%, 50%… Como el defecto está en todos, **la única forma de frenarla es corregir el defecto**.
- **Capacidades desbalanceadas:** un front end con 3.000 hilos llama a un servicio con 450. Normalmente no pasa nada, pero una promoción que lleve tráfico a ese camino lo desborda. Igualar capacidades suele ser inviable: **llamantes y proveedores deben resistir avalanchas**.

### Ley de Little aplicada al pool de hilos

> **L = λ · W** — elementos en el sistema = tasa de llegada × tiempo que permanece cada uno.

*Analogía:* una peluquería con 2 clientes/hora que se quedan 1,5 h tiene en promedio 3 clientes adentro. Con 2 sillas no da abasto aunque nada esté "roto"; y si cada atención se alarga, L crece **sin que llegue un cliente más**.

**En el generador:** λ = 5 pedidos/s y pool de 8 hilos.
- Dependencias sanas: un pedido tarda ~0,05 s → L = 5 × 0,05 = **0,25 hilos** (pool holgado).
- Una dependencia tarda 2 s por llamada × 5 llamadas secuenciales → cada pedido ocupa un hilo **10 s** → L = 5 × 10 = **50 hilos**. Con 8, satura: la tasa sostenible cae a 8/10 = **0,8 pedidos/s**, por más CPU que se agregue.

> **Ninguna dependencia se cayó: alcanzó con que una se pusiera lenta. Cambió W, no λ.**

**Tiempo de residencia = tiempo en cola + tiempo de procesamiento.** Medir solo el procesamiento engaña: el servidor cree que anda bien mientras el cliente ve lentitud. La cola es serial y a la larga **domina**. (Little sirve para dimensionar y predecir la saturación, no para modelar el sistema ya saturado.)

## Las herramientas: cada una corta un tramo de la cadena

| Herramienta | Qué corta |
| --- | --- |
| **Timeout** | La espera indefinida que convierte una dependencia lenta en hilos bloqueados |
| **Reintento** | Las fallas transitorias (las recupera), a costa de más carga |
| **Circuit breaker** | La insistencia contra una dependencia que ya se sabe enferma |
| **Degradación elegante (fallback)** | El error que llega al usuario: responde algo útil en su lugar |
| **Bulkhead** | Que una parte del sistema agote recursos que comparten las demás |
| **Backpressure y load shedding** | La demanda mayor que la capacidad: frenar al productor o rechazar a tiempo |

Ninguna alcanza sola, y combinadas pueden interactuar mal.

## 1. Timeouts

**Para qué:** dejar de esperar una respuesta que probablemente no llegará. *"Hope is not a design method."* Un timeout bien puesto **aísla la falla**: el problema de otro no se vuelve tuyo.

**El timeout no viene puesto.** Muchas librerías **esperan para siempre** por defecto: `requests` de Python no corta sin el parámetro `timeout`; `URLConnection` de Java interpreta 0 como infinito. Lo mismo para recursos internos: **todo pool que bloquea debe tener un plazo máximo de espera** (en la aerolínea, el pool de conexiones bloqueaba sin límite).

| Tipo | Qué acota | Valor típico |
| --- | --- | --- |
| **Connect timeout** | Establecer la conexión | Muy corto (dentro de un datacenter conectar es rápido) |
| **Read / socket timeout** | Espera de datos una vez conectado | Reflejar la latencia normal de la dependencia |
| **Connection request timeout** | Obtener una conexión del pool del cliente | Corto |
| **Deadline (global)** | El tiempo **total** de la operación | Presupuesto de extremo a extremo |

**Ojo:** el read timeout mide el tiempo **entre** datos recibidos, no el total: una respuesta que llega en fragmentos espaciados puede tardar mucho más sin que salte. Por eso hace falta además un **deadline**.

**Cómo elegir el valor:** a partir de la **distribución de latencia** medida en condiciones normales, **del lado del cliente** (incluye red y cola): típicamente **p99 de la línea de base + margen**. Basado en el promedio cortaría muchas llamadas normales; basado en el máximo casi nunca actuaría. Revisarlo cuando cambia la dependencia.

| Timeout muy corto | Timeout muy largo |
| --- | --- |
| Convierte la variabilidad normal en errores (y en reintentos) | No protege: los hilos siguen bloqueados |

### Timeouts en una cadena A → B → C

Para el valor de partida alcanza con medir B desde A (ya incluye lo que B espera a C). Pero la línea de base **no muestra el peor caso interno**: si C falla, B espera su timeout completo, reintenta y vuelve a esperar. Si el timeout de A es menor que eso, **A abandona mientras B sigue trabajando** para una respuesta que nadie recibirá.

> **Tmax = (1 + r) · t + Σ eᵢ** (t = timeout por intento, r = reintentos, eᵢ = esperas entre reintentos). **El Tmax de cada llamada interna tiene que caber dentro del timeout de quien la origina.** En cadenas largas se reparte un **presupuesto de tiempo de extremo a extremo**.

### Timeouts y Fail Fast

"Dos caras de la misma moneda": el **timeout** protege los pedidos **salientes**; **fail fast** rechaza de inmediato los pedidos **entrantes** que ya se sabe que van a fallar (recurso no disponible, breaker abierto, parámetros inválidos), en lugar de gastar recursos y descartar el resultado. **Fallar rápido no es funcionar**: los pedidos siguen fallando; recuperarlos es tarea de los mecanismos siguientes.

## 2. Reintentos: cuándo ayudan y cuándo destruyen

En la Clase 2 el reintento era la solución a un mensaje perdido. Frente a una dependencia **saturada**, es **el combustible de la cascada**.

**La retroalimentación:** más carga → más tiempo de respuesta → se supera el timeout del llamante → **reintento "justo cuando más duele"** → más carga. Ejemplo: un backend atiende 10.000 req/s y recibe 10.100; rechaza 100, que se reintentan → recibe 10.200, rechaza 200 → 10.300… Cada vez más trabajo se dedica a pedidos que van a fallar, y **bajar la carga al nivel previo puede no alcanzar** mientras sigan los reintentos.

**Cuándo sí:** ante **fallas transitorias** (paquete perdido, conexión reiniciada). Ante fallas persistentes o sobrecarga, son destructivos. Condiciones previas: **solo operaciones idempotentes** y **solo errores reintentables** (timeouts, errores de conexión, 5xx; **no** 4xx ni validación).

### La cuenta de la amplificación

> Un pedido con **n** llamadas independientes, cada una con probabilidad de falla **p**: **P(falla) = 1 − (1 − p)ⁿ**

| Escenario (p = 10%, n = 5) | Falla por llamada | Falla del pedido | Intentos por llamada |
| --- | --- | --- | --- |
| Sin reintentos | 10% | 1 − 0,9⁵ ≈ **41%** | 1 |
| Con 1 reintento | 0,1² = 1% | 1 − 0,99⁵ ≈ **4,9%** | 1 + p = 1,1 |
| Con 2 reintentos | 0,1% | ≈ 0,5% | 1 + p + p² = 1,11 |

**El fan-out amplifica** (10% por llamada → 41% por pedido). Y el costo del reintento es bajo **solo si las fallas son independientes**: si la causa es una caída o saturación, p → 1 y **cada llamada se vuelve 1 + r intentos justo sobre quien no puede más**.

**Reintentos en varias capas se multiplican:** 3 capas × 3 intentos = **27** llamadas por pedido; si backend, frontend y JavaScript hacen 4 intentos cada uno, **4³ = 64** intentos sobre la base por una sola acción del usuario. → **Decidir en qué capa se reintenta**, no en todas.

### Backoff exponencial

> **dₙ = min(tope, base · mⁿ)**, típicamente m = 2.

Ejemplo: base 100 ms, tope 1 s → esperas de 100, 200, 400… ms. Con 2 reintentos y timeout de 500 ms por intento: **Tmax = 3 × 500 + 300 = 1.800 ms**, que debe caber en el timeout del llamante.

### Jitter

Si todos los clientes que fallaron juntos esperan lo mismo, **reintentan juntos** (*dogpile* / *thundering herd*). El jitter agrega aleatoriedad:

| Variante | Espera | Ejemplo (dₙ = 200 ms) | Trade-off |
| --- | --- | --- | --- |
| Sin jitter | dₙ | Todos a los 200 ms | Ola sincronizada |
| **Full jitter** | U(0, dₙ) | 30, 120, 190 ms | Dispersa más; alguno puede reintentar casi enseguida |
| **Equal jitter** | dₙ/2 + U(0, dₙ/2) | 110, 160, 195 ms | Garantiza una espera mínima; dispersa menos |

**Backoff + jitter es el estándar** para clientes remotos. El jitter importa cuando hay **muchos** clientes fallando a la vez.

### Presupuesto de reintentos (retry budget)

Limitar los intentos por pedido **no alcanza**: si todo falla, todos agotan sus 3 intentos y la carga casi se triplica. Google combina:
- **Por pedido:** máximo 3 intentos.
- **Por cliente:** solo reintentar mientras los reintentos sean **< 10%** de lo que envía.

Así, en el caso general el crecimiento queda en **1,1×**: si fallan pocos, se reintenta normal; si fallan muchos, se deja de reintentar y el error se propaga (lo correcto si el problema no es transitorio). Variante simple: un tope por proceso (ej.: 60 reintentos/min). **Un rechazo del circuit breaker nunca se reintenta.**

### Caso Pokémon GO

El tráfico real fue ~50× la estimación (se había probado 5×). Los backends se volvieron lentos en vez de rechazar, el balanceador reintentaba los GET y la app reintentaba con "un reintento inmediato y después backoff constante". Ráfagas de errores rápidos **sincronizaron** a los clientes → picos de **20×** el máximo previo. Se mitigó aislando y ampliando balanceadores y limitando tráfico; la corrección de fondo fue **backoff exponencial truncado con jitter**. Lección: sin dispersión, muchos clientes independientes se vuelven **una sola ola**, que puede tumbar incluso componentes ajenos (el balanceo global).

## 3. Circuit breaker

**Idea:** como el disyuntor eléctrico. Envuelve una operación peligrosa y **la evita** cuando la dependencia no está sana. A diferencia del reintento, existe para **prevenir** operaciones, no para repetirlas: degrada automáticamente bajo estrés.

| Estado | Comportamiento | Transición |
| --- | --- | --- |
| **Closed** (normal) | Las llamadas se ejecutan y se registra el resultado | Si las fallas superan el umbral → **Open** |
| **Open** | Las llamadas **fallan inmediatamente**, sin ejecutarse | Pasado el tiempo de espera → **Half-open** |
| **Half-open** | Pasa **una (o pocas) llamada de prueba** | Éxito → **Closed**; falla → **Open** otra vez |

**Qué cuenta como falla:** timeouts, errores de conexión y 5xx. **No los 4xx ni validaciones** (son culpa del llamante; abrirían el breaker por pedidos mal hechos). Puede convenir umbrales distintos por tipo (más bajo para timeouts). El rechazo del breaker debe ser una **excepción distinguible** de un timeout común.

> **Breaker + timeout es obligatorio:** el breaker solo registra una llamada cuando **termina**; sin timeout, la falla lenta **nunca termina** y el breaker no la ve.

### ¿Cuándo abrir? Fallas consecutivas vs. tasa de fallas

Un contador simple no sirve: no es lo mismo 5 fallas en 5 horas que en 30 segundos. Importa la **densidad**.

**Política de k fallas consecutivas** (un éxito reinicia el conteo). Llamadas esperadas hasta ver k fallas seguidas con probabilidad p: **E = (1 − pᵏ) / ((1 − p) · pᵏ)**.

| | p = 10% (normal tolerable) | p = 50% (dependencia enferma) |
| --- | --- | --- |
| k = 3 | ≈ 1.110 llamadas (abre por azar) | ≈ 14 llamadas (detecta rápido) |
| k = 10 | ≈ 1,1 × 10¹⁰ (nunca abre) | ≈ 2.046 llamadas (detecta tarde) |

**El dilema:** ningún k sirve para ambas cosas, porque las fallas consecutivas **mezclan tasa de error con volumen de tráfico**. A 100 llamadas/s, con k = 3 y 10% de errores el breaker abre cada ~11 s; si queda abierto 10 s, está abierto **~47% del tiempo** por una tasa que el sistema toleraba.

**Política de tasa de fallas en una ventana deslizante** (✅ recomendada): mirar las últimas **N** llamadas y abrir si la proporción de fallas ≥ **θ**, pero solo con un **mínimo m** de llamadas. Con N = 50, m = 20, θ = 50%: con 10% real, la probabilidad de abrir es ~10⁻¹² (no abre); con 50% real, abre casi en cuanto junta el mínimo. **Independiente del volumen.** El compromiso pasa al tamaño de ventana: chica = rápida pero ruidosa; grande = estable pero lenta. Variante: abrir también por **proporción de llamadas lentas**.

**Tiempo abierto:** muy corto → se prueba una dependencia aún enferma una y otra vez; muy largo → se degrada de más tras la recuperación. **Half-open:** limitar las pruebas simultáneas (si pasan todas las llamadas, cada hilo lanza su "prueba" contra una dependencia quizás enferma) y considerar exigir **varios éxitos** para cerrar (evita abrir/cerrar en bucle).

**Alcance:** un breaker **por dependencia y por proceso**. Compartir estado entre procesos agrega otra comunicación (y un nuevo modo de falla de la propia protección); un único breaker para todas las dependencias haría que la falla de una desconecte a las demás.

**Con reintentos:** **cada intento** cuenta para el breaker; si un intento lo abre, el siguiente se rechaza y la secuencia termina; y el rechazo **no se reintenta**.

**Observabilidad:** registrar cada cambio de estado y poder consultarlo; la **frecuencia de cambios** es un indicador adelantado de problemas; operaciones necesita poder abrirlo o reiniciarlo a mano.

**Una protección mal implementada:** el breaker es compartido por los hilos, así que debe ser **thread-safe**. Pero si el lock cubre **toda la llamada protegida**, serializa todas las llamadas a la dependencia: mientras un hilo espera una respuesta lenta, los demás esperan el lock. **El lock debe cubrir solo leer/actualizar el estado, nunca la llamada de red.** (Revisar esto antes de adoptar una librería.)

## 4. Degradación elegante (fallback)

Cuando la dependencia no está (timeout, breaker abierto), hay que hacer algo con el pedido: devolver la última respuesta buena o un valor en caché, una respuesta genérica en vez de personalizada, o llamar a un servicio secundario.

> **El fallback es una decisión de producto, no técnica.** ¿Acepta un comercio una orden si no puede confirmar stock? ¿Y si no puede verificar la tarjeta? Hablar del circuit breaker suele ser la mejor forma de abrir esa conversación con el negocio.

**Una buena degradación es:**
- **Explícita** para quien la recibe (ej.: un campo que lista qué partes faltan). Una respuesta degradada presentada como completa convierte una falla visible en un **error silencioso**.
- **Definida de antemano** con los interesados, no improvisada en el `catch`.
- **Más barata** que el camino normal (si cuesta lo mismo, no contiene nada).

**Cuándo NO degradar:** en el generador, omitir los especiales produce una contraseña que **viola la política**: "degradar" pasa a ser "responder algo incorrecto". Igual con un cobro sin verificación. Ahí lo correcto es **fallar explícita y rápidamente**, o **diferir** la operación.

> **Un breaker sin fallback falla rápido, pero sigue fallando.** El breaker protege **al sistema**; solo el fallback (o una decisión de producto equivalente) protege **la experiencia del usuario**.

## 5. Bulkheads (mamparos)

Como los compartimentos estancos de un barco: si se perfora uno, se inunda ese y el barco no se hunde. Se **particionan recursos** para que la falla de una parte no destruya el resto ("salvar parte del barco").

**Niveles de partición:**
- **Hilos en un proceso:** un pool (o tope de concurrencia) **por dependencia**, para que una lenta agote solo su porción. Caso útil: reservar un pool para **tareas administrativas**, para poder diagnosticar o apagar el servicio aunque los hilos de atención estén colgados.
- **CPUs de un servidor:** fijar procesos a núcleos.
- **Instancias o clústeres dedicados:** capacidad reservada para clientes críticos (ej.: servidores dedicados al check-in, que no se afectan si las consultas de estado de vuelo saturan los compartidos).
- **Zonas y regiones** en la nube.

**Costo:** **capacidad ociosa** (la partición reservada está subutilizada casi siempre). No particionar demasiado fino: buscar límites naturales (por llamante, función o topología) según el impacto de negocio.

**En la aerolínea:** el servicio tenía pools separados para llamadas remotas y para HTTP, por eso **seguía respondiendo al monitoreo** y se pudo diagnosticar. Pero el recurso que se agotó (el pool de conexiones a la base) estaba en la partición de los clientes; separar servidores por grupo de clientes habría mantenido el check-in.

## 6. Backpressure y load shedding

*"Todo problema de performance empieza con una cola que se llena en algún lado."* Por la ley de Little, si la cola crece sin límite, el tiempo de respuesta también. → **No debe haber colas no acotadas.** Con una cola acotada llena, las opciones son: fingir aceptar y descartar, aceptar y descartar otro (ej.: el más viejo), **rechazar**, o **bloquear al productor**.

| | Backpressure | Load shedding |
| --- | --- | --- |
| Qué hace | **Desacelera al productor** (lo bloquea hasta que haya lugar); idealmente se propaga hasta el cliente | **Rechaza** el trabajo que no podrá completarse a tiempo |
| Ejemplo | Ventana de recepción de TCP | Un servicio de video en vivo descarta cuadros que ya llegarían tarde |
| Funciona mejor | Dentro del sistema/empresa, con productores **conocidos y finitos** | **En el borde**, ante demanda efectivamente ilimitada |

**Principios del load shedding:**
- **Un rechazo rápido es mejor que un timeout lento** (*fail early and cheaply*).
- Rechazar **lo antes posible**, antes de ocupar recursos en varias capas (los balanceadores del borde son el lugar ideal).
- "Carga demasiado alta" = cuando los pedidos tardan más que el objetivo de servicio, o con un **semáforo** de pedidos simultáneos.
- Cada instancia puede responder **503 en su health check** para que el balanceador la saque un rato.
- Cuando el valor del trabajo **decae con la edad**, descartar lo viejo es mejor que encolarlo.

**Caso Dressy (dos mecanismos sanos que interactúan mal):** la región A empezó a hacer load shedding por CPU. El balanceador estimaba el costo por pedido con la CPU y, como los rechazos "abarataban" los pedidos, concluyó que A era 10× más eficiente y le mandó **más** tráfico (A: 400 req/s; B y C: 40). *"El balanceador no sabía que los pedidos 'eficientes' eran errores."* Corrección: contar cada rechazo como >100% de CPU. **Lección: los mecanismos de control de carga se configuran y evalúan juntos.**

## Combinar mecanismos

**Orden razonable dentro de una llamada (de afuera hacia adentro):**
1. **Bulkhead:** ¿hay lugar en la partición de esa dependencia? Si no, rechazar ya.
2. **Circuit breaker:** si la dependencia está enferma, rechazar sin llamar (va antes de llamar y de reintentar).
3. **Timeout:** el intento real, acotado.
4. **Reintentos:** si la falla es reintentable, esperar con backoff + jitter y **volver a pasar por el breaker** (cada intento cuenta; si abrió, se corta).
5. **Fallback:** si todo falló, aplicar la degradación decidida de antemano.

Cada paso ahorra recursos al siguiente.

**Interacciones peligrosas:** reintentos en varias capas que se multiplican; load shedding que engaña al balanceador (Dressy); reintentos que ocupan el pool que un bulkhead intenta proteger; reintentos sincronizados + balanceadores esperando backends colgados (Pokémon GO).

> **Más patrones no es mejor.** "Cantidad de patrones aplicados" nunca es una métrica de calidad. Cada mecanismo agrega configuración, estados y modos de falla; se justifica por la amenaza concreta que contiene y se **verifica con pruebas de carga y de falla** (Clase 12).

**Si falta un eslabón, los demás no compensan:** un breaker sin timeout no ve la falla lenta; un breaker sin fallback falla rápido pero sigue fallando.

## Caso de estudio: la excepción que dejó en tierra a una aerolínea

Un servicio central de búsqueda de vuelos (CF), **diseñado para alta disponibilidad** (clúster, balanceadores redundantes, base con conmutación automática, segundo datacenter, sin puntos únicos de falla), usado por los quioscos de check-in y la respuesta telefónica vía RMI.

- **23 h:** conmutación manual rutinaria de la base. El monitoreo muestra todo disponible.
- **2:30:** caen **todos** los quioscos del país y la respuesta telefónica (5:30 en la costa este, hora pico). El monitoreo de CF seguía en verde (solo miraba una página de estado).
- **Causa:** cerrar una consulta sobre conexiones rotas por la conmutación lanzaba una excepción **antes** de devolver la conexión al pool. Tras ~40 llamadas el pool se agotó y **todos los hilos quedaron esperando una conexión que nunca llegaría**; los 40 hilos de los quioscos, esperando a CF (RMI sin timeout). Más de 3 h de caída y vuelos demorados hasta las 15 h.

| Eslabón | Mecanismo que lo corta |
| --- | --- |
| Excepción al cerrar la consulta; conexión no devuelta | Revisión y pruebas de falla (la falta original; no todos los defectos se eliminan) |
| Pool agotado; hilos esperando conexión | **Timeout en el pool** |
| Llamadas remotas sin timeout | **Timeout en el cliente** |
| Clientes bloqueando hilos esperando a CF | **Circuit breaker** en el cliente |
| CF compartido por todos los clientes | **Bulkhead** (grupos de servidores por cliente) |
| Quioscos y teléfono caídos | **Fallback** acordado con el negocio |
| Monitoreo que solo miraba una página de estado | **Medir desde el cliente** (Clase 10) |

> **No es realista eliminar todos los defectos: hay que sobrevivirlos.** La pregunta útil es cómo evitar que un defecto en un sistema afecte a todos los demás.

## D = (P, F, G, S, T)

- **P:** impedir que la falla o lentitud de una dependencia se propague.
- **F:** fallas parciales, dependencias lentas, pools finitos, fan-out, reintentos que amplifican, tráfico no controlable.
- **G:** contención del daño, latencia acotada, recuperación automática, degradación **explícita** en vez de caída total.
- **S:** timeouts calibrados con percentiles, reintentos con backoff + jitter y presupuesto, breaker por dependencia con política de tasa, fallback decidido con el negocio, bulkheads, backpressure y load shedding.
- **T:** capacidad ociosa, respuestas degradadas, complejidad de configuración, falsos positivos/negativos del breaker, interacciones entre mecanismos.

## Errores frecuentes

- Agregar reintentos sin evaluar cuánta carga suman cuando la causa es saturación.
- Reintentar en varias capas sin calcular el producto de intentos.
- Elegir el timeout "a ojo" o desde el promedio.
- Medir la saturación con la CPU cuando lo que se agota son los hilos.
- Configurar el breaker por fallas consecutivas sin considerar el volumen de tráfico.
- Creer que un breaker sin fallback protege al usuario, o que toda degradación es aceptable.
- Configurar por separado mecanismos que interactúan (load shedding + balanceo).
- Creer que más patrones = más resiliencia.

> **Un sistema resiliente no es el que no falla, sino el que falla de la forma que se diseñó: rápido, acotado y visible.**

> 📝 **Para el parcial (ver pregunta 5 del simulacro):** distinguir falta/error/falla; explicar por qué una dependencia lenta es peor que una caída; aplicar Little al pool de hilos; calcular 1 − (1 − p)ⁿ con y sin reintentos, Tmax de una llamada con reintentos y la multiplicación de reintentos por capas; elegir un timeout desde percentiles; explicar backoff, jitter y retry budget; describir los estados del breaker, qué cuenta como falla y por qué preferir tasa en ventana a fallas consecutivas; decidir si una degradación es aceptable; justificar un bulkhead; diferenciar backpressure y load shedding; ordenar los mecanismos dentro de una llamada; recorrer la cadena del caso aerolínea.

---

# Clase 12 — Confiabilidad operativa: objetivos, experimentos e incidentes

## La pregunta que faltaba

La Clase 10 enseñó a **ver**; la 11, a **decidir cómo falla**. Falta: **¿cuándo funciona el sistema "suficientemente bien"?** Sin una respuesta numérica, cada discusión (¿libero esta versión?, ¿esta alerta merece despertar a alguien?, ¿alcanza el circuit breaker?, ¿fue grave el incidente?) se resuelve por intuición, jerarquía o por quién argumenta mejor.

### La confiabilidad es gestión de riesgo, no "que nunca falle"

Pasado cierto punto, **más confiabilidad es peor**:
- **El costo no es lineal:** cada nueve extra puede costar 100 veces más (máquinas redundantes y, sobre todo, ingenieros que no construyen funcionalidades).
- **El usuario no lo nota:** su experiencia está dominada por lo menos confiable de la cadena (su celular, su Wi-Fi). No distingue 99,99% de 99,999%.
- **Compite con la velocidad de cambio:** maximizar estabilidad frena las entregas.

Por eso el objetivo de disponibilidad es **a la vez un mínimo y un máximo**: por debajo los usuarios sufren; por encima se desperdicia esfuerzo.

### El ciclo de la clase

**Medir** (SLI, SLO, error budget) → **Alertar** (burn rate) → **Validar** (experimentos bajo falla) → **Responder** (gestión de incidentes) → **Aprender** (postmortems) → las acciones cambian qué se mide y qué se prueba.

> **"La esperanza no es una estrategia."** Ni la confiabilidad, ni los mecanismos de resiliencia, ni la respuesta del equipo se suponen: **se miden, se prueban y se ejercitan**.

Ejemplo de toda la clase: el **generador de contraseñas** de la Clase 11, con **5 solicitudes/s** (18.000 por hora, ~12,96 millones en 30 días).

## 1. Medir: SLI, SLO y SLA

| Término | Qué es | Ejemplo (generador) |
| --- | --- | --- |
| **SLI** (*Service Level Indicator*) | Una **medida** cuantitativa y bien definida de un aspecto del servicio | % de solicitudes que reciben una contraseña válida |
| **SLO** (*Service Level Objective*) | El **objetivo** para ese SLI en una ventana | ≥ 99,9% en los últimos 30 días |
| **SLA** (*Service Level Agreement*) | Un **contrato** con **consecuencias** si no se cumple | < 99,5% en el mes → descuento en la factura |

**Truco para distinguir SLO de SLA:** preguntar *"¿qué pasa si no se cumple?"*. Si no hay consecuencia explícita (reintegro, penalidad), es un SLO. (En la calle se dice "SLA" para todo; casi siempre se refieren a un SLO.)

### SLI: cómo definirlo bien

**Forma recomendada: un cociente.**

> **SLI = eventos buenos / eventos válidos × 100%**

Ventajas: todos los SLI quedan en escala 0–100%, y **mide bien las fallas parciales** (lo normal en sistemas distribuidos).

- **"Válidos"** define la población: los **4xx** (error del cliente) normalmente **se excluyen**.
- **Disponibilidad** como SLI mide la **disponibilidad semántica** (Clase 8), no el uptime: un proceso puede responder al *health check* mientras parte de las solicitudes falla.
- **La corrección entra en el SLI:** si se contara como bueno todo `200`, una contraseña sin especiales (fallback mal decidido) sería un "éxito". El evento bueno es **"200 con contraseña que cumple la política"**.

**SLIs típicos según el servicio:**

| Tipo de servicio | SLIs habituales |
| --- | --- |
| Atiende requests de usuarios | Disponibilidad, latencia, calidad (¿respuesta degradada?) |
| Almacenamiento | Latencia, disponibilidad, **durabilidad** |
| Pipeline de datos | **Frescura**, corrección, cobertura, throughput |
| Cualquiera | Corrección |

**Latencia: dos formas, no equivalentes:**
- *Basada en tiempo:* "p99 < 300 ms en cada ventana de 5 min" (cada ventana es buena o mala).
- *Basada en eventos:* "proporción de solicitudes respondidas en < 300 ms" (cada request es buena o mala). **Más precisa** y con forma de cociente.

**Especificación vs. implementación:** la especificación es lo que le importa al usuario ("solicitudes que reciben contraseña válida"); la implementación es cómo se mide:

| Implementación | Ventaja | Limitación |
| --- | --- | --- |
| Logs del servicio | Barata, disponible ya | No ve lo que nunca llegó al proceso |
| Métricas del balanceador | Más cerca del usuario | No sabe si la contraseña cumple la política |
| Cliente sintético (*prober*) | Recorre todo y valida contenido | Tráfico artificial |
| Instrumentación en el cliente | Lo más cercano a la experiencia real | Cara; mezcla problemas de la red del usuario |

> Medir **lo más cerca del usuario** posible, pero empezar con algo barato y refinar.

### SLO: cómo elegir el valor

- **Partir de lo que le importa al usuario**, no de lo fácil de medir.
- **No copiar el desempeño actual** (te ata a sostenerlo con esfuerzo heroico).
- **Simple:** un SLI por propiedad; nada de "índices de salud" que mezclan todo.
- **Sin absolutos** ("siempre disponible").
- **Pocos SLOs:** si nunca sirve para decidir una prioridad, sobra.
- **Empezar laxo y ajustar.**
- **Margen de seguridad:** SLO interno **más estricto** que el SLA publicado (generador: SLO 99,9% vs. SLA 99,5%; el equipo actúa mucho antes de la penalidad).
- **No sobrecumplir.** Caso **Chubby** (servicio de locks de Google, con Paxos): era tan confiable que otros servicios dependieron de él sin prepararse para su caída; cuando caía, todo se rompía. Solución: si en un trimestre no hubo caídas reales, **se lo apaga a propósito** para que las dependencias indebidas aparezcan temprano.

**Ventana del SLO:**

| Ventana fija (calendario) | Ventana deslizante (últimos N días) |
| --- | --- |
| Simple, se reinicia de golpe el día 1 | Se parece a la memoria del usuario |
| Una caída el día 31 y otra el día 2 caen en meses distintos | El presupuesto se recupera de a poco |

Recomendado: **deslizante de 30 días / 4 semanas**. 7–14 días es muy corta; 90 días es demasiado larga (se podría gastar casi todo en un día y "cumplir").

**Objetivos del generador:** disponibilidad ≥ 99,9% y latencia (< 500 ms) ≥ 99%, ventana deslizante de 30 días, medidos en el balanceador. **Los valores son decisiones, no datos**: quedan como hipótesis hasta tener una línea de base.

## 2. Error budget (presupuesto de error)

> **Error budget = 1 − SLO.** Es la cantidad de falla **tolerada**. "Un SLO para cumplir los demás SLOs."

| SLO | Error budget | Caída total tolerada en 30 días |
| --- | --- | --- |
| 99% | 1% | 7,2 h |
| 99,5% | 0,5% | 3,6 h |
| 99,9% | 0,1% | **43,2 min** |
| 99,95% | 0,05% | 21,6 min |
| 99,99% | 0,01% | 4,32 min |
| 99,999% | 0,001% | 25,9 s |

**Cada nueve extra divide el presupuesto por 10.** En eventos: generador con 99,9% y 12,96 M solicitudes → **~12.960 fallas permitidas** en 30 días.

### ¿Alcanzan los mecanismos de la Clase 11?

| Configuración | Solicitudes que fallan | Fallidas por hora | El presupuesto dura |
| --- | --- | --- | --- |
| Especiales al 10%, sin reintentos | 41% | 7.380 | **1,8 h** |
| Especiales al 10%, con un reintento | 4,9% | ~880 | **14,7 h** |
| Lo que admite el SLO | 0,1% | 18 | 30 días |

> El SLO transforma "está mejor" en **"está dentro" o "está fuera"**. El reintento mejoró 8 veces, pero **no alcanza**.

### Para qué se usa: alinear incentivos

Desarrollo quiere velocidad; operaciones quiere estabilidad. El error budget es el **criterio objetivo**: producto define el SLO, el monitoreo mide, y **mientras quede presupuesto se puede liberar**. Cuando queda poco, es el propio equipo el que pide más pruebas. Las caídas "que no son culpa de nadie" también consumen presupuesto.

**Política de error budget** (acordada **antes**, escrita, con quién decide). Ejemplo para el generador:
- **> 50% de presupuesto:** deploys normales con canary y rollback automático.
- **Entre 50% y 0%:** deploys conservadores (horarios de poco tráfico, escalonados 5% → observar 15 min → siguiente), confiabilidad antes que features.
- **Agotado:** se **congelan** los cambios que no sean de confiabilidad o seguridad.
- **Un incidente que consume > 20%** exige postmortem.

**El costo de cada nueve:** subir de 99,9% a 99,99% en un servicio que factura USD 1 M vale `1 M × 0,0009 = USD 900`. Si cuesta más, no conviene. Otra referencia: la tasa de error de base de los proveedores de Internet es **0,01%–1%**; fallar menos que eso es invisible para el usuario.

> **El 100% casi nunca es el objetivo correcto:** es imposible, caro y el usuario no lo nota.

## 3. Alertar sobre el SLO: burn rate

### Síntomas, no causas

Alertar por **CPU > 80%** o memoria baja produce **falsos positivos** (la CPU sube por un backup o el GC). Resultado: **fatiga de alertas** (*alert fatigue*): se ignoran todas, y el día que una importa nadie la mira.

**Una alerta útil cumple dos condiciones:** (1) cuando suena, **los usuarios realmente tienen un problema**; (2) quien la recibe **puede hacer algo**. Una alerta sobre el SLO cumple la primera por construcción; para la segunda están las trazas, métricas y logs (Clase 10).

**Las alertas por causa no desaparecen:** se reservan para causas **seguras e inminentes** (disco por llenarse, certificado por vencer, una sola réplica sin repuesto), muchas veces como **ticket** y no como aviso nocturno.

### Tres formas de alertar sobre el SLO

1. **Cuando el SLO ya se incumplió:** la más obvia y la menos útil (ya no queda nada que proteger).
2. **Cuando el saldo baja de un umbral (ej.: 30%):** avisa antes pero es tosca (no distingue gastarlo en 3 semanas o en 1 hora) y el umbral se vuelve un "nuevo cero".
3. **Sobre la velocidad de consumo (burn rate):** ✅ recomendada; la urgencia es **proporcional al daño**.

### Burn rate

> **b = tasa de error observada / (1 − SLO)**. Con b = 1 se gasta el presupuesto exactamente en la ventana; con b, se agota en **T / b**.

Para el generador (SLO 99,9%, T = 720 h):

| Tasa de error | Burn rate | Se agota en | Situación |
| --- | --- | --- | --- |
| 0,1% | 1 | 30 días | Justo en el límite |
| 0,6% | 6 | 5 días | Degradación sostenida |
| 1,44% | 14,4 | 50 h | Degradación seria |
| 4,9% | 49 | ~14,7 h | Especiales al 10%, con reintento |
| 41% | 410 | ~1,8 h | Especiales al 10%, sin reintentos |
| 100% | 1000 | 43 min | Caída total |

**Presupuesto consumido** si un burn rate b dura un tiempo t: **b · t / T**.

### Ventanas y umbrales: varias alertas, cada una con dos ventanas

Se busca equilibrar **precisión** (que suene solo con problemas reales), **cobertura** (que todo problema serio alerte), **tiempo de detección** y **tiempo de reinicio** (que deje de sonar al resolverse).
- **Una sola ventana corta:** rápida pero salta con picos irrelevantes.
- **Una sola ventana larga:** precisa pero lenta, y sigue sonando horas después ("¿volvió o es el eco?").
- **Un solo umbral alto:** se pierden consumos lentos (b = 10 agota todo en 3 días sin disparar una alerta de 14,4).

**Solución: multi-ventana, multi-umbral.** Cada alerta exige superar el umbral en una **ventana larga** (el consumo es significativo) **y** en una **ventana corta** (sigue pasando ahora). La corta mide ~1/12 de la larga.

| Severidad | Ventana larga | Ventana corta | Burn rate | Presupuesto consumido al sonar |
| --- | --- | --- | --- | --- |
| **Page** (despierta a la guardia) | 1 h | 5 min | 14,4 | 2% |
| **Page** | 6 h | 30 min | 6 | 5% |
| **Ticket** (día hábil siguiente) | 3 días | 6 h | 1 | 10% |

Ejemplo: en la última hora fallaron 300 de 18.000 (1,67%) y en los últimos 5 min 30 de 1.500 (2%); ambas > 1,44% → **suena**. A ese ritmo el presupuesto se agota en ~43 h.

**Tiempo de detección:** con especiales fallando 10% sin reintentos (41%), la alerta llega en ~2 min; ante caída total, en < 1 min; una degradación leve tarda más (y está bien: consume más lento).

**Servicios con poco tráfico:** con 10 req/h, un solo fallo es burn rate 100. Opciones: tráfico sintético, agrupar servicios que fallan juntos, cambiar el producto (reintentos del cliente), o bajar el SLO / agrandar la ventana.

### Contar solicitudes vs. contar minutos

| | Contar **solicitudes** | Contar **minutos** (bueno/malo) |
| --- | --- | --- |
| Presupuesto (99,9%, 30 días) | 12.960 solicitudes fallidas | 43,2 minutos malos |
| 1 h con 1% de fallas | Consume ~1,4% (problema menor) | Los 60 minutos son malos → **agota todo** (exagera) |
| 4% de requests lentas constante (SLO latencia 99%) | Agota en ~7,5 días (**lo ve**) | Cada minuto tiene p95 bueno → **no lo ve** |
| Uso | Operar el servicio y calcular burn rate | SLA o página de estado ("caído 12 minutos") |

En sistemas distribuidos lo habitual es la **degradación parcial (*brownout*)**, no la caída total: contar solicitudes descuenta exactamente lo que vivieron los usuarios.

**Caso Honeycomb:** una pérdida de memoria tras un deploy hacía caer y reiniciar máquinas; ~1,5% de datos fallaba en ráfagas de 20 min. La alerta de burn rate sonó a la 1:29, la guardia lo tomó como un pico, y recién a la cuarta alerta (9:55) se declaró incidente. **El monitoreo tradicional nunca vio nada** (quedarse sin memoria "era normal"). Se gastó más de 5 veces el presupuesto. Desde entonces usan el burn rate como alerta principal.

## 4. Validar: experimentos bajo falla (Chaos Engineering)

### Por qué no alcanzan los tests

Los tests unitarios/integración usan tiempos ideales y mocks limpios; prueban **funciones**, no **propiedades del sistema completo**. Hay problemas que solo aparecen con todo junto (reintentos que se multiplican, cascadas, lentitud que bloquea hilos ajenos): **la seguridad no se compone**. Y **staging nunca es producción** (menos carga, menos instancias, red sin congestión).

### Definición

> **Chaos Engineering:** disciplina de **experimentar** sobre un sistema para ganar confianza en que soporta condiciones turbulentas en producción. **No es romper cosas al azar**: es provocar fallas **controladas y con método**.

**Por qué provocar fallas a propósito:**
- **Los sistemas pierden margen sin que nadie lo decida:** cada optimización razonable (pool justo, quitar una réplica, ajustar un timeout) consume holgura, y nadie suma el total.
- **Lo que nunca falla no se sabe manejar** (caso Chubby).
- **La falla pequeña y frecuente se vuelve rutina**, como un simulacro de evacuación.

**Ejemplo clásico:** **Chaos Monkey** de Netflix apaga al azar un servidor de un servicio; el servicio debería seguir funcionando con los demás.

### Los cuatro pasos

1. Definir el **steady state** (estado estable): una salida medible que indica comportamiento normal. **→ Es el SLI.** Sin SLI no hay qué observar (por eso primero se mide y después se experimenta).
2. Formular la **hipótesis** de que el steady state se mantiene en un **grupo de control** y en un **grupo experimental**.
3. **Inyectar** variables del mundo real (servidores caídos, latencia, conexiones cortadas).
4. **Intentar refutar** la hipótesis buscando diferencias entre ambos grupos.

**Hipótesis bien escrita:** observable desde afuera y refutable. "Queremos ver si aguanta" no sirve.

### Qué inyectar

- **Apagar servidores** (lo más simple).
- **Agregar latencia** (encuentra lo que apagar no muestra: servicios sin alternativa ante un timeout). **La falla lenta es la más peligrosa.**
- **Hacer fallar una llamada entre servicios:** se **marca** la request al entrar ("cuando el generador llame a especiales, que falle") y la marca viaja como el contexto de una traza. Permite fallar una fracción pequeña.
- Perturbaciones de red (pérdida, partición) y de recursos (CPU, disco).

**Dónde:** al principio, al azar; después, mirar el árbol de llamadas de una traza exitosa y preguntar "¿qué pasa si quito cada llamada?".

### Seguridad del experimento

- No experimentar en producción si cada falla es **irrecuperable** (pagos, datos médicos).
- **Acotar el blast radius** (ej.: 1 de cada 10.000 solicitudes).
- Poder **seguir cada request** (trazas) y conocer la **línea de base** del SLI.
- **Plan de recuperación** probado (el sistema no siempre vuelve solo).
- Escalera de entornos: local → integración → % pequeño de producción → producción.
- **Criterio de aborto** fijado antes, en el idioma de las alertas: abortar si el burn rate de la ventana corta supera lo esperado.
- **El experimento consume error budget**: 1% del tráfico fallando = burn rate 10; 30 min ≈ 10 × 0,5/720 ≈ **0,7%** del presupuesto. Si queda poco, la política puede postergarlo.

**Advertencias reales:** en Google, bloquear el acceso a 1 de 100 bases de prueba tumbó sistemas clave y **revertir falló** (tardó ~1 h): desde entonces se exige probar la reversión. Y "**el silencio no es consentimiento**": una ingeniera avisó por mail que haría caos en staging, nadie respondió, y staging quedó inutilizable una semana.

**Automatizar y repetir** (un buen experimento es aburrido). Las **personas también fallan**: simulacros donde parte del equipo queda "incapacitada" (en Google, ejercicios **DiRT**).

### Template de experimento

| Campo | Contenido |
| --- | --- |
| Pregunta | Qué mecanismo o supuesto se valida |
| Steady state | SLI usado y su línea de base |
| Hipótesis | Qué se espera, en forma refutable |
| Perturbación | Qué se inyecta, dónde, cuánto, por cuánto tiempo |
| Grupos | Control vs. experimental |
| Blast radius | Qué % y qué componentes pueden afectarse |
| Costo esperado | Error budget a consumir |
| Criterio de aborto | Qué señal lo detiene y quién |
| Recuperación | Qué reiniciar, revertir o limpiar |
| Resultado | Qué se observó y qué acciones salen |

**Ejemplo en el generador:** ¿los timeouts y el breaker contienen una dependencia **lenta**? Inyectar +2 s de latencia en las llamadas a especiales del **1%** de las solicitudes durante 30 min. Hipótesis: esas solicitudes **fallan rápido** con error explícito y el grupo de control mantiene sus SLIs. **Refutaría:** que suba la latencia del control (los hilos se comparten → **falta un bulkhead** o el timeout es largo), o que las marcadas tarden 2 s en fallar (**el timeout no está activo**). Repetido sobre "dígitos", **descubre la dependencia oculta** de minúsculas.

> La resiliencia **no se supone: se valida**. "Apagar cosas y mirar" no produce evidencia.

## 5. Responder: gestión de incidentes

**Incidente:** evento que degrada o amenaza el servicio (disponibilidad, latencia, corrección, seguridad de datos) y requiere **respuesta coordinada**. Con SLOs, una alerta de burn rate de severidad *page* suele ser el comienzo. Criterios para declararlo (ej. Google): hace falta un segundo equipo, es visible para clientes, o sigue sin resolverse tras 1 h. **Mejor declararlo temprano** y cerrarlo rápido.

**El incidente no gestionado** (todos hacen "lo que creen", alguien despliega una optimización improvisada y empeora todo) muestra tres peligros: **foco exclusivo en lo técnico**, **mala comunicación** y **trabajo por cuenta propia**.

### Roles (modelo tomado de bomberos / Incident Command System)

| Rol | Responsabilidad |
| --- | --- |
| **Incident commander** | Estado general, organiza y asigna; tiene todos los roles no delegados; remueve obstáculos |
| **Operaciones** | Aplica las herramientas; **único grupo que modifica el sistema** |
| **Comunicación** | Cara pública: actualizaciones periódicas; mantiene el documento |
| **Planificación** | Largo plazo: registra errores, organiza relevos, anota desvíos para revertir |

Además: un **puesto de mando** reconocido (canal), un **documento vivo** (resumen, estado, roles, **criterios de salida**, tareas, línea de tiempo) y **traspaso explícito** del rol ("ahora sos el IC, ¿de acuerdo?").

**Criterio de salida = SLO:** el incidente termina cuando el SLI vuelve al objetivo durante un tiempo acordado, no cuando "parece" resuelto.

**Prioridades:** (1) **detener el daño**, (2) **restablecer el servicio**, (3) **conservar evidencia**. **Primero se mitiga, después se entiende**; la culpa y el rediseño, después.

**La mitigación más frecuente es revertir:** en Google ~**70%** de las caídas vienen de **cambios** en producción. Primera hipótesis: el último cambio.

**Runbooks:** tener pasos documentados reduce el tiempo de recuperación a ~1/3. Pero un documento que intenta cubrir todos los errores envejece y puede ser peligroso. Equilibrio: **ficha breve del servicio** (dueño, guardia, escalamiento, dependencias, dashboards) + **una entrada por alerta** (qué significa, cómo confirmarla, mitigaciones seguras, a quién escalar). En el generador: es seguro revertir el último cambio; **no** es seguro deshabilitar timeouts ni activar un fallback sin especiales.

## 6. Aprender: postmortems

**Postmortem:** registro escrito de un incidente (impacto, acciones, causas y seguimiento) para **evitar que se repita**. No es un castigo.

**Cuándo es obligatorio:** degradación visible por encima de un umbral, **pérdida de datos**, intervención de la guardia (rollback, desvío de tráfico), resolución muy larga, **falla del monitoreo** (se descubrió a mano), o según la política de error budget.

**Estructura:** resumen · impacto (cuantificado, con error budget consumido) · causas · disparador · resolución · detección · acciones (con tipo, dueño y seguimiento) · lecciones (qué salió bien, qué mal y **dónde tuvimos suerte**) · línea de tiempo · información de soporte.

### Blameless (sin culpables)

Se buscan las **causas del sistema** sin acusar personas, asumiendo que todos actuaron bien con la información que tenían. Si se castiga, la gente esconde los problemas. (Viene de la aviación y la salud.) **Blameless no es ausencia de responsabilidad:** igual señala qué mejorar.

> La pregunta útil no es *"¿por qué se equivocó?"* sino ***"¿qué información, presiones y herramientas hicieron que esa decisión pareciera razonable en ese momento?"*** (cuidado con el **sesgo retrospectivo**: después todo parece obvio).

### Disparador, condiciones latentes y factores contribuyentes

Un incidente **rara vez tiene una sola causa**: los sistemas tienen varias defensas que deben fallar juntas y siempre hay **defectos latentes** esperando un disparador. Buscar "**la** causa raíz" lleva a arreglar un solo eslabón.

**Ejemplo (generador):** un deploy cambió la biblioteca HTTP y el timeout hacia especiales pasó al **valor por defecto: infinito**. Días después, especiales tuvo pausas de GC → hilos bloqueados → pool agotado → generador caído. La alerta sonó en 2 min, pero mitigar tardó 25 min porque el cambio venía mezclado con otros.

| Factor | Ejemplo | Acción que sugiere |
| --- | --- | --- |
| Disparador | Pausas de GC de especiales | — |
| Condición latente | Timeout dependiente del valor por defecto | **Prevenir:** el deploy falla si falta el timeout |
| Falla de validación | Sin canary; sin experimento de latencia periódico | **Detectar:** experimento semanal automatizado |
| Falla de diagnóstico | El dashboard no mostraba latencia por dependencia | **Detectar:** panel por dependencia |
| Falla de mitigación | Deploy con muchos cambios, difícil de revertir | **Mitigar:** deploys de un solo cambio para libs de red |

### Buenas acciones

- **Un dueño** por acción (y uno del postmortem).
- **Registradas** con prioridad en el sistema de seguimiento.
- **Verificables** (nada de "mejorar").
- **Prevenir, no solo mitigar**; cambiar sistemas > pedir a personas que cambien.
- Clasificadas: investigar, prevenir, mitigar, reparar, detectar.
- ❌ Malas: "tener más cuidado al desplegar", "mejorar el monitoreo", listas sin dueño. Ojo con acciones propuestas "en caliente" (suelen ser exageradas).

| Postmortem malo | Postmortem bueno |
| --- | --- |
| Jerga interna sin explicar | Lo entiende alguien de otro equipo |
| Impacto vago ("muchos usuarios") | Impacto medido, con su fuente |
| Acciones vagas, sin dueño | Acciones con dueño, prioridad y estado final verificable |
| Señala personas, tono dramático | Describe qué falló en el sistema |
| Se publica meses después, solo al equipo | Circula en días y se comparte ampliamente |

**Cultura:** revisar los postmortems, leerlos en grupo, reconocer públicamente a quien maneja bien un incidente (aunque lo haya causado). Señal de alarma: **incidentes que se repiten**.

## D = (P, F, G, S, T) de la clase

- **P:** operar con un nivel de servicio explícito, verificarlo bajo falla y aprender cuando no se cumple.
- **F:** fallas parciales inevitables, propiedades emergentes, cambios frecuentes, mediciones imperfectas, poco tráfico, personas bajo presión.
- **G:** SLOs medidos desde el usuario, alertas proporcionales, resiliencia validada, respuesta coordinada, acciones que reducen repetición.
- **S:** SLI como cociente, SLO con ventana deslizante, error budget con política, burn rate multi-ventana, experimentos con blast radius y aborto, roles de incidente, runbooks breves, postmortems blameless.
- **T:** cada nueve cuesta más; los experimentos gastan presupuesto y pueden volverse incidentes; las alertas cambian precisión por velocidad; la documentación envejece; los postmortems llevan tiempo.

## Errores frecuentes

- Fijar el SLO en 100% o copiar el desempeño actual.
- Elegir SLIs de infraestructura (CPU) en vez de experiencia de usuario.
- Contar como bueno todo `200` aunque la respuesta sea incorrecta.
- Usar el promedio de latencia en vez de una proporción bajo umbral.
- Alertar cuando el presupuesto ya se agotó o con un umbral fijo.
- Creer que Chaos es romper al azar; experimentar sin hipótesis, steady state ni aborto, o sin calcular el presupuesto.
- Buscar una única causa raíz o reducir todo a "error humano".
- Cerrar postmortems con "tener más cuidado".

**Aplicación al TP (Asteria):** la **garantía** (G) no lleva números ni absolutos; los números (SLI/SLO) son atributos de la **solución** (S/T). El error budget permite **comparar soluciones**: con SLO 99,9% (43,2 min), si cae un datacenter con 1/3 de las instancias, la solución A (reubica en 15 min) consume ~12% del presupuesto y la B (réplicas listas, 1 min) < 1%. Conviene un **SLI por subsistema y por tipo de jugador** (los que ya juegan vs. los que ingresan); el **load shedding** de la Clase 11 es gastar el presupuesto del ingreso para proteger a los que ya están jugando.

> 📝 **Para el parcial:** distinguir SLI/SLO/SLA con ejemplo; escribir un SLI como cociente y decidir qué cuenta como bueno y válido; calcular error budget en tiempo y en eventos; calcular burn rate y en cuánto se agota; explicar por qué alertar por síntomas y con varias ventanas; diseñar un experimento de caos con el template; describir roles de incidente y prioridades; escribir acciones de postmortem verificables y separar disparador de condiciones latentes.

---

# Cierre — Cómo responder en el parcial

El simulacro muestra el formato: **un caso transversal** (Sofía compra la butaca única A17), cinco preguntas que recorren la materia y una consigna fija: **cerrar cada respuesta con una línea D = (P, F, G, S, T) y una métrica o prueba** que permita revisar la decisión. Se corrige **el mecanismo que preserva la garantía y cómo se comporta ante fallas**, no la lista de tecnologías.

## Qué tema de la materia evalúa cada pregunta del simulacro

| Pregunta | Tema | Clases | Idea clave de la respuesta esperada |
| --- | --- | --- | --- |
| 1. Carga y frontera | Monolito modular vs. extraer un servicio | 9, 10 | Primero una medida **interna** (limitar o precalcular Popularidad); extraer solo si el p99 sigue > objetivo **a igual carga**. La autoridad sobre A17 sigue en Inventario |
| 2. Evento y outbox | Dual write, outbox, inbox | 7, 9 | Base primero → compra real sin evento; broker primero → evento falso. Outbox en la misma transacción, relay que republica el **mismo** `event_id`, consumidor que inserta el `event_id` y actualiza **atómicamente** |
| 3. Saga con pago incierto | Saga, estados inciertos, compensación, idempotencia | 2, 9 | Timeout ≠ rechazo: **consultar** a Pagos con la misma clave. Reserva vencida ⇒ Inventario rechaza la transferencia ⇒ **anular** la autorización (otra clave), no reembolsar. Pendiente hasta tener evidencia durable |
| 4. Diagnóstico de incidente | Observabilidad, amplificación por reintentos | 10, 11, 12 | Separar **síntomas** de **hipótesis**; dos hipótesis rivales; métrica + traza + log que las discriminen (sin IDs como etiquetas de métricas); mitigar revirtiendo y acotando reintentos |
| 5. Partición y canary | CAP, resiliencia, Strangler/canary | 1, 8, 9, 11 | La región aislada puede **mostrar** (con atraso) pero **no confirmar** compras; deadline + reintentos acotados con backoff/jitter + breaker + misma clave; canary reversible que no deshace compras |

## Checklist para cualquier respuesta

1. **¿Quién es la autoridad** sobre el dato o el invariante? (Una copia, caché o vista derivada **no** decide.)
2. **¿Qué falla** estoy asumiendo? Moverla de lugar en el timeline (antes/después del COMMIT, del ACK, de publicar).
3. **¿Qué sabe y qué no sabe** cada actor después de un timeout? Timeout **no es** fallo ni rechazo.
4. **¿Qué tiene que ser durable** y **en la misma transacción local**? (Estado + outbox; event_id + efecto; clave + resultado.)
5. **¿Los reintentos usan la misma clave idempotente?** ¿Operaciones distintas (autorizar/anular) usan claves distintas?
6. **¿Qué es sincrónico** (camino crítico antes de responder "éxito") **y qué puede ir después**?
7. **¿Qué costo acepto?** (Latencia, rechazo, atraso de vistas, duplicados, operación de otro servicio.)
8. **¿Cómo lo mido o pruebo?** Una métrica concreta con valor inicial, objetivo o umbral (p99, lag, intentos pendientes > 5 min, errores vs. línea de base) o un experimento de falla.

## Frases que conviene evitar (y qué decir en su lugar)

| Evitar | Decir |
| --- | --- |
| "Usamos Kafka porque escala" | Qué problema resuelve el log y qué costo nuevo trae |
| "Exactly-once" a secas | At-least-once + idempotencia, dentro de qué frontera |
| "Hubo timeout, el servicio cayó" | Las historias compatibles y cómo consultar el resultado durable |
| "Consistencia eventual" sin más | Qué vista puede atrasarse, cuánto y cómo se repara |
| "Hacemos rollback" del pago | Compensación (anular/reembolsar) idempotente que puede fallar |
| "La causa raíz fue X" | Disparador + condiciones latentes + factores contribuyentes |
| "Mejorar el monitoreo" | Una métrica o panel concreto, con dueño |
