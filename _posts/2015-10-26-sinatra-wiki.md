---
layout: post
title: "Montando una wiki con Sinatra"
date: 2015-10-26
tags: Ruby Sinatra Markdown
---
Sinatra es un micro-framework escrito en Ruby que permite crear aplicaciones web con muy pocas líneas de código. Si se combina con Markdown, se
puede crear muy fácilmente una pequeña aplicación capaz de parsear ficheros .md y hacerlos accesibles a través del navegador.

Un servidor tiene la costumbre de tomar notas y apuntes en ficheros de texto plano, que son considerablemente más cómodos de editar que un .doc. Como quiera
que la sintaxis de Markdown es muy intuitiva, es bastante fácil redactar en dicho formato y así poder luego abrir los ficheros en cualquier navegador
con el plugin adecuado (para Chrome hay varios e imagino que lo mismo ocurrirá con el resto de navegadores). No obstante, llevo tiempo con ganas de trastear
con Sinatra y esta me pareció la excusa perfecta: en lugar de abrir los ficheros .md en el navegador, voy a montar una especie de wiki de andar por casa, una
aplicación que liste los ficheros de mi directorio de notas y me permita acceder a ellos, parseando al vuelo el contenido. Vamos con el briconsejo.

Lo primero y más básico es poder acceder a un documento desde el navegador. Para ello, haremos algo tan simple como pasar al nombre del fichero como parte
de la ruta para que la aplicación lo busque en disco, parsee su contenido y lo escupa de vuelta al navegador.

Antes de nada crearemos un fichero llamado *wiki.rb* en el que haremos nuestras cosillas. Y ahora, vamos por partes.

En primer lugar, hagamos que nuestra aplicación reciba el nombre del fichero que queremos mostrar:

{% highlight ruby %}
require 'sinatra'

get '/:article' do
  @article = params['article']
  erb :article
end
{% endhighlight %}

Este pedazo de código es simple como el mecanismo de un botijo. Recibe por parámetro una cadena (el nombre del fichero, vaya) y construye la vista a partir de una plantilla *erb*. Fácil.

Para hacer la plantilla de marras hemos de crear un nuevo directorio llamado *views*, ya que, por defecto, Sinatra busca las plantillas en esa ruta. En nuestro
nuevo directorio crearemos un fichero llamado *article.erb*, que tendrá el siguiente contenido:

{% highlight html %}
<article>
    <%= @article %>
</article>
{% endhighlight %}

Lo único que se hace aquí es renderizar la variable **@article** que contiene el nombre del fichero. Si ahora ejecutamos el comando `ruby wiki.rb`, podremos acceder
desde el navegador a la url localhost:4567/mi-articulo para ver una pantalla en blanco con el texto "mi-articulo". De momento, nada demasiado espectacular.

Vamos con la segunda parte. Tenemos el nombre del fichero, pero necesitamos parsear su contenido y mostrarlo en pantalla. Para esto haremos uso de la gema [*Rdiscount*](http://dafoster.net/projects/rdiscount/).
Primero, una pequeña modificación en el método que recibe la petición:

{% highlight ruby %}
require 'sinatra'
require 'rdiscount'

get '/:article' do
  @article = "../articles/#{params['article']}"
  erb :article
end
{% endhighlight %}

En lugar de devolver simplemente el nombre del fichero, vamos a devolver su ruta. He decidido que voy a guardar los ficheros .md en el directorio *articles*, pero obviamente podría ser
cualquier otro. Ahora crearemos el directorio en cuestión y un fichero de prueba, al que llamaremos *test.md*:

    #Mi primer artículo

    Esto es una lista:

    - Uno
    - Dos
    - Tres

Ahora solo queda hacer otro pequeño cambio en la plantilla y todo listo:

{% highlight html %}
<article>
    <%=  markdown(:"#{@article}") %>
</article>
{% endhighlight %}

Lo que se hace ahora en la plantilla es invocar a la librería *Rdiscount* pasando como parámetro la ruta del documento a parsear. *Et voilà!* si accedemos a la url
localhost:4567/test veremos nuestro fichero en todo su esplendor.

Todo esto está muy bien, pero lo suyo sería poder ver un listado de los ficheros disponibles, por ejemplo en la ruta raíz de la aplicación. Vamos con ello.
Primero creamos un nuevo método controlador para la ruta raíz:

{% highlight ruby %}
get '/' do
  @articles = Dir.entries("articles")
  erb :index
end
{% endhighlight %}

Simplemente listamos todas las entradas del directorio "articles" y las devolvemos al navegador. La plantilla *index.erb* es muy simple:

{% highlight html %}
<ul>
<% @articles.each do |article| %>
    <li>
        <a href='/<%= article %>'><%= article.gsub('_', ' ').capitalize %></a>
    </li>
<% end %>
</ul>
{% endhighlight %}

Aquí se tunean un poco los nombres de los ficheros para hacerlos más legibles; básicamente, se eliminan los guiones bajos y se pone la inicial en mayúsculas.

Hecho esto, si accedemos a la url localhost:4567 veremos un listado con los ficheros del directorio. Y aquí surgen dos problemas: primero, aparecen las entradas del
directorio actual y el directorio padre, mala cosa; segundo, aparece la extensión de los ficheros, que no es malo pero es feo.

Para arreglar esto hacen falta un par de modificaciones en el controlador de la ruta raíz:

{% highlight ruby %}
get '/' do
  @articles = Dir.entries("articles")
  @articles = @articles.reject { |entry| /^\.+$/.match entry } # Rejects . and .. entries
  @articles = @articles.map { |entry| entry.sub('.md', '') } # Removes .md extension
  @articles = @articles.sort { |a, b| a <=> b } # Sorts by name
  erb :index
end
{% endhighlight %}

En resumen, se eliminan las entradas del directorio actual y el directorio padre, se eliminan las extensiones de los ficheros y, de propina, se ordenan por orden
alfabético. Si accedemos a la ruta raíz de nuevo, veremos todo como debe ser.

Como además somos gente elegante, vamos a hacer unas pequeñas refactorizaciones en los controladores para que queden limpitos y legibles:

{% highlight ruby %}
require 'sinatra'
require 'rdiscount'

ARTICLES_PATH = "articles"

get '/:article' do
  @article = "../#{ARTICLES_PATH}/#{params['article']}"
  erb :article
end

get '/' do
  @articles = Dir.entries(ARTICLES_PATH)
    .reject { |entry| /^\.+$/.match entry } # Rejects . and .. entries
    .map { |entry| entry.sub('.md', '') } # Removes .md extension
    .sort { |a, b| a <=> b } # Sorts by name
  erb :index
end
{% endhighlight %}

Y listo. A mayores de todo esto, se podría hacer un formulario para crear nuevos ficheros a través de la aplicación o editar los ya existentes, y tener así una wiki
basada totalmente en ficheros de texto. El código fuente de todo el invento está disponible [aquí](https://github.com/megalomono/wikinatra).