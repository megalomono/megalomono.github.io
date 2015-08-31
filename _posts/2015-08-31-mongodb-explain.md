---
layout: post
title: "Comando explain() en MongoDB"
date: 2015-08-31
tags: MongoDB
---
Al ejecutar una consulta de búsqueda en MongoDB, el motor de almacenaje evalúa varios planes de ejecución
para determinar cual es el más adecuado. Se puede obtener información acerca de una consulta con el comando
*explain()*.

Se puede invocar a este comando de dos maneras. La primera es invocarlo sobre un cursor, de modo que evaluará
la consulta y devolverá la información solicitada.

Por ejemplo, teniendo una colección con 100 elementos:

{% highlight javascript %}
> for (var i = 0; i < 100; i++) { db.foo.insert({ "a": i }) }
WriteResult({ "nInserted" : 1 })
{% endhighlight %}

Podría ejecutarse el siguiente comando.

{% highlight javascript %}
> db.foo.find().explain()
{
	"queryPlanner" : {
		"plannerVersion" : 1,
		"namespace" : "test.foo",
		"indexFilterSet" : false,
		"parsedQuery" : {
			"$and" : [ ]
		},
		"winningPlan" : {
			"stage" : "COLLSCAN",
			"filter" : {
				"$and" : [ ]
			},
			"direction" : "forward"
		},
		"rejectedPlans" : [ ]
	},
	"serverInfo" : {
		"host" : "mongodb-vm",
		"port" : 27017,
		"version" : "3.0.5",
		"gitVersion" : "8bc4ae20708dbb493cb09338d9e7be6698e4a3a3"
	},
	"ok" : 1
}
{% endhighlight %}

El segundo modo de invocar a la función *explain()* es hacerlo sobre una colección. Esto devolverá un *explainable
object* sobre el que pueden ejecutarse las funciones que se deseen para obtener información sobre ellas.

{% highlight javascript %}
> db.foo.explain().find().count()
{
	"queryPlanner" : {
		"plannerVersion" : 1,
		"namespace" : "test.foo",
		"indexFilterSet" : false,
		"winningPlan" : {
			"stage" : "COUNT"
		},
		"rejectedPlans" : [ ]
	},
	"serverInfo" : {
		"host" : "mongodb-vm",
		"port" : 27017,
		"version" : "3.0.5",
		"gitVersion" : "8bc4ae20708dbb493cb09338d9e7be6698e4a3a3"
	},
	"ok" : 1
}
{% endhighlight %}

Este modo de usar *explain()* es más versátil, ya que se pueden encadenar modificadores a la consulta. Si se invoca *explain()*
sobre *find()* esto no se puede hacer, ya que *explain()* devuelve un documento y no un iterador sobre el que seguir
encadenando llamadas.

{% highlight javascript %}
> db.foo.find().explain().count()
2015-08-31T17:52:59.526+0200 E QUERY    TypeError: Object [object Object] has no method 'count'
    at (shell):1:25
{% endhighlight %}

La función *explain()* admite como parámetro un string que indica el modo de verbosidad de la función. Los modos admitidos en MongoDB 3.0
son los siguientes:

- **queryPlanner**: Es el modo de ejecución por defecto. MongoDB ejecuta el planificador de consultas y devuelve la información sobre el plan ganador.
- **executionStats**: MongoDB ejecuta el planificador de consultas, ejecuta el plan ganador y devuelve la información tanto del planificador de consultas como de la ejecución del plan elegido.
- **allPlansExecution**: A mayores de la información aportada por los modos anteriores, también se muestra la información referente a los planes de ejecución descartados.

Por ejemplo, si se desea conocer la información sobre la ejecución del *count()* del ejemplo anterior, solo hace falta invocar a *explain()*
pasando como parámetro la cadena "executionStats".

{% highlight javascript %}
> db.foo.explain("executionStats").find().count()
{
	"queryPlanner" : {
		"plannerVersion" : 1,
		"namespace" : "test.foo",
		"indexFilterSet" : false,
		"winningPlan" : {
			"stage" : "COUNT"
		},
		"rejectedPlans" : [ ]
	},
	"executionStats" : {
		"executionSuccess" : true,
		"nReturned" : 0,
		"executionTimeMillis" : 0,
		"totalKeysExamined" : 0,
		"totalDocsExamined" : 0,
		"executionStages" : {
			"stage" : "COUNT",
			"nReturned" : 0,
			"executionTimeMillisEstimate" : 0,
			"works" : 1,
			"advanced" : 0,
			"needTime" : 0,
			"needFetch" : 0,
			"saveState" : 0,
			"restoreState" : 0,
			"isEOF" : 1,
			"invalidates" : 0,
			"nCounted" : 100,
			"nSkipped" : 0
		}
	},
	"serverInfo" : {
		"host" : "mongodb-vm",
		"port" : 27017,
		"version" : "3.0.5",
		"gitVersion" : "8bc4ae20708dbb493cb09338d9e7be6698e4a3a3"
	},
	"ok" : 1
}
{% endhighlight %}

Obviamente, la información de esta consulta es muy simple, pero en el caso de consultas más complejas el plan de ejecución contendría varios
documentos anidados indicando cada una de las fases por las que pasa la consulta. La información detallada sobre la salida de
la función y su significado puede encontrarse [aquí](http://docs.mongodb.org/manual/reference/explain-results).
