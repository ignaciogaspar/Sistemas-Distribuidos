| Sistemas       | Distribuidos           |               |      |
| -------------- | ---------------------- | ------------- | ---- |
| Clase          | 04 — Consenso          | (Raft, Paxos, | FLP) |
| Departamento   | de Computación         |               |      |
| Ingeniería     | Informática – Comisión | A – Pilar     |      |
| Equipo docente | – 11 de agosto         | de 2026       |      |

Modelo
Sistemas Distribuidos

Objetivo: acordar valor v
Modelo formal
Sistema:
▶ n procesos
▶ canales asíncronos
▶ fallas por crash
SistemasDistribuidos–Clase04 3/19

Modelo formal
Sistema:
▶ n procesos
▶ canales asíncronos
▶ fallas por crash
Objetivo: acordar valor v
SistemasDistribuidos–Clase04 3/19

Propiedades
▶ Agreement
▶ Validity
▶ Termination
SistemasDistribuidos–Clase04 4/19

FLP
Sistemas Distribuidos

FLP
No existe algoritmo determinista que garantice consenso en sistema asíncrono con fallas
SistemasDistribuidos–Clase04 6/19

Idea de la prueba
▶ estados bivalentes
▶ scheduler adversario
▶ ejecución infinita
SistemasDistribuidos–Clase04 7/19

Raft
Sistemas Distribuidos

Estados
▶ follower
▶ candidate
▶ leader
SistemasDistribuidos–Clase04 9/19

Elección
timeout mayoría
Follower Candidate Leader
timeout
SistemasDistribuidos–Clase04 10/19

Replicación
Leader
F1 F2
SistemasDistribuidos–Clase04 11/19

Invariantes
Sistemas Distribuidos

Safety
▶ log matching
▶ leader completeness
SistemasDistribuidos–Clase04 13/19

Quorum
|Q ∩Q | ≥ 1
1 2
SistemasDistribuidos–Clase04 14/19

Paxos
Sistemas Distribuidos

Fases
▶ Prepare
▶ Accept
SistemasDistribuidos–Clase04 16/19

Diagrama
prepare
Proposer Acceptors
promise
SistemasDistribuidos–Clase04 17/19

Cierre
Sistemas Distribuidos

Idea central
Consenso impone orden global a costo de latencia y disponibilidad
SistemasDistribuidos–Clase04 19/19