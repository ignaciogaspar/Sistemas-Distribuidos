| Sistemas | Distribuidos  |              |             |             |     |
| -------- | ------------- | ------------ | ----------- | ----------- | --- |
| Clase    | 08 — De Kafka | y causalidad | a consenso, | CAP, Dynamo | y   |
Spanner
| Departamento   | de Computación         |           |     |     |     |
| -------------- | ---------------------- | --------- | --- | --- | --- |
| Ingeniería     | Informática – Comisión | A – Pilar |     |     |     |
| Equipo docente | – 2026                 |           |     |     |     |

| La historia | que guiará | toda | la clase |     |
| ----------- | ---------- | ---- | -------- | --- |
NÚCLEO
EnestaclaseseguiremoseldestinodeunaúnicaentradaVIPqueSofíayTomásintentancomprarcasialmismotiempo
desderegionesdiferentes.
| Lo que necesitamos | comprender |     | La pregunta                                  | transversal |
| ------------------ | ---------- | --- | -------------------------------------------- | ----------- |
| ▶                  |            |     | ¿Quéerroresinaceptableyquécostoaceptamospara |             |
quéeventosestánrelacionadosycuálesson
evitarlo?
concurrentes;
▶ cómovariasréplicasacuerdanunadecisión;
| ▶                                    |     |     | latencia·indisponibilidad·coordinación· |            |
| ------------------------------------ | --- | --- | --------------------------------------- | ---------- |
| quéocurrecuandolaredimpidecoordinar; |     |     |                                         | reparación |
▶ quécostopagacadagarantía.
SistemasDistribuidos–Clase08 2/83

| Evidencias | de aprendizaje |     |
| ---------- | -------------- | --- |
NÚCLEO
| Durante | la clase | Al cerrar |
| ------- | -------- | --------- |
▶ construirhappened-beforeyreconocerconcurrencia; ▶ distinguirmayoríadeconsensodequorum
| ▶   |     | Dynamo-like; |
| --- | --- | ------------ |
completarrelojesdeLamportyvectoriales;
▶
▶ seguirunaentradaRaftypredeciruncrashouna decidircuándounconflictopuedefusionarse;
▶
| partición. |     | explicarTrueTimeydefenderunapolíticaconfalla, |
| ---------- | --- | --------------------------------------------- |
costoymétrica.
Laevidenciaseráreconstruirunaejecuciónyjustificarporquéelsistemaconfirma,espera,rechaza,conservaversiones
orepara.
SistemasDistribuidos–Clase08 3/83

| Puente | desde | Kafka |
| ------ | ----- | ----- |
Sistemas Distribuidos

| Repaso | visual: un | Mundial | en tiempo | real |     |
| ------ | ---------- | ------- | --------- | ---- | --- |
NÚCLEO
| PartidoA | GoalScored |           | Kafka         | poll       | Marcador |
| -------- | ---------- | --------- | ------------- | ---------- | -------- |
|          |            | Productor |               | Consumidor |          |
|          | eventos    |           | logpreservado |            | web      |
Idea recuperada
Kafka preserva una secuencia de registros para que productores y consumidores avancen de manera
| desacoplada | y recuperable. |     |     |     |     |
| ----------- | -------------- | --- | --- | --- | --- |
SistemasDistribuidos–Clase08 5/83

| Una | única secuencia | no  | alcanza para | muchos partidos |     |
| --- | --------------- | --- | ------------ | --------------- | --- |
NÚCLEO
| Problema | de capacidad      |             | Problema   | de orden              |                |
| -------- | ----------------- | ----------- | ---------- | --------------------- | -------------- |
|          |                   |             | Distribuir | eventos al azar puede | mostrar un gol |
| Muchos   | partidos compiten | por la CPU, | red y      |                       |                |
almacenamiento de una única secuencia. antes de que comience su partido.
PartidoA
unasolasecuencia
PartidoB
yunconsumidor
PartidoC
SistemasDistribuidos–Clase08 6/83

| La key define | el ámbito | de orden |
| ------------- | --------- | -------- |
NÚCLEO
Decisión
Usamosgame_idcomokey: todosloseventosdeunpartidopermanecenenlamismapartición.
|     | P0  | A:Started→YellowCard   |
| --- | --- | ---------------------- |
|     | P1  | B:Started→Substitution |
C:Started→Goal
P2
Ganamosordenlocalporpartidoyparalelismoentrepartidos. NoobtenemosunoffsetglobalgratuitoentreP0,P1y
P2.
SistemasDistribuidos–Clase08 7/83

| Consumer | groups: | repartir | trabajo | y releer | la misma | historia |
| -------- | ------- | -------- | ------- | -------- | -------- | -------- |
NÚCLEO
| Grupo live-score |         |     |     | Grupo analytics      |          |                   |
| ---------------- | ------- | --- | --- | -------------------- | -------- | ----------------- |
|                  |         |     |     | Lee nuevamente       | P0, P1 y | P2, pero conserva |
| ▶ P0 →           | Score-1 |     |     |                      |          |                   |
|                  |         |     |     | sus propios offsets. |          |                   |
| ▶ P1 →           | Score-2 |     |     |                      |          |                   |
▶ →
| P2              | Score-3     |                   |     |     |     |     |
| --------------- | ----------- | ----------------- | --- | --- | --- | --- |
| Las particiones | se reparten | dentro del grupo. |     |     |     |     |
Unaparticióntieneunasolainstanciaactivadentrodeungrupo; gruposdiferentespuedenleer
| la misma | partición con | progreso independiente. |     |     |     |     |
| -------- | ------------- | ----------------------- | --- | --- | --- | --- |
SistemasDistribuidos–Clase08 8/83

| Crash, replay e | idempotencia |     |     |
| --------------- | ------------ | --- | --- |
NÚCLEO
| poll | actualiza | reinicia | deduplica |
| ---- | --------- | -------- | --------- |
CRASH
| P2:420 | marcador | desde420 | event_id |
| ------ | -------- | -------- | -------- |
Límite
At-least-once permite que el registro reaparezca; no garantiza por sí solo que el efecto lógico ocurra
una única vez.
SistemasDistribuidos–Clase08 9/83

| Algunas | métricas | a considerar | en Kafka |     |     |
| ------- | -------- | ------------ | -------- | --- | --- |
NÚCLEO
| Definiciones |     |     | Guardraildocente | Referencia |     |
| ------------ | --- | --- | ---------------- | ---------- | --- |
| ▶            |     |     | mensajehabitual  | <1MB       |     |
RF=3: líder+dosseguidoresconfigurados. datosretenidos/broker ∼1TB
| ▶   |     |     | ingresoconservador/broker | <10MB/s |     |
| --- | --- | --- | ------------------------- | ------- | --- |
ISR:réplicassuficientementesincronizadas.
▶
| minISR=2:   | hacenfaltaalmenosdosréplicasenISR.      |     |                                   |                      |          |
| ----------- | --------------------------------------- | --- | --------------------------------- | -------------------- | -------- |
| ▶ acks=all: | todaslasréplicasdelISRactualconfirmanel |     |                                   |                      |          |
| batch.      |                                         |     | Son presupuestos                  | docentes, no máximos | univer-  |
|             |                                         |     | sales. Debenvalidarseconhardware, |                      | tamañode |
registros,replicaciónypruebasdecarga.
| Si Réplica3    | sale del ISR, pero | Réplica1 y Réplica2 siguen |      |     |     |
| -------------- | ------------------ | -------------------------- | ---- | --- | --- |
| sincronizadas, | la escritura       | continúa. Si solo queda    |      |     |     |
| Réplica1,      | se rechaza: no se  | reconoce éxito con una     | sola |     |     |
copiasincronizada.
|     |     | SistemasDistribuidos–Clase08 |     |     | 10/83 |
| --- | --- | ---------------------------- | --- | --- | ----- |

Puente: Kafka ordena registros; no elige por sí solo una
decisión autoritativa
NÚCLEO
Kafka aporta La nueva pregunta
SidosregionesintentanreservarlaúnicaentradaVIP,¿qué
▶
ordendentrodecadapartición; comandodebeconvertirseenladecisiónautoritativa?
▶
replayyprogresoporgrupo;
▶
replicaciónlíder–seguidor;
▶
desacoplamientoproductor–consumidor.
Pasamosdepreservarunahistoriaaacordarquéhistoriaseráválida.
SistemasDistribuidos–Clase08 11/83

| Por qué  | estos | conceptos | se  |
| -------- | ----- | --------- | --- |
| usan hoy |       |           |     |
Sistemas Distribuidos

Sistema real 1: Kubernetes y etcd
CASOREAL
Qué es Kubernetes
Plataforma que administra aplicaciones en contenedores y mantiene un estado deseado del cluster.
Qué es etcd
Almacén clave–valor distribuido donde el control plane conserva objetos y configuración autoritativa.
etcdutilizaRaftparaacordarunlog;sinmayoríanopuedeconfirmarnuevoscambiosdelcontrol
plane.
SistemasDistribuidos–Clase08 13/83

| Sistema | real 2: Amazon | Dynamo |
| ------- | -------------- | ------ |
CASOREAL
| Qué fue | Dynamo |     |
| ------- | ------ | --- |
Almacén clave–valor altamente disponible diseñado por Amazon para servicios que debían continuar
| respondiendo | durante fallas. |     |
| ------------ | --------------- | --- |
| Decisión     | de diseño       |     |
Conservar versiones concurrentes, responder con quórums flexibles y reparar después, en vez de
| bloquear siempre | por coordinación | global. |
| ---------------- | ---------------- | ------- |
La disponibilidad se paga con versiones, reconciliación, hints y trabajo de reparación.
SistemasDistribuidos–Clase08 14/83

| Sistema | real 3: | Google Spanner |
| ------- | ------- | -------------- |
CASOREAL
Qué es Spanner
Base de datos distribuida globalmente que ofrece transacciones entre grupos de datos y un orden
| compatible | con lo observado | por los clientes. |
| ---------- | ---------------- | ----------------- |
| Decisión   | de diseño        |                   |
Combinar consenso localizado, coordinación transaccional y TrueTime, una abstracción que expone
| incertidumbre | temporal. |     |
| ------------- | --------- | --- |
Spanner paga coordinación y espera temporal para ofrecer garantías globales más fuertes.
SistemasDistribuidos–Clase08 15/83

| Lo que | un ingeniero | decide | realmente |     |
| ------ | ------------ | ------ | --------- | --- |
NÚCLEO
| Operación | Invariante |     | Errortolerable | Evidencia |
| --------- | ---------- | --- | -------------- | --------- |
últimaentrada confirmaciones≤1 rechazotemporal dobleventa,p99
carrito noperderintención ramasfusionables conflictos, convergen-
cia
| ranking | atrasoacotado |     | vistaanterior | lagtemporal |
| ------- | ------------- | --- | ------------- | ----------- |
ledger(registrosconta- nocreardinero esperaorechazo anomalías,abortos
bles)
Lapreguntanoes«¿quétecnologíaesmejor?»,sino«¿quéerroresinaceptableparaestaoperación?».
SistemasDistribuidos–Clase08 16/83

| Mapa | de la | clase |     |     |     |
| ---- | ----- | ----- | --- | --- | --- |
NÚCLEO
| 1. Observar:  | fallas     | parciales | y           | conocimiento | local.     |
| ------------- | ---------- | --------- | ----------- | ------------ | ---------- |
| 2. Describir: | causalidad |           | y relojes   | lógicos.     |            |
| 3. Decidir:   | máquinas   |           | de estado,  | Raft y       | Paxos.     |
| 4. Elegir     | durante    | una       | partición:  | CAP          | y PACELC.  |
| 5. Conservar  |            | o evitar  | conflictos: | Dynamo       | y Spanner. |
SistemasDistribuidos–Clase08 17/83

| Modelo | mínimo | y caso |
| ------ | ------ | ------ |
conductor
Sistemas Distribuidos

Modelo mínimo: qué contiene una réplica
NÚCLEO
proceso
RAM
RéplicaN1
estado
persistente
red
Un nodo puede estar sano localmente y, aun así, ignorar qué ocurrió en otra región.
SistemasDistribuidos–Clase08 19/83

| Tres situaciones | que no debemos | confundir |     |
| ---------------- | -------------- | --------- | --- |
NÚCLEO
| Sano y comunicado | Sano pero | aislado | Caído |
| ----------------- | --------- | ------- | ----- |
Ejecuta y recibe mensajes. Ejecuta, pero sus mensajes no No ejecuta pasos ni responde.
|     | cruzan a | tiempo. |     |
| --- | -------- | ------- | --- |
Desde otro nodo, un timeout puede ser compatible con cualquiera de estas historias.
SistemasDistribuidos–Clase08 20/83

| Timeout: | observación | local, no | diagnóstico |
| -------- | ----------- | --------- | ----------- |
NÚCLEO
request
A B
¿respuesta?
| ▶ la solicitud | nunca llegó; |     |     |
| -------------- | ------------ | --- | --- |
▶
| B está      | lento o pausado; |            |     |
| ----------- | ---------------- | ---------- | --- |
| ▶ B ejecutó | y la respuesta   | se perdió; |     |
▶
| la red | está particionada. |     |     |
| ------ | ------------------ | --- | --- |
El timeout solo prueba que A no obtuvo respuesta dentro de su plazo.
SistemasDistribuidos–Clase08 21/83

| Caso | conductor: | Festival | Austral | Global |     |
| ---- | ---------- | -------- | ------- | ------ | --- |
NÚCLEO
| Recurso       |      |     |     | Servicio replicado      |              |
| ------------- | ---- | --- | --- | ----------------------- | ------------ |
| VIP-01, stock | = 1. |     |     |                         |              |
|               |      |     |     | Cinco réplicas: N1, N2, | N3, N4 y N5. |
Clientes
Invariante
| Sofía compra | en Buenos | Aires; Tomás |     |                      |               |
| ------------ | --------- | ------------ | --- | -------------------- | ------------- |
| compra en    | Madrid.   |              |     | reservas confirmadas | para VIP-01≤1 |
SistemasDistribuidos–Clase08 22/83

| Predicción | inicial |     |     |
| ---------- | ------- | --- | --- |
ACTIVIDAD
| La red se | divide: |     |     |
| --------- | ------- | --- | --- |
{N1,N2,N3}|{N4,N5}
Sofía compra desde el lado de tres nodos y Tomás desde el lado de dos.
| Elija antes  | de conocer | el algoritmo        |          |
| ------------ | ---------- | ------------------- | -------- |
| ¿Qué debería | responder  | el lado minoritario | a Tomás? |
A. Confirmada.
| B. Rechazada | temporalmente.  |               |     |
| ------------ | --------------- | ------------- | --- |
| C. Pendiente | hasta recuperar | coordinación. |     |
SistemasDistribuidos–Clase08 23/83

| Causalidad | y relojes | lógicos |
| ---------- | --------- | ------- |
Sistemas Distribuidos

Happened-before: tres reglas
NÚCLEO
1. Si dos eventos ocurren en el mismo proceso, su orden local induce causalidad.
2. El envío de un mensaje ocurre antes que la recepción de ese mensaje.
3. La relación es transitiva.
a→b∧b→c⇒a→c
significa que existe una cadena de influencia posible desde a hasta c.
SistemasDistribuidos–Clase08 25/83

| Las dos | reservas | nacen en | ramas independientes | y llegan | al  |
| ------- | -------- | -------- | -------------------- | -------- | --- |
líder
NÚCLEO
|     |     | Sofíahaceclick envíareserva |      |     |     |
| --- | --- | --------------------------- | ---- | --- | --- |
|     |     | (a1)                        | (a2) |     |     |
BA
Reserve(Sofía)
LíderN1
recibeaSofía appendlocal Reserve(Tomás)(l3) recibeaTomás
(l1) (l2)
Madrid
|     |     | Tomáshaceclick | envíareserva |     |     |
| --- | --- | -------------- | ------------ | --- | --- |
(m1) (m2)
| Relaciones | causales |     |     |     |     |
| ---------- | -------- | --- | --- | --- | --- |
Concurrencia inicial
|     |     | SistemasDistribuidos–Clase08 |     |     | 26/83 |
| --- | --- | ---------------------------- | --- | --- | ----- |
|     | →a2 | →l1                          | ∥m2 |     |       |
|     | a1  |                              | a2  |     |       |
Cuandoambasreservasseenvían,ningunacontiene
|     | →m2 | →l3 |                         |     |     |
| --- | --- | --- | ----------------------- | --- | --- |
|     | m1  |     | informaciónsobrelaotra. |     |     |
→l2 →l3,peroeseordenderecepciónnoconvierteaa2
| Ellíderordenasuspropioseventosl1 |     |     |     | encausadem2,nia |     |
| -------------------------------- | --- | --- | --- | --------------- | --- |
m2 encausadea2.

| Lamport: | convertir | relaciones | causales | en números | crecientes |
| -------- | --------- | ---------- | -------- | ---------- | ---------- |
NÚCLEO
Variables
L es el contador lógico local. t es la marca lógica incluida por el emisor en el mensaje.
m
Al recibir:
L←max(L,t
m )+1
| ▶ max conserva | la historia | local y la | comunicada; |     |     |
| -------------- | ----------- | ---------- | ----------- | --- | --- |
▶
| +1 coloca | la recepción | estrictamente | después | del envío. |     |
| --------- | ------------ | ------------- | ------- | ---------- | --- |
| Ejemplo:  | L=4, t =7 ⇒  | nuevo L=8.    |         |            |     |
m
SistemasDistribuidos–Clase08 27/83

La misma ejecución con relojes de Lamport
NÚCLEO
clicka1 senda2
L=1 L=2
BA
tm
=
2
LíderN1
receivel1 appendl2 receivel3
L=3 L=4 L=5
2
=
tm
Madrid
clickm1 sendm2
L=1 L=2
EllídercomienzaconL=0:
L(l1)=max(0,2)+1=3,
SistemasDistribuidos–Clase08 28/83
L(l2)=3+1=4,
L(l3)=max(4,2)+1=5.
Los dos mensajes salen con L = 2. La igualdad de timestamps no significa que sean el mismo evento ni que exista
causalidad entre ellos. Lamport garantiza que x → y implica L(x) < L(y), pero el recíproco no es necesariamente
cierto.

| Ejemplo | Lamport | sobre las | dos reservas |     |
| ------- | ------- | --------- | ------------ | --- |
ACTIVIDAD
| Evento         |               | Proceso |     | Lamport |
| -------------- | ------------- | ------- | --- | ------- |
| Sofía presiona | comprar       | BA      |     | 1       |
| BA envía       | reserva       | BA      |     | 2       |
| Tomás presiona | comprar       | Madrid  |     | 1       |
| Madrid         | envía reserva | Madrid  |     | 2       |
| Líder recibe   | a Sofía       | líder   |     | 3       |
| Líder hace     | append        | líder   |     | 4       |
| Líder recibe   | a Tomás       | líder   |     | 5       |
SistemasDistribuidos–Clase08 29/83

Límite de Lamport
NÚCLEO
a→b=⇒L(a)<L(b)
pero:
L(a)<L(b)̸=⇒a→b.
Por qué
Los números de Lamport permiten respetar causalidad y construir un desempate total convencional,
pero no distinguen por sí solos una dependencia real de una simple concurrencia.
SistemasDistribuidos–Clase08 30/83

| Relojes | vectoriales: | registrar | conocimiento | por actor |
| ------- | ------------ | --------- | ------------ | --------- |
NÚCLEO
| Usamos | el orden: |      |                |     |
| ------ | --------- | ---- | -------------- | --- |
|        |           | [BA, | Madrid, Líder] |     |
Cada proceso incrementa su componente y, al recibir, toma el máximo componente a componente
| antes de | incrementar el suyo. |     |     |     |
| -------- | -------------------- | --- | --- | --- |
Comparación
V <V si todos los componentes de V son menores o iguales y al menos uno es estrictamente
| a b       |                   | a                       |     |     |
| --------- | ----------------- | ----------------------- | --- | --- |
| menor. Si | ninguno domina al | otro, son concurrentes. |     |     |
SistemasDistribuidos–Clase08 31/83

| La misma | ejecución | con relojes | vectoriales |     |     |
| -------- | --------- | ----------- | ----------- | --- | --- |
NÚCLEO
|     |     | a1:click a2:send |     |     |     |
| --- | --- | ---------------- | --- | --- | --- |
|     |     | [1,0,0] [2,0,0]  |     |     |     |
BA
l3:recibeaTomás
[2,2,3]
LíderN1
|     |     | l1:recibeaSofía | l2:append |     |     |
| --- | --- | --------------- | --------- | --- | --- |
[2,0,1] [2,0,2]
Madrid
|     |     | m1:click |     | m2:send |     |
| --- | --- | -------- | --- | ------- | --- |
|     |     | [0,1,0]  |     | [0,2,0] |     |
Ordendecomponentes
[2,0,0]∥[0,2,0]
[BA,Madrid,Líder] Los vectores de las dos reservas son incomparables:
|              |                      |           | ninguna reserva | contiene conocimiento | causal de la |
| ------------ | -------------------- | --------- | --------------- | --------------------- | ------------ |
| Cada proceso | incrementa solamente | su propio |                 |                       |              |
otra.
componente.
SistemasDistribuidos–Clase08 32/83

| Cómo actualiza | su reloj | vectorial | el líder |     |     |
| -------------- | -------- | --------- | -------- | --- | --- |
NÚCLEO
Regla de recepción
Alrecibirunmensaje,elproceso:
1. calculaelmáximocomponenteacomponenteentresuvectoryelrecibido;
2. incrementasupropiocomponentepararepresentarelnuevoeventolocal.
|     |           | (               | )                 |     |     |
| --- | --------- | --------------- | ----------------- | --- | --- |
|     | V(l )=max | [0,0,0],[2,0,0] | +[0,0,1]=[2,0,1], |     |     |
1
V(l )=[2,0,1]+[0,0,1]=[2,0,2],
|     | 2         | (               | )                 |     |     |
| --- | --------- | --------------- | ----------------- | --- | --- |
|     | V(l )=max | [2,0,2],[0,2,0] | +[0,0,1]=[2,2,3]. |     |     |
3
¿Qué expresa [2,2,3]? El líder conoce: Elresultadoes[2,2,3],no[2,2,5].
| ▶   |     |     | El valor 5 pertenece | al reloj escalar | de Lamport. |
| --- | --- | --- | -------------------- | ---------------- | ----------- |
doseventosdeBuenosAires;
Enelrelojvectorial,eltercercomponentecuenta
▶
| doseventosdeMadrid; |     |     | solamenteloseventoslocalesdellíder. |     |     |
| ------------------- | --- | --- | ----------------------------------- | --- | --- |
▶ treseventoslocalespropios.
SistemasDistribuidos–Clase08 33/83

| Vector | clocks en la historia |     |
| ------ | --------------------- | --- |
NÚCLEO
| Evento       |                        | Vector  |
| ------------ | ---------------------- | ------- |
| BA envía     | reserva de Sofía       | [2,0,0] |
| Madrid       | envía reserva de Tomás | [0,2,0] |
| Líder recibe | a Sofía                | [2,0,1] |
| Líder hace   | append                 | [2,0,2] |
| Líder recibe | a Tomás                | [2,2,3] |
El reloj escalar del líder termina en L = 5, pero el tercer componente vectorial es 3 porque
cuenta sus tres eventos locales. No se mezclan ambos relojes.
SistemasDistribuidos–Clase08 34/83

| Tres relojes, tres | preguntas |     |
| ------------------ | --------- | --- |
NÚCLEO
| Reloj      | Pregunta               | Uso                         |
| ---------- | ---------------------- | --------------------------- |
| pared      | ¿quéfechayhoraciviles? | auditoría,timestampshumanos |
| monotónico | ¿cuántoduróunaespera?  | timeouts,latencias,leases   |
lógico ¿quéordencausaldeberespetarse? causalidad,versiones,logs
| Losrelojesdescribenrelaciones. | TodavíanodecidenquiénobtieneVIP-01. |     |
| ------------------------------ | ----------------------------------- | --- |
SistemasDistribuidos–Clase08 35/83

| Máquinas | de estado | y con- |
| -------- | --------- | ------ |
senso
Sistemas Distribuidos

Máquina de estados: una regla determinista
NÚCLEO
(estado,comando)−→(nuevo estado,respuesta)
RESERVED
Reserve(Sofía) BYSOFÍA
AVAILABLE
Reserve(Tomás) RESERVED
BYTOMÁS
Una reserva posterior sobre un estado ya reservado se rechaza sin cambiar el estado.
SistemasDistribuidos–Clase08 37/83

| Máquina | de estados | replicada |
| ------- | ---------- | --------- |
NÚCLEO
| Cada réplica | contiene:            |     |
| ------------ | -------------------- | --- |
| ▶ una copia  | del log de comandos; |     |
▶
| el mismo | código determinista; |     |
| -------- | -------------------- | --- |
▶
| una copia | local del estado. |     |
| --------- | ----------------- | --- |
Si todas aplican los mismos comandos comprometidos en el mismo orden, convergen al mismo
estado.
SistemasDistribuidos–Clase08 38/83

| Safety | y liveness |     |     |     |     |
| ------ | ---------- | --- | --- | --- | --- |
NÚCLEO
|              |            |        | Liveness:        | el sistema vuelve | a progresar    |
| ------------ | ---------- | ------ | ---------------- | ----------------- | -------------- |
| Safety: nada | incorrecto | ocurre |                  |                   |                |
|              |            |        | Requiere mayoría | comunicada,       | líder estable, |
▶
| no hay        | dos comandos | distintos        |                      |           |                 |
| ------------- | ------------ | ---------------- | -------------------- | --------- | --------------- |
|               |              |                  | procesos activos     | y entrega | eventual de los |
| comprometidos | en           | el mismo índice; |                      |           |                 |
|               |              |                  | mensajes necesarios. |           |                 |
▶
| una entrada | committed      | no se reemplaza;   |     |     |     |
| ----------- | -------------- | ------------------ | --- | --- | --- |
| ▶ VIP-01    | no se confirma | para dos personas. |     |     |     |
SistemasDistribuidos–Clase08 39/83

| Raft: término, |     | índice |     | y entrada |     | de log |
| -------------- | --- | ------ | --- | --------- | --- | ------ |
NÚCLEO
▶
Término: número creciente que identifica una ronda de elección y liderazgo.
| ▶ Índice:  | posición | estable        | de una | entrada | dentro    | del log. |
| ---------- | -------- | -------------- | ------ | ------- | --------- | -------- |
| ▶ Entrada: | par      | (term,command) |        | en una  | posición. |          |
Ejemplo
| Los índices | 1–86 ya | existen.  | El líder | del término |                               | 12 agrega: |
| ----------- | ------- | --------- | -------- | ----------- | ----------------------------- | ---------- |
|             |         | index=87, | term=12, |             | command=Reserve(VIP-01,Sofía) |            |
SistemasDistribuidos–Clase08 40/83

Mensaje o líder obsoleto
NÚCLEO
Un mensaje es obsoleto cuando contiene un término menor que el término ya conocido por el
receptor.
N1despiertaAppendEntriesterm12 N2líder
term12 term13
rechazado
SistemasDistribuidos–Clase08 41/83

| Raft paso | 1: la propuesta | existe solo | en N1 |     |
| --------- | --------------- | ----------- | ----- | --- |
NÚCLEO
| Nodo | Última entrada |                     |     | Estado |
| ---- | -------------- | ------------------- | --- | ------ |
| N1   | 87 / term      | 12 / Reserve(Sofía) |     | local  |
| N2   | hasta 86       |                     |     | —      |
| N3   | hasta 86       |                     |     | —      |
| N4   | hasta 86       |                     |     | —      |
| N5   | hasta 86       |                     |     | —      |
Todavía no
Una propuesta local no autoriza responder «compra confirmada».
SistemasDistribuidos–Clase08 42/83

| Raft paso | 2: dos | copias todavía | no son mayoría |     |
| --------- | ------ | -------------- | -------------- | --- |
NÚCLEO
| Nodo        | Última entrada        |                     |     | Estado |
| ----------- | --------------------- | ------------------- | --- | ------ |
| N1          | 87 / term             | 12 / Reserve(Sofía) |     | local  |
| N2          | 87 / term             | 12 / Reserve(Sofía) |     | local  |
| N3          | hasta 86              |                     |     | —      |
| N4          | hasta 86              |                     |     | —      |
| N5          | hasta 86              |                     |     | —      |
| N2 persiste | la entrada y responde | al líder.           |     |        |
2/5<3/5
La seguridad no proviene solo del número de copias, sino de una mayoría intersectante y de las
| reglas del | protocolo. |     |     |     |
| ---------- | ---------- | --- | --- | --- |
SistemasDistribuidos–Clase08 43/83

| Raft paso | 3: la tercera | réplica cambia | la situación |     |
| --------- | ------------- | -------------- | ------------ | --- |
NÚCLEO
| Nodo           | Última entrada |                     |     | Estado |
| -------------- | -------------- | ------------------- | --- | ------ |
| N1             | 87 / term      | 12 / Reserve(Sofía) |     | local  |
| N2             | 87 / term      | 12 / Reserve(Sofía) |     | local  |
| N3             | 87 / term      | 12 / Reserve(Sofía) |     | local  |
| N4             | hasta 86       |                     |     | —      |
| N5             | hasta 86       |                     |     | —      |
| N3 persiste la | misma entrada. |                     |     |        |
3/5
Toda mayoría futura de tres nodos debe compartir al menos un nodo con {N1,N2,N3}.
La entrada puede avanzar a committed porque alcanzó la mayoría exigida por el protocolo.
SistemasDistribuidos–Clase08 44/83

| Raft paso | 4: quién | compromete, |     | aplica | y responde |
| --------- | -------- | ----------- | --- | ------ | ---------- |
NÚCLEO
| 1. N2 y        | N3 confirman | al líder    | que persistieron | la entrada. |     |
| -------------- | ------------ | ----------- | ---------------- | ----------- | --- |
| 2. N1 detecta  | mayoría      | y actualiza | commitIndex=87.  |             |     |
| 3. N1 aplica   | la entrada   | en su       | propia máquina   | de estados. |     |
| 4. N1 responde | «compra      | confirmada» | a Sofía.         |             |     |
5. N2 y N3 reciben leaderCommit=87, avanzan y aplican localmente.
| Replicada | ̸= committed | ̸= aplicada | ̸= confirmada | al cliente. |     |
| --------- | ------------ | ----------- | ------------- | ----------- | --- |
SistemasDistribuidos–Clase08 45/83

| Crash antes | y después | de la mayoría |     |     |
| ----------- | --------- | ------------- | --- | --- |
ACTIVIDAD
| Antes de mayoría |     |     | Después | de mayoría |
| ---------------- | --- | --- | ------- | ---------- |
N1 contiene la entrada y cae. No estaba N1, N2 y N3 persistieron; N1 confirmó y cae.
committed y puede desaparecer. Sofía no debió N2 o N3 conserva la decisión comprometida.
recibir éxito.
SistemasDistribuidos–Clase08 46/83

| Total-order | broadcast: | por qué | aparece | aquí |
| ----------- | ---------- | ------- | ------- | ---- |
NÚCLEO
Necesidad
Máquinas deterministas solo convergen si reciben los mismos comandos comprometidos y los
| aplican en | el mismo orden. |     |     |     |
| ---------- | --------------- | --- | --- | --- |
| Relación   | con consenso    |     |     |     |
Consenso elige un valor para una posición. Repetir consenso para 87, 88, 89, ... construye un log
| ordenado; | ese servicio implementa | total-order | broadcast. |     |
| --------- | ----------------------- | ----------- | ---------- | --- |
Un timestamp Lamport etiqueta eventos, pero no garantiza que todas las réplicas reciban el
| mismo | conjunto de comandos | ni que no existan | huecos. |     |
| ----- | -------------------- | ----------------- | ------- | --- |
SistemasDistribuidos–Clase08 47/83

Pausa de proceso, GC y fencing
NÚCLEO
▶ GC: pausa del runtime para recuperar memoria; el proceso puede dejar de ejecutar sin estar
muerto.
▶ Fencing token: número creciente que un recurso externo usa para rechazar operaciones de
una autoridad antigua.
N1 líder term 12 → pausa → N2 líder term 13 → N1 despierta obsoleto
El término protege al cluster; un recurso externo puede requerir además un token de fencing.
SistemasDistribuidos–Clase08 48/83

| Paxos: misma | meta, organización | diferente |
| ------------ | ------------------ | --------- |
EXTENSIÓN
| Paso       | Raft                  | Multi-Paxos                |
| ---------- | --------------------- | -------------------------- |
| liderazgo  | líder,term12          | proposerestable,ballot12   |
| posición   | index87               | slot87                     |
| propuesta  | AppendEntries         | Accept                     |
| réplicas   | followers             | acceptors                  |
| decisión   | mayoríareplicaycommit | mayoríaacepta,valorelegido |
| aplicación | máquinadeestados      | learnerymáquinadeestados   |
Ambosimpidendecisionesincompatiblesparalamismaposición,aunqueorganizanrolesymensajesdeformadiferente.
SistemasDistribuidos–Clase08 49/83

| Caso | real: dónde aparecen |     | estas ideas | en Kubernetes |
| ---- | -------------------- | --- | ----------- | ------------- |
CASOREAL
|          | kubectl | → API server | → líder etcd | → seguidores etcd |
| -------- | ------- | ------------ | ------------ | ----------------- |
| Concepto |         | Aplicación   | en etcd      |                   |
líder y término un líder Raft atiende escrituras y rechaza términos viejos
log e índice cada cambio ocupa una posición del log replicado
| mayoría      | y commit   | se confirma    | tras alcanzar | quorum                    |
| ------------ | ---------- | -------------- | ------------- | ------------------------- |
| máquina      | de estados | todos obtienen | el mismo      | key–value store           |
| WAL/snapshot |            | cada miembro   | recuerda      | decisiones tras reiniciar |
SistemasDistribuidos–Clase08 50/83

| Tres logs | que no | debemos | confundir |     |
| --------- | ------ | ------- | --------- | --- |
NÚCLEO
| Log      |     | Pregunta     |             | Ejemplo        |
| -------- | --- | ------------ | ----------- | -------------- |
| consenso |     | ¿qué comando | se acordó y | en qué logRaft |
orden?
| WALlocal |     | ¿quérecuerdafísicamenteelnodotras |     | WALdelaDB/etcd |
| -------- | --- | --------------------------------- | --- | -------------- |
uncrash?
| eventos |     | ¿quéhechospuedenleeryreprocesar |     | Kafka |
| ------- | --- | ------------------------------- | --- | ----- |
otroscomponentes?
SistemasDistribuidos–Clase08 51/83

| CAP,                  | PACELC | y garantías |
| --------------------- | ------ | ----------- |
| Sistemas Distribuidos |        |             |

| La partición | ocurre | después del commit | de Sofía |
| ------------ | ------ | ------------------ | -------- |
NÚCLEO
{N1,N2,N3}|{N4,N5}
La mayoría conoce la reserva committed de Sofía. La minoría está sana localmente, pero no recibe
| ese hecho. Tomás | intenta comprar | desde Madrid. |     |
| ---------------- | --------------- | ------------- | --- |
SistemasDistribuidos–Clase08 53/83

| Dos historias | indistinguibles | para la minoría |
| ------------- | --------------- | --------------- |
NÚCLEO
| H1  |     | H2  |
| --- | --- | --- |
Sofía ya fue confirmada, pero el mensaje no Sofía nunca fue confirmada.
cruza la partición.
N4 y N5 observan lo mismo en ambas historias. Una respuesta local idéntica puede ser correcta en
| una e incorrecta en | la otra. |     |
| ------------------- | -------- | --- |
SistemasDistribuidos–Clase08 54/83

CAP: la decisión durante una partición
NÚCLEO
▶ Consistencia fuerte: una sola historia compatible con una copia autoritativa.
▶ Disponibilidad CAP: cada solicitud a un nodo no fallado termina con una respuesta válida de
la operación.
▶ Partición: grupos de nodos vivos no pueden intercambiar los mensajes necesarios a tiempo.
Durante la partición, confirmar localmente a Tomás arriesga doble venta; esperar o rechazar
preserva el invariante, pero sacrifica disponibilidad CAP para esa operación.
SistemasDistribuidos–Clase08 55/83

| Uptime, | disponibilidad | CAP y disponibilidad | semántica |
| ------- | -------------- | -------------------- | --------- |
NÚCLEO
| Concepto | Quémide |     |     |
| -------- | ------- | --- | --- |
uptime fraccióndeunaventanamensual,trimestraloanualenqueelprocesoestuvo
alcanzable
disponibilidadCAP silaoperaciónterminaconrespuestaválidaduranteunapartición
disponibilidadsemántica fraccióndeintentosválidosquecumplenlapromesadenegocio
Un503rápidodemuestraqueelprocesoestávivo,peronosatisfacedisponibilidadCAPparalacompra.
SistemasDistribuidos–Clase08 56/83

| PACELC: | el costo | continúa | cuando | la red funciona |
| ------- | -------- | -------- | ------ | --------------- |
NÚCLEO
| Lectura local |     |     | Lectura | coordinada |
| ------------- | --- | --- | ------- | ---------- |
15 ms, pero puede observar una réplica 180 ms, con garantía más fuerte.
atrasada.
PACELC organiza dos preguntas: durante una partición, consistencia o disponibilidad; en
| régimen normal, | latencia | o consistencia. |     |     |
| --------------- | -------- | --------------- | --- | --- |
SistemasDistribuidos–Clase08 57/83

Recencia, linearizabilidad y serializabilidad
NÚCLEO
Concepto Definiciónbreve
recencia quétannuevoeselestadoobservado
linearizabilidad cadaoperaciónpareceatómicaentreinvocaciónyrespuestayrespetaelordenrealde
operacionesnosuperpuestas
serializabilidad elresultadodetransaccionesequivaleaalgúnordenserial,sinexigirordenrealexterno
consistenciaexterna serializabilidadmásrespetodelordenobservadoporlosclientes
SistemasDistribuidos–Clase08 58/83

| Quórums, | versiones | y Dy- |
| -------- | --------- | ----- |
namo
Sistemas Distribuidos

| Mayoría | en consenso | y quorum | Dynamo-like | no son sinónimos |
| ------- | ----------- | -------- | ----------- | ---------------- |
NÚCLEO
|     |     | Consenso |     | Dynamo-like |
| --- | --- | -------- | --- | ----------- |
pregunta quéentradaautoritativaseelige cuántasréplicasparticipan
| unidad |     | posición/slotdellog |     | lecturaoescrituradeunakey |
| ------ | --- | ------------------- | --- | ------------------------- |
concurrencia protocoloimpidecommitsincompatibles puedeconservarversionesconcurrentes
resultado ordenautoritativo conjuntodeversionesypolíticademerge
intersección partedereglasdeseguridad propiedadcombinatoriadeR,W,N
SistemasDistribuidos–Clase08 60/83

| Por qué | necesitamos | intersección |
| ------- | ----------- | ------------ |
NÚCLEO
Con N=5, una escritura llega a {N1,N2} y una lectura posterior consulta {N4,N5}. Los
conjuntos son disjuntos: la lectura puede no encontrar ninguna copia actualizada.
Queremos que todo conjunto ideal de lectura comparta al menos un nodo con todo conjunto
| ideal de | escritura. |     |
| -------- | ---------- | --- |
SistemasDistribuidos–Clase08 61/83

| N, R y W: | construir | la intersección |     |
| --------- | --------- | --------------- | --- |
ACTIVIDAD
▶
| N: réplicas     | preferidas para | la key.      |               |
| --------------- | --------------- | ------------ | ------------- |
| ▶ W: respuestas | requeridas      | para aceptar | la escritura. |
▶
| R: respuestas | requeridas | para completar | la lectura. |
| ------------- | ---------- | -------------- | ----------- |
R+W>N
Con N=5, W=3, R=3, los conjuntos ideales deben intersectarse.
La intersección no crea automáticamente linearizabilidad, líder, orden total ni resolución
semántica.
SistemasDistribuidos–Clase08 62/83

| Contraejemplo | 1: escrituras | concurrentes |
| ------------- | ------------- | ------------ |
NÚCLEO
Dos clientes escriben versiones distintas sin conocer la otra. El nodo de intersección puede
| conservar ambas ramas. |               |         |
| ---------------------- | ------------- | ------- |
| Por qué contradice     | la conclusión | ingenua |
R+W>N permite descubrir que existen versiones distintas, pero no decide cuál es posterior si
fueron concurrentes.
SistemasDistribuidos–Clase08 63/83

| Contraejemplo | 2: sloppy | quorum |
| ------------- | --------- | ------ |
NÚCLEO
Durante una falla, la escritura se almacena temporalmente en nodos fuera del conjunto preferido.
Antes del hinted handoff, una lectura sobre las réplicas preferidas puede no cruzarse con el conjunto
efectivo de escritura.
Lección
La fórmula usa conjuntos ideales; la implementación real debe considerar dónde quedaron
| efectivamente las copias. |     |     |
| ------------------------- | --- | --- |
SistemasDistribuidos–Clase08 64/83

| Contraejemplo |     | 3: last-write-wins |     | y relojes | desalineados |
| ------------- | --- | ------------------ | --- | --------- | ------------ |
NÚCLEO
Una actualización causalmente posterior recibe un timestamp físico menor por desalineación del
| reloj y puede | ser descartada | como «más | vieja». |     |     |
| ------------- | -------------- | --------- | ------- | --- | --- |
Lección
Intersección y timestamps físicos no sustituyen un modelo correcto de causalidad.
SistemasDistribuidos–Clase08 65/83

| Vector | clocks reaparecen |     | como genealogía | del carrito |
| ------ | ----------------- | --- | --------------- | ----------- |
NÚCLEO
| Partimos de: |     |     |     |     |
| ------------ | --- | --- | --- | --- |
v =[0,0]
0
| Durante la partición: |        |                |                          |     |
| --------------------- | ------ | -------------- | ------------------------ | --- |
|                       |        |                | BA agrega libro→[1,0]    |     |
|                       |        | Madrid         | agrega auriculares→[0,1] |     |
| Ninguna versión       | domina | a la otra: son | ramas concurrentes.      |     |
SistemasDistribuidos–Clase08 66/83

| Carrito | fusionable; |     | entrada | VIP no | fusionable |     |     |     |
| ------- | ----------- | --- | ------- | ------ | ---------- | --- | --- | --- |
NÚCLEO
VIP-01
Carrito
|                 |              |               |            |     | No existe un     | merge válido | «Sofía + Tomás».       | El  |
| --------------- | ------------ | ------------- | ---------- | --- | ---------------- | ------------ | ---------------------- | --- |
| Puede conservar | libro        | y auriculares | y producir |     |                  |              |                        |     |
|                 |              |               |            |     | invariante exige | una única    | decisión autoritativa. |     |
| una versión     | descendiente | que incluya   | ambas      |     |                  |              |                        |     |
intenciones.
|     |     |     |     | SistemasDistribuidos–Clase08 |     |     |     | 67/83 |
| --- | --- | --- | --- | ---------------------------- | --- | --- | --- | ----- |

| Dynamo: | localizar, | escribir, | conservar | y reparar |
| ------- | ---------- | --------- | --------- | --------- |
NÚCLEO
| 1. aplicar   | hash a la key          | y construir una      | preference list;  |     |
| ------------ | ---------------------- | -------------------- | ----------------- | --- |
| 2. escribir  | en réplicas hasta      | reunir W respuestas; |                   |     |
| 3. conservar | versiones concurrentes | en                   | vez de perderlas; |     |
| 4. responder | según la política      | de disponibilidad;   |                   |     |
5. reparar la divergencia en segundo plano o durante lecturas.
SistemasDistribuidos–Clase08 68/83

Sloppy quorum, hint y hinted handoff
NÚCLEO
▶ Sloppy quorum: aceptar temporalmente la copia en un nodo alternativo.
▶ Hint: metadato que recuerda cuál era el propietario preferido.
▶ Hinted handoff: devolver la copia al propietario cuando se recupera.
La disponibilidad desplaza costo hacia almacenamiento temporal, red, I/O y reparación
posterior.
SistemasDistribuidos–Clase08 69/83

Read repair, anti-entropy y tasa de divergencia
NÚCLEO
▶ Read repair: corrige las réplicas consultadas al detectar versiones atrasadas.
▶ Anti-entropy: compara rangos en background para reparar datos fríos.
▶ Tasa de divergencia: diferencias nuevas creadas por unidad de tiempo.
▶ Capacidad de reparación: diferencias corregidas por unidad de tiempo.
D >R ⇒la deuda crece
nuevo resuelto
SistemasDistribuidos–Clase08 70/83

| Por qué | Dynamo | aceptó esa | complejidad |
| ------- | ------ | ---------- | ----------- |
CASOREAL
El Dynamo original incorporó consistent hashing, preference lists, quórums, versionado, sloppy
quorum, hinted handoff, read repair y anti-entropy para sostener la prioridad always-on de algunos
| servicios de | Amazon. |     |     |
| ------------ | ------- | --- | --- |
No son sinónimos de «complejidad»; cada mecanismo paga una parte concreta de la
| disponibilidad | durante | fallas. |     |
| -------------- | ------- | ------- | --- |
SistemasDistribuidos–Clase08 71/83

| Spanner, | TrueTime | y orden |
| -------- | -------- | ------- |
externo
Sistemas Distribuidos

| Por qué | aparece | Spanner | en la misma | historia |
| ------- | ------- | ------- | ----------- | -------- |
NÚCLEO
| La compra | de VIP-01    | puede involucrar: |     |     |
| --------- | ------------ | ----------------- | --- | --- |
| ▶ grupo   | de entradas; |                   |     |     |
▶
| grupo   | de pagos;   |              |     |     |
| ------- | ----------- | ------------ | --- | --- |
| ▶ grupo | de derechos | del usuario. |     |     |
La operación debe ser atómica entre grupos y, si Madrid inicia una acción después de observar la
confirmación de Buenos Aires, el orden externo debe respetarse.
SistemasDistribuidos–Clase08 73/83

| Spanner: | consenso | localizado | y transacción | multi-grupo |
| -------- | -------- | ---------- | ------------- | ----------- |
NÚCLEO
Cada grupo de datos mantiene replicación y consenso propios. Una transacción que toca varios
grupos coordina solo los grupos necesarios y asigna un timestamp de commit común.
El objetivo no es un único líder global para toda la base, sino coordinación localizada con una
| garantía | transaccional global. |     |     |     |
| -------- | --------------------- | --- | --- | --- |
SistemasDistribuidos–Clase08 74/83

| TrueTime: | intervalo | de tiempo | con incertidumbre | explícita |
| --------- | --------- | --------- | ----------------- | --------- |
NÚCLEO
Spanner sincroniza servidores con referencias como GPS y relojes atómicos mediante time masters.
| Entre sincronizaciones | existe | deriva. |     |     |
| ---------------------- | ------ | ------- | --- | --- |
TT.now()=[earliest,latest]
Conceptualmente:
|     |     | earliest= b t−ϵ, | latest= b t+ϵ |     |
| --- | --- | ---------------- | ------------- | --- |
TrueTime no afirma conocer un instante exacto: expone una cota que incluye error de
| sincronización | y deriva. |     |     |     |
| -------------- | --------- | --- | --- | --- |
SistemasDistribuidos–Clase08 75/83

| Quién | asigna el | timestamp | de commit |     |
| ----- | --------- | --------- | --------- | --- |
NÚCLEO
| El coordinador | de la transacción | elige s | respetando: |     |
| -------------- | ----------------- | ------- | ----------- | --- |
▶
| timestamps | de preparación        | de los grupos; |     |     |
| ---------- | --------------------- | -------------- | --- | --- |
| ▶ la cota  | superior de TrueTime; |                |     |     |
▶
| dependencias | y restricciones | de la | transacción. |     |
| ------------ | --------------- | ----- | ------------ | --- |
Ejemplo:
|     |     | TT.now()=[4,16] | ms, s=16 | ms  |
| --- | --- | --------------- | -------- | --- |
SistemasDistribuidos–Clase08 76/83

| Commit | wait: esperar | hasta demostrar | que s quedó | en el |
| ------ | ------------- | --------------- | ----------- | ----- |
pasado
NÚCLEO
| El sistema | espera hasta observar: |     |     |     |
| ---------- | ---------------------- | --- | --- | --- |
TT.now().earliest>s
Por ejemplo:
TT.now()=[17,29] ms⇒17>16
La espera convierte incertidumbre temporal acotada en una garantía de orden externo.
|     |     | SistemasDistribuidos–Clase08 |     | 77/83 |
| --- | --- | ---------------------------- | --- | ----- |

| Por | qué se llama | consistencia | externa |
| --- | ------------ | ------------ | ------- |
NÚCLEO
Si un cliente recibe el commit de T y después inicia T , toda serialización válida debe respetar:
1 2
T <T
1 2
Se llama externa porque respeta el orden observado por clientes fuera del motor, no solo algún
| orden serial | internamente | válido. |     |
| ------------ | ------------ | ------- | --- |
SistemasDistribuidos–Clase08 78/83

Integración y cierre
Sistemas Distribuidos

| Cada mecanismo | responde | una pregunta | distinta |
| -------------- | -------- | ------------ | -------- |
NÚCLEO
| Mecanismo      | Pregunta                                         |     |     |
| -------------- | ------------------------------------------------ | --- | --- |
| Lamport/vector | ¿quéeventosestánrelacionadososonconcurrentes?    |     |     |
| Raft/Paxos     | ¿quécomandoseconvierteendecisiónautoritativa?    |     |     |
| WALlocal       | ¿quérecuerdacadanododespuésdeuncrash?            |     |     |
| Kafka          | ¿quéhechospuedenleeryreprocesarotroscomponentes? |     |     |
| CAP/PACELC     | ¿quéerroryquécostoaceptamosbajoredimperfecta?    |     |     |
| Dynamo         | ¿cómoseguirrespondiendoyreparardivergencia?      |     |     |
| Spanner        | ¿cómosostenertransaccionesglobalesyordenexterno? |     |     |
SistemasDistribuidos–Clase08 80/83

| Actividad | integradora: | cuatro operaciones, | cuatro decisiones |
| --------- | ------------ | ------------------- | ----------------- |
ACTIVIDAD
Para cada operación declare invariante, mecanismo, comportamiento bajo partición, costo normal y
prueba de falla:
| 1. última    | entrada VIP;       |          |     |
| ------------ | ------------------ | -------- | --- |
| 2. carrito   | compartido;        |          |     |
| 3. líder del | control plane;     |          |     |
| 4. ranking   | global que entrega | premios. |     |
SistemasDistribuidos–Clase08 81/83

Exit ticket
ACTIVIDAD
| Responda   | en una        | frase     | cada punto: |                           |                   |
| ---------- | ------------- | --------- | ----------- | ------------------------- | ----------------- |
| 1. ¿Por    | qué L(a)<L(b) |           | no          | prueba causalidad?        |                   |
| 2. ¿Cómo   | detecta       | un        | vector      | que dos eventos           | son concurrentes? |
| 3. ¿Cuándo | queda         | committed |             | la entrada                | 87 en Raft?       |
| 4. ¿Qué    | diferencia    | hay       | entre       | log de consenso,          | WAL y Kafka?      |
| 5. ¿Por    | qué R+W>N     |           | no          | implica linearizabilidad? |                   |
| 6. ¿Qué    | costo         | paga      | Spanner     | para obtener              | orden externo?    |
SistemasDistribuidos–Clase08 82/83

Bibliografía guiada
RESPALDO
▶
Kleppmann,DesigningData-IntensiveApplications,caps. 5,8y9.
▶
Lamport,Time,Clocks,andtheOrderingofEvents.
▶
OngaroyOusterhout,InSearchofanUnderstandableConsensusAlgorithm.
▶
Lamport,PaxosMadeSimple.
▶
DeCandiaetal.,Dynamo.
▶
Corbettetal.,Spanner.
SistemasDistribuidos–Clase08 83/83