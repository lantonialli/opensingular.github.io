---
layout: single
title: "Guia Rápido (Form)"
permalink: /docs/basic/form
sidebar:
  nav: "docs"
---

A seguir serão apresentados os passos básicos para construir um formulário utilizando o Singular Form. Apesar de não ser estritamente necessário, é recomendado ler também a [documentação do Singular Form](/docs/form).

### Configuração

Para iniciar o desenvolvimento de formulários utilizando o **Singular Form**. Basta adicionar as dependências necessárias
conforme explicado [aqui][MAVEN]. Este guia usará o módulo `form-wicket` para a geração de formulário
via **API Wicket**.

### Implementação

Um formulário no Singular Form é composto por uma ou mais definições de tipos (*types*) cujo uso é restrito a formulários cujo dicionário de tipos (*dictionary*) que carregaram tais definições previamente (*type loading*) ou carregaram pacotes (*packages*) contendo tais definições. Um pacote é um agrupador de tipos.

Para construir um formulário é preciso definir um pacote a partir do qual serão carregados os tipos. Um pacote é uma subclasse pública e concreta da classe `br.net.mirante.singular.form.SPackage`.

O código abaixo exemplifica a criação de um pacote vazio (sem definição de tipos):


{% highlight java %}

public class SPackagePassaporte extends SPackage {

    public static final String PACOTE = "formulario.passaporte";
    public static final String TIPO = "SolicitacaoPassaporte";
    public static final String NOME_COMPLETO = PACOTE + "." + TIPO;

    public SPackageCancelamento() {
        super(PACOTE);
    }
    
    protected void carregarDefinicoes(PackageBuilder pb) {}
}
{% endhighlight %}

<div class="note">
<h5>Defina Constantes!</h5>
  <p> É uma boa prática no Singular Form definir o três constantes públicas como no código exemplo anterior. O nome do pacote e o nome completo do tipo raiz do pacote frequentemente precisam ser referenciados ao se utilizar funcionalidades avançadas do Singular Form.
</p>
</div>


### Definição do formulário

Todo pacote possui um tipo raiz a partir do qual os demais tipos são criados e associados.
O tipo raiz é sempre um tipo composto. A criação dos tipos de um pacote se inicia pelo método `carregarDefinicoes` do pacote.

O código abaixo exemplifica a criação de um tipo raiz:

{% highlight java %}
public class SPackagePassaporte extends SPackage {

    public static final String PACOTE = "formulario.passaporte";
    public static final String TIPO = "SolicitacaoPassaporte";
    public static final String NOME_COMPLETO = PACOTE + "." + TIPO;    
    ...
    protected void carregarDefinicoes(PackageBuilder pb) {
        final STypeComposite<?> solicitacao = pb.createCompositeType(TIPO);
        solicitacao
            .asAtr()
            .label("Solicitação de Passaporte");
    }
...
{% endhighlight %}

<div class="note info">
<h5>Título do Formulário</h5>
  <p> O rótulo (<i>label</i>) do tipo composto raiz é utilizado como título do formulário.
</p>
</div>

O exemplo utilizado é uma solicitação de passaporte onde o solicitante deve informar: nome completo, CPF e data de nascimento.
No código abaixo exemplifica a criação de um campo de nome. No método `addFieldString` é informado um identificar único e imutável para o tipo e um rótulo para o campo.

{% highlight java %}
public class SPackagePassaporte extends SPackage {

    public static final String PACOTE = "formulario.passaporte";
    public static final String TIPO = "SolicitacaoPassaporte";
    public static final String NOME_COMPLETO = PACOTE + "." + TIPO;    
    ...
    protected void carregarDefinicoes(PackageBuilder pb) {
        final STypeComposite<?> solicitacao = pb.createCompositeType(TIPO);
        solicitacao
            .asAtr().label("Solicitação de Passaporte");
            
        solicitacao.
            .addFieldString("nomeCompleto")
            .asAtr().label("Nome Completo")
    }
...
{% endhighlight %}

O código a seguir exemplifica a adição dos campos de CPF e data de nascimento. Para esses dois casos o Singular Form conta com tipos pré-definidos com validações e máscaras. Para dispor esses dois campos lado a lado basta indicar a proporcionalidade desejada dentro do *grid system* do [twitter bootstrap](http://getbootstrap.com/css/#grid) que conta com 12 colunas virtuais de largura.

{% highlight java %}
public class SPackagePassaporte extends SPackage {

    public static final String PACOTE = "formulario.passaporte";
    public static final String TIPO = "SolicitacaoPassaporte";
    public static final String NOME_COMPLETO = PACOTE + "." + TIPO;    
    ...
    protected void carregarDefinicoes(PackageBuilder pb) {
        final STypeComposite<?> solicitacao = pb.createCompositeType(TIPO);
        solicitacao
            .asAtr().label("Solicitação de Passaporte");
            
        solicitacao.
            .addFieldString("nomeCompleto")
            .asAtr().label("Nome Completo")
        
        solicitacao
                .addField("CPF", STypeCPF.class)
                .asAtr().label("CPF")
                .asAtrBootstrap().colPreference(4);

        solicitacao
                .addFieldDate("dataNacimento")
                .asAtr().label("Data de Nascimento")
                .asAtrBootstrap().colPreference(4);
    }
...
{% endhighlight %}

No exemplo acima os campos ocupam preferencialmente 4/12 avos da largura da tela (nesse caso 25%) quando a tela tiver tamanho adequado. Se a tela for menor o Singular Form ajusta automáticamente a proporção para uma melhor disposição dos campos.

A figura abaixo ilustra o formulário apresentado pela definição acima:

![Formulário de Solicitação de Passaporte](/img/form-solicitacao-passaporte.png)


### Configuração Wicket-Spring
Para fazer o setup inicial do Singular Form é preciso declarar uma factory para um *bean* (um bean singleton DI JSR-330) instância de `br.net.mirante.singular.form.spring.SpringFormConfig`
O exemplo abaixo apresenta uma factory utilizando os beans default do Singular Server em um contexto Spring.

{% highlight java %}
@Configuration
public class SingularServerFormConfigFactory {

    @Inject
    private SingularServerDocumentFactory singularServerDocumentFactory;

    @Inject
    private SingularServerSpringTypeLoader serverSpringTypeLoader;

    @Bean
    public SpringFormConfig formConfigWithDatabase() {
        SpringFormConfig formConfigWithDatabase = new SpringFormConfig();
        formConfigWithDatabase.setTypeLoader(serverSpringTypeLoader);
        formConfigWithDatabase.setDocumentFactory(singularServerDocumentFactory);
        return formConfigWithDatabase;
    }
}
{% endhighlight %}

Por fim, para apresentar o formulário é preciso criar uma página Wicket subclasse de `br.net.mirante.singular.util.wicket.template.SingularTemplate`. Essa página já trará a configuração automática dos recursos estáticos necessários para o Singular Form.
`br.net.mirante.singular.form.wicket.panel.SingularFormPanel`. Por exemplo:

{% highlight java %}
public class FormularioPassaporte extends SingularTemplate {

    @Inject
    @Named("formConfigWithDatabase")
    private SFormConfig singularFormConfig;

    public FormularioPassaporte(){
        add(new SingularFormPanel("singular-panel", singularFormConfig) {

            protected SInstance createInstance(SFormConfig singularFormConfig) {
                RefType refType = singularFormConfig
                        .getTypeLoader()
                        .loadRefTypeOrException(SPackagePassaporte.NOME_COMPLETO);

                return singularFormConfig
                        .getDocumentFactory()
                        .createInstance(refType);
            }
        });
    }
}
{% endhighlight %}

Código **HTML** da pagina Wicket:

{% highlight html %}
<html>
    <wicket:extend>
        <div wicket:id="singular-panel"></div>
    </wicket:extend>
</html>
{% endhighlight %}

Acesse a página onde o painel foi adicionado para visualizar o formulário criado.

<div class="note info">
  <h5>Dica para o Desenvolvedor</h5>
  <p>
    Existe um módulo dedicado para documentar o uso do <strong>Singula Form</strong>. Este módulo
    usa a <strong>API Wicket</strong> e um banco <strong>HSQL</strong>. Para mais detalhes
    carregue o módulo <code>ui-showcase</code>.
  </p>
</div>

A documentação dos principais conceitos do Singular Form está disponível na seção [Singular Form][SingularForm].

[MAVEN]: /docs/maven/
[SingularForm]: /docs/form/
