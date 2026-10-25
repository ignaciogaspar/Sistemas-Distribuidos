| Sistemas       | Distribuidos           |             |           |           |
| -------------- | ---------------------- | ----------- | --------- | --------- |
| Clase          | 03 — Consistencia,     | replicación | y modelos | de estado |
| Departamento   | de Computación         |             |           |           |
| Ingeniería     | Informática – Comisión | A – Pilar   |           |           |
| Equipo docente | – 11 de agosto         | de 2026     |           |           |

Del RPC al estado
Sistemas Distribuidos

Nuevo problema
¿Cómo mantenemos estado cuando hay múltiples copias?
De la comunicación al estado
Clase 2:
▶ RPC bajo incertidumbre
▶ timeouts, retries, idempotencia
SistemasDistribuidos–Clase03 3/26

| De la comunicación | al  | estado |     |
| ------------------ | --- | ------ | --- |
Clase 2:
| ▶ RPC bajo incertidumbre |              |     |     |
| ------------------------ | ------------ | --- | --- |
| ▶ timeouts, retries,     | idempotencia |     |     |
Nuevo problema
| ¿Cómo mantenemos | estado cuando | hay múltiples | copias? |
| ---------------- | ------------- | ------------- | ------- |
SistemasDistribuidos–Clase03 3/26

Motivaciones:
▶ disponibilidad
| ▶ latencia | (lecturas cercanas) |     |
| ---------- | ------------------- | --- |
▶
| tolerancia | a fallas |     |
| ---------- | -------- | --- |
Hecho estructural
| Sin coordinación | perfecta, | las réplicas divergen. |
| ---------------- | --------- | ---------------------- |
Replicación
Replicación = múltiples nodos mantienen el mismo estado lógico.
SistemasDistribuidos–Clase03 4/26

Hecho estructural
Sin coordinación perfecta, las réplicas divergen.
Replicación
Replicación = múltiples nodos mantienen el mismo estado lógico.
Motivaciones:
▶ disponibilidad
▶ latencia (lecturas cercanas)
▶ tolerancia a fallas
SistemasDistribuidos–Clase03 4/26

Replicación
Replicación = múltiples nodos mantienen el mismo estado lógico.
Motivaciones:
▶ disponibilidad
| ▶ latencia | (lecturas cercanas) |     |
| ---------- | ------------------- | --- |
▶
| tolerancia | a fallas |     |
| ---------- | -------- | --- |
Hecho estructural
| Sin coordinación | perfecta, | las réplicas divergen. |
| ---------------- | --------- | ---------------------- |
SistemasDistribuidos–Clase03 4/26

Conclusión
La divergencia no es un bug: es una consecuencia del modelo.
Por qué divergen
▶ latencias variables
▶ pérdidas/reintentos
▶ escrituras concurrentes
▶ particiones
SistemasDistribuidos–Clase03 5/26

| Por qué     | divergen  |     |     |
| ----------- | --------- | --- | --- |
| ▶ latencias | variables |     |     |
▶ pérdidas/reintentos
▶
| escrituras | concurrentes |     |     |
| ---------- | ------------ | --- | --- |
▶
particiones
Conclusión
| La divergencia | no es un bug: | es una consecuencia | del modelo. |
| -------------- | ------------- | ------------------- | ----------- |
SistemasDistribuidos–Clase03 5/26

Modelos de consistencia
Sistemas Distribuidos

| ¿Qué         | es consistencia? |              |                 |
| ------------ | ---------------- | ------------ | --------------- |
| Consistencia | define:          |              |                 |
| qué valores  | pueden observar  | los clientes | y en qué orden. |
SistemasDistribuidos–Clase03 7/26

Intuición
Cada operación ocurre en un punto único entre invocación y respuesta.
| Linearizabilidad |     | (consistencia | fuerte) |
| ---------------- | --- | ------------- | ------- |
▶
| equivalente | a una sola      | copia |     |
| ----------- | --------------- | ----- | --- |
| ▶ respeta   | orden de tiempo | real  |     |
SistemasDistribuidos–Clase03 8/26

| Linearizabilidad |     | (consistencia | fuerte) |
| ---------------- | --- | ------------- | ------- |
▶
| equivalente | a una sola      | copia |     |
| ----------- | --------------- | ----- | --- |
| ▶ respeta   | orden de tiempo | real  |     |
Intuición
Cada operación ocurre en un punto único entre invocación y respuesta.
SistemasDistribuidos–Clase03 8/26

|            |                  |     | Consistencia causal  |           |                 |        |
| ---------- | ---------------- | --- | -------------------- | --------- | --------------- | ------ |
|            |                  |     | Respeta dependencias | causales, | no impone orden | total. |
| Causalidad | (happens-before) |     |                      |           |                 |        |
→
| Relación a | b si:       |                    |     |     |     |     |
| ---------- | ----------- | ------------------ | --- | --- | --- | --- |
| ▶ mismo    | proceso y a | ocurre antes que b |     |     |     |     |
| ▶ a envía  | mensaje que | causa b            |     |     |     |     |
▶
| la relación | se propaga | por transitividad |     |     |     |     |
| ----------- | ---------- | ----------------- | --- | --- | --- | --- |
SistemasDistribuidos–Clase03 9/26

| Causalidad | (happens-before) |     |     |     |
| ---------- | ---------------- | --- | --- | --- |
→
| Relación a | b si:       |              |       |     |
| ---------- | ----------- | ------------ | ----- | --- |
| ▶ mismo    | proceso y a | ocurre antes | que b |     |
| ▶ a envía  | mensaje que | causa b      |       |     |
▶
| la relación          | se propaga | por transitividad |              |        |
| -------------------- | ---------- | ----------------- | ------------ | ------ |
| Consistencia         | causal     |                   |              |        |
| Respeta dependencias | causales,  | no                | impone orden | total. |
SistemasDistribuidos–Clase03 9/26

|          |            |                    |                | Pregunta    | correcta     |                |         |
| -------- | ---------- | ------------------ | -------------- | ----------- | ------------ | -------------- | ------- |
|          |            |                    |                | ¿Qué evento | pudo influir | sobre qué otro | evento? |
| Tiempo   | físico no  | alcanza            |                |             |              |                |         |
| Comparar | timestamps | de nodos distintos | puede engañar: |             |              |                |         |
▶
| relojes  | locales derivan | o se corrigen;     |     |     |     |     |     |
| -------- | --------------- | ------------------ | --- | --- | --- | --- | --- |
| ▶ la red | reordena o      | demora mensajes;   |     |     |     |     |     |
| ▶ una    | hora menor no   | prueba causalidad. |     |     |     |     |     |
SistemasDistribuidos–Clase03 10/26

| Tiempo   | físico     |     | no alcanza |           |                |
| -------- | ---------- | --- | ---------- | --------- | -------------- |
| Comparar | timestamps |     | de nodos   | distintos | puede engañar: |
▶
| relojes     | locales    | derivan | o         | se corrigen; |         |
| ----------- | ---------- | ------- | --------- | ------------ | ------- |
| ▶ la red    | reordena   | o       | demora    | mensajes;    |         |
| ▶ una       | hora menor |         | no prueba | causalidad.  |         |
| Pregunta    | correcta   |         |           |              |         |
| ¿Qué evento | pudo       | influir | sobre     | qué otro     | evento? |
SistemasDistribuidos–Clase03 10/26

a → b =⇒ L(a) < L(b)
Límite
|              |            |                     | L(a) < L(b) | no prueba | que a → b. |
| ------------ | ---------- | ------------------- | ----------- | --------- | ---------- |
| Relojes      | de Lamport |                     |             |           |            |
| Cada proceso | mantiene   | un contador lógico: |             |           |            |
▶
| incrementa    | antes                      | de cada evento local; |     |     |     |
| ------------- | -------------------------- | --------------------- | --- | --- | --- |
| ▶ adjunta     | el contador                | al enviar mensajes;   |     |     |     |
| ▶ al recibir, | usa max(local,recibido)+1. |                       |     |     |     |
SistemasDistribuidos–Clase03 11/26

| Relojes      | de Lamport |     |                  |
| ------------ | ---------- | --- | ---------------- |
| Cada proceso | mantiene   | un  | contador lógico: |
▶
| incrementa    | antes                      | de cada | evento local;        |
| ------------- | -------------------------- | ------- | -------------------- |
| ▶ adjunta     | el contador                | al      | enviar mensajes;     |
| ▶ al recibir, | usa max(local,recibido)+1. |         |                      |
|               |                            |         | a → b =⇒ L(a) < L(b) |
Límite
| L(a) < L(b) | no prueba | que | a → b. |
| ----------- | --------- | --- | ------ |
SistemasDistribuidos–Clase03 11/26

| Eventual     | consistency          |                        |
| ------------ | -------------------- | ---------------------- |
| ▶ sin nuevas | escrituras,          | las réplicas convergen |
| ▶ no hay     | garantías inmediatas | de lectura             |
SistemasDistribuidos–Clase03 12/26

Trade-off
| ▶ fuerte: coordinación  | + latencia  | + menor disponibilidad |
| ----------------------- | ----------- | ---------------------- |
| ▶ débil: disponibilidad | + anomalías | temporales             |
SistemasDistribuidos–Clase03 13/26

Arquitecturas de replicación
Sistemas Distribuidos

Leader-based
▶ un líder serializa escrituras
▶ followers replican (sync/async)
SistemasDistribuidos–Clase03 15/26

Multi-leader
▶ múltiples líderes
▶ escritura local rápida
▶ conflictos inevitables
SistemasDistribuidos–Clase03 16/26

| Leaderless | (Dynamo) |     |
| ---------- | -------- | --- |
▶ N réplicas
| ▶ escribir | a W, leer | de R |
| ---------- | --------- | ---- |
▶
| condición | clásica: | R+W > N |
| --------- | -------- | ------- |
SistemasDistribuidos–Clase03 17/26

Propiedad
Si R+W > N, hay intersección de quorums.
Quorums
▶ write quorum W
▶ read quorum R
SistemasDistribuidos–Clase03 18/26

Quorums
| ▶ write | quorum W |     |
| ------- | -------- | --- |
▶
| read | quorum R |     |
| ---- | -------- | --- |
Propiedad
| Si R+W | > N, hay intersección | de quorums. |
| ------ | --------------------- | ----------- |
SistemasDistribuidos–Clase03 18/26

Conflictos y convergencia
Sistemas Distribuidos

Conflictos
| ▶ escrituras | concurrentes       | sobre mismo | objeto |
| ------------ | ------------------ | ----------- | ------ |
| ▶ orden      | global no definido |             |        |
SistemasDistribuidos–Clase03 20/26

Resolución
▶ LWW (last-write-wins)
▶ merge semántico
▶ versionado (vector clocks)
SistemasDistribuidos–Clase03 21/26

∥
(2,1,0) (1,2,0)
Uso
Detectar conflicto y forzar merge en vez de sobrescribir datos.
| Vector clocks     |                |                  |         |
| ----------------- | -------------- | ---------------- | ------- |
| Permiten comparar | versiones:     |                  |         |
| ▶ si un vector    | domina a otro, | hay descendencia | causal; |
▶
| si no son | comparables, hay | concurrencia. |     |
| --------- | ---------------- | ------------- | --- |
SistemasDistribuidos–Clase03 22/26

| Vector clocks     |                |                  |         |
| ----------------- | -------------- | ---------------- | ------- |
| Permiten comparar | versiones:     |                  |         |
| ▶ si un vector    | domina a otro, | hay descendencia | causal; |
▶
| si no son | comparables, hay | concurrencia. |     |
| --------- | ---------------- | ------------- | --- |
∥
|     |     | (2,1,0) | (1,2,0) |
| --- | --- | ------- | ------- |
Uso
Detectar conflicto y forzar merge en vez de sobrescribir datos.
SistemasDistribuidos–Clase03 22/26

Anti-entropy
| ▶ sincronización       | periódica entre | réplicas |
| ---------------------- | --------------- | -------- |
| ▶ gossip / read-repair |                 |          |
SistemasDistribuidos–Clase03 23/26

Modelo del curso
Sistemas Distribuidos

| Aplicación  | de D      | = (P,F,G,S,T) |
| ----------- | --------- | ------------- |
| ▶ P: estado | replicado |               |
▶
| F: delay, | pérdida, partición |     |
| --------- | ------------------ | --- |
▶
| G: (linearizable | / causal        | / eventual) |
| ---------------- | --------------- | ----------- |
| ▶ S: leader,     | multi-leader,   | quorum      |
| ▶ T: latencia,   | disponibilidad, | complejidad |
SistemasDistribuidos–Clase03 25/26

Cierre
Idea central
Replicar estado obliga a elegir qué inconsistencias aceptar y cuándo.
SistemasDistribuidos–Clase03 26/26