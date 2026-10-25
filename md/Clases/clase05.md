| Sistemas       | Distribuidos           |              |           |           |
| -------------- | ---------------------- | ------------ | --------- | --------- |
| Clase          | 05 — Serializabilidad, |              | anomalías | y 2PC/3PC |
| Departamento   | de                     | Computación  |           |           |
| Ingeniería     | Informática            | – Comisión   | A – Pilar |           |
| Equipo docente | –                      | 11 de agosto | de 2026   |           |

Motivación
Sistemas Distribuidos

Problema
¿Cómo ejecutar múltiples operaciones concurrentes sin violar invariantes?
| De orden | global | a correctitud        |     |
| -------- | ------ | -------------------- | --- |
| ▶        | ⇒      |                      |     |
| Consenso | orden  | total de operaciones |     |
▶
| Falta: | correctitud | lógica de operaciones | compuestas |
| ------ | ----------- | --------------------- | ---------- |
SistemasDistribuidos–Clase05 3/23

| De orden | global | a correctitud        |     |
| -------- | ------ | -------------------- | --- |
| ▶        | ⇒      |                      |     |
| Consenso | orden  | total de operaciones |     |
▶
| Falta: | correctitud | lógica de operaciones | compuestas |
| ------ | ----------- | --------------------- | ---------- |
Problema
¿Cómo ejecutar múltiples operaciones concurrentes sin violar invariantes?
SistemasDistribuidos–Clase05 3/23

ACID:
▶
| Atomicidad: | todo o nada |     |
| ----------- | ----------- | --- |
▶
| Consistencia:  | preserva     | invariantes        |
| -------------- | ------------ | ------------------ |
| ▶ Aislamiento: | ejecución    | como si fuera sola |
| ▶ Durabilidad: | persistencia | tras commit        |
Transacción
Definición
Unidad lógica de trabajo que debe ejecutarse de forma atómica y aislada.
SistemasDistribuidos–Clase05 4/23

Transacción
Definición
Unidad lógica de trabajo que debe ejecutarse de forma atómica y aislada.
ACID:
▶
| Atomicidad: | todo o nada |     |
| ----------- | ----------- | --- |
▶
| Consistencia:  | preserva     | invariantes        |
| -------------- | ------------ | ------------------ |
| ▶ Aislamiento: | ejecución    | como si fuera sola |
| ▶ Durabilidad: | persistencia | tras commit        |
SistemasDistribuidos–Clase05 4/23

Serializabilidad
Sistemas Distribuidos

Idea central
Una ejecución concurrente es correcta si equivale a una ejecución secuencial.
SistemasDistribuidos–Clase05 6/23

Criterio
Si el grafo tiene ciclo ⇒ no serializable
Grafo de precedencia
▶ nodos: transacciones
▶ aristas: dependencias por conflictos
SistemasDistribuidos–Clase05 7/23

| Grafo    | de precedencia |     |
| -------- | -------------- | --- |
| ▶ nodos: | transacciones  |     |
▶
| aristas: | dependencias | por conflictos |
| -------- | ------------ | -------------- |
Criterio
⇒
| Si el grafo | tiene ciclo | no serializable |
| ----------- | ----------- | --------------- |
SistemasDistribuidos–Clase05 7/23

Resultado: no existe orden secuencial equivalente
Ejemplo de ciclo
RW
T T
1 2
RW
SistemasDistribuidos–Clase05 8/23

| Ejemplo | de ciclo |     |
| ------- | -------- | --- |
RW
T T
1 2
RW
| Resultado: | no existe orden | secuencial equivalente |
| ---------- | --------------- | ---------------------- |
SistemasDistribuidos–Clase05 8/23

Anomalías
Sistemas Distribuidos

Dirty read
▶ T escribe
1
▶ T lee valor no confirmado
2
▶ T aborta
1
SistemasDistribuidos–Clase05 10/23

| Non-repeatable | read |     |
| -------------- | ---- | --- |
▶
| T lee | valor |     |
| ----- | ----- | --- |
1
| ▶ T lo | modifica |     |
| ------ | -------- | --- |
2
| ▶ T vuelve | a leer distinto | valor |
| ---------- | --------------- | ----- |
1
SistemasDistribuidos–Clase05 11/23

Phantom
▶
| T lee | conjunto |     |
| ----- | -------- | --- |
1
| ▶ T inserta | nuevo registro |     |
| ----------- | -------------- | --- |
2
| ▶ T vuelve | a leer ⇒ aparece | “fantasma” |
| ---------- | ---------------- | ---------- |
1
SistemasDistribuidos–Clase05 12/23

Clave
⇒
|     |     |     | Snapshot Isolation | permite esto | no es serializable |
| --- | --- | --- | ------------------ | ------------ | ------------------ |
Write skew
| ▶ ambas transacciones | leen | mismo estado |     |     |     |
| --------------------- | ---- | ------------ | --- | --- | --- |
▶
| ambas escriben | sin conflicto | directo |     |     |     |
| -------------- | ------------- | ------- | --- | --- | --- |
SistemasDistribuidos–Clase05 13/23

Write skew
| ▶ ambas transacciones |     | leen | mismo estado |
| --------------------- | --- | ---- | ------------ |
▶
| ambas escriben | sin | conflicto | directo |
| -------------- | --- | --------- | ------- |
Clave
⇒
| Snapshot Isolation | permite | esto | no es serializable |
| ------------------ | ------- | ---- | ------------------ |
SistemasDistribuidos–Clase05 13/23

Niveles de aislamiento
Sistemas Distribuidos

SQL estándar
▶ Read Uncommitted
▶ Read Committed
▶ Repeatable Read
▶ Serializable
SistemasDistribuidos–Clase05 15/23

| Snapshot   | Isolation    |                |      |
| ---------- | ------------ | -------------- | ---- |
| ▶ snapshot | consistente  | al inicio      |      |
| ▶ evita    | dirty read y | non-repeatable | read |
▶
| NO evita | write skew |     |     |
| -------- | ---------- | --- | --- |
SistemasDistribuidos–Clase05 16/23

Commit distribuido
Sistemas Distribuidos

Problema
| Commit | en múltiples nodos | debe ser atómico |
| ------ | ------------------ | ---------------- |
SistemasDistribuidos–Clase05 18/23

2PC
Fase 1: Prepare
| ▶ coordinador | pregunta | “¿puedes | committear?” |
| ------------- | -------- | -------- | ------------ |
Fase 2: Commit
| ▶           |          | ⇒       |     |
| ----------- | -------- | ------- | --- |
| si todos    | dicen sí | commit  |     |
| ▶ si alguno | dice no  | ⇒ abort |     |
SistemasDistribuidos–Clase05 19/23

Problema de 2PC
▶ coordinador cae
▶ participantes quedan bloqueados
SistemasDistribuidos–Clase05 20/23

Limitación:
▶ requiere sincronía parcial
3PC
▶ agrega fase intermedia (pre-commit)
▶ reduce bloqueo
SistemasDistribuidos–Clase05 21/23

3PC
▶
| agrega fase | intermedia | (pre-commit) |
| ----------- | ---------- | ------------ |
▶
reduce bloqueo
Limitación:
| ▶ requiere | sincronía parcial |     |
| ---------- | ----------------- | --- |
SistemasDistribuidos–Clase05 21/23

Cierre
Sistemas Distribuidos

Idea central
Serializabilidad es el estándar de corrección, pero requiere coordinación fuerte.
SistemasDistribuidos–Clase05 23/23