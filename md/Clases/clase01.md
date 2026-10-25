| Sistemas       | Distribuidos     |              |             |
| -------------- | ---------------- | ------------ | ----------- |
| Clase          | 01 — Pensamiento |              | distribuido |
| Departamento   | de               | Computación  |             |
| Ingeniería     | Informática      | – Comisión   | A – Pilar   |
| Equipo docente | –                | 11 de agosto | de 2026     |

Presentación
Sistemas Distribuidos

Plantel docente
▶ Mariano Benitez
▶ Tomás Najun
▶ Rodolfo Sumoza
▶ Alejandro Furfaro
SistemasDistribuidos–Clase01 3/35

Entrada en calor
Sistemas Distribuidos

| Un problema |           |          | aparentemente |       | simple         |
| ----------- | --------- | -------- | ------------- | ----- | -------------- |
| Un usuario  | hace      | clic     | en “pagar”.   |       |                |
| ▶ El        | cliente   | envía    | la solicitud. |       |                |
| ▶ La        | respuesta | no       | llega después | de un | tiempo.        |
| Para pensar |           | mientras | arrancamos    |       |                |
| ¿Qué pasó?  | ¿Qué      | puedo    | saber?        | ¿Qué  | es importante? |
SistemasDistribuidos–Clase01 5/35

Presentación
Sistemas Distribuidos

Contenido
▶ Presentación de la asignatura.
▶ Objetivos de la asignatura.
▶ Contenido de la asignatura.
▶ Plantel docente.
▶ Esquema de evaluación.
SistemasDistribuidos–Clase01 7/35

Presentación
Vamos a hablar de sistemas distribuídos,
donde múltiples componentes trabajan en conjunto.
Vamos a analizar problemas y posibles soluciones
de acuerdo a las restricciones y capacidades deseadas.
SistemasDistribuidos–Clase01 8/35

Objetivos
Conocimientos
▶ Conocer técnicas, algoritmos, diseños e implementaciones de sistemas distribuidos
modernos.
▶ Distinguir características para la toma de decisión en cuanto al dominio.
Habilidades
▶ Relevar contextos de sistemas distribuídos.
▶ Analizar esos contextos e identificar características.
▶ Proponer y defender alternativas para resolver esos contextos.
SistemasDistribuidos–Clase01 9/35

Contenido
▶ Pensamiento distribuido: fallas parciales, incertidumbre, observación incompleta y
trade-offs.
▶ Comunicación: RPC, timeouts, retries, idempotencia y semánticas de entrega.
▶ Estado replicado: consistencia, replicación, quorums, conflictos y convergencia.
▶ Coordinación: consenso, elección de líder, logs replicados y límites de disponibilidad.
▶ Transacciones y almacenamiento: serializabilidad, aislamiento, commit distribuido,
WAL, MVCC e índices.
▶ Arquitecturas distribuidas: eventos, streaming, sagas, colas, microservicios y
resiliencia.
▶ Operación y diseño: observabilidad, performance, escalabilidad, incidentes y
decisiones de arquitectura.
SistemasDistribuidos–Clase01 10/35

| Esquema | de evaluación |     |     |
| ------- | ------------- | --- | --- |
▶
| Parcial   | con recuperatorio | al              | final de la cursada;    |
| --------- | ----------------- | --------------- | ----------------------- |
| ▶ Trabajo | práctico          | por grupo,      | con defensa individual; |
| ▶ Nota    | cursada >=        | 6 => promociona |                         |
SistemasDistribuidos–Clase01 11/35

Retomamos
Sistemas Distribuidos

Pregunta
¿El pago se realizó o no?
Respuesta correcta
No lo sabemos. Y, en general, no podemos saberlo con certeza.
| Un problema |           | aparentemente | simple |
| ----------- | --------- | ------------- | ------ |
| Un usuario  | hace clic | en “pagar”.   |        |
▶
| El   | cliente envía | la solicitud. |               |
| ---- | ------------- | ------------- | ------------- |
| ▶ La | respuesta no  | llega después | de un tiempo. |
SistemasDistribuidos–Clase01 13/35

Respuesta correcta
No lo sabemos. Y, en general, no podemos saberlo con certeza.
| Un problema |           | aparentemente | simple |
| ----------- | --------- | ------------- | ------ |
| Un usuario  | hace clic | en “pagar”.   |        |
▶
| El   | cliente envía | la solicitud. |               |
| ---- | ------------- | ------------- | ------------- |
| ▶ La | respuesta no  | llega después | de un tiempo. |
Pregunta
| ¿El pago | se realizó o | no? |     |
| -------- | ------------ | --- | --- |
SistemasDistribuidos–Clase01 13/35

| Un problema |           | aparentemente | simple |
| ----------- | --------- | ------------- | ------ |
| Un usuario  | hace clic | en “pagar”.   |        |
▶
| El   | cliente envía | la solicitud. |               |
| ---- | ------------- | ------------- | ------------- |
| ▶ La | respuesta no  | llega después | de un tiempo. |
Pregunta
| ¿El pago  | se realizó o | no? |     |
| --------- | ------------ | --- | --- |
| Respuesta | correcta     |     |     |
No lo sabemos. Y, en general, no podemos saberlo con certeza.
SistemasDistribuidos–Clase01 13/35

Lo único observable (en principio) para el cliente es la falta de confirmación en un cierto
intervalo.
Visualización del problema
solicitud de pago
Cliente Servidor
respuesta perdida / demorada
SistemasDistribuidos–Clase01 14/35

Visualización del problema
solicitud de pago
Cliente Servidor
respuesta perdida / demorada
Lo único observable (en principio) para el cliente es la falta de confirmación en un cierto
intervalo.
SistemasDistribuidos–Clase01 14/35

Idea clave
|                |           |             |            |     | Para el cliente, | ambas historias | son indistinguibles. |
| -------------- | --------- | ----------- | ---------- | --- | ---------------- | --------------- | -------------------- |
| Dos historias, | una misma | observación |            |     |                  |                 |                      |
| Historia A     |           |             | Historia B |     |                  |                 |                      |
| ▶              |           |             | ▶          |     |                  |                 |                      |
El servidor nunca recibió la solicitud; El servidor sí procesó el pago;
| ▶            |                  |     | ▶            |                  |     |     |     |
| ------------ | ---------------- | --- | ------------ | ---------------- | --- | --- | --- |
| no hubo      | procesamiento;   |     | la respuesta | se perdió;       |     |     |     |
| ▶ el cliente | observa timeout. |     | ▶ el cliente | observa timeout. |     |     |     |
SistemasDistribuidos–Clase01 15/35

| Dos historias, | una | misma observación |            |     |
| -------------- | --- | ----------------- | ---------- | --- |
| Historia A     |     |                   | Historia B |     |
| ▶              |     |                   | ▶          |     |
El servidor nunca recibió la solicitud; El servidor sí procesó el pago;
| ▶            |                  |     | ▶            |                  |
| ------------ | ---------------- | --- | ------------ | ---------------- |
| no hubo      | procesamiento;   |     | la respuesta | se perdió;       |
| ▶ el cliente | observa timeout. |     | ▶ el cliente | observa timeout. |
Idea clave
| Para el cliente, | ambas historias | son indistinguibles. |     |     |
| ---------------- | --------------- | -------------------- | --- | --- |
SistemasDistribuidos–Clase01 15/35

¿Qué afirmaciones pueden hacer? ¿Qué propuestas pueden surgir?
|     |     |     |     |     | ¿Pudieron | separar información | de análisis | de propuestas? |
| --- | --- | --- | --- | --- | --------- | ------------------- | ----------- | -------------- |
Debate
| Tormenta  | de ideas   | / No juzgamos     |                 |           |     |     |     |     |
| --------- | ---------- | ----------------- | --------------- | --------- | --- | --- | --- | --- |
| Enfocados | en el caso | como se presentó, | los escuchamos: |           |     |     |     |     |
|           |            | ¿Qué preguntas    | pudieron        | formular? |     |     |     |     |
SistemasDistribuidos–Clase01 16/35

|     |     |     |     |     | ¿Pudieron | separar información | de análisis | de propuestas? |
| --- | --- | --- | --- | --- | --------- | ------------------- | ----------- | -------------- |
Debate
| Tormenta  | de ideas   | / No juzgamos     |                 |           |     |     |     |     |
| --------- | ---------- | ----------------- | --------------- | --------- | --- | --- | --- | --- |
| Enfocados | en el caso | como se presentó, | los escuchamos: |           |     |     |     |     |
|           |            | ¿Qué preguntas    | pudieron        | formular? |     |     |     |     |
¿Qué afirmaciones pueden hacer? ¿Qué propuestas pueden surgir?
SistemasDistribuidos–Clase01 16/35

Debate
| Tormenta  | de ideas   | / No juzgamos     |                 |           |     |
| --------- | ---------- | ----------------- | --------------- | --------- | --- |
| Enfocados | en el caso | como se presentó, | los escuchamos: |           |     |
|           |            | ¿Qué preguntas    | pudieron        | formular? |     |
¿Qué afirmaciones pueden hacer? ¿Qué propuestas pueden surgir?
|     | ¿Pudieron | separar información |     | de análisis | de propuestas? |
| --- | --------- | ------------------- | --- | ----------- | -------------- |
SistemasDistribuidos–Clase01 16/35

El enfoque
| Estructurar | apropiadamente | una         | propuesta | y defenderla  | es una |
| ----------- | -------------- | ----------- | --------- | ------------- | ------ |
|             | habilidad      | fundamental | para      | un ingeniero. |        |
Planteamos un marco sencillo para organizar una presentación.
SistemasDistribuidos–Clase01 17/35

| Relevar, | analizar, | proponer |     |          |
| -------- | --------- | -------- | --- | -------- |
| Relevar  |           | Analizar |     | Proponer |
▶ Síntomas observables; ▶ Causas compatibles; ▶ Garantía buscada;
▶ contexto y restricciones; ▶ evidencia disponible; ▶ solución elegida;
| ▶     |             |     | ▶                      | ▶                   |
| ----- | ----------- | --- | ---------------------- | ------------------- |
| datos | que faltan. |     | hipótesis descartadas. | trade-off aceptado. |
Primero se separa cada dimensión para no confundir observación, inferencia y decisión.
| Después | se conectan | para justificar | una propuesta coherente. |     |
| ------- | ----------- | --------------- | ------------------------ | --- |
SistemasDistribuidos–Clase01 18/35

La ruptura conceptual
Sistemas Distribuidos

El problema central no es solo técnico. Es epistemológico: el sistema no entrega
|         |                |           | conocimiento | perfecto sobre | su propio estado. |
| ------- | -------------- | --------- | ------------ | -------------- | ----------------- |
| La idea | más importante | del curso |              |                |                   |
Afirmación
En sistemas distribuidos, un observador remoto no puede distinguir perfectamente entre:
▶
| Una   | falla real en el destino; |        |     |     |     |
| ----- | ------------------------- | ------ | --- | --- | --- |
| ▶ Una | falla en la comunicación; |        |     |     |     |
| ▶ una | demora suficientemente    | larga. |     |     |     |
SistemasDistribuidos–Clase01 20/35

| La idea | más importante | del curso |
| ------- | -------------- | --------- |
Afirmación
En sistemas distribuidos, un observador remoto no puede distinguir perfectamente entre:
▶
| Una   | falla real en el destino; |        |
| ----- | ------------------------- | ------ |
| ▶ Una | falla en la comunicación; |        |
| ▶ una | demora suficientemente    | larga. |
El problema central no es solo técnico. Es epistemológico: el sistema no entrega
| conocimiento | perfecto sobre | su propio estado. |
| ------------ | -------------- | ----------------- |
SistemasDistribuidos–Clase01 20/35

|     |     |     |     | Cambio de | paradigma |     |
| --- | --- | --- | --- | --------- | --------- | --- |
El objetivo deja de ser “adivinar el estado real” y pasa a ser “garantizar un comportamiento
|               |               |                      |           | razonable bajo | información | incompleta”. |
| ------------- | ------------- | -------------------- | --------- | -------------- | ----------- | ------------ |
| Qué significa | esto          | en la práctica       |           |                |             |              |
| Si no podés   | saber con     | certeza qué pasó,    | entonces: |                |             |              |
| ▶ No          | podés basarte | en una visión global | perfecta; |                |             |              |
▶
no podés exigir soluciones que dependan de información imposible;
| ▶ tenés | que diseñar | bajo incertidumbre. |     |     |     |     |
| ------- | ----------- | ------------------- | --- | --- | --- | --- |
SistemasDistribuidos–Clase01 21/35

| Qué significa |               | esto        | en la práctica    |           |
| ------------- | ------------- | ----------- | ----------------- | --------- |
| Si no podés   | saber         | con certeza | qué pasó,         | entonces: |
| ▶ No          | podés basarte | en          | una visión global | perfecta; |
▶
no podés exigir soluciones que dependan de información imposible;
| ▶ tenés | que diseñar  | bajo | incertidumbre. |     |
| ------- | ------------ | ---- | -------------- | --- |
| Cambio  | de paradigma |      |                |     |
El objetivo deja de ser “adivinar el estado real” y pasa a ser “garantizar un comportamiento
| razonable | bajo información |     | incompleta”. |     |
| --------- | ---------------- | --- | ------------ | --- |
SistemasDistribuidos–Clase01 21/35

| Qué | vuelve | distribuido | a un |
| --- | ------ | ----------- | ---- |
problema
| Sistemas | Distribuidos |     |     |
| -------- | ------------ | --- | --- |

Consecuencias:
|     |     |     | ▶ La comunicación | no es instantánea; |     |
| --- | --- | --- | ----------------- | ------------------ | --- |
▶
|            |           |     | las fallas son    | parciales;        |              |
| ---------- | --------- | --- | ----------------- | ----------------- | ------------ |
|            |           |     | ▶ el conocimiento | del estado global | es limitado. |
| Definición | operativa |     |                   |                   |              |
Un sistema distribuido es un conjunto de procesos o nodos que:
| ▶ Se ejecutan  | en máquinas     | distintas;   |     |     |     |
| -------------- | --------------- | ------------ | --- | --- | --- |
| ▶ se comunican | por intercambio | de mensajes; |     |     |     |
▶
cooperan para ofrecer una abstracción que el usuario percibe como un solo sistema.
SistemasDistribuidos–Clase01 23/35

| Definición | operativa |     |
| ---------- | --------- | --- |
Un sistema distribuido es un conjunto de procesos o nodos que:
| ▶ Se ejecutan  | en máquinas     | distintas;   |
| -------------- | --------------- | ------------ |
| ▶ se comunican | por intercambio | de mensajes; |
▶
cooperan para ofrecer una abstracción que el usuario percibe como un solo sistema.
Consecuencias:
| ▶ La comunicación | no es | instantánea; |
| ----------------- | ----- | ------------ |
▶
| las fallas        | son parciales; |                     |
| ----------------- | -------------- | ------------------- |
| ▶ el conocimiento | del estado     | global es limitado. |
SistemasDistribuidos–Clase01 23/35

|     |     |     | Lo importante | es esto: |     |     |
| --- | --- | --- | ------------- | -------- | --- | --- |
El problema se vuelve distribuido cuando la corrección depende de coordinar componentes
|            |                   |           | que no comparten | memoria, reloj | ni observación | perfecta. |
| ---------- | ----------------- | --------- | ---------------- | -------------- | -------------- | --------- |
| Qué no     | alcanza           | con decir |                  |                |                |           |
| No alcanza | con afirmar:      |           |                  |                |                |           |
| ▶ “Hay     | varias máquinas”; |           |                  |                |                |           |
| ▶ “hay     | red”;             |           |                  |                |                |           |
▶
| “hay | paralelismo”. |     |     |     |     |     |
| ---- | ------------- | --- | --- | --- | --- | --- |
SistemasDistribuidos–Clase01 24/35

| Qué no     | alcanza           | con decir |     |
| ---------- | ----------------- | --------- | --- |
| No alcanza | con afirmar:      |           |     |
| ▶ “Hay     | varias máquinas”; |           |     |
| ▶ “hay     | red”;             |           |     |
▶
| “hay          | paralelismo”. |     |     |
| ------------- | ------------- | --- | --- |
| Lo importante | es esto:      |     |     |
El problema se vuelve distribuido cuando la corrección depende de coordinar componentes
| que no comparten | memoria, | reloj ni observación | perfecta. |
| ---------------- | -------- | -------------------- | --------- |
SistemasDistribuidos–Clase01 24/35

|     |     |     |     |     | Modelo | asíncrono: |     |     |
| --- | --- | --- | --- | --- | ------ | ---------- | --- | --- |
▶
|     |     |     |     |     | No existen | cotas confiables | conocidas; |     |
| --- | --- | --- | --- | --- | ---------- | ---------------- | ---------- | --- |
▶
una demora arbitrariamente grande es indistinguible de una falla.
|        |          |          |           |     | Internet real | se parece mucho | más al segundo | caso. |
| ------ | -------- | -------- | --------- | --- | ------------- | --------------- | -------------- | ----- |
| Modelo | síncrono | y modelo | asíncrono |     |               |                 |                |       |
Modelo síncrono:
▶
| Existen | cotas conocidas | para    | red y cómputo;      |             |     |     |     |     |
| ------- | --------------- | ------- | ------------------- | ----------- | --- | --- | --- | --- |
| ▶ una   | demora fuera    | de cota | puede interpretarse | como falla. |     |     |     |     |
SistemasDistribuidos–Clase01 25/35

|        |          |          |           | Internet real | se parece mucho | más al segundo | caso. |
| ------ | -------- | -------- | --------- | ------------- | --------------- | -------------- | ----- |
| Modelo | síncrono | y modelo | asíncrono |               |                 |                |       |
Modelo síncrono:
▶
| Existen | cotas conocidas | para red      | y cómputo;                |     |     |     |     |
| ------- | --------------- | ------------- | ------------------------- | --- | --- | --- | --- |
| ▶ una   | demora fuera    | de cota puede | interpretarse como falla. |     |     |     |     |
Modelo asíncrono:
▶
| No existen | cotas | confiables conocidas; |     |     |     |     |     |
| ---------- | ----- | --------------------- | --- | --- | --- | --- | --- |
▶
una demora arbitrariamente grande es indistinguible de una falla.
SistemasDistribuidos–Clase01 25/35

| Modelo | síncrono  | y   | modelo | asíncrono |     |
| ------ | --------- | --- | ------ | --------- | --- |
| Modelo | síncrono: |     |        |           |     |
▶
| Existen | cotas conocidas |         | para red | y cómputo;    |             |
| ------- | --------------- | ------- | -------- | ------------- | ----------- |
| ▶ una   | demora fuera    | de cota | puede    | interpretarse | como falla. |
| Modelo  | asíncrono:      |         |          |               |             |
▶
| No existen | cotas | confiables | conocidas; |     |     |
| ---------- | ----- | ---------- | ---------- | --- | --- |
▶
una demora arbitrariamente grande es indistinguible de una falla.
| Internet | real se parece | mucho | más al | segundo | caso. |
| -------- | -------------- | ----- | ------ | ------- | ----- |
SistemasDistribuidos–Clase01 25/35

| Cómo | pensaremos | el resto | del |
| ---- | ---------- | -------- | --- |
curso
| Sistemas Distribuidos |     |     |     |
| --------------------- | --- | --- | --- |

| Framework         | del curso     |         |                 |
| ----------------- | ------------- | ------- | --------------- |
| Vamos a organizar | toda decisión | técnica | con el esquema: |
D = (P,F,G,S,T)
▶
P: problema;
| ▶ F: fallas asumidas; |            |     |     |
| --------------------- | ---------- | --- | --- |
| ▶ G: garantía         | requerida; |     |     |
▶
| S: solución    | elegida;  |     |     |
| -------------- | --------- | --- | --- |
| ▶ T: trade-off | aceptado. |     |     |
SistemasDistribuidos–Clase01 27/35

| Ejemplo      | con el           | framework |     |
| ------------ | ---------------- | --------- | --- |
| Caso: evitar | pagos duplicados |           |     |
▶
| P: pago      | remoto con       | timeout;     |              |
| ------------ | ---------------- | ------------ | ------------ |
| ▶ F: pérdida | de respuesta,    | delay, retry | del cliente; |
| ▶ G: no      | más de un débito | efectivo;    |              |
▶
S: request ID único + de-duplicación + persistencia del resultado;
| ▶ T: mayor | complejidad | y más estado. |     |
| ---------- | ----------- | ------------- | --- |
SistemasDistribuidos–Clase01 28/35

Es saber:
▶
Qué información no tenés;
▶ qué falla estás modelando;
▶ qué garantía necesitás realmente;
▶
qué costo aceptás pagar por ella.
| Qué significa pensar     | distributivamente |             |
| ------------------------ | ----------------- | ----------- |
| Pensar distributivamente | no es memorizar   | algoritmos. |
SistemasDistribuidos–Clase01 29/35

| Qué significa pensar     | distributivamente |             |
| ------------------------ | ----------------- | ----------- |
| Pensar distributivamente | no es memorizar   | algoritmos. |
Es saber:
▶
| Qué información              | no tenés;  |     |
| ---------------------------- | ---------- | --- |
| ▶ qué falla estás modelando; |            |     |
| ▶ qué garantía necesitás     | realmente; |     |
▶
| qué costo aceptás | pagar por ella. |     |
| ----------------- | --------------- | --- |
SistemasDistribuidos–Clase01 29/35

| CAP: | el teorema | bien | enten- |
| ---- | ---------- | ---- | ------ |
dido
| Sistemas Distribuidos |     |     |     |
| --------------------- | --- | --- | --- |

Las particiones son una falla que el sistema debe tolerar (son un hecho de la vida), la
|               |          |     | decisión real | es entre consistencia | y disponibilidad. |
| ------------- | -------- | --- | ------------- | --------------------- | ----------------- |
| CAP: en pocas | palabras |     |               |                       |                   |
En un sistema distribuido que replica estado, no se puede garantizar al mismo tiempo:
▶ Consistencia: todas las respuestas observan una única versión correcta del estado;
| ▶ Disponibilidad: | toda operación | recibe respuesta; |     |     |     |
| ----------------- | -------------- | ----------------- | --- | --- | --- |
▶
Tolerancia a particiones: el sistema sigue operando aunque haya nodos
incomunicados.
SistemasDistribuidos–Clase01 31/35

| CAP: en | pocas palabras |     |
| ------- | -------------- | --- |
En un sistema distribuido que replica estado, no se puede garantizar al mismo tiempo:
▶ Consistencia: todas las respuestas observan una única versión correcta del estado;
| ▶ Disponibilidad: | toda operación | recibe respuesta; |
| ----------------- | -------------- | ----------------- |
▶
Tolerancia a particiones: el sistema sigue operando aunque haya nodos
incomunicados.
Las particiones son una falla que el sistema debe tolerar (son un hecho de la vida), la
| decisión real | es entre consistencia | y disponibilidad. |
| ------------- | --------------------- | ----------------- |
SistemasDistribuidos–Clase01 31/35

| CAP: | particiones |     |     |     |
| ---- | ----------- | --- | --- | --- |
Una partición ocurre cuando dos partes del sistema siguen vivas, pero no pueden
| comunicarse | entre sí           | de manera confiable. |            |           |
| ----------- | ------------------ | -------------------- | ---------- | --------- |
| ▶ Los       | nodos no comparten | una visión           |            |           |
| única       | del estado;        |                      | sin enlace |           |
|             |                    |                      | Réplica A  | Réplica B |
| ▶ cada      | lado puede         | recibir pedidos de   |            |           |
clientes;
| ▶ no hay | coordinación | inmediata para | Cliente 1 | Cliente 2 |
| -------- | ------------ | -------------- | --------- | --------- |
| decidir  | globalmente. |                |           |           |
SistemasDistribuidos–Clase01 32/35

Particiones
Consistencia Disponibilidad
| CAP: La | decisión | real |     |
| ------- | -------- | ---- | --- |
Ante partición:
| ▶ O rechazás | operaciones | para preservar | corrección; |
| ------------ | ----------- | -------------- | ----------- |
▶ o respondés igual para preservar disponibilidad, aceptando posibles inconsistencias.
SistemasDistribuidos–Clase01 33/35

| CAP: La | decisión | real |     |
| ------- | -------- | ---- | --- |
Ante partición:
| ▶ O rechazás | operaciones | para preservar | corrección; |
| ------------ | ----------- | -------------- | ----------- |
▶ o respondés igual para preservar disponibilidad, aceptando posibles inconsistencias.
Particiones
|     |     | Consistencia | Disponibilidad |
| --- | --- | ------------ | -------------- |
SistemasDistribuidos–Clase01 33/35

| Errores | conceptuales            | frecuentes |                 |
| ------- | ----------------------- | ---------- | --------------- |
| ▶ Creer | que timeout implica     | crash;     |                 |
| ▶ creer | que la red es confiable | salvo      | casos extremos; |
▶
| asumir | una visión global | perfecta | del sistema; |
| ------ | ----------------- | -------- | ------------ |
▶ tratar CAP como slogan y no como restricción bajo partición.
| ▶ ¿Algún | otro? |     |     |
| -------- | ----- | --- | --- |
SistemasDistribuidos–Clase01 34/35

| No se trata | de encontrar | una solución | perfecta, sino | de: |
| ----------- | ------------ | ------------ | -------------- | --- |
▶
| Explicitar   | supuestos;   |        |         |     |
| ------------ | ------------ | ------ | ------- | --- |
| ▶ elegir     | garantías;   |        |         |     |
| ▶ y defender | el trade-off | que se | acepta. |     |
Cierre
Idea final
El pensamiento distribuido consiste en decidir correctamente bajo incertidumbre.
SistemasDistribuidos–Clase01 35/35

Cierre
Idea final
El pensamiento distribuido consiste en decidir correctamente bajo incertidumbre.
| No se trata | de encontrar | una solución | perfecta, sino | de: |
| ----------- | ------------ | ------------ | -------------- | --- |
▶
| Explicitar   | supuestos;   |        |         |     |
| ------------ | ------------ | ------ | ------- | --- |
| ▶ elegir     | garantías;   |        |         |     |
| ▶ y defender | el trade-off | que se | acepta. |     |
SistemasDistribuidos–Clase01 35/35