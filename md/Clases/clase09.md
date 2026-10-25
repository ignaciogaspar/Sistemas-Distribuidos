| Sistemas       | Distribuidos   |                  |          |        |         |
| -------------- | -------------- | ---------------- | -------- | ------ | ------- |
| Clase          | 09 — Fronteras | de arquitectura: |          |        |         |
| monolito       | modular,       | servicios,       | eventos, | outbox | y sagas |
| Departamento   | de Computación |                  |          |        |         |
| Ingeniería     | Informática    | – Comisión A     | – Pilar  |        |         |
| Equipo docente | – 2026         |                  |          |        |         |

| La  | historia | que | guiará | toda | la clase |     |     |
| --- | -------- | --- | ------ | ---- | -------- | --- | --- |
NÚCLEO
Asteria Online es un mundo virtual medieval multijugador. Hoy no diseñaremos el juego
completo: seguiremos una única compra para descubrir cuándo una separación arquitectónica
| ayuda | y cuándo | solo | agrega red, | fallas y operación. |     |     |     |
| ----- | -------- | ---- | ----------- | ------------------- | --- | --- | --- |
Inventario
|     | Sofía |     | Mercado |     |     | Pagos | Ranking |
| --- | ----- | --- | ------- | --- | --- | ----- | ------- |
reservay
|     | jugadora |     | listalaespada |     |     | autoriza | actualizadespués |
| --- | -------- | --- | ------------- | --- | --- | -------- | ---------------- |
transfiere
Objeto concreto
Sofía intenta comprar la pieza única EspadaSolar#17. La compra involucra mercado, inventario,
pago y efectos posteriores; el objeto no puede terminar con dos propietarios.
SistemasDistribuidos–Clase09 2/92

| El alcance:  | qué           | entra y qué queda | fuera                |            |
| ------------ | ------------- | ----------------- | -------------------- | ---------- |
| Analizaremos |               |                   | No diseñaremos       | hoy        |
| ▶ catálogo   | y publicación | de la pieza;      | ▶                    |            |
|              |               |                   | física y renderizado | del mundo; |
▶
reserva y propiedad en inventario; ▶ protocolo de movimiento en tiempo real;
▶
| autorización | del pago; |     | ▶ emparejamiento | de jugadores |
| ------------ | --------- | --- | ---------------- | ------------ |
▶
| ranking | y notificaciones | posteriores; | (matchmaking); |     |
| ------- | ---------------- | ------------ | -------------- | --- |
| ▶       |                  |              | ▶              |     |
equipos, puesta en producción, datos y detección de trampas (anti-cheat);
| fallas. |     |     | ▶ infraestructura | gráfica del cliente. |
| ------- | --- | --- | ----------------- | -------------------- |
Acotarelcasoevitausarpalabrasgrandessobreunsistemaindefinido. Cadadecisiónsejustificará
| sobre esta | compra y sus | fallas. |     |     |
| ---------- | ------------ | ------- | --- | --- |
NÚCLEO
SistemasDistribuidos–Clase09 3/92

| La compra | de EspadaSolar#17: | historia | completa |     |
| --------- | ------------------ | -------- | -------- | --- |
CASOCONDUCTOR
5. Inventario
1. Sofía 2. Mercado 3. Inventario 4. Pagos transfiere 6. Rankingy
pulsacomprar reserva#17
|          | validaoferta | autoriza | propietario | notificación |
| -------- | ------------ | -------- | ----------- | ------------ |
| No puede | romperse     | Puede    | demorarse   |              |
▶
| ▶   |     | elranking; |     |     |
| --- | --- | ---------- | --- | --- |
lapiezatienecomomáximounpropietario;
| ▶                                |     | ▶               |     |     |
| -------------------------------- | --- | --------------- | --- | --- |
| nosecobradosveceselmismointento; |     | lanotificación; |     |     |
▶
| ▶   |     | unavistadeanalítica; |     |     |
| --- | --- | -------------------- | --- | --- |
unacompraconfirmadanodesaparece.
▶
elhistorialderivado.
SistemasDistribuidos–Clase09 4/92

| Vocabulario | intuitivo | I: dominio, | capacidad | e invariante |
| ----------- | --------- | ----------- | --------- | ------------ |
VOCABULARIOPREVIO
| Concepto | Ideaintuitiva |     | EjemploenAsteriaOnline |     |
| -------- | ------------- | --- | ---------------------- | --- |
Dominio Parte de la realidad con problemas, reglas El comercio de objetos incluye ofertas,
|     | ypalabraspropias. |     | vendedores,compradores,preciosyventas. |     |
| --- | ----------------- | --- | -------------------------------------- | --- |
Capacidad Responsabilidad estable que el negocio Gestionar la propiedad: reservar,transferir,
necesita sostener. Agrupa reglas, datos liberaryconsultarquiénposeeunapieza.
|     | y varias | operaciones; no es | una acción |     |
| --- | -------- | ------------------ | ---------- | --- |
aislada.
Invariante Reglaquedebeseguirsiendoverdaderaen EspadaSolar#17 tiene como máximo un
|     | todoestadoválido. |     | propietarioyunintentonosecobradosveces. |     |
| --- | ----------------- | --- | --------------------------------------- | --- |
Primeroidentificamosunaresponsabilidadylareglaquedebepreservar. Reciéndespuésdecidimossiseimplementará
comomódulo,procesooservicio.
SistemasDistribuidos–Clase09 5/92

Vocabulario intuitivo II: frontera y dato autoritativo
Frontera
Límite explícito que separa dos ámbitos del sistema. Indica qué responsabilidades, reglas y datos
quedan de cada lado y por qué puertas se permite interactuar.
frontera
Inventario
API/comando Mercado
propiedady
oferta,vendedoryprecio
reservade#17
eventoconfirmado
Inventarioconservaeldatoautoritativosobreelpropietario. Mercadonolomodificadirectamente:
cruza la frontera por una operación permitida o consume un hecho ya confirmado.
VOCABULARIOPREVIO
SistemasDistribuidos–Clase09 6/92

| Vocabulario | intuitivo | III: autoridad | y responsabilidad |
| ----------- | --------- | -------------- | ----------------- |
VOCABULARIOPREVIO
| Concepto | Definicióndetrabajo |     | EnAsteria |
| -------- | ------------------- | --- | --------- |
Autoridad Componenteconlaúltimapalabrasobreun Inventario decide quién es el propietario
|     | hechodenegociodeterminado. |     | válidodeEspadaSolar#17. |
| --- | -------------------------- | --- | ----------------------- |
Responsabilidad (own- Equipo que puede cambiar reglas y datos El equipo de Inventario mantiene el
ership) y que responde por métricas, incidentes y invariante de propiedad y atiende una
|     | recuperación. |     | duplicación. |
| --- | ------------- | --- | ------------ |
Datoautoritativo Registroreconocidocomoverdadoficialpara propietario(#17)=Sofía; Ranking
|     | decidir. |     | mantieneunavistaderivada. |
| --- | -------- | --- | ------------------------- |
Autoridadrespondequiéndecideelhecho;responsabilidadrespondequiénsostieneesadecisióndurantetodala
vidadelsistema.
SistemasDistribuidos–Clase09 7/92

| Vocabulario | intuitivo | IV: versión | y puesta | en producción |
| ----------- | --------- | ----------- | -------- | ------------- |
VOCABULARIOPREVIO
| Término | Definicióndetrabajo |     |     | Preguntaobservable |
| ------- | ------------------- | --- | --- | ------------------ |
Unidaddepublicación Conjunto de componentes que se publica, ¿MercadoeInventariodebensalirsiempre
|     | actualizaoretiracomounamismaversión. |     |     | juntos? |
| --- | ------------------------------------ | --- | --- | ------- |
Puestaenproducción Hacerqueunaversiónatiendatráficoreal. ¿Ranking puede publicar una versión sin
volverapublicarInventario?
Vuelta atrás de versión Retirar una versión problemática y regresar ¿Podemos volver sin interrumpir las
| (rollback) | tráficoosoftwareaunaversiónconocida. |     |     | compras? |
| ---------- | ------------------------------------ | --- | --- | -------- |
Volverelsoftwareaunaversiónanteriornodeshacepagos,eventosnitransferenciasdepropiedadyaconfirmadas. Esos
efectosrequierenreparacióndenegocio.
SistemasDistribuidos–Clase09 8/92

| Vocabulario | intuitivo | V: autonomía |
| ----------- | --------- | ------------ |
VOCABULARIOPREVIO
| Definición de | trabajo |     |
| ------------- | ------- | --- |
Laautonomíaeselgradoenqueunapartepuedecambiar,publicarunaversión,escalarorecuperarsesinobligaralrestoa
hacerloalmismotiempo.
| Dimensión |     | PreguntaobservableenAsteria |
| --------- | --- | --------------------------- |
Cambio ¿Inventario puede modificar su lógica sin exigir cambios simultáneos en
Mercado?
Puestaenproducción ¿RankingpuedesalirovolveratrássinvolverapublicarInventario?
Escalado ¿RankingpuedeagregarrecursosduranteuntorneosinescalartambiénPagos?
Recuperación ¿UnafalladeRankingpuederesolversesindetenerlacompra?
Separarnocreaautonomíaporsísolo: unabasecompartida,versionescoordinadasounequiposincapacidadoperativa
puedenmantenerelacoplamiento.
SistemasDistribuidos–Clase09 9/92

| Dos formas | de ejecutar | una secuencia | de  | trabajo |
| ---------- | ----------- | ------------- | --- | ------- |
VOCABULARIOPREVIO
| Cadena sincrónica |     |     | Continuación | asíncrona |
| ----------------- | --- | --- | ------------ | --------- |
La petición espera una respuesta antes de Un hecho durable permite que otros continúen
| continuar: |     |     | después: |               |
| ---------- | --- | --- | -------- | ------------- |
| A→B→C→D    |     |     |          | A→log⇒{C,D,E} |
Si C tarda o falla, A todavía no puede terminar. A no espera a todos los lectores.
No son estilos universales. Una autorización de pago puede necesitar respuesta inmediata;
| actualizar ranking | normalmente | no. |     |     |
| ------------------ | ----------- | --- | --- | --- |
SistemasDistribuidos–Clase09 10/92

| Cómo vamos | a recorrer | la historia |     |     |
| ---------- | ---------- | ----------- | --- | --- |
NÚCLEO
| 1. Comprender    | 2. Dibujar | 3. Cruzar | 4. Preservar | 5. Migrary |
| ---------------- | ---------- | --------- | ------------ | ---------- |
| elsistemaactual  | fronteras  | lared     | integridad   | operar     |
| Pregunta que une | la clase   |           |              |            |
¿Québeneficioconcretoobtienecadaseparaciónyquénuevafalla,latencia,datoduplicadooresponsabilidadoperacional
introduce?
SistemasDistribuidos–Clase09 11/92

| Evidencias | de aprendizaje |     | que | produciremos |     |     |
| ---------- | -------------- | --- | --- | ------------ | --- | --- |
NÚCLEO
| Durante | la clase |     |     | Al cerrar |     |     |
| ------- | -------- | --- | --- | --------- | --- | --- |
| ▶       |          |     |     | ▶         |     |     |
delimitar una capacidad y su dato justificar una frontera con evidencia;
autoritativo;
|     |     |     |     | ▶ diseñar outbox | e idempotencia; |     |
| --- | --- | --- | --- | ---------------- | --------------- | --- |
▶
| transformar | una llamada | local en | una |               |               |     |
| ----------- | ----------- | -------- | --- | ------------- | ------------- | --- |
|             |             |          |     | ▶ modelar una | saga durable; |     |
| cadena      | sincrónica; |          |     |               |               |     |
▶
| ▶     |              |                |     | proponer | una migración reversible | y   |
| ----- | ------------ | -------------- | --- | -------- | ------------------------ | --- |
| mover | un crash por | un dual write; |     |          |                          |     |
métricas.
| ▶ localizar | qué invariante | deja de ser | local.                       |     |     |       |
| ----------- | -------------- | ----------- | ---------------------------- | --- | --- | ----- |
|             |                |             | SistemasDistribuidos–Clase09 |     |     | 12/92 |

| De una | regla        | local |       |
| ------ | ------------ | ----- | ----- |
| a una  | coordinación |       | entre |
fronteras
| Sistemas Distribuidos |     |     |     |
| --------------------- | --- | --- | --- |

| Puente | desde | la Clase 08: | ¿cuándo | hace falta | un protocolo? |
| ------ | ----- | ------------ | ------- | ---------- | ------------- |
NÚCLEO
Regla local
Si una sola autoridad posee el dato, puede preservar el invariante dentro de una transacción local:
| Inventario | decide quién | posee EspadaSolar#17. |     |     |     |
| ---------- | ------------ | --------------------- | --- | --- | --- |
Regla distribuida
Si la decisión atraviesa procesos independientes, hace falta un protocolo de coordinación: una
secuencia acordada de mensajes, estados durables, IDs, timeouts, retries y reglas de recuperación.
Mover una frontera no elimina la regla de negocio: transforma una transacción local en un
| protocolo | que debe | sobrevivir fallas parciales. |     |     |     |
| --------- | -------- | ---------------------------- | --- | --- | --- |
SistemasDistribuidos–Clase09 14/92

Diagnóstico de entrada: vote antes de discutir
ACTIVIDAD
1. Un monolito termina necesariamente como una maraña de dependencias.
2. Un microservicio es una clase de negocio expuesta por HTTP.
3. Separar procesos vuelve independientes a los equipos automáticamente.
4. Publicar un evento después de guardar en la base resuelve el dual write.
5. Una compensación de saga equivale a deshacer perfectamente la operación.
Consigna
Vote V/F. No busque todavía la definición perfecta: anote qué palabra le genera desconfianza.
SistemasDistribuidos–Clase09 15/92

| Módulo, | proceso, | servicio | y microservicio | no son sinónimos |
| ------- | -------- | -------- | --------------- | ---------------- |
VOCABULARIOJUSTOATIEMPO
| Término | Quées |     |     | Ejemplo |
| ------- | ----- | --- | --- | ------- |
Módulo fronteradecódigoenunaaplicación;expone móduloInventariodentrodelmonolito.
unainterfazinternayocultadetalles.
Proceso programaenejecución,conmemoriayfallas una instancia ejecutando el servicio de
|     | propias. |     |     | Inventario. |
| --- | -------- | --- | --- | ----------- |
Servicio capacidad accesible mediante un contrato, API —interfaz explícita de operaciones
normalmenteatravésdered. permitidas—parareservaroconsultaruna
pieza.
Microservicio servicio de una capacidad coherente, con Ranking separado, operado por un equipo
|     | despliegue | y operación | razonablemente | inde- yalimentadoporeventos. |
| --- | ---------- | ----------- | -------------- | ---------------------------- |
pendientes.
SistemasDistribuidos–Clase09 16/92

| Punto | de  | partida: | Asteria | funciona | como una | aplicación |
| ----- | --- | -------- | ------- | -------- | -------- | ---------- |
modular
CASOCONDUCTOR
unaaplicaciónquesepublicacomounaunidad
|        | Cuentas                        | Mercado | Inventario | Pagos                       | Ranking        | Chat           |
| ------ | ------------------------------ | ------- | ---------- | --------------------------- | -------------- | -------------- |
| Lo que | funciona                       | bien    |            | Señales                     | de que empieza | a quedar corto |
| ▶      |                                |         |            | ▶                           |                |                |
|        | llamadasytransaccioneslocales; |         |            | Rankingprovocapicosdecarga; |                |                |
| ▶      |                                |         |            | ▶                           |                |                |
diagnósticoyvueltaatrásunificados; Pagosexigecontrolespropios;
| ▶   |                         |     |     | ▶                     |     |     |
| --- | ----------------------- | --- | --- | --------------------- | --- | --- |
|     | cambioscruzadossimples; |     |     | Combatecreceporzonas; |     |     |
| ▶   |                         |     |     | ▶                     |     |     |
fronterasfácilesderefactorizar. losequipossebloqueanparapublicarcambios.
SistemasDistribuidos–Clase09 17/92

| La escala | de Asteria | cambia | la escena |     |
| --------- | ---------- | ------ | --------- | --- |
CASOCONDUCTOR
| Supuesto | para seguir la historia |     |     |     |
| -------- | ----------------------- | --- | --- | --- |
Sofía ya no compra en un mundo con pocos jugadores. Hay muchos compradores en distintas
regiones, torneos que disparan consultas a Ranking y más ofertas que se publican a la vez.
| Cambian | las cargas |     | La regla sigue | siendo la misma |
| ------- | ---------- | --- | -------------- | --------------- |
Catálogo y Ranking reciben muchas lecturas; Sofía y otro jugador pueden intentar comprar
las compras modifican ofertas y reservas. Esas #17 a la vez. Inventario debe decidir una única
capacidades no crecen al mismo ritmo. reserva válida antes de transferirla.
Conclusión: más usuarios exigen escalar; separar capacidades requiere un límite medido.
SistemasDistribuidos–Clase09 18/92

Primeras respuestas sin separar capacidades
Presiónobservable Respuestadentrodelaaplicación Costoylímite
modular
Mássolicitudes réplicasdelprocesoybalanceode sereplicatodalaaplicación;labase
tráfico puedeseguirsaturada
Muchaslecturas cachédecatálogoyvistasdeRanking invalidaciónylecturasatrasadas;la
comprarevalidalaoferta
Torneosexigentes diferircálculosdeRankingylimitarsu cola,atrasoytrabajoderecuperación;
usoderecursos puedepersistirlacontención
Comprassimultáneas reservacondicionalenlabasede contenciónenlapieza;mediresperay
Inventario conflictos
Criterio: separarsiestasmedidasnoalcanzanparacumplirelp99decompra,aislarlacargaomejorarelCtAieSmOpCoOhNaDsUtaCTOR
producción.
SistemasDistribuidos–Clase09 19/92

| Antes de | separar: | ¿qué problema | concreto | queremos | resolver? |
| -------- | -------- | ------------- | -------- | -------- | --------- |
CRITERIODEDECISIÓN
| Necesidadobservable |     | Límitedeldiseñoactual |     | Evidencia |     |
| ------------------- | --- | --------------------- | --- | --------- | --- |
Cargadiferente eltorneodeRankingvuelvelentalacompra CPU,colayp99porcapacidad
Ritmosdecambio uncambiodeRankingesperalapublicación tiempohastaproducciónycoordina-
|     |     | global |     | ciones |     |
| --- | --- | ------ | --- | ------ | --- |
Aislamientodefallas una falla secundaria afecta el camino de alcancedelincidente
compra
Seguridadoregulación Pagos necesita permisos, auditoría y incidentesycontroles
secretosespecíficos
Responsabilidaddeequipos nadie opera una parte sin pedir permiso a bloqueosyticketscruzados
otros
Una necesidad concreta de separación es una limitación observable que la modularidad interna ya no resuelve
suficientemente. Sinproblema,evidenciaehipótesisdemejora,distribuiressoloagregarcomplejidad.
SistemasDistribuidos–Clase09 20/92

| La analogía | del edificio: | mapeo explícito |     |
| ----------- | ------------- | --------------- | --- |
edificio=unaversiónpublicadacomoconjunto
|          | Mercado    | Inventario | Pagos  |
| -------- | ---------- | ---------- | ------ |
|          | módulo     | módulo     | módulo |
| Edificio | Habitación | Puerta     | Pared  |
Aplicaciónquesepublicacomo Móduloconresponsabilidad APIinternapermitida. Reglaqueimpidetocar
| unaunidad. | definida. |     | internalsotablasajenas. |
| ---------- | --------- | --- | ----------------------- |
Compartirunprocesosimplificacomunicaciónytransacciones. Elproblemanoespublicarlosmódulosjuntos,sinoperder
lasfronterasinternasqueseparanresponsabilidadesyregulancómointeractúan.
INTUICIÓN
SistemasDistribuidos–Clase09 21/92

| ¿Qué significa  | separar        | en este      | caso?           |            |
| --------------- | -------------- | ------------ | --------------- | ---------- |
|                 | Mercado        |              | Inventario      | Pagos      |
|                 |                | red+contrato | red+contrato    |            |
|                 | procesoy       |              | autoridadde     | controlesy |
|                 | versiónpropios |              | propiedad       | proveedor  |
| Puede habilitar |                |              | No lo garantiza |            |
▶
publicarversionesporseparado; ▶ unabasecompartidamantieneacoplamiento;
▶
escalarsololaparteexigida; ▶ versionescoordinadaseliminanindependencia;
| ▶   |     |     | ▶   |     |
| --- | --- | --- | --- | --- |
limitarelalcancedeunafalla; contratosfrágilesobliganacambiarjuntos;
| ▶   |     |     | ▶   |     |
| --- | --- | --- | --- | --- |
asignarresponsabilidadcompleta. sinresponsabledeincidentesnohayautonomía
operativa.
Separar es una hipótesis: aceptamos red y fallas parciales para intentar obtener una independencia medible. Debe
comprobarseconevidencia.
NÚCLEO
SistemasDistribuidos–Clase09 22/92

| Predicción | individual: | ¿dónde | pondría | la primera | frontera? |
| ---------- | ----------- | ------ | ------- | ---------- | --------- |
ACTIVIDAD
Elija una capacidad de Asteria que mantendría dentro de la aplicación y una que
| separaría.       | Para cada una escriba: |          |     |     |     |
| ---------------- | ---------------------- | -------- | --- | --- | --- |
| 1. una necesidad | concreta;              |          |     |     |     |
| 2. una autonomía | que espera             | obtener; |     |     |     |
| 3. una falla     | nueva que aparecería.  |          |     |     |     |
Guarde la respuesta: volveremos después de estudiar responsabilidad, comunicación, outbox
y sagas.
SistemasDistribuidos–Clase09 23/92

| Cinco preguntas | antes de | dibujar servicios |     |
| --------------- | -------- | ----------------- | --- |
NÚCLEO
1. ¿Qué 5. ¿Qué
|     | 2. ¿Quiénpuede | 3. ¿Quénecesita | 4. ¿Quéfalla |
| --- | -------------- | --------------- | ------------ |
capacidad independencia
|     | cambiareldato? | respuestaahora? | agregalared? |
| --- | -------------- | --------------- | ------------ |
yreglacontiene? podemosmedir?
SistemasDistribuidos–Clase09 24/92

Primero el significado:
contextos y responsabili-
dades
Sistemas Distribuidos

| Contexto | delimitado: | una misma | palabra | puede nombrar | cosas |
| -------- | ----------- | --------- | ------- | ------------- | ----- |
distintas
NÚCLEO
| Palabra | Contexto | Significadoexactoenesecontexto |     |     |     |
| ------- | -------- | ------------------------------ | --- | --- | --- |
Objeto Catálogo tipodepieza: nombre,rarezayatributosbasedeEspadaSolar
Objeto Inventario instanciaúnica#17: propietario,estadoyversión
Objeto Mercado ofertadeventa: vendedor,precio,monedayestadodepublicación
Objeto Combate arma equipada: estadísticas efectivas, durabilidad y tiempo de
reutilización
Unboundedcontextocontextodelimitadoesunafronteradentrodelacualelvocabulario,elmodeloylasreglas
mantienenunsignificadocoherente.
|     |     | SistemasDistribuidos–Clase09 |     |     | 26/92 |
| --- | --- | ---------------------------- | --- | --- | ----- |

| Por qué | no conviene | un único | objeto universal |     |
| ------- | ----------- | -------- | ---------------- | --- |
NÚCLEO
Catálogo referencia Mercado item_id Inventario vistaautorizada Combate
|     | plantilla | oferta | pieza#17 | armaequipada |
| --- | --------- | ------ | -------- | ------------ |
▶
Catálogo puede cambiar atributos base sin modificar el historial de propiedad.
▶
Mercado puede cerrar una oferta sin borrar la instancia del inventario.
▶ Combate necesita una vista rápida, pero no debe convertirse en dueño de la pieza.
| Traducción | explícita |     |     |     |
| ---------- | --------- | --- | --- | --- |
Los contextos intercambian IDs, comandos y eventos; no comparten silenciosamente un objeto
universal mutable.
SistemasDistribuidos–Clase09 27/92

| Un contexto |                | delimitado | no obliga | a crear       | un       | proceso |
| ----------- | -------------- | ---------- | --------- | ------------- | -------- | ------- |
| Dentro      | de un monolito |            |           | Como servicio | separado |         |
| ▶ módulo    | Inventario;    |            |           | ▶             |          |         |
|             |                |            |           | proceso       | propio;  |         |
| ▶ API       | interna;       |            |           | ▶ API o       | mensajes | de red; |
▶
esquema o espacio de nombres ▶ publicación de versión y escalado
protegido;
propios;
| ▶ pruebas | de  | arquitectura; |     | ▶ datos | bajo autoridad | exclusiva; |
| --------- | --- | ------------- | --- | ------- | -------------- | ---------- |
▶
| transacción |     | local. |     | ▶ nuevas | fallas | parciales. |
| ----------- | --- | ------ | --- | -------- | ------ | ---------- |
Primerosedescubrelafronterasemántica. Laseparaciónfísicasedecidedespués,cuando
existe una necesidad concreta que justifica asumir el costo adicional de distribuir.
NÚCLEO
SistemasDistribuidos–Clase09 28/92

| Responsabilidad: | quién puede | decidir | la verdad sobre | #17 |
| ---------------- | ----------- | ------- | --------------- | --- |
NÚCLEO
Combate
consulta/equipa
| Mercado  |                  | Inventario |     |     |
| -------- | ---------------- | ---------- | --- | --- |
| solicita | únicoquemodifica |            |     |     |
reservar/transferir elpropietario
Auditoría
consume
transferencias
| Regla | Estado |     | Equipo |     |
| ----- | ------ | --- | ------ | --- |
Inventariodefinecuándolapieza SoloInventariomodificaeldato Cambialaregla,observamétricasy
| puedecambiardedueño. | autoritativo. |     | atiendeincidentes. |     |
| -------------------- | ------------- | --- | ------------------ | --- |
SistemasDistribuidos–Clase09 29/92

Sofía y Leo compiten por la misma espada
CASOCONDUCTOR
Sofía
attempt=S Inventario
UnareservaOK
reservacondicional
laotrarechazada
atómicade#17
Leo
attempt=L
La decisión que protege el invariante
Inventario cambia DISPONIBLE a RESERVADA(attempt) solo si #17 todavía está disponible. La
transacción serializa la competencia: una petición obtiene éxito y la otra debe elegir otra pieza.
Timeout: Sofía conserva el ID de intento al reintentar, para evitar otra compra lógica.
SistemasDistribuidos–Clase09 30/92

| Cuando | el invariante | cruza | fronteras | aparece | un protocolo |
| ------ | ------------- | ----- | --------- | ------- | ------------ |
NÚCLEO
|     | Mercado | Inventario | Pagos | Inventario | Mercado |
| --- | ------- | ---------- | ----- | ---------- | ------- |
reserva#17
|               | creaintento |      | autoriza | transfiere | confirma |
| ------------- | ----------- | ---- | -------- | ---------- | -------- |
| Qué significa | protocolo   | aquí |          |            |          |
Secuenciaexplícitademensajes,estadosdurablesyreglasparacontinuar,reintentarorepararcuandolosparticipantesno
compartenunatransacciónyalgunopuedefallar.
| ▶ identidaddecadaintento; |     |     | ▶   | significadodeuntimeout; |     |
| ------------------------- | --- | --- | --- | ----------------------- | --- |
| ▶                         |     |     | ▶   |                         |     |
| autoridaddecadadato;      |     |     |     | políticadereintento;    |     |
▶ estadoqueseguardaantesderesponder; ▶ reparacióndeefectosparciales.
SistemasDistribuidos–Clase09 31/92

| Aplicación | modular | o servi- |
| ---------- | ------- | -------- |
cios separados:
| simplicidad | o independencia |     |
| ----------- | --------------- | --- |
medible
Sistemas Distribuidos

| Monolito  | modular: | una      | solución, no | una etapa vergonzosa  |     |
| --------- | -------- | -------- | ------------ | --------------------- | --- |
| Dimensión |          | Monolito | modular      | Aplicación enmarañada |     |
Dependencias explícitas y verificables cualquier parte llama a cualquier
otra
Datos acceso mediante APIs internas tablas compartidas sin autoridad
Cambios contratos y pruebas internas efectos laterales impredecibles
Responsabilidad responsables claros todos pueden cambiar todo
| Publicación |     | conjunta | por decisión | conjunta porque | nada puede |
| ----------- | --- | -------- | ------------ | --------------- | ---------- |
separarse
Compartir un proceso simplifica comunicación y transacciones. El problema aparece cuando el
código se vuelve una maraña sin límites internos, no por el mero hecho de publicar los módulos
juntos.
NÚCLEO
SistemasDistribuidos–Clase09 33/92

| Cuándo | conviene | mantener | Asteria | modular | y unida |
| ------ | -------- | -------- | ------- | ------- | ------- |
NÚCLEO
▶
| El dominio | todavía | cambia y las | fronteras no | están maduras. |     |
| ---------- | ------- | ------------ | ------------ | -------------- | --- |
▶ El equipo es pequeño y modifica varios módulos en conjunto.
▶ La compra necesita invariantes que caben en una transacción local.
▶
| La carga | de los componentes | todavía | es parecida. |     |     |
| -------- | ------------------ | ------- | ------------ | --- | --- |
▶ El equipo aún no posee herramientas y guardias para operar muchos servicios.
| Ventaja | de esperar |     |     |     |     |
| ------- | ---------- | --- | --- | --- | --- |
Refactorizar una frontera dentro del proceso es más barato que cambiarla cuando ya es una
API de red, un esquema de eventos almacenado y una responsabilidad operativa
independiente.
SistemasDistribuidos–Clase09 34/92

| Qué independencia |         | puede justificar | una      | separación |
| ----------------- | ------- | ---------------- | -------- | ---------- |
| Independencia     | buscada | Evidencia        | concreta |            |
Publicar versiones RankingsaleovuelveatrássincoordinarMercadoniInventario
Escalar Ranking absorbe un torneo sin multiplicar recursos de Pagos
| Aislar fallas |     | una caída | de Chat no interrumpe | la compra |
| ------------- | --- | --------- | --------------------- | --------- |
Evolucionar datos Inventario cambia internals sin acceso directo de otros
componentes
Responsabilidad de equipo un grupo decide cambios y atiende incidentes de la capacidad
Separar no crea independencia automáticamente. Una base compartida, versiones coordinadas o
contratos frágiles pueden dejar el sistema igual de acoplado, pero con red en el medio.
NÚCLEO
SistemasDistribuidos–Clase09 35/92

| Primera | frontera: | separar | Ranking |     |
| ------- | --------- | ------- | ------- | --- |
CASOCONDUCTOR
| Síntoma | observado |     |     |     |
| ------- | --------- | --- | --- | --- |
El torneo carga CPU y base de datos compartidas: el p99 de compra sube. Ranking ya se calcula
después de comprar, pero comparte recursos y versión publicada con Mercado.
| Decisión | y beneficio esperado |     | Costo que | aceptamos |
| -------- | -------------------- | --- | --------- | --------- |
Ranking procesa compras confirmadas. Puede Entrega durable del evento, atraso de la vista,
escalar y publicar versiones sin cargar el flujo de duplicados, reconstrucción, observabilidad y
| compra. |     |     | guardia del | nuevo servicio. |
| ------- | --- | --- | ----------- | --------------- |
Contrato: Mercado publica PurchaseConfirmed con outbox e idempotencia.
SistemasDistribuidos–Clase09 36/92

| Sofía compra | desde | otra región |     |     |
| ------------ | ----- | ----------- | --- | --- |
CASOCONDUCTOR
| Lecturas cercanas | al jugador |     | Escritura bajo | una autoridad |
| ----------------- | ---------- | --- | -------------- | ------------- |
Catálogo y Ranking pueden usar cachés o Antes de confirmar #17, Mercado pide a
réplicas regionales. Sofía ve la oferta con menos Inventario reservarla. Esa operación necesita
latencia, aunque la copia puede estar atrasada. una decisión coordinada y puede sumar tiempo
|     |     |     | de red entre | regiones. |
| --- | --- | --- | ------------ | --------- |
Una oferta visible en caché no garantiza disponibilidad. La reserva autoritativa decide si Sofía
| gana; acercar | lecturas no habilita | dos propietarios. |     |     |
| ------------- | -------------------- | ----------------- | --- | --- |
SistemasDistribuidos–Clase09 37/92

| De llamada | local | a cadena | sincrónica |     |     |
| ---------- | ----- | -------- | ---------- | --- | --- |
NÚCLEO
|     | APIde espera |         | espera     | espera | termina Respuesta |
| --- | ------------ | ------- | ---------- | ------ | ----------------- |
|     |              | Mercado | Inventario | Pagos  |                   |
|     | compra       |         |            |        | aSofía            |
Definición
Unacadenasincrónicaesunflujodondelapeticiónoriginalnopuedeterminarhastarecibirlasrespuestasrequeridasde
variasdependenciassucesivas.
▶
laslatenciasseacumulan;
▶
untimeoutremotoesambiguo;
▶
ladisponibilidaddelflujodependedetodoslosparticipantesrequeridos.
SistemasDistribuidos–Clase09 38/92

El costo adicional de distribuir: qué aparece al cruzar la red
NÚCLEO
| Comunicación | Entrega | Datos | Operación | Seguridad |
| ------------ | ------- | ----- | --------- | --------- |
Contratos
| latenciay | retrye | consistencia | trazasy | identidad |
| --------- | ------ | ------------ | ------- | --------- |
compatibilidad
| timeout | idempotencia | ycopias | guardias | ysecretos |
| ------- | ------------ | ------- | -------- | --------- |
Regla económica
Aceptamosestecostosolocuandoresuelveunanecesidadconcretaylamejorapuedemedirse: menortiempohasta
producción,escalaseparada,menoralcancedeincidentesoreglasmejoraisladas.
Larednoeliminaelacoplamiento: puedetransformarloentimeouts,contratos,duplicadosycoordinaciónentreequipos.
SistemasDistribuidos–Clase09 39/92

Disponibilidad compuesta: una aproximación útil, no una ley
NÚCLEO
Si cuatro dependencias independientes son obligatorias y cada una está disponible el 99.9%:
A ≈0.9994 ≈0.996
flujo
Qué muestra Qué omite
Cada dependencia requerida puede bloquear el Fallas correlacionadas, red, colas, retries,
camino crítico. cachés, degradación y dependencia condicional.
Más procesos no implican más disponibilidad. El objetivo es retirar dependencias no críticas o
permitir degradación, no multiplicar cajas.
SistemasDistribuidos–Clase09 40/92

La ruta crítica de la compra de Sofía
CASOCONDUCTOR
Pasoantesderesponderéxito Resultadorequerido Costosicruzaprocesos
Mercadovalidalaoferta intentoaceptado llamadayestadodurable
Inventarioreserva#17 exclusióndeotroscompradores red,timeoutyposiblecontención
Pagosautoriza respuestadelproveedor latenciaeincertidumbredepago
Inventariotransfiere Sofíaquedapropietaria otrallamadayrecuperaciónsifalla
Mercadoconfirma comprayavisopendientesenun escrituralocalyposiblereintento
COMMITlocal
Costoacumulado: sesumanlasesperasobligatoriasylosreintentos. RankingyNotificacionesquedanfueradelcamino
crítico,conatrasomedible.
SistemasDistribuidos–Clase09 41/92

| Muchos | servicios, | pero una | sola unidad | de cambio |
| ------ | ---------- | -------- | ----------- | --------- |
ANTIPATRÓN
|     |     | Mercado | Inventario Pagos | Ranking |
| --- | --- | ------- | ---------------- | ------- |
mismabase+versionescoordinadas+modelos
compartidos+cadenaRPCobligatoria
| Por qué | se usa la expresión | “monolito | distribuido” |     |
| ------- | ------------------- | --------- | ------------ | --- |
Es una etiqueta crítica y paradójica: físicamente hay varios procesos y red, pero los cambios, datos
y operación siguen comportándose como una única pieza inseparable.
Se pagan latencia, timeouts y observabilidad distribuida sin obtener independencia. El nombre
describe un antipatrón; no propone una arquitectura deseable.
SistemasDistribuidos–Clase09 42/92

| Casos | reales: la | forma depende | de la necesidad |     |     |
| ----- | ---------- | ------------- | --------------- | --- | --- |
CASOSREALES
Shopify: reforzar límites internos Netflix: servicios con plataforma
madura
| Una aplicación | grande puede | mejorar módulos | y   |     |     |
| -------------- | ------------ | --------------- | --- | --- | --- |
dependencias dentro del mismo proceso La separación puede tener sentido cuando
mientras publicar todo junto siga siendo cargas, equipos y ciclos son distintos y existen
| aceptable. |     |     | herramientas   | para publicar, | observar y recuperar |
| ---------- | --- | --- | -------------- | -------------- | -------------------- |
|            |     |     | cada servicio. |                |                      |
Copiar la forma sin compartir la necesidad y la capacidad operativa produce complejidad sin
beneficio. Primero se declara el problema; después se elige la estructura.
Fuente/referencia:ShopifyEngineering;NetflixTechBlog;Fowler,MonolithFirstyMicroserviceTrade-Offs.
SistemasDistribuidos–Clase09 43/92

| Cruzar | la frontera: |           |     |
| ------ | ------------ | --------- | --- |
| RPC,   | comandos,    | consultas | y   |
eventos
| Sistemas Distribuidos |     |     |     |
| --------------------- | --- | --- | --- |

| Llamada | local y RPC | se parecen | en el código, | no en | la falla |
| ------- | ----------- | ---------- | ------------- | ----- | -------- |
NÚCLEO
RPC
RemoteProcedureCallollamadaaprocedimientoremoto: contratoquehaceparecerunainteracciónderedcomouna
llamadaaunafunción. Simplificalainterfaz;noeliminalaincertidumbredistribuida.
| Llamada                 | local |     | RPC                               |     |     |
| ----------------------- | ----- | --- | --------------------------------- | --- | --- |
| ▶                       |       |     | ▶                                 |     |     |
| mismoproceso;           |       |     | serializaciónyred;                |     |     |
| ▶                       |       |     | ▶                                 |     |     |
| errorinmediatoyvisible; |       |     | peticiónorespuestapuedenperderse; |     |     |
| ▶                       |       |     | ▶                                 |     |     |
unapilayunentornodeejecución; elreceptorpuedeejecutaryelemisorobservar
| ▶   |     |     | timeout; |     |     |
| --- | --- | --- | -------- | --- | --- |
transacciónlocalposible.
▶
requiereidentidad,retryyobservabilidad.
|     |     | SistemasDistribuidos–Clase09 |     |     | 45/92 |
| --- | --- | ---------------------------- | --- | --- | ----- |

| Timeout | ambiguo: | ¿Inventario | reservó | la espada? |
| ------- | -------- | ----------- | ------- | ---------- |
EJECUCIÓNADVERSARIAL
envía
|     |     | ReserveItem |     | timeout |
| --- | --- | ----------- | --- | ------- |
Mercado
respuestaperdida
Inventario
reserva#17 envíaOK
durable
| Lo único | que Mercado | sabe |     |     |
| -------- | ----------- | ---- | --- | --- |
Norecibiórespuestadentrodesuplazo. NosabesiInventarionorecibió,fallóantesdereservar,reservóyperdióla
respuesta,osiguelento.
Elretrynecesitaunaclaveidempotenteestableyunresultadodurableparanocrearunasegundareservalógica.
SistemasDistribuidos–Clase09 46/92

| Comando, | evento | y consulta: | tres | contratos | distintos |
| -------- | ------ | ----------- | ---- | --------- | --------- |
NÚCLEO
| Mensaje | Semántica |                |             | Ejemplo               | Asteria |
| ------- | --------- | -------------- | ----------- | --------------------- | ------- |
| Comando | pedir     | al responsable | autorita-   | ReserveItem(item_17)  |         |
|         | tivo      | que intente    | una acción; |                       |         |
|         | puede     | rechazar       |             |                       |         |
| Evento  | declarar  | un hecho       | que ya      | ItemReserved(item_17) |         |
ocurrió
| Consulta | pedirinformaciónsincambiarel |              |     | GetItemOwner(item_17) |     |
| -------- | ---------------------------- | ------------ | --- | --------------------- | --- |
|          | estado                       | autoritativo |     |                       |     |
Un nombre en pasado no convierte un comando en evento. La diferencia está en quién decide
| y si el receptor | puede rechazar | la acción. |     |     |     |
| ---------------- | -------------- | ---------- | --- | --- | --- |
SistemasDistribuidos–Clase09 47/92

| Elegir RPC | o evento | desde | la necesidad | del flujo |
| ---------- | -------- | ----- | ------------ | --------- |
NÚCLEO
¿Lapeticiónnecesita
estarespuestaparapoder
terminarcorrectamente?
|     | caminocrítico |     |     | desacoplar |
| --- | ------------- | --- | --- | ---------- |
Sí No
| RPCocomando |     |     |     | eventoyprocesamiento |
| ----------- | --- | --- | --- | -------------------- |
conrespuesta posterior
| Ejemplos sincrónicos |     |     | Ejemplos | asíncronos |
| -------------------- | --- | --- | -------- | ---------- |
reservar la pieza, autorizar pago, consultar una notificación, ranking, analítica y otras vistas
| regla necesaria. |     |     | derivadas. |     |
| ---------------- | --- | --- | ---------- | --- |
SistemasDistribuidos–Clase09 48/92

Necesidad: varios componentes deben enterarse sin bloquear
la compra
PROBLEMA
Ranking
hechopreservado
Mercado
Purchase- Notificaciones
confirmacompra
Confirmed
Auditoría
Problema Necesidad
Mercado no debería esperar a todos los lectores Conservar un aviso que otros puedan leer,
ni conocer cada uso futuro. reintentar y procesar a su propio ritmo.
SistemasDistribuidos–Clase09 49/92

| Intuición: | un evento | es un aviso | durable | sobre un | hecho |
| ---------- | --------- | ----------- | ------- | -------- | ----- |
INTUICIÓN
Ejemplo
PurchaseConfirmed(purchase_id=42, item=17, buyer=Sofía)informaqueMercadoyaconfirmólacompratras
conocerlatransferencia. Describepasadoconfirmado;noordenaunaacciónfutura.
| Qué permite                |     |     | Qué exige         |     |     |
| -------------------------- | --- | --- | ----------------- | --- | --- |
| ▶                          |     |     | ▶                 |     |     |
| lectoresindependientes;    |     |     | identidadestable; |     |     |
| ▶ replaydespuésdeunacaída; |     |     | ▶                 |     |     |
significadoinequívoco;
| ▶                                   |     |     | ▶                  |     |     |
| ----------------------------------- | --- | --- | ------------------ | --- | --- |
| nuevasvistassinmodificarInventario. |     |     | formatocompatible; |     |     |
▶
políticadeduplicadosyorden.
Sielmensajepretendequeundestinatariohagaalgo,esuncomando,aunqueviajeporunbroker.
SistemasDistribuidos–Clase09 50/92

| Contrato | de evento: | la interfaz | pública | que queda | almacenada |
| -------- | ---------- | ----------- | ------- | --------- | ---------- |
FORMALIZACIÓN
Definición
Elcontratodeuneventoespecificaquéhechorepresenta,quécampostransporta,cómoseidentifica,quéordenpuede
suponerseycómoevolucionasinromperconsumidores.
| Parte     |     | EnPurchaseConfirmed                   |     |     |     |
| --------- | --- | ------------------------------------- | --- | --- | --- |
| Identidad |     | event_id,purchase_idyversióndelevento |     |     |     |
Semántica Mercadoconfirmólacompratraslatransferencia;noesunaintenciónpendiente
| Esquema |     | nombres,tiposyobligatoriedaddecampos |     |     |     |
| ------- | --- | ------------------------------------ | --- | --- | --- |
Evolución reglasparaagregaroretirarcampossinromperlectoresviejos
Entrega posibilidaddeduplicado,ordenrelevanteporcomprayreplay
Se parece a una API porque otros componentes dependen de esa interfaz. La diferencia es que queda almacenada:
consumidoresnuevosoatrasadosleeneventosproducidosporversionesanteriores.
SistemasDistribuidos–Clase09 51/92

| Ventajas | y costos | de publicar | un contrato | de evento |     |
| -------- | -------- | ----------- | ----------- | --------- | --- |
TRADE-OFF
| Ventajas |     |     | Costos |     |     |
| -------- | --- | --- | ------ | --- | --- |
▶
| ▶ desacopla | productor                | y lectores; | duplicados       | y atraso;     |     |
| ----------- | ------------------------ | ----------- | ---------------- | ------------- | --- |
| ▶           |                          |             | ▶ compatibilidad | durante años; |     |
| permite     | replay y reconstrucción; |             |                  |               |     |
▶
▶ agrega lectores sin cambiar Inventario; observabilidad de lag y fallas;
| ▶      |                     |            | ▶ mala semántica | se propaga | a todos. |
| ------ | ------------------- | ---------- | ---------------- | ---------- | -------- |
| retira | efectos secundarios | del camino |                  |            |          |
crítico.
Identidad: cuándodosregistrosrepresentanelmismohecho. Semántica: quéafirma. Evolución:
| cómo cambia | el formato | sin romper la historia | almacenada. |     |     |
| ----------- | ---------- | ---------------------- | ----------- | --- | --- |
SistemasDistribuidos–Clase09 52/92

Preservar integridad:
| dos escrituras, | outbox | e   |
| --------------- | ------ | --- |
idempotencia
Sistemas Distribuidos

Dual write: una intención, dos sistemas sin transacción común
NÚCLEO
La necesidad
Basededatos
Mercadodebe: compra
1. confirmarlacompraensubasededatos(DB); CONFIRMADA
2. publicarPurchaseConfirmedparaRankingy
Notificaciones. Mercado
Broker
Purchase
Confirmed
NoexisteunúnicoCOMMITqueabarqueautomáticamentelabaselocalyelbrokerremoto.
SistemasDistribuidos–Clase09 54/92

| Dos órdenes, | dos ventanas | de falla |
| ------------ | ------------ | -------- |
EJECUCIÓNADVERSARIAL
| Base de datos | primero | Broker primero |
| ------------- | ------- | -------------- |
1. confirmarcompraenlabase; 1. publicarevento;
2. crash; 2. crashoreversióndelatransacciónlocal;
3. eventonuncapublicado. 3. labasenoconfirmalacompra.
Resultado: compraautoritativasinnotificaciónniranking. Resultado: consumidoresreaccionanaunacompra
inexistente.
Cambiarelordennoeliminalaventana: solocambiaquémitadquedasola.
SistemasDistribuidos–Clase09 55/92

Intuición de outbox: dejar el aviso en la misma operación local
guardarcompra
CONFIRMADA
Transacciónlocal
deMercado
guardaraviso
pendiente
enoutbox
Imagen mental
DespuésdeconocerlatransferenciadeInventario, Mercadoanotalacompraconfirmadayelavisoque
deberá enviar. Si la transacción local confirma, existen ambos; si aborta, no existe ninguno.
Laoutboxevitaolvidarlaintencióndepublicarsielprocesocaedespuésdelcommitlocal. Todavía
hace falta un proceso que publique el aviso.
INTUICIÓN
SistemasDistribuidos–Clase09 56/92

| Definición | formal: | transactional | outbox |     |
| ---------- | ------- | ------------- | ------ | --- |
FORMALIZACIÓN
Definición
Una transactional outbox es una tabla o colección persistente donde el servicio inserta un evento
pendiente dentro de la misma transacción local que modifica el estado de negocio.
Relay
|     | Mercado     | filaoutbox |                   | Broker        |
| --- | ----------- | ---------- | ----------------- | ------------- |
|     | COMMITlocal |            | publica/reintenta |               |
|     |             | PENDING    |                   | eventodurable |
▶
| el relay         | puede leer por | polling o CDC;    |     |     |
| ---------------- | -------------- | ----------------- | --- | --- |
| ▶ la publicación | suele ser      | al menos una vez; |     |     |
▶
el consumidor deduplica mediante event_id o inbox transaccional.
SistemasDistribuidos–Clase09 57/92

| Qué resuelve | outbox | y qué deja | abierto |     |
| ------------ | ------ | ---------- | ------- | --- |
TRADE-OFF
| Resuelve |     |     | No resuelve | sola |
| -------- | --- | --- | ----------- | ---- |
| ▶        |     |     | ▶           |      |
estado sin intención durable de publicar; duplicados del relay;
▶ crash entre commit local y registro del ▶ COMMIT global con Inventario o Pagos;
aviso;
|                |             |            | ▶ consumidores | no idempotentes; |
| -------------- | ----------- | ---------- | -------------- | ---------------- |
| ▶ recuperación | del trabajo | pendiente. |                |                  |
▶
|     |     |     | atraso | y crecimiento de la tabla. |
| --- | --- | --- | ------ | -------------------------- |
Métricas mínimas: filas pendientes, edad de la más antigua, reintentos y tiempo extremo a
| extremo hasta | el consumidor. |     |     |     |
| ------------- | -------------- | --- | --- | --- |
SistemasDistribuidos–Clase09 58/92

| Sofía | ve éxito; | Ranking | todavía | puede estar | atrasado |
| ----- | --------- | ------- | ------- | ----------- | -------- |
CASOCONDUCTOR
1. Inventario confirma la transferencia. Mercado guarda CONFIRMADA y PurchaseConfirmed
| pendiente | en un | mismo COMMIT | local. |     |     |
| --------- | ----- | ------------ | ------ | --- | --- |
2. Sofía recibe la confirmación basada en ese estado autoritativo. El relay aún puede estar
| detenido | y Ranking | mostrar la | puntuación | anterior. |     |
| -------- | --------- | ---------- | ---------- | --------- | --- |
3. El relay publica o reintenta; Ranking deduplica por event_id y actualiza su proyección.
| Beneficio |     |     |     | Costo observable |     |
| --------- | --- | --- | --- | ---------------- | --- |
Una falla de Ranking no borra la compra Medir edad de la outbox, atraso de Ranking,
confirmada y la proyección puede recuperarse. duplicados y tiempo de reconstrucción.
SistemasDistribuidos–Clase09 59/92

Coordinar la compra:
saga y reparación
Sistemas Distribuidos

| Tres autoridades | independientes |     | participan | en la compra |
| ---------------- | -------------- | --- | ---------- | ------------ |
VOCABULARIOJUSTOATIEMPO
| Autoridad | Hecho | que decide | Por | qué es independiente |
| --------- | ----- | ---------- | --- | -------------------- |
Mercado intento y estado de la oferta posee su estado, transacción y fallas
Inventario reserva y propietario de #17 confirmaensupropiabaseyreinicia
aparte
Pagos autorizaciónorechazodelpago depende de un proveedor y no
|                       |          |     | comparte | COMMIT con Inventario |
| --------------------- | -------- | --- | -------- | --------------------- |
| Independencia en esta | historia |     |          |                       |
Cada autoridad puede confirmar, fallar, reintentar o recuperarse sin que las otras participen de la
misma transacción atómica. Ninguna puede deshacer por sí sola todo el proceso.
SistemasDistribuidos–Clase09 61/92

| Por | qué aparece |     | una | saga | en Asteria |     |
| --- | ----------- | --- | --- | ---- | ---------- | --- |
NECESIDAD
Problema
La compra atraviesa Mercado, Inventario y Pagos. Cada autoridad confirma localmente, pero no
| existe un | COMMIT | único | que | abarque | a las tres. |     |
| --------- | ------ | ----- | --- | ------- | ----------- | --- |
Inventario
|     | Mercado |     |     | Inventario |     | Pagos |
| --- | ------- | --- | --- | ---------- | --- | ----- |
propiedad
|     | intentocreado |     |     | piezareservada |     | pagoautorizado |
| --- | ------------- | --- | --- | -------------- | --- | -------------- |
transferida
Necesitamos conservar el estado del proceso y decidir qué hacer si un paso tarda, se duplica,
| queda | incierto | o falla | después | de que | otros ya confirmaron. |     |
| ----- | -------- | ------- | ------- | ------ | --------------------- | --- |
SistemasDistribuidos–Clase09 62/92

| Intuición: | una | saga es | un itinerario | durable | de negocio |
| ---------- | --- | ------- | ------------- | ------- | ---------- |
INTUICIÓN
| Imagen | mental |     |     |     |     |
| ------ | ------ | --- | --- | --- | --- |
La compra es un viaje con varias paradas. En cada parada se anota durablemente dónde quedó, qué
| respuesta | recibió y cuál | es el próximo | paso permitido. |             |            |
| --------- | -------------- | ------------- | --------------- | ----------- | ---------- |
|           |                |               | PAGO            | PROPIEDAD   |            |
|           | CREADA         | RESERVADA     |                 |             | COMPLETADA |
|           |                |               | AUTORIZADO      | TRANSFERIDA |            |
Si el proceso reinicia, no vuelve a empezar a ciegas: lee el estado durable y continúa, consulta
| o repara | desde allí. |     |     |     |     |
| -------- | ----------- | --- | --- | --- | --- |
SistemasDistribuidos–Clase09 63/92

| Definición | formal: | saga |     |     |
| ---------- | ------- | ---- | --- | --- |
FORMALIZACIÓN
Definición
Una saga coordina una operación mediante transacciones locales. Conserva estados durables y
define reintentos, consultas, timeouts y compensaciones para resultados parciales.
| Registra |            |     | Decide       |     |
| -------- | ---------- | --- | ------------ | --- |
| ▶        |            |     | ▶ continuar; |     |
| saga_id  | e intento; |     |              |     |
▶
| ▶ estado | y deadline;        |     | consultar o        | reintentar; |
| -------- | ------------------ | --- | ------------------ | ----------- |
| ▶        |                    |     | ▶                  |             |
| comandos | y respuestas;      |     | compensar;         |             |
| ▶        |                    |     | ▶ pasar a revisión | manual.     |
| último   | resultado durable. |     |                    |             |
SistemasDistribuidos–Clase09 64/92

| Camino | feliz: qué | confirma | cada participante |     |
| ------ | ---------- | -------- | ----------------- | --- |
NÚCLEO
3. Inventario
|     | 1. Inventario | 2. Pagos |     | 4. Mercado |
| --- | ------------- | -------- | --- | ---------- |
transfiere
|     | reserva#17 | autorizacargo |     | confirmacompra |
| --- | ---------- | ------------- | --- | -------------- |
propiedad
| Pregunta | previa |     |     |     |
| -------- | ------ | --- | --- | --- |
¿En qué estado puede responderse “compra confirmada” y qué evidencia durable demuestra
| que la pieza | y el cargo quedaron | coherentes? |     |     |
| ------------ | ------------------- | ----------- | --- | --- |
SistemasDistribuidos–Clase09 65/92

| Falla realista: | el pago | puede quedar | incierto |
| --------------- | ------- | ------------ | -------- |
EJECUCIÓNADVERSARIAL
Authorize
|     | payment |     | timeout |
| --- | ------- | --- | ------- |
Saga
respuestaperdida
Pagos
|     |     | cargo | envíaOK |
| --- | --- | ----- | ------- |
autorizado
Reintentar ciegamente puede cobrar dos veces. La saga necesita un payment_attempt_id, consulta de estado y un
estadodurablePAYMENT_UNKNOWN.
SistemasDistribuidos–Clase09 66/92

Falla posterior: pago autorizado, transferencia imposible
NÚCLEO
Transferencia
ReservaOK AutorizaciónOK
FALLA
Anular
Liberarreserva
autorización
Estado global
La saga pasa a COMPENSATING; solo termina cuando conoce el resultado durable de cada
compensación o deriva el caso a intervención manual. Si el cargo ya se capturó, puede
hacer falta un reembolso.
SistemasDistribuidos–Clase09 67/92

Tres formas de “volver atrás” que no son equivalentes
VOCABULARIOJUSTOATIEMPO
Mecanismo Quéhace Ejemplo
Rollbacktransaccional abortacambioslocalesaúnnoconfirmados la base revierte una transacción sin
COMMIT
Vueltaatrásdeversión devuelve tráfico o software a una versión se retira una versión defectuosa de
anterior Ranking
Compensacióndenegocio ejecuta una nueva acción para reparar un anularautorizaciónyliberarlareserva
efectoconfirmado
Volverunaversióndesoftwarenoborrapagos,eventosnicambiosdepropiedadyaproducidos. Esosefectosrequieren
accionesexplícitas.
SistemasDistribuidos–Clase09 68/92

| Compensación: | reparar | el negocio | sin borrar | la historia |
| ------------- | ------- | ---------- | ---------- | ----------- |
NÚCLEO
Definición
Una compensación es una nueva acción de negocio que intenta neutralizar o reparar un efecto ya
confirmado. También puede fallar, duplicarse o requerir intervención humana.
Transferencia
| Pagoautorizado |     |     | Anularautorización | Liberarreserva |
| -------------- | --- | --- | ------------------ | -------------- |
imposible
El historial conserva autorización, falla, anulación solicitada y resultado. Si el pago se capturó,
| se solicita reembolso | en vez de | anular la autorización. |     |     |
| --------------------- | --------- | ----------------------- | --- | --- |
SistemasDistribuidos–Clase09 69/92

| Intuición: | coreografía | y orquestación |     |
| ---------- | ----------- | -------------- | --- |
VOCABULARIOJUSTOATIEMPO
| Coreografía |     |     | Orquestación |
| ----------- | --- | --- | ------------ |
Como bailarines que conocen las señales: cada Como una dirección de escena: un coordinador
participante observa un hecho y decide su conserva el estado del proceso y envía la
| próximo movimiento, | sin un director | que | próxima instrucción. |
| ------------------- | --------------- | --- | -------------------- |
| conserve todo       | el plan.        |     |                      |
Son dos formas de coordinar la misma saga. Ninguna elimina estados durables, idempotencia,
| timeouts ni | reparación. |     |     |
| ----------- | ----------- | --- | --- |
SistemasDistribuidos–Clase09 70/92

| Coreografía | en Asteria: | cada servicio | reacciona | a hechos |
| ----------- | ----------- | ------------- | --------- | -------- |
NÚCLEO
|              | Inventario              | Pagos      |            |                |
| ------------ | ----------------------- | ---------- | ---------- | -------------- |
|              |                         |            | Inventario | Mercado        |
|              | publica publicaPayment- |            |            |                |
|              |                         |            | transfiere | completacompra |
| ItemReserved |                         | Authorized |            |                |
| Ventaja      |                         | Riesgo     |            |                |
Participantespocoacoplados;nuevoslectoresseagregan Elprocesoglobalquedarepartidoentrecontratos;ciclosy
| confacilidad. |     | fallassondifícilesdevisualizar. |     |     |
| ------------- | --- | ------------------------------- | --- | --- |
SistemasDistribuidos–Clase09 71/92

Orquestación en Asteria: un coordinador conserva el proceso
NÚCLEO
Inventario
Coordinador
decompra
Pagos
estadodurable
delasaga
Mercado
Ventaja Riesgo
Unlugarvisibleparaestado,deadlines,reintentos, Puedeconcentrardemasiadalógica;coordina,peronodebe
compensacioneseintervenciónhumana. apropiarsedelosdatosdeotros.
SistemasDistribuidos–Clase09 72/92

| Coreografía | u orquestación: |     | elegir por | el proceso |
| ----------- | --------------- | --- | ---------- | ---------- |
TRADE-OFF
| Dimensión |     | Coreografía |     | Orquestación |
| --------- | --- | ----------- | --- | ------------ |
Próximopaso cadaservicioreaccionaahechos coordinadorenvíacomandos
| Estadoglobal |     | repartido |     | visibleenelcoordinador |
| ------------ | --- | --------- | --- | ---------------------- |
Cambiodeflujo revisarvarioscontratos modificarelprocesocoordinador
| Fallaslargas/humanas |     | difícilesdeseguir |     | fácilesdesupervisar |
| -------------------- | --- | ----------------- | --- | ------------------- |
Riesgo ciclosylógicaemergente coordinadorcondemasiadaresponsabil-
idad
Para la compra de #17, la orquestación suele ser más clara cuando hay pago incierto, compensaciones y revisión
manual;RankingyNotificacionespuedencontinuarcoreografiados.
SistemasDistribuidos–Clase09 73/92

| Migrar y operar: |      |            |     |
| ---------------- | ---- | ---------- | --- |
| la frontera      | debe | sobrevivir | en  |
producción
Sistemas Distribuidos

| ¿Qué extraer       | primero      | en Asteria? |             |              |          |
| ------------------ | ------------ | ----------- | ----------- | ------------ | -------- |
| Ranking: candidato | inicial      |             | Inventario: | extracción   | riesgosa |
| ▶ carga diferente  | y con picos; |             | ▶ preserva  | propiedad    | única;   |
| ▶ tolera atraso    | acotado;     |             | ▶ participa | en el camino | crítico; |
▶ puede construirse como proyección; ▶ genera sagas y estados intermedios;
▶ falla sin detener la compra; ▶ exige idempotencia y operación madura;
| ▶   |     |     | ▶   |     |     |
| --- | --- | --- | --- | --- | --- |
escala y despliega de forma distinta. una falla puede duplicar o bloquear objetos.
La primera frontera debe ofrecer beneficio y aprendizaje con riesgo controlable.
NÚCLEO
SistemasDistribuidos–Clase09 75/92

| Migración | gradual: | patrón | Strangler | Fig |
| --------- | -------- | ------ | --------- | --- |
VOCABULARIOJUSTOATIEMPO
Intuición
En vez de reemplazar toda la aplicación en una fecha única, una parte nueva crece al costado de la
anterior. El tráfico se desvía por porciones hasta retirar el camino viejo.
Definición
El patrón Strangler Fig es una migración incremental donde el componente antiguo y el nuevo
| coexisten y | una fachada decide | qué camino | utilizar. |     |
| ----------- | ------------------ | ---------- | --------- | --- |
No significa simplemente “crear un microservicio”. Es un método para reducir riesgo: coexistir,
| comparar, | desviar y retirar. |     |     |     |
| --------- | ------------------ | --- | --- | --- |
SistemasDistribuidos–Clase09 76/92

| Prueba | gradual | y vuelta | atrás: limitar | el riesgo |
| ------ | ------- | -------- | -------------- | --------- |
VOCABULARIOJUSTOATIEMPO
| Prueba | canary |     | Vuelta | atrás de versión |
| ------ | ------ | --- | ------ | ---------------- |
Una fracción pequeña del tráfico real usa la Si las métricas empeoran, el router devuelve el
versión nueva. Se comparan errores, latencia y tráfico al camino anterior. No deshace efectos
de negocio ya confirmados.
| resultados | antes de ampliar. |     |     |     |
| ---------- | ----------------- | --- | --- | --- |
La migración es reversible mientras ambos caminos coexistentes puedan recibir tráfico y los
| datos | se mantengan | comparables. |     |     |
| ----- | ------------ | ------------ | --- | --- |
SistemasDistribuidos–Clase09 77/92

| Migración | gradual | aplicada | a Ranking |
| --------- | ------- | -------- | --------- |
NÚCLEO
Eventosde
Rankingdel
|     | comprasy |     | monolito |
| --- | -------- | --- | -------- |
combate API/router
eligelectura
Nuevoservicio
deRanking
| 1. alimentar | ambos           | cálculos;                 |             |
| ------------ | --------------- | ------------------------- | ----------- |
| 2. comparar  | exactitud,      | atraso y costo;           |             |
| 3. enviar    | una fracción    | pequeña al camino         | nuevo;      |
| 4. volver    | al anterior     | si las métricas empeoran; |             |
| 5. retirar   | el código viejo | después de evidencia      | suficiente. |
SistemasDistribuidos–Clase09 78/92

Balance de decisiones en Asteria
Decisión Beneficioesperado Quécuestasostener Cómocomprobar
Réplicasycaché lecturasmásrápidas recursos,invalidación,oferta p99yaciertosdecaché
potencialmentevieja
Rankingseparado compramenosexpuestaal outbox,broker,atraso,duplicados, p99,lageincidentes
torneo otraguardia
SiPagossesepara controlesycambiosdepago RPC,timeoutambiguo,IDs fallasdepagoyauditoría
aislados idempotentes
SiInventariosesepara escalayversiónpropias llamadascríticas,sagay conflictos,latenciay
recuperacióndereservas comprascorrectas
CASOCONDUCTOR
Decisión: probarRankingprimero. Inventarioquedaunidomientrassucostodistribuidosupereelbeneficiomedido.
SistemasDistribuidos–Clase09 79/92

| Vocabulario | operativo | I: CI, CD | y preparación | operativa |
| ----------- | --------- | --------- | ------------- | --------- |
VOCABULARIOJUSTOATIEMPO
| Término | Definicióndetrabajo |     |     |     |
| ------- | ------------------- | --- | --- | --- |
CI—IntegraciónContinua integrar cambios con frecuencia y ejecutar compilación, pruebas y controles
automáticos
CD—EntregaContinua mantener cada cambio validado listo para producción; la promoción puede ser
manual
CD—DespliegueContinuo ponerautomáticamente enproduccióntodo cambioque superaloscontroles; hay
queaclararquésignificadoseusa
Preparación operativa evidencia de que un equipo puede publicar, observar, recuperar y sostener una
| (readiness) | capacidadenproducción |     |     |     |
| ----------- | --------------------- | --- | --- | --- |
CIyCDautomatizanelcaminodelcambio;nosustituyencontratoscompatibles,recuperación,métricasniresponsables
deincidentes.
SistemasDistribuidos–Clase09 80/92

Vocabulario operativo II: guardia, procedimiento, SLO y trazas
VOCABULARIOJUSTOATIEMPO
Término Definicióndetrabajo
Guardia(guardia) rotaciónresponsablederecibiralertasycoordinarlarespuestaaincidentes
Procedimiento(runbook) pasosescritosparadiagnosticar,mitigaryrecuperarunafallarepetible
SLI indicadormedido,porejemplocomprascorrectasolatenciap99
SLO metacuantificadaparaunSLIduranteunaventana,porejemplo99,9%decompras
correctaspormes
Trazasdistribuidas correlacióndelrecorridodeunapeticiónoeventoatravésdevariosprocesos
Estascapacidadeshacenvisiblequiénoperalafrontera,quépromesaintentacumplirycómoinvestigarunafallaentre
servicios.
SistemasDistribuidos–Clase09 81/92

| Preparación | operativa: | separar | implica | sostener | otro sistema |
| ----------- | ---------- | ------- | ------- | -------- | ------------ |
Definición
Existe preparación operativa cuando el equipo demuestra que puede publicar, observar y recuperar la
capacidad en producción; no alcanza con que un chequeo /ready responda.
| ▶ CI y     | publicación de versiones; |     | ▶ SLO     | y alertas;        |     |
| ---------- | ------------------------- | --- | --------- | ----------------- | --- |
| ▶ vuelta   | atrás probada;            |     | ▶ guardia | y procedimientos; |     |
| ▶ permisos | y secretos;               |     | ▶ backup  | y recuperación;   |     |
| ▶ logs,    | métricas y trazas;        |     | ▶ pruebas | de falla;         |     |
| ▶          |                           |     | ▶         |                   |     |
| retries    | e idempotencia;           |     | capacidad | y costos.         |     |
Sin esta preparación, separar puede demorar cambios y reducir confiabilidad aunque el diagrama
| parezca | moderno. |     |     |     |     |
| ------- | -------- | --- | --- | --- | --- |
NÚCLEO
SistemasDistribuidos–Clase09 82/92

| Cómo leer | las métricas | de autonomía |
| --------- | ------------ | ------------ |
VOCABULARIOJUSTOATIEMPO
| Métrica | Quésignificayquépreguntaresponde |     |
| ------- | -------------------------------- | --- |
Tiempo hasta producción desde que un cambio está listo hasta que funciona: ¿la frontera acelera el
| (tiempohastaproducción) | cambio? |     |
| ----------------------- | ------- | --- |
Publicacionessincoordinación versionesquesalensinsincronizarotrosequipos
Escalaindependiente recursosagregadossoloalacapacidadquecrece
Alcance del incidente (alcance usuarios,regiones,datosycapacidadesafectados
delincidente)
p99 valorpordebajodelcualquedael99%deobservaciones; muestralacolade
latencia
Coordinacionesentreequipos aprobaciones,ticketsybloqueosnecesariosparacambiaruoperar
SistemasDistribuidos–Clase09 83/92

| Métricas | para | comprobar | que la separación | valió | la pena |
| -------- | ---- | --------- | ----------------- | ----- | ------- |
NÚCLEO
| Métrica |     | Quémide |     | Quéesperamos |     |
| ------- | --- | ------- | --- | ------------ | --- |
Tiempohastaproducción cambiolistoaversiónfuncionando bajarsinaumentarincidentes
Publicaciones sin versionesquenoesperanaotrosequipos aumentar
coordinación
Escalaporcapacidad recursossolodondecrececarga evitarescalartodoAsteria
| Alcancedelincidente    |     | usuariosycapacidadesafectados |     | reducir      |     |
| ---------------------- | --- | ----------------------------- | --- | ------------ | --- |
| p99yerrores            |     | experienciadelacolayfallas    |     | cumplirelSLO |     |
| Coordinacionescruzadas |     | trabajomanualentreequipos     |     | reducir      |     |
Hacefaltaunalíneabaseanterior. Sielnuevoserviciosiguecompartiendobase,versionesyguardiasconlaaplicación,
laindependenciaprometidanosematerializó.
SistemasDistribuidos–Clase09 84/92

| Actividad: | defender | una frontera | de Asteria |     |
| ---------- | -------- | ------------ | ---------- | --- |
ACTIVIDAD
| Elijan una | capacidad |     | Completen | el argumento |
| ---------- | --------- | --- | --------- | ------------ |
▶
| inventario; |     |     | 1. regla | e invariante; |
| ----------- | --- | --- | -------- | ------------- |
▶
| pagos;     |     |     | 2. responsable | y dato autoritativo; |
| ---------- | --- | --- | -------------- | -------------------- |
| ▶ ranking; |     |     | 3. alternativa | simple;              |
▶
| chat;     |           |     | 4. frontera    | propuesta;      |
| --------- | --------- | --- | -------------- | --------------- |
| ▶ combate | por zona. |     | 5. RPC/evento; |                 |
|           |           |     | 6. falla       | y recuperación; |
|           |           |     | 7. métrica     | de éxito.       |
SistemasDistribuidos–Clase09 85/92

| Método | reusable | para decidir | una frontera |     |     |
| ------ | -------- | ------------ | ------------ | --- | --- |
NÚCLEO
1. Problema 2. Regla,dato 3. Alternativa 4. Costo,falla 5. Métrica
|           | yevidencia | yautoridad | mássimple | ymecanismo | yrevisión |
| --------- | ---------- | ---------- | --------- | ---------- | --------- |
| Marco del | curso      |            |           |            |           |
D=(P,F,G,S,T): latecnologíaaparecedespuésdedeclararproblema,fallas,garantíasycostoaceptado.
SistemasDistribuidos–Clase09 86/92

| Cierre comparativo: | por qué              | distribuir          | y cuándo | no hacerlo |
| ------------------- | -------------------- | ------------------- | -------- | ---------- |
| Capacidad           | Límitedeldiseñounido | Separaciónpropuesta |          | Prueba     |
Ranking picosafectanlacompra proyección por eventos; versión p99, atraso y tiempo
|     |     | propia |     | hastaproducción |
| --- | --- | ------ | --- | --------------- |
Pagos controles y proveedor frenan serviciopropiosiexisteoperación errores, auditoría y
|     | cambios | madura |     | coordinaciones |
| --- | ------- | ------ | --- | -------------- |
Inventario la transacción local protege mantener unido hasta justificar duplicaciones, laten-
|     | propiedad | unasaga |     | ciaycarga |
| --- | --------- | ------- | --- | --------- |
La aplicación unida deja de alcanzar cuando una necesidad medible persiste y no se resuelve con modularidad interna.
Distribuiresaceptaruncostopararesolverla;silasmétricasnomejoran,ladecisióndeberevisarse.
SÍNTESISDEDECISIÓN
SistemasDistribuidos–Clase09 87/92

| Volvemos | a   | la  | predicción | inicial |
| -------- | --- | --- | ---------- | ------- |
ACTIVIDAD
| Revise su | primera | respuesta | y complete: |     |
| --------- | ------- | --------- | ----------- | --- |
1. ¿La capacidad elegida tiene un lenguaje e invariante coherentes?
| 2. ¿Existe | un responsable |          | técnico       | y operacional? |
| ---------- | -------------- | -------- | ------------- | -------------- |
| 3. ¿Qué    | autonomía      | concreta | se            | obtendría?     |
| 4. ¿Qué    | falla nueva    | aparece  | al            | cruzar la red? |
| 5. ¿Cómo   | se preserva    |          | la integridad | del flujo?     |
| 6. ¿Qué    | métrica        | podría   | refutar       | su decisión?   |
Cambiar de opinión es evidencia de aprendizaje si ahora puede justificar mejor la frontera.
SistemasDistribuidos–Clase09 88/92

Volvemos al diagnóstico inicial
ACTIVIDAD
1. Falso: un monolito puede imponer módulos, APIs y responsabilidades claras.
2. Falso: un microservicio no es una clase remota; representa una capacidad con autonomía
operativa.
3. Falso: servicios separados pueden seguir acoplados por base de datos, contratos y versiones
coordinadas.
4. Falso: la base de datos y el broker son dos escrituras; una caída entre ambas deja
inconsistencia.
5. Falso: compensar crea un nuevo efecto; las acciones previas siguen formando parte de la
historia.
SistemasDistribuidos–Clase09 89/92

| Síntesis: | una frontera | debe | justificar | su costo |     |
| --------- | ------------ | ---- | ---------- | -------- | --- |
NÚCLEO
|     | Necesidad  | Regla,dato | Contrato      | Falla         | Mejora |
| --- | ---------- | ---------- | ------------- | ------------- | ------ |
|     | observable | yautoridad | ycomunicación | yrecuperación | medida |
Distribuirnoesprogresoporsímismo. Unafronteravalecuandoresuelveunanecesidadconcreta,lamejoraaparece
enmétricasyelequipopuedeoperarlasfallasnuevas.
SistemasDistribuidos–Clase09 90/92

Exit ticket
ACTIVIDAD
| Responda | brevemente: |     |     |     |
| -------- | ----------- | --- | --- | --- |
1. ¿Qué diferencia una frontera semántica de una frontera de red?
| 2. ¿Qué | autonomía     | concreta | justificaría | separar Ranking?       |
| ------- | ------------- | -------- | ------------ | ---------------------- |
| 3. ¿Qué | sabe Mercado  | después  | de un        | timeout de Inventario? |
| 4. ¿Qué | resuelve      | outbox y | qué deja     | abierto?               |
| 5. ¿Por | qué compensar | no       | equivale a   | rollback?              |
SistemasDistribuidos–Clase09 91/92

| Fuentes | para profundizar |     |     |
| ------- | ---------------- | --- | --- |
NÚCLEO
▶ Kleppmann, Designing Data-Intensive Applications: servicios, RPC, transacciones, streams y
| sistemas | derivados. |     |     |
| -------- | ---------- | --- | --- |
▶ Evans, Domain-Driven Design: bounded contexts y lenguaje ubicuo.
▶
Fowler, Monolith First, Microservice Trade-Offs y Strangler Fig.
▶
Shopify Engineering: modular monolith, componentización y límites internos.
| ▶ AWS Prescriptive | Guidance: | transactional | outbox y sagas. |
| ------------------ | --------- | ------------- | --------------- |
▶
Richardson, Microservices Patterns: sagas, contratos y datos por servicio.
▶ DORA, Accelerate State of DevOps: tiempo hasta producción, frecuencia de puesta en
| producción | y confiabilidad | operacional. |     |
| ---------- | --------------- | ------------ | --- |
SistemasDistribuidos–Clase09 92/92