---
layout: post
title: "Configuración del conector AJP con SpringBoot"
date: 2015-08-30
tags: Spring
---
Una instalación típica de un servidor Tomcat acostumbra a correr detrás de un servidor Apache que actúa
como balanceador de carga, redistribuyendo las peticiones HTTP mediante un mecanismo de enrutación. Para
enviar las peticiones desde el servidor web al contenedor de servlets se emplea el protocolo AJP.

Si se emplea SpringBoot y un servidor Tomcat embebido el conector por defecto es el HTTP sobre el puerto
8080. Para configurar el conector AJP solo hay que definir el siguiente *bean*:

{% highlight java %}
@Bean
public EmbeddedServletContainerFactory tomcat() {
    TomcatEmbeddedServletContainerFactory tomcatFactory = 
        new TomcatEmbeddedServletContainerFactory();
    tomcatFactory.setProtocol("AJP/1.3");
    tomcatFactory.setPort(8009);
    return tomcatFactory;
}
{% endhighlight %}

Esto se puede hacer en cualquier clase de configuración o en la propia clase de arranque:

{% highlight java %}
@SpringBootApplication
public class MyApp {

    public static void main(String[] args) {
        SpringApplication.run(MyApp.class, args);
    }

    @Bean
    public EmbeddedServletContainerFactory tomcat() {
        TomcatEmbeddedServletContainerFactory tomcatFactory = 
            new TomcatEmbeddedServletContainerFactory();
        tomcatFactory.setProtocol("AJP/1.3");
        tomcatFactory.setPort(8009);
        return tomcatFactory;
    }
}
{% endhighlight %}