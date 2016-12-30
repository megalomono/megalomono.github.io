---
layout: post
title: "SOLID: Inversión de dependencias"
date: 2016-12-30
tags: OOP
---
El desarrollo de software es, desde el inicio, un complejo ciclo de elección y arrepentimiento: diseñar un modelo adecuado, escoger las tecnologías para la implementación,
comprometerse con una fecha de entrega y renegar de todas estas decisiones como si hubiesen sido tomadas por un lémur borracho.

<p class="text-center"><img src="/assets/dilbert-drunken-lemurs.jpg" alt="Drunken lemurs"/></p>

Este ciclo miserable no hace sino desequilibrarse con el tiempo: la capacidad de elección se ve cada vez más reducida y lastrada por el pasado mientras el arrepentimiento
aumenta hacia cotas insospechadas, hasta llegar a ese momento en el que alguien dice aquello de "hay que rehacer esto desde cero"*.

En este punto hay que hacer un ejercicio de sinceridad y asumir que el software no se degrada por sí solo. No es un componente físico que se deteriore con el tiempo.
El software tiende hacia la entropía, es cierto, pero el único agente entrópico en este mundillo es el propio desarrollador. La parte buena de esto es que es barato impedir
la degradación del software; la parte mala es que depende de la disciplina del programador.

La ingeniería del software dispone de varias herramientas para ayudar en este problema. La más conocida son los patrones de diseño, muchos de los cuales están descritos en el
famoso [libro de los cuatro](https://www.amazon.es/Design-Patterns-Elements-Reusable-Object-Oriented-ebook/dp/B000SEIBB8). De aquí se extrae el principio de **programar contra
interfaces y no contra implementaciones**. Esta idea es el meollo de todo el asunto y ha dado pie al concepto de *inversión de dependencias*, tal y como lo ha descrito
[Robert Martin](https://www.amazon.es/Principles-Patterns-Practices-Robert-2006-07-30/dp/B01MSK2U6V/ref=asap_bc?ie=UTF8):

> A. High-level modules should not depend on low-level modules. Both should depend on abstractions.
>
> B. Abstractions should not depend upon details. Details should depend upon abstractions.

Programar contra interfaces es una práctica más o menos extendida y aceptada, pero no sucede lo mismo con la inversión de dependencias, que generalmente es vista como "esa cosa
que hace X" (donde X es el contenedor de dependencias favorito), pero sin entender realmente en que consiste.

Supongamos una arquitectura de capas típicas con las siguientes clases:

<p class="text-center"><img src="/assets/layer-architecture.png" alt="Layer architecture"/></p>

La clase controladora recibe la petición del cliente y ataca a una interfaz de servicio. Dicha interfaz tiene una implementación que a su vez ataca a la interfaz de acceso
a datos para las entidades involucradas en el caso de uso. Todo claro hasta aquí. Esto funciona bien y respeta el principio de programar contra interfaces y no contra implementaciones.
Pero ¿cómo es la división en capas de estas clases? Típicamente, así:

<p class="text-center"><img src="/assets/wrong-dependecies.png" alt="Wrong dependencies"/></p>

El problema con este diseño es que la capa de servicios, de alto nivel, depende de la capa de acceso a datos, de bajo nivel. Ojo, estamos hablando de niveles conceptuales, no de cercanía
al cliente: la vista es superior en la pila de llamadas, pero en cuanto a diseño es una capa puramente instrumental. La clave aquí es aislar lo verdadermente importante en el
sistema, la esencia del mismo. Si las reglas del dominio, la articulación de los casos de uso, se encuentran en la capa de servicios (quizá un nombre desafortunado) ¿por qué esta capa
depende de algo mutable y circunstancial como es la capa de acceso a datos? ¿Por qué es la capa de acceso a datos la que define la interfaz que proporciona a los servicios?

Vamos a hacer algunos cambios en el diseño:

<p class="text-center"><img src="/assets/right-dependecies.png" alt="Right dependencies"/></p>

En primer lugar, un cambio estético: el concepto servicio es algo demasiado ambiguo, así que la capa de servicios pasa a ser capa de negocio para reflejar mejor su importancia.
En segundo lugar, lo realmente importante: la interfaz de acceso a datos pasa a ser propiedad de la capa de negocio. Dicho de otro modo, la capa de negocio, el núcleo conceptual
del software, ya no depende de ningún otro componente del sistema, sino que define sus propios puntos de enganche para que otros componentes le proporcionen la funcionalidad
que necesita.

Este cambio, trivial en apariencia, permite aislar los elementos más estables de aquellos más proclives al cambio. Además, facilita sobremanera el desarrollo y las pruebas del
software, además de permitir reutilizar código en otro contexto operativo: cambiar la base de datos por servicios web o pasar de una interfaz web a una de escritorio, por ejemplo,
podría hacerse sin tocar la capa de negocio. Esto es lo que se conoce como arquitectura de puertos y adaptadores o [arquitectura hexagonal](http://alistair.cockburn.us/Hexagonal+architecture).

La inversión de dependencias permite aislar componentes, aislar componentes permite acotar los cambios y acotar los cambios reduce los errores. Dicho de otro modo, la inversión
de dependencias ayuda a controlar la entropía del software y así poder dedicar más neuronas a tomar elecciones y menos a arrepentirse.

***
\* *Normalmente es alguien muy inteligente que se ha encontrado con el marrón de mantener el proyecto y que, por supuesto, no habría cometido los mismos errores si hubiese
estado al cargo desde el principio...*