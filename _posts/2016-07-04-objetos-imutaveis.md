---
layout: post
title: "Objetos Imutáveis"
date: 2016-07-04
description: O conceito de Imutabilidade nos faz ter disciplina e simplicidade para codificar.
summary: O conceito de Imutabilidade nos faz ter disciplina e simplicidade para codificar.
image: /images/photo-1448357019934-caa4696bb949.jpg
categories: 
  - OO
tags:
  - imutabilidade
keywords:
  - immutable
  - imutavel
  - imutabilidade
--- 

Objetos Imutáveis são seguros, são *thread-safe*, simples de entender, construir e testar. Evitam acoplamento temporal, previnem a referencia nil/NULL e não precisam utilizar "cópia defensiva".

<!--more-->

![Imagem]({{ page.image }})

##Introdução {#introducao}

Um [Objeto Imutável](https://en.wikipedia.org/wiki/Immutable_object) é uma instância que, após inicializada através do 
[construtor]({% post_url 2016-03-21-construtores-da-classe-primario-secundarios %}) 
de sua Classe, nunca terá seu Estado alterado para o mundo exterior até o fim de sua vida.

Se você está lendo isso pela primeira vez talvez não consiga visualizar os benefícios das **restrições** que a Imutabilidade traz ao Desenvolvimento de Software.

Sim, restrições, e isso é ótimo.

<blockquote>
  Objetos Imutáveis são livres de efeitos colaterais externos. Eles são criados representando um momento específico no tempo e devem permanecer inalterados até a sua morte.
  <footer><cite title="eBook OPP">eBook — @ObjectPascalProgramming</cite></footer>
</blockquote>

A Imutabilidade irá lhe ajudar a **codificar melhor**, muito melhor, devido as restrições impostas.

Você terá que pensar cuidadosamente em suas Classes, porque elas existem, qual o real trabalho delas... Seus Objetos serão imutáveis então você não poderá alterar o comportamento deles em *runtime* utilizando 
[*Setters*]({% post_url 2016-06-27-getters-e-setters %}). Consequentemente cada Objeto deverá ter uma **construção simples** e fazer apenas um único trabalho. Os argumentos desse trabalho só poderão ser informados no construtor e ninguém quer um construtor complexo com 10 parâmetros, certo?

Por causa da Imutabilidade você será obrigado a fazer as Classes com poucos métodos e poucos argumentos no construtor.

Imutabilidade nos faz ter **disciplina** para codificar.

##Vantagens da Imutabilidade {#vantagens-da-imutabilidade}

Restrições são ótimas para o código. Restrições, limites, condições... tudo isso traz ordem, clareza e entendimento.

Se você restringir, terá menos coisas para se preocupar, cuidar, modificar e testar.

Além da disciplina, vejamos algumas outras vantagens em utilizarmos a Imutabilidade.

###1-Segurança {#seguranca}

Objetos Imutáveis são seguros. Se o construtor de uma Classe retornar para o seu programa uma instância de um Objeto Imutável, significa que essa instância é segura para usar. Tudo o que deveria ter sido inicializado nessa instância que dependia do mundo externo, já foi verificado e satisfeito. Não é necessário utilizar nenhum outro `Setter`. Não há mais dependências externas. Tudo está pronto. Nada poderá ser quebrado.

###2-*Thread-safe* {#thread-safe}

Objetos Imutáveis são *thread-safe* por definição. Se seu estado não modifica, seu Objeto pode ser compartilhado por mais de uma *thread*, ao mesmo tempo, sem necessidade de sincronismos. Isso nos traz performance. Não há necessidade de *lock* nas threads. A informação irá fluir com segurança e rapidez.

###3-Simples de Entender, Construir e Testar {#entender-construir-testar}

Objetos Imutáveis só recebem seus argumentos no construtor da Classe. Não queremos 10 ou mais argumentos para iniciar um Objeto. No máximo deveríamos utilizar 5 argumentos — é a regra que tento seguir.

Se uma Classe não tem modificadores externos e os construtores tem apenas 5 ou menos argumentos, então o Objeto será:

  1. Simples de entender porque terá menos código para ler (toda a dependência externa estará apenas no construtor);
  2. Fácil de construir por possuir poucos argumentos (lembre-se: 5 no máximo);
  3. Fácil de testar porque o único ponto de variação do comportamento são os argumentos (trate-os em Testes de Unidade e terá coberto todas as possibilidades de comportamento que o Objeto poderá ter).

###4-Evitam Acoplamento Temporal {#acoplamento-temporal}

Acoplamento Temporal é quando métodos ou funções só podem ser executadas numa sequência pré-determinada. 

Isso é programação procedural. 

Vejamos um exemplo:

    begin
      Query := TQuery.New(Database);
      Query.SQL.Text := 
        'SELECT id, name FROM Customer WHERE id = :id';
      Query.Params.ParamByName('id').Value := 10;
      Query.Open;
      ShowMessage(Query.FieldByName('name').AsString);
    end;

No exemplo acima o Objeto Query é criado. Temos que chamar seus métodos ou propriedades numa sequência correta ou uma `Exception` será gerada. Esse é um "código padrão" que encontramos na maioria dos sistemas, certo?

Esse código não é Orientado a Objetos, é Procedural. Apesar de haver Objetos envolvidos, eles não estão conversando entre si. Há um "controlador" (você) que informa passo-a-passo o que fazer. 

  1. A *query* deve ser informada;
  2. Os parâmetros devem ser informados;
  3. Executa a *query*;
  4. Obtém o resultado;
  
Como seria o mesmo exemplo utilizando um código Orientado a Objetos, declarativo, com Objetos conversando entre si?

Existem várias opções. Vou propor uma delas como exemplo:

    begin
      ShowMessage(
        TQuery.New(
          Database, 
          TStatement.New(
            'SELECT id, name FROM Customer WHERE id = :id', 
            TParam.New(ftInteger, 10)
          )
        )
        .Open
        .Fields('name').AsString
      );
    end;

Não há um controlador. Não há procedimentos um após o outro. O que existe é uma combinação de Objetos que trabalham entre si para gerar um resultado.

  1. Só é possível criar um `TStatement` se passar um SQL e seus parâmetros;
  2. Só é possível criar um `TQuery` se passar uma instância de *Database* e `TStatement`; 
  3. O método `Open` pode ser executado a qualquer momento e ele gera um outro Objeto, por exemplo, um `TDataSet` (não pense no mesmo `TDataSet` que já existe na VCL/LCL);
  4. Um `TDataSet` tem uma lista `Fields` que, pelo nome, retorna um `Field` que é exibido na forma de `string`.

Não há acoplamento temporal.

Quando você executa o método `Open` tudo já foi configurado antes.

###5-Previne a Referencia nil/NULL {#previne-referencia-null}

Esse é um item óbvio quando utilizamos Objetos Imutáveis.

Não existe instâncias [nil/NULL]({% post_url 2016-04-11-nao-utilize-nil-ou-null %}).

Se você não tem métodos de alteração dos atributos e os argumentos necessários para inicializar um Objeto Imutável são passados apenas no construtor da Classe, basta testar a existência de nil/NULL no construtor. Feito.

###6-Não precisam usar "Cópia Defensiva" {#copia-defensiva}

Esse é um problema que muitos programadores Object Pascal não dão muita importância. Ele é mais conhecido no Java. Mas vou lhe dar um exemplo em Object Pascal:

    type
      TQuery = class(TInterfacedObject, IQuery)
      private
        FDatabase: TDatabase;
        FSQL: TStrings;
        FParams: TParams;
      public
        constructor Create(Database: TDatabase);
        class function New(Database: TDatabase): IQuery;
        function SQL: TStrings;
        function Params: TParams;
        function Open: IQuery;
      end;

    begin
      Query := TQuery.New(Database);
      Query.SQL.Text := 
        'SELECT id, name FROM Customer WHERE id = :id';
      Query.Params.ParamByName('id').Value := 10;
      Query.Open;
      ShowMessage(Query.FieldByName('name').AsString);
    end;

A Classe `TQuery` tem 2 defeitos:

  1. Acoplamento Temporal. Se `Open` for executado antes da inicilização de `SQL` e `Params`, haverá problemas;
  2. Os atributos `FSQL` e `FParams`, apesar de serem privados e não terem nenhum método `Setter` para atualizá-los, ainda assim seus valores podem ser atualizados de fora do Objeto (chamadas a `SQL.Text` e `Params.ParamByName`).
  
Isso acontece porque `SQL` e `Params` são Classes com métodos Mutáveis.

Para que não aconteça isso você deveria criar cópias defensivas desses mesmos Objetos e retorná-los nos métodos ao invés de utilizar a referência dos atributos privados.

Mas teríamos outro problema, *memory leak*. O programador não saberia quando destruir um Objeto porque ele não saberia se é uma cópia ou a referência do atributo privado do Objeto. Java não tem esse problema porque tem o *Garbage Collector*.

O que devemos fazer, em Object Pascal, é só utilizar tipos Imutáveis ou retornar tipos de [Interfaces]({% post_url 2016-01-18-interfaces-em-todo-lugar %})
que não tenham métodos/propriedades que permitam alterar seu estado interno. Interfaces com contagem de referência seriam destruídas automaticamete pelo compilador.

##Desvantagens da Imutabilidade {#desvantagens-da-imutabilidade}

Na engenharia tudo tem um preço. A Orientação a Objetos tem a vantagem do encapsulamento e polimorfismo, mas dizem que tem um custo alto no uso da memória do computador.

Temos que escolher as ferramentas corretas dependendo do trabalho a ser feito, pois não dá para termos só vantagens utilizando uma única ferramenta.

Então vejamos algumas desvantagens em utilizarmos a Imutabilidade.

###1-Impacto na Performance {#impacto-na-performance}

Imutabilidade exige que criemos uma nova instância se alguma modificação for necessária no estado do Objeto. 
Não podemos alterar o estado, então o Objeto deve criar uma cópia de si mesmo, com sutis diferenças.

Isso pode impactar na performance, mas há técnicas para minimizar esse custo.

O framework [Immutable-js](), um framework do Facebook que utiliza o conceito da Imutabilidade, faz uso de estruturas compartilhas para minimizar a criação de novas instâncias.

###2-Mudança de Pensamento {#mudanca-de-pensamento}

Com Objetos Imutáveis você não poderá pensar em execuções linha-a-linha. Tem-se que usar uma programação mais declarativa. Essa pode ser uma transição difícil de fazer.

Nas [Linguagens Funcionais](https://en.wikipedia.org/wiki/Functional_programming) como Clojure, Haskell ou F# é natural utilizar Imutabilidade, pois esse é o *default* nessas linguagens. 
O código é declarativo e funcional, não procedural. No código funcional, o valor de uma função de saída depende somente dos argumentos que são passados para a função. E funções podem retornar funções e recebê-las também. É um estilo completamente diferente do código procedural.

Linguagens imperativas como Java, C/C++ ou Object Pascal não tem estruturas imutáveis por padrão. Precisa ser simulado. Precisamos pensar, deliberadamente, em tornar algo imutável. Cada retorno de método, cada Objeto ou argumento. Essa é uma desvantagem, mas que diminui com o tempo e prática.

###3-GUI, *Widgets*, etc

Temos Objetos que representam *widgets* na tela do usuário. Fazer esses Objetos Imutáveis seria complicado. É possível, mas não é eficaz — teríamos que mudar a forma de pensar e escrever GUI.

Tais Objetos como `TEdit`, `TMemo`, etc são melhores construídos sendo Mutáveis.

Utilizando a forma que temos hoje para criar Objetos na tela, se tívessemos que criar um novo Objeto sempre que o usuário alterasse o `Text` ou `Caption` de algum *widget*, a performance cairia drásticamente além de haver problemas com a GUI.

Então, para construir GUI, Objetos Mutáveis ainda são melhores nessa área.

##Conclusão {#conclusao}

Imutabilidade me faz ter disciplina e simplicidade.

Apresentei a teoria e os motivos — ou alguns deles — para você começar a utilizar Objetos Imutáveis.

Na teoria pode parecer fácil mas na prática é muito mais difícil do que parece. Exige quase uma lobotomia. Você deve extrair o **pensamento procedural**, que está muito ligado a mutabilidade, e começar a utilizar o **pensamento funcional** onde tudo é imutável.

Vejo a Orientação a Objetos como um caminho do meio. Um caminho mais equilibrado, mais simples, entre os paradigmas procedural e funcional.

Utilizando a Orientação a Objetos podemos utilizar o que há de melhor entre esses dois paradigmas.

Faça sua escolha.

Até logo.

