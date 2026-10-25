| Sistemas Distribuidos |     |     |
| --------------------- | --- | --- |
Clase 07 — Festival Austral: del checkout al log distribuido y
recuperable
| Departamento de        | Computación |           |
| ---------------------- | ----------- | --------- |
| Ingeniería Informática | – Comisión  | A – Pilar |
| Equipo docente –       | 2026        |           |

Desafío de apertura: ¿qué es Kafka y por qué aparece tanto?
ACTIVIDAD
Tienentresminutos: busquenenGoogleypreparenunarespuestabreveparaleerenvozalta.
1. ¿QuéesApacheKafka?
2. ¿Paraqué,cuántoydóndeenlarealidadseusakafka?
SistemasDistribuidos–Clase07 2/82

| Caso conductor: | CampusShop |     | - la historia | completa |
| --------------- | ---------- | --- | ------------- | -------- |
NÚCLEO
| Sofía — pedido | 9F2A |     | Tomás | — pedido 1C7B |
| -------------- | ---- | --- | ----- | ------------- |
OrderCreated→StockReserved→PaymentAuthorized OrderCreated→StockReserved→PaymentRejected
| →OrderConfirmed. |     |     | →StockReleased→OrderCancelled. |     |
| ---------------- | --- | --- | ------------------------------ | --- |
Después,elserviciodeentradas Tomásnodebe
| generasucódigoQR. |     |     | recibirQR. |     |
| ----------------- | --- | --- | ---------- | --- |
El accidente
| QR — Quick | Response. Código | bidimensional |     |     |
| ---------- | ---------------- | ------------- | --- | --- |
ElservicioqueemiteQRcaedurante40segundos;Analítica
queCampusShopusacomoentradadigitalalFestival
sigueatrasándosemientrasnuevascomprasllegan.
Austral.
Seguiremosestasdoshistoriasdeprincipioafin. Cadaconceptoaparecerámientrasnosenfocamosendistintasfallas
vinculadasa9F2Ao1C7B.
SistemasDistribuidos–Clase07 3/82

| Por qué | dividimos | la historia | en  | cinco capítulos |
| ------- | --------- | ----------- | --- | --------------- |
NÚCLEO
| Capítulo | Problemadelahistoria |     |     | Conceptosquenecesitaremos |
| -------- | -------------------- | --- | --- | ------------------------- |
1. Publicar El checkout no debe esperar a QR y productor,broker,log,Kafka,ACK
Analítica,peroelhechodebesobrevivir.
2. Ordenar Los estados de cada pedido deben topic,partición,key,offset,hotpartition
|     | conservar | una secuencia | sin serializar |     |
| --- | --------- | ------------- | -------------- | --- |
todaslascompras.
3. Consumir QR cae después de actuar y otra poll, grupo, committed offset, replay, inbox,
|     | instancia | debe continuar | sin perder | ni rebalanceo,lag |
| --- | --------- | -------------- | ---------- | ----------------- |
duplicar.
4. Publicarseguro Labasepuedeconfirmarelpedidoyel WAL local, outbox, relay, CDC, Debezium,
|     | procesocaerantesdepublicar. |     |     | fronteraexactly-once |
| --- | --------------------------- | --- | --- | -------------------- |
5. Operar el Analítica recibe una historia ilimitada, stream, pipeline, estado, event time, ventana,
| stream | conatrasoyeventosfueradeorden. |     |     | watermark |
| ------ | ------------------------------ | --- | --- | --------- |
Ladivisiónnoesuníndicedeproductos: eslacronologíaquepuedenllevarafallasenunsistemadistribuido.
SistemasDistribuidos–Clase07 4/82

| Qué significa | “compra | confirmada” | en  | esta clase |
| ------------- | ------- | ----------- | --- | ---------- |
NÚCLEO
| Antes de mostrar | el OK |     | Después | del OK |
| ---------------- | ----- | --- | ------- | ------ |
| ▶ pedidodurable; |       |     | ▶       |        |
emitirelcódigoQR;
▶
| stockreservado;   |     |     | ▶ enviarunanotificación; |     |
| ----------------- | --- | --- | ------------------------ | --- |
| ▶ pagoautorizado; |     |     | ▶                        |     |
actualizartablero,rankingehistorial.
▶
intencióndurabledepublicarOrderConfirmed.
Esaslecturasposterioressonvistas derivadasoproyecciones: representanunhechoyaconfirmado; nodecidensila
comprafueválida.
| Semántica elegida |     |     |     |     |
| ----------------- | --- | --- | --- | --- |
SielpagodeTomásfalla,elpedidosecancela,lareservaseliberaynoexisteOrderConfirmed(1C7B).
SistemasDistribuidos–Clase07 5/82

| Checkout: |     | qué es | y qué | no es |     |     |     |
| --------- | --- | ------ | ----- | ----- | --- | --- | --- |
NÚCLEO
|            |     |             |              |     | No es lo mismo | que... |     |
| ---------- | --- | ----------- | ------------ | --- | -------------- | ------ | --- |
| Checkout   | es  | el servicio | o flujo      | de  |                |        |     |
|            |     |             |              |     | ▶ la base de   | datos; |     |
| aplicación | que | recibe      | la intención | de  |                |        |     |
▶
| compra,  | coordina     | los pasos    | críticos | y   | el WAL del    | motor;      |     |
| -------- | ------------ | ------------ | -------- | --- | ------------- | ----------- | --- |
| registra | el resultado | del pedido.  |          |     |               |             |     |
|          |              |              |          |     | ▶ el broker   | de eventos; |     |
|          |              |              |          |     | ▶ el servicio | de QR.      |     |
| ▶ Recibe | la petición  | del usuario. |          |     |               |             |     |
▶
| Usa servicios |     | de Pago e | Inventario. |     |     |     |     |
| ------------- | --- | --------- | ----------- | --- | --- | --- | --- |
Checkoutsolicitaunatransacción;elmotor
| ▶ Persiste | el estado | del pedido. |            |     |               |                 |              |
| ---------- | --------- | ----------- | ---------- | --- | ------------- | --------------- | ------------ |
|            |           |             |            |     | de la base    | usa su WAL para | recuperar el |
| ▶          |           |             |            |     | commit local. |                 |              |
| Responde   | cuando    | se cumple   | la promesa |     |               |                 |              |
elegida.
SistemasDistribuidos–Clase07 6/82

De llamadas directas a una historia compartida
NÚCLEO
Fan-out sincrónico Historia compartida y preservada
UnapeticiónseramificaenvariasllamadasyCheckout CheckoutpublicaunavezOrderConfirmed(9F2A).Cada
esperalasrespuestasantesdeterminar. lectoravanzayserecuperadeformaindependiente.
QR QR
Logpersistido
Checkout Notificación Checkout Notificación
yreleíble
Analítica Analítica
Problema
Eltrabajosaledelcaminocríticoypuederecuperarse
Elcomponentemáslentofijalalatencia;unadependencia sinqueCheckoutsigavivo.
secundariapuedehacerfallarunacompraválida.
SistemasDistribuidos–Clase07 7/82

| Base mínima | I: quiénes | participan |
| ----------- | ---------- | ---------- |
NÚCLEO
| Actor           | Definicióndetrabajo                     |     |
| --------------- | --------------------------------------- | --- |
| Usuario/cliente | Personaoaplicaciónqueiniciaunapetición. |     |
Balanceadordecarga Distribuyepeticionesentrevariasinstanciasdelservicio.
InstanciadeAPI Procesoquerecibeunapeticiónyejecutalógicadeaplicación.
Productor Códigooprocesoquepublicaunregistroenellogdistribuido.
Broker Procesoservidorque aloja particiones, conserva registros y atiendeproductores
yconsumidores.
Consumidor Procesoqueleeregistrosyproduceunefectooestadoderivado.
ServiciodeQR Consumidorquegenerayconservaunaentradadigitalidentificable.
SistemasDistribuidos–Clase07 8/82

| Base mínima | II: cómo | un proceso | usa los recursos | de un nodo |
| ----------- | -------- | ---------- | ---------------- | ---------- |
NÚCLEO
CPU:unoomásnúcleosejecutanelcódigodelproceso:
Nodofísico,VMohost
validación,serialización,compresiónológica.
Proceso
RAM:conservabuffers,lotes,cachésyestadotemporal.
software
NIC:interfazqueenvíayrecibetráficodered.
Mediopersistente: SSD,NVMeodiscoqueconserva
|         | SSD/ |     | segmentosyestadodurable. |     |
| ------- | ---- | --- | ------------------------ | --- |
| CPU RAM | NIC  |     |                          |     |
disco
SistemasDistribuidos–Clase07 9/82

| Base mínima | III: comando, | evento, | registro | y log |
| ----------- | ------------- | ------- | -------- | ----- |
NÚCLEO
| Término | Definiciónyejemplo |     |     |     |
| ------- | ------------------ | --- | --- | --- |
Comando Intenciónquepuedeaceptarseorechazarse: ConfirmOrder(9F2A).
Evento Hechoqueeldominioconsideraocurrido: OrderConfirmed(9F2A).
Registro Representaciónalmacenadadelhecho: identidad,key,tipo,datosymetadatos.
| Append | Agregarunregistroalfinaldeunapartición. |     |     |     |
| ------ | --------------------------------------- | --- | --- | --- |
Log Secuenciaordenadaypreservadaderegistrosquepuederecorrerseyreleerse.
Replay Volveraleerregistrosparareconstruirprocesamientooestado.
Eleventoeselsignificado;elregistroessurepresentaciónalmacenada;ellogeslasecuenciadeesosregistros.
SistemasDistribuidos–Clase07 10/82

El accidente de las 12:00:02
PREDICCIÓN
QR-3
CRASH
emitecódigo
OrderConfirmed
9F2A
Analítica
seatrasa
Predicción
¿Cómo recupera QR el trabajo? ¿Qué historia, marca de progreso e identidad deben conservarse
para no depender de que Checkout siga vivo?
Guardesurespuesta: lacerraremosenelCapítulo3conpoll,committedoffset,replayeinbox.
SistemasDistribuidos–Clase07 11/82

| Pregunta | central: | del hecho | al efecto | recuperable |     |
| -------- | -------- | --------- | --------- | ----------- | --- |
NÚCLEO
¿Cómoconvertirdoscomprasconcurrentesenhistoriascompartidasquedistintoscomponentes
puedan procesar y reprocesar sin perder progreso ni repetir el efecto lógico?
|     | Hecho          | Registro | LectorQR       |     | UnQR   |
| --- | -------------- | -------- | -------------- | --- | ------ |
|     | OrderConfirmed | P2:420   | progresopropio |     | lógico |
SistemasDistribuidos–Clase07 12/82

| Evidencias | de aprendizaje |     | esperadas |
| ---------- | -------------- | --- | --------- |
NÚCLEO
| Al finalizar, | debemos poder | entender: |     |
| ------------- | ------------- | --------- | --- |
1. elegir una key y explicar qué orden conserva y qué carga concentra;
2. mover un crash en el timeline y distinguir pérdida, repetición y replay seguro;
3. explicar cómo un consumidor reinicia desde progreso durable;
4. diseñar outbox e inbox y declarar la frontera de una garantía exactly-once.
La evidencia será un diagrama, una decisión y una explicación breve; no una lista memorizada
| de componentes | de Kafka. |     |     |
| -------------- | --------- | --- | --- |
SistemasDistribuidos–Clase07 13/82

Diagnóstico: razonemos estas preguntas
ACTIVIDAD
1. ¿Los estados de una misma compra deben conservar un orden?
2. ¿Las compras de Sofía y Tomás necesitan un orden global único?
3. Si QR cae después de emitir el código, ¿qué debe reaparecer al reiniciar?
4. Si la base guardó la compra pero el hecho no se publicó, ¿cómo se enteran QR y Analítica?
5. Si una partición acumula trabajo, ¿agregar consumidores siempre la divide?
SistemasDistribuidos–Clase07 14/82

| Puente | desde | la clase | 6: tres historias | distintas |
| ------ | ----- | -------- | ----------------- | --------- |
NÚCLEO
| Objeto   |     | Quiénloadministra |     | Paraquésirve                          |
| -------- | --- | ----------------- | --- | ------------------------------------- |
| WALlocal |     | motordelabase     |     | recuperarpáginasytransaccionesdelnodo |
Outbox aplicación+baselocal conservar,juntoconelnegocio,laintenciónde
publicar
Logdistribuido clusterdebrokers preservar y compartir registros para múltiples
consumidores
Checkout no “ejecuta el WAL”: solicita una transacción; el motor usa WAL parael commit local. La outbox enlaza
esecommitconellogdistribuido,queotrosprocesospuedenleeryreleer.
SistemasDistribuidos–Clase07 15/82

| Actividad: | ¿qué ocurre | antes del | OK y qué | puede ocurrir |
| ---------- | ----------- | --------- | -------- | ------------- |
después?
ACTIVIDAD
| Clasifique cada   | efecto de 9F2A:     |                |       |     |
| ----------------- | ------------------- | -------------- | ----- | --- |
| Efecto            |                     | ¿Antes del OK? | Razón |     |
| Pedido durable    |                     |                |       |     |
| Reserva de        | stock               |                |       |     |
| Autorización      | de pago             |                |       |     |
| Intención durable | de publicar         |                |       |     |
| Emitir QR         |                     |                |       |     |
| Actualizar        | analítica / ranking |                |       |     |
SistemasDistribuidos–Clase07 16/82

Respuesta: qué debe ocurrir antes y después del OK
NÚCLEO
Efecto Momento Justificación
Pedido,reservaypago antesdelOK hacen verdadera la promesa “compra
confirmada”
Intencióndurabledepublicar antesdelOK evitaconfirmarsinunasalidarecupera-
ble
QR,notificaciónyanalítica despuésdelOK pueden demorarse o reintentarse sin
invalidarlacompra
Para1C7B,elpagoesrechazado: antesderesponderdebenquedarregistradaslaliberacióndelareservaylacancelación.
NodebeemitirseQR.
SistemasDistribuidos–Clase07 17/82

| Camino | crítico: | lo que | el usuario | debe esperar |
| ------ | -------- | ------ | ---------- | ------------ |
NÚCLEO
Definición
El camino crítico de la petición es la secuencia de pasos que debe terminar antes de poder
responder correctamente al usuario. Su duración condiciona la latencia del Checkout.
Confirmar
|     | Petición | Reservar | Autorizar |     |
| --- | -------- | -------- | --------- | --- |
pedido 200OK
|     | 9F2A | stock | pago |     |
| --- | ---- | ----- | ---- | --- |
+intención
Criterio
Si la caída de un paso vuelve falsa la frase “compra confirmada”, ese paso pertenece al camino
crítico.
SistemasDistribuidos–Clase07 18/82

| Efectos | asincrónicos: | necesarios, | pero fuera | de la espera |
| ------- | ------------- | ----------- | ---------- | ------------ |
NÚCLEO
| Qué significa | asincrónico | aquí |     |     |
| ------------- | ----------- | ---- | --- | --- |
Checkout no espera que el efecto termine para responder. El efecto sigue siendo necesario, pero
puede ejecutarse después, reintentarse y recuperarse de forma independiente.
GenerarQR
Evento
9F2A
preservado Notificar
confirmada
OrderConfirmed
Analítica
Asíncrononosignificaprescindible. Significaqueelefectonodeterminalavalidezdelacompra
| y puede | progresar a su | propio ritmo. |     |     |
| ------- | -------------- | ------------- | --- | --- |
SistemasDistribuidos–Clase07 19/82

| Capítulo              | 1 —         | publicar | el  |
| --------------------- | ----------- | -------- | --- |
| hecho                 | sin esperar | a todos  |     |
| Sistemas Distribuidos |             |          |     |

| Capítulo | 1: dónde | estamos | en la historia |     |     |
| -------- | -------- | ------- | -------------- | --- | --- |
NÚCLEO
Sofía ya recibió “compra confirmada”. QR y Analítica todavía no terminaron y no deben
| mantener                       | bloqueado al | checkout. |              |                     |           |
| ------------------------------ | ------------ | --------- | ------------ | ------------------- | --------- |
| Necesidad                      |              |           | Pregunta     | del capítulo        |           |
| Conservar OrderConfirmed(9F2A) |              | fuera     | de la        |                     |           |
|                                |              |           | ¿Qué actores | publican, conservan | y leen el |
memoria del productor y permitir que varios hecho, y qué significa que el broker lo haya
| lectores lo procesen | a ritmos | distintos. | confirmado? |     |     |
| -------------------- | -------- | ---------- | ----------- | --- | --- |
SistemasDistribuidos–Clase07 21/82

| Arquitectura | mínima: | productor, | brokers | y consumidores |
| ------------ | ------- | ---------- | ------- | -------------- |
NÚCLEO
QR
|     | OrderWorkflowpublica | Clusterdebrokers   |     |     |
| --- | -------------------- | ------------------ | --- | --- |
|     | productor            | historiapreservada |     |     |
Analítica
El productor agrega un registro a una historia compartida y persistida; cada aplicación
consumidora conserva su propio progreso y puede recuperarse de manera independiente.
SistemasDistribuidos–Clase07 22/82

| Kafka: | una implementación |     | real del | modelo |     |     |
| ------ | ------------------ | --- | -------- | ------ | --- | --- |
CASOREAL
ApacheKafkaesunaplataformadistribuidadeeventstreaming: productorespublicanregistros,
brokers los conservan en logs particionados y consumidores los leen con progreso propio.
| Qué problema | resuelve    | aquí               | Qué | no resuelve    | automáticamente     |        |
| ------------ | ----------- | ------------------ | --- | -------------- | ------------------- | ------ |
|              |             |                    | No  | decide la key, | no hace idempotente | el QR, |
| Desacoplar   | el checkout | de QR y Analítica, |     |                |                     |        |
conservar el hecho y permitir replay después de no elimina dual write y no define la semántica
| una caída. |     |                              | de  | “compra confirmada”. |     |       |
| ---------- | --- | ---------------------------- | --- | -------------------- | --- | ----- |
|            |     | SistemasDistribuidos–Clase07 |     |                      |     | 23/82 |

Del software al hardware: recorrido mínimo de un evento
NÚCLEO
Nodo Proceso Recursosutilizados
Aplicación productor elcódigoejecutaenCPU;RAMformaelbatch;NIC
loenvía
Brokerlíder broker NICrecibe;CPUvalida;RAM/pagecachebufferiza;
SSDodiscoconservasegmentos
Brokerseguidor réplica NICrecibelacopia;CPUverifica;mediopersistente
laconserva
Consumidor lector NICrecibelote;RAMbufferiza;CPUejecutalógica;
DB/SSDconservaelefecto
“Publicarunevento”sematerializacomoinstrucciones,buffers,paquetesyescriturasenvariosnodos. Elprocesousa
losrecursos;laCPUnoeselproductornielbroker.
SistemasDistribuidos–Clase07 24/82

| Capítulo | 2 —         | ordenar | cada     |
| -------- | ----------- | ------- | -------- |
| compra   | sin ordenar |         | el mundo |
entero
Sistemas Distribuidos

| Capítulo | 2: dónde | estamos | en la historia |     |     |
| -------- | -------- | ------- | -------------- | --- | --- |
NÚCLEO
El hecho ya puede publicarse. Ahora debemos mantener coherente el ciclo de vida de cada
pedido sin obligar a que todas las compras del Festival formen una sola fila global.
| Pregunta |     |     | Decisión |     |     |
| -------- | --- | --- | -------- | --- | --- |
¿Qué eventos deben permanecer juntos porque Elegir una key, una partición y un ámbito de
|             |               |     | orden compatible | con causalidad | y carga. |
| ----------- | ------------- | --- | ---------------- | -------------- | -------- |
| uno depende | del anterior? |     |                  |                |          |
SistemasDistribuidos–Clase07 26/82

Vocabulario previo del capítulo: topic y partición
NÚCLEO
Topic: categoría lógica de registros con un contrato común; usaremos orders.lifecycle.
Partición: uno de los logs ordenados que componen el topic. Se almacena, replica y consume
como unidad.
Límite
El orden existe dentro de cada partición. No aparece un orden global gratuito entre particiones
diferentes.
SistemasDistribuidos–Clase07 27/82

P0, P1 y P2: tres particiones del mismo topic
NÚCLEO
P0: [105][106][107]...
Topic
P1: [221][222][223]...
orders.lifecycle
P2: [417][418][419][420]...
Losoffsetsseinterpretandentrodeunapartición. P0:107yP2:419noformanunordenglobal
entre sí.
SistemasDistribuidos–Clase07 28/82

| Consideremos | las dos historias |     |     |
| ------------ | ----------------- | --- | --- |
NÚCLEO
| Compra 9F2A          | — Sofía | Compra 1C7B        | — Tomás |
| -------------------- | ------- | ------------------ | ------- |
| 1. OrderCreated      |         | 1. OrderCreated    |         |
| 2. StockReserved     |         | 2. StockReserved   |         |
| 3. PaymentAuthorized |         | 3. PaymentRejected |         |
| 4. OrderConfirmed    |         | 4. StockReleased   |         |
ElQRseráunaconsecuenciaposteriordelaconfirmacióny 5. OrderCancelled
podrárecuperarsedesdeellog.
Pregunta
¿Quérelacionescausalesdebenconservarsedentrodecadacomprayquéeventosdecomprasdistintaspuedenintercalarse?
SistemasDistribuidos–Clase07 29/82

| Causalidad | necesaria: | qué orden | no podemos | perder |
| ---------- | ---------- | --------- | ---------- | ------ |
NÚCLEO
|     | Created | Stock    | Payment    | Order     |
| --- | ------- | -------- | ---------- | --------- |
|     | 9F2A    | Reserved | Authorized | Confirmed |
Lakeynocreacausalidad: reúneenunaparticiónloseventoscuyoordendebepreservarse. Elproductorodueñodel
agregadodebeemitirloscoherentemente;uncampoaggregate_seqpuededetectarhuecos,repeticiónodesorden.
▶
OrderConfirmed(9F2A)presuponereservaypagoautorizados.
▶ OrderCancelled(1C7B)presuponerechazoyliberacióndestock.
▶
| 9F2Ay1C7Bpuedenintercalarse: |     | sonhistoriasindependientes. |     |     |
| ---------------------------- | --- | --------------------------- | --- | --- |
SistemasDistribuidos–Clase07 30/82

| Qué | es una key | de particionamiento | y   | qué no | hace |     |
| --- | ---------- | ------------------- | --- | ------ | ---- | --- |
NÚCLEO
La key es un valor estable extraído del registro. El productor aplica una función determinista
| para | seleccionar una | partición. |     |     |     |     |
| ---- | --------------- | ---------- | --- | --- | --- | --- |
partición=h(key)modP
No hace
Sí hace
|            |                   |                        | No valida | el orden   | del negocio, | no evita hot       |
| ---------- | ----------------- | ---------------------- | --------- | ---------- | ------------ | ------------------ |
| Mantiene   | la misma key      | en una misma partición |           |            |              |                    |
|            |                   |                        | keys y no | crea orden | global       | entre particiones. |
| y habilita | orden local entre | sus appends.           |           |            |              |                    |
SistemasDistribuidos–Clase07 31/82

Elección para la historia: key = order_id
NÚCLEO
Eventos9F2A
h(9F2A)mod3 P2
key=9F2A
Eventos1C7B
h(1C7B)mod3 P0
key=1C7B
No elegimos manualmente la partición para cada evento: elegimos una key estable y la función
produce siempre el mismo destino. Los valores P0 y P2 son ilustrativos para esta historia.
SistemasDistribuidos–Clase07 32/82

| Las dos | historias | ya ubicadas | en tres | particiones |
| ------- | --------- | ----------- | ------- | ----------- |
NÚCLEO
| Partición | Registrosenordendeappendyoffset |     |     |     |
| --------- | ------------------------------- | --- | --- | --- |
P0 105:Created(1C7B)→106:StockReserved→107:PaymentRejected→108:StockReleased→
109:Cancelled
P1 Ningún registro de las dos compras analizadas cae aquí. Otras keys podrían usarla en
producción.
417:Created(9F2A)→418:StockReserved→419:PaymentAuthorized→420:Confirmed
P2
| Resultado | preciso |     |     |     |
| --------- | ------- | --- | --- | --- |
Todoslosregistrosde1C7BpermanecenjuntosenP0ytodoslosde9F2AenP2. Laparticiónconservasuordendeappend;
aggregate_seqpermitecomprobarelciclodevidaesperado.
SistemasDistribuidos–Clase07 33/82

| Anatomía | de un registro | de la | historia 9F2A |
| -------- | -------------- | ----- | ------------- |
NÚCLEO
| Campo |     | Ejemployfunción |                                 |
| ----- | --- | --------------- | ------------------------------- |
|       |     | evt-9F2A-04:    | identidadúnicaparadeduplicación |
event_id
| aggregate_id     |     | 9F2A:keyqueagrupalahistoriadelpedido    |                     |
| ---------------- | --- | --------------------------------------- | ------------------- |
| aggregate_seq    |     | 4: posicióncausaldentrodelciclodevida   |                     |
| type             |     | OrderConfirmed:                         | significadodelhecho |
| partition/offset |     | P2:420: posicióndelregistrodentrodellog |                     |
Identidad de evento, identidad del pedido, secuencia del agregado y offset responden preguntas distintas; no son
intercambiables.
SistemasDistribuidos–Clase07 34/82

| Qué ganamos | y   | a qué renunciamos |     |     |     |
| ----------- | --- | ----------------- | --- | --- | --- |
NÚCLEO
| Ganamos   |         |     | Renunciamos  |             |              |
| --------- | ------- | --- | ------------ | ----------- | ------------ |
| ▶         |         |     | ▶            |             |              |
| orden por | pedido; |     | orden global | entre todas | las compras; |
▶ paralelismo entre pedidos; ▶ procesamiento paralelo dentro de una
| ▶              |                |            | misma key;    |         |                        |
| -------------- | -------------- | ---------- | ------------- | ------- | ---------------------- |
| distribución   | de escritura   | y consumo; |               |         |                        |
|                |                |            | ▶ rebalancear | una key | sin mover su historia. |
| ▶ recuperación | por partición. |            |               |         |                        |
La partición es simultáneamente una frontera de orden, capacidad y paralelismo.
SistemasDistribuidos–Clase07 35/82

| Qué hace | buena | a una key |
| -------- | ----- | --------- |
NÚCLEO
| Criterio |     | Preguntadediseño |
| -------- | --- | ---------------- |
Causalidad ¿Agrupaexactamenteloseventoscuyoordenrelativoimporta?
Estabilidad ¿Elvalorpermaneceigualdurantetodalavidadelahistoria?
Cardinalidad ¿Existensuficientesvaloresdistintospararepartirlacarga?
Granularidad ¿Evitaserializareventosquenonecesitanesperarentresí?
Distribución ¿Ningúnvalorconcentraunafraccióndesproporcionadadeltráfico?
| Observabilidad |     | ¿Podemosmedirbytes,QPSylagporkey/partición? |
| -------------- | --- | ------------------------------------------- |
Una buena key es el identificador estable más pequeño que preserva la causalidad necesaria y distribuye historias
independientes.
SistemasDistribuidos–Clase07 36/82

| Tres keys | posibles | para el mismo | evento |
| --------- | -------- | ------------- | ------ |
NÚCLEO
| Key |     | Efectosobreelorden | Efectosobrelacarga |
| --- | --- | ------------------ | ------------------ |
order_id conservaelciclodevidadelpedido muchospedidossereparten
event_id eventos del mismo pedido pueden excelentereparto,causalidadrota
separarse
| campus_id |                   | ordenatodoelcampus | unaventamasivaconcentratodo |
| --------- | ----------------- | ------------------ | --------------------------- |
| No existe | una key universal |                    |                             |
Lakeydependedelinvarianteydelordenquenecesitaesetopic. Otravistapuederequerirotrotopicounaproyección
derivada.
SistemasDistribuidos–Clase07 37/82

| Hot key | y hot partition: | una key | concentra | la carga |
| ------- | ---------------- | ------- | --------- | -------- |
EXTENSIÓN
Una hot key produce mucho más tráfico que las demás. Como una key pertenece a una sola
| partición, | puede convertirla | en una hot partition. |     |     |
| ---------- | ----------------- | --------------------- | --- | --- |
▶ order_id: muchos pedidos independientes suelen repartirse bien.
▶ campus_id: toda la venta del Festival termina en una única partición.
▶
Una entidad individual muy activa no puede paralelizarse sin redefinir qué orden necesita.
Opciones
Rediseñar la key, dividir la entidad si la semántica lo permite, aceptar el cuello de botella o agregar
| resultados | parciales en otra etapa. |     |     |     |
| ---------- | ------------------------ | --- | --- | --- |
SistemasDistribuidos–Clase07 38/82

| Agregación |     | en  | dos etapas | —   | también | llamada | jerárquica |
| ---------- | --- | --- | ---------- | --- | ------- | ------- | ---------- |
EXTENSIÓN
Idea
En vez de enviar todos los eventos a una única key global, cada partición calcula un resultado
| parcial y | una segunda |     | etapa combina    | los parciales. |           |     |         |
| --------- | ----------- | --- | ---------------- | -------------- | --------- | --- | ------- |
|           |             |     | ventas P0+ventas | P1+ventas      | P2=ventas |     | totales |
Límite
Sirve para conteos, sumas, mínimos, No sirve para dividir arbitrariamente la historia
| máximos | e histogramas |     | combinables. |     | ordenada | de  | 9F2A. |
| ------- | ------------- | --- | ------------ | --- | -------- | --- | ----- |
SistemasDistribuidos–Clase07 39/82

| De particiones | lógicas | a brokers | físicos |     |
| -------------- | ------- | --------- | ------- | --- |
NÚCLEO
|     | BrokerA     |     | BrokerB     | BrokerC     |
| --- | ----------- | --- | ----------- | ----------- |
|     | líderP0     |     | líderP1     | líderP2     |
|     | copiasP1,P2 |     | copiasP0,P2 | copiasP0,P1 |
▶ P0 contiene la historia 1C7B; su líder atiende appends y lecturas.
▶
| P2 contiene | la historia 9F2A; | su líder | está en otro nodo. |     |
| ----------- | ----------------- | -------- | ------------------ | --- |
▶ Las copias protegen frente a pérdida de nodo, pero consumen red y disco.
SistemasDistribuidos–Clase07 40/82

Publicar Confirmed(9F2A): del productor al ACK
NÚCLEO
Productor append BrokerC políticacumplida
ACK
batchenRAM líderdeP2
réplica réplica
BrokerA BrokerB
copiadeP2 copiadeP2
Ellíderrespondecuandosecumplelapolíticaconfiguradadeconfirmación. ElACKnosignificaautomáticamente“un
fsyncindependienteporevento”.
SistemasDistribuidos–Clase07 41/82

| Timeout | del productor: | el evento | 420 puede | o no existir |
| ------- | -------------- | --------- | --------- | ------------ |
NÚCLEO
| Historia A |     |     | Historia B |     |
| ---------- | --- | --- | ---------- | --- |
Confirmed(9F2A) nunca llegó a P2. P2 agregó el evento en offset 420, pero se
Reintentar es necesario. perdió el ACK. Reintentar puede duplicarlo.
La idempotencia del productor usa identidad y secuencia para que el broker reconozca reenvíos
| del mismo | batch. No deduplica | el cobro ni | el email externo. |     |
| --------- | ------------------- | ----------- | ----------------- | --- |
SistemasDistribuidos–Clase07 42/82

| Capítulo              | 3    | — QR cae: | otro |
| --------------------- | ---- | --------- | ---- |
| lector                | debe | continuar |      |
| Sistemas Distribuidos |      |           |      |

| Capítulo | 3: dónde | estamos | en la historia |     |     |     |
| -------- | -------- | ------- | -------------- | --- | --- | --- |
NÚCLEO
OrderConfirmed(9F2A) quedó en P2:420. El servicio QR emitió la entrada digital y cayó
| antes    | de registrar cuánto | había avanzado. |           |           |      |     |
| -------- | ------------------- | --------------- | --------- | --------- | ---- | --- |
| Problema |                     |                 | Conceptos | que nacen | aquí |     |
Otra instancia debe continuar sin perder la poll, position, committed offset, replay,
| confirmación | ni emitir | dos QR lógicos. |                              |               |            |        |
| ------------ | --------- | --------------- | ---------------------------- | ------------- | ---------- | ------ |
|              |           |                 | consumer                     | group, inbox, | rebalanceo | y lag. |
|              |           |                 | SistemasDistribuidos–Clase07 |               |            | 44/82  |

| Vocabulario | previo del | capítulo: | leer y guardar | progreso |
| ----------- | ---------- | --------- | -------------- | -------- |
NÚCLEO
| Término | Significadoenlahistoria                            |     |     |     |
| ------- | -------------------------------------------------- | --- | --- | --- |
| poll    | llamadaconlaqueQRsolicitaalbrokerunlotederegistros |     |     |     |
Polling repeticióndeesaconsultamientraslaaplicacióncontinúaviva
| Position | próximooffsetquelainstanciaactualintentaráleer |     |     |     |
| -------- | ---------------------------------------------- | --- | --- | --- |
Committedoffset progresodurabledesdeelqueotrainstanciapuedereiniciar
| Replay | relecturaderegistrosyavistosporunainstanciaanterior |     |     |     |
| ------ | --------------------------------------------------- | --- | --- | --- |
SistemasDistribuidos–Clase07 45/82

| Un ciclo | de consumo | de QR |
| -------- | ---------- | ----- |
NÚCLEO
emitirQR
poll recibirlote guardar
417–420 para9F2A
P2 offset421
El broker entrega registros; QR decide el efecto. Guardar offset 421 significa que la próxima
| instancia | comenzará después | de P2:420. |
| --------- | ----------------- | ---------- |
SistemasDistribuidos–Clase07 46/82

| Consumer | group: | una aplicación | lógica | con varias instancias |
| -------- | ------ | -------------- | ------ | --------------------- |
NÚCLEO
Definición
Unconsumergroupesunconjuntodeprocesosqueejecutanlamismaaplicaciónconsumidoraycompartenun
identificador,group.id. Elbrokerreparteparticionescompletasentrelosmiembrosactivos.
|     |     | P0   | P1        | P2   |
| --- | --- | ---- | --------- | ---- |
|     |     | 1C7B | sinclaves | 9F2A |
|     |     | QR-1 | QR-2      | QR-3 |
grupoqr-service
Dentrodeungrupo,cadaparticióntieneunasolainstanciaactiva. Unainstanciapuederecibirvarias; lasinstancias
sobrantesquedanociosas.
SistemasDistribuidos–Clase07 47/82

| Dos aplicaciones | pueden | leer la misma | historia |
| ---------------- | ------ | ------------- | -------- |
NÚCLEO
| Grupo QR              |     | Grupo    | Analítica            |
| --------------------- | --- | -------- | -------------------- |
| group.id = qr-service |     | group.id | = festival-analytics |
| ▶ P0→QR-1             |     | ▶        |                      |
tambiénleeP0,P1yP2;
| ▶         |     | ▶                              |     |
| --------- | --- | ------------------------------ | --- |
| P1→QR-2   |     | mantieneoffsetsindependientes; |     |
| ▶ P2→QR-3 |     | ▶                              |     |
puedeestaratrasadoaunqueQRestéaldía.
Generaentradasdigitalesyconservasuprogresopor
partición.
Lasparticionesserepartenentreinstanciasdelmismogrupo;noserepartenentretodaslasaplicacionesdelsistema.
SistemasDistribuidos–Clase07 48/82

| Las dos | historias | vistas por | el grupo | QR  |
| ------- | --------- | ---------- | -------- | --- |
NÚCLEO
| QR-1 lee | P0  |     | QR-3 | lee P2 |
| -------- | --- | --- | ---- | ------ |
1C7BterminaenOrderCancelled. Elconsumidorregistra 9F2AalcanzaOrderConfirmed. Elconsumidorgenerauna
| quenodebeemitirentrada. |     |     | entradaQRidentificadaporelpedido. |     |
| ----------------------- | --- | --- | --------------------------------- | --- |
Consecuencia
Lospedidosseprocesanenparalelo;elefectodependedeltipodeevento,nosolodehaberleídounregistro.
SistemasDistribuidos–Clase07 49/82

El accidente de QR-3
PREDICCIÓN
QR-3 lee OrderConfirmed(9F2A) en P2:420, persiste el código de entrada y cae antes de guardar
su committed offset.
| poll | QR  | restart |
| ---- | --- | ------- |
CRASH
| 420 | emitido | desde420 |
| --- | ------- | -------- |
Predicción
¿Qué reaparece y cómo evitamos crear dos entradas válidas para la misma compra?
SistemasDistribuidos–Clase07 50/82

| At-most-once: | progreso | antes que | efecto |
| ------------- | -------- | --------- | ------ |
NÚCLEO
|     | 420→commit | 421→crash→efecto |                |
| --- | ---------- | ---------------- | -------------- |
|     | poll       |                  | nunca aplicado |
Resultado
El evento no reaparece, pero se puede perder la emisión del QR.
At-most-oncepriorizanorepetir;unafallaentreprogresoyefectopuedeproducirpérdidalógica.
SistemasDistribuidos–Clase07 51/82

| At-least-once: | efecto antes | que progreso |
| -------------- | ------------ | ------------ |
NÚCLEO
|     | 420→emitir | QR→crash→replay |
| --- | ---------- | --------------- |
poll 420
Resultado
El evento reaparece; si el efecto no es idempotente, pueden emitirse dos QR válidos.
At-least-oncefacilitaprogresofrenteafallas,peroexigeidempotenciaodeduplicacióndelefecto.
SistemasDistribuidos–Clase07 52/82

| Inbox idempotente: |     | recordar | el evento | junto con | el efecto |
| ------------------ | --- | -------- | --------- | --------- | --------- |
NÚCLEO
|     | event_id | Transacciónlocal |     | commit |     |
| --- | -------- | ---------------- | --- | ------ | --- |
DBdurable
|     | evt-9F2A-04 | inbox+QR |     | offset |     |
| --- | ----------- | -------- | --- | ------ | --- |
Si 420 reaparece, la inbox demuestra que el QR ya fue emitido. Identidad y efecto deben
| confirmarse atómicamente. |     |     |     |     |     |
| ------------------------- | --- | --- | --- | --- | --- |
SistemasDistribuidos–Clase07 53/82

| Actividad: | mueva | el crash | en la historia | 9F2A |
| ---------- | ----- | -------- | -------------- | ---- |
ACTIVIDAD
| Ubique el crash  | en cuatro       | cortes:         |         |     |
| ---------------- | --------------- | --------------- | ------- | --- |
| 1. antes de      | emitir QR;      |                 |         |     |
| 2. después       | de emitir QR,   | antes de inbox; |         |     |
| 3. después       | de inbox, antes | de committed    | offset; |     |
| 4. después       | del committed   | offset.         |         |     |
| Para cada corte, | responda:       |                 |         |     |
| ▶ ¿qué estado    | es durable?;    |                 |         |     |
▶
| ¿qué offset | reaparece?; |     |     |     |
| ----------- | ----------- | --- | --- | --- |
▶
| ¿se pierde  | o duplica    | el efecto?; |     |     |
| ----------- | ------------ | ----------- | --- | --- |
| ▶ ¿qué debe | ser atómico? |             |     |     |
SistemasDistribuidos–Clase07 54/82

Por qué aparecen rebalanceo y lag en esta historia
NÚCLEO
1. QR-3eraresponsabledeP2ycaedespuésdeemitirelQRde9F2A.
2. ElgrupodebeasignarP2aotrainstanciaparacontinuar.
3. Mientrascambialaasignación,P2siguerecibiendonuevosregistros.
Rebalanceo: ¿quiénprocesaahoraP2? Lag: ¿cuánto trabajo quedó pendiente mientras P2
noavanzabaalritmodeingreso?
Sondosconsecuenciasdelmismoaccidente: reasignarresponsabilidadymedirladeudaacumulada.
SistemasDistribuidos–Clase07 55/82

| Rebalanceo: | QR-3 cae | y P2 cambia | de dueño |
| ----------- | -------- | ----------- | -------- |
NÚCLEO
|     |     | P2 antes | QR-3  |
| --- | --- | -------- | ----- |
|     |     | 9F2A     | caído |
después
QR-2
nueva
asignación
Rebalanceo: elgrupodetectacambiosdemembresíayrecalculaquéinstanciaprocesacadapartición.
▶ elconsumopuedepausarseduranteelcambio;
▶
QR-2retomaP2desdeelcommittedoffset;
▶ laasignaciónpuedevolveracambiaranteotrafalla.
SistemasDistribuidos–Clase07 56/82

| Lag: | la deuda de | procesamiento |     | de P2 |
| ---- | ----------- | ------------- | --- | ----- |
NÚCLEO
Por qué aparece
Durante el rebalanceo y la recuperación, el broker sigue aceptando nuevos registros en P2. Si
ingresan más registros por segundo de los que QR-2 procesa, crece el trabajo pendiente.
|        |                   | L =O       | −O           |                 |
| ------ | ----------------- | ---------- | ------------ | --------------- |
|        |                   | o          | finaldeP2    | confirmadoporQR |
| Lag en | offsets: cantidad | aproximada | de registros | pendientes.     |
Lag temporal: edad de la compra confirmada más antigua que todavía no obtuvo QR.
Interpretación
El lag no es la falla original: es la señal observable de que producción y consumo avanzan a ritmos
diferentes.
SistemasDistribuidos–Clase07 57/82

| Dashboard | por | partición: | no diagnostique |     | solo el promedio |
| --------- | --- | ---------- | --------------- | --- | ---------------- |
EXTENSIÓN
|     | Partición | Ingreso/s | Proceso/s | Lag CPU    | Hipótesis          |
| --- | --------- | --------- | --------- | ---------- | ------------------ |
|     | P0        | 220       | 240       | 0 45%      | sana               |
|     | P1        | 210       | 230       | 12 48%     | fluctuación        |
|     | P2        | 520       | 260       | 19.400 91% | carga/skew/restore |
Unahotpartition,unrestoreounadependenciabasedeQRlentapuedenproducirelmismosíntomasuperficial: lag
creciente.
SistemasDistribuidos–Clase07 58/82

| Capítulo  | 4 —     | el pedido |      | existe: |
| --------- | ------- | --------- | ---- | ------- |
| el evento | también |           | debe | existir |
Sistemas Distribuidos

| Capítulo | 4: dónde | estamos | en la historia |     |     |
| -------- | -------- | ------- | -------------- | --- | --- |
NÚCLEO
Ya sabemos publicar, particionar y recuperar consumidores. Falta una falla anterior: la base
de Orders puede confirmar 9F2A y el proceso caer antes de que exista OrderConfirmed en el
broker.
| Riesgo |     |     | Necesidad |     |     |
| ------ | --- | --- | --------- | --- | --- |
El usuario ve OK, pero QR y Analítica jamás Unir el commit local y la intención de publicar
| reciben el hecho. |     |     | sin depender | de dos escrituras | remotas |
| ----------------- | --- | --- | ------------ | ----------------- | ------- |
atómicas.
SistemasDistribuidos–Clase07 60/82

| La nueva | falla: pedido | confirmado, | evento ausente |
| -------- | ------------- | ----------- | -------------- |
NÚCLEO
Checkout confirma 9F2A en la base y luego intenta publicar OrderConfirmed.
DBOrders Broker
|     | Checkout |     | CRASH |
| --- | -------- | --- | ----- |
COMMIT sinevento
Consecuencia
La compra existe, pero QR y Analítica nunca reciben el hecho.
SistemasDistribuidos–Clase07 61/82

Dual write: dos escrituras independientes para un mismo hecho
NÚCLEO
Definición
Existe dual write cuando una operación de negocio necesita modificar dos sistemas que no
comparten una transacción atómica; aquí, la base de pedidos y el broker de eventos.
| Base primero |     | Broker | primero |     |     |
| ------------ | --- | ------ | ------- | --- | --- |
|              | → → |        |         | →   | →   |
DB confirma crash la compra existe, pero Evento publicado crash o rollback los
| el evento | falta. | consumidores |     | ven una compra | que no quedó |
| --------- | ------ | ------------ | --- | -------------- | ------------ |
confirmada.
Necesitamos el mismo resultado lógico —pedido confirmado y hecho publicable—, pero no
| existe | un COMMIT único que abarque | ambas escrituras | remotas. |     |     |
| ------ | --------------------------- | ---------------- | -------- | --- | --- |
SistemasDistribuidos–Clase07 62/82

Transactional outbox: el pedido y su evento nacen juntos
NÚCLEO
UnaoutboxtransaccionalesunatabladelamismabasedondeCheckoutguardaeleventopendiente. Lafilaoutbox
seinsertaenelmismoCOMMITlocalqueconfirmaelpedido.
Unatransacción
Checkout pedido+ DBdurable
filaoutbox
SielCOMMITexiste,existenjuntoselnegocioylaintencióndepublicar. Siaborta,noexisteninguno.
Qué deja abierto
Otroprocesodebepublicarlafila. Puederepetirla,porloquelosconsumidoresposterioresnecesitanidentidade
idempotencia.
SistemasDistribuidos–Clase07 63/82

| Relay, polling | y CDC: | cómo | sale el evento | de la outbox |
| -------------- | ------ | ---- | -------------- | ------------ |
NÚCLEO
| Término | Definición |     |     |     |
| ------- | ---------- | --- | --- | --- |
Relay Procesoquetomafilaspendientesdelaoutboxylaspublicaenelbroker.
Polling Elrelayconsultaperiódicamentelatablaoutboxparaencontrartrabajo.
| CDC | ChangeDataCapture: |     | capturacambiosyaconfirmadosenunabase. |     |
| --- | ------------------ | --- | ------------------------------------- | --- |
WAL/binlog Registros internos desde los cuales una herramienta CDC puede observar esos
cambios.
Downstream Componentesubicadosmásadelanteenelflujo,comoQRoAnalítica,quereciben
yprocesanelevento.
Límite
ElrelaypuedepublicaryperderelACK;alreintentar,elbrokerpuederecibirotracopia. Laoutboxevita“DBsí/evento
no”,peronoeliminaduplicadosenlosconsumidoresposteriores.
SistemasDistribuidos–Clase07 64/82

| Debezium: | un relay | basado | en CDC |     |
| --------- | -------- | ------ | ------ | --- |
CASOREAL
Debezium es una plataforma de conectoresCDC. Puede leer cambios confirmados desde WAL
o binlog y transformarlos en eventos publicados, frecuentemente en Kafka.
|     | DBOrders | WAL/binlog | Debezium | Topic |
| --- | -------- | ---------- | -------- | ----- |
Debezium es una implementación posible del relay; la idea arquitectónica es la outbox durable
| más publicación | repetible. |     |     |     |
| --------------- | ---------- | --- | --- | --- |
SistemasDistribuidos–Clase07 65/82

| Flujo completo |     | de 9F2A: | outbox, | broker | e inbox |
| -------------- | --- | -------- | ------- | ------ | ------- |
NÚCLEO
|     |     | DB  |     |     | DBQR |
| --- | --- | --- | --- | --- | ---- |
Broker
|     | Checkout | pedido+ | Relay |     | QR-2 inbox+ |
| --- | -------- | ------- | ----- | --- | ----------- |
P2
|     |     | outbox |     |     | código |
| --- | --- | ------ | --- | --- | ------ |
▶
Outboxvuelvedurablelaintencióndepublicar.
▶ Elbrokerconservaypermitereplay.
▶
InboxvuelveidempotentelaemisióndelQR.
▶ Elmismoevent_idconectaelrecorridoextremoaextremo.
SistemasDistribuidos–Clase07 66/82

| Exactly-once: |     | declare el objeto | y la frontera |     |     |
| ------------- | --- | ----------------- | ------------- | --- | --- |
NÚCLEO
“Exactly-once” no es una propiedad universal. Debe especificar qué estado, outputs y progreso
| se confirman  | juntos.            |                   |               |               |                      |
| ------------- | ------------------ | ----------------- | ------------- | ------------- | -------------------- |
|               |                    |                   | Fuera de      | la frontera   |                      |
| Dentro        | de la frontera     |                   |               |               |                      |
|               |                    |                   | Email, pago   | o API externa | requieren identidad, |
| Input, estado | local, output      | y progreso pueden |               |               |                      |
|               |                    |                   | idempotencia, | deduplicación | o reconciliación     |
| participar    | en una transacción | coordinada.       |               |               |                      |
propia.
SistemasDistribuidos–Clase07 67/82

| Capítulo | 5 —          | Analítica | debe      |
| -------- | ------------ | --------- | --------- |
| operar   | una historia |           | que nunca |
termina
Sistemas Distribuidos

| Capítulo | 5: dónde | estamos | en la historia |
| -------- | -------- | ------- | -------------- |
NÚCLEO
Pedidos y QR ya son recuperables. Analítica lee el mismo log con otro grupo y debe producir
el tablero de ventas del Festival mientras siguen llegando eventos atrasados.
| Pregunta |     |     | Nueva dificultad |
| -------- | --- | --- | ---------------- |
¿Cómo convertir una secuencia ilimitada en La hora de ocurrencia y la hora de
procesamiento pueden ser diferentes.
| resultados finitos | por minuto | o por ventana? |     |
| ------------------ | ---------- | -------------- | --- |
SistemasDistribuidos–Clase07 69/82

| Stream | y pipeline: |     | Analítica | procesa | la venta | del Festival |
| ------ | ----------- | --- | --------- | ------- | -------- | ------------ |
NÚCLEO
| Stream: | secuencia | potencialmente | ilimitada | de eventos. |     |     |
| ------- | --------- | -------------- | --------- | ----------- | --- | --- |
Pipeline: etapas que filtran, transforman, agrupan o enrutan esos eventos.
|     |     | topic  | filtrar | agrupar  | sumar       | tablero  |
| --- | --- | ------ | ------- | -------- | ----------- | -------- |
|     |     | orders | estados | 5minutos | ventasnetas | Festival |
SistemasDistribuidos–Clase07 70/82

| Stateless | y stateful: | qué debe | recordar | Analítica |
| --------- | ----------- | -------- | -------- | --------- |
NÚCLEO
| Stateless |     |     | Stateful |     |
| --------- | --- | --- | -------- | --- |
Transforma cada registro aislado: normalizar Mantiene información entre eventos: ventas
moneda, validar esquema o seleccionar campos. confirmadas menos cancelaciones por ventana.
El estado puede vivir en RAM para velocidad, en SSD local para recuperación rápida y en un
| changelog | remoto para reconstrucción | si se | pierde el nodo. |     |
| --------- | -------------------------- | ----- | --------------- | --- |
SistemasDistribuidos–Clase07 71/82

| Event | time y processing |     | time: 1C7B | llega tarde |
| ----- | ----------------- | --- | ---------- | ----------- |
NÚCLEO
| Event      | time: cuándo      | ocurrió el hecho | en el dominio. |           |
| ---------- | ----------------- | ---------------- | -------------- | --------- |
| Processing | time: cuándo      | lo procesa       | Analítica.     |           |
|            | Cancelled(1C7B)   |                  |                | procesado |
|            | eventtime12:04:50 |                  |                | 12:07:12  |
tiempo
broker
12:07:10
Consecuencia
Si la ventana 12:00–12:05 ya fue publicada, Analítica debe corregirla, reabrirla, descartar el evento o
| enviarlo a | una salida lateral | según la | política elegida. |     |
| ---------- | ------------------ | -------- | ----------------- | --- |
SistemasDistribuidos–Clase07 72/82

Watermark: estimar cuánto avanzó el tiempo de los eventos
NÚCLEO
Problema
Analíticanopuedeesperarparasiempreporuneventoantiguo,perollegadayocurrencianotienenelmismoorden.
Unaestrategiacomúncalcula,porpartición,elmayoreventtime observadomenosunmargendedesordenpermitido
∆:
Wi=max(eventtimeenPi)−∆.
Elprogresoglobalquedalimitadoporlaparticiónactivamásatrasada:
W=minWi.
i
Lectura
SiW>12:05,seconsiderarazonablecerrar12:00–12:05. Esunapolíticadeespera,nounapromesadequejamásllegará
algoanterior.
SistemasDistribuidos–Clase07 73/82

| Ejemplo | de watermark | para la ventana | 12:00–12:05 |
| ------- | ------------ | --------------- | ----------- |
NÚCLEO
Supongaunmargendedesordenpermitidode∆=2minutos:
|     | Partición | Mayoreventtimeobservado | Watermarklocal |
| --- | --------- | ----------------------- | -------------- |
|     | P0        | 12:07:20                | 12:05:20       |
|     | P1        | 12:07:10                | 12:05:10       |
|     | P2        | 12:06:50                | 12:04:50       |
W=min(12:05:20,12:05:10,12:04:50)=12:04:50
Decisión
Todavíanosecierralaventana: P2mantieneelprogresoglobalantesde12:05. CuandoP2avancelosuficiente,Wsuperará
12:05. SidespuésllegaCancelled(1C7B)coneventtime12:04:50,serálatedata.
Una partición sin eventos puede marcarse temporalmente como inactiva; de lo contrario, podría frenar el watermark
globaldeformaindefinida.
SistemasDistribuidos–Clase07 74/82

| SLO: cuánto | atraso | estamos | dispuestos | a aceptar |
| ----------- | ------ | ------- | ---------- | --------- |
NÚCLEO
SLO — Service Level Objective, u objetivo de nivel de servicio: meta cuantificable que
| el sistema | intenta cumplir | durante un | período. |     |
| ---------- | --------------- | ---------- | -------- | --- |
▶
QR: “el 99% de las compras confirmadas recibe su código en menos de 30 segundos”.
▶ Analítica: “el lag temporal p95 permanece por debajo de 60 segundos”.
▶
Ranking: “las posiciones visibles tienen menos de 10 segundos de atraso durante el 99% de la
hora pico”.
El SLO convierte “debe ser rápido” en una condición medible para dimensionar particiones,
| consumidores, | rebalances | y políticas | de late data. |     |
| ------------- | ---------- | ----------- | ------------- | --- |
SistemasDistribuidos–Clase07 75/82

| Transferencia: |     | ranking | de Asteria | Online |
| -------------- | --- | ------- | ---------- | ------ |
ACTIVIDAD
| Ahora transfiera | el               | método fuera    | del Festival:     |                 |
| ---------------- | ---------------- | --------------- | ----------------- | --------------- |
| 1. evento        | y event_id;      |                 |                   |                 |
| 2. key y         | orden necesario; |                 |                   |                 |
| 3. número        | inicial          | de particiones; |                   |                 |
| 4. estado        | que conserva     | el consumidor;  |                   |                 |
| 5. política      | de late          | data;           |                   |                 |
| 6. SLO de        | atraso           | —objetivo       | medible y ventana | de evaluación—; |
| 7. prueba        | de crash,        | replay y        | rebalanceo.       |                 |
| Pregunta         | de transferencia |                 |                   |                 |
¿Qué cambia si el ranking entrega premios irreversibles al cerrar la temporada?
SistemasDistribuidos–Clase07 76/82

La historia completa de 9F2A y 1C7B
NÚCLEO
1. Checkoutcoordinalospasoscríticosyguardapedido+intencióndepublicar.
2. Relaypublicaeventosconkey=order_id.
3. 9F2AllegaaP2y1C7BaP0consecuenciadeagregadoverificable.
4. QRyAnalíticaleenlamismahistoriaengruposindependientes.
5. SiQR-3cae,QR-2retomaP2desdeelcommittedoffset.
6. InboxevitadosQRlógicosantereplay.
7. Analíticacalculaventanasporeventtimeycorrigelatedatasegúnpolítica.
Cadaconceptorespondióaunafallaconcretadelasmismasdoscompras.
SistemasDistribuidos–Clase07 77/82

| Método | reusable |     | para | diseñar |     | un pipeline |
| ------ | -------- | --- | ---- | ------- | --- | ----------- |
NÚCLEO
| Para cada  | flujo, responda |           | en este        | orden:       |               |             |
| ---------- | --------------- | --------- | -------------- | ------------ | ------------- | ----------- |
| 1. ¿Cuál   | es el hecho     | y         | cuál su        | identidad    | estable?      |             |
| 2. ¿Qué    | eventos         | requieren | orden          | relativo?    |               |             |
| 3. ¿Qué    | key agrupa      | esa       | causalidad     | sin          | concentrar    | carga?      |
| 4. ¿Qué    | particiones     | y         | consumidores   |              | necesita el   | throughput? |
| 5. ¿Dónde  | se guarda       | el        | progreso       | recuperable? |               |             |
| 6. ¿Qué    | efecto          | debe ser  | idempotente?   |              |               |             |
| 7. ¿Existe | dual            | write?    | ¿Outbox/inbox? |              |               |             |
| 8. ¿Qué    | métricas        | detectan  | skew,          | lag          | y rebalances? |             |
SistemasDistribuidos–Clase07 78/82

| Volvemos | al Diagnóstico |     | inicial |     |
| -------- | -------------- | --- | ------- | --- |
ACTIVIDAD
Después de entender lo anterior podríamos responder con criterio:
1. ¿Por qué order_id reúne la historia y qué condición adicional preserva la causalidad?
| 2. ¿Por | qué campus_id | puede | producir una | hot partition? |
| ------- | ------------- | ----- | ------------ | -------------- |
3. ¿Qué reaparece si el QR fue durable pero el committed offset no?
| 4. ¿Qué | resuelve outbox | y qué        | deja abierto? |                |
| ------- | --------------- | ------------ | ------------- | -------------- |
| 5. ¿Por | qué agregar     | consumidores | no divide     | una partición? |
SistemasDistribuidos–Clase07 79/82

| Síntesis: | cinco ideas | que deben | quedar |
| --------- | ----------- | --------- | ------ |
NÚCLEO
1. La key reúne el ámbito de orden; el productor y la secuencia del agregado deben representar
| correctamente | la causalidad. |     |     |
| ------------- | -------------- | --- | --- |
2. Una partición ofrece orden local y es unidad de almacenamiento y paralelismo.
3. Committed offset registra progreso; no confirma por sí solo el efecto de negocio.
4. Replay es normal; idempotencia e inbox vuelven seguro el efecto.
5. Outbox conecta una transacción local con publicación repetible, no con exactly-once universal.
SistemasDistribuidos–Clase07 80/82

Exit ticket
ACTIVIDAD
| En una hoja, | responda:     |                          |                   |
| ------------ | ------------- | ------------------------ | ----------------- |
| 1. Defina    | una buena key | en una frase y aplíquela | a 9F2A.           |
| 2. Dibuje    | dónde quedan  | 9F2A y 1C7B en           | tres particiones. |
3. Explique qué ocurre si QR cae después de emitir la entrada y antes del committed offset.
4. Nombre una métrica que distinga hot partition de atraso general.
SistemasDistribuidos–Clase07 81/82

Fuentes y material de respaldo
RESPALDO
▶
ApacheKafkaDocumentation: topics,partitions,producer,consumergroupsydeliverysemantics.
▶
Krepsetal.: Kafka: aDistributedMessagingSystemforLogProcessing.
▶
Kleppmann: DesigningData-IntensiveApplications,capítulossobrelogs,particionadoystreamprocessing.
▶
DebeziumDocumentation: transactionaloutboxyCDC.
▶
ApacheFlinkDocumentation: eventtime,watermarksystatefulprocessing.
Leasiempreladocumentaciónbuscandocincopreguntas: orden,progreso,falla,efectoycostooperacional.
SistemasDistribuidos–Clase07 82/82