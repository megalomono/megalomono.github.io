---
layout: post
title: "Instalar un conjunto de replicas en MongoDB"
date: 2015-09-10
tags: MongoDB
---
El primer paso a la hora de garantizar la disponibilidad de una base de datos de MongoDB es construir un
conjunto de réplicas. Un conjunto de réplicas está formado por varios nodos que contienen copias de la base
de datos y que proporcionan redundancia durante las operaciones en producción.

Un conjunto de réplicas consta de un nodo primario y un mínimo de dos nodos secundarios, de modo que cuando
el nodo primario falle se pueda completar el proceso de votación para seleccionar un nuevo nodo primario. Para
crear un conjunto de réplicas sencillo de tres nodos hay que seguir los siguientes pasos.

En primer lugar, se crean los directorios donde cada uno de los nodos guardará sus datos.

{% highlight bash %}
~$ mkdir -p /data/rs1 /data/rs2 /data/rs3
{% endhighlight %}

A continuación se levantan los nodos. Para ello se ejecuta el comando *mongod* tres veces, una por cada nodo,
con las siguientes opciones.

* --replSet: Indica el conjunto de réplicas al que pertenece el nodo.
* --logpath: La ruta donde se guardarán los ficheros de log del nodo.
* --dbpath: El directorio donde se almacenan los datos del nodo.
* --port: El puerto de escucha del nodo.
* --oplogSize: El tamaño máximo en MB del log de operaciones de replicación.
* --fork: Se indica al proceso mongod que se ejecute en segundo plano.

{% highlight bash %}
~$ mongod --replSet myrs --logpath "1.log" --dbpath /data/rs1 --port 27017 --oplogSize 64 --fork
~$ mongod --replSet myrs --logpath "2.log" --dbpath /data/rs2 --port 27018 --oplogSize 64 --fork
~$ mongod --replSet myrs --logpath "3.log" --dbpath /data/rs3 --port 27019 --oplogSize 64 --fork
{% endhighlight %}

Hecho esto, desde uno de los nodos en ejecución,

{% highlight bash %}
~$ mongo --port 27017
{% endhighlight %}

se configura el conjunto de réplicas:

{% highlight javascript %}
>var config = { _id: "myrs", members:[
    { _id : 0, host : "localhost:27017"},
    { _id : 1, host : "localhost:27018"},
    { _id : 2, host : "localhost:27019"} ]
};
>rs.initiate(config);
{ "ok" : 1 }
myrs:OTHER>
myrs:PRIMARY> 
{% endhighlight %}

Mediante el comando *rs.status()* se puede ver el estado del conjunto de réplicas.

{% highlight javascript %}
m101:PRIMARY> rs.status()
{
	"set" : "myrs",
	"date" : ISODate("2015-09-10T19:00:15.859Z"),
	"myState" : 1,
	"members" : [
		{
			"_id" : 0,
			"name" : "localhost:27017",
			"health" : 1,
			"state" : 1,
			"stateStr" : "PRIMARY",
			"uptime" : 58,
			"optime" : Timestamp(1441910099, 1),
			"optimeDate" : ISODate("2015-09-10T18:34:59Z"),
			"electionTime" : Timestamp(1441911571, 1),
			"electionDate" : ISODate("2015-09-10T18:59:31Z"),
			"configVersion" : 1,
			"self" : true
		},
		{
			"_id" : 1,
			"name" : "localhost:27018",
			"health" : 1,
			"state" : 2,
			"stateStr" : "SECONDARY",
			"uptime" : 43,
			"optime" : Timestamp(1441910099, 1),
			"optimeDate" : ISODate("2015-09-10T18:34:59Z"),
			"lastHeartbeat" : ISODate("2015-09-10T19:00:13.968Z"),
			"lastHeartbeatRecv" : ISODate("2015-09-10T19:00:14.052Z"),
			"pingMs" : 0,
			"configVersion" : 1
		},
		{
			"_id" : 2,
			"name" : "localhost:27019",
			"health" : 1,
			"state" : 2,
			"stateStr" : "SECONDARY",
			"uptime" : 37,
			"optime" : Timestamp(1441910099, 1),
			"optimeDate" : ISODate("2015-09-10T18:34:59Z"),
			"lastHeartbeat" : ISODate("2015-09-10T19:00:13.986Z"),
			"lastHeartbeatRecv" : ISODate("2015-09-10T19:00:14.773Z"),
			"pingMs" : 0,
			"configVersion" : 1
		}
	],
	"ok" : 1
}
{% endhighlight %}

Como siempre, la información detallada sobre todo esto se encuentra en la
[documentación de MongoDB](https://docs.mongodb.org/manual/core/replication-introduction/).