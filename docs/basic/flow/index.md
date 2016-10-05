---
layout: single
title: "Guia Rápido (Flow)"
permalink: /docs/basic/flow
sidebar:
  nav: "docs"
---

Para iniciar o seu entendimento deste projeto, aqui está um guia sumarizando o uso do **Singular Flow**.

### Configuração

O primeiro passo é fornecer uma implementação para a classe `SingularFlowConfigurationBean`. Para tal pode-se usar
a implementação fornecida pela classe `HibernateSingularFlowConfigurationBean`.

Declare o seguinte **bean** no arquivo `applicationContext.xml`:

{% highlight xml %}
<context:component-scan base-package="br.net.mirante.singular"/>
<bean name="singularFlowConfigurationBean" class="br.net.mirante.singular.persistence.util.HibernateSingularFlowConfigurationBean">
    <property name="sessionFactory" ref="sessionFactory"/>
    <property name="definitionsBasePackage" value="br.net.mirante.singular"/>
    <property name="userService" ref="userService"/>
</bean>
{% endhighlight %}

O serviço `userService` deve implementar a interface `IUserService`.

Por fim, é preciso fornecer uma implementação para o agendador de processos. A implementação padrão fornecida usa
a **API Quartz**. Se desejar usá-la, basta configurar o **Quartz**. Uma forma de realizar essa configuração é
através da adição do arquivo `quartz.properties` ao projeto. Para mais informações consulte a documentação da
**API Quartz** [aqui][APIQuartz].

Exemplo de uma configuração mínima para a **API Quartz** a seguir:

{% highlight properties %}
org.quartz.scheduler.instanceName=SingularFlowSchedulerDev
org.quartz.scheduler.instanceId=DEV
org.quartz.jobStore.class=org.quartz.simpl.RAMJobStore
{% endhighlight %}

### Implementação

A implementação do `userService` pode ser tão simples como a que se segue:

{% highlight java %}
@Named
@Transactional
public class SingularDefaultUserService implements IUserService {

    @Inject
    private ActorDAO actorDAO;

    @Override
    public MUser getUserIfAvailable() {
        MUser muser = /* lógica para recuperar o usuário logado */;
        return muser;

    }

    @Override
    public boolean canBeAllocated(MUser mUser) {
        return true;
    }

    @Override
    public MUser findUserByCod(String username) {
        return actorDAO.buscarPorCodUsuario(username);
    }

    @Override
    public MUser saveUserIfNeeded(MUser mUser) {
        return actorDAO.saveUserIfNeeded(mUser);
    }

    @Override
    public MUser saveUserIfNeeded(String codUsuario) {
        return actorDAO.saveUserIfNeeded(codUsuario);
    }
}

{% endhighlight %}

Agora vamos criar um fluxo. Para isso criaremos uma classe que herda de `ProcessDefinition<ProcessInstance>`.
Por exemplo:

{% highlight java %}
public class Solicitacao extends ProcessDefinition<ProcessInstance> {
    /* Implementação... */
}
{% endhighlight %}

Primeiramente criaremos as tarefas deste fluxo como se segue:

{% highlight java %}
public enum SolicitacaoTask implements ITaskDefinition {
    AGUARDANDO_ANALISE("Aguardando análise"),
    APROVADO("Aprovado"),
    REPROVADO("Reprovado");

    private final String name;

    SolicitacaoTask(String name) {
        this.name = name;
    }

    @Override
    public String getName() {
        return name;
    }
}
{% endhighlight %}

Agora basta implementar o método `createFlowMap` como se segue:

{% highlight java %}
@Override
protected FlowMap createFlowMap() {
    setName("Geral", "Solicitação");

    FlowBuilderImpl flow = new FlowBuilderImpl(this);
    
    flow.addPeopleTask(AGUARDANDO_ANALISE);
    flow.addEnd(APROVADO);
    flow.addEnd(REPROVADO);
    
    flow.setStartTask(AGUARDANDO_ANALISE);

    flow.from(AGUARDANDO_ANALISE).go(APROVAR, APROVADO);
    flow.from(AGUARDANDO_ANALISE).go(REPROVAR, REPROVADO);
    
    flow.addEnd(APROVADO);
    flow.addEnd(REPROVADO);

    return flow.build();
}
{% endhighlight %}

Pronto! Para criar e disparar uma instância da nossa solicitação fazemos apenas o seguinte:

{% highlight java %}
/* Disparando um novo processo... */
public ProcessInstance startInstance() {
    Solicitacao solicitacao = new Solicitacao();
    ProcessInstance id = solicitacao.newInstance();
    id.start();
    return id;
}
{% endhighlight %}

<div class="note info">
  <h5>Dica para o Desenvolvedor</h5>
  <p>
    Existe um módulo dedicado para suportar testes de integração e unitários. Este módulo
    usa a <strong>API Spring Test</strong> e um banco <strong>HSQL</strong>. Para mais detalhes
    carregue o módulo <code>flow-test</code>.
  </p>
</div>

A documentação detalhada será apresentada na seção [Singular Flow][SingularFlow].

[APIQuartz]: http://quartz-scheduler.org/documentation/quartz-2.x/configuration
[SingularFlow]: /docs/flow/
