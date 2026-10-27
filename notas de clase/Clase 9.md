# Clase 9

antología → termino que es valido para esta frontera, y podría tener otro conjunto
donde use esa palabra pero tenga otro significado

Frontera: limite explicito que separa dos ámbitos del sistema

servicio: contrato

microservicio: funcion especifica, autonomo

no puede existir la espada de sofia como en el catalogo como objeto, mercado

como objeto, combate como objeto. NO, el objeto debe vivir en una sola de las

instancias, el inventario tiene la posta y puede cambiarle los atributos, y los

demas miran los atrubutos y la ofrecen. El objeto autoritativo debe estar en un
solo modulo. El resto hacen referencia a esa instancia

Patron de paralelizacion: las particiones en kafka, logro dividir en particiones,
atras es viene un dato, y yo se a que particion pertenece. Kafka lo hace con

funciones hash, viene un dato, lo alimento con funcion hash, y siempre para ese
dato me va a dar a que particion pertenece, permite que todas las particiones
tengan uniformemente requeridas, que no haya hot partition.

Aplicación modular o servicios

Cuando NO distribuir:

entro en una empresa y no tiene claro la idea de su aplicacion y como quieren
crecer bla bla. Hasta que las reglas no esten claras, no distribuir, tienen que tener
el problema claro

Tengo un equipo pequeño, 2 o 3, decir vamos a migrar, no se va a poder

mantener, y no vamos a contratar a miles, lo mantengo lo mas unido posible

Las compras necesitan invariantes que caben en una transacción local

La carga de los componentes todavía nos la podemos bancar

El equipo aun no posee herramientas y guardias para operar muchos servicios

Dos formas de coordinar la saga:

Coreografia → la roma de desiciones es descentralizada

Clase 91

---

Orquestacion → dependo de un director que dice que hacer. Es tipico de un lider.
Google spanner que tiene multipaxos

riesgo: saturacion, del coordinador unico

Coreografia u orque?

por ejemplo, para el inventario, elijiria orquestacion, ya que no tengo problemas

de inconsistencias ni incoherencias. El inventario, como modifica el dato critico, lo

ideal seria de orquestacion

Clase 92
