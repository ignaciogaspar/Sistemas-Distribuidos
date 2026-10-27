# Clase 4

Problema de consenso:

n procesos

canales asincrónicos

fallas por crash

FLP

No existe algoritmo determinista que garantice consenso en sistema asincrono

con fallas

Habla de que podria esperar toda la vida, y si puedo esperar toda la vida, no
puedo garantizar, necesito timeouts o algo.

Raft

Estados:

follower

candidate

leader

El leader tiene que garantizar que confirma esta operacion, garantizando que se
va a guardar en por lo menos una n cantidad de nodos. Si se me caen los nodos

(menos de la mitad) la operacion va a estar guardada.

n/2 +1 de nodos (mas de la mitad de los nodos) que confirman algo, para
asegurarme de que por lo menos hay uno que tiene el valor consensuado.

algoritmo que sirve para mantener una copia centralizada entre muchos nodos

solo tolera fallas de crash, no si algun nodo arranca a hacer alguna boludez.

el lider es quien manda, cuando la mitad + 1, responde okey, se asegura que hay
n/2 + 1 y ya responde que si, no espera al resto.

tiene que ser la mayoria del numero original, no importa cuantos estan vivos o no

Clase 41

---

version 1, 2, 3, 4, va aumentando

si un nodo estaba en la version 1, y se habia caido y despues vuelve a la vida,

cuando el lider le manda la version 4, el nodo le va a decir rejected porque estaba

en la version 1.

el nodo tiene lista de por donde va cada nodo

A → 4

B → 4

C → 4

D → 1

E → 1

el lider sabe por donde van porque d y e no le venian respondiendo, cuando le
mando 4, me dicen reject, entonces les mando el 3, me dicen reject, y entonces le
mando el 2, y me doy cuenta de que aparecio, entonces le mando ahi 3 y 4. por

eso el lider tiene que guardar una cola de todo el historial de logs.

si no reciben ticks del lider y hay timeout, el lider se cayo entonces se propone un
candidato, y si la mayoria dice ok, (te propones, y el resto te confirma), todos
tienen la cola de logs,

Sirve para guardar un log de operaciones distribuidas. Me doy cuenta que es el
ultimo, el que tiene el commit index mayor. No existe un valor valido, tengo que
preguntarle a todos, y ver cual es el commit index mayor para ver cual es el ultimo

cuando aparece devuelta el lider, va a querer mandar un mensaje y le dicen no,
ahora sos follower.

Paxos

sirve para tener solo un valor, solo se preocupa por tener un valor. Se hacen 2
rondas, cualqueira que tenga un cambio para proponer, tiene un indice, que es un

numero, y le tira a todos los nodos, quiero publicar el indice 4 y le pregunta si le
aceptan el numero 4, Responden okey si su index es menor (significa que todos
estan mas atrasados que yo)

fase 1: propongo el 4 (todos me aceptan porque indice < propuesto)

Clase 42

---

fase 2: propongo el valor abc

el indice es incremental consecutivo, la idea es que incremente porque comparas

tu numero con el del otro.

Camino feliz: todos me dicen que si, entonces mando: aceptame el valor abc en el

indice 4

fase 1 PROPOSE: consigo que me acepten el indice. consigo permiso para

proponer

fase 2 ACCEPT: mando el valor. mando el dato

ahora, porai un nodo se durmio, y se quedo en el 4, y el resto avanzo y esta por el

indice 7 (que se lo aceptaron por mayoria), si el dice: che me aceptan el 5, y el

resto le dice no, vamos por el 7, entonces el lo actualiza y dice, aceptenme el 8, y
con el mismo valor. Si el se desperto entre el 6 y 7, le mandan el 7 y lo recibio.

Que pasa si hay 2 que quieren mandar al mismo tiempo?

PAXOS: En cualquiera de los nodos puede venir un cambio.

Cuantas mas fallas queres cubrir, mas interacciones tenes que hacer.

si hay un servidor que miente, se puede romper Raft, entonces nunca podrias
tener una garantia donde se banque un servidor que miente. Si uso algoritmo de
consenso, tengo que saber que se banca.

el numero del indice, en paxos, se compone tambien por el nodo que lo manda, no
va a pasar de que hayan dos numeros iguales.

Raft: operaciones de jugador: garantia de que la info de jugador no se pierde ante
determinada falla. Como tenes un log de historia y lo queres aplicar en orden

Paxos: una batalla, tenemos un monton de nodos que proponen cambios porque
estan peleandose, UN SOLO VALOR

Clase 43
