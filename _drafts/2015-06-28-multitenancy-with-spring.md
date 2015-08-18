---
layout: post
title: "Arquitectura multicliente con Spring"
date: 2015-06-28
---
A la hora de implementar una arquitectura multicliente existen tres opciones: tener una aplicación por cliente, tener una única aplicación y
una base de datos por cliente o tener una única aplicación y una única base de datos para todos los clientes. Aquí describo como implementar
la tercera opción empleando Spring Security y Spring Data.

Antes de entrar en materia es bueno darle un par de vueltas a la cabeza y pensar que condiciones tiene que cumplir el invento:

* Al emplear la misma base de datos para todos los clientes es necesario filtrar los resultados de las consultas realizadas.

* Por motivos obvios, no se puede guardar ninguna entidad multicliente sin haber establecido a qué cliente pertenece.

Para evitar andar carrentado identificadores de clientes de un lado a otro por todas las capas, haremos que la aplicación añada automáticamente
el filtro del cliente a cada consulta realizada, de forma que para el resto del código sea indiferente el hecho de tratar con varios clientes.

###Primeros pasos

En primer lugar creamos el proyecto, obviamente. En este caso voy a emplear Gradle para la gestión de dependencias; también se puede utilizar Maven,
según gustos. Como no quiero complicarme la vida voy a hacer uso de los starters de Spring Boot. El fichero build.gradle queda de la siguiente forma:

{% highlight groovy %}
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:1.2.3.RELEASE")
    }
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'spring-boot'

jar {
    baseName = 'multitenancy-example'
    version =  '0.1.0'
}

repositories {
    mavenCentral()
}

sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    compile("org.springframework.boot:spring-boot-starter-thymeleaf")
    compile("org.springframework.boot:spring-boot-starter-data-jpa")
    compile("com.h2database:h2")
    compile("org.springframework.boot:spring-boot-starter-security")
    testCompile("junit:junit")
}
{% endhighlight %}

Las dependencias importantes aquí son las de Spring Data y Spring Security. A mayores incluyo una base de datos en memoria y Thymeleaf para la capa web,
pero esto va a gusto del consumidor.


