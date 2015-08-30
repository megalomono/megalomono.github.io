---
layout: post
title: "Comandos básicos para MongoDB"
date: 2015-08-12
tags: MongoDB
---
MongoDB es una base de datos no relacional basada en documentos. Esto significa que almacena la
información como colecciones de documentos, concretamente documentos en formato JSON. El modo más
rápido de realizar consultas y otras operaciones contra una base de datos Mongo es a través del Mongo Shell.

Tras arrancar Mongo Shell mediante el comando *mongo*, los mensajes de log indicarán sobre qué base
de datos se está trabajando. Esto mismo se puede comprobar fácilmente tecleando *db*, que imprimirá
por pantalla el nombre de la base de datos actual. En realidad, *db* es un objeto Javascript que representa
a la base de datos y que tiene como propiedades cada una de las colecciones de la misma. En MongoDB no
existe ningún lenguaje intermedio del estilo de SQL para operar sobre los datos; todas las operaciones
en MongoDB son llamadas a funciones sobre una colección.

###Inserciones

Para insertar un nuevo documento en una colección se emplearía la función *insert()*. Si la colección no
existiese, MongoDB la crearía.

{% highlight javascript %}
> db.rebels.insert({ "name" : "Luke Skywalker", "spaceship" : "X-Wing fighter" })
{% endhighlight %}

La función *insert()* acepta como parámetro un objeto JSON, así que seria posible hacer algo como esto:

{% highlight javascript %}
> var obiwan = {}
> obiwan.name = "Obi-Wan Kenobi"
> obiwan.job = "Jedi"
> db.rebels.insert(obiwan)
{% endhighlight %}

###Búsquedas

Para recuperar un documento insertado hay dos opciones. La primera es utilizar la función *findOne()*,
que devuelve el primer documento de la colección que cumple los criterios de búsqueda:

{% highlight javascript %}
> db.rebels.findOne()
{
    "_id" : ObjectId("55cb6ff60319cca29dcba39e"),
    "name" : "Luke Skywalker",
    "spaceship" : "X-Wing fighter"
}
{% endhighlight %}
    
La función *findOne()* admite dos parámetros: un filtro (la clausula WHERE de SQL) y una proyección
(la clausula SELECT). Ambos parámetros son opcionales y son objetos JSON. Si por ejemplo se desea
recuperar el rebelde cuyo nombre es "Han Solo" y mostrar su nave espacial, la consulta sería:

{% highlight javascript %}
> db.rebels.findOne({ "name" : "Han Solo" }, { "_id" : false, "spaceship" : true })
{ "spaceship" : "Millenium Falcon" }
{% endhighlight %}

Por defecto, Mongo devuelve siempre el campo "_id" del documento, salvo que se le especifique lo
contrario.

La otra opción para recuperar los documentos de una colección es emplear la función *find()*. Una llamada sin
parámetros a esta función devuelve todos los documentos de una colección. Por defecto, Mongo Shell recupera
los documentos en lotes, avanzando al siguiente lote por demanda explícita del usuario mediante el
comando *it*. Dado que Mongo mantiene un cursor abierto hasta que se termina de recuperar la colección,
hacer esto no suele ser buena idea. No obstante, la opción está ahí.

La función *find()* admite los mismos parámetros que la función *findOne()*, de modo que se pueden filtrar
los resultados y proyectar los campos que se desee recuperar.

{% highlight javascript %}
> db.rebels.find({ "race" : "wookiee" }, { "_id" : false, "name" : true })
{ "name" : "Chewbacca" }
{% endhighlight %}
    
Además de filtrar por los propios campos de la colección, Mongo ofrece varios operadores que permiten afinar
las búsquedas. Por ejemplo, si se quieren recuperar los rebeldes con más de 700 años de edad se podría hacer
uso del operador *$gt*:

{% highlight javascript %}
> db.rebels.find({ "age" : { $gt : 700 } }, { "_id" : false })
{ "name" : "Yoda", "age" : 900 }
{% endhighlight %}

Existen varios operadores que permiten realizar distintos filtrados, todos ellos explicados en la [documentación
de MongoDB](http://docs.mongodb.org/manual/reference/operator/query/).

Conviene resaltar que tanto *findOne()* como *find()* devolveran aquellos documentos que cumplen **todas** las
condiciones del filtro; dicho de otro modo, se construyen predicados conjuntivos por defecto. Se puede construir
un predicado disyuntivo aplicando el operador *$or* sobre un array de condiciones. Por ejemplo, para recuperar
a los rebeldes que, o bien pertenecen a la familia Skywalker, o bien son wookies, habría que ejecutar la siguiente
consulta (la función *pretty()* simplemente formatea el resultado para hacerlo más legible):

{% highlight javascript %}
> db.rebels.find({ $or: [{ "name" : { $regex : "Skywalker" } }, { "race" : "wookiee" }]}).pretty()
{
    "_id" : ObjectId("55cb8670519cca29dcba3a0"),
    "name" : "Luke Skywalker",
    "spaceship" : "X-Wing fighter"
}
{
    "_id" : ObjectId("55cb86e80319cca29dcba3a1"),
    "name" : "Chewbacca",
    "race" : "wookiee"
}
{% endhighlight %}

Por último, para contar los resultados (el equivalente al COUNT de SQL) existe la función *count()*, que se
comporta de forma análoga a *find()*:

{% highlight javascript %}
> db.rebels.count()
5
{% endhighlight %}

###Actualizaciones

A la hora de actualizar un documento MongoDB ofrece la función *update()*. Esta función puede usarse de dos maneras:
o bien para realizar una modificación de todo el documento, o bien para modificar solo determinados campos.
En ambos casos *update()* recibe como primer parámetro un filtro para establecer sobre qué documentos se realizarán
las modificaciones; el segundo argumento varía según como se desee ejecutar la función.

Con la primera opción, *update()* remplaza todos los campos del documento por los valores indicados, sobreescribiendo
el documento a todos los efectos (a excepción del campo "_id", que es inmutable).

{% highlight javascript %}
> db.rebels.update({ "name" : "Luke Skywalker" }, { "name" : "Luke Skywalker", "affiliation" : "Light side" })
> db.rebels.find({ "name" : "Luke Skywalker"} ).pretty()
{
    "_id" : ObjectId("55cb8670519cca29dcba3a0"),
    "name" : "Luke Skywalker",
    "affiliation" : "Light side"
}
{% endhighlight %}

Con esta consulta se ha actualizado a Luke Skywalker y ahora está en el lado de la luz, pero por el camino ha perdido su
nave. Para mantener toda la información anterior habría que añadir a la consulta de modificación todos y cada uno de los
campos del documento, sufran o no cambios. Por motivos obvios, esto es poco práctico.

Para evitar este problema existe el segundo modo de ejecutar la función, que es básicamente igual a lo anterior pero con
una pequeña modificación que consiste en el uso del operador *$set*. Por ejemplo, para que Luke se pase al lado oscuro,
pero sin perder nada con el cambio, bastaría con lo siguiente:

{% highlight javascript %}
> db.rebels.update({ "name" : "Luke Skywalker" }, { $set: { "affiliation" : "Dark side" } })
> db.rebels.find({ "name" : "Luke Skywalker"} ).pretty()
{
    "_id" : ObjectId("55cb8670519cca29dcba3a0"),
    "name" : "Luke Skywalker",
    "affiliation" : "Dark side"
}
{% endhighlight %}

Este último método es mucho más práctico y seguro, ya que no es necesario conocer y propagar todos los campos de un documento
para conservar su información.

Se puede eliminar un campo que ya no sea necesario ejecutando la función *update()* con el operador *$unset* e indicando
los campos a eliminar.

{% highlight javascript %}
> db.rebels.update({ "name" : "Luke Skywalker" }, { $unset: { "affiliation" : 1 } })
> db.rebels.find({ "name" : "Luke Skywalker"})
{ "_id" : ObjectId("55cb8670519cca29dcba3a0"), "name" : "Luke Skywalker" }
{% endhighlight %}

Por último, hay una opción muy útil a la hora de realizar una modificación en un documento y es que es posible indicar
a MongoDB que, si no existe el documento para modificar, cree un nuevo documento con los valores indicados mediante el
parámetro *upsert: true*.

{% highlight javascript %}
> db.rebels.find({ "name" : "R2D2"} )
> db.rebels.update({ "name" : "R2D2" }, { $set: { "race" : "Droid" } }, { upsert : true })
> db.rebels.find({ "name" : "R2D2"} )
{ "_id" : ObjectId("55ccw2703b996fff1ed3f8c3"), "name" : "R2D2", "race" : "Droid" }
{% endhighlight %}

Es importante tener en cuenta que la función *update()* **afecta únicamente al primer documento que coincida con el filtro de
búsqueda**. Para realizar una modificación que afecte a todos los registros que coincidan con las condiciones de filtrado es
necesario añadir la opción *multi: true*.

###Borrados

Finalmente, para completar el conjunto de operaciones CRUD, existe la opción de borrar un documento. Para esto existe la
función *remove()*.

{% highlight javascript %}
> db.rebels.remove({ "name" : "Yoda" })
> db.rebels.find({ "name" : "Yoda"})
{% endhighlight %}

Al contrario que la función *update()*, *remove()* **afecta a todos los documentos que coincidan con el filtro de búsqueda**.
Es decir, llamar a *remove()* con el filtro de búsqueda {} eliminará todos los documentos de la colección. No obstante, si esto
último es lo que se quiere hacer realmente, es mucho más eficiente emplear la función *drop()*; mientras que *remove()* recorre
la colección eliminando los documentos uno a uno, *drop()* elimina la colección directamente.

{% highlight javascript %}
> db.rebels.drop()
true
> db.rebels.count()
0
{% endhighlight %}

###Conclusión

Las funciones aquí mostradas cubren las necesidades básicas de cualquier base de datos: inserciones, lecturas, modificaciones y
borrados. Obviamente, existe la posibilidad de ejecutar operaciones más avanzadas. Se puede obtener más información sobre
estas operaciones en la [documentación de MongoDB](http://docs.mongodb.org/manual/), donde se explican en detalle todas las
funcionalidades que ofrece este sistema. 

