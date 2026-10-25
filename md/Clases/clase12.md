| Sistemas Distribuidos |     |     |
| --------------------- | --- | --- |
Clase 12 — SLOs, error budgets, experimentos bajo falla, inci-
dentes y postmortems
| Departamento de        | Computación |           |
| ---------------------- | ----------- | --------- |
| Ingeniería Informática | – Comisión  | A – Pilar |
| Equipo docente –       | 2026        |           |

¿Cuándo anda
suficientemente bien?
Sistemas Distribuidos

|     |     |     |     |     | La pregunta  | que falta  |             |                 |       |
| --- | --- | --- | --- | --- | ------------ | ---------- | ----------- | --------------- | ----- |
|     |     |     |     |     | ¿Cuándo está | el sistema | funcionando | suficientemente | bien? |
Sin una respuesta explícita, cada discusión se resuelve por intuición, por jerarquía o por
|         |                  |                    |               |        | quién argumenta | mejor:        |                  |                     |     |
| ------- | ---------------- | ------------------ | ------------- | ------ | --------------- | ------------- | ---------------- | ------------------- | --- |
|         |                  |                    |               |        | ▶ ¿Liberamos    | esta versión  | hoy?             |                     |     |
|         |                  |                    |               |        | ▶ ¿Esta         | alerta merece | despertar a      | alguien a las 3 AM? |     |
|         |                  |                    |               |        | ▶ ¿El circuit   | breaker       | que configuramos | alcanza?            |     |
|         |                  |                    |               |        | ▶ ¿El incidente | de ayer       | fue grave?       |                     |     |
| Dónde   | nos quedamos     |                    |               |        |                 |               |                  |                     |     |
| ▶ Clase | 10: cómo ver     | lo que pasa dentro | del sistema.  |        |                 |               |                  |                     |     |
| ▶ Clase | 11: cómo decidir | de antemano        | de qué manera | falla. |                 |               |                  |                     |     |
SistemasDistribuidos–Clase12 3/61

Sin una respuesta explícita, cada discusión se resuelve por intuición, por jerarquía o por
|              |                  |                             |               |        | quién argumenta | mejor:        |                     |             |
| ------------ | ---------------- | --------------------------- | ------------- | ------ | --------------- | ------------- | ------------------- | ----------- |
|              |                  |                             |               |        | ▶ ¿Liberamos    | esta versión  | hoy?                |             |
|              |                  |                             |               |        | ▶ ¿Esta         | alerta merece | despertar a alguien | a las 3 AM? |
|              |                  |                             |               |        | ▶ ¿El circuit   | breaker       | que configuramos    | alcanza?    |
|              |                  |                             |               |        | ▶ ¿El incidente | de ayer       | fue grave?          |             |
| Dónde        | nos quedamos     |                             |               |        |                 |               |                     |             |
| ▶ Clase      | 10: cómo ver     | lo que pasa dentro          | del sistema.  |        |                 |               |                     |             |
| ▶ Clase      | 11: cómo decidir | de antemano                 | de qué manera | falla. |                 |               |                     |             |
| La pregunta  | que falta        |                             |               |        |                 |               |                     |             |
| ¿Cuándo está | el sistema       | funcionando suficientemente |               | bien?  |                 |               |                     |             |
SistemasDistribuidos–Clase12 3/61

| Dónde        | nos quedamos     |             |             |                 |               |        |
| ------------ | ---------------- | ----------- | ----------- | --------------- | ------------- | ------ |
| ▶ Clase      | 10: cómo ver     | lo que      | pasa        | dentro          | del sistema.  |        |
| ▶ Clase      | 11: cómo decidir |             | de antemano |                 | de qué manera | falla. |
| La pregunta  | que falta        |             |             |                 |               |        |
| ¿Cuándo está | el sistema       | funcionando |             | suficientemente |               | bien?  |
Sin una respuesta explícita, cada discusión se resuelve por intuición, por jerarquía o por
| quién argumenta | mejor:        |                  |           |          |             |     |
| --------------- | ------------- | ---------------- | --------- | -------- | ----------- | --- |
| ▶ ¿Liberamos    | esta versión  | hoy?             |           |          |             |     |
| ▶ ¿Esta         | alerta merece | despertar        | a alguien |          | a las 3 AM? |     |
| ▶ ¿El circuit   | breaker       | que configuramos |           | alcanza? |             |     |
| ▶ ¿El incidente | de ayer       | fue grave?       |           |          |             |     |
SistemasDistribuidos–Clase12 3/61

|     |     |     |     | Para el trabajo | práctico |     |     |
| --- | --- | --- | --- | --------------- | -------- | --- | --- |
En Asteria Online la garantía no lleva números. Los números de hoy (SLO, error budget)
|          |          |     |     | van en la solución | y en su trade-off. | Lo vemos | al final. |
| -------- | -------- | --- | --- | ------------------ | ------------------ | -------- | --------- |
| Los seis | temas de | hoy |     |                    |                    |          |           |
1. SLI, SLO y SLA: definir con números qué es “suficientemente bien”.
2. Error budget: cuánta falla se tolera, y cómo se usa para decidir.
3. Alertas sobre el objetivo (burn rate): avisar a tiempo, sin falsas alarmas.
4. Experimentos bajo falla (Chaos Engineering): comprobar antes de un incidente
| real            | que el sistema | se comporta como | suponemos.           |     |     |     |     |
| --------------- | -------------- | ---------------- | -------------------- | --- | --- | --- | --- |
| 5. Gestión      | de incidentes: | responder        | de forma coordinada. |     |     |     |     |
| 6. Postmortems: | aprender       | para que         | no se repita.        |     |     |     |     |
SistemasDistribuidos–Clase12 4/61

| Los seis | temas | de  | hoy |     |     |     |
| -------- | ----- | --- | --- | --- | --- | --- |
1. SLI, SLO y SLA: definir con números qué es “suficientemente bien”.
2. Error budget: cuánta falla se tolera, y cómo se usa para decidir.
3. Alertas sobre el objetivo (burn rate): avisar a tiempo, sin falsas alarmas.
4. Experimentos bajo falla (Chaos Engineering): comprobar antes de un incidente
| real            | que el | sistema     | se comporta | como | suponemos.    |             |
| --------------- | ------ | ----------- | ----------- | ---- | ------------- | ----------- |
| 5. Gestión      | de     | incidentes: | responder   |      | de forma      | coordinada. |
| 6. Postmortems: |        | aprender    | para        | que  | no se repita. |             |
| Para el trabajo |        | práctico    |             |      |               |             |
En Asteria Online la garantía no lleva números. Los números de hoy (SLO, error budget)
| van en la | solución | y en | su trade-off. | Lo  | vemos | al final. |
| --------- | -------- | ---- | ------------- | --- | ----- | --------- |
SistemasDistribuidos–Clase12 4/61

Pasado cierto punto, más
|     |     |     |     | confiabilidad | es peor: |     |     |
| --- | --- | --- | --- | ------------- | -------- | --- | --- |
▶ El costo no es lineal: cada mejora puede costar cien veces más que la anterior.
Réplicas, capacidad ociosa y, sobre todo, ingenieros que no construyen funcionalidades.
▶ El usuario no lo nota: acostumbrado a un servicio confiable al 99%, nadie distingue
|     |     |     |     | un servicio | al 99,99% | de uno | al 99,999%. |
| --- | --- | --- | --- | ----------- | --------- | ------ | ----------- |
▶ Compite con la velocidad de cambio: más estabilidad, menos cambios.
|     |     |     |     | El objetivo | es mínimo | y máximo | a la vez |
| --- | --- | --- | --- | ----------- | --------- | -------- | -------- |
Por debajo, los usuarios sufren. Por encima, nadie nota el nueve de más, y ese esfuerzo
|                  |             |                |                 | podría haberse | usado | en otra cosa. |     |
| ---------------- | ----------- | -------------- | --------------- | -------------- | ----- | ------------- | --- |
| La confiabilidad |             | es gestión     | de riesgo       |                |       |               |     |
| El objetivo      | podría ser: | que el sistema | no falle nunca. |                |       |               |     |
SistemasDistribuidos–Clase12 5/61

|     |     |     |     | El objetivo | es mínimo | y máximo | a la vez |
| --- | --- | --- | --- | ----------- | --------- | -------- | -------- |
Por debajo, los usuarios sufren. Por encima, nadie nota el nueve de más, y ese esfuerzo
|                  |     |            |           | podría haberse | usado | en otra cosa. |     |
| ---------------- | --- | ---------- | --------- | -------------- | ----- | ------------- | --- |
| La confiabilidad |     | es gestión | de riesgo |                |       |               |     |
El objetivo podría ser: que el sistema no falle nunca. Pasado cierto punto, más
| confiabilidad | es peor: |     |     |     |     |     |     |
| ------------- | -------- | --- | --- | --- | --- | --- | --- |
▶ El costo no es lineal: cada mejora puede costar cien veces más que la anterior.
Réplicas, capacidad ociosa y, sobre todo, ingenieros que no construyen funcionalidades.
▶ El usuario no lo nota: acostumbrado a un servicio confiable al 99%, nadie distingue
| un servicio | al 99,99% | de uno al | 99,999%. |     |     |     |     |
| ----------- | --------- | --------- | -------- | --- | --- | --- | --- |
▶ Compite con la velocidad de cambio: más estabilidad, menos cambios.
SistemasDistribuidos–Clase12 5/61

| La confiabilidad |     | es gestión | de riesgo |
| ---------------- | --- | ---------- | --------- |
El objetivo podría ser: que el sistema no falle nunca. Pasado cierto punto, más
| confiabilidad | es peor: |     |     |
| ------------- | -------- | --- | --- |
▶ El costo no es lineal: cada mejora puede costar cien veces más que la anterior.
Réplicas, capacidad ociosa y, sobre todo, ingenieros que no construyen funcionalidades.
▶ El usuario no lo nota: acostumbrado a un servicio confiable al 99%, nadie distingue
| un servicio | al 99,99% | de uno | al 99,999%. |
| ----------- | --------- | ------ | ----------- |
▶ Compite con la velocidad de cambio: más estabilidad, menos cambios.
| El objetivo | es mínimo | y máximo | a la vez |
| ----------- | --------- | -------- | -------- |
Por debajo, los usuarios sufren. Por encima, nadie nota el nueve de más, y ese esfuerzo
| podría haberse | usado | en otra cosa. |     |
| -------------- | ----- | ------------- | --- |
SistemasDistribuidos–Clase12 5/61

Medir define “normal”; alertar lo vigila; validar y responder lo usan como referencia; aprender lo
corrige.
Cómo se encadenan los temas
Medir Alertar
SLI,SLO,errorbudget burnrate
elSLIesel
steadystate
lasaccionescambian
quésemideyprueba
Validar
experimentosbajofalla
Aprender
postmortem
unexperimentopuede
volverseincidente
Responder
gestióndeincidentes
SistemasDistribuidos–Clase12 6/61

Cómo se encadenan los temas
Medir Alertar
SLI,SLO,errorbudget burnrate
elSLIesel
steadystate
lasaccionescambian
quésemideyprueba
Validar
experimentosbajofalla
Aprender
postmortem
unexperimentopuede
volverseincidente
Responder
gestióndeincidentes
Medir define “normal”; alertar lo vigila; validar y responder lo usan como referencia; aprender lo
corrige.
SistemasDistribuidos–Clase12 6/61

Para quien lo usa, “funcionar bien” es: recibir una contraseña válida y recibirla rápido. Todavía
|            |         |              |                |     | vago: hay | que convertirlo | en números. |
| ---------- | ------- | ------------ | -------------- | --- | --------- | --------------- | ----------- |
| El ejemplo | de hoy: | el generador | de contraseñas |     |           |                 |             |
mayúsculas
minúsculas
solicitud
|     | Usuario | Generador |     | oculta |     |     |     |
| --- | ------- | --------- | --- | ------ | --- | --- | --- |
dígitos
especiales
▶ Lo de la clase 11: 5 llamadas a cada hijo, 20 por solicitud. minúsculas llama por dentro a
dígitos. Una contraseña sin especiales no es una degradación aceptable.
▶ Supuesto para las cuentas: 5 solicitudes/s ⇒ 18000 por hora y unas 12,96 millones en 30
(5×60×60×24×30).
días
SistemasDistribuidos–Clase12 7/61

| El ejemplo | de hoy: | el generador | de contraseñas |     |
| ---------- | ------- | ------------ | -------------- | --- |
mayúsculas
minúsculas
solicitud
|     | Usuario |     | Generador | oculta |
| --- | ------- | --- | --------- | ------ |
dígitos
especiales
▶ Lo de la clase 11: 5 llamadas a cada hijo, 20 por solicitud. minúsculas llama por dentro a
dígitos. Una contraseña sin especiales no es una degradación aceptable.
▶ Supuesto para las cuentas: 5 solicitudes/s ⇒ 18000 por hora y unas 12,96 millones en 30
(5×60×60×24×30).
días
Para quien lo usa, “funcionar bien” es: recibir una contraseña válida y recibirla rápido. Todavía
| vago: hay | que convertirlo | en números. |     |     |
| --------- | --------------- | ----------- | --- | --- |
SistemasDistribuidos–Clase12 7/61

| Medir: | SLI, | SLO | y SLA |
| ------ | ---- | --- | ----- |
Sistemas Distribuidos

|     |     |     |     | Un acuerdo | explícito tiene       | tres piezas |        |
| --- | --- | --- | --- | ---------- | --------------------- | ----------- | ------ |
|     |     |     |     | 1. Qué     | se mide: el indicador | (SLI).      |        |
|     |     |     |     | 2. Qué     | valor es aceptable:   | el objetivo | (SLO). |
3. Qué pasa si no se cumple: el acuerdo con consecuencias (SLA).
| Una promesa | implícita |     |     |     |     |     |     |
| ----------- | --------- | --- | --- | --- | --- | --- | --- |
Todo servicio lleva una promesa: que responda, rápido, y bien. Mientras sea implícita, cada
| uno la lee a su | manera: |     |     |     |     |     |     |
| --------------- | ------- | --- | --- | --- | --- | --- | --- |
▶ quien desarrolla: “anda bien si no hay errores en los logs”;
| ▶ quien lo usa:   | “si la página | de registro | no se traba”; |     |     |     |     |
| ----------------- | ------------- | ----------- | ------------- | --- | --- | --- | --- |
| ▶ quien lo vende: | “si nadie     | se queja”.  |               |     |     |     |     |
SistemasDistribuidos–Clase12 9/61

| Una promesa | implícita |     |     |
| ----------- | --------- | --- | --- |
Todo servicio lleva una promesa: que responda, rápido, y bien. Mientras sea implícita, cada
| uno la lee | a su manera: |     |     |
| ---------- | ------------ | --- | --- |
▶ quien desarrolla: “anda bien si no hay errores en los logs”;
| ▶ quien    | lo usa: “si la página | de registro | no se traba”; |
| ---------- | --------------------- | ----------- | ------------- |
| ▶ quien    | lo vende: “si nadie   | se queja”.  |               |
| Un acuerdo | explícito tiene       | tres piezas |               |
| 1. Qué     | se mide: el indicador | (SLI).      |               |
| 2. Qué     | valor es aceptable:   | el objetivo | (SLO).        |
3. Qué pasa si no se cumple: el acuerdo con consecuencias (SLA).
SistemasDistribuidos–Clase12 9/61

En el generador
▶ SLI: el porcentaje de solicitudes que reciben una contraseña válida.
▶ SLO: ese porcentaje es al menos 99,9% en los últimos 30 días.
▶ SLA: si en un mes queda por debajo de 99,5%, el cliente recibe un descuento (ilustrativo).
SLI, SLO y SLA
Las tres siglas empiezan con SL, Service Level: el nivel de servicio, es decir, qué tan bien
le funciona el servicio a quien lo usa.
▶ SLI (Service Level Indicator): una medida cuantitativa del nivel de servicio.
▶ SLO (Service Level Objective): un valor objetivo para un SLI: “SLI ≥ objetivo”.
▶ SLA (Service Level Agreement): un contrato con consecuencias si no se cumplen sus
SLOs.
SistemasDistribuidos–Clase12 10/61

SLI, SLO y SLA
Las tres siglas empiezan con SL, Service Level: el nivel de servicio, es decir, qué tan bien
le funciona el servicio a quien lo usa.
▶ SLI (Service Level Indicator): una medida cuantitativa del nivel de servicio.
▶ SLO (Service Level Objective): un valor objetivo para un SLI: “SLI ≥ objetivo”.
▶ SLA (Service Level Agreement): un contrato con consecuencias si no se cumplen sus
SLOs.
En el generador
▶ SLI: el porcentaje de solicitudes que reciben una contraseña válida.
▶ SLO: ese porcentaje es al menos 99,9% en los últimos 30 días.
▶ SLA: si en un mes queda por debajo de 99,5%, el cliente recibe un descuento (ilustrativo).
SistemasDistribuidos–Clase12 10/61

“Válidos” define la población
Una solicitud mal formada (4xx) no dice nada sobre la salud del servicio: se excluye.
| El SLI como | proporción | de eventos      | buenos |
| ----------- | ---------- | --------------- | ------ |
|             |            | eventos buenos  |        |
|             |            | SLI =           | ×100%  |
|             |            | eventos válidos |        |
▶ Todos los SLIs en la misma escala: de 0% a 100%, donde 100% es lo mejor.
▶
Mide bien las fallas parciales, que en un sistema distribuido son la norma.
▶ Disponibilidad = solicitudes exitosas / válidas: la disponibilidad semántica de la
| clase 8, | no el uptime | del proceso. |     |
| -------- | ------------ | ------------ | --- |
SistemasDistribuidos–Clase12 11/61

| El SLI como | proporción | de eventos      | buenos |
| ----------- | ---------- | --------------- | ------ |
|             |            | eventos buenos  |        |
|             |            | SLI =           | ×100%  |
|             |            | eventos válidos |        |
▶ Todos los SLIs en la misma escala: de 0% a 100%, donde 100% es lo mejor.
▶
Mide bien las fallas parciales, que en un sistema distribuido son la norma.
▶ Disponibilidad = solicitudes exitosas / válidas: la disponibilidad semántica de la
| clase 8,         | no el uptime | del proceso. |     |
| ---------------- | ------------ | ------------ | --- |
| “Válidos” define | la población |              |     |
Una solicitud mal formada (4xx) no dice nada sobre la salud del servicio: se excluye.
SistemasDistribuidos–Clase12 11/61

|     |     |     |     | Dónde | Ventaja | Limitación |
| --- | --- | --- | --- | ----- | ------- | ---------- |
Logsdelgenerador Barata,desdeeldíauno. Noveloquenuncallegóalproceso.
Balanceador Máscercadelusuario. No sabe si la contraseña cumple la
política.
Clientesintético(prober) Recorrido completo, valida con- Tráficoartificial.
tenido.
Instrumentaciónenelcliente Lomásreal. Cara;mezclalareddelusuario.
Regla: medir lo más cerca del usuario que se pueda. Para empezar, lo que cueste poco.
| Especificación | e   | implementación |     |     |     |     |
| -------------- | --- | -------------- | --- | --- | --- | --- |
Especificación: lo que le importa al usuario. “La proporción de solicitudes de generación
| de contraseña   | que reciben | una contraseña  | válida”. |     |     |     |
| --------------- | ----------- | --------------- | -------- | --- | --- | --- |
| Implementación: | dónde       | y cómo se mide. |          |     |     |     |
SistemasDistribuidos–Clase12 12/61

| Especificación | e   | implementación |     |     |
| -------------- | --- | -------------- | --- | --- |
Especificación: lo que le importa al usuario. “La proporción de solicitudes de generación
| de contraseña   | que reciben | una contraseña  | válida”. |            |
| --------------- | ----------- | --------------- | -------- | ---------- |
| Implementación: | dónde       | y cómo se mide. |          |            |
| Dónde           |             | Ventaja         |          | Limitación |
Logsdelgenerador Barata,desdeeldíauno. Noveloquenuncallegóalproceso.
Balanceador Máscercadelusuario. No sabe si la contraseña cumple la
política.
Clientesintético(prober) Recorrido completo, valida con- Tráficoartificial.
tenido.
Instrumentaciónenelcliente Lomásreal. Cara;mezclalareddelusuario.
Regla: medir lo más cerca del usuario que se pueda. Para empezar, lo que cueste poco.
SistemasDistribuidos–Clase12 12/61

|     |     |     |     |     | La corrección | entra en | el SLI |
| --- | --- | --- | --- | --- | ------------- | -------- | ------ |
Si “bueno” fuera cualquier HTTP 200, una contraseña sin especiales devuelta por un
fallback mal decidido contaría como éxito. El evento bueno es: 200 con una contraseña
|                  |       |                 |          |     | que cumple | la política. |     |
| ---------------- | ----- | --------------- | -------- | --- | ---------- | ------------ | --- |
| Qué medir        | según | el tipo de      | servicio |     |            |              |     |
| Tipo de servicio |       | SLIs habituales |          |     |            |              |     |
Atiende solicitudes Disponibilidad, latencia, calidad (¿estaba degradada?)
| Almacenamiento |     | Latencia, | disponibilidad, | durabilidad |     |     |     |
| -------------- | --- | --------- | --------------- | ----------- | --- | --- | --- |
Pipeline de datos Frescura, corrección, cobertura, throughput
| Cualquiera |     | Corrección: | ¿la respuesta | era la correcta? |     |     |     |
| ---------- | --- | ----------- | ------------- | ---------------- | --- | --- | --- |
SistemasDistribuidos–Clase12 13/61

| Qué medir | según    | el tipo de      | servicio |     |
| --------- | -------- | --------------- | -------- | --- |
| Tipo de   | servicio | SLIs habituales |          |     |
Atiende solicitudes Disponibilidad, latencia, calidad (¿estaba degradada?)
| Almacenamiento |     | Latencia, | disponibilidad, | durabilidad |
| -------------- | --- | --------- | --------------- | ----------- |
Pipeline de datos Frescura, corrección, cobertura, throughput
| Cualquiera    |          | Corrección: | ¿la respuesta | era la correcta? |
| ------------- | -------- | ----------- | ------------- | ---------------- |
| La corrección | entra en | el SLI      |               |                  |
Si “bueno” fuera cualquier HTTP 200, una contraseña sin especiales devuelta por un
fallback mal decidido contaría como éxito. El evento bueno es: 200 con una contraseña
| que cumple | la política. |     |     |     |
| ---------- | ------------ | --- | --- | --- |
SistemasDistribuidos–Clase12 13/61

▶
No copiar el desempeño actual: puede atar al equipo a esfuerzos heroicos.
▶
Mantenerlo simple: un “índice de salud” que promedia disponibilidad, latencia y CPU puede
|     |     |     | no moverse | aunque todo | cambie. | Un SLI por propiedad. |
| --- | --- | --- | ---------- | ----------- | ------- | --------------------- |
▶
|     |     |     | Evitar absolutos: | “siempre | disponible” | no es un requisito. |
| --- | --- | --- | ----------------- | -------- | ----------- | ------------------- |
▶
Pocos SLOs: si uno nunca sirve para ganar una discusión de prioridades, sobra.
▶
|     |     |     | La perfección | puede | esperar: empezar | laxo y ajustar. |
| --- | --- | --- | ------------- | ----- | ---------------- | --------------- |
La ventana
Deslizante de 30 días (o 4 semanas): se parece a la memoria del usuario. Una ventana fija
se reinicia de golpe; una de 90 días permite gastar el 90% del presupuesto en un día y
cumplir igual.
| SLO: cómo | elegir | el valor |     |     |     |     |
| --------- | ------ | -------- | --- | --- | --- | --- |
Empezar por lo que le importa al usuario, no por lo que es fácil de medir.
SistemasDistribuidos–Clase12 14/61

La ventana
Deslizante de 30 días (o 4 semanas): se parece a la memoria del usuario. Una ventana fija
se reinicia de golpe; una de 90 días permite gastar el 90% del presupuesto en un día y
cumplir igual.
| SLO: cómo |     | elegir | el  | valor |     |
| --------- | --- | ------ | --- | ----- | --- |
Empezar por lo que le importa al usuario, no por lo que es fácil de medir.
▶
No copiar el desempeño actual: puede atar al equipo a esfuerzos heroicos.
▶
Mantenerlo simple: un “índice de salud” que promedia disponibilidad, latencia y CPU puede
| no moverse | aunque | todo | cambie. | Un  | SLI por propiedad. |
| ---------- | ------ | ---- | ------- | --- | ------------------ |
▶
| Evitar absolutos: |     | “siempre |     | disponible” | no es un requisito. |
| ----------------- | --- | -------- | --- | ----------- | ------------------- |
▶
Pocos SLOs: si uno nunca sirve para ganar una discusión de prioridades, sobra.
▶
| La perfección |     | puede | esperar: | empezar | laxo y ajustar. |
| ------------- | --- | ----- | -------- | ------- | --------------- |
SistemasDistribuidos–Clase12 14/61

| SLO: cómo |     | elegir | el  | valor |     |
| --------- | --- | ------ | --- | ----- | --- |
Empezar por lo que le importa al usuario, no por lo que es fácil de medir.
▶
No copiar el desempeño actual: puede atar al equipo a esfuerzos heroicos.
▶
Mantenerlo simple: un “índice de salud” que promedia disponibilidad, latencia y CPU puede
| no moverse | aunque | todo | cambie. | Un  | SLI por propiedad. |
| ---------- | ------ | ---- | ------- | --- | ------------------ |
▶
| Evitar absolutos: |     | “siempre |     | disponible” | no es un requisito. |
| ----------------- | --- | -------- | --- | ----------- | ------------------- |
▶
Pocos SLOs: si uno nunca sirve para ganar una discusión de prioridades, sobra.
▶
| La perfección |     | puede | esperar: | empezar | laxo y ajustar. |
| ------------- | --- | ----- | -------- | ------- | --------------- |
La ventana
Deslizante de 30 días (o 4 semanas): se parece a la memoria del usuario. Una ventana fija
se reinicia de golpe; una de 90 días permite gastar el 90% del presupuesto en un día y
cumplir igual.
SistemasDistribuidos–Clase12 14/61

No sobrecumplir. Los usuarios construyen sobre lo que el servicio realmente da. Si rinde
mucho mejor que su SLO, otros dependen de él como si no pudiera fallar, y cuando falla
|     |     |     | no saben funcionar | sin                | él. |     |     |
| --- | --- | --- | ------------------ | ------------------ | --- | --- | --- |
|     |     |     | Tres formas        | de no sobrecumplir |     |     |     |
▶ Diseñarlo para que no sea más rápido cuando tiene poca carga.
|     |     |     | ▶ Rechazar | a propósito | una pequeña | parte de | las solicitudes. |
| --- | --- | --- | ---------- | ----------- | ----------- | -------- | ---------------- |
▶ La más drástica: apagarlo a propósito de vez en cuando, si no hubo caídas reales. Se usa en
servicios de infraestructura de los que dependen muchos otros.
| Margen | de seguridad | y no sobrecumplir |     |     |     |     |     |
| ------ | ------------ | ----------------- | --- | --- | --- | --- | --- |
| Margen | de seguridad |                   |     |     |     |     |     |
Usar internamente un SLO más estricto que el que se publica: da tiempo a corregir un
| problema | antes de que lo note | el usuario. |     |     |     |     |     |
| -------- | -------------------- | ----------- | --- | --- | --- | --- | --- |
SistemasDistribuidos–Clase12 15/61

|     |     |     | Tres formas | de no sobrecumplir |     |     |     |
| --- | --- | --- | ----------- | ------------------ | --- | --- | --- |
▶ Diseñarlo para que no sea más rápido cuando tiene poca carga.
|     |     |     | ▶ Rechazar | a propósito | una pequeña | parte de | las solicitudes. |
| --- | --- | --- | ---------- | ----------- | ----------- | -------- | ---------------- |
▶ La más drástica: apagarlo a propósito de vez en cuando, si no hubo caídas reales. Se usa en
servicios de infraestructura de los que dependen muchos otros.
| Margen | de seguridad | y no sobrecumplir |     |     |     |     |     |
| ------ | ------------ | ----------------- | --- | --- | --- | --- | --- |
| Margen | de seguridad |                   |     |     |     |     |     |
Usar internamente un SLO más estricto que el que se publica: da tiempo a corregir un
| problema | antes de que lo note | el usuario. |     |     |     |     |     |
| -------- | -------------------- | ----------- | --- | --- | --- | --- | --- |
No sobrecumplir. Los usuarios construyen sobre lo que el servicio realmente da. Si rinde
mucho mejor que su SLO, otros dependen de él como si no pudiera fallar, y cuando falla
| no saben | funcionar sin él. |     |     |     |     |     |     |
| -------- | ----------------- | --- | --- | --- | --- | --- | --- |
SistemasDistribuidos–Clase12 15/61

| Margen | de seguridad | y   | no sobrecumplir |     |
| ------ | ------------ | --- | --------------- | --- |
| Margen | de seguridad |     |                 |     |
Usar internamente un SLO más estricto que el que se publica: da tiempo a corregir un
| problema | antes de que | lo note el usuario. |     |     |
| -------- | ------------ | ------------------- | --- | --- |
No sobrecumplir. Los usuarios construyen sobre lo que el servicio realmente da. Si rinde
mucho mejor que su SLO, otros dependen de él como si no pudiera fallar, y cuando falla
| no saben    | funcionar sin      | él. |     |     |
| ----------- | ------------------ | --- | --- | --- |
| Tres formas | de no sobrecumplir |     |     |     |
▶ Diseñarlo para que no sea más rápido cuando tiene poca carga.
| ▶ Rechazar | a propósito | una pequeña | parte de | las solicitudes. |
| ---------- | ----------- | ----------- | -------- | ---------------- |
▶ La más drástica: apagarlo a propósito de vez en cuando, si no hubo caídas reales. Se usa en
servicios de infraestructura de los que dependen muchos otros.
SistemasDistribuidos–Clase12 15/61

SLA < SLO interno
El generador promete 99,5% en el SLA y se exige 99,9% en el SLO. El equipo actúa al
apartarse del 99,9%, mucho antes de que se active la penalidad.
No todo servicio tiene SLA. Con o sin él, se opera con SLIs y SLOs.
SLA: el acuerdo
La prueba
¿Qué pasa si no se cumple? Si no hay una consecuencia explícita (reintegro, penalidad), es
un SLO.
▶ Es una decisión comercial y legal. Al equipo técnico le toca decir cuán difícil es
cumplirlo y que sus SLIs se puedan medir sin discusión.
▶ Conviene prometer poco: cuantos más clientes tiene un SLA, más difícil es cambiarlo.
SistemasDistribuidos–Clase12 16/61

SLA: el acuerdo
La prueba
¿Qué pasa si no se cumple? Si no hay una consecuencia explícita (reintegro, penalidad), es
un SLO.
▶ Es una decisión comercial y legal. Al equipo técnico le toca decir cuán difícil es
cumplirlo y que sus SLIs se puedan medir sin discusión.
▶ Conviene prometer poco: cuantos más clientes tiene un SLA, más difícil es cambiarlo.
SLA < SLO interno
El generador promete 99,5% en el SLA y se exige 99,9% en el SLO. El equipo actúa al
apartarse del 99,9%, mucho antes de que se active la penalidad.
No todo servicio tiene SLA. Con o sin él, se opera con SLIs y SLOs.
SistemasDistribuidos–Clase12 16/61

SLI de latencia: el p99 de las solicitudes válidas, medido en el balanceador.
SLOs
Disponibilidad ≥ 99,9% y p99 de latencia < 500ms, sobre 30 días deslizantes.
| Los objetivos          | del generador |                    |     |     |
| ---------------------- | ------------- | ------------------ | --- | --- |
| SLI de disponibilidad, | medido        | en el balanceador: |     |     |
respuestas 200 con una contraseña que cumple la política 2xx buenas
=
|     | todas las solicitudes−respuestas |     | 4xx | 2xx+5xx |
| --- | -------------------------------- | --- | --- | ------- |
Las 5xx incluyen los timeouts y las conexiones cortadas (502, 504): son las fallas del servicio.
SistemasDistribuidos–Clase12 17/61

SLOs
Disponibilidad ≥ 99,9% y p99 de latencia < 500ms, sobre 30 días deslizantes.
| Los objetivos          | del generador |                    |     |     |
| ---------------------- | ------------- | ------------------ | --- | --- |
| SLI de disponibilidad, | medido        | en el balanceador: |     |     |
respuestas 200 con una contraseña que cumple la política 2xx buenas
=
|     | todas las solicitudes−respuestas |     | 4xx | 2xx+5xx |
| --- | -------------------------------- | --- | --- | ------- |
Las 5xx incluyen los timeouts y las conexiones cortadas (502, 504): son las fallas del servicio.
SLI de latencia: el p99 de las solicitudes válidas, medido en el balanceador.
SistemasDistribuidos–Clase12 17/61

| Los objetivos          | del generador |                    |     |     |
| ---------------------- | ------------- | ------------------ | --- | --- |
| SLI de disponibilidad, | medido        | en el balanceador: |     |     |
respuestas 200 con una contraseña que cumple la política 2xx buenas
=
|     | todas las solicitudes−respuestas |     | 4xx | 2xx+5xx |
| --- | -------------------------------- | --- | --- | ------- |
Las 5xx incluyen los timeouts y las conexiones cortadas (502, 504): son las fallas del servicio.
SLI de latencia: el p99 de las solicitudes válidas, medido en el balanceador.
SLOs
Disponibilidad ≥ 99,9% y p99 de latencia < 500ms, sobre 30 días deslizantes.
SistemasDistribuidos–Clase12 17/61

Error budget
Sistemas Distribuidos

▶
|     |     |     | Cada nueve | divide el presupuesto | por diez. |
| --- | --- | --- | ---------- | --------------------- | --------- |
▶
|     |     |     | En solicitudes, | el generador      | (99,9%)   |
| --- | --- | --- | --------------- | ----------------- | --------- |
|     |     |     | puede fallar    | 12960 solicitudes | en 30     |
|     |     |     | días: el        | 0,1% de 12,96     | millones. |
▶
|               |        |                 | O sea, 18 | por hora en | promedio. |
| ------------- | ------ | --------------- | --------- | ----------- | --------- |
| Error budget: | cuánto | se puede fallar |           |             |           |
Si el SLO no es 100%, hay una falla tolerada: el error budget es 1−SLO.
| SLO     | Caída en 30 días |     |     |     |     |
| ------- | ---------------- | --- | --- | --- | --- |
| 99%     | 7,2              | h   |     |     |     |
| 99,5%   | 3,6              | h   |     |     |     |
| 99,9%   | 43,2 min         |     |     |     |     |
| 99,95%  | 21,6 min         |     |     |     |     |
| 99,99%  | 4,32 min         |     |     |     |     |
| 99,999% | 25,9             | s   |     |     |     |
SistemasDistribuidos–Clase12 19/61

| Error budget: | cuánto | se puede fallar |     |     |     |
| ------------- | ------ | --------------- | --- | --- | --- |
Si el SLO no es 100%, hay una falla tolerada: el error budget es 1−SLO.
▶
|     |     |     | Cada nueve | divide el presupuesto | por diez. |
| --- | --- | --- | ---------- | --------------------- | --------- |
▶
| SLO   | Caída en 30 días |     | En solicitudes, | el generador      | (99,9%)   |
| ----- | ---------------- | --- | --------------- | ----------------- | --------- |
|       |                  |     | puede fallar    | 12960 solicitudes | en 30     |
| 99%   | 7,2              | h   |                 |                   |           |
|       |                  |     | días: el        | 0,1% de 12,96     | millones. |
| 99,5% | 3,6              | h   |                 |                   |           |
▶
| 99,9%   | 43,2 min |     | O sea, 18 | por hora en | promedio. |
| ------- | -------- | --- | --------- | ----------- | --------- |
| 99,95%  | 21,6 min |     |           |             |           |
| 99,99%  | 4,32 min |     |           |             |           |
| 99,999% | 25,9     | s   |           |             |           |
SistemasDistribuidos–Clase12 19/61

|     |     |     |     | 1. Producto | define el SLO. |     |     |
| --- | --- | --- | --- | ----------- | -------------- | --- | --- |
2. El monitoreo mide la disponibilidad real: un tercero neutral.
|     |     |     |     | 3. La diferencia | es el margen  | de falla      | que queda. |
| --- | --- | --- | --- | ---------------- | ------------- | ------------- | ---------- |
|     |     |     |     | 4. Mientras      | quede margen, | se despliega. |            |
El efecto
El equipo se regula solo: con presupuesto amplio, arriesga; casi agotado, es el propio
equipo el que pide más pruebas. Las caídas ajenas (la red, un centro de datos) también lo
|      |            |                    |     | gastan: la responsabilidad | es  | compartida. |     |
| ---- | ---------- | ------------------ | --- | -------------------------- | --- | ----------- | --- |
| Para | qué sirve: | alinear incentivos |     |                            |     |             |     |
Desarrollo quiere velocidad; operaciones, estabilidad. ¿Cuánto probar? ¿Cada cuánto
| desplegar? | ¿Cuánto dura | el canary (clase | 9)? |     |     |     |     |
| ---------- | ------------ | ---------------- | --- | --- | --- | --- | --- |
SistemasDistribuidos–Clase12 20/61

El efecto
El equipo se regula solo: con presupuesto amplio, arriesga; casi agotado, es el propio
equipo el que pide más pruebas. Las caídas ajenas (la red, un centro de datos) también lo
gastan: la responsabilidad es compartida.
| Para | qué sirve: | alinear | incentivos |     |
| ---- | ---------- | ------- | ---------- | --- |
Desarrollo quiere velocidad; operaciones, estabilidad. ¿Cuánto probar? ¿Cada cuánto
| desplegar?  | ¿Cuánto dura | el canary | (clase | 9)? |
| ----------- | ------------ | --------- | ------ | --- |
| 1. Producto | define       | el SLO.   |        |     |
2. El monitoreo mide la disponibilidad real: un tercero neutral.
| 3. La       | diferencia es el | margen  | de falla      | que queda. |
| ----------- | ---------------- | ------- | ------------- | ---------- |
| 4. Mientras | quede            | margen, | se despliega. |            |
SistemasDistribuidos–Clase12 20/61

| Para | qué sirve: | alinear | incentivos |     |
| ---- | ---------- | ------- | ---------- | --- |
Desarrollo quiere velocidad; operaciones, estabilidad. ¿Cuánto probar? ¿Cada cuánto
| desplegar?  | ¿Cuánto dura | el canary | (clase | 9)? |
| ----------- | ------------ | --------- | ------ | --- |
| 1. Producto | define       | el SLO.   |        |     |
2. El monitoreo mide la disponibilidad real: un tercero neutral.
| 3. La       | diferencia es el | margen  | de falla      | que queda. |
| ----------- | ---------------- | ------- | ------------- | ---------- |
| 4. Mientras | quede            | margen, | se despliega. |            |
El efecto
El equipo se regula solo: con presupuesto amplio, arriesga; casi agotado, es el propio
equipo el que pide más pruebas. Las caídas ajenas (la red, un centro de datos) también lo
| gastan: | la responsabilidad | es compartida. |     |     |
| ------- | ------------------ | -------------- | --- | --- |
SistemasDistribuidos–Clase12 20/61

| La política | de error | budget |
| ----------- | -------- | ------ |
Las consecuencias se acuerdan antes, por escrito, con quién decide si hay desacuerdo. Una
| política posible | para el generador: |         |
| ---------------- | ------------------ | ------- |
| Presupuesto      | Qué                | se hace |
Más del 50% Despliegues normales: siempre con canary y reversión automática.
Entre 50% y 0% Desplieguesen horariosdepoco tráfico, y escalonados: 5% deltráfico,
15 minutos mirando métricas, siguiente escalón. Más lento y tedioso.
Agotado Se congela todo lo que no sea confiabilidad o seguridad.
Y una regla más: un incidente que consume más del 20% del presupuesto exige un postmortem.
SistemasDistribuidos–Clase12 21/61

La pendiente de la línea es el burn rate: cuanto más empinada, más rápido se agota. Las bandas
|      |          |                |     |     |     | son la política | de error budget | de la próxima | slide. |
| ---- | -------- | -------------- | --- | --- | --- | --------------- | --------------- | ------------- | ------ |
| Cómo | se gasta | el presupuesto |     |     |     |                 |                 |               |        |
presupuestorestante(100%=12960fallas)
100%
75%
50%
ritmopermitido:
llegaa0eldía30
25%
0%
|     | 0   | 5   | 10 15 | 20 25 | 30  |     |     |     |     |
| --- | --- | --- | ----- | ----- | --- | --- | --- | --- | --- |
díasdelaventana
SistemasDistribuidos–Clase12 22/61

La pendiente de la línea es el burn rate: cuanto más empinada, más rápido se agota. Las bandas
|      |          |                |     |     |     | son la política | de error budget | de la próxima | slide. |
| ---- | -------- | -------------- | --- | --- | --- | --------------- | --------------- | ------------- | ------ |
| Cómo | se gasta | el presupuesto |     |     |     |                 |                 |               |        |
presupuestorestante(100%=12960fallas)
consumonormal
100%
75%
50%
ritmopermitido:
llegaa0eldía30
25%
0%
|     | 0   | 5   | 10 15 | 20 25 | 30  |     |     |     |     |
| --- | --- | --- | ----- | ----- | --- | --- | --- | --- | --- |
díasdelaventana
SistemasDistribuidos–Clase12 22/61

Las bandas
|      |          |                |     |     |     | son la política | de error budget | de la próxima | slide. |
| ---- | -------- | -------------- | --- | --- | --- | --------------- | --------------- | ------------- | ------ |
| Cómo | se gasta | el presupuesto |     |     |     |                 |                 |               |        |
presupuestorestante(100%=12960fallas)
consumonormal
100%
incidente:
−20%degolpe
|     | 75% |     |     | sinresolverdeltodo: |     |     |     |     |     |
| --- | --- | --- | --- | ------------------- | --- | --- | --- | --- | --- |
másempinada
50%
ritmopermitido:
llegaa0eldía30
25%
0%
|     | 0   | 5   | 10 15 | 20 25 | 30  |     |     |     |     |
| --- | --- | --- | ----- | ----- | --- | --- | --- | --- | --- |
díasdelaventana
La pendiente de la línea es el burn rate: cuanto más empinada, más rápido se agota.
SistemasDistribuidos–Clase12 22/61

Cómo se gasta el presupuesto
presupuestorestante(100%=12960fallas)
consumonormal
100%
desplieguesnormales
incidente:
75% −20%degolpe sinresolverdeltodo:
másempinada
50%
ritmopermitido:
llegaa0eldía30
25%
desplieguesconservadores
0%
cambioscongelados
0 5 10 15 20 25 30
díasdelaventana
La pendiente de la línea es el burn rate: cuanto más empinada, más rápido se agota. Las bandas
son la política de error budget de la próxima slide.
SistemasDistribuidos–Clase12 22/61

No monetario: el usuario ya pierde solicitudes por su propia conexión (su proveedor, el
celular, el Wi-Fi). Esa tasa de error de base está entre 0,01% y 1%. Por debajo, el
usuario no distingue los errores del servicio de los de su red.
Idea central
El SLO no es una aspiración: es una frontera de decisión. El 100% casi nunca es el
objetivo correcto: es imposible, es caro y el usuario no lo nota.
El costo de cada nueve
Monetario: subir de 99,9% a 99,99% en un servicio que factura 1 millón de dólares gana
0,09%: 1M×0,0009 = 900 dólares. Si el nueve cuesta más, no conviene.
SistemasDistribuidos–Clase12 23/61

Idea central
El SLO no es una aspiración: es una frontera de decisión. El 100% casi nunca es el
objetivo correcto: es imposible, es caro y el usuario no lo nota.
El costo de cada nueve
Monetario: subir de 99,9% a 99,99% en un servicio que factura 1 millón de dólares gana
0,09%: 1M×0,0009 = 900 dólares. Si el nueve cuesta más, no conviene.
No monetario: el usuario ya pierde solicitudes por su propia conexión (su proveedor, el
celular, el Wi-Fi). Esa tasa de error de base está entre 0,01% y 1%. Por debajo, el
usuario no distingue los errores del servicio de los de su red.
SistemasDistribuidos–Clase12 23/61

El costo de cada nueve
Monetario: subir de 99,9% a 99,99% en un servicio que factura 1 millón de dólares gana
0,09%: 1M×0,0009 = 900 dólares. Si el nueve cuesta más, no conviene.
No monetario: el usuario ya pierde solicitudes por su propia conexión (su proveedor, el
celular, el Wi-Fi). Esa tasa de error de base está entre 0,01% y 1%. Por debajo, el
usuario no distingue los errores del servicio de los de su red.
Idea central
El SLO no es una aspiración: es una frontera de decisión. El 100% casi nunca es el
objetivo correcto: es imposible, es caro y el usuario no lo nota.
SistemasDistribuidos–Clase12 23/61

| Alertar | sobre | el SLO |
| ------- | ----- | ------ |
Sistemas Distribuidos

▶
La CPU sube por un backup o el garbage collector: falsos positivos.
▶
Una alarma que suena seguido sin consecuencias se ignora: fatiga de alertas (alert fatigue).
| El día     | que importa,    | nadie la mira.            |              |
| ---------- | --------------- | ------------------------- | ------------ |
| Una alerta | útil            |                           |              |
| 1. Cuando  | suena, los      | usuarios realmente tienen | un problema. |
| 2. Quien   | la recibe puede | hacer algo.               |              |
La del SLO cumple la 1 porque vigila el SLI: si cae, hay usuarios afectados, aunque la CPU esté
baja. Dice qué pasa, no por qué: para eso, trazas, métricas y logs (clase 10).
Excepción: alertar por una causa segura e inminente, aunque el usuario todavía no lo note: disco
casi lleno, certificado por vencer, servicio sin redundancia. Muchas veces, como ticket.
Síntomas, no causas
El monitoreo tradicional alerta sobre lo fácil de medir: CPU > 80%, memoria, disco. Son
causas posibles.
SistemasDistribuidos–Clase12 25/61

Una alerta útil
1. Cuando suena, los usuarios realmente tienen un problema.
2. Quien la recibe puede hacer algo.
La del SLO cumple la 1 porque vigila el SLI: si cae, hay usuarios afectados, aunque la CPU esté
baja. Dice qué pasa, no por qué: para eso, trazas, métricas y logs (clase 10).
Excepción: alertar por una causa segura e inminente, aunque el usuario todavía no lo note: disco
casi lleno, certificado por vencer, servicio sin redundancia. Muchas veces, como ticket.
Síntomas, no causas
El monitoreo tradicional alerta sobre lo fácil de medir: CPU > 80%, memoria, disco. Son
causas posibles.
▶ La CPU sube por un backup o el garbage collector: falsos positivos.
▶ Una alarma que suena seguido sin consecuencias se ignora: fatiga de alertas (alert fatigue).
El día que importa, nadie la mira.
SistemasDistribuidos–Clase12 25/61

Excepción: alertar por una causa segura e inminente, aunque el usuario todavía no lo note: disco
casi lleno, certificado por vencer, servicio sin redundancia. Muchas veces, como ticket.
| Síntomas, | no causas |     |     |
| --------- | --------- | --- | --- |
El monitoreo tradicional alerta sobre lo fácil de medir: CPU > 80%, memoria, disco. Son
causas posibles.
▶
La CPU sube por un backup o el garbage collector: falsos positivos.
▶
Una alarma que suena seguido sin consecuencias se ignora: fatiga de alertas (alert fatigue).
| El día     | que importa,    | nadie la mira.            |              |
| ---------- | --------------- | ------------------------- | ------------ |
| Una alerta | útil            |                           |              |
| 1. Cuando  | suena, los      | usuarios realmente tienen | un problema. |
| 2. Quien   | la recibe puede | hacer algo.               |              |
La del SLO cumple la 1 porque vigila el SLI: si cae, hay usuarios afectados, aunque la CPU esté
baja. Dice qué pasa, no por qué: para eso, trazas, métricas y logs (clase 10).
SistemasDistribuidos–Clase12 25/61

| Síntomas, | no causas |     |     |
| --------- | --------- | --- | --- |
El monitoreo tradicional alerta sobre lo fácil de medir: CPU > 80%, memoria, disco. Son
causas posibles.
▶
La CPU sube por un backup o el garbage collector: falsos positivos.
▶
Una alarma que suena seguido sin consecuencias se ignora: fatiga de alertas (alert fatigue).
| El día     | que importa,    | nadie la mira.            |              |
| ---------- | --------------- | ------------------------- | ------------ |
| Una alerta | útil            |                           |              |
| 1. Cuando  | suena, los      | usuarios realmente tienen | un problema. |
| 2. Quien   | la recibe puede | hacer algo.               |              |
La del SLO cumple la 1 porque vigila el SLI: si cae, hay usuarios afectados, aunque la CPU esté
baja. Dice qué pasa, no por qué: para eso, trazas, métricas y logs (clase 10).
Excepción: alertar por una causa segura e inminente, aunque el usuario todavía no lo note: disco
casi lleno, certificado por vencer, servicio sin redundancia. Muchas veces, como ticket.
SistemasDistribuidos–Clase12 25/61

|     |     |     |     |     |     |     | Llega tarde: | no queda nada | que proteger. |
| --- | --- | --- | --- | --- | --- | --- | ------------ | ------------- | ------------- |
2. Cuando el saldo baja de un umbral (queda menos del 30%). Tosca: no distingue
gastar el 70% en tres semanas de gastarlo en una hora; el umbral se vuelve un nuevo
cero.
3. Según la velocidad de consumo: con las fallas de la última hora, ¿a qué ritmo se
|     |     |     |     | gasta? Si      | lo agotaría | mucho antes | de 30 días, | actuar ya. |     |
| --- | --- | --- | --- | -------------- | ----------- | ----------- | ----------- | ---------- | --- |
|     |     |     |     | La recomendada | es la 3     |             |             |            |     |
El SLO define qué es falla y cuánto presupuesto hay; la alerta mide a qué velocidad se
|             |     |              |         | gasta. Urgencia | proporcional | al daño. |     |     |     |
| ----------- | --- | ------------ | ------- | --------------- | ------------ | -------- | --- | --- | --- |
| ¿Qué vigila | una | alerta sobre | el SLO? |                 |              |          |     |     |     |
El SLO del generador: 99,9% en 30 días. Presupuesto: 12960 fallas. ¿Cuándo suena la
alerta?
| 1. Cuando | el SLO | ya no se cumple. |     |     |     |     |     |     |     |
| --------- | ------ | ---------------- | --- | --- | --- | --- | --- | --- | --- |
SistemasDistribuidos–Clase12 26/61

2. Cuando el saldo baja de un umbral (queda menos del 30%). Tosca: no distingue
gastar el 70% en tres semanas de gastarlo en una hora; el umbral se vuelve un nuevo
cero.
3. Según la velocidad de consumo: con las fallas de la última hora, ¿a qué ritmo se
|     |     |     |     | gasta? Si      | lo agotaría | mucho antes de | 30 días, actuar | ya. |
| --- | --- | --- | --- | -------------- | ----------- | -------------- | --------------- | --- |
|     |     |     |     | La recomendada | es la 3     |                |                 |     |
El SLO define qué es falla y cuánto presupuesto hay; la alerta mide a qué velocidad se
|             |            |       |         | gasta. Urgencia | proporcional | al daño. |     |     |
| ----------- | ---------- | ----- | ------- | --------------- | ------------ | -------- | --- | --- |
| ¿Qué vigila | una alerta | sobre | el SLO? |                 |              |          |     |     |
El SLO del generador: 99,9% en 30 días. Presupuesto: 12960 fallas. ¿Cuándo suena la
alerta?
1. Cuando el SLO ya no se cumple. Llega tarde: no queda nada que proteger.
SistemasDistribuidos–Clase12 26/61

Tosca: no distingue
gastar el 70% en tres semanas de gastarlo en una hora; el umbral se vuelve un nuevo
cero.
3. Según la velocidad de consumo: con las fallas de la última hora, ¿a qué ritmo se
|     |     |     |     |     | gasta? Si      | lo agotaría | mucho antes de | 30 días, actuar | ya. |
| --- | --- | --- | --- | --- | -------------- | ----------- | -------------- | --------------- | --- |
|     |     |     |     |     | La recomendada | es la 3     |                |                 |     |
El SLO define qué es falla y cuánto presupuesto hay; la alerta mide a qué velocidad se
|             |     |              |         |     | gasta. Urgencia | proporcional | al daño. |     |     |
| ----------- | --- | ------------ | ------- | --- | --------------- | ------------ | -------- | --- | --- |
| ¿Qué vigila | una | alerta sobre | el SLO? |     |                 |              |          |     |     |
El SLO del generador: 99,9% en 30 días. Presupuesto: 12960 fallas. ¿Cuándo suena la
alerta?
1. Cuando el SLO ya no se cumple. Llega tarde: no queda nada que proteger.
| 2. Cuando | el saldo baja | de un umbral | (queda menos | del 30%). |     |     |     |     |     |
| --------- | ------------- | ------------ | ------------ | --------- | --- | --- | --- | --- | --- |
SistemasDistribuidos–Clase12 26/61

3. Según la velocidad de consumo: con las fallas de la última hora, ¿a qué ritmo se
|     |     |     |     | gasta? Si      | lo agotaría | mucho antes de | 30 días, actuar | ya. |
| --- | --- | --- | --- | -------------- | ----------- | -------------- | --------------- | --- |
|     |     |     |     | La recomendada | es la 3     |                |                 |     |
El SLO define qué es falla y cuánto presupuesto hay; la alerta mide a qué velocidad se
|             |            |       |         | gasta. Urgencia | proporcional | al daño. |     |     |
| ----------- | ---------- | ----- | ------- | --------------- | ------------ | -------- | --- | --- |
| ¿Qué vigila | una alerta | sobre | el SLO? |                 |              |          |     |     |
El SLO del generador: 99,9% en 30 días. Presupuesto: 12960 fallas. ¿Cuándo suena la
alerta?
1. Cuando el SLO ya no se cumple. Llega tarde: no queda nada que proteger.
2. Cuando el saldo baja de un umbral (queda menos del 30%). Tosca: no distingue
gastar el 70% en tres semanas de gastarlo en una hora; el umbral se vuelve un nuevo
cero.
SistemasDistribuidos–Clase12 26/61

|     |     |     |     |     | La recomendada | es la 3 |     |
| --- | --- | --- | --- | --- | -------------- | ------- | --- |
El SLO define qué es falla y cuánto presupuesto hay; la alerta mide a qué velocidad se
|             |            |       |         |     | gasta. Urgencia | proporcional | al daño. |
| ----------- | ---------- | ----- | ------- | --- | --------------- | ------------ | -------- |
| ¿Qué vigila | una alerta | sobre | el SLO? |     |                 |              |          |
El SLO del generador: 99,9% en 30 días. Presupuesto: 12960 fallas. ¿Cuándo suena la
alerta?
1. Cuando el SLO ya no se cumple. Llega tarde: no queda nada que proteger.
2. Cuando el saldo baja de un umbral (queda menos del 30%). Tosca: no distingue
gastar el 70% en tres semanas de gastarlo en una hora; el umbral se vuelve un nuevo
cero.
3. Según la velocidad de consumo: con las fallas de la última hora, ¿a qué ritmo se
| gasta? Si | lo agotaría mucho | antes de | 30 días, actuar | ya. |     |     |     |
| --------- | ----------------- | -------- | --------------- | --- | --- | --- | --- |
SistemasDistribuidos–Clase12 26/61

| ¿Qué vigila | una alerta | sobre | el SLO? |     |
| ----------- | ---------- | ----- | ------- | --- |
El SLO del generador: 99,9% en 30 días. Presupuesto: 12960 fallas. ¿Cuándo suena la
alerta?
1. Cuando el SLO ya no se cumple. Llega tarde: no queda nada que proteger.
2. Cuando el saldo baja de un umbral (queda menos del 30%). Tosca: no distingue
gastar el 70% en tres semanas de gastarlo en una hora; el umbral se vuelve un nuevo
cero.
3. Según la velocidad de consumo: con las fallas de la última hora, ¿a qué ritmo se
| gasta? Si      | lo agotaría | mucho antes de | 30 días, actuar | ya. |
| -------------- | ----------- | -------------- | --------------- | --- |
| La recomendada | es la 3     |                |                 |     |
El SLO define qué es falla y cuánto presupuesto hay; la alerta mide a qué velocidad se
| gasta. Urgencia | proporcional | al daño. |     |     |
| --------------- | ------------ | -------- | --- | --- |
SistemasDistribuidos–Clase12 26/61

Ejemplo: última hora, 300 fallidas de 18000 (1,67%); últimos 5 minutos, 30 de 1500 (2%). Suena.
|     |     |     |     |     | A 300 fallas | por hora, | el presupuesto | (12960) dura | 43 horas. |
| --- | --- | --- | --- | --- | ------------ | --------- | -------------- | ------------ | --------- |
▶ Alerta inmediata (page): despierta a la persona de guardia (on-call). No puede esperar.
▶ Ticket: tarea para el día hábil siguiente. Se gasta rápido de más, pero no tanto.
¿De dónde sale el 1,44%? ¿Para qué la segunda ventana? Siguientes slides.
| La alerta | del generador, | concreta |     |     |     |     |     |     |     |
| --------- | -------------- | -------- | --- | --- | --- | --- | --- | --- | --- |
Tasa de error en una ventana w = solicitudes fallidas en w / solicitudes válidas en w.
Alerta inmediata
|     | tasa de | error de la última | hora | > 1,44% |     |     |     |     |     |
| --- | ------- | ------------------ | ---- | ------- | --- | --- | --- | --- | --- |
y
|     | tasa de error | de los últimos | 5 minutos | > 1,44% |     |     |     |     |     |
| --- | ------------- | -------------- | --------- | ------- | --- | --- | --- | --- | --- |
SistemasDistribuidos–Clase12 27/61

▶ Alerta inmediata (page): despierta a la persona de guardia (on-call). No puede esperar.
▶ Ticket: tarea para el día hábil siguiente. Se gasta rápido de más, pero no tanto.
¿De dónde sale el 1,44%? ¿Para qué la segunda ventana? Siguientes slides.
| La alerta | del | generador, | concreta |     |     |
| --------- | --- | ---------- | -------- | --- | --- |
Tasa de error en una ventana w = solicitudes fallidas en w / solicitudes válidas en w.
Alerta inmediata
|     |     | tasa | de error de la | última hora | > 1,44% |
| --- | --- | ---- | -------------- | ----------- | ------- |
y
|     |     | tasa de error | de los últimos | 5 minutos | > 1,44% |
| --- | --- | ------------- | -------------- | --------- | ------- |
Ejemplo: última hora, 300 fallidas de 18000 (1,67%); últimos 5 minutos, 30 de 1500 (2%). Suena.
| A 300 fallas | por hora, | el presupuesto | (12960) dura | 43 horas. |     |
| ------------ | --------- | -------------- | ------------ | --------- | --- |
SistemasDistribuidos–Clase12 27/61

| La alerta | del | generador, | concreta |     |     |
| --------- | --- | ---------- | -------- | --- | --- |
Tasa de error en una ventana w = solicitudes fallidas en w / solicitudes válidas en w.
Alerta inmediata
|     |     | tasa | de error de la | última hora | > 1,44% |
| --- | --- | ---- | -------------- | ----------- | ------- |
y
|     |     | tasa de error | de los últimos | 5 minutos | > 1,44% |
| --- | --- | ------------- | -------------- | --------- | ------- |
Ejemplo: última hora, 300 fallidas de 18000 (1,67%); últimos 5 minutos, 30 de 1500 (2%). Suena.
| A 300 fallas | por hora, | el presupuesto | (12960) dura | 43 horas. |     |
| ------------ | --------- | -------------- | ------------ | --------- | --- |
▶ Alerta inmediata (page): despierta a la persona de guardia (on-call). No puede esperar.
▶ Ticket: tarea para el día hábil siguiente. Se gasta rápido de más, pero no tanto.
¿De dónde sale el 1,44%? ¿Para qué la segunda ventana? Siguientes slides.
SistemasDistribuidos–Clase12 27/61

|     | 1−s=0,001, |     |     | ⇒   |     |
| --- | ---------- | --- | --- | --- | --- |
Generador: s=0,999, T=720h b=e/0,001; se agota en 720/b horas.
| Tasa de | error (e) Burn | rate | Se  | agota en | Situación                     |
| ------- | -------------- | ---- | --- | -------- | ----------------------------- |
|         | 0,1%           |      | 1   | 30días   | justoenellímite               |
|         | 0,6%           |      | 6   | 5días    | degradaciónsostenida          |
|         | 1,44%          | 14,4 |     | 50h      | degradaciónseria              |
|         | 4,9%           | 49   |     | 14,7h    | especialesal10%,unreintento   |
|         | 41%            | 410  |     | 1,8h     | especialesal10%,sinreintentos |
|         | 100%           | 1000 |     | 43min    | caídatotal                    |
Burn rate
e T
b = se agota en
1−s b
e: tasa de error · s: objetivo del SLO · T: ventana. Con b=1 se gasta justo el presupuesto.
SistemasDistribuidos–Clase12 28/61

Burn rate
|     |     | e   |     |     | T        |
| --- | --- | --- | --- | --- | -------- |
|     | b   | =   |     | se  | agota en |
|     |     | 1−s |     |     | b        |
e: tasa de error · s: objetivo del SLO · T: ventana. Con b=1 se gasta justo el presupuesto.
|     | 1−s=0,001, |     |     | ⇒   |     |
| --- | ---------- | --- | --- | --- | --- |
Generador: s=0,999, T=720h b=e/0,001; se agota en 720/b horas.
| Tasa de | error (e) Burn | rate | Se  | agota en | Situación                     |
| ------- | -------------- | ---- | --- | -------- | ----------------------------- |
|         | 0,1%           |      | 1   | 30días   | justoenellímite               |
|         | 0,6%           |      | 6   | 5días    | degradaciónsostenida          |
|         | 1,44%          | 14,4 |     | 50h      | degradaciónseria              |
|         | 4,9%           | 49   |     | 14,7h    | especialesal10%,unreintento   |
|         | 41%            | 410  |     | 1,8h     | especialesal10%,sinreintentos |
|         | 100%           | 1000 |     | 43min    | caídatotal                    |
SistemasDistribuidos–Clase12 28/61

▶ Una sola ventana no alcanza. Corta (5 min): rápida pero ruidosa, un pico breve la dispara.
Larga (un día): precisa, pero detecta tarde y sigue sonando horas después de resuelto.
▶ Un solo umbral tampoco. Si suena solo con b>14,4, un consumo sostenido a b=10
nunca la dispara, y agota el presupuesto en 3 días.
La solución
Varios umbrales: los rápidos despiertan a alguien, los lentos generan un ticket. Y cada
uno exige dos ventanas a la vez: la larga confirma que es significativo; la corta, que sigue
ocurriendo. La corta apaga la alerta minutos después de resuelto.
Elegir la ventana y el umbral
Cuatro propiedades: precisión (si suena, hay problema), cobertura (todo problema serio
suena), tiempo de detección y tiempo de reinicio (cuánto sigue sonando después de
resuelto).
SistemasDistribuidos–Clase12 29/61

▶ Un solo umbral tampoco. Si suena solo con b>14,4, un consumo sostenido a b=10
nunca la dispara, y agota el presupuesto en 3 días.
La solución
Varios umbrales: los rápidos despiertan a alguien, los lentos generan un ticket. Y cada
uno exige dos ventanas a la vez: la larga confirma que es significativo; la corta, que sigue
ocurriendo. La corta apaga la alerta minutos después de resuelto.
Elegir la ventana y el umbral
Cuatro propiedades: precisión (si suena, hay problema), cobertura (todo problema serio
suena), tiempo de detección y tiempo de reinicio (cuánto sigue sonando después de
resuelto).
▶ Una sola ventana no alcanza. Corta (5 min): rápida pero ruidosa, un pico breve la dispara.
Larga (un día): precisa, pero detecta tarde y sigue sonando horas después de resuelto.
SistemasDistribuidos–Clase12 29/61

La solución
Varios umbrales: los rápidos despiertan a alguien, los lentos generan un ticket. Y cada
uno exige dos ventanas a la vez: la larga confirma que es significativo; la corta, que sigue
ocurriendo. La corta apaga la alerta minutos después de resuelto.
Elegir la ventana y el umbral
Cuatro propiedades: precisión (si suena, hay problema), cobertura (todo problema serio
suena), tiempo de detección y tiempo de reinicio (cuánto sigue sonando después de
resuelto).
▶ Una sola ventana no alcanza. Corta (5 min): rápida pero ruidosa, un pico breve la dispara.
Larga (un día): precisa, pero detecta tarde y sigue sonando horas después de resuelto.
▶ Un solo umbral tampoco. Si suena solo con b>14,4, un consumo sostenido a b=10
nunca la dispara, y agota el presupuesto en 3 días.
SistemasDistribuidos–Clase12 29/61

Elegir la ventana y el umbral
Cuatro propiedades: precisión (si suena, hay problema), cobertura (todo problema serio
suena), tiempo de detección y tiempo de reinicio (cuánto sigue sonando después de
resuelto).
▶ Una sola ventana no alcanza. Corta (5 min): rápida pero ruidosa, un pico breve la dispara.
Larga (un día): precisa, pero detecta tarde y sigue sonando horas después de resuelto.
▶ Un solo umbral tampoco. Si suena solo con b>14,4, un consumo sostenido a b=10
nunca la dispara, y agota el presupuesto en 3 días.
La solución
Varios umbrales: los rápidos despiertan a alguien, los lentos generan un ticket. Y cada
uno exige dos ventanas a la vez: la larga confirma que es significativo; la corta, que sigue
ocurriendo. La corta apaga la alerta minutos después de resuelto.
SistemasDistribuidos–Clase12 29/61

Si un burn rate b dura un tiempo t (acá, la ventana larga), se consume b·t/T: 14,4×1/720=2%,
6×6/720=5%, 1×72/720=10%. La ventana corta mide la duodécima parte de la larga.
Tiempo de detección
La alerta de 1 h suena cuando el promedio de la hora llega a 1,44%. Con especiales al
10% sin reintentos (e = 41%): unos 2 minutos. Caída total: menos de 1 minuto.
| Varias ventanas, | varios  | umbrales |         |        |      |                |     |
| ---------------- | ------- | -------- | ------- | ------ | ---- | -------------- | --- |
| Severidad        | Ventana | larga    | Ventana | corta  | Burn | rate Consumido |     |
| page             |         | 1        | h       | 5 min  |      | 14,4           | 2%  |
| page             |         | 6        | h       | 30 min |      | 6              | 5%  |
| Ticket           |         | 3 días   |         | 6      | h    | 1              | 10% |
SistemasDistribuidos–Clase12 30/61

Tiempo de detección
La alerta de 1 h suena cuando el promedio de la hora llega a 1,44%. Con especiales al
10% sin reintentos (e = 41%): unos 2 minutos. Caída total: menos de 1 minuto.
| Varias ventanas, | varios  | umbrales |         |        |      |                |     |
| ---------------- | ------- | -------- | ------- | ------ | ---- | -------------- | --- |
| Severidad        | Ventana | larga    | Ventana | corta  | Burn | rate Consumido |     |
| page             |         | 1        | h       | 5 min  |      | 14,4           | 2%  |
| page             |         | 6        | h       | 30 min |      | 6              | 5%  |
| Ticket           |         | 3 días   |         | 6      | h    | 1              | 10% |
Si un burn rate b dura un tiempo t (acá, la ventana larga), se consume b·t/T: 14,4×1/720=2%,
6×6/720=5%, 1×72/720=10%. La ventana corta mide la duodécima parte de la larga.
SistemasDistribuidos–Clase12 30/61

| Varias ventanas, | varios  | umbrales |         |        |      |                |     |
| ---------------- | ------- | -------- | ------- | ------ | ---- | -------------- | --- |
| Severidad        | Ventana | larga    | Ventana | corta  | Burn | rate Consumido |     |
| page             |         | 1        | h       | 5 min  |      | 14,4           | 2%  |
| page             |         | 6        | h       | 30 min |      | 6              | 5%  |
| Ticket           |         | 3 días   |         | 6      | h    | 1              | 10% |
Si un burn rate b dura un tiempo t (acá, la ventana larga), se consume b·t/T: 14,4×1/720=2%,
6×6/720=5%, 1×72/720=10%. La ventana corta mide la duodécima parte de la larga.
Tiempo de detección
La alerta de 1 h suena cuando el promedio de la hora llega a 1,44%. Con especiales al
10% sin reintentos (e = 41%): unos 2 minutos. Caída total: menos de 1 minuto.
SistemasDistribuidos–Clase12 30/61

▶
|     |     |     | Generar tráfico | artificial | para tener | más señal. |     |
| --- | --- | --- | --------------- | ---------- | ---------- | ---------- | --- |
▶ Medir juntos servicios que fallan juntos (comparten una base de datos).
▶ Cambiar el producto: reintentos del cliente con backoff y jitter, alternativas.
▶
|           |          |         | Bajar el SLO | o agrandar | la ventana: | decisión | de producto. |
| --------- | -------- | ------- | ------------ | ---------- | ----------- | -------- | ------------ |
| Servicios | con poco | tráfico |              |            |             |          |              |
10 solicitudes por hora con SLO de 99,9%: el presupuesto es de unas 7 fallas en 30 días.
Una sola falla en una hora es un 10% de error, burn rate 100, y gasta el 13,9% del mes.
SistemasDistribuidos–Clase12 31/61

| Servicios | con poco | tráfico |     |     |
| --------- | -------- | ------- | --- | --- |
10 solicitudes por hora con SLO de 99,9%: el presupuesto es de unas 7 fallas en 30 días.
Una sola falla en una hora es un 10% de error, burn rate 100, y gasta el 13,9% del mes.
▶
| Generar | tráfico artificial | para tener | más señal. |     |
| ------- | ------------------ | ---------- | ---------- | --- |
▶ Medir juntos servicios que fallan juntos (comparten una base de datos).
▶ Cambiar el producto: reintentos del cliente con backoff y jitter, alternativas.
▶
| Bajar | el SLO o agrandar | la ventana: | decisión | de producto. |
| ----- | ----------------- | ----------- | -------- | ------------ |
SistemasDistribuidos–Clase12 31/61

Contar minutos exagera algunos problemas y oculta otros. En un sistema distribuido lo habitual es
la degradación parcial (brownout).
▶ Solicitudes: paraoperarelservicioycalcularelburnrate,sobretodoconSLOsde99,9%omás.
▶ Minutos: paraunSLAounapáginadeestado(“12minutoscaídoestemes”),conmuchotráficoy
umbralespensadosparacaídas.
| Contar minutos | o contar     | solicitudes |               |                |      |
| -------------- | ------------ | ----------- | ------------- | -------------- | ---- |
| Qué pasa       | Contando     | solicitudes | Contando      | minutos        |      |
|                | presupuesto: | 12960fallas | malo si falla | el 0,1% o más; | pre- |
supuesto: 43,2minutos
Durante 1 hora falla el 1%: 180 180/12960 ≈ 1,4% del pre- Los 60 minutos son malos: se
| de18000. | supuesto. |                              | agota. |     |       |
| -------- | --------- | ---------------------------- | ------ | --- | ----- |
|          |           | SistemasDistribuidos–Clase12 |        |     | 32/61 |

| Contar minutos | o contar     | solicitudes |               |                |      |
| -------------- | ------------ | ----------- | ------------- | -------------- | ---- |
| Qué pasa       | Contando     | solicitudes | Contando      | minutos        |      |
|                | presupuesto: | 12960fallas | malo si falla | el 0,1% o más; | pre- |
supuesto: 43,2minutos
Durante 1 hora falla el 1%: 180 180/12960 ≈ 1,4% del pre- Los 60 minutos son malos: se
| de18000. | supuesto. |     | agota. |     |     |
| -------- | --------- | --- | ------ | --- | --- |
Contar minutos exagera algunos problemas y oculta otros. En un sistema distribuido lo habitual es
| la degradación parcial | (brownout). |     |     |     |     |
| ---------------------- | ----------- | --- | --- | --- | --- |
▶ Solicitudes: paraoperarelservicioycalcularelburnrate,sobretodoconSLOsde99,9%omás.
▶ Minutos: paraunSLAounapáginadeestado(“12minutoscaídoestemes”),conmuchotráficoy
umbralespensadosparacaídas.
|     |     | SistemasDistribuidos–Clase12 |     |     | 32/61 |
| --- | --- | ---------------------------- | --- | --- | ----- |

▶ Una madrugada falla el 1,5% durante 20 minutos: consumo 300 veces más rápido que el
|     |     |     |     | permitido. | Suena a la | 1:29. |     |
| --- | --- | --- | --- | ---------- | ---------- | ----- | --- |
▶
La guardia ve el servicio recuperado: “un pico”. Recién a la cuarta alerta, a las 9:55, se declara
un incidente.
▶ El monitoreo tradicional (sondas fallidas seguidas) nunca vio nada.
|     |     |     |     | La causa y | la lección |     |     |
| --- | --- | --- | --- | ---------- | ---------- | --- | --- |
Una pérdida de memoria: las máquinas caían y se reiniciaban casi juntas. Quedarse sin
memoria era normal; lo anómalo era el efecto sobre los usuarios. Gastaron más de cinco
veces el presupuesto, y para ellos la alerta del SLO pasó a ser la principal. El SRE Book es
|       |           |                 |     | más prudente: | conserva | las alertas | por causas inminentes. |
| ----- | --------- | --------------- | --- | ------------- | -------- | ----------- | ---------------------- |
| Caso: | la alerta | que nadie creía |     |               |          |             |                        |
Honeycomb: servicio de ingesta de datos de clientes, SLO de 99,995%, con alerta de
| consumo | del presupuesto. | El equipo todavía | no le creía. |     |     |     |     |
| ------- | ---------------- | ----------------- | ------------ | --- | --- | --- | --- |
SistemasDistribuidos–Clase12 33/61

|     |     |     |     | La causa y | la lección |     |     |
| --- | --- | --- | --- | ---------- | ---------- | --- | --- |
Una pérdida de memoria: las máquinas caían y se reiniciaban casi juntas. Quedarse sin
memoria era normal; lo anómalo era el efecto sobre los usuarios. Gastaron más de cinco
veces el presupuesto, y para ellos la alerta del SLO pasó a ser la principal. El SRE Book es
|       |           |                 |     | más prudente: | conserva | las alertas | por causas inminentes. |
| ----- | --------- | --------------- | --- | ------------- | -------- | ----------- | ---------------------- |
| Caso: | la alerta | que nadie creía |     |               |          |             |                        |
Honeycomb: servicio de ingesta de datos de clientes, SLO de 99,995%, con alerta de
| consumo | del presupuesto. | El equipo todavía | no le creía. |     |     |     |     |
| ------- | ---------------- | ----------------- | ------------ | --- | --- | --- | --- |
▶ Una madrugada falla el 1,5% durante 20 minutos: consumo 300 veces más rápido que el
| permitido. | Suena a | la 1:29. |     |     |     |     |     |
| ---------- | ------- | -------- | --- | --- | --- | --- | --- |
▶
La guardia ve el servicio recuperado: “un pico”. Recién a la cuarta alerta, a las 9:55, se declara
un incidente.
▶ El monitoreo tradicional (sondas fallidas seguidas) nunca vio nada.
SistemasDistribuidos–Clase12 33/61

| Caso: | la alerta | que nadie | creía |
| ----- | --------- | --------- | ----- |
Honeycomb: servicio de ingesta de datos de clientes, SLO de 99,995%, con alerta de
| consumo | del presupuesto. | El equipo | todavía no le creía. |
| ------- | ---------------- | --------- | -------------------- |
▶ Una madrugada falla el 1,5% durante 20 minutos: consumo 300 veces más rápido que el
| permitido. | Suena a | la 1:29. |     |
| ---------- | ------- | -------- | --- |
▶
La guardia ve el servicio recuperado: “un pico”. Recién a la cuarta alerta, a las 9:55, se declara
un incidente.
▶ El monitoreo tradicional (sondas fallidas seguidas) nunca vio nada.
| La causa | y la lección |     |     |
| -------- | ------------ | --- | --- |
Una pérdida de memoria: las máquinas caían y se reiniciaban casi juntas. Quedarse sin
memoria era normal; lo anómalo era el efecto sobre los usuarios. Gastaron más de cinco
veces el presupuesto, y para ellos la alerta del SLO pasó a ser la principal. El SRE Book es
| más prudente: | conserva | las alertas | por causas inminentes. |
| ------------- | -------- | ----------- | ---------------------- |
SistemasDistribuidos–Clase12 33/61

Validar:
| experimentos | bajo | falla |
| ------------ | ---- | ----- |
Sistemas Distribuidos

La seguridad no se compone
Un cliente con timeout de 50ms llama a dos proveedores: 20ms en promedio, casi nunca
más de 30ms. Cada uno, solo, es seguro. Los dos en secuencia cumplen en promedio
(40ms), pero cuando los dos se demoran un poco a la vez, la suma pasa los 50ms.
El generador hace 20 llamadas por solicitud, cada una razonable por separado.
▶ Las pruebas unitarias y de integración suponen tiempos ideales y mocks limpios.
▶ Staging nunca es producción: menos servidores, menos tráfico, red sin congestión.
Por qué no alcanza con probar los componentes
La clase 11 configuró timeouts, reintentos y breakers. ¿Cómo sabemos que funcionan?
SistemasDistribuidos–Clase12 35/61

▶ Las pruebas unitarias y de integración suponen tiempos ideales y mocks limpios.
▶ Staging nunca es producción: menos servidores, menos tráfico, red sin congestión.
Por qué no alcanza con probar los componentes
La clase 11 configuró timeouts, reintentos y breakers. ¿Cómo sabemos que funcionan?
La seguridad no se compone
Un cliente con timeout de 50ms llama a dos proveedores: 20ms en promedio, casi nunca
más de 30ms. Cada uno, solo, es seguro. Los dos en secuencia cumplen en promedio
(40ms), pero cuando los dos se demoran un poco a la vez, la suma pasa los 50ms.
El generador hace 20 llamadas por solicitud, cada una razonable por separado.
SistemasDistribuidos–Clase12 35/61

Por qué no alcanza con probar los componentes
La clase 11 configuró timeouts, reintentos y breakers. ¿Cómo sabemos que funcionan?
La seguridad no se compone
Un cliente con timeout de 50ms llama a dos proveedores: 20ms en promedio, casi nunca
más de 30ms. Cada uno, solo, es seguro. Los dos en secuencia cumplen en promedio
(40ms), pero cuando los dos se demoran un poco a la vez, la suma pasa los 50ms.
El generador hace 20 llamadas por solicitud, cada una razonable por separado.
▶ Las pruebas unitarias y de integración suponen tiempos ideales y mocks limpios.
▶ Staging nunca es producción: menos servidores, menos tráfico, red sin congestión.
SistemasDistribuidos–Clase12 35/61

¿Por qué provocar fallas a propósito?
▶ Los sistemas pierden margen sin que nadie lo decida: cada optimización consume holgura.
Un pool justo para 5 solicitudes/s anda bien hasta que especiales se pone lento.
▶ Lo que nunca falla no se sabe manejar: nadie sabe cómo se comporta el resto sin ese
componente.
▶ La falla pequeña y frecuente se vuelve rutina: como un simulacro de evacuación.
Chaos Monkey (Netflix): apaga al azar un servidor de un servicio. Si el servicio no sigue con los
demás, el equipo lo corrige.
Chaos Engineering
Definición
Hacer experimentos sobre un sistema para ganar confianza en que soporta condiciones
turbulentas en producción. No deduce qué debería pasar: provoca la falla de forma
controlada y observa qué pasa.
SistemasDistribuidos–Clase12 36/61

Chaos Monkey (Netflix): apaga al azar un servidor de un servicio. Si el servicio no sigue con los
demás, el equipo lo corrige.
Chaos Engineering
Definición
Hacer experimentos sobre un sistema para ganar confianza en que soporta condiciones
turbulentas en producción. No deduce qué debería pasar: provoca la falla de forma
controlada y observa qué pasa.
¿Por qué provocar fallas a propósito?
▶ Los sistemas pierden margen sin que nadie lo decida: cada optimización consume holgura.
Un pool justo para 5 solicitudes/s anda bien hasta que especiales se pone lento.
▶ Lo que nunca falla no se sabe manejar: nadie sabe cómo se comporta el resto sin ese
componente.
▶ La falla pequeña y frecuente se vuelve rutina: como un simulacro de evacuación.
SistemasDistribuidos–Clase12 36/61

| Chaos | Engineering |     |
| ----- | ----------- | --- |
Definición
Hacer experimentos sobre un sistema para ganar confianza en que soporta condiciones
turbulentas en producción. No deduce qué debería pasar: provoca la falla de forma
| controlada | y observa       | qué pasa.    |
| ---------- | --------------- | ------------ |
| ¿Por qué   | provocar fallas | a propósito? |
▶ Los sistemas pierden margen sin que nadie lo decida: cada optimización consume holgura.
Un pool justo para 5 solicitudes/s anda bien hasta que especiales se pone lento.
▶
Lo que nunca falla no se sabe manejar: nadie sabe cómo se comporta el resto sin ese
componente.
▶
La falla pequeña y frecuente se vuelve rutina: como un simulacro de evacuación.
Chaos Monkey (Netflix): apaga al azar un servidor de un servicio. Si el servicio no sigue con los
| demás, | el equipo lo corrige. |     |
| ------ | --------------------- | --- |
SistemasDistribuidos–Clase12 36/61

El steady state es el SLI
Es exactamente lo que construimos al medir. Sin SLIs no hay qué observar. Por eso
primero medimos.
Además: hipótesis sobre lo que se ve desde afuera · fallas que ocurren en la realidad · en
producción cuando sea seguro · automatizado · con el daño acotado (blast radius).
Un experimento tiene cuatro pasos
1. Definir el steady state: una salida medible que indica comportamiento normal.
2. Formular la hipótesis: el steady state se mantiene en un grupo de control y en uno
experimental.
3. Inyectar fallas (fault injection) que reflejen eventos reales.
4. Intentar refutar la hipótesis: buscar una diferencia entre los grupos.
SistemasDistribuidos–Clase12 37/61

Además: hipótesis sobre lo que se ve desde afuera · fallas que ocurren en la realidad · en
producción cuando sea seguro · automatizado · con el daño acotado (blast radius).
Un experimento tiene cuatro pasos
1. Definir el steady state: una salida medible que indica comportamiento normal.
2. Formular la hipótesis: el steady state se mantiene en un grupo de control y en uno
experimental.
3. Inyectar fallas (fault injection) que reflejen eventos reales.
4. Intentar refutar la hipótesis: buscar una diferencia entre los grupos.
El steady state es el SLI
Es exactamente lo que construimos al medir. Sin SLIs no hay qué observar. Por eso
primero medimos.
SistemasDistribuidos–Clase12 37/61

Un experimento tiene cuatro pasos
1. Definir el steady state: una salida medible que indica comportamiento normal.
2. Formular la hipótesis: el steady state se mantiene en un grupo de control y en uno
experimental.
3. Inyectar fallas (fault injection) que reflejen eventos reales.
4. Intentar refutar la hipótesis: buscar una diferencia entre los grupos.
El steady state es el SLI
Es exactamente lo que construimos al medir. Sin SLIs no hay qué observar. Por eso
primero medimos.
Además: hipótesis sobre lo que se ve desde afuera · fallas que ocurren en la realidad · en
producción cuando sea seguro · automatizado · con el daño acotado (blast radius).
SistemasDistribuidos–Clase12 37/61

|     |     |     |     |     | Dos | preguntas | antes de | empezar: |     |
| --- | --- | --- | --- | --- | --- | --------- | -------- | -------- | --- |
▶ ¿Se puede medir hoy el steady state? Si el SLI saliera solo de los logs del generador, un
|     |     |     |     |     |     | experimento | que lo tumba | no mostraría | ninguna falla. |
| --- | --- | --- | --- | --- | --- | ----------- | ------------ | ------------ | -------------- |
▶ ¿Qué resultado la refutaría? Siempre hay fallas por la red del usuario: lo que importa es la
|     |     |     |     |     |     | diferencia | con el grupo | de control. |     |
| --- | --- | --- | --- | --- | --- | ---------- | ------------ | ----------- | --- |
La hipótesis
Mala
Buena
| “Queremos | ver si aguanta.” |               |                |                 |                 |     |     |     |     |
| --------- | ---------------- | ------------- | -------------- | --------------- | --------------- | --- | --- | --- | --- |
|           |                  |               | “Si especiales | se pone lenta,  | las solicitudes |     |     |     |     |
| No dice   | qué observar ni  | qué resultado | la refuta.     |                 |                 |     |     |     |     |
|           |                  |               | afectadas      | fallan rápido y | las demás       |     |     |     |     |
mantienen su SLI.”
|     |     |     | Observable | desde afuera. Refutable. |     |     |     |     |     |
| --- | --- | --- | ---------- | ------------------------ | --- | --- | --- | --- | --- |
SistemasDistribuidos–Clase12 38/61

La hipótesis
Mala
Buena
| “Queremos     | ver          | si aguanta.” |           |            |                |                          |                 |
| ------------- | ------------ | ------------ | --------- | ---------- | -------------- | ------------------------ | --------------- |
|               |              |              |           |            | “Si especiales | se pone lenta,           | las solicitudes |
| No dice       | qué observar | ni qué       | resultado | la refuta. |                |                          |                 |
|               |              |              |           |            | afectadas      | fallan rápido y          | las demás       |
|               |              |              |           |            | mantienen      | su SLI.”                 |                 |
|               |              |              |           |            | Observable     | desde afuera. Refutable. |                 |
| Dos preguntas |              | antes de     | empezar:  |            |                |                          |                 |
▶ ¿Se puede medir hoy el steady state? Si el SLI saliera solo de los logs del generador, un
| experimento |     | que lo tumba | no mostraría | ninguna | falla. |     |     |
| ----------- | --- | ------------ | ------------ | ------- | ------ | --- | --- |
▶ ¿Qué resultado la refutaría? Siempre hay fallas por la red del usuario: lo que importa es la
| diferencia |     | con el grupo | de control. |     |     |     |     |
| ---------- | --- | ------------ | ----------- | --- | --- | --- | --- |
SistemasDistribuidos–Clase12 38/61

|     |     |     |     | Y también: | pérdida de      | paquetes, particiones, | CPU, disco. |
| --- | --- | --- | --- | ---------- | --------------- | ---------------------- | ----------- |
|     |     |     |     | Probar     | las dos: rápida | y lenta                |             |
Una caída es una falla rápida; la latencia, una lenta. Muchos sistemas toleran que una
dependencia desaparezca y muy mal que se ponga lenta (clase 11).
Al principio, al azar. Después, mirar el árbol de llamadas de una traza y preguntar qué pasa si se
|     |     |     |     | quita cada | una. |     |     |
| --- | --- | --- | --- | ---------- | ---- | --- | --- |
Qué inyectar
| Apagar servidores. | ¿El servicio | sigue con | los que quedan? |     |     |     |     |
| ------------------ | ------------ | --------- | --------------- | --- | --- | --- | --- |
Agregar latencia. Encuentra lo que apagar no muestra: servicios sin alternativa ante un timeout,
errores que aparecen cuando las respuestas llegan en otro orden.
Hacer fallar una llamada. La solicitud se marca al entrar (“cuando el generador llame a
especiales, fallar”) y la marca viaja como el contexto de una traza (clase 10).
|     | Permite afectar | una sola llamada, | para pocas solicitudes. |     |     |     |     |
| --- | --------------- | ----------------- | ----------------------- | --- | --- | --- | --- |
SistemasDistribuidos–Clase12 39/61

Qué inyectar
| Apagar | servidores. | ¿El servicio | sigue con | los que quedan? |
| ------ | ----------- | ------------ | --------- | --------------- |
Agregar latencia. Encuentra lo que apagar no muestra: servicios sin alternativa ante un timeout,
errores que aparecen cuando las respuestas llegan en otro orden.
Hacer fallar una llamada. La solicitud se marca al entrar (“cuando el generador llame a
especiales, fallar”) y la marca viaja como el contexto de una traza (clase 10).
|            | Permite  | afectar      | una sola llamada, | para pocas solicitudes. |
| ---------- | -------- | ------------ | ----------------- | ----------------------- |
| Y también: | pérdida  | de paquetes, | particiones,      | CPU, disco.             |
| Probar     | las dos: | rápida       | y lenta           |                         |
Una caída es una falla rápida; la latencia, una lenta. Muchos sistemas toleran que una
dependencia desaparezca y muy mal que se ponga lenta (clase 11).
Al principio, al azar. Después, mirar el árbol de llamadas de una traza y preguntar qué pasa si se
| quita cada | una. |     |     |     |
| ---------- | ---- | --- | --- | --- |
SistemasDistribuidos–Clase12 39/61

El experimento consume presupuesto
Perturbar el 1% del tráfico, y que todo eso falle, es burn rate 10. Treinta minutos:
10×0,5/720 ≈ 0,7% del mes. Con poco presupuesto, la política puede postergar el
experimento.
Criterio de aborto en el idioma de las alertas: se detiene si el burn rate de la ventana corta supera
lo esperado.
Seguridad del experimento
▶ Si cada solicitud fallida es irrecuperable (un pago, un dato médico), no se experimenta en
producción.
▶ Acotar el blast radius: empezar por “falla una de cada 10000 solicitudes”.
▶ Poder seguir una solicitud entre servicios y conocer la línea de base del SLI.
▶ Tener un plan de recuperación: apagar la perturbación no siempre alcanza.
▶ Escalera: local → integración → poco tráfico real → producción.
SistemasDistribuidos–Clase12 40/61

| Seguridad | del experimento |     |     |
| --------- | --------------- | --- | --- |
▶
Si cada solicitud fallida es irrecuperable (un pago, un dato médico), no se experimenta en
producción.
▶
Acotar el blast radius: empezar por “falla una de cada 10000 solicitudes”.
▶
Poder seguir una solicitud entre servicios y conocer la línea de base del SLI.
▶
Tener un plan de recuperación: apagar la perturbación no siempre alcanza.
| ▶               | →                   | →            | →                |
| --------------- | ------------------- | ------------ | ---------------- |
| Escalera: local | integración         | poco tráfico | real producción. |
| El experimento  | consume presupuesto |              |                  |
Perturbar el 1% del tráfico, y que todo eso falle, es burn rate 10. Treinta minutos:
| 10×0,5/720 ≈ |     |     |     |
| ------------ | --- | --- | --- |
0,7% del mes. Con poco presupuesto, la política puede postergar el
experimento.
Criterio de aborto en el idioma de las alertas: se detiene si el burn rate de la ventana corta supera
lo esperado.
SistemasDistribuidos–Clase12 40/61

|     |     |     |     | “Si nadie responde, | empiezo” |     |
| --- | --- | --- | --- | ------------------- | -------- | --- |
Avisó por correo que iba a experimentar en el entorno de pruebas. Casi nadie respondió. El
|     |     |     |     | entorno quedó | inutilizable | una semana. |
| --- | --- | --- | --- | ------------- | ------------ | ----------- |
Lección
Probar la reversión antes. El silencio no es consentimiento. Los primeros experimentos,
chicos y acordados.
Dos advertencias
| Una prueba | de Google | que se salió | de control |     |     |     |
| ---------- | --------- | ------------ | ---------- | --- | --- | --- |
Bloquearon el acceso a una base de datos de prueba entre cien. En minutos, usuarios
internos y externos perdieron acceso a sistemas clave. Abortaron, pero la reversión falló:
| nunca se | había probado. |     |     |     |     |     |
| -------- | -------------- | --- | --- | --- | --- | --- |
SistemasDistribuidos–Clase12 41/61

Lección
Probar la reversión antes. El silencio no es consentimiento. Los primeros experimentos,
chicos y acordados.
Dos advertencias
| Una prueba | de Google | que se salió | de control |
| ---------- | --------- | ------------ | ---------- |
Bloquearon el acceso a una base de datos de prueba entre cien. En minutos, usuarios
internos y externos perdieron acceso a sistemas clave. Abortaron, pero la reversión falló:
| nunca se  | había probado. |          |     |
| --------- | -------------- | -------- | --- |
| “Si nadie | responde,      | empiezo” |     |
Avisó por correo que iba a experimentar en el entorno de pruebas. Casi nadie respondió. El
| entorno quedó | inutilizable | una semana. |     |
| ------------- | ------------ | ----------- | --- |
SistemasDistribuidos–Clase12 41/61

Dos advertencias
| Una prueba | de Google | que se salió | de control |
| ---------- | --------- | ------------ | ---------- |
Bloquearon el acceso a una base de datos de prueba entre cien. En minutos, usuarios
internos y externos perdieron acceso a sistemas clave. Abortaron, pero la reversión falló:
| nunca se  | había probado. |          |     |
| --------- | -------------- | -------- | --- |
| “Si nadie | responde,      | empiezo” |     |
Avisó por correo que iba a experimentar en el entorno de pruebas. Casi nadie respondió. El
| entorno quedó | inutilizable | una semana. |     |
| ------------- | ------------ | ----------- | --- |
Lección
Probar la reversión antes. El silencio no es consentimiento. Los primeros experimentos,
chicos y acordados.
SistemasDistribuidos–Clase12 41/61

▶ Si encuentra una debilidad: corregirla y buscar el mismo tipo de problema en otras
partes.
▶ Después, automatizarlo para que corra periódicamente. Con límites: no apagar el
último servidor; no hacer fallar a la vez una dependencia y su alternativa.
Las personas también fallan
Simulacros en los que parte del equipo queda “fuera” un día: aparecen los procesos que
dependen de una sola persona. Google los hace periódicamente con el nombre DiRT
(Disaster Recovery Testing), e incluye la práctica de la gestión de incidentes. Siempre con
una palabra clave para abortar.
Automatizar y repetir
En el mejor caso, un experimento es aburrido: se inyecta la falla y el sistema sigue igual.
SistemasDistribuidos–Clase12 42/61

Las personas también fallan
Simulacros en los que parte del equipo queda “fuera” un día: aparecen los procesos que
dependen de una sola persona. Google los hace periódicamente con el nombre DiRT
(Disaster Recovery Testing), e incluye la práctica de la gestión de incidentes. Siempre con
una palabra clave para abortar.
Automatizar y repetir
En el mejor caso, un experimento es aburrido: se inyecta la falla y el sistema sigue igual.
▶ Si encuentra una debilidad: corregirla y buscar el mismo tipo de problema en otras
partes.
▶ Después, automatizarlo para que corra periódicamente. Con límites: no apagar el
último servidor; no hacer fallar a la vez una dependencia y su alternativa.
SistemasDistribuidos–Clase12 42/61

Automatizar y repetir
En el mejor caso, un experimento es aburrido: se inyecta la falla y el sistema sigue igual.
▶ Si encuentra una debilidad: corregirla y buscar el mismo tipo de problema en otras
partes.
▶ Después, automatizarlo para que corra periódicamente. Con límites: no apagar el
último servidor; no hacer fallar a la vez una dependencia y su alternativa.
Las personas también fallan
Simulacros en los que parte del equipo queda “fuera” un día: aparecen los procesos que
dependen de una sola persona. Google los hace periódicamente con el nombre DiRT
(Disaster Recovery Testing), e incluye la práctica de la gestión de incidentes. Siempre con
una palabra clave para abortar.
SistemasDistribuidos–Clase12 42/61

| Template | del experimento |     |
| -------- | --------------- | --- |
Escribir el diseño antes: obliga a decidir lo que se improvisaría mal, se comparte con los
| equipos afectados | y termina siendo                           | el informe. |
| ----------------- | ------------------------------------------ | ----------- |
| Campo             | Contenido                                  |             |
| Pregunta          | Quémecanismoosupuestosevalida.             |             |
| Steadystate       | SLIusadoysuvalorenlalíneadebase.           |             |
| Hipótesis         | Quéseespera,escritodeformarefutable.       |             |
| Perturbación      | Quéseinyecta,dónde,cuántoyporcuántotiempo. |             |
| Grupos            | Quétráficoescontrolycuálexperimental.      |             |
| Blastradius       | Quéporcentajedeltráficoyquécomponentes.    |             |
| Costoesperado     | Cuántoerrorbudgetseconsume.                |             |
| Criteriodeaborto  | Quéseñallodetieneyquiénlodetiene.          |             |
| Recuperación      | Quéreiniciar,revertirolimpiar.             |             |
Resultado Quéseobservó,silahipótesissesostuvo,quéaccionessiguen.
SistemasDistribuidos–Clase12 43/61

Qué la refutaría
Que suba la latencia del grupo de control: los hilos se comparten (falta un bulkhead o el timeout es
largo). Que las marcadas tarden 2s en fallar: el timeout no está activo.
| Un experimento |     | sobre | el generador |     |
| -------------- | --- | ----- | ------------ | --- |
Pregunta: ¿los timeouts y el breaker de la clase 11 contienen una dependencia lenta?
Steady state: SLIs de disponibilidad y latencia, en el balanceador.
Hipótesis: si las llamadas a especiales del 1% de las solicitudes tardan 2s más, esas fallan
|     | rápido y | con error explícito, | y el resto mantiene | sus SLIs. |
| --- | -------- | -------------------- | ------------------- | --------- |
Grupos: latencia solo en las solicitudes marcadas (1%); el resto es control. 30 minutos.
| Costo: | burn rate | ≈10; ≈0,7% | del presupuesto. |     |
| ------ | --------- | ---------- | ---------------- | --- |
Aborto: burn rate de 5 minutos >14,4, o el grupo de control fuera de su línea de base.
SistemasDistribuidos–Clase12 44/61

| Un experimento |     | sobre | el generador |     |
| -------------- | --- | ----- | ------------ | --- |
Pregunta: ¿los timeouts y el breaker de la clase 11 contienen una dependencia lenta?
Steady state: SLIs de disponibilidad y latencia, en el balanceador.
Hipótesis: si las llamadas a especiales del 1% de las solicitudes tardan 2s más, esas fallan
|     | rápido y | con error explícito, | y el resto mantiene | sus SLIs. |
| --- | -------- | -------------------- | ------------------- | --------- |
Grupos: latencia solo en las solicitudes marcadas (1%); el resto es control. 30 minutos.
| Costo: | burn rate | ≈10; ≈0,7% | del presupuesto. |     |
| ------ | --------- | ---------- | ---------------- | --- |
Aborto: burn rate de 5 minutos >14,4, o el grupo de control fuera de su línea de base.
Qué la refutaría
Que suba la latencia del grupo de control: los hilos se comparten (falta un bulkhead o el timeout es
largo). Que las marcadas tarden 2s en fallar: el timeout no está activo.
SistemasDistribuidos–Clase12 44/61

mayúsculas
minúsculas
solicitud
Usuario Generador !
dígitos
especiales
Se refuta: las llamadas a minúsculas también se vuelven lentas, porque minúsculas consulta a
dígitos por dentro. El experimento descubre la dependencia que no estaba en el diagrama.
Acción: documentarla y revisar que el timeout de minúsculas contemple el de dígitos (clase 11).
Lo que no se esperaba
Mismo diseño sobre dígitos. Hipótesis: “solo se ven afectadas las llamadas a dígitos”.
SistemasDistribuidos–Clase12 45/61

Se refuta: las llamadas a minúsculas también se vuelven lentas, porque minúsculas consulta a
dígitos por dentro. El experimento descubre la dependencia que no estaba en el diagrama.
Acción: documentarla y revisar que el timeout de minúsculas contemple el de dígitos (clase 11).
Lo que no se esperaba
Mismo diseño sobre dígitos. Hipótesis: “solo se ven afectadas las llamadas a dígitos”.
mayúsculas
minúsculas
solicitud
Usuario Generador !
dígitos
especiales
SistemasDistribuidos–Clase12 45/61

Lo que no se esperaba
Mismo diseño sobre dígitos. Hipótesis: “solo se ven afectadas las llamadas a dígitos”.
mayúsculas
minúsculas
solicitud
Usuario Generador !
dígitos
especiales
Se refuta: las llamadas a minúsculas también se vuelven lentas, porque minúsculas consulta a
dígitos por dentro. El experimento descubre la dependencia que no estaba en el diagrama.
Acción: documentarla y revisar que el timeout de minúsculas contemple el de dígitos (clase 11).
SistemasDistribuidos–Clase12 45/61

Responder:
gestión de incidentes
Sistemas Distribuidos

|     |     |     |     | Condiciones     | claras, | fijadas de antemano |     |
| --- | --- | --- | --- | --------------- | ------- | ------------------- | --- |
|     |     |     |     | Es un incidente | si:     |                     |     |
▶
|     |     |     |     | hace | falta involucrar | a un segundo       | equipo; |
| --- | --- | --- | --- | ---- | ---------------- | ------------------ | ------- |
|     |     |     |     | ▶ la | caída es visible | para los clientes; |         |
▶ sigue sin resolverse tras una hora de análisis concentrado.
Con SLOs: una alerta de burn rate page suele ser el comienzo de un incidente. Mejor declararlo
|     |                 |          |            | temprano | y cerrarlo rápido | que tarde. |     |
| --- | --------------- | -------- | ---------- | -------- | ----------------- | ---------- | --- |
| Qué | es un incidente | y cuándo | declararlo |          |                   |            |     |
Un evento que degrada o amenaza el servicio y requiere una respuesta coordinada. Una
| solicitud | fallida aislada no | lo es. |     |     |     |     |     |
| --------- | ------------------ | ------ | --- | --- | --- | --- | --- |
SistemasDistribuidos–Clase12 47/61

Con SLOs: una alerta de burn rate page suele ser el comienzo de un incidente. Mejor declararlo
|     |                 |          |            | temprano | y cerrarlo rápido | que tarde. |
| --- | --------------- | -------- | ---------- | -------- | ----------------- | ---------- |
| Qué | es un incidente | y cuándo | declararlo |          |                   |            |
Un evento que degrada o amenaza el servicio y requiere una respuesta coordinada. Una
| solicitud       | fallida aislada | no lo es.           |     |     |     |     |
| --------------- | --------------- | ------------------- | --- | --- | --- | --- |
| Condiciones     | claras,         | fijadas de antemano |     |     |     |     |
| Es un incidente | si:             |                     |     |     |     |     |
▶
| hace       | falta involucrar | a un segundo       | equipo; |     |     |     |
| ---------- | ---------------- | ------------------ | ------- | --- | --- | --- |
| ▶ la caída | es visible       | para los clientes; |         |     |     |     |
▶ sigue sin resolverse tras una hora de análisis concentrado.
SistemasDistribuidos–Clase12 47/61

| Qué | es un incidente | y cuándo | declararlo |
| --- | --------------- | -------- | ---------- |
Un evento que degrada o amenaza el servicio y requiere una respuesta coordinada. Una
| solicitud       | fallida aislada | no lo es.           |     |
| --------------- | --------------- | ------------------- | --- |
| Condiciones     | claras,         | fijadas de antemano |     |
| Es un incidente | si:             |                     |     |
▶
| hace | falta involucrar | a un segundo       | equipo; |
| ---- | ---------------- | ------------------ | ------- |
| ▶ la | caída es visible | para los clientes; |         |
▶ sigue sin resolverse tras una hora de análisis concentrado.
Con SLOs: una alerta de burn rate page suele ser el comienzo de un incidente. Mejor declararlo
| temprano | y cerrarlo rápido | que tarde. |     |
| -------- | ----------------- | ---------- | --- |
SistemasDistribuidos–Clase12 47/61

Foco exclusivo en lo técnico: quien sabe arreglar no puede ver el cuadro general.
| Mala comunicación: | nadie | sabe qué | hacen los demás. |
| ------------------ | ----- | -------- | ---------------- |
Trabajo por cuenta propia: cambios con buena intención, sin coordinar.
| Todos hacían | su trabajo como | lo entendían. | Falta un proceso. |
| ------------ | --------------- | ------------- | ----------------- |
El incidente no gestionado
Cae un centro de datos, después otro y otro. La guardia revierte sin éxito, llama a quien
escribió el código (3:30 AM en su zona), otros “miran” desde sus terminales, los directivos
piden fechas. Alguien despliega una optimización que se le ocurrió en el momento. Los
servidores mueren.
SistemasDistribuidos–Clase12 48/61

|              |               |     |     | Todos hacían | su trabajo como | lo entendían. | Falta un proceso. |
| ------------ | ------------- | --- | --- | ------------ | --------------- | ------------- | ----------------- |
| El incidente | no gestionado |     |     |              |                 |               |                   |
Cae un centro de datos, después otro y otro. La guardia revierte sin éxito, llama a quien
escribió el código (3:30 AM en su zona), otros “miran” desde sus terminales, los directivos
piden fechas. Alguien despliega una optimización que se le ocurrió en el momento. Los
servidores mueren.
Foco exclusivo en lo técnico: quien sabe arreglar no puede ver el cuadro general.
| Mala comunicación: | nadie sabe | qué hacen | los demás. |     |     |     |     |
| ------------------ | ---------- | --------- | ---------- | --- | --- | --- | --- |
Trabajo por cuenta propia: cambios con buena intención, sin coordinar.
SistemasDistribuidos–Clase12 48/61

| El incidente | no gestionado |     |     |
| ------------ | ------------- | --- | --- |
Cae un centro de datos, después otro y otro. La guardia revierte sin éxito, llama a quien
escribió el código (3:30 AM en su zona), otros “miran” desde sus terminales, los directivos
piden fechas. Alguien despliega una optimización que se le ocurrió en el momento. Los
| servidores | mueren. |     |     |
| ---------- | ------- | --- | --- |
Foco exclusivo en lo técnico: quien sabe arreglar no puede ver el cuadro general.
| Mala comunicación: | nadie | sabe qué | hacen los demás. |
| ------------------ | ----- | -------- | ---------------- |
Trabajo por cuenta propia: cambios con buena intención, sin coordinar.
| Todos hacían | su trabajo como | lo entendían. | Falta un proceso. |
| ------------ | --------------- | ------------- | ----------------- |
SistemasDistribuidos–Clase12 48/61

Cada uno sabe su rol y no invade el ajeno: eso da más autonomía. En un equipo chico, una
persona tiene varios roles, pero explícitos.
Roles
Tomado de los servicios de emergencia (bomberos, defensa civil): muchas personas que no
trabajan juntas a diario.
Rol Responsabilidad
Incident commander Estado general, asigna tareas, remueve obstáculos. Tiene los roles
que no delegó.
Operaciones Aplica los cambios. Único grupo que toca el sistema.
Comunicación Actualiza periódicamente al equipo y a los interesados.
Planificación Lo de largo plazo: registrar, relevos, qué revertir al final.
SistemasDistribuidos–Clase12 49/61

Roles
Tomado de los servicios de emergencia (bomberos, defensa civil): muchas personas que no
| trabajan | juntas a diario. |                 |
| -------- | ---------------- | --------------- |
| Rol      |                  | Responsabilidad |
Incident commander Estado general, asigna tareas, remueve obstáculos. Tiene los roles
que no delegó.
Operaciones Aplica los cambios. Único grupo que toca el sistema.
Comunicación Actualiza periódicamente al equipo y a los interesados.
Planificación Lo de largo plazo: registrar, relevos, qué revertir al final.
Cada uno sabe su rol y no invade el ajeno: eso da más autonomía. En un equipo chico, una
| persona tiene | varios roles, | pero explícitos. |
| ------------- | ------------- | ---------------- |
SistemasDistribuidos–Clase12 49/61

El orden
Detener el daño, restablecer el servicio y conservar la evidencia. Primero se mitiga,
después se entiende. Durante el incidente no se discuten culpas.
|     |     |     | La mitigación | más frecuente: | revertir |     |
| --- | --- | --- | ------------- | -------------- | -------- | --- |
En Google, cerca del 70% de las caídas viene de un cambio. Un cambio de configuración
aplicado a todos los servidores a la vez los hizo caer en bucle; se revirtió en cinco minutos.
|           |                |               | No había pasado | un canary completo | porque | “no era riesgoso”. |
| --------- | -------------- | ------------- | --------------- | ------------------ | ------ | ------------------ |
| Documento | vivo, traspaso | y prioridades |                 |                    |        |                    |
▶
Puesto de mando: un lugar o canal donde encontrar al incident commander.
▶
Documento vivo: resumen, estado, roles, tareas, línea de tiempo y criterios de salida:
| “dentro de | los SLOs durante | más de 30 minutos”. |     |     |     |     |
| ---------- | ---------------- | ------------------- | --- | --- | --- | --- |
▶ Traspaso explícito: “ahora sos el incident commander, ¿de acuerdo?”.
SistemasDistribuidos–Clase12 50/61

|     |     |     | La mitigación | más frecuente: | revertir |     |
| --- | --- | --- | ------------- | -------------- | -------- | --- |
En Google, cerca del 70% de las caídas viene de un cambio. Un cambio de configuración
aplicado a todos los servidores a la vez los hizo caer en bucle; se revirtió en cinco minutos.
|           |                |               | No había pasado | un canary completo | porque | “no era riesgoso”. |
| --------- | -------------- | ------------- | --------------- | ------------------ | ------ | ------------------ |
| Documento | vivo, traspaso | y prioridades |                 |                    |        |                    |
▶
Puesto de mando: un lugar o canal donde encontrar al incident commander.
▶
Documento vivo: resumen, estado, roles, tareas, línea de tiempo y criterios de salida:
| “dentro de | los SLOs durante | más de 30 minutos”. |     |     |     |     |
| ---------- | ---------------- | ------------------- | --- | --- | --- | --- |
▶ Traspaso explícito: “ahora sos el incident commander, ¿de acuerdo?”.
El orden
Detener el daño, restablecer el servicio y conservar la evidencia. Primero se mitiga,
después se entiende. Durante el incidente no se discuten culpas.
SistemasDistribuidos–Clase12 50/61

| Documento | vivo, traspaso | y prioridades |     |
| --------- | -------------- | ------------- | --- |
▶
Puesto de mando: un lugar o canal donde encontrar al incident commander.
▶
Documento vivo: resumen, estado, roles, tareas, línea de tiempo y criterios de salida:
| “dentro | de los SLOs durante | más de 30 minutos”. |     |
| ------- | ------------------- | ------------------- | --- |
▶ Traspaso explícito: “ahora sos el incident commander, ¿de acuerdo?”.
El orden
Detener el daño, restablecer el servicio y conservar la evidencia. Primero se mitiga,
después se entiende. Durante el incidente no se discuten culpas.
| La mitigación | más frecuente: | revertir |     |
| ------------- | -------------- | -------- | --- |
En Google, cerca del 70% de las caídas viene de un cambio. Un cambio de configuración
aplicado a todos los servidores a la vez los hizo caer en bucle; se revirtió en cinco minutos.
| No había pasado | un canary | completo porque | “no era riesgoso”. |
| --------------- | --------- | --------------- | ------------------ |
SistemasDistribuidos–Clase12 50/61

Runbook breve del generador
Dueño,guardia,escalamiento. Dependede: mayúsculas,minúsculas(quedependededígitos),dígitos,especiales.
Dashboards: SLIs,latenciapordependencia,breakers.
Alerta“burnratealto,1h”: segastóel2%delmesenunahora. Confirmareneldashboard;mirarlatenciayerrorespor
dependencia. Seguro: revertirelúltimocambio. No: deshabilitartimeouts,niunfallbacksinespeciales. Escalaraldueñode
ladependencia.
Runbooks
A favor En contra
En Google, tener los pasos escritos reduce a un Un documento con todos los errores posibles
tercio el tiempo de recuperación. Se combina envejece rápido. Uno equivocado es peor que
con ensayar incidentes pasados. ninguno.
SistemasDistribuidos–Clase12 51/61

Runbooks
| A favor |     | En contra |
| ------- | --- | --------- |
En Google, tener los pasos escritos reduce a un Un documento con todos los errores posibles
tercio el tiempo de recuperación. Se combina envejece rápido. Uno equivocado es peor que
| con ensayar | incidentes pasados. | ninguno. |
| ----------- | ------------------- | -------- |
| Runbook     | breve del generador |          |
Dueño,guardia,escalamiento. Dependede: mayúsculas,minúsculas(quedependededígitos),dígitos,especiales.
| Dashboards: | SLIs,latenciapordependencia,breakers. |     |
| ----------- | ------------------------------------- | --- |
Alerta“burnratealto,1h”: segastóel2%delmesenunahora. Confirmareneldashboard;mirarlatenciayerrorespor
dependencia. Seguro: revertirelúltimocambio. No: deshabilitartimeouts,niunfallbacksinespeciales. Escalaraldueñode
ladependencia.
SistemasDistribuidos–Clase12 51/61

Aprender: postmortems
Sistemas Distribuidos

| Cuándo | es obligatorio | Qué contiene |     |
| ------ | -------------- | ------------ | --- |
▶
caídavisiblesobreunumbral; ▶ resumeneimpacto(conerrorbudget);
| ▶                        |     | ▶                         |              |
| ------------------------ | --- | ------------------------- | ------------ |
| pérdidadedatos;          |     | disparadorycondiciones;   |              |
| ▶                        |     | ▶                         |              |
| intervencióndelaguardia; |     | resoluciónydetección;     |              |
| ▶                        |     | ▶                         |              |
| resoluciónlenta;         |     | accionescondueño;         |              |
| ▶ falladelmonitoreo;     |     | ▶ quésalióbien,mal,ydónde | hubo suerte; |
| ▶                        |     | ▶                         |              |
| másdel20%delpresupuesto. |     | líneadetiempo.            |              |
“Dónde hubo suerte”: lo que salió bien sin que nadie lo garantizara. La próxima vez puede no estar.
Postmortem
Registro escrito del incidente: impacto, cómo se mitigó, causas y acciones para que no
se repita. No es un castigo.
SistemasDistribuidos–Clase12 53/61

“Dónde hubo suerte”: lo que salió bien sin que nadie lo garantizara. La próxima vez puede no estar.
Postmortem
Registro escrito del incidente: impacto, cómo se mitigó, causas y acciones para que no
| se repita. | No es un castigo. |              |     |
| ---------- | ----------------- | ------------ | --- |
| Cuándo     | es obligatorio    | Qué contiene |     |
▶ caídavisiblesobreunumbral; ▶ resumeneimpacto(conerrorbudget);
| ▶                        |     | ▶                         |              |
| ------------------------ | --- | ------------------------- | ------------ |
| pérdidadedatos;          |     | disparadorycondiciones;   |              |
| ▶                        |     | ▶                         |              |
| intervencióndelaguardia; |     | resoluciónydetección;     |              |
| ▶                        |     | ▶                         |              |
| resoluciónlenta;         |     | accionescondueño;         |              |
| ▶ falladelmonitoreo;     |     | ▶ quésalióbien,mal,ydónde | hubo suerte; |
| ▶                        |     | ▶                         |              |
| másdel20%delpresupuesto. |     | líneadetiempo.            |              |
SistemasDistribuidos–Clase12 53/61

Postmortem
Registro escrito del incidente: impacto, cómo se mitigó, causas y acciones para que no
| se repita. | No es un castigo. |              |     |
| ---------- | ----------------- | ------------ | --- |
| Cuándo     | es obligatorio    | Qué contiene |     |
▶ caídavisiblesobreunumbral; ▶ resumeneimpacto(conerrorbudget);
| ▶                        |     | ▶                         |              |
| ------------------------ | --- | ------------------------- | ------------ |
| pérdidadedatos;          |     | disparadorycondiciones;   |              |
| ▶                        |     | ▶                         |              |
| intervencióndelaguardia; |     | resoluciónydetección;     |              |
| ▶                        |     | ▶                         |              |
| resoluciónlenta;         |     | accionescondueño;         |              |
| ▶ falladelmonitoreo;     |     | ▶ quésalióbien,mal,ydónde | hubo suerte; |
| ▶                        |     | ▶                         |              |
| másdel20%delpresupuesto. |     | líneadetiempo.            |              |
“Dónde hubo suerte”: lo que salió bien sin que nadie lo garantizara. La próxima vez puede no estar.
SistemasDistribuidos–Clase12 53/61

▶
|     |     |     | Con culpa, | los problemas | se esconden. |
| --- | --- | --- | ---------- | ------------- | ------------ |
▶
Viene de la salud y la aviación: cada error fortalece el sistema.
▶
No se puede “arreglar” a las personas; sí los sistemas que las ayudan a decidir.
|     |     | La  | pregunta útil |     |     |
| --- | --- | --- | ------------- | --- | --- |
No “¿por qué se equivocó?”, sino “¿qué información, presiones y herramientas hicieron que
esa decisión pareciera razonable?”. Después, todo parece obvio: sesgo retrospectivo
(hindsight bias).
Blameless no es sin responsabilidad: igual señala qué mejorar, apoyado en el sistema.
Blameless
Buscar las causas sin acusar a personas ni equipos. Se supone que todos actuaron con
| buena intención | y con la información | que tenían. |     |     |     |
| --------------- | -------------------- | ----------- | --- | --- | --- |
SistemasDistribuidos–Clase12 54/61

La pregunta útil
No “¿por qué se equivocó?”, sino “¿qué información, presiones y herramientas hicieron que
esa decisión pareciera razonable?”. Después, todo parece obvio: sesgo retrospectivo
(hindsight bias).
Blameless no es sin responsabilidad: igual señala qué mejorar, apoyado en el sistema.
Blameless
Buscar las causas sin acusar a personas ni equipos. Se supone que todos actuaron con
buena intención y con la información que tenían.
▶ Con culpa, los problemas se esconden.
▶ Viene de la salud y la aviación: cada error fortalece el sistema.
▶ No se puede “arreglar” a las personas; sí los sistemas que las ayudan a decidir.
SistemasDistribuidos–Clase12 54/61

Blameless
Buscar las causas sin acusar a personas ni equipos. Se supone que todos actuaron con
| buena intención | y con la información | que tenían. |
| --------------- | -------------------- | ----------- |
▶
| Con culpa, | los problemas | se esconden. |
| ---------- | ------------- | ------------ |
▶
Viene de la salud y la aviación: cada error fortalece el sistema.
▶
No se puede “arreglar” a las personas; sí los sistemas que las ayudan a decidir.
| La pregunta | útil |     |
| ----------- | ---- | --- |
No “¿por qué se equivocó?”, sino “¿qué información, presiones y herramientas hicieron que
esa decisión pareciera razonable?”. Después, todo parece obvio: sesgo retrospectivo
| (hindsight bias). |     |     |
| ----------------- | --- | --- |
Blameless no es sin responsabilidad: igual señala qué mejorar, apoyado en el sistema.
SistemasDistribuidos–Clase12 54/61

Incidente ficticio en el generador
Un despliegue cambia la biblioteca HTTP y el timeout hacia especiales pasa a ser el por defecto:
esperar para siempre. Días después, especiales tiene pausas de garbage collection: hilos
bloqueados, pool agotado. La alerta suena en 2 minutos; mitigar lleva 25.
▶
Disparador: laspausasdeespeciales.
▶
Latente: untimeoutquedependíadelvalorpordefecto.
▶
Validación: elcambio“noerariesgoso”,sincanary;ningúnexperimentoperiódicodelatencia.
▶
Diagnóstico: eldashboardnomostrabalatenciapordependencia.
▶
Mitigación: varioscambiosjuntos,difícilesderevertir.
Disparador y condiciones
Disparador: el evento que lo inició. Condiciones: defectos y decisiones previas que
esperaban ese evento. Para que un sistema caiga suelen fallar varias defensas a la vez:
buscar “la” causa raíz (root cause) corrige un solo eslabón.
SistemasDistribuidos–Clase12 55/61

▶
Disparador: laspausasdeespeciales.
▶
Latente: untimeoutquedependíadelvalorpordefecto.
▶
Validación: elcambio“noerariesgoso”,sincanary;ningúnexperimentoperiódicodelatencia.
▶
Diagnóstico: eldashboardnomostrabalatenciapordependencia.
▶
Mitigación: varioscambiosjuntos,difícilesderevertir.
Disparador y condiciones
Disparador: el evento que lo inició. Condiciones: defectos y decisiones previas que
esperaban ese evento. Para que un sistema caiga suelen fallar varias defensas a la vez:
buscar “la” causa raíz (root cause) corrige un solo eslabón.
Incidente ficticio en el generador
Un despliegue cambia la biblioteca HTTP y el timeout hacia especiales pasa a ser el por defecto:
esperar para siempre. Días después, especiales tiene pausas de garbage collection: hilos
bloqueados, pool agotado. La alerta suena en 2 minutos; mitigar lleva 25.
SistemasDistribuidos–Clase12 55/61

| Disparador | y condiciones |     |     |
| ---------- | ------------- | --- | --- |
Disparador: el evento que lo inició. Condiciones: defectos y decisiones previas que
esperaban ese evento. Para que un sistema caiga suelen fallar varias defensas a la vez:
| buscar “la” | causa raíz (root | cause) corrige | un solo eslabón. |
| ----------- | ---------------- | -------------- | ---------------- |
| Incidente   | ficticio en el   | generador      |                  |
Un despliegue cambia la biblioteca HTTP y el timeout hacia especiales pasa a ser el por defecto:
esperar para siempre. Días después, especiales tiene pausas de garbage collection: hilos
bloqueados, pool agotado. La alerta suena en 2 minutos; mitigar lleva 25.
| ▶ Disparador: | laspausasdeespeciales.                  |     |     |
| ------------- | --------------------------------------- | --- | --- |
| ▶ Latente:    | untimeoutquedependíadelvalorpordefecto. |     |     |
▶
Validación: elcambio“noerariesgoso”,sincanary;ningúnexperimentoperiódicodelatencia.
▶
| Diagnóstico: | eldashboardnomostrabalatenciapordependencia. |     |     |
| ------------ | -------------------------------------------- | --- | --- |
▶
| Mitigación: | varioscambiosjuntos,difícilesderevertir. |     |     |
| ----------- | ---------------------------------------- | --- | --- |
SistemasDistribuidos–Clase12 55/61

Malas
Buenas (generador)
“Tener más cuidado al desplegar.”
“Mejorar el monitoreo.” Prevenir: eldesplieguefallasifaltaeltimeoutdeuna
Una lista sin dueños ni fechas. dependencia.
Detectar: experimentosemanaldelatenciapor
dependencia.
Detectar: paneldelatenciapordependencia.
Mitigar: unsolocambiopordespliegueenbibliotecas
dered.
Y al revés: las acciones propuestas en caliente suelen ser exageradas. Revisarlas con calma.
Acciones que cambian el sistema
Un postmortem vale lo que valen sus acciones: un dueño cada una, registradas,
verificables, que prevengan y no solo mitiguen. Cambiar el sistema es más confiable que
pedirle a la gente que cambie.
SistemasDistribuidos–Clase12 56/61

Y al revés: las acciones propuestas en caliente suelen ser exageradas. Revisarlas con calma.
| Acciones | que cambian | el sistema |
| -------- | ----------- | ---------- |
Un postmortem vale lo que valen sus acciones: un dueño cada una, registradas,
verificables, que prevengan y no solo mitiguen. Cambiar el sistema es más confiable que
| pedirle a la | gente que cambie. |     |
| ------------ | ----------------- | --- |
Malas
Buenas (generador)
| “Tener más | cuidado al desplegar.” |     |
| ---------- | ---------------------- | --- |
“Mejorar el monitoreo.” Prevenir: eldesplieguefallasifaltaeltimeoutdeuna
dependencia.
| Una lista sin | dueños ni fechas. |     |
| ------------- | ----------------- | --- |
Detectar: experimentosemanaldelatenciapor
dependencia.
Detectar: paneldelatenciapordependencia.
Mitigar: unsolocambiopordespliegueenbibliotecas
dered.
SistemasDistribuidos–Clase12 56/61

| Acciones | que cambian | el sistema |
| -------- | ----------- | ---------- |
Un postmortem vale lo que valen sus acciones: un dueño cada una, registradas,
verificables, que prevengan y no solo mitiguen. Cambiar el sistema es más confiable que
| pedirle a la | gente que cambie. |     |
| ------------ | ----------------- | --- |
Malas
Buenas (generador)
| “Tener más | cuidado al desplegar.” |     |
| ---------- | ---------------------- | --- |
“Mejorar el monitoreo.” Prevenir: eldesplieguefallasifaltaeltimeoutdeuna
dependencia.
| Una lista sin | dueños ni fechas. |     |
| ------------- | ----------------- | --- |
Detectar: experimentosemanaldelatenciapor
dependencia.
Detectar: paneldelatenciapordependencia.
Mitigar: unsolocambiopordespliegueenbibliotecas
dered.
Y al revés: las acciones propuestas en caliente suelen ser exageradas. Revisarlas con calma.
SistemasDistribuidos–Clase12 56/61

|     |     |     |     | Un postmortem | que nadie | revisa es como | si no existiera |
| --- | --- | --- | --- | ------------- | --------- | -------------- | --------------- |
Revisarlo, compartirlo, leerlo en grupo, reconocer a quien manejó bien el incidente. Y una
|               |            |       |     | señal de alarma: | incidentes | que se repiten. |     |
| ------------- | ---------- | ----- | --- | ---------------- | ---------- | --------------- | --- |
| Un postmortem | malo y uno | bueno |     |                  |            |                 |     |
| Malo          |            | Bueno |     |                  |            |                 |     |
Términos internos sin explicar. Lo entiende alguien de otro equipo.
| “Afectó a muchos | usuarios.” | Impacto medido, | con su fuente. |     |     |     |     |
| ---------------- | ---------- | --------------- | -------------- | --- | --- | --- | --- |
Acciones vagas, sin dueño, misma prioridad. Dueño, prioridad, estado final verificable.
Señala personas, tono dramático. Qué falló en el sistema, no quién.
| Meses después, | solo en el equipo. | En pocos días, | compartido. |     |     |     |     |
| -------------- | ------------------ | -------------- | ----------- | --- | --- | --- | --- |
SistemasDistribuidos–Clase12 57/61

| Un postmortem |     | malo y uno | bueno |     |
| ------------- | --- | ---------- | ----- | --- |
| Malo          |     |            | Bueno |     |
Términos internos sin explicar. Lo entiende alguien de otro equipo.
| “Afectó a muchos | usuarios.” |     | Impacto medido, | con su fuente. |
| ---------------- | ---------- | --- | --------------- | -------------- |
Acciones vagas, sin dueño, misma prioridad. Dueño, prioridad, estado final verificable.
Señala personas, tono dramático. Qué falló en el sistema, no quién.
| Meses después, | solo en   | el equipo. | En pocos días,       | compartido. |
| -------------- | --------- | ---------- | -------------------- | ----------- |
| Un postmortem  | que nadie | revisa es  | como si no existiera |             |
Revisarlo, compartirlo, leerlo en grupo, reconocer a quien manejó bien el incidente. Y una
| señal de alarma: | incidentes | que se repiten. |     |     |
| ---------------- | ---------- | --------------- | --- | --- |
SistemasDistribuidos–Clase12 57/61

Cierre
Sistemas Distribuidos

La brecha de la Entrega 3, en error budget. SLO de 99,9% para las instancias de
combate (43,2 min en 30 días). Cae un datacenter con un tercio de las instancias:
▶ Solución A, reubica en 15 min: equivale a 5 min de caída total, 12% del presupuesto.
▶ Solución B, réplicas listas, 1 min: unos 20 s, menos del 1%. A cambio, cuesta más.
▶ Un SLI por subsistema (directorio, combate, pagos) y por tipo de jugador: priorizar a
quienes ya juegan es gastar el presupuesto del ingreso para proteger el de la partida (load
shedding, clase 11).
▶ Validar sin implementar: ¿qué falla se inyectaría y qué resultado sostendría la decisión CAP
elegida?
Númerosilustrativos.
Aplicación al trabajo práctico: Asteria Online
La garantía no lleva números; la solución, sí
G: “el jugador no debe perder el oro pagado por un combate que no llegó a resolverse”. El
SLI y el SLO de cuánto lo logra son de cada solución: van en S y en T.
SistemasDistribuidos–Clase12 59/61

▶ Un SLI por subsistema (directorio, combate, pagos) y por tipo de jugador: priorizar a
quienes ya juegan es gastar el presupuesto del ingreso para proteger el de la partida (load
shedding, clase 11).
▶ Validar sin implementar: ¿qué falla se inyectaría y qué resultado sostendría la decisión CAP
elegida?
Númerosilustrativos.
Aplicación al trabajo práctico: Asteria Online
La garantía no lleva números; la solución, sí
G: “el jugador no debe perder el oro pagado por un combate que no llegó a resolverse”. El
SLI y el SLO de cuánto lo logra son de cada solución: van en S y en T.
La brecha de la Entrega 3, en error budget. SLO de 99,9% para las instancias de
combate (43,2 min en 30 días). Cae un datacenter con un tercio de las instancias:
▶ Solución A, reubica en 15 min: equivale a 5 min de caída total, 12% del presupuesto.
▶ Solución B, réplicas listas, 1 min: unos 20 s, menos del 1%. A cambio, cuesta más.
SistemasDistribuidos–Clase12 59/61

Aplicación al trabajo práctico: Asteria Online
La garantía no lleva números; la solución, sí
G: “el jugador no debe perder el oro pagado por un combate que no llegó a resolverse”. El
SLI y el SLO de cuánto lo logra son de cada solución: van en S y en T.
La brecha de la Entrega 3, en error budget. SLO de 99,9% para las instancias de
combate (43,2 min en 30 días). Cae un datacenter con un tercio de las instancias:
▶ Solución A, reubica en 15 min: equivale a 5 min de caída total, 12% del presupuesto.
▶ Solución B, réplicas listas, 1 min: unos 20 s, menos del 1%. A cambio, cuesta más.
▶ Un SLI por subsistema (directorio, combate, pagos) y por tipo de jugador: priorizar a
quienes ya juegan es gastar el presupuesto del ingreso para proteger el de la partida (load
shedding, clase 11).
▶ Validar sin implementar: ¿qué falla se inyectaría y qué resultado sostendría la decisión CAP
elegida?
Númerosilustrativos.
SistemasDistribuidos–Clase12 59/61

| Modelo | del Curso: | Confiabilidad | en D = (P,F,G,S,T) |
| ------ | ---------- | ------------- | ------------------ |
P (Problema): Operar con un nivel de servicio explícito, verificar que se cumple bajo falla y
|     | aprender | cuando no. |     |
| --- | -------- | ---------- | --- |
F (Factores): Fallas parciales, propiedades emergentes, cambios frecuentes, mediciones imperfectas,
|     | poco tráfico, | decisiones bajo | presión. |
| --- | ------------- | --------------- | -------- |
G (Garantías): SLOs medidos desde el usuario, alertas proporcionales a la gravedad, hipótesis
validadas, respuesta coordinada, acciones que reducen la repetición.
S (Solución): SLI como cociente + SLO de 30 días + error budget con política + burn rate con
varias ventanas + experimentos con criterio de aborto + roles + runbooks +
|     | postmortems | blameless. |     |
| --- | ----------- | ---------- | --- |
T (Trade-offs): Cada nueve cuesta más; los experimentos gastan presupuesto; precisión contra
|     | detección; | la documentación | envejece. |
| --- | ---------- | ---------------- | --------- |
SistemasDistribuidos–Clase12 60/61

▶ El SLO convierte “anda bien” en un número; el error budget, en una regla para decidir.
▶ El burn rate avisa con urgencia proporcional al daño.
▶ Los experimentos validan la resiliencia de la clase 11, con costo calculado.
▶ Roles claros reducen el daño; el postmortem blameless convierte el incidente en acciones.
Cierre
Idea central de cátedra
Una organización madura no se define por no tener incidentes, sino por medir con objetivos
explícitos, validar sus supuestos antes de que lo haga la realidad y aprender cada vez que
algo falla.
SistemasDistribuidos–Clase12 61/61

Cierre
| Idea central | de cátedra |     |     |
| ------------ | ---------- | --- | --- |
Una organización madura no se define por no tener incidentes, sino por medir con objetivos
explícitos, validar sus supuestos antes de que lo haga la realidad y aprender cada vez que
algo falla.
▶ El SLO convierte “anda bien” en un número; el error budget, en una regla para decidir.
| ▶ El burn | rate avisa con urgencia | proporcional | al daño. |
| --------- | ----------------------- | ------------ | -------- |
▶ Los experimentos validan la resiliencia de la clase 11, con costo calculado.
▶ Roles claros reducen el daño; el postmortem blameless convierte el incidente en acciones.
SistemasDistribuidos–Clase12 61/61