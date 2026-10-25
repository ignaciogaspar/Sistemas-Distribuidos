# Guía del simulacro de parcial

> Explicación del simulacro de parcial de Sistemas Distribuidos 2026 (`SD_Parcial_Simulacro-1-1.pdf`) y de su pauta de corrección (`SD_Simulacro_Respuestas_Esperadas_2026.pdf`). Para cada pregunta: qué pide, qué temas de la materia usa, **cómo razonar paso a paso para llegar a la respuesta**, la respuesta esperada y cómo se reparten los puntos.
>
> Complementa al `Resumen-Sistemas-Distribuidos.md`: acá no se re-explican los conceptos, se muestra **cómo aplicarlos**.

---

# 1. Cómo es el examen

## Formato

| Aspecto | Detalle |
| --- | --- |
| Duración | 180 minutos |
| Puntaje | 100 puntos, 5 preguntas |
| Estructura | **Un único caso transversal** (la compra de Sofía) que se usa en las 5 preguntas |
| Extensión | **Máximo 250 palabras por pregunta** (las tablas o diagramas pedidos no cuentan como texto) |
| Supuestos | Se pueden declarar **hasta dos supuestos por pregunta**, compatibles con el caso |
| Cierre obligatorio | **Cada pregunta termina con una línea D = (P, F, G, S, T) y una métrica o prueba** que permita revisar la decisión |

| Pregunta | Tema | Minutos sugeridos | Puntos |
| --- | --- | --- | --- |
| 1. Crecer y decidir una frontera | Monolito modular vs. extraer un servicio | 25 | 20 |
| 2. Confirmación y evento durable | Dual write, outbox, idempotencia del consumidor | 30 | 20 |
| 3. Pago incierto y reserva vencida | Saga, timeout, compensación | 40 | 25 |
| 4. Investigar un incidente | Observabilidad, reintentos que amplifican | 30 | 20 |
| 5. Partición regional y migración | CAP, resiliencia, canary | 25 | 15 |

## Qué se corrige (y qué no)

- ✅ **El mecanismo que preserva la garantía y cómo se comporta ante fallas.**
- ✅ Decisiones concretas que usen **los datos provistos** (números, tiempos, porcentajes).
- ✅ Diseños equivalentes bien justificados (no hay una única respuesta válida).
- ❌ **No** se pide código ni una lista de tecnologías. Decir "uso Kafka" o "uso Redis" no suma nada.
- ❌ **No** se aceptan garantías mágicas: "exactly-once", "transacción global", "consistencia" sin decir cuál.

## Cómo leer el caso antes de empezar (lectura inicial)

Antes de responder nada, conviene subrayar en el enunciado **cinco cosas**, porque toda la corrección gira alrededor de ellas:

1. **¿Quién es la autoridad de cada dato?** → Inventario decide reserva y titularidad de A17; Pagos decide la autorización; Venta decide el estado del intento (CONFIRMADA).
2. **¿Cuál es el invariante?** → A17 tiene como máximo un titular; no hay dos autorizaciones lógicas por intento.
3. **¿Cuándo se puede decir "éxito"?** → Solo tras autorización válida **+** transferencia válida **+** COMMIT durable del intento en Venta.
4. **¿Qué no existe?** → **No hay transacción global** entre autoridades ni con el broker.
5. **¿Qué puede atrasarse?** → Popularidad y Notificaciones.

---

# 2. El caso transversal: la compra de Sofía

Una plataforma vende entradas numeradas. Sofía quiere la **butaca única A17**; puede haber compradores simultáneos en dos regiones (R1 y R2).

## Las cinco responsabilidades

| Responsabilidad | Qué decide | Tipo |
| --- | --- | --- |
| **Venta** | Oferta, intento (crea y guarda el ID del intento) y confirmación final | Autoridad del intento |
| **Inventario** | Reserva y titularidad de A17 | **Única autoridad** sobre A17 |
| **Pagos** | Autorización contra un proveedor externo | Autoridad del pago |
| **Popularidad** | Vista derivada (conteos) | Derivada, puede atrasarse |
| **Notificaciones** | Aviso posterior | Posterior, puede atrasarse |

## Reglas del caso que hay que tener presentes siempre

- **La reserva vence a los 90 segundos**, según el reloj y el estado de **Inventario**. Verificar vigencia y transferir son **atómicos en Inventario**.
- **Pagos solo autoriza**: pone una **retención**, no cobra. Si una autorización ya no corresponde, se **anula** (no se reembolsa, porque nunca hubo cobro).
- **Claves idempotentes estables** derivadas del intento y del tipo de operación: `i-42:autorizar`, `i-42:anular`. **Solo los reintentos de la misma operación reutilizan su clave**; operaciones distintas usan claves distintas.
- **Las autoridades permiten consultar resultados durables** (puedo preguntar "¿qué pasó con `i-42:autorizar`?").
- **Cada autoridad tiene transacciones locales, pero no hay transacción global** entre autoridades ni con el broker.

## Camino feliz (lo vas a necesitar en varias preguntas)

1. Venta crea el intento `i-42`.
2. Inventario **reserva** A17 (vence en 90 s).
3. Pagos **autoriza** (`i-42:autorizar`).
4. Inventario **verifica vigencia y transfiere** atómicamente.
5. Venta **confirma durablemente** (COMMIT de CONFIRMADA).
6. Venta responde "compra confirmada" a Sofía.
7. *Después:* Popularidad y Notificaciones.

> **Invariantes transversales que la pauta exige en todas las respuestas:** Inventario decide reserva y titularidad (un catálogo o una copia **no** la conceden); Pagos solo autoriza o anula; Venta comunica éxito tras transferencia válida y su propio COMMIT; los reintentos conservan la clave de **su** operación; Popularidad y Notificaciones son posteriores; **nada depende de un COMMIT global**.

---

# 3. Método general para cualquier pregunta

1. **Leé el formato pedido primero** (tabla de dos filas, hasta seis pasos, hasta siete estados…). Respetarlo es parte del puntaje.
2. **Ubicá la autoridad y el invariante** que están en juego en esa pregunta.
3. **Dibujá el timeline** y **mové la falla** (antes/después del COMMIT, del ACK, de publicar, del vencimiento).
4. **Preguntate qué sabe y qué no sabe** cada actor en ese momento. Un timeout **no es** un rechazo.
5. **Decidí qué tiene que ser durable y en la misma transacción local.**
6. **Proponé el mecanismo** (outbox, consulta por clave, compensación, breaker…).
7. **Nombrá el costo** que aceptás.
8. **Cerrá con D = (P, F, G, S, T) + una métrica con valor inicial, objetivo o umbral.**

### Cómo escribir bien la línea D

| Letra | Qué poner | Error típico |
| --- | --- | --- |
| P | El problema **observable** del caso (con el dato: "p99 sube a 3,8 s") | Ponerlo genérico ("problemas de rendimiento") |
| F | La falla **concreta** asumida (caída entre COMMIT y publicar, respuesta perdida…) | "Fallas de red" sin más |
| G | La garantía, **sin números ni absolutos** ("un único titular", "evento fiel a Venta") | Prometer "nunca falla" |
| S | El mecanismo | Nombrar una tecnología |
| T | El costo que aceptás (atraso, duplicados, menor disponibilidad, otro servicio que operar) | Omitirlo o decir "ninguno" |

### Cómo escribir bien la métrica

Una métrica sirve si tiene **qué se mide + desde dónde hasta dónde + umbral o comparación**. Ejemplos buenos:
- "p99 de compra: 3,8 s → < 2 s **a igual carga** (1500 lecturas + 30 compras/s)".
- "p99 desde el COMMIT de Venta hasta la aplicación en Popularidad < 30 s".
- "Cantidad de intentos en PAGO_INCIERTO o ANULACION_PENDIENTE por más de 5 minutos".

---

# 4. Pregunta 1 — Crecer y decidir una frontera (20 puntos)

## Qué plantea

Las cinco responsabilidades son **módulos de una misma aplicación** (comparten deploy y CPU). Durante una promoción: **1500 consultas/s** y **30 compras/s**. Un cálculo de **Popularidad** sube la **CPU de la aplicación al 92%** y el **p99 de compra pasa de 0,9 a 3,8 s**. La **CPU de la base de Inventario está en 42%**. Objetivo: **p99 < 2 s** sin doble titularidad.

Pide: (a) una medida dentro de la aplicación y una primera frontera física, elegir una como experimento y decir qué evidencia decide el paso siguiente; (b) el camino crítico de la compra en ≤ 6 pasos y qué queda fuera, con la autoridad sobre A17; (c) 2 beneficios y 2 costos de la frontera; (d) línea D + métrica.

**Temas:** Clase 9 (monolito modular, cuándo separar, camino crítico, costo de distribuir), Clase 10 (leer señales).

## Cómo razonar

**Paso 1 — Leer los datos como un diagnóstico.**
- La CPU **de la aplicación** está al 92% y lo que la consume es **Popularidad**.
- La base de **Inventario** está al 42%: **no es el cuello de botella**. → Extraer Inventario sería un error (lo dice la pauta).
- Popularidad es una **vista derivada** que **puede atrasarse**: no necesita estar en el camino de la compra.

> Conclusión: el problema es **contención de CPU** entre un trabajo secundario (Popularidad) y el camino crítico (compra).

**Paso 2 — Aplicar la regla de la Clase 9: primero lo simple.** Antes de crear un servicio, probar una medida **interna**: diferir, precalcular, limitar la concurrencia del cálculo de Popularidad o darle menos prioridad. Si eso no alcanza, recién ahí la **primera frontera física** es **extraer Popularidad** (es la que causa el problema y es la más fácil de separar porque es derivada).

**Paso 3 — Diseñar el experimento con criterio de paso.** "Ensayar **a la misma carga** (1500 lecturas + 30 compras/s). Si el p99 sigue > 2 s y hay espera por CPU de la aplicación, extraer Popularidad." Sin "misma carga" la comparación no vale.

**Paso 4 — Camino crítico.** ¿Qué tiene que pasar **antes** de decir "confirmada"? Reserva, autorización, transferencia, COMMIT de Venta. Popularidad y Notificaciones van **después**.

**Paso 5 — Beneficios y costos verificables.** No vale "escala mejor". Un beneficio verificable se puede **medir contra el síntoma**: "el cálculo deja de competir por CPU → baja el p99 de compra". Los costos son los de distribuir: eventos y red (atraso, duplicados → conteo eventual), otro servicio que operar y observar.

## Respuesta esperada

**a)** Primero diferir o precalcular Popularidad con **concurrencia limitada** dentro de la aplicación. Ensayar a igual carga (1500 lecturas y 30 compras/s): si el p99 sigue > 2 s con espera por CPU de la aplicación, **extraer Popularidad** con recursos propios. La CPU de Inventario al 42% **no** justifica extraer Inventario.

**b)** Secuencia de seis pasos:
1. Venta crea el intento.
2. Inventario reserva.
3. Pagos autoriza.
4. Inventario verifica y transfiere atómicamente.
5. Venta confirma durablemente.
6. Venta responde a Sofía.

*Fuera del camino:* Popularidad y Notificaciones. *Autoridad sobre A17:* Inventario.

**c)**
- Beneficio 1: separa el consumo de CPU del cálculo, contrastable con el p99 de compra.
- Beneficio 2: Popularidad se despliega y dimensiona de forma independiente.
- Costo 1: eventos y red pueden retrasarse o duplicarse → el conteo pasa a ser **eventual**.
- Costo 2: hay que **operar y observar otro servicio** y su comunicación.

**d)** D = (P: p99 sube a 3,8 s; F: cálculo que compite por CPU; G: p99 < 2 s y un único titular; S: limitar el cálculo y extraer Popularidad si persiste; T: atraso y operación adicional). **Métrica:** p99 de compra 3,8 s → < 2 s a carga equivalente.

## Cómo se reparten los puntos

| Inciso | Puntos | Qué tiene que aparecer |
| --- | --- | --- |
| a | 6 | Medida interna focalizada en Popularidad (2); frontera física = Popularidad, con razón (2); experimento a igual carga con criterio de paso (2) |
| b | 5 | Reserva, autorización, transferencia y COMMIT en orden (3); autoridad en Inventario (1); efectos posteriores identificados (1) |
| c | 5 | Dos beneficios contrastables (2); dos costos causales (2); conexión con la contención observada (1) |
| d | 4 | D coherente con garantía y costo (2); p99 basal/objetivo a igual carga (2) |

**Variantes válidas:** límites de concurrencia, caché o precálculo interno. **No se exige outbox** en esta pregunta.

## Errores típicos

- Extraer Inventario o Pagos "porque son críticos" (los datos dicen que el problema es Popularidad).
- Saltar directo a microservicios sin probar la medida interna.
- Poner Popularidad dentro del camino crítico.
- Beneficios no medibles ("más escalable", "más moderno").

---

# 5. Pregunta 2 — Confirmación y evento durable (20 puntos)

## Qué plantea

Venta, Inventario y Pagos son servicios con bases propias. La transferencia ya se confirmó en Inventario y Venta marca la compra como CONFIRMADA. Venta debe avisar `CompraConfirmada(intent_id, ticket_id, event_id)`. Hoy hace **COMMIT en su base y después publica** al broker, y **cae justo entre ambos pasos**. Otro propone **publicar primero y después COMMIT**, y en ese caso Venta cae o aborta tras publicar. El relay y los consumidores también pueden caer y reintentar; **no hay exactly-once**.

Pide: (a) tabla de dos filas comparando ambos órdenes; (b) diseño con outbox en ≤ 6 pasos; (c) cómo Popularidad evita aplicar dos veces el aviso y qué atraso puede ver Sofía; (d) D + métrica operacional.

**Temas:** Clase 7 y 9 (dual write, transactional outbox, relay, inbox/idempotencia del consumidor).

## Cómo razonar

**Paso 1 — Reconocer el patrón: es un *dual write*.** Dos escrituras en sistemas independientes (base de Venta y broker) sin transacción común. **Ningún orden lo resuelve**, y la tabla tiene que mostrar que cada orden falla **de forma distinta**.

**Paso 2 — Mover el crash en cada orden y preguntar qué queda en cada lugar:**
- *Base primero:* la base dice CONFIRMADA, el broker no tiene nada → la vista derivada **omite una compra real**.
- *Broker primero:* el broker tiene el evento, la base no confirmó → la vista **afirma una confirmación falsa**.

**Paso 3 — Decir quién es la autoridad:** Venta es la autoridad de CONFIRMADA. Lo que diga Popularidad es una vista.

**Paso 4 — Aplicar outbox:** meter el estado y el evento pendiente **en la misma transacción local** de Venta. Así o existen ambos o ninguno. Después un relay publica.

**Paso 5 — Pensar qué falla queda abierta:** el relay puede publicar, caer antes de marcar la fila como enviada y **volver a publicar**. → Hay **duplicados** → el consumidor tiene que ser idempotente (event_id único + efecto en la misma transacción).

**Paso 6 — Semántica del evento:** `CompraConfirmada` **afirma un hecho** (un COMMIT de Venta ya ocurrió), **no ordena** confirmar nada.

## Respuesta esperada

**a)** Venta es autoridad de CONFIRMADA; Inventario ya lo fue de la transferencia.

| Orden | Venta | Broker | Vista derivada |
| --- | --- | --- | --- |
| A: base → broker | CONFIRMADA | Sin evento | **Omite** una compra real |
| B: broker → base | **No** CONFIRMADA | Evento visible | Puede afirmar una **confirmación falsa** |

**b)**
1. Venta registra CONFIRMADA **y** una fila en la outbox con `event_id` estable, **en una transacción local**.
2. Hace COMMIT.
3. El relay lee las filas pendientes.
4. Publica el evento.
5. Tras el ACK del broker, registra el progreso (marca la fila).
6. Si cae antes de registrar progreso, al reanudar **vuelve a publicar el mismo evento** (mismo `event_id`).

`CompraConfirmada` afirma un COMMIT de Venta; no ordena confirmar.

**c)** Popularidad **inserta el `event_id` (único) y actualiza el conteo en una misma transacción local**; recién después reconoce el mensaje. Un duplicado falla por unicidad y **no vuelve a incrementar**. Limitación: la compra de Sofía puede estar confirmada mientras **la vista o el aviso siguen atrasados**.

**d)** D = (P: doble escritura; F: caída o reentrega; G: evento fiel a Venta y actualización reintentable; S: outbox y deduplicación; T: duplicados y lag). **Métrica:** p99 desde el COMMIT de Venta hasta la aplicación en Popularidad, con SLO propuesto < 30 s.

## Cómo se reparten los puntos

| Inciso | Puntos | Qué tiene que aparecer |
| --- | --- | --- |
| a | 5 | A: hecho real omitido en la vista (2); B: evento sin confirmación de Venta (2); autoridad de confirmación local (1) |
| b | 7 | Estado y outbox en la misma transacción (3); relay que reanuda y republica el mismo ID (2); semántica del evento (1); sin transacción global (1) |
| c | 4 | Registro del ID y cambio de la vista atómicos (3); reconoce posible atraso (1) |
| d | 4 | D coherente (2); métrica con inicio, fin y umbral (2) |

**Variantes válidas:** cursor del relay o marca de envío, si se tolera repetición.

## Errores típicos (la pauta los marca explícitamente)

- **Suponer exactly-once** ("el relay publica una sola vez"): no se concede.
- **Registrar el ID antes del efecto en transacciones separadas**: si cae en el medio, **se pierde el aviso** (el ID dice "procesado" pero el conteo no cambió).
- Proponer "una transacción que abarque base y broker" (no existe).
- Decir que el evento "manda" a Popularidad a hacer algo (es un hecho, no un comando).

---

# 6. Pregunta 3 — Pago incierto y reserva vencida (25 puntos, la más pesada)

## Qué plantea (línea de tiempo)

| Tiempo | Evento |
| --- | --- |
| t = 0 | Inventario reserva A17 para el intento `i-42` (vence en t = 90) |
| t = 64 | Venta pide a Pagos la autorización. **Pagos la registra, pero la respuesta se pierde** |
| t = 90 | Inventario da por **vencida** la reserva |
| t = 92 | **Otra compradora** obtiene A17 válidamente |
| t = 94 | Venta observa el **timeout**; puede consultar el resultado durable en Pagos |

No se capturó dinero. Pide: (a) hasta 7 estados y 8 transiciones (camino normal + estados inciertos), con autoridad y evidencia durable; (b) qué sabe y qué no sabe Venta tras el timeout y qué hace, usando `i-42`; (c) resultado para Sofía, reparación, su falla posible y cuándo el intento es terminal, y por qué no se puede transferir con la reserva vieja; (d) D + métrica de intentos atascados.

**Temas:** Clase 1–2 (timeout ≠ falla, indistinguibilidad, idempotencia), Clase 9 (saga, estados inciertos, compensación), Clase 5 (no hay commit global).

## Cómo razonar

**Paso 1 — Reconocer que es una saga.** Tres autoridades, sin transacción global → Venta coordina con **estados durables**. Hay que modelar el camino feliz **y** los estados de incertidumbre.

**Paso 2 — ¿Qué sabe Venta en t = 94?** Solo que **no recibió respuesta**. Por la Clase 1, hay dos historias indistinguibles: Pagos nunca recibió el pedido, o lo autorizó y se perdió la respuesta. → **Timeout no es rechazo.** Por eso hace falta un estado **PAGO_INCIERTO** y la acción correcta es **consultar** a Pagos por la clave `i-42:autorizar` (o reintentar **con la misma clave**, que no crea otra autorización lógica).

**Paso 3 — Resultado de la consulta: Pagos dice APROBADA.** Pero la reserva **venció** en t = 90 y A17 ya es de otra persona desde t = 92.

**Paso 4 — ¿Puedo transferir?** No. La autoridad es Inventario, que verifica vigencia y transfiere **atómicamente**: con la reserva vencida, **rechaza**. Y la reserva nueva de la otra compradora **no le da nada a Sofía**. Transferir igual violaría el invariante (dos titulares).

**Paso 5 — ¿Cómo reparo?** Hay una retención en la tarjeta de Sofía que ya no corresponde → **compensar**. Como no hubo captura, **se anula, no se reembolsa**. La anulación es **otra operación → otra clave**: `i-42:anular`.

**Paso 6 — ¿Y si la anulación falla o se pierde su respuesta?** Igual que antes: consultar o reintentar con `i-42:anular`, y **quedarse en ANULACION_PENDIENTE** hasta tener evidencia durable de la anulación. Recién ahí el intento es **terminal (fallo)**.

**Paso 7 — Métrica:** intentos atascados en estados intermedios por demasiado tiempo.

## Respuesta esperada

**a)** Siete estados: `INICIADO`, `RESERVADO`, `PAGO_INCIERTO`, `AUTORIZADO`, `TRANSFERIDO`, `ANULACION_PENDIENTE`, `TERMINADO` (éxito/fallo).

| Origen | Destino | Autoridad y evidencia durable |
| --- | --- | --- |
| INICIADO | RESERVADO | Inventario: hold vigente |
| RESERVADO | PAGO_INCIERTO | Venta: clave durable persistida; envía a Pagos |
| PAGO_INCIERTO | AUTORIZADO | Pagos: aprobación durable (respuesta o consulta) |
| PAGO_INCIERTO | TERMINADO (fallo) | Pagos: rechazo durable, sin retención |
| AUTORIZADO | TRANSFERIDO | Inventario: vigencia + transferencia atómica |
| AUTORIZADO | ANULACION_PENDIENTE | Inventario rechaza; Venta persiste pendiente y ordena anular |
| ANULACION_PENDIENTE | TERMINADO (fallo) | Pagos: anulación durable; Venta registra fallo |
| TRANSFERIDO | TERMINADO (éxito) | Venta: CONFIRMADA durable; responde a Sofía |

> Fijate que Venta pasa a `PAGO_INCIERTO` **antes** de enviar el pedido a Pagos: así, si Venta misma se cae, al reiniciar sabe que tiene que consultar.

**b)** En t = 94 Venta **no conoce** el resultado porque la respuesta se perdió; sabe que mandó el pedido, no si se ejecutó. **Consulta** a Pagos por `i-42:autorizar`; si tiene que reintentar, usa **la misma clave**, así nunca se crea una segunda autorización lógica. Pagos devuelve APROBADA (durable).

**c)** La reserva de Sofía **venció**; la reserva nueva de otra compradora tampoco le concede titularidad. Inventario **debe rechazar** la transferencia por el hold vencido. Venta **no confirma** y solicita `i-42:anular`. Si la anulación falla o se pierde la respuesta, consulta o reintenta **con esa clave** y **permanece pendiente**. Solo declara **fallo terminal** cuando Pagos acredita la anulación durable. **No corresponde reembolso** porque no hubo captura. (Y en el camino normal, si Venta cae después de transferir, consulta a Inventario y completa su COMMIT antes de responder éxito.)

**d)** D = (P: aprobación sin hold vigente; F: respuesta perdida y vencimiento; G: titularidad única sin retención huérfana; S: consulta y anulación; T: espera y conciliación). **Métrica:** cantidad de intentos en `PAGO_INCIERTO` o `ANULACION_PENDIENTE` por más de 5 minutos.

## Cómo se reparten los puntos

| Inciso | Puntos | Qué tiene que aparecer |
| --- | --- | --- |
| a | 7 | Camino normal hasta el COMMIT de Venta (3); incertidumbre durable (2); rama de vencimiento y reparación (2) |
| b | 6 | Timeout no es rechazo (2); consulta durable por clave (2); misma clave al reintentar la autorización (2) |
| c | 7 | No transferir ni confirmar con hold vencido (2); anulación con **otra** clave (2); pendiente hasta evidencia durable (2); no reembolso (1) |
| d | 5 | D completo y coherente (3); métrica de pendientes con umbral (2) |

**Variantes válidas:** otros nombres de estados y recuperación por consulta; no se exige un estado TRANSFERIDO separado si se explica cómo reconstruir una transferencia previa al COMMIT de Venta.

## Errores típicos

- Tratar el timeout como "el pago falló" y empezar un intento nuevo (→ **doble autorización**).
- Reintentar la autorización **con una clave nueva**.
- Usar la misma clave para autorizar y anular.
- Transferir A17 "porque Sofía pagó" (viola el invariante; la autoridad es Inventario).
- Hablar de **reembolso** (no hubo cobro: es **anulación**).
- Declarar el intento terminado sin evidencia durable de la anulación.

---

# 7. Pregunta 4 — Investigar un incidente sin adivinar la causa (20 puntos)

## Qué plantea

A las **21:05** se publica una versión que **reduce el timeout de la llamada a Pagos de 2,0 a 0,8 s**. A las 21:10:

| Señal | Antes | Después |
| --- | --- | --- |
| Intentos de compra por segundo | 50 | 50 (la tasa no cambió) |
| Errores visibles para el usuario | 0,3% | **8%** |
| p99 de compra | 1,2 s | **10 s** |
| p95 de autorización en Pagos (medido en Pagos) | 0,7 s | **1,8 s** |
| Llamadas a Pagos por intento | 1,1 | **2,5** |
| Uso del pool de conexiones de Venta | 40% | **100%** |
| CPU de la base de Venta | 43% | 45% |
| Edad de la fila outbox más antigua | < 5 s | **150 s** |

Las trazas fallidas muestran timeouts y reintentos, pero **no prueban** qué componente empezó. La base no está saturada.

Pide: (a) separar síntomas, hipótesis de causa próxima y mecanismo de amplificación, con **dos hipótesis rivales**, sin declarar causa definitiva; (b) una métrica, una traza y un log para discriminarlas; (c) mitigación inmediata + comprobación de que el usuario se recuperó, distinguiendo mitigación de corrección; (d) D + métrica.

**Temas:** Clase 10 (síntoma vs. causa, core analysis loop, métricas/trazas/logs, cardinalidad), Clase 11 (timeouts, reintentos que amplifican, pool agotado), Clase 12 (mitigar revirtiendo).

## Cómo razonar

**Paso 1 — Clasificar las señales.** Errores, p99, pool al 100% y edad de la outbox son **síntomas**: describen el efecto, no la causa. La CPU de la base estable **descarta** a la base.

**Paso 2 — Ver el mecanismo de amplificación.** Las llamadas a Pagos por intento pasaron de **1,1 a 2,5**: hay **reintentos**. Con la misma tasa de entrada, más llamadas → más carga en Pagos y más tiempo ocupando el pool de Venta (Little: sube W → sube L → pool al 100%). Y si el pool está lleno, también se atrasa el relay de la outbox. Esto ocurre **en ambas hipótesis**, por eso no prueba cuál fue el origen.

**Paso 3 — Formular dos hipótesis rivales compatibles con los datos:**
- **H1:** el **nuevo timeout de 0,8 s es demasiado corto** (el p95 normal de Pagos era 0,7 s, así que el p99 seguramente supera 0,8 s): corta respuestas que iban a llegar → reintentos → más carga → Pagos se pone más lento → más timeouts. **El deploy desencadenó la espiral.**
- **H2:** **Pagos o el proveedor se volvió lento por otra causa** independiente, coincidiendo con el deploy; los reintentos solo amplifican.

**Paso 4 — ¿Qué observación las distingue?** La clave es mirar **el primer intento** (no los reintentos) **a lo largo del tiempo**:
- Si la latencia del **primer intento** era estable al principio y **empezó a subir después** de que crecieran los reintentos → apoya **H1** (los reintentos causaron la lentitud).
- Si la latencia del primer intento **subió antes** o independientemente del corte a 0,8 s → apoya **H2**.

Para eso hacen falta: una **métrica** por número de intento (sin IDs como etiquetas, Clase 10), una **traza** que muestre espera por el pool, el span de Pagos, el corte a 0,8 s y el reintento, y un **log estructurado** con el ordinal del intento, el timeout aplicado y la duración (sin datos de tarjeta).

**Paso 5 — Mitigar sin esperar a saber la causa (Clase 12: primero mitigar).** La mitigación más rápida y reversible es **revertir el último cambio** (el timeout) y **acotar los reintentos** con la misma clave. Comprobar la recuperación **desde el usuario** y **a igual carga**. La corrección permanente viene después y depende de qué hipótesis se confirme.

## Respuesta esperada

**a)** Síntomas: errores 0,3% → 8%, p99 1,2 → 10 s, pool 40% → 100% y outbox < 5 → 150 s. **H1:** el timeout de 0,8 s corta respuestas y desencadena reintentos. **H2:** el proveedor o Pagos se volvió lento en las primeras llamadas por un factor independiente. En ambas, el paso de **1,1 → 2,5 llamadas por compra amplifica** carga y espera; **la causa inicial no está probada**.

**b)**
- **Métrica:** p95 de duración en Pagos **del primer intento**, por cohorte/tiempo, antes y después de 21:05, **sin IDs como etiquetas**. Estable al comienzo y creciente después con los reintentos → favorece **H1**; alza previa en los primeros intentos → favorece **H2**.
- **Traza:** comparar espera por el pool de Venta, span de Pagos, corte a 0,8 s y reintento; la secuencia discrimina.
- **Log:** `trace_id`, `intent_id`, versión, **ordinal del intento**, timeout, duración y resultado, **sin tarjeta ni token**; contrastar los tiempos de los primeros fallos con los reintentos.

**c)** **Mitigación:** revertir el timeout y acotar los reintentos con clave estable. **Comprobación:** a 50 compras/s, errores y p99 vuelven cerca de 0,3% y 1,2 s. **Corrección permanente (después):** ajustar timeout y reintentos con latencias medidas y, si se confirma H2, corregir Pagos o el proveedor.

**d)** D = (P: compra lenta/fallida; F: timeout, demora y reintento; G: recuperar la compra correcta; S: rollback, reintentos acotados, diagnóstico; T: más espera y observabilidad). **Métrica:** errores visibles próximos al 0,3% basal con 50 compras/s.

## Cómo se reparten los puntos

| Inciso | Puntos | Qué tiene que aparecer |
| --- | --- | --- |
| a | 5 | Separar síntomas de causas (1); H1 y H2 compatibles (2); amplificación y causalidad incierta (2) |
| b | 7 | Métrica de primeros intentos con dimensión y tiempo (2); traza de pool y Pagos (2); log con ordinal/resultado sin datos sensibles (2); predicciones que diferencian (1) |
| c | 4 | Mitigación reversible y reintentos limitados (2); recuperar la experiencia a igual carga (1); corrección posterior condicional (1) |
| d | 4 | D completo (3); métrica de recuperación (1) |

> La pauta advierte: **el p95 agregado de Pagos o la edad de la outbox no prueban por sí solos quién inició la falla.**

## Errores típicos

- Declarar "la causa fue el timeout" como certeza (se pide **no** adivinar).
- Culpar a la base (su CPU no cambió).
- Proponer una métrica con `intent_id` o `user_id` como etiqueta (**explosión de cardinalidad**).
- Loguear datos de tarjeta.
- Proponer "agregar más reintentos" o "más instancias" sin entender la amplificación.
- No distinguir mitigación (revertir ahora) de corrección (ajustar después).

---

# 8. Pregunta 5 — Partición regional y migración controlada (15 puntos)

## Qué plantea

Inventario tiene **una sola autoridad de escritura en R1**. R2 tiene **copias de lectura** del catálogo y de Popularidad (pueden estar atrasadas). Durante **70 s se corta** el enlace R2–R1; R1 sigue funcionando. Una nueva compradora en R2 ve A17 en el catálogo y pulsa comprar; **no tiene reserva previa**. Después de recuperar el enlace, se quiere probar una nueva Popularidad mandándole el **5% de las lecturas de R2** (canary), con la versión vieja disponible. Objetivos: sin doble titularidad; errores < 0,5% con el camino disponible; p99 < 2 s; atraso de Popularidad < 30 s.

Pide: (a) qué puede mostrar R2 y qué responde a la compra, con garantía y costo; (b) límite de espera, reintentos acotados, qué hace el breaker y por qué la misma clave; (c) dos comprobaciones para ampliar o revertir el canary y cómo volver al camino viejo sin deshacer compras; (d) D + métrica.

**Temas:** Clase 1 y 8 (CAP: elegir consistencia), Clase 11 (deadline, backoff, jitter, circuit breaker), Clase 9 (canary / Strangler Fig), Clase 7 (event_id para comparar).

## Cómo razonar

**Paso 1 — Es CAP en estado puro.** R2 está particionada de la autoridad. Si R2 "vende" para estar disponible, puede haber **doble titular** (la misma situación que la entrada VIP de la Clase 8). El invariante gana → **elegir consistencia**: R2 **no confirma**.

**Paso 2 — Separar lectura de escritura.** Mostrar datos es seguro si se aclara que **pueden estar atrasados**: una oferta visible en caché **no concede** la butaca (Clase 9). Comprar requiere a la autoridad.

**Paso 3 — Resiliencia de la llamada a R1.** No esperar los 70 s: un **plazo total acotado** (dentro del p99 objetivo de 2 s), **pocos reintentos con backoff y jitter**, siempre **con la misma clave** (`id-intento:reservar`) para no crear dos reservas. Si sigue fallando, el **breaker abre** y rechaza rápido; luego **prueba con pocas llamadas** (half-open). Si el resultado de un intento quedó incierto, al sanar **consultar** a Inventario por la clave.

**Paso 4 — Canary reversible.** Se amplía solo si pasa comprobaciones **funcionales** (¿cuenta bien?) y **de atraso** (< 30 s). Para comparar conteos sin trampas, usar los eventos `CompraConfirmada` con `event_id` único **al mismo corte**. Revertir = que el **router devuelva las lecturas** a la versión vieja. **El canary afecta lecturas de Popularidad, no titularidad**: revertirlo no deshace ninguna compra.

## Respuesta esperada

**a)** R2 puede **mostrar** la oferta y el conteo de Popularidad **indicando posible atraso**, nunca disponibilidad garantizada. A la nueva compra, sin reserva y sin comunicación con Inventario en R1, responde **temporalmente indisponible**: no autoriza pago ni confirma. Preserva la **titularidad única** a costo de **menor disponibilidad** durante la partición.

**b)** Plazo total de **como máximo 2 s** (no esperar los 70 s); **pocos reintentos** de `id-intento:reservar` con **backoff y jitter**, siempre con **la misma clave**. Si la falla persiste, el **breaker abre** y rechaza rápido; luego **prueba con pocas llamadas**. Si un resultado fue incierto, **consulta** a Inventario por la clave al sanar el enlace. No confirma automáticamente; puede rechazar fallas transitorias.

**c)** **Comprobación 1:** comparar ambos conteos contra `CompraConfirmada` con `event_id` único al mismo corte de eventos. **Comprobación 2:** atraso de la vista nueva < 30 s en operación normal. Si ambas pasan, ampliar el 5%; si no, el **router devuelve las lecturas** a la versión anterior. Costo: mantener dos proyecciones; redirigir lecturas **no revierte compras**.

**d)** D = (P: lectura y compra regional; F: partición y retraso de la proyección; G: dueño único y compra correcta cuando hay camino; S: confirmar solo en R1, breaker y canary reversible; T: menor disponibilidad y lag). **Métrica adicional:** p99 de compra en R2 < 2 s, por cohorte, tras sanar el enlace.

## Cómo se reparten los puntos

| Inciso | Puntos | Qué tiene que aparecer |
| --- | --- | --- |
| a | 5 | Copias obsoletas y oferta no autoritativa (2); sin reserva no se confirma (2); costo visible (1) |
| b | 4 | Deadline y reintentos acotados con backoff/jitter (1); breaker y prueba limitada (1); clave constante (1); consultar si el resultado es incierto (1) |
| c | 3 | Comparación funcional del conteo (1); lag < 30 s (1); revertir la ruta de lecturas sin deshacer ventas (1) |
| d | 3 | D con garantía y costo (2); métrica extra de p99 con contexto (1) |

**Variante válida:** responder "pendiente" en lugar de "indisponible" **solo si** el intento pudo llegar a la autoridad y se evita una confirmación tardía inesperada.

## Errores típicos

- Dejar que R2 venda "y reconciliar después" (con un recurso único no hay merge válido: Clase 8).
- Decir que la butaca está "disponible" porque aparece en el catálogo de R2.
- Esperar 70 s o reintentar sin límite.
- Usar claves distintas en cada reintento.
- Pensar que revertir el canary deshace compras.
- Comparar los conteos de las dos Popularidades en momentos distintos.

---

# 9. Resumen: qué temas entran y cómo se conectan

| Concepto de la materia | Dónde aparece en el simulacro |
| --- | --- |
| Indistinguibilidad, timeout ≠ falla (Clase 1–2) | P3 (pago incierto), P5 (resultado incierto tras la partición) |
| Idempotencia con clave estable (Clase 2) | P3 (`i-42:autorizar` / `i-42:anular`), P5 (`reservar`), P4 (reintentos) |
| CAP, elegir consistencia bajo partición (Clase 1, 8) | P5 |
| Autoridad e invariante (Clase 8, 9) | Todas: Inventario decide A17 |
| Outbox, dual write, consumidor idempotente (Clase 7, 9) | P2, P5 (event_id para comparar) |
| Saga, estados inciertos, compensación (Clase 9) | P3 |
| Monolito modular vs. extraer servicio, camino crítico (Clase 9) | P1 |
| Canary / migración reversible (Clase 9) | P5 |
| Síntoma vs. causa, métrica/traza/log, cardinalidad (Clase 10) | P4 |
| Timeouts, reintentos que amplifican, backoff, jitter, breaker (Clase 11) | P4, P5 |
| Mitigar primero, revertir, comprobar desde el usuario (Clase 12) | P4 |
| p99, percentiles, métricas con umbral (Clase 10, 12) | Todas las líneas de métrica |

## Las cinco ideas que más puntos dan

1. **La autoridad decide; las copias muestran.** Ninguna vista, caché o región aislada concede titularidad.
2. **Timeout no es rechazo: se consulta por la clave.** Reintentar siempre con la misma clave; operaciones distintas, claves distintas.
3. **No hay transacción global: lo que tiene que ir junto va en la misma transacción local** (estado + outbox; event_id + efecto).
4. **Lo que ya ocurrió se compensa, no se borra** (anular la retención, no "hacer rollback").
5. **Toda decisión se cierra con D = (P, F, G, S, T) y una métrica con umbral, medida a igual carga.**
