# Clase 8

Offset: contador que va diciendo el orden de las tareas en las que se van

ejecutando las tareas. Es el orden del consumidor. El consumidor consume los

logs y el offset le va diciendo hasta donde llegue. El conusmidor si crashea, dice:

“aa, llegue hasta aqui” Por eso en las peliculas si las dejamos y una semana mas

tarde volvemos, gracias al offset en netflix sabe hasta donde habias llegado

Raft vs Paxos, raft llega a solucion de quorum, si ese valor es el que todos

estamos de acuerdo, le podes dar una configuracion: mayoria, 3/4, espera a que

el consenso se llega y una vez que llegan, se envia la informacion, se satura mas

la red pero no se necesita lider

en cambio en paxos se necesita lider, es mas facil porque hay lider pero que pasa

si se cae el lider?

Lo que un ingeniero decide realmente

venta de ultima entrada, ademas nos ofrecen consumision o pochoclos o, si es un
concierto, en madrid agregaron remeras pero aca en arg gorros. A quien le otorgo
esa compra, pero si es la ultima entrada, tengo que decirle que no a tomas, el

error tolerable es el rechazo temporal,

Declarar: operacion, invariante, error tolerable, evidencia

Happened-before: tres reglas

es dificil decir: esto causo tal cosa en un sistema dist, es mas facil decir esto
ocurrio antes

1.  Si dos eventos ocurren en el mismo proceso, su orden local induce causalidad

2.  El envio de un mensaje ocurre antes que la recepcion de ese mensaje

3.  La relacion transitiva. Si a causo b y b causo c, de a puede derivar a c.

Ordenar la causalidad de los eventos

Lamport: convertir relaciones causales en numeros crecientes

Clase 81

---

Va acumulando un contador.

Raft: termino, indice y entrada de log

Termino

Indice

Entrada

Term es un reloj de Lamport, es un contador

linearizabilidad vs seri

Quorums, versiones y Dynamo

Consenso:

Dynamo-like: ahora no es siempre always on.

Entropia: que tanta info o no tengo. Evitar la desinformacion.

Spanner

Clase 82
