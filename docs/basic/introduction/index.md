---
layout: single
title: "Introdução"
permalink: /docs/basic/introduction
sidebar:
  nav: "docs"
---

Esta é a documentação dos módulos **Java** do **Projeto Singular**. A seguir serão sumarizados cada um dos
sub-módulos do projeto.

## Singular Flow

Módulo responsável pela criação, configuração e gerenciamento dos fluxos de processos do sistema. Com este módulo o
desenvolvedor será capaz de projetar e implementar os processos do sistema, assim como operá-los e
analisá-los através da extração de indicadores em tempo-real.

### Flow UI-Admin

Módulo de monitoramento (BAM) de processos. Este é um sub-módulo do **Singular Flow** que pode ser embutido em outra
aplicação.

Este módulo contém um _dashboard_ com diversos indicadores por processos da aplicação. Também é capaz de listar e
detalhar os processos e suas instâncias.

#### Dicas Importantes

Este módulo é usado como um _overlay_ **MAVEN** de outros sistemas, mas também pode ser
implantado de forma _standalone_.

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

## Singular Form

Este é o módulo responsável pela geração automática de formulários com base em metadados fornecidos pelo desenvolvedor.

### Form Showcase

Implante este módulo para verficar o funcionamento do **Singular Form** e aprender a usá-lo. Neste _showcase_ há
diversos exemplos que mostram as opções disponibilizadas pela **API**.
