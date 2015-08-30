---
layout: post
title: "Búsquedas avanzadas en MongoDB"
date: 2015-08-13
tags: MongoDB
---
Las funciones de búsqueda de MongoDB van más allá de filtrar los documentos de una colección por
los valores de sus campos simples; también es posible filtrar por arrays y documentos anidados,
así como modificar las consultas para acotar el número de resultados o aplicar criterios de ordenación
a una colección.

En realidad, cuando se invoca a la función *find()* lo que está sucediendo entre bambalinas es que
MongoDB crea un cursor sobre la colección en cuestión. Este cursor es un objeto iterador y como tal
puede ser almacenado en una variable para manipularlo a nuestro antojo o invocar funciones sobre él.

{% highlight javascript %}
> var cursor = db.rebels.find()
> cursor.hasNext()
true
> cursor.next()
{
    "_id" : ObjectId("55cb8670519cca29dcba3a0"),
    "name" : "Luke Skywalker",
    "spaceship" : "X-Wing fighter"
}
{% endhighlight %}

¿Y qué cosas se pueden hacer con un cursor? Aparte de recorrer la colección, los cursores permiten
definir criterios de ordenación, limitar el número de resultados o saltar algunos de los documentos
de la colección al ejecutar la consulta.

Es importante tener en cuenta que estas operaciones se realizan en el servidor y no en cliente. Esto
implica que **cualquier modificación sobre el cursor debe realizarse antes de recuperar ningún resultado**,
ya que los cambios no podrían aplicarse a colecciónes ya recuperadas.

###Ordenación

Para ordenar el resultado de una consulta se emplea la función *sort()*. Esta función recibe como
parámetro un documento JSON indicando los campos para la ordenación y el sentido de la misma.

Por ejemplo, para listar a los rebeldes de nuestra colección ordenados por nombre en orden alfabético
descendente se haría lo siguiente:

{% highlight javascript %}
> db.rebels.find({}, { "_id" : false, "name" : true }).sort({ "name" : -1 })
{ "name" : "Yoda" }
{ "name" : "Obi-Wan Kenobi" }
{ "name" : "Luke Skywalker" }
{ "name" : "Han Solo" }
{ "name" : "Chewbacca" }
{% endhighlight %}

###Limitar el número de resultados

Para limitar el número de resultados devueltos por una consulta se emplea la función *limit()*. Esta
función recibe como parámetro el número de resultados que se desean obtener.

{% highlight javascript %}
> db.rebels.find({}, { "_id" : false, "name" : true }).sort({ "name" : -1 }).limit(2)
{ "name" : "Yoda" }
{ "name" : "Obi-Wan Kenobi" }
{% endhighlight %}

###Omitir 'n' resultados

Para omitir 'n' resultados de una consulta se emplea la función *skip()*. Esta función recibe como
parámetro el número de resultados que se desean omitir.

{% highlight javascript %}
> db.rebels.find({}, { "_id" : false, "name" : true }).sort({ "name" : -1 }).skip(2)
{ "name" : "Luke Skywalker" }
{ "name" : "Han Solo" }
{ "name" : "Chewbacca" }
{% endhighlight %}

Las funciones *limit()* y *skip()* se pueden combinar para acotar el resultado de la consulta:

{% highlight javascript %}
> db.rebels.find({}, { "_id" : false, "name" : true }).sort({ "name" : -1 }).skip(2).limit(2)
{ "name" : "Luke Skywalker" }
{ "name" : "Han Solo" }
{% endhighlight %}

###Filtrar arrays

La naturaleza polimórfica de MongoDB también permite filtrar por los valores contenidos en un campo
de tipo array como si fuese un simple string. Por ejemplo, teniendo un documento como este:

{% highlight javascript %}
{
    "_id" : ObjectId("55cb8670519cca29dcba3a0"),
    "name" : "Luke Skywalker",
    "spaceship" : "X-Wing fighter",
    "visitedPlanets" : [ "Tatooine", "Yavin 4", "Hoth", "Dagobah", "Endor" ]
}
{% endhighlight %}

se podría hacer una búsqueda con el siguiente filtro:

{% highlight javascript %}
> db.rebels.find({ "visitedPlantes" : "Hoth" }, { "_id" : false, "name" : true })
{ "name" : "Luke Skywalker" }
{% endhighlight %}

Es decir, MongoDB comprobará si el valor del filtro está contenido en el array.

###Filtrar documentos anidados

Dado que es posible que un documento contenga otros documentos anidados, es posible también realizar
búsquedas en función de los valores de dichos documentos anidados.

{% highlight javascript %}
{
    "name" : "Luke Skywalker",
    "spaceship" : "X-Wing fighter",
    "father" : { "name" : "Darth Vader" }
}
{
    "name" : "Leia Organa",
    "father" : { "name" : "Darth Vader" }
}
{% endhighlight %}

Para buscar a los hijos de Darh Vader habría que ejecutar la siguiente consulta:

{% highlight javascript %}
> db.rebels.find({ "father.name" : "Darth Vader" }, { "_id" : false, "name" : true })
{ "name" : "Luke Skywalker" }
{ "name" : "Leia Organa" }
{% endhighlight %}

Nótese que para acceder a los campos de los documentos anidados se emplea la notación de punto
(\<documento\>.\<campo\>) y que, aunque MongoDB es permisivo en cuanto al uso de comillas en
los nombres de los campos, en este caso es necesario emplearlas para evitar un error de sintaxis:

{% highlight javascript %}
> db.rebels.find({ father.name : "Darth Vader" })
2015-08-13T10:44:02.086+0200 E QUERY    SyntaxError: Unexpected token .
{% endhighlight %}

###Conclusión

Estas son las principales opciones de búsqueda que ofrece MongoDB. Todas ellas se pueden combinar entre sí,
dando como resultado un conjunto muy potente de herramientas para explotar la información de la base de datos
de forma bastante sencilla. Como siempre, para obtener más información es conveniente revisar la
[documentación de MongoDB](http://docs.mongodb.org/manual/). 