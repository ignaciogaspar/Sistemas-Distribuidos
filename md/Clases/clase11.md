| Sistemas Distribuidos |     |     |
| --------------------- | --- | --- |
Clase 11 — Circuit breakers, backpressure, bulkheads, retries y
graceful degradation
| Departamento de        | Computación |           |
| ---------------------- | ----------- | --------- |
| Ingeniería Informática | – Comisión  | A – Pilar |
| Equipo docente –       | 2026        |           |

| Apertura: | de la | observación | a   |
| --------- | ----- | ----------- | --- |
la contención
Sistemas Distribuidos

| Dónde           | nos quedamos  |        |          |                  |
| --------------- | ------------- | ------ | -------- | ---------------- |
| La clase pasada | terminó       | con la | cascada  | de cuatro pasos: |
| 1. Una          | base de datos | aguas  | abajo se | degrada.         |
| 2. El servicio  | intermedio    | agota  | su pool  | de hilos.        |
3. Los clientes aguas arriba reintentan y amplifican el tráfico.
| 4. El borde | empieza | a devolver | HTTP | 504. |
| ----------- | ------- | ---------- | ---- | ---- |
Ya sabemos ver esa cascada. Esta clase: qué hace el sistema mientras ocurre.
El retry
En clases anteriores era la solución a un mensaje perdido. Frente a una dependencia
| saturada, | es el combustible. |     |     |     |
| --------- | ------------------ | --- | --- | --- |
SistemasDistribuidos–Clase11 3/61

Falta, error y falla
crackstopper crackstopper
Falta (fault) Error Falla (failure)
estadointernoincorrecto: comportamientovisible: elsistemanoresponde:
unaconexiónno lleganexcepcionesenvez poolagotado,quioscos
vuelvealpool devuelos decheck-inparados
▶ La grieta avanza de un eslabón al siguiente; el acoplamiento la acelera.
▶ Las dependencias van a fallar. Lo que se diseña es dónde se corta, como las zonas
de deformación de un auto.
▶ “Failure is in the eye of the beholder”: para un monitoreo que solo mira una página de
estado, un servicio con todos los hilos bloqueados está sano.
Quioscosdecheck-in: terminalesdeautoserviciodelosaeropuertos. Nygard,ReleaseIt! (2018),cap.3;ejemplosdelcasode
laaerolínea(cap.2). Apunte: “Introducción”.
SistemasDistribuidos–Clase11 4/61

Anatomía de una
falla en cascada
Sistemas Distribuidos

| Falla | rápida y | falla lenta |     |
| ----- | -------- | ----------- | --- |
elhilovuelveenmilisegundosyatiendeotrospedidos
Conexiónrechazada
| Respuestaquenollega |     | elhiloquedabloqueadoesperando |     |
| ------------------- | --- | ----------------------------- | --- |
tiempo
| Nygard,   | cap. 4          |                |                    |
| --------- | --------------- | -------------- | ------------------ |
| “Clearly, | a slow response | is a lot worse | than no response.” |
Una dependencia lenta es más peligrosa que una caída: la caída libera los recursos del
| llamante;         | la lentitud   | los retiene.                      |     |
| ----------------- | ------------- | --------------------------------- | --- |
| Nygard,ReleaseIt! | (2018),cap.4. | Apunte: “Fallarápidayfallalenta”. |     |
SistemasDistribuidos–Clase11 6/61

| Hilos bloqueados: |               | el  | pool                          | se agota |
| ----------------- | ------------- | --- | ----------------------------- | -------- |
|                   | poolde8hilos: |     | todosesperandoaunadependencia |          |
CPU:casinula
(nadiecalcula,todosesperan)
pedidosencola
▶
Casi ninguna caída real es un proceso que termina: es un proceso vivo con todos sus
| hilos esperando. | El  | sistema | no “se | cayó”, se colgó. |
| ---------------- | --- | ------- | ------ | ---------------- |
▶
Con el sistema saturado la CPU está baja. La saturación se mide sobre el recurso
| que se agota: | hilos | ocupados | contra | hilos disponibles. |
| ------------- | ----- | -------- | ------ | ------------------ |
Nygard,ReleaseIt! (2018),cap.4,BlockedThreads. Apunte: “Hilosbloqueadosypoolsagotados”.
SistemasDistribuidos–Clase11 7/61

| Reacciones | en cadena | y capacidades | desbalanceadas |
| ---------- | --------- | ------------- | -------------- |
cadacajaesunservidordetrásdeunbalanceador;elnúmero,lapartedelacargaqueatiende
|     | servidor1 | servidor2 servidor3 servidor4 | servidor5 |
| --- | --------- | ----------------------------- | --------- |
antes
|     | 20% | 20% 20% 20% | 20% |
| --- | --- | ----------- | --- |
cargadecadauno
cadasobrevivientecarga
un25%más(25/20)
después
|     | caído | 25% 25% 25% | 25% |
| --- | ----- | ----------- | --- |
dequecaeuno
▶
Reacción en cadena: si la causa era una condición de carga, los sobrevivientes ahora
son más propensos a fallar, y cada baja recarga a los demás. Como el defecto está en
| todos, solo | se detiene corrigiéndolo. |     |     |
| ----------- | ------------------------- | --- | --- |
▶ Capacidades desbalanceadas: un front end con 3000 hilos frente a un servicio con
450. Una promoción que lleve tráfico a ese camino lo desborda.
Nygard,ReleaseIt! (2018),cap.4(elejemplooriginalusaochoservidores). Apunte: “Reaccionesencadena”.
SistemasDistribuidos–Clase11 8/61

| Ley de | Little: | una peluquería |     |     |     |
| ------ | ------- | -------------- | --- | --- | --- |
L=λ·W
peluquería
▶
| λ: cuántos   | clientes    | llegan por hora.   |     |         |        |
| ------------ | ----------- | ------------------ | --- | ------- | ------ |
| ▶            |             |                    |     |         | espera |
| W: cuántas   | horas       | se queda cada uno. |     |         |        |
| ▶ L: cuántos | hay adentro | a la vez.          |     | 2sillas |        |
Llegan 2 por hora y cada uno se queda 1,5 Clientes = pedidos · sillas = hilos del pool ·
horas: L=2×1,5=3.
|     |     |     | tiempo adentro | = tiempo | que un pedido ocupa |
| --- | --- | --- | -------------- | -------- | ------------------- |
un hilo.
Con solo 2 sillas no da abasto, aunque nada esté “roto”. Si cada corte se alarga, W sube
| y L sube | sin que llegue | un solo cliente | más. |     |     |
| -------- | -------------- | --------------- | ---- | --- | --- |
Little(1961). Laanalogíaesdelacátedra. Apunte: “LaleydeLittleaplicadaalpooldehilos”.
SistemasDistribuidos–Clase11 9/61

| La cuenta | con un | pool de | hilos |     |     |     |     |
| --------- | ------ | ------- | ----- | --- | --- | --- | --- |
Un servicio recibe λ = 5 pedidos por segundo y atiende con 8 hilos. Cada pedido hace 5
| llamadas seguidas | a una dependencia. |     |        |         |        |           |          |
| ----------------- | ------------------ | --- | ------ | ------- | ------ | --------- | -------- |
|                   |                    |     | W (por | pedido) | L = λW | ¿Alcanzan | 8 hilos? |
| Dependencia       | sana               |     |        | 0,05 s  | 0,25   | sobran    |          |
5×2
| Dependencia | con 2 s | por llamada |     | = 10 s | 50  | no: | satura |
| ----------- | ------- | ----------- | --- | ------ | --- | --- | ------ |
Consecuencia
Saturado, el servicio sostiene 8/10 = 0,8 pedidos por segundo, no 5. El resto espera en una
| cola. Cambió | W, no λ: | nadie se cayó. |     |     |     |     |     |
| ------------ | -------- | -------------- | --- | --- | --- | --- | --- |
El cliente ve tiempo de residencia = tiempo en cola + tiempo de procesamiento. Medir solo el
| segundo es                     | la trampa.                  |     |                              |     |     |     |       |
| ------------------------------ | --------------------------- | --- | ---------------------------- | --- | --- | --- | ----- |
| Little(1961);Nygard,ReleaseIt! | (2018),cap.9,DemandControl. |     |                              |     |     |     |       |
|                                |                             |     | SistemasDistribuidos–Clase11 |     |     |     | 10/61 |

| Resiliencia | y sus | herramientas |
| ----------- | ----- | ------------ |
Sistemas Distribuidos

| Dónde | corta | cada | herramienta |     |     |     |     |
| ----- | ----- | ---- | ----------- | --- | --- | --- | --- |
Resiliencia: seguir funcionando ante fallas parciales, aunque sea degradado. No es que
| nada falle: | es contener | la      | propagación. |          |         |          |           |
| ----------- | ----------- | ------- | ------------ | -------- | ------- | -------- | --------- |
|             |             | timeout |              | bulkhead |         | fallback |           |
|             | dependencia |         | hilos        |          | pool    |          | error     |
|             |             |         | bloqueados   |          | agotado |          | alusuario |
lentaocaída
backpressure,
|     |     | circuitbreaker | reintentos(concuidado) |     |     |     |     |
| --- | --- | -------------- | ---------------------- | --- | --- | --- | --- |
loadshedding
| Timeout   |     |     | cortalaesperaindefinida.                        |     |     |     |     |
| --------- | --- | --- | ----------------------------------------------- | --- | --- | --- | --- |
| Reintento |     |     | recuperalasfallastransitorias,acostademáscarga. |     |     |     |     |
Circuit breaker dejadeinsistircontraunadependenciaqueyasesabeenferma.
| Fallback |     |     | respondealgoútilenlugardelerror. |     |     |     |     |
| -------- | --- | --- | -------------------------------- | --- | --- | --- | --- |
Bulkhead impidequeunaparteagotelosrecursosquecompartenlasdemás.
Backpressure, load shedding frenanorechazanlademandamayorquelacapacidad.
Apunte: “Resilienciaysusherramientas”.
SistemasDistribuidos–Clase11 12/61

| El sistema | de la | demo |
| ---------- | ----- | ---- |
Sistemas Distribuidos

| Un generador | de  | contraseñas |     |
| ------------ | --- | ----------- | --- |
upper
lower
5pedidos/s
|     |     | k6  | Generator |
| --- | --- | --- | --------- |
digit
Toxiproxy special
▶ k6 manda 5 pedidos por segundo a ritmo fijo: sigue mandando aunque el sistema se
| ponga lento. | Corta cada | pedido a | los 10 s. |
| ------------ | ---------- | -------- | --------- |
▶ El Generator arma cada contraseña llamando a cuatro servicios hijos, con un pool de
8 hilos.
▶
Toxiproxy, delante de special, inyecta las fallas sin tocar su código.
▶ Dependencia oculta (punteada): lower llama por dentro a digit.
SistemasDistribuidos–Clase11 14/61

| Cómo         | se corrieron | los experimentos: |            | escenarios |
| ------------ | ------------ | ----------------- | ---------- | ---------- |
| Un escenario | es la falla  | que Toxiproxy     | inyecta en | special:   |
| baseline:    | sin fallas.  | Es la línea de    | base.      |            |
latency 2000: cada llamada tarda 2 s más. La dependencia responde bien, pero lento.
errors 0.1 / 0.5: se corta la conexión en el 10% o el 50% de las llamadas.
intermittent: special se cae 20 s y vuelve 20 s, cinco veces. Con fast rechaza la conexión
|     | (falla rápida); | con | no responde (falla | lenta). |
| --- | --------------- | --- | ------------------ | ------- |
hang
Cada experimento duró de 3 a 4 minutos, con k6 enviando 5 pedidos por segundo. Los números de
cada slide se midieron en Prometheus sobre la ventana del experimento. La traza y la tabla de la
línea de base salen de una segunda corrida con la misma configuración.
| Cómo leer | las capturas |     |     |     |
| --------- | ------------ | --- | --- | --- |
Cada línea amarilla punteada vertical marca el momento en que se aplicó una
configuración: un escenario o un preset. Lo que está a su derecha corre con la
| configuración | nueva. |     |     |     |
| ------------- | ------ | --- | --- | --- |
SistemasDistribuidos–Clase11 15/61

| Cómo | se corrieron | los experimentos: |     | presets |     |     |
| ---- | ------------ | ----------------- | --- | ------- | --- | --- |
Un preset es la configuración de protecciones del Generator; se cambia en vivo.
| Preset                          |     |     | Timeout             | Reintentos   | Circuitbreaker   | Fallback |
| ------------------------------- | --- | --- | ------------------- | ------------ | ---------------- | -------- |
| 00-no-protection                |     |     | —                   | —            | —                | —        |
| 01-timeout-only                 |     |     | sí                  | —            | —                | —        |
| 02-timeout-retry-naive          |     |     | sí                  | 3,inmediatos | —                | —        |
| 03-timeout-retry-backoff-jitter |     |     | sí 2,backoff+jitter |              | —                | —        |
| 04-cb-consecutive-fragile       |     |     | sí                  | —            | 3fallasseguidas  | sí       |
| 04b-cb-consecutive-tolerant     |     |     | sí                  | —            | 10fallasseguidas | sí       |
| 05-cb-rate                      |     |     | sí                  | —            | tasa(50/20/50%)  | sí       |
| 06-complete                     |     |     | sí 2,backoff+jitter |              | tasa(20/10/50%)  | sí       |
|                                 |     |     | sí                  | —            | tasa(50/20/50%)  | —        |
07-cb-no-fallback
Tasa: ventana/mínimodellamadas/umbral. Timeouthaciaspecial: 100ms. Breakerabierto: 10s.
Fallback de la demo: siunallamadafallaporcualquiermotivo(timeout,errororechazodelbreaker),la
contraseñasearmasinesetipodecaracteresysemarcadegraded. Noessoloparaelbreakerabierto.
SistemasDistribuidos–Clase11 16/61

Fan-out: una traza de la línea de base
Recortedelcomienzodeunatraza. Paraloscaracteresenmayúscula,elGeneratorhacedos llamadasa
upper: cadaintento(generator.attempt)esunHTTP GETquellegaalservicioupper. Despuésrepiteel
patrónconlower,digityspecial,unotrasotro. Enlalíneadebase,cadapedidohizoenpromedio4,9
llamadasacadaserviciohijo: todapolíticaporllamadasemultiplicaporpedido.
SistemasDistribuidos–Clase11 17/61

| Línea de | base: latencia | de cada | dependencia |
| -------- | -------------- | ------- | ----------- |
▶
El p99 de upper, digit y special ronda los 40 ms. Es el dato para elegir los timeouts.
▶
lower tiene un p99 de 97 ms porque incluye la llamada interna a digit: medida desde el
| llamante, | la cadena ya | está adentro. |     |
| --------- | ------------ | ------------- | --- |
SistemasDistribuidos–Clase11 18/61

Experimento:
una dependencia lenta
Sistemas Distribuidos

| Sin protección: | el pool | se llena |
| --------------- | ------- | -------- |
Escenario: latency 2000 · Preset: 00-no-protection · líneaamarilla: seaplicaunaconfiguración
Protecciones: ninguna
Alospocossegundosdelalíneaamarilla,los8 hilosquedanocupadosynoseliberan. LeydeLittleconlo
medido: cadapedidotardóW≈10,4s,asíqueharíanfaltaL=5×10,4≈52 hilos. Secompletaron163
| pedidosen210s: 0,78 porsegundo,quees8/10,4. |     |     |
| ------------------------------------------- | --- | --- |
SistemasDistribuidos–Clase11 20/61

Sin protección: el usuario espera hasta rendirse
Escenario: latency 2000 · Preset: 00-no-protection · líneaamarilla: seaplicaunaconfiguración
Protecciones: ninguna
Lalatenciaquevek6subehasta10 s,queessupropiotimeout: enlaventanamedida,todoslospedidosdel
clienteterminaronportimeout. Lalentituddeunadependenciasevolviólafalladelservicioentero.
SistemasDistribuidos–Clase11 21/61

Sin protección: la CPU baja
Escenario: latency 2000 · Preset: 00-no-protection · líneaamarilla: seaplicaunaconfiguración
Protecciones: ninguna
LaCPUdelGeneratorcaede0,27a0,06núcleosjustocuandoelserviciodejadeatender. Loshilosesperan,
nocalculan: unpaneldeCPU“enverde”escompatibleconunserviciobloqueado.
SistemasDistribuidos–Clase11 22/61

Una respuesta lenta es peor que ninguna
Escenario: intermittent fast → hang · Preset: 00-no-protection · líneaamarilla: seaplicauna
configuración
Protecciones: ninguna
specialcae20sdecada40. Confast(hastalas14:04)fallael48%delospedidos,perorápido: elpool
casinoseusa. Conhangelpoolse llena en cada caídayel36%terminaportimeoutdelcliente.
SistemasDistribuidos–Clase11 23/61

Timeouts
Sistemas Distribuidos

| Timeout | de conexión | y timeout | de lectura |
| ------- | ----------- | --------- | ---------- |
cliente servidor
abrirconexión
timeoutdeconexión
pedido
timeoutdelectura
(readosockettimeout): elservidortrabaja
|     | esperaentredatosrecibidos |     | respuesta |
| --- | ------------------------- | --- | --------- |
▶ “Hope is not a design method”: el timeout no viene puesto. requests (Python) no corta
nunca sin el parámetro timeout; en Java, URLConnection con 0 espera para siempre.
▶
El de lectura mide el tiempo entre datos, no el total: para acotar el total hace falta un plazo
| global | (deadline). |     |     |
| ------ | ----------- | --- | --- |
Nygard,ReleaseIt! (2018),cap.5. Documentaciónderequestsydejava.net.URLConnection. Apunte: “Timeouts”.
SistemasDistribuidos–Clase11 25/61

| Cómo elegir | el valor: | p99 + margen |
| ----------- | --------- | ------------ |
p50
timeout
p99
margen
latenciadeladependencia(medidadesdeelllamante)
▶ Con la media, el timeout cortaría muchas llamadas normales; con el máximo, casi nunca
actuaría.
| ▶   |     | ≈41 |
| --- | --- | --- |
En la demo: p99 de special ms en la línea de base; los presets usan un timeout de
| lectura de 100 | ms. |     |
| -------------- | --- | --- |
▶
El percentil sale de buckets de histograma: es aproximado. Alcanza para calibrar, con margen.
| Apunte: “Cómoelegirelvalor”. | Clase10: | percentiles. |
| ---------------------------- | -------- | ------------ |
SistemasDistribuidos–Clase11 26/61

| Timeouts        | en una      | cadena | de llamadas  |     |
| --------------- | ----------- | ------ | ------------ | --- |
| Con reintentos, | una llamada | lógica | puede tardar |     |
∑
r
|     |     | T   | = (1+r)·t+ | e   |
| --- | --- | --- | ---------- | --- |
|     |     | max | i=1        | i   |
t: timeoutdecadaintento;r: cantidaddereintentos(1+rintentos);e: esperaantesdelreintentoi.
i
Tmax=3×500+300=1800ms:
tienequecabereneltimeoutdelllamante
|     | intento500ms |     | intento500ms | intento500ms |
| --- | ------------ | --- | ------------ | ------------ |
|     |              | 100 | 200          |              |
esperasentreintentos(backoff)
▶
Para el valor de partida no hace falta conocer la cadena: el p99 de B medido desde A ya
| incluye | lo que B espera | a C. |     |     |
| ------- | --------------- | ---- | --- | --- |
▶ Lo que la línea de base no muestra es el peor caso interno. Si el timeout de A es menor que
| el T | de B, A abandona | mientras | B sigue trabajando. |     |
| ---- | ---------------- | -------- | ------------------- | --- |
max
Apunte: “Timeoutsenunacadenadellamadas”.
SistemasDistribuidos–Clase11 27/61

Experimento: solo timeouts
Escenario: latency 2000 · Preset: 00-no-protection, luego 01-timeout-only · líneaamarilla: seaplica
unaconfiguración
Protecciones: 00ninguna;01solotimeout
Alaplicar01(segundalíneaamarilla),cadallamadaaspecialsecortaalos100msyelhiloselibera: elpool
bajade8aunpromediode1,5hilosocupados. Se terminó la saturación.
SistemasDistribuidos–Clase11 28/61

Experimento: solo timeouts, pero todo falla
Escenario: latency 2000 · Preset: 01-timeout-only · líneaamarilla: seaplicaunaconfiguración
Protecciones: solotimeout
El98%delospedidosrespondeHTTP500,ahorarápido(200msdemedia). Solosesalvanlospocosqueno
llegaronapedircaracteresespeciales. Fallar rápido no es funcionar: eltimeoutprotegealllamante;al
usuarioloprotegenlosmecanismosquesiguen.
SistemasDistribuidos–Clase11 29/61

Reintentos
Sistemas Distribuidos

Cuándo destruyen: la retroalimentación
máscarga respuestasmáslentas
reintentos timeoutsdelllamante
▶ El llamante abandona y reintenta “exactly when it hurts the worst”.
▶ Un backend atiende 10000 consultas por segundo y recibe 10100: rechaza 100, que vuelven
como reintentos (10200), luego 300 rechazos… Bajar la carga al nivel previo puede no alcanzar
para salir.
▶ Ayudan frente a fallas transitorias. Solo operaciones idempotentes, y solo errores
reintentables (timeouts, conexión, 5xx; no 4xx).
Nygard,ReleaseIt! (2018),caps.5y9;Beyeretal.,SiteReliabilityEngineering(2016),cap.22.
SistemasDistribuidos–Clase11 31/61

| La cuenta de | la amplificación |     |     |     |
| ------------ | ---------------- | --- | --- | --- |
Unpedidohacenllamadasycadaunafallacon
p=10%:
pedidosquefallan
P(falladelpedido)=1−(1−p)n
60%
41%
| 40% | sinreintento |     | ▶           |       |
| --- | ------------ | --- | ----------- | ----- |
|     |              |     | n=5: 1−0,95 | ≈41%. |
▶
Conunreintentolallamadafallasifallanlos
20%
|     | 4,9% con1reintento |     | dosintentos:                        | p2 =1%,yelpedido, |
| --- | ------------------ | --- | ----------------------------------- | ----------------- |
|     |                    |     | 1−0,995                             | ≈4,9%.            |
| 1   | 5                  | 10  | ▶ Costo: 1+p=1,1intentosporllamada. |                   |
llamadasporpedido(n)
Peroanteunacaída,p→1:
sisepermitenr
reintentosporllamada(acár=1),cada
llamadapasaaser1+rintentos.
Apunte: “Lacuentadelaamplificación”.
SistemasDistribuidos–Clase11 32/61

Reintentos en capas
pedidodelusuario: 1
capa1: 3intentos
capa2: 9
capa3: 27
▶ Cada capa reintenta y los factores se multiplican: 33 =27 llamadas por pedido. Con 3
reintentos (4 intentos) en tres capas, 43 =64 intentos sobre la base de datos.
▶ Decidir en qué capa se reintenta. Y un rechazo del breaker nunca se reintenta.
Beyeretal.,SiteReliabilityEngineering(2016),cap.22.
SistemasDistribuidos–Clase11 33/61

| Backoff | exponencial | y jitter |     |     |
| ------- | ----------- | -------- | --- | --- |
d = min(tope, base·mn). Base 100 ms, m = 2: el primer reintento espera 100 ms, el
n
segundo 200 ms. Pero si todos esperan lo mismo, reintentan juntos.
×3
| sinjitter |     |     | lostresjuntosalos200ms |     |
| --------- | --- | --- | ---------------------- | --- |
e∼U(0,d):
| fulljitter  |     |       | 30,120,190      |             |
| ----------- | --- | ----- | --------------- | ----------- |
| equaljitter |     |       | e=d/2+U(0,d/2): | 110,160,195 |
|             | 0ms | 100ms | 200ms           |             |
Tres clientes fallan a la vez y esperan su segundo reintento (d=200 ms). Full jitter dispersa más;
equal jitter garantiza una espera mínima. El efecto solo se nota con muchos clientes.
Brooker,ExponentialBackoffandJitter,AWSArchitectureBlog(2015). Nygard(2018),cap.4,Dogpile.
SistemasDistribuidos–Clase11 34/61

| Presupuesto | de reintentos |     |
| ----------- | ------------- | --- |
Limitar los intentos por pedido no alcanza: si la dependencia entera está sobrecargada,
todos los pedidos agotan sus intentos y la carga casi se triplica.
Dos límites (Google)
▶
| Por pedido: | como máximo | 3 intentos. |
| ----------- | ----------- | ----------- |
▶ Por cliente: reintentar solo mientras los reintentos sean menos del 10% de los
| pedidos que | envía. |     |
| ----------- | ------ | --- |
▶
Solo con el primero, los pedidos crecen hasta algo menos del triple; con los dos, en el caso
| general, 1,1 | veces. |     |
| ------------ | ------ | --- |
▶
El segundo se ajusta solo: si fallan muchos pedidos, el problema no es transitorio y se deja de
reintentar.
▶
Variante simple: un tope por proceso, por ejemplo 60 reintentos por minuto.
Beyeretal.,SiteReliabilityEngineering(2016),caps.21y22. Apunte: “Presupuestodereintentos”.
SistemasDistribuidos–Clase11 35/61

| Caso Pokémon | GO  |     |     |
| ------------ | --- | --- | --- |
▶ Se probó la carga hasta 5 veces la estimación más optimista. El lanzamiento real tuvo
| casi 50 | veces esa estimación. |     |     |
| ------- | --------------------- | --- | --- |
▶ La app reintentaba con “a single immediate retry, followed by constant backoff”.
▶
Los backends devolvían errores rápidos: eso sincronizó los reintentos. Picos de hasta
| 20 veces | el máximo global | anterior (thundering | herd). |
| -------- | ---------------- | -------------------- | ------ |
▶
Mitigación de fondo: jitter y backoff exponencial truncado en los clientes.
Lección
Muchos clientes independientes, con reintentos sin dispersión, se vuelven una sola ola.
Betheaetal.,enBeyeretal.,TheSiteReliabilityWorkbook(2018),cap.11.
SistemasDistribuidos–Clase11 36/61

Experimento: reintentos contra 10% de errores
Escenario: errors 0.1 · Preset: 00 → 02 → 03 · líneaamarilla: seaplicaunaconfiguración
Protecciones: 00ninguna · 02timeout+3reintentosinmediatos · 03timeout+2reintentosconbackoffyjitter
Sinprotecciónfallael36,8%delospedidos(lacuentaconn=4,9da40%;comonvaríadepedidoen
pedido,semidealgomenos). Con3reintentosinmediatos(02)fallael0,14%;con2reintentosconbackoffy
jitter(03),el0,55%.
SistemasDistribuidos–Clase11 37/61

Experimento: el costo de los reintentos
Escenario: errors 0.1 · Preset: 00 → 02 → 03 · líneaamarilla: seaplicaunaconfiguración
Protecciones: 00ninguna · 02timeout+3reintentosinmediatos · 03timeout+2reintentosconbackoffyjitter
Conreintentos,cadapedidohace5,4 llamadasaspecial,contra4,9enlalíneadebase: un10%más,el
1+pdelacuenta(sinprotecciónson3,9: elpedidosecortaenlaprimerafalla). Entre02y03no hay
diferencia visible: conunsoloclientenohaysincronizaciónqueromper.
SistemasDistribuidos–Clase11 38/61

Circuit breaker
Sistemas Distribuidos

La máquina de estados
fallas≥umbral(segúnlapolítica)
Closed Open
lasllamadasseejecutan; lasllamadasserechazan
secuentanlasfallas sinejecutarse
pasóeltiempo
deespera
laspruebasnecesarias
tienenéxito
unapruebafalla
Half-open
Qué es
Un componente que “falla primero” para que el sistema no falle: existe para prevenir
operaciones, no para reejecutarlas.
Nygard,ReleaseIt! (2018),cap.5,CircuitBreaker.
SistemasDistribuidos–Clase11 40/61

| Las decisiones |     | de configuración |
| -------------- | --- | ---------------- |
Qué cuenta como falla: timeouts, errores de conexión y 5xx. No los 4xx ni los errores de validación.
Con timeouts: sin timeout, la llamada lenta no termina y el breaker no ve la falla más peligrosa.
Cuándo abrir: fallas consecutivas o tasa de fallas en ventana (siguientes slides).
Cuánto tiempo abierto: muy corto, muchas pruebas fallidas; muy largo, degradación de más.
Half-open: cuántas pruebas a la vez y cuántos éxitos para cerrar.
Alcance: un breaker por dependencia y por proceso (en la demo, generator-special y los
otros tres).
Con reintentos: cada intento cuenta; un rechazo no se reintenta.
Implementación: thread-safe, con el lock solo sobre el estado, nunca sobre la llamada de red: si no,
|     | serializa todas | las llamadas. |
| --- | --------------- | ------------- |
Observabilidad: los cambios de estado se registran y se grafican.
| Nygard,ReleaseIt! | (2018),cap.5. | Apunte: “Circuitbreaker”. |
| ----------------- | ------------- | ------------------------- |
SistemasDistribuidos–Clase11 41/61

| Cuándo | abrir: | dos políticas, | paso a | paso |
| ------ | ------ | -------------- | ------ | ---- |
falla éxito
|     | Fallasconsecutivas(k=3): |       | cualquieréxitoreinicialacuenta |        |
| --- | ------------------------ | ----- | ------------------------------ | ------ |
|     |                          | 1 2 3 | 4 5 6                          | 7 abre |
|     | seguidas:                | 0 1 2 | 0 1 2                          | 3      |
Tasaenventana(últimas10,mínimo5,umbral50%)
|     |     |       | abre: | llegóalmínimo |
| --- | --- | ----- | ----- | ------------- |
|     |     | 1 2 3 | 4 5   |               |
con60%
antesnodecide,aunquelatasa
|     | tasa: | 100% 50% 67% | 75% 60% |     |
| --- | ----- | ------------ | ------- | --- |
supereelumbral
En los dos casos, abierto se rechaza todo durante el tiempo de espera; después, en half-open, una
| llamada de | prueba decide | si cierra o vuelve | a abrir. |     |
| ---------- | ------------- | ------------------ | -------- | --- |
Apunte: “Cuándoabrir: fallasconsecutivascontratasadefallas”(tablaspasoapaso).
SistemasDistribuidos–Clase11 42/61

| El dilema | de las fallas | consecutivas |     |     |
| --------- | ------------- | ------------ | --- | --- |
Con probabilidad de falla p, llamadas esperadas hasta ver k fallas seguidas:
1−pk
|     |     | E(k,p) | =   |     |
| --- | --- | ------ | --- | --- |
(1−p)pk
|     |     |     | p = 10% | p = 50% |
| --- | --- | --- | ------- | ------- |
≈
|                | k = 3        | 1110       | llamadas | 14 llamadas     |
| -------------- | ------------ | ---------- | -------- | --------------- |
|                | k = 10       | ≈ 1,1×1010 | (nunca)  | ≈ 2046 llamadas |
| Ningún k sirve | para las dos | cosas      |          |                 |
Tolerar el 10% y detectar rápido el 50%. Las fallas consecutivas mezclan tasa de error
con volumen de tráfico: a 100 llamadas por segundo, con k = 3 y 10 s abierto, el breaker
abre cada ≈ 11 s y está abierto cerca del 47% del tiempo con un 10% de errores.
Cuentapropia(rachasenensayosindependientesdeBernoulli). Apunte: “Eldilema”.
SistemasDistribuidos–Clase11 43/61

| La solución: | tasa | de fallas | en una | ventana |
| ------------ | ---- | --------- | ------ | ------- |
Ventana deslizante con las últimas N llamadas: abre cuando la proporción de fallas llega al
| umbral, pero | solo tras   | un mínimo | de llamadas. |                 |
| ------------ | ----------- | --------- | ------------ | --------------- |
| ▶ Preset     | 05-cb-rate: | ventana   | 50, mínimo   | 20, umbral 50%. |
▶ Con p = 10%: abrir exige 25 o más fallas en 50. Probabilidad del orden de 10 −12: no
abre.
▶ Con p = 50%: la primera evaluación, con 20 llamadas, ya tiene cerca de un 59% de
probabilidad de ver 10 o más fallas. Abre en decenas de llamadas, sin importar el
volumen.
Compromiso
Ventana chica: rápida pero ruidosa. Ventana grande: estable pero lenta. Variante: abrir
| también por | llamadas | lentas. |     |     |
| ----------- | -------- | ------- | --- | --- |
Cuentapropia(distribuciónbinomial). Apunte: “Políticadetasadefallasenventana”.
SistemasDistribuidos–Clase11 44/61

| Experimento: | breaker | y fallback | contra | la dependencia | lenta |
| ------------ | ------- | ---------- | ------ | -------------- | ----- |
Escenario: latency 2000 · Preset: 00 → 01 → 06 · líneaamarilla: seaplicaunaconfiguración
Protecciones: 00ninguna · 01timeout · 06timeout+reintentos+breakerportasa+fallback
Lalatenciacaecon01ysiguebajacon06(p99delcliente: 0,67 s). Ladiferenciaeselresultado: con06el
breakerseabre,lasllamadasserechazansinesperaryhaycero errores;el98%delascontraseñassalesin
caracteresespeciales(degraded).
|     |     | SistemasDistribuidos–Clase11 |     |     | 45/61 |
| --- | --- | ---------------------------- | --- | --- | ----- |

Experimento: el mismo breaker, sin fallback
Escenario: latency 2000 · Preset: 06 → 07 · líneaamarilla: seaplicaunaconfiguración
Protecciones: 06timeout+reintentos+breaker+fallback · 07timeout+breaker,sinfallback
Con06lospedidossalendegradados(amarillo);con07elbreakerabreigual,perosinfallbacksalencomo
error(rojo): el98%. Elbreakerprotegealsistema;queelusuariorecibaalgoútilesunadecisióndeproducto,
elfallback.
SistemasDistribuidos–Clase11 46/61

Experimento: 10% de errores con fallas consecutivas
Escenario: errors 0.1 · Preset: 04-cb-consecutive-fragile · líneaamarilla: seaplicaunaconfiguración
Protecciones: timeout+breakerpor3fallasseguidas+fallback;sinreintentos
Un10%deerroresqueelsistematolerabaabrióelbreaker5 veces en 210 s(lacuentadaunarachade3
fallascada≈1110 llamadas,cercadeunminutoaesteritmo). Mientrasestáabierto,todaslasllamadasa
specialserechazanyvanalfallback: saliódegradadoel53%delospedidos.
SistemasDistribuidos–Clase11 47/61

Experimento: 10% de errores con tasa en ventana
Escenario: errors 0.1 · Preset: 05-cb-rate · líneaamarilla: seaplicaunaconfiguración
Protecciones: timeout+breakerportasa(ventana50,mínimo20,50%)+fallback;sinreintentos
Elbreakerdespecialno abre nunca: latasa(10%)estálejosdelumbral(50%). Aunasísaledegradadoel
38%delospedidos: enlademoelfallbackactúaantecualquierllamadaquefalla,nosoloconelbreaker
abierto. Sedegradansololospedidosconalgunaconexióncortada.
SistemasDistribuidos–Clase11 48/61

| Experimento: | 50% de | errores, | las tres | configuraciones |
| ------------ | ------ | -------- | -------- | --------------- |
Escenario: errors 0.5 · Preset: 04 → 04b → 05 · líneaamarilla: seaplicaunaconfiguración
Protecciones: timeout+breaker+fallback;elbreakerabrecon3fallasseguidas(04),con10(04b)oportasa(05)
04(k=3,desde13:42)abreenseguida. 04b(k=10,desde13:44)tardamás de 2 minutosenabrirpor
primeravezydejapasar1867llamadasfallidasen210s. 05(tasa,desde13:48)abreenseguida: 31llamadas
| fallidasen90s,igualque04. | El dilema, | medido. |     |     |
| ------------------------- | ---------- | ------- | --- | --- |
SistemasDistribuidos–Clase11 49/61

Experimento: caída intermitente
Escenario: intermittent fast · Preset: 06-complete · líneaamarilla: seaplicaunaconfiguración
Protecciones: timeout+reintentos+breakerportasa+fallback
Elbreakerdespecialseabreencadacaídaysecierracuandospecialvuelve. Hubo0 errores: durantelas
caídaslospedidossalierondegradados(49%). Elestadohalf-openduraloquetardaunaprueba
(milisegundos)ynollegaaverseconelmuestreodelpanel,cada5s.
SistemasDistribuidos–Clase11 50/61

| Degradación, |       | bulkheads |
| ------------ | ----- | --------- |
| y control    | de la | demanda   |
Sistemas Distribuidos

| Degradación |              | elegante |               | (graceful | degradation) |              |                   |
| ----------- | ------------ | -------- | ------------- | --------- | ------------ | ------------ | ----------------- |
|             |              |          | normal        |           |              | sinspecial:  | HTTP200,degradada |
|             | {”password”: |          | ”k7#Qa2!mXp”, |           |              | {”password”: | ”k7aQa2mXpR”,     |
|             | ”degraded”:  |          | []}           |           |              | ”degraded”:  | [”special”]}      |
▶ El fallback es una decisión de producto: caché, respuesta genérica, servicio secundario. Se
| decide | con los | interesados, |     | de antemano. |     |     |     |
| ------ | ------- | ------------ | --- | ------------ | --- | --- | --- |
▶ Tiene que ser explícito (el campo degraded) y más barato que el camino normal.
▶ A veces no es aceptable: una contraseña sin caracteres especiales puede violar la política de
contraseñas. Degradar pasa a ser responder algo incorrecto; corresponde fallar explícito.
| Nygard,ReleaseIt! | (2018),cap.5. |     | Apunte: | “Degradaciónelegante”. |     |     |     |
| ----------------- | ------------- | --- | ------- | ---------------------- | --- | --- | --- |
SistemasDistribuidos–Clase11 52/61

| Bulkheads: | compartimentos |     |     | estancos |
| ---------- | -------------- | --- | --- | -------- |
unpoolcompartido unpoolpordependencia
speciallentabloquealos8hilos specialagotasolosuparte;
lasdemássiguenatendiendo
▶
Como los mamparos de un barco: el agua inunda un compartimento y el barco no se hunde.
| “Salvar | parte del | barco”. |     |     |
| ------- | --------- | ------- | --- | --- |
▶ Formas: hilos por dependencia (y un pool reservado para administración), procesadores,
| instancias | dedicadas | a clientes | críticos, | zonas y regiones. |
| ---------- | --------- | ---------- | --------- | ----------------- |
▶
Costo: capacidad ociosa. Hay que buscar los límites naturales donde la partición conviene.
| Nygard,ReleaseIt! | (2018),cap.5,Bulkheads. |     | Apunte: | “Bulkheads”. |
| ----------------- | ----------------------- | --- | ------- | ------------ |
SistemasDistribuidos–Clase11 53/61

| Backpressure | y   | load shedding |               |
| ------------ | --- | ------------- | ------------- |
|              |     | loadshedding: | rechazar(503) |
colaacotada(llena)
productor consumidor
|     | backpressure: | frenaralproductor |     |
| --- | ------------- | ----------------- | --- |
▶ “Every performance problem starts with a queue backing up somewhere.” Sin límite, la cola
| crece y, | por Little, también | el tiempo | de respuesta. |
| -------- | ------------------- | --------- | ------------- |
▶ Backpressure desacelera al productor (TCP lo hace con su ventana). Sirve con productores
conocidos.
▶
Load shedding rechaza lo que no se va a completar a tiempo, lo antes posible: un rechazo
rápido es mejor que un timeout lento. Ejemplo: un servicio de video en vivo descarta los
| cuadros | que ya llegarían | tarde (ejemplo | de la cátedra). |
| ------- | ---------------- | -------------- | --------------- |
Nygard,ReleaseIt! (2018),caps.5y9. Apunte: “Backpressureyloadshedding”.
SistemasDistribuidos–Clase11 54/61

| Dos mecanismos |     | sanos | que interactúan |     | mal: Dressy |
| -------------- | --- | ----- | --------------- | --- | ----------- |
▶
Caso ilustrativo (ficticio) del Workbook: tres regiones detrás de un balanceador.
▶
Se activó load shedding por CPU. La región A empieza a rechazar 10%, 20%, 50%
| de los pedidos; | su  | CPU queda | constante. |     |     |
| --------------- | --- | --------- | ---------- | --- | --- |
▶ El balanceador estima el costo por pedido con la CPU: ve la región A como diez veces
más eficiente y le manda más tráfico (400 pedidos por segundo contra 40 en B y C).
Lección
“The load balancer didn’t know that the ‘eﬀicient’ requests were errors.” Cada mecanismo
| debe examinarse | junto | con los demás, | no  | por separado. |     |
| --------------- | ----- | -------------- | --- | ------------- | --- |
Betheaetal.,TheSiteReliabilityWorkbook(2018),cap.11.
SistemasDistribuidos–Clase11 55/61

Combinar mecanismos
Sistemas Distribuidos

| El  | orden | dentro | de una |     | llamada |     |     |
| --- | ----- | ------ | ------ | --- | ------- | --- | --- |
vuelveapasarporelbreaker
|     | bulkhead |     | breaker |     |     | timeout | reintento |
| --- | -------- | --- | ------- | --- | --- | ------- | --------- |
fallback
|     | ¿haylugar? |     | ¿estásana? |     | intentoacotado |     | backoff+jitter |
| --- | ---------- | --- | ---------- | --- | -------------- | --- | -------------- |
rechazo
▶
Cada mecanismo ahorra recursos al siguiente: bulkhead y breaker evitan intentos inútiles, el
timeout acota los que se hacen, los reintentos recuperan lo transitorio sin saltarse las
|     | protecciones, | el fallback | decide | qué | hacer | con el resto. |     |
| --- | ------------- | ----------- | ------ | --- | ----- | ------------- | --- |
▶ Más patrones no es mejor: “Count of patterns applied” nunca es una buena métrica. Cada
|                   | uno agrega | configuración | y       | modos                 | de falla | propios. |     |
| ----------------- | ---------- | ------------- | ------- | --------------------- | -------- | -------- | --- |
| Nygard,ReleaseIt! |            | (2018),cap.5. | Apunte: | “Combinarmecanismos”. |          |          |     |
SistemasDistribuidos–Clase11 57/61

| Caso: |             | la excepción |           | que | dejó     | en tierra  | a una aerolínea |          |
| ----- | ----------- | ------------ | --------- | --- | -------- | ---------- | --------------- | -------- |
|       |             | 23h          | 0:30      |     | 2:30     | 5:30este   |                 | ≈15h     |
|       | conmutación |              | sedapor   |     | caenlos  | horapico   |                 | termina  |
|       | delabase    |              | terminada |     | quioscos | decheck-in |                 | elatraso |
▶
Unaexcepciónalcerrarlaconsultaimpedíadevolverlaconexiónalpool. Trasunascuarentallamadas,el
poolseagotó.
▶ TodosloshilosdeCFquedaronesperandounaconexión,ylos40hilosdecadaservidordelosquioscos
decheck-in(lasterminalesdeautoserviciodelosaeropuertos),esperandoaCFporRMI,sintimeout.
▶
|       | Elmonitoreosolomirabaunapáginadeestado: |         |     |     |     | CFparecíasano. |     |     |
| ----- | --------------------------------------- | ------- | --- | --- | --- | -------------- | --- | --- |
| Dónde | se                                      | cortaba |     |     |     |                |     |     |
Timeout en el pool y en el cliente · breaker en el cliente · bulkheads en CF · fallback
| decidido |     | con el | negocio | · medir | desde | el cliente. |     |     |
| -------- | --- | ------ | ------- | ------- | ----- | ----------- | --- | --- |
Nygard,ReleaseIt! (2018),caps.2y3. Apunte: “Casodeestudio”(relatocompleto).
SistemasDistribuidos–Clase11 58/61

Cierre
Sistemas Distribuidos

| Modelo | del Curso: | Resiliencia | en D = (P,F,G,S,T) |
| ------ | ---------- | ----------- | ------------------ |
P (Problema): Impedir que la falla o la lentitud de una dependencia se propague al resto
del sistema.
F (Factores): Fallas parciales, dependencias lentas, pools finitos de hilos, fan-out,
|     | reintentos | que amplifican | la carga. |
| --- | ---------- | -------------- | --------- |
G (Garantías): Contención del daño, latencia acotada, recuperación automática,
|     | degradación | explícita. |     |
| --- | ----------- | ---------- | --- |
S (Solución): Timeouts calibrados con percentiles + reintentos con backoff y jitter +
breaker por dependencia (tasa de fallas) + fallback decidido con el negocio.
T (Trade-offs): Degradación de la respuesta, capacidad ociosa (bulkheads), complejidad de
configuración, falsos positivos del breaker, interacción entre mecanismos.
SistemasDistribuidos–Clase11 60/61

Cierre
Idea central de cátedra
La resiliencia distribuida es control de propagación de falla, no negación de la falla.
▶ Sin protección, una dependencia lenta saturó el pool y tumbó el servicio entero.
▶ Los timeouts cortaron la saturación; los reintentos recuperaron lo transitorio; el
breaker con fallback convirtió errores en respuestas degradadas.
▶ La política del breaker importa: con fallas consecutivas, ningún umbral sirve para el
10% y el 50% a la vez.
SistemasDistribuidos–Clase11 61/61