---
layout: post
title: "Múltiples filtros de seguridad con SpringBoot"
date: 2015-07-02
tags: Spring
---
En una aplicación web típica suelen realizarse dos tipos de peticiones: las que recargan la página
completa y las peticiones AJAX. Estas dos variantes requieren un control de seguridad distinto, ya que
mientras una petición de carga de página será redirigida en caso de no cumplir los criterios de seguridad,
una petición ajax debe recibir como respuesta un error 401.

Esto es fácil de controlar con SpringBoot y Spring Security; simplemente hay que configurar dos filtros
*http*, uno para cada tipo de peticion, numerándolos con la anotación *@Order*. Así, el filtro para las
peticiones AJAX quedaría de la siguiente manera:

{% highlight java %}
@Configuration
@Order(1)
public static class ApiWebSecurityConfig extends
        WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.antMatcher("/rest/**").authorizeRequests().anyRequest()
                .authenticated().and().httpBasic();
    }
}
{% endhighlight %}
    
En este caso, las peticiones AJAX se realizarán contra URLs que comiencen por */rest*, mientras que el
resto de peticiones serán filtradas por el segundo adaptador:

{% highlight java %}
@Configuration
@Order(2)
public static class FormWebSecurityConfig extends
        WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        // Los recursos estáticos no requieren autenticación
        http.authorizeRequests()
                .antMatchers("/css/**", "/js/**", "/img/**", "/libs/**")
                .permitAll();

        // El resto de URLs requieren que el usuario esté autenticado
        http.authorizeRequests().anyRequest().authenticated().and()
                .formLogin().usernameParameter("email").loginPage("/login")
                .permitAll().and().logout().permitAll();
    }

}
{% endhighlight %}
    
Con los dos adaptadores y la configuración del servicio de autenticación de usuarios,
la clase completa de configuración de Spring Security sería algo tal que así:
    
{% highlight java %}
@Configuration
@EnableWebMvcSecurity
public class SecurityConfig {

    @Autowired
    private UserDetailsService userDetailsService;

    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth)
            throws Exception {
        auth.userDetailsService(userDetailsService).passwordEncoder(
                new BCryptPasswordEncoder());
    }

    @Configuration
    @Order(1)
    public static class ApiWebSecurityConfig extends
            WebSecurityConfigurerAdapter {

        @Override
        protected void configure(HttpSecurity http) throws Exception {
            http.antMatcher("/rest/**").authorizeRequests().anyRequest()
                    .authenticated().and().httpBasic();
        }
    }

    @Configuration
    @Order(2)
    public static class FormWebSecurityConfig extends
            WebSecurityConfigurerAdapter {

        @Override
        protected void configure(HttpSecurity http) throws Exception {
            http.authorizeRequests()
                    .antMatchers("/css/**", "/js/**", "/img/**", "/libs/**")
                    .permitAll();

            http.authorizeRequests().anyRequest().authenticated().and()
                    .formLogin().usernameParameter("email").loginPage("/login")
                    .permitAll().and().logout().permitAll();
        }

    }
}
{% endhighlight %}