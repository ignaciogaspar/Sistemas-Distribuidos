| Sistemas       | Distribuidos |              |                    |               |
| -------------- | ------------ | ------------ | ------------------ | ------------- |
| Clase          | 06 — Storage | distribuido: |                    |               |
| almacenamiento |              | permanente,  | acceso concurrente | y rendimiento |
bajo fallas
| Departamento   | de          | Informática |               |     |
| -------------- | ----------- | ----------- | ------------- | --- |
| Ingeniería     | Informática | – Comisión  | única – Pilar |     |
| Equipo docente | – 2026      |             |               |     |

| Pregunta | central: | ¿qué resuelve | el storage | distribuido? |
| -------- | -------- | ------------- | ---------- | ------------ |
NÚCLEO
Enestaclase,storagedistribuidosignificaalmacenamiento Operaciónlógica
distribuido: nodos y protocolos que convierten operaciones “confirmarcompra”
enestadopermanente,recuperableyconsultableauncuando
| ocurrenfallas. |     |     |     | Capadistribuida |
| -------------- | --- | --- | --- | --------------- |
particiona,
| ▶   |     |     |     | replicayordena |
| --- | --- | --- | --- | -------------- |
Conservadecisionesdespuésdeunreinicio.
▶ Coordinalecturasyescriturassimultáneas.
Motoreslocales
▶
| Encuentradatosconlatenciapredecible.       |     |     |     | persisten,versionan |
| ------------------------------------------ | --- | --- | --- | ------------------- |
| ▶ Absorbecargasinacumulartrabajosinlímite. |     |     |     | eindexan            |
Estadopermanente
recuperableyconsultable
SistemasDistribuidos–Clase06 2/73

| Dónde | se ubica | el storage | dentro del | sistema | distribuido |
| ----- | -------- | ---------- | ---------- | ------- | ----------- |
NÚCLEO
nodosdealmacenamiento
NodoA
motor+medio
persistente
|     |          |             | Capadistribuida   |     | NodoB       |
| --- | -------- | ----------- | ----------------- | --- | ----------- |
|     |          | Serviciosde | enrutamiento,     |     | motor+medio |
|     | Clientes | aplicación  |                   |     |             |
|     |          |             | réplicayautoridad |     | persistente |
NodoC
motor+medio
persistente
| Capa distribuida |     |     | Motor | local |     |
| ---------------- | --- | --- | ----- | ----- | --- |
Convierteesasdecisionesenbytespersistentes,versiones
Ubicayreplicalosdatos;además,decidequénodotiene
visibles,índicesyprocedimientosderecuperación.
autoridadparaconfirmarcadaoperación.
SistemasDistribuidos–Clase06 3/73

Qué llamaremos “motor de almacenamiento”
NÚCLEO
interiordeunnododealmacenamiento
Planificador
yejecutor
Árboleseíndices
archivosordenados
Lecturao Gestor Memoria
escritura transaccional intermedia
Páginasyarchivos
Registrode enmediopersistente
recuperación
Definición
El motor de almacenamiento es el software que traduce operaciones lógicas en bytes,
versiones, estructuras de acceso y reglas de recuperación sobre un medio persistente.
SistemasDistribuidos–Clase06 4/73

Caso conductor: CampusShop
NÚCLEO
Unaestudiantecompralaúltimaunidaddeunproductodel Operación9F2A
campus. LaAPI(interfazdeprogramacióndeaplicaciones) confirmarcompra
recibelaoperaciónconidentificadorestable9F2Ayresponde
“compra confirmada”. Un segundo después, el nodo pierde pedido+inventario
energía. +índice
▶ respuestaalcliente
Lacompradebeexistirexactamenteunavezalreiniciar.
“confirmada”
▶
Launidaddescontadanodebereaparecerenelinventario.
▶ Unreportelargodebeconservarunavistacoherente cortedeenergía
mientrasentrannuevascompras. yreinicio
▶
Laconsultadelasúltimas20comprasdebeevitarrevisar
todalatabla. Predicción inicial
▶
Unacargasostenidanodebecrearunacolainternasin
¿Quéevidenciadebesobrevivirparaquelascinco
límite.
afirmacionesanterioressigansiendoverdaderas?
SistemasDistribuidos–Clase06 5/73

| Camino | crítico | y patrón de | entrada/salida |     |     |     |
| ------ | ------- | ----------- | -------------- | --- | --- | --- |
NÚCLEO
| Camino | crítico de una | escritura | E/S | aleatoria y secuencial |     |     |
| ------ | -------------- | --------- | --- | ---------------------- | --- | --- |
Secuenciadepasosquedebecompletarseantesdeenviar Aleatorianosignificaimprevisible: significaaccedera
laconfirmaciónalcliente. Elpasomáslentodeesa ubicacionesfísicasnocontiguas,porejemploP17,P9001e
secuenciacondicionalalatenciaobservada. I5. Secuencialsignificaagregarbytescontiguosalfinalde
unarchivoolog.
|         | WAL     | actualizar |     |                   |            |         |
| ------- | ------- | ---------- | --- | ----------------- | ---------- | ------- |
| request |         | confirmar  |     |                   |            |         |
|         | durable | memoria    |     |                   |            |         |
|         |         |            |     | dispersa página17 | página9001 | índice5 |
|         |         |            |     | byte100           | 101        | 102     |
secuencial
Ladiferenciadecostodependedeldispositivo. Aquíimportalaformadelacceso: variasubicacionesdispersasfrenteaunappendcontiguoquepuede
agruparse.
SistemasDistribuidos–Clase06 6/73

| Árbol | B y LSM-tree: | dos formas | de organizar | datos |
| ----- | ------------- | ---------- | ------------ | ----- |
NÚCLEO
Árbol B (B-tree). Índice ordenado y balanceado LSM-tree — Log-Structured Merge-tree. En
cuyosnodossuelencorresponderapáginas. Permite español: árbol de fusión estructurado como log.
localizar una clave o recorrer un rango con pocas Acumulaescriturasenmemoria,lasvuelcaaarchivos
lecturasdepáginas. ordenadoseinmutablesyluegofusionaesosarchivos.
| Costo característico |     |     | Qué cambia |     |
| -------------------- | --- | --- | ---------- | --- |
Unaescriturapuedemodificarlapáginadedatosyunoo Evitareescribirinmediatamentemuchasubicaciones
másíndices;elcostoapareceenelcaminocríticooen dispersasantesdelACK.Elcostoreaparecedespuésen
| mantenimientoposterior. |     |     | flush,compactación,lecturasyespacio. |     |
| ----------------------- | --- | --- | ------------------------------------ | --- |
Idea central
Ningunaestructuraeliminatrabajo. Cadaunadecidecuándo,dóndeyconquépatróndeE/Ssepaga.
SistemasDistribuidos–Clase06 7/73

| Vocabulario | inicial: | compactación, | trabajo | pendiente | y pausa |
| ----------- | -------- | ------------- | ------- | --------- | ------- |
de escritura
NÚCLEO
| Término |     | Significadoinicial |     |     |     |
| ------- | --- | ------------------ | --- | --- | --- |
Compactación Procesodefondoquefusionaarchivosordenados,reducesolapamientoyelimina
versionesobsoletascuandoyaessegurohacerlo.
Trabajopendiente(backlog) Cantidad de bytes o archivos que esperan ser compactados. Si ingresa trabajo
másrápidodeloqueseprocesa,elbacklogcrece.
Regulacióndeescrituras Mecanismodeadmisiónqueretrasa,bloqueaolimitanuevassolicitudesparaque
lacolainternanocrezcasinlímite.
Pausa de escritura (write Casofuertederegulación: elmotorhaceesperaralosescritoresodejadeadmitir
| stall) |     | temporalmentenuevasescriturashastarecuperarcapacidad. |     |     |     |
| ------ | --- | ----------------------------------------------------- | --- | --- | --- |
Seregulalasolicituddeescritura: elhilopuedeesperar,recibirunretardo,consumirmenospermisosdeadmisióno,
segúnlaimplementación,obtenerunarespuestadesobrecarga.
SistemasDistribuidos–Clase06 8/73

Cinco responsabilidades, no cinco términos aislados
NÚCLEO
Responsabilidad Preguntaconcreta Mecanismoqueestudiaremos
Registrardecisiones ¿Cómo sobrevive una compra ya WALyrecuperación
confirmada?
Controlarconcurrencia ¿Qué puede observar un reporte MVCCeinstantáneaslógicas
mientrascambianlosdatos?
Elegirunarutafísica ¿Cómo encontrar 20 filas entre un ÁrbolB,planificadoryEXPLAIN
millón?
Optimizarescrituras ¿Cómo agrupar escrituras y evitar ÁrbolBoLSM-tree
accesosdispersosinmediatos?
Protegercapacidad ¿Quéocurresilacompactaciónno Trabajo pendiente, regulación y pausa de
alcanzaelritmodeingreso? escritura
Ningún mecanismo constituye por sí solo el almacenamiento completo. Cada uno cubre una responsabilidad, ofrece
unagarantíaparcialydesplazauncosto.
SistemasDistribuidos–Clase06 9/73

| Del | dato lógico | a los objetos | físicos |     |
| --- | ----------- | ------------- | ------- | --- |
NÚCLEO
|     | Solicituddecompra | Filanuevaen | PáginaP17    | Archivodedatos |
| --- | ----------------- | ----------- | ------------ | -------------- |
|     | identificador9F2A | purchases   | bloquefísico | enelmedio      |
|     |                   | EntradaWAL  | Entradade    |                |
paraID9F2A índiceI5
rutahaciaP17
| Qué significa | 9F2A |     | Qué no significa |     |
| ------------- | ---- | --- | ---------------- | --- |
Es un identificador estable de la No es el contenido de la fila ni una posición
operación lógica. Si el cliente reintenta, física. P17 e I5 son etiquetas didácticas
vuelve a enviar 9F2A para que el servidor para una página de datos y una entrada de
| reconozca | la misma compra. |     | índice. |     |
| --------- | ---------------- | --- | ------- | --- |
SistemasDistribuidos–Clase06 10/73

Vocabulario previo I: confirmar y convertir algo en “durable”
NÚCLEO
Término Significadoenestaclase
COMMIT Decisióndelmotordeaceptardefinitivamentelosefectosdeunatransacción. El
COMMITpuedeestardecididoantesdequetodaslaspáginashayansidoescritas.
ACK Acknowledgement,oacusederecibo. Eslarespuestaenviadaalclientecuandose
cumpliólagarantíaqueelsistemapromete.
Xdurable LaevidencianecesariaparapoderreconstruirXdespuésdeocurridalafallaqueel
modelodeclara. “Durable”siempredebeleerserespectodeunafallaconcreta.
Logdurable PorcióndelWALcuyosbytesnecesariosparalarecuperaciónyasobreviviríanala
falladeclarada.
Flushdurable Operación de volcado ya concluida que llevó los bytes requeridos del WAL más
alládelafronteradurable.
Regla de lectura
Cuandoaparezca“Xdurable”,pregunte: ¿quéfallaseconsiderayquéevidenciadeXseguirádisponibledespuésdeesafalla?
SistemasDistribuidos–Clase06 11/73

| Vocabulario | previo | II: escribir | y vaciar | buffers |
| ----------- | ------ | ------------ | -------- | ------- |
NÚCLEO
| Término | Significadoenestaclase |     |     |     |
| ------- | ---------------------- | --- | --- | --- |
write/append Copiaroagregarbytesaunacapadelsistema;lallamadapuedeterminarmientras
losdatossiguenenmemoriaocaché.
Flushovolcado Mover datos acumulados en un buffer hacia una capa inferior del sistema de
|     | almacenamiento. |     | Unflushcomúnnoimplicanecesariamentedurabilidad. |     |
| --- | --------------- | --- | ----------------------------------------------- | --- |
fsync Solicitar al sistema operativo que sincronice los datos relevantes con el medio
persistente,deacuerdoconelhardwareylaconfiguración.
Entrada/salida(E/S) Transferenciadedatosentrememoria,sistemaoperativoydispositivopersistente.
Páginasucia(dirtypage) Páginamodificadaenmemoriaquetodavíanofueescritaenelarchivodedatos.
| Pregunta de ingeniería |     |     |     |     |
| ---------------------- | --- | --- | --- | --- |
Cuandoalguiendiga“yaestáescrito”,pregunte: ¿enquécapaterminólaoperaciónyquéfallatodavíapodríaborraresos
bytes?
SistemasDistribuidos–Clase06 12/73

| Vocabulario | previo III: | fallar y recuperar |
| ----------- | ----------- | ------------------ |
NÚCLEO
| Término | Significadoenestaclase |     |
| ------- | ---------------------- | --- |
Caídaabrupta(crash) Pérdida repentina de un proceso, del sistema operativo o de energía. El modelo
debeindicarquécapassepierden.
Recuperación(recovery) Procedimientodereinicioqueusalaevidenciapersistidaparareconstruirunestado
lógicoválido.
REDO Reaplicación de un cambio confirmado que aún no estaba materializado en su
páginadedatosoíndice.
Punto de control (check- Punto durable que acota desde qué posición del WAL debe comenzar la
| point) | recuperación. |     |
| ------ | ------------- | --- |
LSN Log Sequence Number: número de secuencia que identifica una posición en el
WAL;noesunrelojfísico.
Estostérminosdescribencómoelmotortransformabytespersistidosenunestadológicoválidodespuésdeunafalla.
SistemasDistribuidos–Clase06 13/73

| Vocabulario | previo IV: | concurrencia | y consistencia |     |
| ----------- | ---------- | ------------ | -------------- | --- |
NÚCLEO
| Término | Significado | en esta clase |     |     |
| ------- | ----------- | ------------- | --- | --- |
Versión Representación de un dato creada por una transacción y visible
|     | para | determinados lectores. |     |     |
| --- | ---- | ---------------------- | --- | --- |
Snapshot Instantánea lógica que determina qué transacciones y versiones
|     | puede | observar una lectura. |     |     |
| --- | ----- | --------------------- | --- | --- |
Vista coherente Resultado cuyas filas obedecen la misma regla de visibilidad, sin
|     | mezclar | momentos arbitrarios. |     |     |
| --- | ------- | --------------------- | --- | --- |
Serializabilidad Garantía de que el resultado concurrente equivale a alguna
|     | ejecución | de las transacciones | una detrás | de otra. |
| --- | --------- | -------------------- | ---------- | -------- |
SistemasDistribuidos–Clase06 14/73

| Vocabulario | previo V: | consultas | y definición | del esquema |
| ----------- | --------- | --------- | ------------ | ----------- |
NÚCLEO
| Término | Significado | en esta | clase |     |
| ------- | ----------- | ------- | ----- | --- |
SQL Structured Query Language: lenguaje estructurado de consultas
|     | para | leer y modificar | datos. |     |
| --- | ---- | ---------------- | ------ | --- |
DDL Data Definition Language: parte de SQL que crea o modifica
|     | estructuras, | por ejemplo | CREATE INDEX. |     |
| --- | ------------ | ----------- | ------------- | --- |
Planificador Componente que compara rutas físicas posibles y elige un plan
estimado.
EXPLAIN Comando que muestra el plan elegido por el planificador.
EXPLAIN ANALYZE Ejecuta la consulta y agrega mediciones reales al plan.
SistemasDistribuidos–Clase06 15/73

| Repaso   | ágil:          | cinco | puentes |
| -------- | -------------- | ----- | ------- |
| hacia el | almacenamiento |       |         |
Sistemas Distribuidos

Una frase por cada clase previa
NÚCLEO
Clase1—Incertidumbre: sinosabemosquéocurrió,elalmacenamientoconservaevidenciapararesponderde
maneraestable.
Clase2—RPC(llamadaaprocedimientoremoto)yreintento: elreintentoessegurosolosiidentidad,
efectoyresultadosobrevivenalafalla.
Clase3—Replicación: variascopiasnoindicanporsísolascuálessondurables,actualesniautorizadas.
Clase4—Consenso: elprotocolodecidequédeberecordarse;cadanododebepersistirlorealmente.
Clase5—Transacciones: ACID(atomicidad,consistencia,aislamientoydurabilidad)defineelcontrato;WAL
yMVCCimplementanpartedeesecontrato.
Transición. Hoypreguntamosquébytes,esperasyestructurasvuelvenverdaderaspalabrascomo“COMMIT”,“durable”y
“aislado”.
SistemasDistribuidos–Clase06 17/73

Diagnóstico inicial: cinco situaciones
ACTIVIDAD
Diagnóstico
Alfinalizarestaclasedebemossercapacesdedescribirelcomportamientoqueseesperaylafallaqueestáconsiderada.
1. Laaplicaciónmostró“compraconfirmada”yenseguidasecortólaenergía. ¿Puededesapareceralreiniciar?
2. Unreportecomenzóalas12:00ydurantesuejecuciónentraronnuevascompras. ¿Debencambiarsustotalesamitad
delcálculo?
3. Paramostrar20comprasentreunmillón,¿esrazonablerevisarsiempretodaslasfilas?
4. Ingresan90megabytesporsegundo(MB/s)ysecompactan60MB/s. ¿Puedeeltrabajopendientepermanecer
acotado?
5. TresnodosconservaneldatosoloenmemoriaRAM(memoriadeaccesoaleatorio). ¿Sobreviveuncortedeenergía
común?
SistemasDistribuidos–Clase06 18/73

| Mapa conceptual: | dónde | aparece | cada | mecanismo |
| ---------------- | ----- | ------- | ---- | --------- |
NÚCLEO
Protección
| Decisióndurable | Historiaaccedida | Rutadeacceso |     | Caminode |
| --------------- | ---------------- | ------------ | --- | -------- |
decapacidad
| WAL+ | concurrentemente | índice+ |     | escritura |
| ---- | ---------------- | ------- | --- | --------- |
compactación
| recuperación | MVCC | EXPLAIN |     | árbolB/LSM |
| ------------ | ---- | ------- | --- | ---------- |
+pausa
El hilo conductor será siempre el mismo: problema, mecanismo, garantía, costo y
evidencia.
SistemasDistribuidos–Clase06 19/73

| Evidencias | de aprendizaje | de  | esta clase |
| ---------- | -------------- | --- | ---------- |
NÚCLEO
| Durante | la clase |     | Al cerrar |
| ------- | -------- | --- | --------- |
| ▶       |          |     | ▶         |
completar un timeline WAL y justificar diagnosticar trabajo pendiente y una
| cuándo | enviar el ACK; |     | pausa de escritura; |
| ------ | -------------- | --- | ------------------- |
| ▶      |                |     | ▶                   |
completar un timeline MVCC y elegir entre árbol B, LSM-tree o una
justificar qué versión es visible; combinación según la carga;
| ▶   |     |     | ▶   |
| --- | --- | --- | --- |
leer un plan EXPLAIN antes y después defender la decisión con falla, garantía,
| de crear | un índice. |     | costo y métrica. |
| -------- | ---------- | --- | ---------------- |
SistemasDistribuidos–Clase06 20/73

| WAL                   | y recuperación: |              |
| --------------------- | --------------- | ------------ |
| hacer                 | durable         | una decisión |
| Sistemas Distribuidos |                 |              |

CampusShop: qué cambia físicamente al confirmar una compra
NÚCLEO
La operación con ID 9F2A inserta una fila en purchases, Solicitud
descuenta una unidad en inventory y actualiza un índice IDdeoperación9F2A
porestudianteyfecha.
purchases: nuevafila
▶
9F2Aidentificalamismacompralógicasielcliente
reintenta.
inventory: stock−1
▶
Lasfilasvivenlógicamenteentablas.
▶
Elmotormodificapáginasenmemoria.
▶ índice: nuevaentrada
ElWALconservaevidenciaparalarecuperación.
WAL:cambios+COMMIT
SistemasDistribuidos–Clase06 22/73

La frontera durable
NÚCLEO
Definición
Aplicación
| Lafrontera | durableeselpuntoapartirdelcual, |           |            | bajo |
| ---------- | ------------------------------- | --------- | ---------- | ---- |
| el modelo  | de falla declarado,             | permanece | disponible | la   |
Bufferdelproceso evidencia necesaria para reconstruir la operación.
| No existe | una frontera | universal |     |     |
| --------- | ------------ | --------- | --- | --- |
Cachédelsistemaoperativo
| Depende  | de la falla considerada | y de               | la semántica |     |
| -------- | ----------------------- | ------------------ | ------------ | --- |
| efectiva | de fsync, del           | sistema operativo, | del          |     |
Cachédeldispositivo
| controlador, | del dispositivo | y de su configuración. |     |     |
| ------------ | --------------- | ---------------------- | --- | --- |
crucedurable
Mediopersistente
| SistemasDistribuidos–Clase06 |     |     |     | 23/73 |
| ---------------------------- | --- | --- | --- | ----- |

| Qué | es WAL | y por qué | se dibuja | separado | de las | páginas |
| --- | ------ | --------- | --------- | -------- | ------ | ------- |
NÚCLEO
| Definición | intuitiva |     |     |     |     |     |
| ---------- | --------- | --- | --- | --- | --- | --- |
Gestortransaccional
WALeselregistrosecuencialdelmotor.Primeroconservaevidenciapara
reconstruirunadecisión;después,laspáginaspuedenpersistirseenun
ordenfísicomásconveniente.
|            |         |     |     | 1. WALenbuffer |     | 2. páginas |
| ---------- | ------- | --- | --- | -------------- | --- | ---------- |
| Definición | técnica |     |     |                |     |            |
|            |         |     |     | aúnvolátil     |     | enmemoria  |
Laevidenciaderecuperacióndebeserdurableantesquelapágina 4. páginas
modificada,yelregistroCOMMITdebeserdurableantesdelACK. 3. WALdurable
endisco
hastaCOMMIT
Porquésedibujaaparte.WALypáginassevuelvendurablesen másadelante
momentosdistintos,aunqueambospertenezcanalmismomotor.
Predicción
5.recuperación
SielACKyasalióylapáginapersistenteconservalaversiónanterior,¿qué yREDO
evidenciapermitereconstruirelcambio?
|     |     |     |     | Dos persistencias | distintas: el | WAL conserva evidencia; |
| --- | --- | --- | --- | ----------------- | ------------- | ----------------------- |
laspáginasmaterializanelestado.
SistemasDistribuidos–Clase06 24/73

| WAL: dos | reglas, dos | fallas que | evitan |
| -------- | ----------- | ---------- | ------ |
NÚCLEO
Regla 1 — log antes que página Regla 2 — COMMIT antes que ACK
registroWAL→flushdurable→páginadedatos.
| Orden: |     |     | Orden: registroCOMMIT→flushdurable→ACKal |
| ------ | --- | --- | ---------------------------------------- |
cliente.
▶
| Propósito: | quetodapáginapersistidatenga |     |     |
| ---------- | ---------------------------- | --- | --- |
▶ Propósito: quecadaconfirmaciónenviadapueda
evidenciasuficienteparaserinterpretadaorehecha.
| ▶             |                           |     | reconstruirsetrasuncrash. |
| ------------- | ------------------------- | --- | ------------------------- |
| Fallaevitada: | unapáginanuevasinregistro |     |                           |
▶
recuperabledelcambioquelaprodujo. Fallaevitada: mostrar“confirmada”yperderluego
ladecisióndeCOMMIT.
Garantía buscada: después de la recuperación aparecen todas las transacciones confirmadas y ninguna transacción
incompletasepresentacomoconfirmada.
SistemasDistribuidos–Clase06 25/73

| Esquema | de funcionamiento | del WAL |     |
| ------- | ----------------- | ------- | --- |
NÚCLEO
unúnicomotordealmacenamiento
Gestor
ArchivoWAL
transaccional
Serviciode
| Cliente | aplicación |     | Recuperación |
| ------- | ---------- | --- | ------------ |
Páginasen Páginaseíndices
persistentes
memoria
| ▶   |     | ▶   |     |
| --- | --- | --- | --- |
1. Registrar: el gestor agrega evidencia al 3. Materializar: las páginas se escriben
| WAL. |     | después en | el dispositivo persistente. |
| ---- | --- | ---------- | --------------------------- |
▶ 2. Modificar: las páginas cambian ▶ 4. Recuperar: tras un crash, el WAL
| primero | en memoria. | permite completarlas. |     |
| ------- | ----------- | --------------------- | --- |
SistemasDistribuidos–Clase06 26/73

Timeline del WAL
ACTIVIDAD
Vamosaubicar: solicitud,cambios,páginasucia,
registroCOMMIT,flushdurable,ACK,crashyREDO.
Cliente
Servicio/API
Motor/memoria
ArchivoWAL
Páginas/índices
tiempo
Predicción
¿Después de qué evento sería legítimo mostrar “compra confirmada” sin exigir que las
páginas finales ya estén actualizadas en el dispositivo?
SistemasDistribuidos–Clase06 27/73

6.flushdurable 8.crash
7.ACK
confirmada
2.preparar
cambios
3.WAL 5.registro
UPDATE COMMIT
4.páginas
9.REDO
sucias
Construcción progresiva del timeline WAL
NÚCLEO
Cliente 1.solicitud9F2A
Servicio
Motor/memoria
WAL
Páginas/índices
tiempo
Paso 1
El cliente envía una intención con identidad estable.
SistemasDistribuidos–Clase06 28/73

6.flushdurable 8.crash
7.ACK
confirmada
3.WAL 5.registro
UPDATE COMMIT
4.páginas
9.REDO
sucias
Construcción progresiva del timeline WAL
NÚCLEO
Cliente 1.solicitud9F2A
Servicio
Motor/memoria 2.preparar
cambios
WAL
Páginas/índices
tiempo
Paso 2
El motor prepara cambios lógicos en memoria.
SistemasDistribuidos–Clase06 28/73

6.flushdurable 8.crash
7.ACK
confirmada
5.registro
COMMIT
4.páginas
9.REDO
sucias
Construcción progresiva del timeline WAL
NÚCLEO
Cliente 1.solicitud9F2A
Servicio
Motor/memoria 2.preparar
cambios
3.WAL
WAL
UPDATE
Páginas/índices
tiempo
Paso 3
El WAL describe cómo rehacerlos; todavía puede estar en buffers.
SistemasDistribuidos–Clase06 28/73

6.flushdurable 8.crash
7.ACK
confirmada
5.registro
COMMIT
9.REDO
Construcción progresiva del timeline WAL
NÚCLEO
Cliente 1.solicitud9F2A
Servicio
Motor/memoria 2.preparar
cambios
3.WAL
WAL
UPDATE
Páginas/índices 4.páginas
sucias
tiempo
Paso 4
Las páginas se modifican en memoria y quedan sucias.
SistemasDistribuidos–Clase06 28/73

6.flushdurable 8.crash
7.ACK
confirmada
9.REDO
| Construcción | progresiva | del | timeline WAL |
| ------------ | ---------- | --- | ------------ |
NÚCLEO
Cliente 1.solicitud9F2A
Servicio
2.preparar
Motor/memoria
cambios
|                 |     | 3.WAL     | 5.registro |
| --------------- | --- | --------- | ---------- |
|                 | WAL | UPDATE    | COMMIT     |
| Páginas/índices |     | 4.páginas |            |
sucias
tiempo
Paso 5
El registro COMMIT expresa la decisión, pero aún no es durable.
SistemasDistribuidos–Clase06 28/73

8.crash
7.ACK
confirmada
9.REDO
| Construcción | progresiva | del | timeline WAL |
| ------------ | ---------- | --- | ------------ |
NÚCLEO
6.flushdurable
Cliente 1.solicitud9F2A
Servicio
2.preparar
Motor/memoria
cambios
|                 |     | 3.WAL     | 5.registro |
| --------------- | --- | --------- | ---------- |
|                 | WAL | UPDATE    | COMMIT     |
| Páginas/índices |     | 4.páginas |            |
sucias
tiempo
Paso 6
El flush lleva el WAL relevante más allá de la frontera durable declarada.
SistemasDistribuidos–Clase06 28/73

8.crash
9.REDO
| Construcción | progresiva | del | timeline WAL |
| ------------ | ---------- | --- | ------------ |
NÚCLEO
6.flushdurable
Cliente 1.solicitud9F2A 7.ACK
confirmada
Servicio
2.preparar
Motor/memoria
cambios
|                 |     | 3.WAL     | 5.registro |
| --------------- | --- | --------- | ---------- |
|                 | WAL | UPDATE    | COMMIT     |
| Páginas/índices |     | 4.páginas |            |
sucias
tiempo
Paso 7
Ahora el servicio puede confirmar sin esperar que todas las páginas estén escritas.
SistemasDistribuidos–Clase06 28/73

9.REDO
| Construcción | progresiva |     | del timeline | WAL |
| ------------ | ---------- | --- | ------------ | --- |
NÚCLEO
6.flushdurable 8.crash
|     | Cliente 1.solicitud9F2A |     |     | 7.ACK |
| --- | ----------------------- | --- | --- | ----- |
confirmada
Servicio
2.preparar
Motor/memoria
cambios
|                 |     |     | 3.WAL 5.registro |     |
| --------------- | --- | --- | ---------------- | --- |
|                 | WAL |     | UPDATE COMMIT    |     |
| Páginas/índices |     |     | 4.páginas        |     |
sucias
tiempo
Paso 8
| El crash borra | memoria | y buffers | volátiles. |     |
| -------------- | ------- | --------- | ---------- | --- |
SistemasDistribuidos–Clase06 28/73

| Construcción | progresiva | del | timeline WAL |     |
| ------------ | ---------- | --- | ------------ | --- |
NÚCLEO
|     |     |     | 6.flushdurable 8.crash |     |
| --- | --- | --- | ---------------------- | --- |
Cliente 1.solicitud9F2A 7.ACK
confirmada
Servicio
2.preparar
Motor/memoria
cambios
|                 |     | 3.WAL     | 5.registro |        |
| --------------- | --- | --------- | ---------- | ------ |
|                 | WAL | UPDATE    | COMMIT     |        |
| Páginas/índices |     | 4.páginas |            | 9.REDO |
sucias
tiempo
Paso 9
Recuperación lee el WAL y REDO materializa en las páginas los cambios confirmados que
faltan.
SistemasDistribuidos–Clase06 28/73

| Criterio | de correctitud | de la recuperación |     |
| -------- | -------------- | ------------------ | --- |
NÚCLEO
Puntodelcrash Evidenciadurabledisponible Resultadocorrectodespuésderecuperar
AntesdelCOMMITdurable Puede haber registros de cambio, La compra no aparece confirmada; el trabajo
pero no una decisión durable de incompletosedescartaoreviertesegúneldiseño.
COMMIT
Después del COMMIT RegistrosdecambioyCOMMITen REDOreconstruyepedido,stockeíndice.
| durable, | antes de escribir | elWALdurable |     |
| -------- | ----------------- | ------------ | --- |
páginas
DespuésdelCOMMITydelas WALypáginasyaestánmaterializa- Larecuperaciónverificaelestado;repetirREDO
| páginas |     | dos | nodebeproducirunsegundoefectológico. |
| ------- | --- | --- | ------------------------------------ |
Lostresresultadossoncorrectosporqueaplicanelmismocriterio: conservartodaslastransaccionesconfirmadasyno
convertirunatransacciónincompletaenconfirmada.
SistemasDistribuidos–Clase06 29/73

| Group commit: | compartir | el costo | sin debilitar | la garantía |
| ------------- | --------- | -------- | ------------- | ----------- |
NÚCLEO
T1 compra
|     |     |     | unflushdurable | tresACK |
| --- | --- | --- | -------------- | ------- |
T2 pago BufferWAL
|     |     |     | compartido | despuésdelflush |
| --- | --- | --- | ---------- | --------------- |
tresCOMMIT
T3 reserva
Ganancia: un acceso físico amortiza varios COMMIT. Costo: cada transacción puede
esperar brevemente.
Garantía: cada ACK sigue saliendo solamente después de que su COMMIT quedó durable.
SistemasDistribuidos–Clase06 30/73

| Checkpoint: | un punto | durable | desde dónde | comenzar | una |
| ----------- | -------- | ------- | ----------- | -------- | --- |
recuperación
NÚCLEO
|     |                       | checkpoint             | crash |              |     |
| --- | --------------------- | ---------------------- | ----- | ------------ | --- |
|     | páginasmaterializadas | WALquedebereproducirse |       | LSN/progreso |     |
Qué es
Registra suficiente progreso durable para que la recuperación no comience desde el origen del WAL.
▶
Más frecuente: menos WAL por reproducir, pero más entrada/salida y posibles picos.
▶ Más espaciado: operación normal más suave, pero recuperación y retención mayores.
|     |     | SistemasDistribuidos–Clase06 |     |     | 31/73 |
| --- | --- | ---------------------------- | --- | --- | ----- |

| Caso real: | PostgreSQL | desacopla | el COMMIT | de las páginas |
| ---------- | ---------- | --------- | --------- | -------------- |
NÚCLEO
▶
ElWALrelevantedebellegaralalmacenamientoestableantes
queloscambiosenlosarchivosdedatos.
| ▶   |     |     |     | WALsecuencial |
| --- | --- | --- | --- | ------------- |
Trasuncrash,larecuperaciónreproduceloqueindicaelWAL
pararestaurarunestadoválido.
▶
Laspáginaspersistentespuedenconservartemporalmenteuna
COMMITdurable
versiónanterioralúltimoCOMMIT.
▶ Elhardwareylaconfiguraciónformanpartedelagarantía
| efectiva. |     |     |     | páginapersistente |
| --------- | --- | --- | --- | ----------------- |
conversiónanterior
| En esta lámina, | una página | aún no actualizada | es una |     |
| --------------- | ---------- | ------------------ | ------ | --- |
recuperación/REDO
páginacuyoarchivopersistentetodavíanoincorporóuncambio
actualizalapágina
| confirmado. | La evidencia durable | está en el WAL | y REDO |     |
| ----------- | -------------------- | -------------- | ------ | --- |
permitematerializarelcambioalreiniciar.
Fuente/referencia:DocumentaciónoficialdePostgreSQL,secciónWrite-AheadLogging.
SistemasDistribuidos–Clase06 32/73

| MVCC:    | versiones    | para |        |
| -------- | ------------ | ---- | ------ |
| lecturas | concurrentes |      | coher- |
entes
Sistemas Distribuidos

| Pensemos         | en ejemplo | real:    | se crea y   | consulta un reporte | y   |
| ---------------- | ---------- | -------- | ----------- | ------------------- | --- |
| concurrentemente |            | se hacen | 900 compras | nuevas              |     |
NÚCLEO
Unreportedelcomedorcomienzaalas12:00ytarda40segundos. Duranteeseintervalo
entran 900 compras. ¿Debe mezclar filas de momentos distintos? ¿Debe bloquear a
| todos los | escritores? |     |     |     |     |
| --------- | ----------- | --- | --- | --- | --- |
¿Bloquearescritores,
|     | Reportelargo |     | 900escrituras  | mezclarmomentos |     |
| --- | ------------ | --- | -------------- | --------------- | --- |
|     | inicia12:00  |     | 12:00–12:00:40 | oconservaruna   |     |
vistacoherente?
|     |     |     | SistemasDistribuidos–Clase06 |     | 34/73 |
| --- | --- | --- | ---------------------------- | --- | ----- |

Vocabulario de MVCC
NÚCLEO
Término Definicióndetrabajo
Versión Representación de una fila creada por una transacción y válida para determinados
lectores.
Transacciónlectora Operaciónqueconsultadatosbajounaregladevisibilidad.
Transacciónescritora Operaciónquecreaunaversiónnuevaynoborradeinmediatolaanterior.
Snapshot Instantánealógicaquedeterminaquétransaccionesconfirmadasyquéversionesson
visibles.
Visibilidad Decisióndesiunaversiónpertenecealahistoriaqueesesnapshotpuedeobservar.
Vistacoherente Resultadocuyasfilasseseleccionanconlamismaregladevisibilidad.
Serializabilidad Garantía de que la ejecución concurrente equivale a alguna ejecución serial de las
transacciones.
Límite de la metáfora
“InstantáneaoFotografía”nosignificacopiartodalabase. Tampocoimplicaserializabilidadporsísola: describeunaregla
paraelegirversiones.
SistemasDistribuidos–Clase06 35/73

| MVCC en | una imagen: | dos lectores, | dos vistas | coherentes |
| ------- | ----------- | ------------- | ---------- | ---------- |
NÚCLEO
Reporteiniciadoantes
sigueleyendov1
|     | Antes | Escritor |     |     |
| --- | ----- | -------- | --- | --- |
creav2=135y
v1=120ya
|     | estáconfirmada | ejecutaCOMMIT |     |     |
| --- | -------------- | ------------- | --- | --- |
Lecturainiciadadespués
puedeleerv2
Idea central
MVCC conserva versiones y aplica reglas de visibilidad. Dos lectores pueden observar
versiones diferentes sin que el reporte antiguo cambie a mitad de ejecución.
SistemasDistribuidos–Clase06 36/73

Timeline MVCC: cuatro carriles y términos conocidos
ACTIVIDAD
Ubiquemos: v1,iniciodelsnapshot,lectura,creación
dev2,COMMIT,segundalectura,finyreclamación.
LectorTR
EscritorTW
Versiones
Lecturavisible
tiempo
Consideración
Después del COMMIT de una v2, ¿qué lee una segunda consulta dentro del mismo reporte
y qué lee un reporte nuevo?
SistemasDistribuidos–Clase06 37/73

s
2
n
.
a
i
p
n
s
i
h
ci
o
a
t
T
S
R
R
7.terminaTR
4.crea 5.COMMIT
v2 lectornuevo:v2 8.v1reclamable
3.leev1 TR:v1
Construcción progresiva del timeline MVCC
NÚCLEO
LectorTR
EscritorTW
Versiones 1.v1
Lecturavisible
tiempo
Paso 1
v1 es la versión confirmada existente.
SistemasDistribuidos–Clase06 38/73

7.terminaTR
4.crea 5.COMMIT
v2 lectornuevo:v2 8.v1reclamable
3.leev1 TR:v1
Construcción progresiva del timeline MVCC
NÚCLEO
LectorTR
s
2
n
.
a
i
p
n
s
i
h
ci
o
a
t
T
S
R
R
EscritorTW
Versiones 1.v1
Lecturavisible
tiempo
Paso 2
T fija una regla de visibilidad: su snapshot.
R
SistemasDistribuidos–Clase06 38/73

7.terminaTR
4.crea 5.COMMIT
v2 lectornuevo:v2 8.v1reclamable
TR:v1
| Construcción |     | progresiva | del timeline | MVCC |
| ------------ | --- | ---------- | ------------ | ---- |
NÚCLEO
2 . i n i ci a T R
|     | LectorTR | s n a p s h o t S |     |     |
| --- | -------- | ----------------- | --- | --- |
R
EscritorTW
|     | Versiones      | 1.v1 |         |     |
| --- | -------------- | ---- | ------- | --- |
|     | Lecturavisible |      | 3.leev1 |     |
tiempo
Paso 3
| La primera | lectura elige | v1. |     |     |
| ---------- | ------------- | --- | --- | --- |
SistemasDistribuidos–Clase06 38/73

7.terminaTR
5.COMMIT
lectornuevo:v2 8.v1reclamable
TR:v1
| Construcción | progresiva | del timeline | MVCC |
| ------------ | ---------- | ------------ | ---- |
NÚCLEO
|          | 2 . i n i ci a T    |     |     |
| -------- | ------------------- | --- | --- |
| LectorTR | s n a p s h o t S R |     |     |
R
EscritorTW 4.crea
| Versiones      | 1.v1 | v2      |     |
| -------------- | ---- | ------- | --- |
| Lecturavisible |      | 3.leev1 |     |
tiempo
Paso 4
| T crea v2 sin borrar | v1. |     |     |
| -------------------- | --- | --- | --- |
W
SistemasDistribuidos–Clase06 38/73

7.terminaTR
lectornuevo:v2 8.v1reclamable
TR:v1
| Construcción | progresiva | del timeline | MVCC |
| ------------ | ---------- | ------------ | ---- |
NÚCLEO
2 . i n i ci a T R
|     | LectorTR s n a | p s h o t S |     |
| --- | -------------- | ----------- | --- |
R
|     | EscritorTW     | 4.crea  | 5.COMMIT |
| --- | -------------- | ------- | -------- |
|     | Versiones 1.v1 | v2      |          |
|     | Lecturavisible | 3.leev1 |          |
tiempo
Paso 5
| v2 confirma | y queda disponible | para snapshots | posteriores. |
| ----------- | ------------------ | -------------- | ------------ |
SistemasDistribuidos–Clase06 38/73

7.terminaTR
8.v1reclamable
| Construcción |     | progresiva | del timeline | MVCC |
| ------------ | --- | ---------- | ------------ | ---- |
NÚCLEO
|     |          | 2 . i n i ci | a T R |     |
| --- | -------- | ------------ | ----- | --- |
|     | LectorTR | s n a p s h  | o t S |     |
R
|     | EscritorTW     |      | 4.crea 5.COMMIT |                |
| --- | -------------- | ---- | --------------- | -------------- |
|     | Versiones      | 1.v1 | v2              | lectornuevo:v2 |
|     | Lecturavisible |      | 3.leev1         | TR:v1          |
tiempo
Paso 6
| T mantiene | v1; | un lector nuevo | puede elegir v2. |     |
| ---------- | --- | --------------- | ---------------- | --- |
R
SistemasDistribuidos–Clase06 38/73

8.v1reclamable
| Construcción |     | progresiva | del timeline | MVCC |
| ------------ | --- | ---------- | ------------ | ---- |
NÚCLEO
|     |          | 2 . i n i ci | a T R |             |
| --- | -------- | ------------ | ----- | ----------- |
|     | LectorTR | s n a p s h  | o t S | 7.terminaTR |
R
|     | EscritorTW     |      | 4.crea 5.COMMIT |                |
| --- | -------------- | ---- | --------------- | -------------- |
|     | Versiones      | 1.v1 | v2              | lectornuevo:v2 |
|     | Lecturavisible |      | 3.leev1         | TR:v1          |
tiempo
Paso 7
| Termina | el snapshot | largo. |     |     |
| ------- | ----------- | ------ | --- | --- |
SistemasDistribuidos–Clase06 38/73

| Construcción |     | progresiva | del | timeline | MVCC |     |
| ------------ | --- | ---------- | --- | -------- | ---- | --- |
NÚCLEO
|     |          | 2 . i n i ci | a T R |     |             |     |
| --- | -------- | ------------ | ----- | --- | ----------- | --- |
|     | LectorTR | s n a p s h  | o t S |     | 7.terminaTR |     |
R
|     | EscritorTW     |      |         | 4.crea 5.COMMIT |                |                |
| --- | -------------- | ---- | ------- | --------------- | -------------- | -------------- |
|     | Versiones      | 1.v1 |         | v2              | lectornuevo:v2 | 8.v1reclamable |
|     | Lecturavisible |      | 3.leev1 |                 | TR:v1          |                |
tiempo
Paso 8
v1 puede limpiarse cuando ninguna regla de visibilidad la necesite.
SistemasDistribuidos–Clase06 38/73

Regla simplificada de visibilidad
NÚCLEO
v1visible v2visible
historialógica
snapshotA snapshotB
eligev1 eligev2
Lectura conceptual
Una versión es visible si su creador pertenece a la historia confirmada que el snapshot
puede observar y su reemplazo/eliminación todavía no era visible para ese snapshot.
SistemasDistribuidos–Clase06 39/73

| Caso | real: PostgreSQL, |     | versiones |     | de fila | y VACUUM |
| ---- | ----------------- | --- | --------- | --- | ------- | -------- |
NÚCLEO
| ▶ Un UPDATE  | crea     | una       | versión nueva | de la | fila. |     |
| ------------ | -------- | --------- | ------------- | ----- | ----- | --- |
| ▶ La versión | anterior | permanece | mientras      | algún |       |     |
v1visible
| snapshot | pueda    | verla.  |              |        |     |     |
| -------- | -------- | ------- | ------------ | ------ | --- | --- |
| ▶ VACUUM | recupera | espacio | reutilizable | cuando | la  |     |
UPDATEcreav2
| historia | deja        | de ser necesaria. |                     |     |     |     |
| -------- | ----------- | ----------------- | ------------------- | --- | --- | --- |
| ▶ Una    | transacción | larga             | retrasa la limpieza |     | y   |     |
snapshotretienev1
| aumenta | bloat, | E/S y | presión sobre | índices. |     |     |
| ------- | ------ | ----- | ------------- | -------- | --- | --- |
VACUUMcuandoesseguro
Fuente/referencia:PostgreSQL,documentaciónoficialsobreMVCCyroutinevacuuming.
SistemasDistribuidos–Clase06 40/73

| Condición | de  | carrera y | write skew bajo | MVCC |
| --------- | --- | --------- | --------------- | ---- |
NÚCLEO
| Condición | de carrera |     |     |     |
| --------- | ---------- | --- | --- | --- |
Invariante: almenosun
Elresultadodependedelordenrelativoenqueoperaciones supervisordebe
| concurrentesleenymodificanestadocompartido. |     |     | Dos | quedaractivo |
| ------------------------------------------- | --- | --- | --- | ------------ |
ejecucionesválidasporseparadopuedenproducirjuntasun
resultadoinválido.
T1 leeA=1,B=1 T2 leeA=1,B=1
desactivaA desactivaB
| Write skew | o sesgo | de escritura |     |     |
| ---------- | ------- | ------------ | --- | --- |
Dostransaccionesleenlamismainstantánea,actualizan
| filasdiferentesyambasconfirman. |     | Noexisteunconflicto |     |     |
| ------------------------------- | --- | ------------------- | --- | --- |
Ambasconfirman
directodeescritura,peroseviolauninvariantequeabarca
A=0,B=0
variasfilas.
Conclusión
MVCCadministraversiones;nodefineporsísoloserializabilidad. Elniveldeaislamientodebedetectar,bloquearoabortar
ejecucionesquepodríanviolarinvariantes.
SistemasDistribuidos–Clase06 41/73

| Índices               | y EXPLAIN: |         |       |
| --------------------- | ---------- | ------- | ----- |
| de una                | versión    | visible | a una |
| ruta eficiente        |            |         |       |
| Sistemas Distribuidos |            |         |       |

| Transición: | MVCC | decide | qué | versión; | el  | índice ayuda a |
| ----------- | ---- | ------ | --- | -------- | --- | -------------- |
encontrarla
NÚCLEO
La pregunta que acabamos de resolver con Lanuevapreguntafísica
| MVCC |     |     |     | ¿Cómo localiza | el motor | las filas candidatas sin |
| ---- | --- | --- | --- | -------------- | -------- | ------------------------ |
Dadas varias versiones de una fila, ¿cuál puede recorrertodaslaspáginasdelatabla?
observarestatransacciónsegúnsusnapshot?
|     | Planificador |     | Rutafísica | Filasyversiones |     | MVCCverifica |
| --- | ------------ | --- | ---------- | --------------- | --- | ------------ |
ConsultaSQL
|                  | compararutas |     | tablaoíndice | candidatas |     | visibilidad |
| ---------------- | ------------ | --- | ------------ | ---------- | --- | ----------- |
| Puente entre los | temas        |     |              |            |     |             |
Elíndicenodecidequéversiónescorrecta: reduceelespaciodebúsqueda. MVCCconservalasemánticadelalectura,y
EXPLAINpermitecomprobarquérutaeligióelplanificador.
SistemasDistribuidos–Clase06 43/73

| CampusShop: | la consulta | define | el patrón | de acceso |
| ----------- | ----------- | ------ | --------- | --------- |
NÚCLEO
Preguntadeproducto: mostrarlasúltimas20comprasconfirmadasdeunaestudiante.
SELECTid,total,confirmed_at
FROMpurchases
WHEREstudent_id=8421ANDstatus=’CONFIRMED’
ORDERBYconfirmed_atDESC
LIMIT20;
|                | 1. igualdad     | 2. igualdad   | 3. orden  | 4. detener |
| -------------- | --------------- | ------------- | --------- | ---------- |
|                | student_id      | status        | fechaDESC | alreunir20 |
| De la consulta | a una hipótesis | de estructura |           |            |
Larutaidealdeberíaubicarprimerostudent_id=8421,conservarsolostatus=’CONFIRMED’,recorrerconfirmed_aten
ordendescendenteydetenersealobtener20filas. Uníndicecompuestopuedematerializarprecisamenteeseorden.
SistemasDistribuidos–Clase06 44/73

Árbol B: páginas balanceadas para evitar un recorrido completo
NÚCLEO
Páginaraízdelíndice:
separadores8000 | 8500
HojaL17 HojaL42 HojaL43
claves< 8000 8000–8499 claves≥ 8500
Definición conectada con las páginas Por qué necesita pocos accesos
Cadanododelárbolsuelealmacenarseenunapágina. Las Elárbolpermanecebalanceadoytienealtofan-out: enuna
clavesdelaspáginasinternasactúancomoseparadores;las páginacabenmuchasclavesyreferencias. Poreso,auncon
hojascontienenclavesordenadasyreferenciashaciafilaso millonesdefilas,larutadesdelaraízhastaunahojasuele
páginasdedatos. tenerpocosniveles.
Costo que no debe ocultarse
Insertar,actualizaroborrarpuedemodificarpáginasdelíndicey,cuandounapáginasellena,provocarunadivisión. Elárbol
aceleradeterminadasconsultasacambiodemásespacioytrabajodeescritura.
SistemasDistribuidos–Clase06 45/73

| CampusShop: | el árbol | convierte | el predicado |     | en  | una ruta |
| ----------- | -------- | --------- | ------------ | --- | --- | -------- |
NÚCLEO
HojaL42
| Prefijobuscado | Raíz |     |     | Recorrer |     | Detener |
| -------------- | ---- | --- | --- | -------- | --- | ------- |
8421, entradas
|           | eligeelrango |           |     | fechaDESC |     | alreunir20 |
| --------- | ------------ | --------- | --- | --------- | --- | ---------- |
| CONFIRMED |              | contiguas |     |           |     |            |
Comprobaciónfinal
Entradasdelahoja,yaordenadas
Lasreferenciasllevanaversiones
(8421,CONFIRMED,12:05)->P17
(8421,CONFIRMED,11:58)->P81 candidatas. MVCCverificacuálesson
| .   |     |     | visiblesparaelsnapshotdelaconsulta |     |     |     |
| --- | --- | --- | ---------------------------------- | --- | --- | --- |
.
| .   |     |     | antesdedevolverlas. |     |     |     |
| --- | --- | --- | ------------------- | --- | --- | --- |
Aporteytransición. ElárbolBreducepáginascandidatas,conservaelordendelaclavecompuestaypermitedetener
el recorrido temprano. Todavía falta demostrar que el planificador lo elija: EXPLAIN mostrará la ruta y EXPLAIN
ANALYZEpermitirácontrastarlaconfilas,buffersytiemposobservados.
SistemasDistribuidos–Clase06 46/73

| Tener un | índice no | garantiza | usarlo: | EXPLAIN | muestra | la  |
| -------- | --------- | --------- | ------- | ------- | ------- | --- |
decisión
NÚCLEO
| EXPLAIN                               |     |         | EXPLAIN                                   | ANALYZE |     |          |
| ------------------------------------- | --- | ------- | ----------------------------------------- | ------- | --- | -------- |
|                                       |     |         | Ejecutalaconsultayagregamedicionesreales: |         |     | tiempos, |
| Muestraelplanelegidoysusestimaciones: |     | formade |                                           |         |     |          |
filas,repeticionesy,conBUFFERS,entrada/salida
acceso,filtros,orden,costoycantidadesperadadefilas.
observada.
SeqScan
ConsultaSQL
recorrertabla
Planificador
comparaalternativas
|     | Estadísticas     |     |     | IndexScan      |     |     |
| --- | ---------------- | --- | --- | -------------- | --- | --- |
|     | ycostosestimados |     |     | recorrerárbolB |     |     |
Laestructuradisponibleessolounaalternativa. Laevidenciaempiezacuandoobservamosquéplanfueelegido,cuánto
trabajorealizóysilasestimacionescoincidieronconlaejecución.
|     |     | SistemasDistribuidos–Clase06 |     |     |     | 47/73 |
| --- | --- | ---------------------------- | --- | --- | --- | ----- |

Cómo leer un plan: de abajo hacia arriba
NÚCLEO
LIMIT:entrega20filas
SORT:ordenaporfechadescendente
Empieceporelnodoinferior:
allíapareceeltrabajofísico
FILTER:conservaestudianteyestado inicial. Luegosubaypregunte
cuántotrabajodescartao
transformacadanodo.
SEQSCAN:visitalatabla
SistemasDistribuidos–Clase06 48/73

| Sin índice: | mucho | trabajo para encontrar | 20 filas |     |
| ----------- | ----- | ---------------------- | -------- | --- |
NÚCLEO
| Limit (actual | time=12.55..12.58 | rows=20) |                 |     |
| ------------- | ----------------- | -------- | --------------- | --- |
| -> Sort       | (rows=20)         |          | Tres evidencias |     |
| Sort Key:     | confirmed_at DESC |          |                 |     |
▶
| -> Seq    | Scan on purchases  |      | Seq Scan.     |              |
| --------- | ------------------ | ---- | ------------- | ------------ |
| Filter:   | student_id=8421    |      | ▶             |              |
|           |                    |      | 9980 filas    | descartadas. |
| Rows      | Removed by Filter: | 9980 |               |              |
| Buffers:  | shared hit=840     |      | ▶ 840 buffers | tocados.     |
| Execution | Time: 12.84 ms     |      |               |              |
Interpretación
El motor recorre, filtra y ordena un conjunto mucho mayor que las 20 filas finales.
SistemasDistribuidos–Clase06 49/73

| Con | índice compuesto: |     | el plan hace | menos | trabajo |
| --- | ----------------- | --- | ------------ | ----- | ------- |
NÚCLEO
| Limit (actual                    | time=0.20..0.24 | rows=20) |     |           |                      |
| -------------------------------- | --------------- | -------- | --- | --------- | -------------------- |
| -> Index                         | Scan using      |          |     | Quécambió |                      |
| idx_purchase_student_status_date |                 |          |     |           | ▶ Setocaron5buffers. |
| on purchases                     |                 |          |     |           | ▶                    |
Seobtuvieron20filasútiles.
| Index | Cond: student_id=8421 |     |     |     |     |
| ----- | --------------------- | --- | --- | --- | --- |
▶ Nohuboordenamiento
| AND | status=’CONFIRMED’ |     |     |     |     |
| --- | ------------------ | --- | --- | --- | --- |
separado.
| Buffers: | shared hit=5 |     |     |     |     |
| -------- | ------------ | --- | --- | --- | --- |
▶
| Execution | Time: 0.27 ms |     |     |     | LIMITdetuvoelrecorrido |
| --------- | ------------- | --- | --- | --- | ---------------------- |
temprano.
DDL utilizado
| CREATE INDEX | ... ON purchases(student_id, |     | status, |     |     |
| ------------ | ---------------------------- | --- | ------- | --- | --- |
| confirmed_at | DESC);                       |     |         |     |     |
SistemasDistribuidos–Clase06 50/73

| Comparar | planes | sin | convertir | los | números | en una | promesa |
| -------- | ------ | --- | --------- | --- | ------- | ------ | ------- |
universal
NÚCLEO
|     |     | Evidencia         |             | Sin índice | Con índice |         |     |
| --- | --- | ----------------- | ----------- | ---------- | ---------- | ------- | --- |
|     |     | Acceso            |             | Seq Scan   | Index      | Scan    |     |
|     |     | Filas descartadas |             | 9980       | 0 en el    | ejemplo |     |
|     |     | Buffers           |             | 840        | 5          |         |     |
|     |     | Sort explícito    |             | Sí         | No         |         |     |
|     |     | Tiempo            | ilustrativo | 12.84 ms   | 0.27       | ms      |     |
Cuidado
Los valores dependen de datos, estadísticas, caché, selectividad y hardware. La lección es
| aprender | a leer evidencia, | no  | memorizar | una cifra. |     |     |     |
| -------- | ----------------- | --- | --------- | ---------- | --- | --- | --- |
SistemasDistribuidos–Clase06 51/73

| El costo | del índice | aparece | en cada | escritura |     |
| -------- | ---------- | ------- | ------- | --------- | --- |
NÚCLEO
|          | INSERT     | tabla      | índice1 | índice2  | índice3          |
| -------- | ---------- | ---------- | ------- | -------- | ---------------- |
|          |            | máspáginas | másWAL  | máscaché | másmantenimiento |
| Regla de | ingeniería |            |         |          |                  |
No se agrega un índice porque “parece útil”. Se formula una hipótesis, se observa el plan y
| se mide la | carga de trabajo | real. |     |     |     |
| ---------- | ---------------- | ----- | --- | --- | --- |
SistemasDistribuidos–Clase06 52/73

| LSM-tree:    | escrituras | rápidas, |
| ------------ | ---------- | -------- |
| organización | posterior  |          |
Sistemas Distribuidos

Qué es una LSM-tree y por qué aparece
NÚCLEO
Definición
WAL
UnaLSM-treeacumulacambiosenunaestructuraordenadaen
memoria. Cuandosellena,lavuelcacomounaSSTable(Sorted
StringTable: archivoordenadoeinmutabledeparesclave–valor).
memtableordenada
MástardefusionavariasSSTablesmediantecompactación.
flush
Enelcaminocríticodelasolicitud,elmotorevitaactualizar
deinmediatomuchaspáginasdispersas. Registralaescritura SSTablesenL0
ymodificamemoria;eltrabajodereorganizaciónfísicaqueda primernivelpersistente
paraflushycompactaciónensegundoplano.
compactación
No elimina costo
nivelesL1,L2,...
Desplazatrabajo: puedeaumentarbytesreescritos,archivos
consultados,espaciotemporalylatenciacuandoelmantenimiento
quedaatrás.
SistemasDistribuidos–Clase06 54/73

| Vocabulario | LSM | antes de | hablar de | compactación |
| ----------- | --- | -------- | --------- | ------------ |
NÚCLEO
| Término |     | Significado |     |     |
| ------- | --- | ----------- | --- | --- |
Memtable Tabla ordenada en memoria que recibe las escrituras recientes.
Flush Volcado de una memtable inmutable a un archivo persistente.
SSTable Sorted String Table: archivo ordenado e inmutable de pares
clave–valor.
Run Secuencia o archivo ya ordenado que puede participar en una
fusión.
Nivel L0, L1, L2 L0 es el primer nivel persistente que recibe archivos del flush;
L1,L2ysiguientesorganizanSSTablesprogresivamentemayores
|     |     | según la política | de compactación. |     |
| --- | --- | ----------------- | ---------------- | --- |
Marca de borrado Registroquerepresentaunaeliminaciónydebeconservarsehasta
| (tombstone) |     | que sea seguro | olvidar versiones | anteriores. |
| ----------- | --- | -------------- | ----------------- | ----------- |
SistemasDistribuidos–Clase06 55/73

Vocabulario LSM: mantenimiento y protección
NÚCLEO
Término Significado
Fusiónordenada(merge) Recorre varias secuencias ya ordenadas y produce otra secuencia ordenada,
eligiendoencadapasolamenorclavedisponible.
Compactación Fusiónordenadaqueademásreducesolapamientoyeliminaversionesobsoletas
otombstonescuandoesseguro.
Trabajopendiente(backlog) BytesoSSTablesqueesperancompactación. Crececuandolatasadeingreso
superasostenidamentelacapacidaddemantenimiento.
Regulacióndeadmisión Elmotorintroduceespera,bloqueahilosolimitapermisosparareducirlatasa
denuevasescrituras.
Pausadeescritura(writestall) Regulación fuerte en la que una solicitud de escritura no avanza hasta que el
trabajodefondoreduceelbacklogoseliberaunrecurso.
Sepuederetrasarobloquearlasolicitudqueintentagenerarmástrabajo.
SistemasDistribuidos–Clase06 56/73

| Caso contrastante: | telemetría | del campus |     |
| ------------------ | ---------- | ---------- | --- |
NÚCLEO
Sensoresdeaulas,energíayocupaciónproducen80.000eventosporsegundo. Lasconsultasrecorrenundispositivoy
unaventanatemporal.
sensordeaula
|     | canaldeingestión | claveordenada | motorLSM |
| --- | ---------------- | ------------- | -------- |
medidordeenergía
|     | 80.000eventos/s | (dispositivo,tiempo) | retencióntemporal |
| --- | --------------- | -------------------- | ----------------- |
sensorclimático
| ▶   |     | ▶   |     |
| --- | --- | --- | --- |
Tasadeescrituraaltaysostenida. Retenciónyborradoporantigüedad.
| ▶   |     | ▶   |     |
| --- | --- | --- | --- |
Clavesordenablespordispositivoytiempo. Búsquedaspuntualesyrangostemporales.
SistemasDistribuidos–Clase06 57/73

| Ruta | de escritura | LSM: de | la solicitud | a una SSTable | L0  |
| ---- | ------------ | ------- | ------------ | ------------- | --- |
NÚCLEO
|     | escritura |     | memtable | volcado | SSTable |
| --- | --------- | --- | -------- | ------- | ------- |
WAL memtable
|            | (k,v)     |     | inmutable | (flush) | nivelL0 |
| ---------- | --------- | --- | --------- | ------- | ------- |
| Qué ocurre | antes del | ACK |           |         |         |
Según la garantía de durabilidad, la escritura se registra en el WAL y se incorpora a la memtable. El
flush de la memtable inmutable hacia una SSTable L0 puede ocurrir después y no tiene por qué
| bloquear  | esa misma solicitud. |     |     |     |     |
| --------- | -------------------- | --- | --- | --- | --- |
| Qué queda | para después         |     |     |     |     |
L0 puede acumular archivos solapados. La compactación posterior reorganiza esos archivos y los
| desplaza | hacia niveles | más estables. |     |     |     |
| -------- | ------------- | ------------- | --- | --- | --- |
SistemasDistribuidos–Clase06 58/73

Compactación: merge ordenado y eliminación segura
NÚCLEO
SSTableA
a,c1,e,g
mergeordenado
SSTableB SSTableresultante
comparaclaves
b,c2,f,h a,b,e,f,g,h
conservalaversiónvigente
marcadeborrado
eliminarc
Qué se paga
La compactación lee y reescribe datos, utiliza espacio temporal, compite por E/S y puede aumentar
el percentil 99 de latencia.
SistemasDistribuidos–Clase06 59/73

| Filtro de | Bloom: | decidir qué | SSTables | se pueden | omitir |
| --------- | ------ | ----------- | -------- | --------- | ------ |
NÚCLEO
Un filtro de Bloom es un resumen probabilístico y compacto de las claves de una SSTable. Para una búsqueda de
claveexactanodevuelveelvalor: respondedefinitivamenteausenteoposiblementepresente.
algúnbites0
|     |     |     | buscarz | definitivamente |     |
| --- | --- | --- | ------- | --------------- | --- |
ausente
|     | SSTableA        | FiltrodeBloom   |     |     |     |
| --- | --------------- | --------------- | --- | --- | --- |
|     | claves: a,c,e,g | resumendeclaves |     |     |     |
todoslosbitsson1
|     |     |     | buscarc |     | posiblemente |
| --- | --- | --- | ------- | --- | ------------ |
presente
▶
| Definitivamenteausente: |     | elmotoromiteesaSSTableparalabúsqueda. |     |     |     |
| ----------------------- | --- | ------------------------------------- | --- | --- | --- |
▶ Posiblementepresente: elmotorconsultaelíndicey,sicorresponde,losbloquesdedatos;puedeserunfalso
positivo.
▶
Unfiltrobienconstruidonodebeproducirfalsosnegativos: nopuededescartarunarchivoquesícontienelaclave.
SistemasDistribuidos–Clase06 60/73

| Ruta | de lectura LSM: | descartar | antes | de leer el dispositivo |
| ---- | --------------- | --------- | ----- | ---------------------- |
NÚCLEO
No
omitirSSTable
|     | revisar | SSTable | ¿Bloomdice |     |
| --- | ------- | ------- | ---------- | --- |
buscarclavek
|     | memtables | candidata | “talvez”? |     |
| --- | --------- | --------- | --------- | --- |
cachéo
Sí
bloqueen
consultaríndice dispositivo
| Orden | de búsqueda |     | Resultado | correcto |
| ----- | ----------- | --- | --------- | -------- |
Primeroserevisalomásreciente. ParacadaSSTable Siaparecenvariasversiones,elmotoreligelamásreciente
candidata,Bloompuedeevitarunalecturafísica. queseavisibleyrespetalasmarcasdeborrado.
▶
Unfalsopositivoagregatrabajo;unfalsonegativoharíaperderundatoexistenteynoesadmisible.
▶ Lacachédebloquesevitaaccederaldispositivocuandoelbloquerequeridoyaestáenmemoria.
SistemasDistribuidos–Clase06 61/73

| Panel de | compactación | I: qué señales | observar |
| -------- | ------------ | -------------- | -------- |
NÚCLEO
| Señal         |     | Valorilustrativo | Quéinforma                      |
| ------------- | --- | ---------------- | ------------------------------- |
| Ingresológico |     | 90MB/s           | Trabajonuevorecibidoporelmotor. |
Capacidaddecompactación 60MB/s Ritmodereorganizacióndearchivos.
| Déficitsostenido |     | 30MB/s | Trabajoqueseacumulaporsegundo. |
| ---------------- | --- | ------ | ------------------------------ |
| Trabajopendiente |     | 18GB   | Datostodavíasincompactar.      |
| ArchivosenL0     |     | 28de36 | Cercaníaalumbraldeprotección.  |
Percentil99deescritura Enaumento Impactosobrelasoperacionesmáslentas.
Consideración. Sedebenrelacionarelingreso,lacapacidaddemantenimiento,elcrecimientodeltrabajopendienteyla
latencia;ningunacifraaisladabastaparadiagnosticar.
SistemasDistribuidos–Clase06 62/73

| Panel de | compactación | II: leer | la evolución |     |
| -------- | ------------ | -------- | ------------ | --- |
NÚCLEO
|     | Trabajo pendiente | de compactación | durante | 10 minutos |
| --- | ----------------- | --------------- | ------- | ---------- |
GB
umbral:bloquearadmisión
umbral:retrasarnuevasescrituras
minutos
|     | 0   |     | 10  |     |
| --- | --- | --- | --- | --- |
Cadena causal
Ingreso mayor que capacidad de compactación ⇒ backlog creciente ⇒ más SSTables en L0 ⇒ más
⇒
trabajo de lectura y mayor latencia regulación de nuevas escrituras.
SistemasDistribuidos–Clase06 63/73

| Cómo regula | escrituras | un write | stall |
| ----------- | ---------- | -------- | ----- |
NÚCLEO
Implementaciones típicas
|     | nuevasolicitud | controldeadmisión |     |
| --- | -------------- | ----------------- | --- |
|     | deescritura    | revisaL0ybacklog  | ▶   |
dormirobloquearelhiloescritor;
▶
limitartokens,concurrenciaobytespor
flush/ segundo;
| bajadelumbral |     | superaumbral |     |
| ------------- | --- | ------------ | --- |
compactación ▶
reanudar retrasarobloquear esperaraquedisminuyanlasSSTablesL0;
reducentrabajo
▶ enalgunossistemas,responder
sobrecarga.
Interpretación
Lapausaprotegeunacolafinita;nocreacapacidad. Siesfrecuenteoprolongada,debenrevisarsecarga,hardwareypolítica
decompactación.
SistemasDistribuidos–Clase06 64/73

| Tres amplificaciones: | medir | el trabajo | adicional |     |
| --------------------- | ----- | ---------- | --------- | --- |
NÚCLEO
| Métrica | Definición |     | Ejemplo |     |
| ------- | ---------- | --- | ------- | --- |
Amplificacióndeescritura Bytesfísicosescritosdivididosporbyteslógicos 1 GB lógico produce 6 GB
aceptados. IncluyeWAL,flush,compactacióne físicos: amplificación=6.
índicessegúnelalcancemedido.
Amplificacióndelectura Estructuras o bloques adicionales que deben Una clave obliga a revisar 4
|     | consultarseparaunalecturalógica. |     | SSTables antes | de hallar su |
| --- | -------------------------------- | --- | -------------- | ------------ |
versión.
Amplificacióndeespacio Espacio físico ocupado dividido por el tamaño 1 GB vivo ocupa 1,5 GB:
delestadológicovivo. amplificación=1,5.
“Amplificación” no significa error: significa que la implementación realiza más trabajo físico que la operación lógica
visibleparalaaplicación.
SistemasDistribuidos–Clase06 65/73

| Compactación | por niveles | y por grupos: | qué cambia |
| ------------ | ----------- | ------------- | ---------- |
NÚCLEO
| Por niveles | (leveled) | Por grupos | (tiered / universal) |
| ----------- | --------- | ---------- | -------------------- |
Cadanivelprocurarangospocosolapados.Losdatossefusionanvariasvecesal Seconservanvariosrunscompletos,normalmentesolapados,ysefusionaun
avanzarhacianivelesmayores. grupocuandoalcanzaciertotamañoocantidad.
run1 run2 run3
| L0  | a–m g–z |     |     |
| --- | ------- | --- | --- |
a–z a–z a–z
|     | merge |     | mergedelgrupo |
| --- | ----- | --- | ------------- |
nuevorun
|     | L1 a–f g–m n–z |     |     |
| --- | -------------- | --- | --- |
a–z
▶ ▶
| menosarchivoscandidatosporclave; |     | menosreescriturainmediata; |     |
| -------------------------------- | --- | -------------------------- | --- |
▶ ▶
| mejorcontroldelespacio; |     | másarchivospotencialesporlectura; |     |
| ----------------------- | --- | --------------------------------- | --- |
▶ ▶
másespaciotemporaldurantelafusión.
másreescriturasdurantelosmerges.
Consultaporclaveexacta: buscaunidentificadorconcreto,como9F2A.Sisuspercentiles95y99debenserbajos,
revisarmuchosrunsresultacostoso.
SistemasDistribuidos–Clase06 66/73

| Integración: | decidir | con |
| ------------ | ------- | --- |
evidencia
Sistemas Distribuidos

| Árbol | B o LSM-tree: | orientación, | no ley universal |
| ----- | ------------- | ------------ | ---------------- |
NÚCLEO
| Criterio | ÁrbolBorientadoapáginas |     | LSM-treeorientadaaruns |
| -------- | ----------------------- | --- | ---------------------- |
Escritura Actualizapáginasdedatoseíndices;puededividirpáginas. WAL+memtable+flush;compactaciónposterior.
Claveexacta Pocaspáginassielíndicecoincideconlaconsulta. VariasSSTablesposibles;Bloomycachéreducenlecturas.
Rangos Recorridonaturaldehojasordenadas. Eficientesihaypocosrunsrelevantes.
Mantenimiento Divisiones,VACUUM,bloatyreconstrucción. Tombstones,compactación,amplificacionesystalls.
Cargafrecuente Transaccionesmixtas;clavesexactasyrangos. Ingestiónaltaysostenida;escriturasagrupables.
Preguntacrítica ¿Qué consultas justifican el índice y cuánto encarece ¿Puedeelmantenimientosostenerelingresosinbacklog
|     | escribir? |     | creciente? |
| --- | --------- | --- | ---------- |
Criterio
Comparepatróndeacceso,garantía,capacidadsostenible,latenciaenpercentiles95y99yamplificacionesmedidas;noelnombredelproducto.
SistemasDistribuidos–Clase06 68/73

| Dos casos | cercanos, | dos hipótesis | defendibles |
| --------- | --------- | ------------- | ----------- |
NÚCLEO
| CampusShop                    |     |     | Telemetríadelcampus          |
| ----------------------------- | --- | --- | ---------------------------- |
| ▶                             |     |     | ▶                            |
| Transaccionescortas.          |     |     | Ingestiónmasivaysecuencial.  |
| ▶                             |     |     | ▶                            |
| Inventarioeidempotencia.      |     |     | Rangospordispositivoytiempo. |
| ▶                             |     |     | ▶                            |
| Consultasporestudianteyfecha. |     |     | Retencióntemporal.           |
| ▶                             |     |     | ▶                            |
| Percentil99predecible.        |     |     | Vistasconfrescuraeventual.   |
Hipótesis: WAL+MVCC+árbolB. Hipótesis: WAL+LSM+compactacióntemporal.
| Todavía falta | evidencia |     |     |
| ------------- | --------- | --- | --- |
Medirpercentiles95y99,throughputsostenible,objetivodepuntoderecuperación(RPO),objetivodetiempode
recuperación(RTO),amplificación,backlog,tasadeaciertosdecachéyespacio.
SistemasDistribuidos–Clase06 69/73

Repasemos
ACTIVIDAD
▶ El COMMIT puede ser durable aunque la página siga atrasada.
▶
| Copiar | en memoria | principal | no equivale | a persistir. |
| ------ | ---------- | --------- | ----------- | ------------ |
▶
| MVCC | reduce contención, |     | pero no elimina | todas las anomalías. |
| ---- | ------------------ | --- | --------------- | -------------------- |
▶ Un índice puede no utilizarse y siempre tiene costo de mantenimiento.
| ▶ Una LSM | desplaza | trabajo | hacia el flush | y la compactación. |
| --------- | -------- | ------- | -------------- | ------------------ |
Patrón común
Ningún mecanismo elimina el costo: cambia cuándo, dónde y cómo se paga.
SistemasDistribuidos–Clase06 70/73

| Método |     | reusable | para decidir | almacenamiento |     |     |     |
| ------ | --- | -------- | ------------ | -------------- | --- | --- | --- |
NÚCLEO
|     | 1.  | Carga | 2. Garantía | 3. Mecanismo |     | 4. Costo | 5. Evidencia |
| --- | --- | ----- | ----------- | ------------ | --- | -------- | ------------ |
revisar
▶ Describa proporciones de lectura y escritura, tamaño, sesgo, crecimiento y consultas.
▶ Declare qué falla debe tolerarse y qué significa durable en ese modelo.
▶
| Elija         | un  | mecanismo      | inicial, no     | un producto    | por reputación. |     |     |
| ------------- | --- | -------------- | --------------- | -------------- | --------------- | --- | --- |
| ▶ Identifique |     | el costo       | que se desplaza | a otra         | etapa.          |     |     |
| ▶ Diseñe      |     | un experimento | capaz           | de contradecir | su elección.    |     |     |
SistemasDistribuidos–Clase06 71/73

| Cierre: | de la intuición | a la evidencia |     |
| ------- | --------------- | -------------- | --- |
NÚCLEO
compactación
WAL índice+
|     | promesa | MVCC | +pausa |
| --- | ------- | ---- | ------ |
permite EXPLAIN
|     | denegocio | mantienevistas | protegen |
| --- | --------- | -------------- | -------- |
recuperar reducentrabajo
capacidad
Idea final
El almacenamiento no es una caja debajo del sistema. Es la capa donde las promesas
lógicas se convierten en bytes, esperas, recuperación y riesgos medibles.
SistemasDistribuidos–Clase06 72/73

Exit ticket
ACTIVIDAD
| En casa, procure   | responder | en tres líneas | por punto:         |
| ------------------ | --------- | -------------- | ------------------ |
| 1. ¿Qué diferencia | hay entre | aceptado,      | escrito y durable? |
2. ¿Qué condición de carrera ilustra el write skew y por qué MVCC no basta para evitarla?
| 3. ¿Qué diferencia | hay entre | EXPLAIN y | EXPLAIN ANALYZE? |
| ------------------ | --------- | --------- | ---------------- |
4. ¿Cómo aplica realmente el motor una pausa de escritura y qué métricas justifican revisarla?
| Puente a la | clase 7 |     |     |
| ----------- | ------- | --- | --- |
Hoy vimos cómo un nodo conserva y organiza su historia. La próxima clase estudia cómo un log se
convierte en una interfaz distribuida entre productores y consumidores.
SistemasDistribuidos–Clase06 73/73