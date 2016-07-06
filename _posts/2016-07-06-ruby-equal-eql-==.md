---
layout: post
title: "equal?, eql? y =="
date: 2016-07-06
tags: Ruby
---
A la hora de comparar dos objetos, Ruby ofrece tres opciones: *equal?*, *eql?* y *==*. ¿Qué les diferencia? En principio nada:
los tres métodos comprueban que los dos objetos comparados sean el mismo. El meollo del asunto está en el uso que hacen las propias
librerías de Ruby de estos métodos, y las implicaciones que tiene sobreescribirlos.

Supongamos que tenemos la siguiente clase:

{% highlight ruby %}
class Coordinate

  attr_reader :x, :y
  
  def initialize(x, y)
    @x = x
    @y = y
  end

end
{% endhighlight %}

Primera prueba:

    irb(main):002:0> c_1 = Coordinate.new 1, 0
    => #<Coordinate:0x000000033302e0 @x=1, @y=0>
    irb(main):003:0> c_2 = Coordinate.new 1, 0
    => #<Coordinate:0x000000032a9920 @x=1, @y=0>
    irb(main):004:0> c_1 == c_2
    => false
    irb(main):005:0> c_1.eql? c_2
    => false
    irb(main):006:0> c_1.equal? c_2
    => false

Ok, era previsible. Como tiene sentido que dos coordenadas con los mismos valores sean iguales, independientemente de su posición en memoria,
vamos a sobreescribir el método *==*:

{% highlight ruby %}
class Coordinate

  attr_reader :x, :y

  def initialize(x, y)
    @x = x
    @y = y
  end
  
  def ==(other)
    @x == other.x && @y == other.y
  end

end
{% endhighlight %}

Veamos que sucede ahora:

    irb(main):003:0> c_1 = Coordinate.new 1, 0
    => #<Coordinate:0x000000032bbf08 @x=1, @y=0>
    irb(main):004:0> c_2 = Coordinate.new 1, 0
    => #<Coordinate:0x00000002b94400 @x=1, @y=0>
    irb(main):005:0> c_1 == c_2
    => true
    irb(main):006:0> c_1.eql? c_2
    => false
    irb(main):007:0> c_1.equal? c_2
    => false

Tiene sentido. Pero ¿y si hacemos lo siguiente?:

    irb(main):008:0> my_hash = Hash.new
    => {}
    irb(main):009:0> my_hash[c_1] = 'coordenada 1'
    => "coordenada 1"
    irb(main):010:0> my_hash[c_2]
    => nil

¡Ouch! Parece que *Hash* no utiliza el método *==* a la hora de comparar los objetos que emplea como claves. ¿Cómo carajo compara las claves entonces?
Un rápido vistazo a la documentación de Ruby nos aclara el asunto:

> Two objects refer to the same hash key when their hash value is identical and the two objects are eql? to each other.

Es decir, que toca sobreescribir los métodos *eql?* y *hash*. Para ello, creamos un alias del método *eql?* al método *==*, ya que parece razonable
que ambos devuelvan el mismo valor. Para el método *hash* vamos a devolver el *hash* del array de atributos del objeto, pero se podría generar de otro
modo:

{% highlight ruby %}
class Coordinate

  attr_reader :x, :y
  
  def initialize(x, y)
    @x = x
    @y = y
  end
  
  def ==(another_coordinate)
    @x == another_coordinate.x && @y == another_coordinate.y
  end
  
  alias :eql? :==
  
  def hash
    [@x, @y].hash
  end
  
end
{% endhighlight %}

Veamos ahora:

    irb(main):002:0> c_1 = Coordinate.new 1, 0
    => #<Coordinate:0x0000000076bbf0 @x=1, @y=0>
    irb(main):003:0> c_2 = Coordinate.new 1, 0
    => #<Coordinate:0x000000007a6980 @x=1, @y=0>
    irb(main):004:0> my_hash = Hash.new
    => {}
    irb(main):005:0> my_hash[c_1] = 'coordenada 1'
    => "coordenada 1"
    irb(main):006:0> my_hash[c_2]
    => "coordenada 1"

Esto tiene mejor pinta.

La conclusión de todo esto es que *==* es el método tradicionalmente sobreescrito para dotar a la comparación
de sentido dentro del dominio en el que se está trabajando.

En cuanto a los métodos *eql?* y *hash*, suele ser buena idea sobreescribirlos tal y como se ha hecho en el ejemplo, ya que otras clases
emplean estos métodos para determinar si dos objetos son iguales. En este caso la liebre saltó al emplear las coordenadas como clave en un hash,
pero también puede darse al calcular la intersección de dos arrays con el método *&*, por ejemplo.

¿Qué sucede con el método *equal?*? Algo similar a lo que sucede con *eql?*: otras clases lo emplean para determinar si dos objetos son el mismo (que no iguales),
de modo que no es buena idea sobreescribirlo.

Para más info, echa un ojo [aquí](http://ruby-doc.org/core-2.2.0/Object.html#method-i-eql-3F).