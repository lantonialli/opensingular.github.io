---
layout: single
title: "Singular Flow"
permalink: /docs/flow
sidebar:
  nav: "docs"
---

  Por meio do Singular Flow é possível automatizar os mais diversos fluxos de negócio. A definição do fluxo é feita de maneira programática reduzindo drasticamente a impedância entre definições declarativas e sua respectiva interpretação pelo motor de fluxo.
  
  A seguir serão apresentados os principais conceitos do Singular Flow exemplificando como esses conceitos podem ser aplicados.
  
  
# Definição do Fluxo

O ponto de partida da automação de um fluxo no Singular Flow é a criação de uma definição. Uma definição é uma classe pública, concreta, com um construtor sem parâmetros e subclasse de `br.net.mirante.singular.flow.core.ProcessDefinition`.

Essa subclasse precisa satisfazer:

1. Ao menos um dos construtores da superclasse. O mais simples deles solicita um identificador `java.lang.String` único e imutável e a classe utilizada para representar as instâncias do fluxo. Frequentemente o segundo parâmetro é a classe padrão  `br.net.mirante.singular.flow.core.ProcessInstance`.

2. Fornecer implementação para o método `protected FlowMap createFlowMap();`. Esse método deve retornar uma definição sintáticamente consistente do fluxo de negócio.

Veja o exemplo abaixo:
{% highlight java %}
public class SolicitacaoPassaporte extends ProcessDefinition<ProcessInstance> {

    public SolicitacaoPassaporte() {
       super("SolicitacaoPassaporte", ProcessInstance.class);
    }

    @Override
    protected FlowMap createFlowMap() {...}

}
{% endhighlight %}

## FlowMap

 O Objeto do tipo `br.net.mirante.singular.flow.core.FlowMap` contém a definição completa do fluxo.
 
  No Singular Flow um fluxo é um conjunto de transições (*transitions*) e tarefas (*tasks*) que forma um grafo direcionado contendo ao menos uma atividade de início  (*start*), uma atividade de fim (*end*) e um caminho (conjunto de transições) capaz de alcançar uma atividade fim a partir de uma atividade de início.
  
  
### Tarefas

Uma tarefa ou *task* é um estado do fluxo e representa uma etapa do fluxo de negócio. As tarefas do Singular Flow compartilham duas características básicas:

* Descrição da tarefa.
* Identificador `java.lang.String` único e imutável (para um mesmo fluxo).

 Para criar uma nova tarefa é preciso informar essas duas características fornecendo uma implementação da interface `br.net.mirante.singular.flow.core.builder.ITaskDefinition`
 
 

<div class="note">
<h5>TaskDefinition Lambda</h5>
  <p>
	TaskDefinition é uma interface funcional é seu uso pode ser substituído por um <code>lambda</code> (Supplier) que retorna apenas a descrição da tarefa. Nesse caso a chave é derivada a partir da descrição: <code>() -> "Analisar solicitação"</code>
</p>

</div>

O Singular Flow conta com 4 tipos básicos de tarefas (*tasks*): Java, People, Wait e End. Cada tarefa tem características e configurações específicas como detalhado nas seções a seguir.

#### Tarefas Java

Tarefas java são tarefas de execução automática de código java. Frequentemente esse tipo de tarefa é utilizado para fazer integrações: chamada a serviços, cargas em banco de dados, envios de e-mail etc. Essa tarefa é executada periodicamente pelo motor de fluxo até que o código executado finalize com sucesso.
 
O código a ser executado é uma instância de `br.net.mirante.singular.flow.core.MTaskJava.ImplTaskJava`. Essa implementação é usualmente substituída por um método ou lambda que recebe como parâmetro um objeto do tipo `br.net.mirante.singular.flow.core.ExecutionContext` e retorna um `java.lang.Object` que representa uma decisão do fluxo ou *null* para seguir para a próxima tarefa.

Veja o exemplo a seguir:

{% highlight java %}
...
    protected FlowMap createFlowMap() {
    	FlowMap map = new FlowMap(this);
    	...
	map.addJavaTask(() -> "Enviar E-mail").call( ctx -> {
            enviarEmail(ctx.getProcessInstance().getDescription(),"fulano@cicrano.com");
            return null;//null executa a transicao default;
        });
...
{% endhighlight %}

#### Tarefas People

Essas são as tarefas destinadas a serem executadas por pessoas. Usualmente essas tarefas envolver a visualização e/ou complementação de dados e/ou uma tomada de decisão.
A tomada de decisão define o caminho pelo qual o fluxo deve seguir e a complementação dos dados é normalmente resultado de análise humana.

Para esse tipo de tarefa é necessário informar a página que será apresentada ao usuário no momento da execução da tarefa. Para isso é preciso fornecer uma implementação da interface `br.net.mirante.singular.flow.core.ITaskPageStrategy`.

Veja o exemplo a seguir:

{% highlight java %}
...
    protected FlowMap createFlowMap() {
    	FlowMap map = new FlowMap(this);
    	...
	map.addPeopleTask(() -> "Aguardar ajuste")
	.setExecutionPage(
		SingularServerTaskPageStrategy.of(AnalisarPassaportePage.class)
	);
...
{% endhighlight %}

No exemplo acima foi utilizada uma implementação de `ITaskPageStrategy` que designa uma página Wicket para ser apresentada no momento da execução da tarefa.


#### Tarefas Wait

Esse tipo de tarefa indica que o fluxo deve ficar parado nessa tarefa até que um dos três casos ocorra:

1. A tarefa seja notificada que deve dar continuidade ao fluxo (geralmente isso é feito por meio de uma página por interação do usuário).
2. Uma data alvo seja alcançada.


{% highlight java %}
...
    protected FlowMap createFlowMap() {
    	FlowMap map = new FlowMap(this);
    	...
    	/*caso 1*/	
	map.addWaitTask(() -> "Aguardar ajuste")
	.setExecutionPage(
		SingularServerTaskPageStrategy.of(SolicitarPassaportePage.class)
	);
	/*caso 2*/
	map.addWaitTask(() -> "Aguardar 30 dias")
	.withTargetDate((processInstance, taskInstance) -> {
            System.out.println("30 dias se passaram?");
            return Date.from(
            	processInstance.getBeginDate().toInstant().plus(30, DAYS)
            	);
        });
...
{% endhighlight %}

#### Tarefas End

Esse é o tipo mais simples de tarefa. Trata-se de uma tipo de marcação para indicar que o fluxo chegou a um ponto de parada. É possível definir múltiplos pontos de parada.

{% highlight java %}
...
    protected FlowMap createFlowMap() {
    	FlowMap map = new FlowMap(this);
    	...
	map.addEnd(() -> "Reprovado");
...
{% endhighlight %}



<div class="note info">
<h5>Enum para TaskDefinition</h5>
  <p>
  	É uma boa prática definir uma <i>inner enum</i> na classe de definição do fluxo para declarar as diferentes definições de tarefas do fluxo:
  
</p>

</div>

{% highlight java %}
public class SolicitacaoPassaporte extends ProcessDefinition<ProcessInstance> {

   public enum PeticaoTask implements ITaskDefinition {   
        ANALISAR("Analisar solicitação"),
	COMPLEMENTAR("Ajustar solicitação"),
	APROVADO("Aprovado"),
	REPROVADO("Reprovado");
        private final String name;
        PeticaoTask(String name) { this.name = name;}
        @Override
        public String getName() {return name;}
    }
	...

}
{% endhighlight %}

### Transições
  
 A transições ligam um estado a outro. Para definir uma transição basta associar um estado a outro. É possível também fornecer um nome a esta transição. Veja o exemplo a seguir:
 
 
{% highlight java %}
...
    protected FlowMap createFlowMap() {
        FlowMap map = new FlowMap(this);
   	 ...
    	MTaskJava email = map.addJavaTask(() -> "Enviar E-mail").call( ctx -> {
            enviarEmail(ctx.getProcessInstance().getDescription());
            return null;//null executa a transicao default;
        });
   	MTaskEnd reprovado = map.addEnd(() -> "Reprovado");
    
   	 /*transicao da tarefa java 'Enviar E-mail' para a tarefa fim 'Reprovado'*/
   	email.addTransition(reprovado);
...
{% endhighlight %}
    
<div class="note">
<h5>Recapitulando</h5>
  <p>
(...) um fluxo é um conjunto de transições e tarefas que forma um grafo direcionado contendo ao menos uma atividade de início, uma atividade de fim e um caminho (conjunto de transições) capaz de alcançar uma atividade fim a partir de uma atividade de início.(...)</p>
</div>
  
### Construção de um FlowMap

 A classe `FlowMap` contém todos os métodos necessários para montar a definição do fluxo. Embora a montagem de um fluxo utilizando as interfaces do `FlowMap` seja simples, recomendamos o uso de algum *Builder* para auxiliar essa tarefa. Até então os exemplos apresentados utilizam exclusivamente a API do FlowMap. O Singular Flow disponibiliza um *Builder* default:  `br.net.mirante.singular.flow.core.builder.FlowBuilderImpl` 
 Esse *Builder* pode ser substituído por implementações customizadas para as necessidades de cada cliente.

  
## Juntando todas as partes

Com base nos conceitos apresentados até então montamos um exemplo de um fluxo onde:

* Uma pessoa que submete dados à polícia federal para solicitação de passaporte.
* Um analista da polícia federal é responsável por verificar a consistência desses dados.
* O analista da polícia federal pode solicitar complementação ou ajuste das informações.
* Ao final o analista decide se o solicitante está apto ou não a receber o passaporte.


{% highlight java %}
public class SolicitacaoPassaporte extends ProcessDefinition<ProcessInstance> {

   public enum PeticaoTask implements ITaskDefinition {   
        ANALISAR("Analisar solicitação"),
	COMPLEMENTAR("Ajustar solicitação"),
	APROVADO("Aprovado"),
	REPROVADO("Reprovado");
        private final String name;
        PeticaoTask(String name) { this.name = name;}
        @Override
        public String getName() {return name;}
    }
    
    public SolicitacaoPassaporte() {
       super("SolicitacaoPassaporte", ProcessInstance.class);
    }
    
    protected FlowMap createFlowMap() {
        FlowBuilderImpl flow = new FlowBuilderImpl(this);
        /*tarefas*/
        flow.addPeopleTask(ANALISAR)
           .withExecutionPage(
           	SingularServerTaskPageStrategy.of(AnalisarPassaportePage.class));
        flow.addWaitTask(COMPLEMENTAR))
           .withExecutionPage(
           	SingularServerTaskPageStrategy.of(ComplementarSolicitacao.class))
        flow.addEnd(APROVADO);
        flow.addEnd(REPROVADO);
        
	/*transicoes*/
	flow.from(ANALISAR).go("Solicitar Ajustes", COMPLEMENTAR);
	flow.from(ANALISAR).go("Aprovar", APROVADO);
	flow.from(ANALISAR).go("Reprovar", REPROVADO);
	flow.from(COMPLEMENTAR).go("Finalizar Complementação", ANALISAR);
	
	flow.setStartTask(ANALISAR);	
        return flow.build();
    }
}
{% endhighlight %}

No exemplo acima apresenta um fluxo definido com o *Builder* default do Singular Flow.

 A Figura abaixo ilustra o fluxo definido acima.

![Fluxo de Passaporte](/img/fluxo-passaporte.png)

