---
layout: single
title: "Wicket"
permalink: /docs/wicket
sidebar:
  nav: "docs"
---

O núcleo do Singular Form não é vinculado a nenhuma tecnologia de apresentação específica, mas para a geração da interface 
de usuário, é provida uma integração padrão com o framework [Apache Wicket](http://wicket.apache.org){:target="_blank"}.

<div class="note info">
    Antes de ler o restante desta seção, você deve conhecer o sistema de tipos do Singular. Caso não conheção,
    <a href="/docs/form/#sistema-de-tipos">visite esta página</a>.
</div>

## Configuração necessária

Para configurar o form-wicket no seu projeto, além de configurar a dependência no pom.xml 
([veja como aqui](/docs/maven/#singular-form)) é necessário realizar a configuração do `SingularFormConfigWicketImpl`
passando para ele um ServiceRegistry que saberá como fazer lookup de serviços específicos da sua aplicação.

Em um projeto utilizando spring a configuração no application.xml seria parecida com a seguinte:

{% highlight xml %}
<bean id="singularFormConfig" class="br.net.mirante.singular.form.wicket.SingularFormConfigWicketImpl">
    <property name="serviceRegistry" ref="springServiceRegistry"/>
</bean>
{% endhighlight %}


## Como funciona

A construção da interface se dá de forma recursiva pelo método `UIBuilderWicket.build()`, que define qual o **mapper** 
(instância de `IWicketComponentMapper`) que deve ser utilizado para uma determinada instância (do tipo `SInstance`) e o executa 
para a criação dos componentes de interface correspondentes.

Cada passo da construção recebe o **objeto de contexto** (instância de `WicketBuildContext`), que contém as informações necessárias 
para o mapper, como o **container** (componente Wicket do tipo `BSContainer`) que representa a `<div>` na qual o elemento atual 
estará contido e a **instância** (guardada em um objeto `IModel<SInstance>` do Wicket) correspondente para este mapper (de acordo com 
o `SType` da instância). O mapper então cria, dentro do container, o conjunto de componentes correspondente à instância. Por exemplo, 
uma instância do tipo `SIString` resultaria em um `TextField`, acompanhado de sua label e painel de feedback de erros de validação.

Tipos compostos, como `STypeComposite` e `STypeList`, possuem campos de tipos arbitrários. Seus mappers cuidam da construção do 
layout dos componentes correspondentes, mas a construção dos mesmos, em geral, é delegada novamente ao objeto `UIBuilderWicket`, 
que possui a lógica de resolução de mappers. Ele pode ser acessado de dentro de um mapper a partir do objeto de contexto 
(`WicketBuildContext.getUiBuilderWicket()`).


## Estendendo o framework

### Mappers customizados

É possível estender o conjunto de componentes de interface, ou fazer customizações avançadas aos componentes já existentes, 
utilizando mappers customizados.

Caso seja necessário um mapper customizado para algum dos tipos definidos no seu pacote, basta utilizar o método 
`withCustomMapper` da classe `SType`, passando como parâmetro um `ISupplier` que cria o seu mapper customizado.

O layout utilizado nesta integração se baseia no [Bootstrap](http://getbootstrap.com/css/#grid). São fornecidos componentes que 
auxiliam na construção da telas utilizando somente Java (em geral, as classes com o prefixo `BS*`) com um alto nível de controle, mas 
é também possível utilizar os mecanismos do framework Wicket de resolução de templates HTML para a construção de componentes 
totalmente customizados.

#### Exemplo: mapper para `<video>`

- Definição de um campo "video" do tipo String, com uma validação simples sobre a validade da URL, e a especificação de 
`VideoMapper` como um mapper customizado.

{% highlight java %}
    STypeComposite<SIComposite> tipoMyForm = pb.createCompositeType("testForm");
    tipoMyForm.addFieldString("video")
        .addInstanceValidator(v -> {
            try {
                new URL(v.getInstance().getValue());
            } catch (Exception e) {
                v.error("URL inválida");
            }
        })
        //@destacar
        .withCustomMapper(() -> new VideoMapper())
        .asAtr().label("Vídeo").required(true);
{% endhighlight %}

- Implementação de `VideoMapper`, utilizando a API Java para a construção da estrutura HTML. Note que são tratados os dois modos
de visualização, EDIT e READ_ONLY de formas diferentes.

{% highlight java %}
    class VideoMapper implements IWicketComponentMapper {
        @Override
        public void buildView(WicketBuildContext ctx) {
            final IMappingModel<String> labelModel = IMappingModel.of(ctx.getModel()).map(it -> it.asAtr().getLabel());
            switch (ctx.getViewMode()) {
                case EDIT:
                    ctx.getContainer()
                        .appendComponent(id -> new BSFormGroup(id, BSGridSize.MD)
                            .appendControls(12, controlsId -> new BSControls(controlsId)
                                .appendLabel(new Label("label", labelModel))
                                .appendInputText(ctx.configure(this, new TextField<>("url", ctx.getValueModel())))
                                .appendFeedback(ctx.createFeedbackCompactPanel("feedback"))
                                .appendHelpBlock($m.ofValue("Exemplos: "
                                    + "<ul>"
                                    + " <li>http://techslides.com/demos/sample-videos/small.mp4</li>"
                                    + " <li>http://www.sample-videos.com/video/mp4/720/big_buck_bunny_720p_1mb.mp4</li>"
                                    + "</ul>"), false)));
                    break;
                case READ_ONLY:
                    Video video = new Video("video", ctx.getModel().getObject().getValueWithDefault(String.class));
                    video.setWidth(320);
                    ctx.getContainer().appendTag("video", video);
                    break;
            }
        }
    }
{% endhighlight %}

- HTML gerado no modo EDIT (alguns atributos, como id, foram retirados por simplicidade)
{% highlight html %}
<div data-instance-path="testForm.video">
  <div class="can-have-error form-group">
    <div class=" col-md-12">
      <label>Vídeo</label>
      <input type="text" class="form-control" value="">
      <span class="help-block feedback"></span>
      <span class="help-block">Exemplos:
        <ul>
          <li>http://techslides.com/demos/sample-videos/small.mp4</li>
          <li>http://www.sample-videos.com/video/mp4/720/big_buck_bunny_720p_1mb.mp4</li>
        </ul>
      </span>
    </div>
  </div>
</div>
{% endhighlight %}

- HTML gerado no modo READ_ONLY (alguns atributos, como id, foram retirados por simplicidade)
{% highlight html %}
<div data-instance-path="testForm.video">
  <video src="http://techslides.com/demos/sample-videos/small.mp4" controls="controls" width="320"></video>
</div>
{% endhighlight %}


#### Exemplo: mapper para um componente de abas, recursivo

- Um mapper customizado de um container, que exemplifica a recursão: cada campo do `SIComposite` é constuído como conteúdo de uma aba.

{% highlight java %}
class CaseCustomTabbedPanelMapperPackage extends SPackage {

    @Override
    protected void onLoadPackage(PackageBuilder pb) {

        STypeComposite<SIComposite> form = pb.createCompositeType("testForm");

        STypeComposite<SIComposite> tab1 = form.addFieldComposite("tab1");
        tab1.asAtr().label("Aba 1");
        tab1.addFieldString("texto1").asAtr().label("Texto 1");
        tab1.addFieldString("texto2").asAtr().label("Texto 2");

        STypeComposite<SIComposite> tab2 = form.addFieldComposite("tab2");
        tab2.asAtr().label("Aba 2");
        tab2.addFieldBoolean("bool1").asAtr().label("Boolean 1");
        tab2.addFieldBoolean("bool2").asAtr().label("Boolean 2");

        STypeComposite<SIComposite> tab3 = form.addFieldComposite("tab3");
        tab3.asAtr().label("Aba 3");
        tab3.addFieldInteger("int1").asAtr().label("Inteiro 1");
        tab3.addFieldInteger("int2").asAtr().label("Inteiro 2");

        form.withCustomMapper(() -> new CustomTabMapper());
    }

    // Mapper recursivo
    static final class CustomTabMapper implements IWicketComponentMapper {

        @Override
        @SuppressWarnings("unchecked")
        public void buildView(WicketBuildContext ctx) {
            final IModel<? extends SInstance> model = ctx.getModel();
            final STypeComposite<SIComposite> stype = (STypeComposite<SIComposite>) model.getObject().getType();

            final BSContainer<?> container = ctx.getContainer();
            final List<CustomTab> tabs = new ArrayList<>();
            for (SType<?> field : stype.getFields()) {
                tabs.add(new CustomTab(ctx, new SInstanceFieldModel<>(model, field.getNameSimple())));
            }
            container.appendComponent(id -> new CustomAjaxTabbedPanel(id, tabs));
        }
    }

    // Implementação de Tab que cria o contexto e o container de cada filho, e chama o builder para construí-los.
    static final class CustomTab implements ITab {
        private final SInstanceFieldModel<SInstance> model;
        private final WicketBuildContext             parentCtx;
        public CustomTab(WicketBuildContext parentCtx, SInstanceFieldModel<SInstance> model) {
            this.parentCtx = parentCtx;
            this.model = model;
        }
        @Override
        public IModel<String> getTitle() {
            return IReadOnlyModel.of(() -> model.getObject().asAtr().getLabel());
        }
        
¢        @Override
        public WebMarkupContainer getPanel(String containerId) {
            // container filho
            final BSContainer<?> childContainer = new BSContainer<>(containerId);
            // contexto filho
            final WicketBuildContext childCtx = parentCtx.createChild(childContainer, true, model);

            // chamada recursiva ao UIBuilderWicket
            childCtx.getUiBuilderWicket().build(childCtx, childCtx.getViewMode());

            return childContainer;
        }
        @Override
        public boolean isVisible() {
            return true;
        }
    }

    // Utilizando AjaxTabbedPanel built-in no wicket-extensions, customizado para funcionar com o bootstrap. 
    static final class CustomAjaxTabbedPanel extends AjaxTabbedPanel<CustomTab> {
        private CustomAjaxTabbedPanel(String id, List<CustomTab> tabs) {
            super(id, tabs);
        }
        @Override
        protected String getSelectedTabCssClass() {
            return "active";
        }
        @Override
        protected String getTabContainerCssClass() {
            return "nav nav-tabs";
        }
    }
}
{% endhighlight %}
