---
layout: single
title: "Configuração"
permalink: /docs/basic/configuration
sidebar:
  nav: "docs"
---

Esta seção mostrará como configurar cada um dos módulos **Java** do **Projeto Singular**. Essas configurações
usam outras **API's**, cujas referências estão listadas a seguir:

### Links

* Spring:
    * [Docs](https://spring.io/docs)
    * [Getting Started Guides](https://spring.io/guides#gs)
    * [Reference Documentation](https://spring.io/docs/reference)
* Quartz
    * [Documentation](http://quartz-scheduler.org/documentation)
    * [Configuration Reference](http://quartz-scheduler.org/documentation/quartz-2.x/configuration/)
* Wicket
    * [Documentation](https://wicket.apache.org/learn/)

## Dependências

Primeiramente inclua as dependências necessárias à aplicação. A forma mais fácil de realizar esta operação
é com o uso do [MAVEN][MAVEN].

## Singular Flow

O próximo passo é fornecer uma implementação para a classe `SingularFlowConfigurationBean`. Para tal pode-se usar
a implementação fornecida pela classe `HibernateSingularFlowConfigurationBean` [javadoc&nbsp;&rarr;][javadocHibernateSingularFlowConfigurationBean]

Uma aplicação que usa a **API Spring** pode simplesmente definir o seguinte **bean**
no arquivo `applicationContext.xml`:

{% highlight xml %}
<context:component-scan base-package="br.net.mirante.singular"/>
<bean name="singularFlowConfigurationBean" class="br.net.mirante.singular.persistence.util.HibernateSingularFlowConfigurationBean">
    <property name="sessionFactory" ref="sessionFactory"/>
    <property name="definitionsBasePackage" value="br.net.mirante.singular"/>
    <property name="userService" ref="userService"/>
</bean>
{% endhighlight %}

O serviço `userService` deve implementar a interface `IUserService` [javadoc&nbsp;&rarr;][javadocIUserService]

### Flow Schedule

Por fim, é preciso fornecer uma implementação para o agendador de processos. A implementação padrão fornecida usa
a **API Quartz**. Se desejar usá-la, basta configurar o **Quartz**. Uma forma de realizar essa configuração é
através da adição do arquivo `quartz.properties` à aplicação.

Exemplo de uma configuração mínima para a **API Quartz** a seguir:

{% highlight properties %}
org.quartz.scheduler.instanceName=SingularFlowSchedulerDev
org.quartz.scheduler.instanceId=DEV
org.quartz.jobStore.class=org.quartz.simpl.RAMJobStore
{% endhighlight %}

Para mais detalhes veja o **javadoc** da classe [QuartzSchedulerFactory][javadocQuartzSchedulerFactory].

### Flow UI-Admin

Este módulo é usado como um _overlay_ **MAVEN** de outros sistemas, mas também pode ser
implantado de forma _standalone_.

Para embutir este módulo em sua aplicação basta adicionar as seguintes dependências [MAVEN][MAVEN]:

{% highlight xml %}
<dependency>
    <groupId>br.net.mirante</groupId>
    <artifactId>flow-core</artifactId>
    <version>${singular.version}</version>
</dependency>
<dependency>
    <groupId>br.net.mirante</groupId>
    <artifactId>flow-persistence</artifactId>
    <version>${singular.version}</version>
</dependency>
<dependency>
    <groupId>br.net.mirante</groupId>
    <artifactId>flow-ui-admin</artifactId>
    <version>${singular.version}</version>
    <type>war</type>
</dependency>
<dependency>
    <groupId>br.net.mirante</groupId>
    <artifactId>flow-ui-admin</artifactId>
    <version>${singular.version}</version>
    <classifier>classes</classifier>
    <scope>provided</scope>
</dependency>
{% endhighlight %}

O módulo `flow-ui-admin` usa a **API yFiles**. Se essa **API** não estiver instalada no servidor de aplicação,
também adicione as seguintes dependências:

{% highlight xml %}
<dependency>
    <groupId>y.ed</groupId>
    <artifactId>yfiles</artifactId>
    <version>2.12.1</version>
</dependency>
<dependency>
    <groupId>y.ed</groupId>
    <artifactId>ybpmn</artifactId>
    <version>2.1.1</version>
</dependency>
{% endhighlight %}

O passo seguinte é configurar a aplicação via **API Spring**. Inclua a seguinte linha no `applicationContext.xml`
da aplicação:

{% highlight xml %}
<import resource="singularUIAdminContext.xml"/>
{% endhighlight %}

O conteúdo do arquivo `singularUIAdminContext.xml` deve se parecer com o seguinte:

{% highlight xml %}
<import resource="conf/adminDefault.xml"/>

<alias name="dataSource" alias="dataSourceSingular"/>

<bean id="uiAdminWicketFilterContext" class="br.net.mirante.singular.wicket.UIAdminWicketFilterContext">
    <constructor-arg value="admin"/>
</bean>

<bean id="uiAdminFacade" class="com.application.singular.SingularAdminFacade"/>
{% endhighlight %}

No exemplo acima a aplicação já possuia um _bean_ chamado **dataSource** do tipo `javax.sql.DataSource`.
Opcionalmente, como no exemplo acima, a _facade_ `br.net.mirante.singular.service.UIAdminFacade` pode
ser customizada [javadoc&nbsp;&rarr;][javadocUIAdminFacade]

Por fim, precisamos configurar o filtro da **API Wicket** usada por este módulo. Adicione o seguinte
filtro no arquivo `web.xml`:

{% highlight xml %}
<filter>
    <filter-name>admin</filter-name>
    <filter-class>org.apache.wicket.protocol.http.WicketFilter</filter-class>
    <init-param>
        <param-name>applicationClassName</param-name>
        <param-value>com.miranteinfo.alocpro.wicket.SingularAdminApp</param-value>
    </init-param>
    <init-param>
        <param-name>filterMappingUrlPattern</param-name>
        <param-value>/admin/*</param-value>
    </init-param>
</filter>
{% endhighlight %}

Nesse exemplo a interface pode ser acessa através da URL `<url_da_aplicacao>/admin`.

#### Dicas Importantes

Por padrão, o modo iniciado é o de produção. Nesse modo apenas usuários autenticados com
o papel **ADMIN** podem acessar o sistema.

<div class="note">
  <h5>Nota para o Desenvolvedor</h5>
  <p>
    Para iniciar em modo de desenvolvimento defina a variável de sistema <strong>singular.development</strong>.
    Por exemplo, passe o seguinte argumento para o <strong>Tomcat</strong>:
    <code>-Dsingular.development=true</code>
  </p>
</div>

[MAVEN]: /docs/maven
[javadocIUserService]: /javadoc/index.html?br/net/mirante/singular/flow/core/service/IUserService.html
[javadocHibernateSingularFlowConfigurationBean]: /javadoc/index.html?br/net/mirante/singular/persistence/util/HibernateSingularFlowConfigurationBean.html
[javadocUIAdminFacade]: /javadoc/index.html?br/net/mirante/singular/service/UIAdminFacade.html
[javadocQuartzSchedulerFactory]: /javadoc/index.html?br/net/mirante/singular/flow/schedule/quartz/QuartzSchedulerFactory.html
