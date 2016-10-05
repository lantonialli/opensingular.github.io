---
layout: single
title: "Singular Form"
permalink: /docs/form
sidebar:
  nav: "docs"
---

A criação de formulários no Form são realizados por meio da definição dos dados para que posteriormente eles sejam
apresentados como um formulário para preenchimento do usuário. Para realizar a configuração o desenvolvedor necessita
criar um Pacote que contém a definição dos dados que precisam ser preenchidos. Cada dado deve ter um tipo com seus
respectivos atributos.

## Estrutura de Tipos

Todo tipo definido no Singular Form é subclasse de `SType`. As principais implementações de SType são o `STypeSimple`, `STypeComposite`, 
`STypeCode` e `STypeList`. Por meio desses tipos é possível compor diferentes estruturas de informações e modos de exibição destas.

## Tipos
O tipos do Singular Form ampliam os tipos básicos do Java adicionando capacidades de serialização em linguagens de definição de dados como XML e JSON além de permitir a adição de atributos que definem como será a representação e comportamento no formulário gerado.

### Tipo Simples

O tipo simples é representado pelas classes que herdam de `STypeSimple` e são utilizados para definir dados com referência direta aos tipos primitivos. Por exemplo: o tipo `STypeInteger` define um campo numérico. No quadro a baixo é possível observar como é definido um campo numérico com o nome `idade`.

{% highlight java %} 
public class Pessoa extends SPackage {

    @Override
    protected void carregarDefinicoes(PackageBuilder pb) {
        final STypeComposite pessoa = pb.createCompositeType("pessoa");
        pessoa.addField("idade", STypeInteger.class);
    }

}
{% endhighlight %}

##### Exemplos de tipos simples

<div class="mobile-side-scroller">
<table>
    <thead>
    <tr>
        <th>Nome</th>
        <th>Descrição</th>
    </tr>
    </thead>
    <tbody>
    <tr>
        <td><p><code>STypeDecimal</code></p></td>
        <td>Número decimal</td>
    </tr>
    <tr>
        <td><p><code>STypeInteger</code></p></td>
        <td>Número inteiro</td>
    </tr>
    <tr>
        <td><p><code>STypeDateHour</code></p></td>
        <td>Data com a respectiva hora</td>
    </tr>
    <tr>
        <td><p><code>STypeString</code></p></td>
        <td>Texto</td>
    </tr>
    <tr>
        <td><p><code>STypeYearMonth</code></p></td>
        <td>Data composta apenas por mês e ano</td>
    </tr>
    <tr>
        <td><p><code>STypeBoolean</code></p></td>
        <td>Valor booleano</td>
    </tr>
    <tr>
        <td><p><code>STypeDate</code></p></td>
        <td>Data composta por dia, mês e ano</td>
    </tr>
    </tbody>
</table>
</div>


### Tipo Composto

O tipo composto é utilizado para definir estruturas de dados complexas. Por meio dos tipos compostos é possível fazer uma composição de outros tipos como apresentado no quadro abaixo onde são adicionados os campos `nome` e "idade" ao tipo composto `pessoa`.

{% highlight java %} 
public class Pessoa extends SPackage {

    @Override
    protected void carregarDefinicoes(PackageBuilder pb) {
        final STypeComposite pessoa = pb.createCompositeType("pessoa");
        pessoa.addField("nome", STypeString.class);
        pessoa.addField("idade", STypeInteger.class);
    }

}
{% endhighlight %}

### Tipo Lista

O tipo lista é utilizado para representar lista de outros tipos como demonstrado no quadro abaixo onde é adicionado o campo `emails` ao tipo composto `pessoa`.

{% highlight java %} 
public class Pessoa extends SPackage {

    @Override
    protected void carregarDefinicoes(PackageBuilder pb) {
        final STypeComposite pessoa = pb.createCompositeType("pessoa");
        pessoa.addFieldListOf("emails", STypeEMail.class);
    }

}
{% endhighlight %}

## Atributos

Os atributos enriquecem os tipos permitindo controlar o comportamento dos componentes gerados. Com o uso de atributos é possível definir qual será o rótulo (*label*) que acompanhara um campo de texto ou definir quando um campo de data será apresentado.

No exemplo a seguir, é adicionado um campo do tipo String de nome `descricao` e é adicionado o rótulo `Descrição`.

{% highlight java %} 
public class Pessoa extends SPackage {

    @Override
    protected void carregarDefinicoes(PackageBuilder pb) {
        final STypeComposite pessoa    = pb
                .createCompositeType("pessoa");
        final SType          descricao = pessoa
                .addField("descricao", STypeString.class);

        descricao.asAtr().label("Descrição");
    }

}
{% endhighlight %}

## Views 

A visualização define como será disposto o componente no formulário para cada tipo adicionado. Todos os tipos simples possuem uma forma de visualização padrão, porém é possível configurar uma visualização diferente de acordo a necessidade. No quadro abaixo é configurado o tipo de visualização `SViewListByTable` para o campo do tipo lista `emails`  que resulta em uma tabela de emails com a opção de adicionar elementos dinamicamente.

{% highlight java %} 
public class Pessoa extends SPackage {

    @Override
    protected void carregarDefinicoes(PackageBuilder pb) {
        final STypeComposite usuario = pb
                .createCompositeType("usuario");
        final STypeList      emails  = usuario
                .addFieldListOf("emails", STypeEMail.class);
        emails.withView(SViewListByTable::new);
    }

}
{% endhighlight %}


### Alguns exemplos de visualizações

<div class="mobile-side-scroller">
<table>
    <thead>
    <tr>
        <th>Nome</th>
        <th>Descrição</th>
    </tr>
    </thead>
    <tbody>
    <tr>
        <td><p><code>SViewSelectionByRadio</code></p></td>
        <td>Seleção através de radio buttons.</td>
    </tr>
    <tr>
        <td><p><code>SViewSelectionBySelect</code></p></td>
        <td>Seleção através de select</td>
    </tr>
    <tr>
        <td><p><code>SViewAutoComplete</code></p></td>
        <td>Seleção através de componente que filtra resultados de acordo com o texto digitado</td>
    </tr>
    </tbody>
</table>
</div>


## Pacotes

Os pacotes do Singular Form são estruturas agrupadoras de tipos e atributos. Por meio de pacotes é possível agrupar tipos relacionados de forma a facilitar seu reuso. A tabela abaixo relaciona os principais pacotes do singular.

<div class="mobile-side-scroller">
<table>
    <thead>
    <tr>
        <th>Nome</th>
        <th>Descrição</th>
    </tr>
    </thead>
    <tbody>
    <tr>
        <td><p><code>SPackageCore</code></p></td>
        <td>Contem os principais tipos do form, como STypeSimple e STypeString</td>
    </tr>
    <tr>
        <td><p><code>SPackageBasic</code></p></td>
        <td>Contem os principais atributos do form, como controle de visibilidade e de obrigatoriedade</td>
    </tr>
    <tr>
        <td><p><code>SPackageBootstrap</code></p></td>
        <td>Contem atributos que configuram a apresentação do form utilizando a biblioteca de estilos bootstrap.</td>
    </tr>
    </tbody>
</table>
</div>

