# Clase 6

Camino critico

al meno los pasos que debo seguir, para al menos un contrato y una falla, llegue

hasta aqui y me salve, no decir “puse la plata y el alfajor no cayo” para levantar un

like y decir llegue al punto seguro

Camino critico de una escritura

secuencia de pasos que debe completarse antes de enviar la confirmacion al

cliente.

Arbol B y LSM-tree: dos formas de organizar datos

arbol B: son arboles binarios que tienen una busqueda eficiente porque los datos
estan organizados de un lado los mayores y luego los menores entonces esta

balanceado, y en vez de recorrer todo el almacenamiento, es mas rapida la
busqueda.

pero no es gratis porque tengo que lidear el costo con los indices

Vocabulario inicial: compactacion, trabajo pendiente y pausa de
escritura

compactacion: proceso de fondo que fusiona archivos ordenados, reduce
solapamiento y elimina versiones obsoletas cuando ya es seguro hacerlo

trabajo pendiente (backlog): cantidad de bytes o archivos que esperan ser
compactados

regulacion de escrituras: mecanismo de admision que retrasa, bloquea o limita
nuevas solicitudes para que la cola interna no crezca sin limita

Cinco responsabilidades, no cinco terminos aislados

RESPONSABILIDAD

PREGUNTA CONCRETA

MECANISMO QUE

ESTUDIAMOS

Clase 61

---

Registrar decisiones

Controlar concurrencia

WAL y recuperacion

MVCC e instantaneas

logicas

pagina sucia (dirty page): hago modificacion en una pagina, y hasta que no hace

flush o va a la pagina persistente, queda marcada como che “ha sido modificada

pero aun no ha sido transladada, no ha sido registrada esa modificacion”

cuando alguien dice “ya esta escrito” que quiere decir? donde termino la

operacion? de que falla me puedo recuperar?

caida abrupta (crash)

recovery: procedimiento de reinicio

REDO: si se hizo el cambio y dado ese cambio, quiero volver al estado anterior

punto de control (checkpoint): pone una cota

version

snapshot: fortografia del estado, del proceso de los datos en un momento
determinado

vista coherente

serializacion

snapshot vs checkpoint: snapshot no es un momento donde pienso que la data
esta estable y que todo es consistente.

SQL

DDL

Planificador

EXPLAIN

Clase 62

---

EXPLAIN ANALYSE

WAL: (registro) cambios + COMMIT

cuando uso arbol B vs cuando uso LMS

Clase 63
