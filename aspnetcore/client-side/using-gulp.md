---
title: "Usando o Gulp no núcleo do ASP.NET"
author: rick-anderson
description: Saiba como usar o Gulp em ASP.NET Core.
manager: wpickett
ms.author: riande
ms.custom: H1Hack27Feb2017
ms.date: 02/28/2017
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: client-side/using-gulp
ms.openlocfilehash: 0a3443e8187d46992f55dc537d0f400c6771c50c
ms.sourcegitcommit: 7ac15eaae20b6d70e65f3650af050a7880115cbf
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/02/2018
---
# <a name="introduction-to-using-gulp-in-aspnet-core"></a>Introdução ao uso de Gulp no núcleo do ASP.NET 

Por [Erik Reitan](https://github.com/Erikre), [Scott Addie](https://scottaddie.com), [Daniel Roth](https://github.com/danroth27), e [Shayne Boyer](https://twitter.com/spboyer)

Em um aplicativo web moderna típico, o processo de compilação pode:

* Agrupar e minificada arquivos JavaScript e CSS.
* Execute ferramentas para chamar as tarefas de empacotamento e minimização antes de cada compilação.
* Compilar menos ou SASS arquivos CSS.
* Compile arquivos CoffeeScript ou TypeScript para JavaScript.

Um *executor de tarefas* é uma ferramenta que automatiza a essas tarefas de rotina de desenvolvimento e muito mais. O Visual Studio fornece suporte interno para dois executores de tarefas comuns de baseados em JavaScript: [Gulp](https://gulpjs.com/) e [Grunt](using-grunt.md).

## <a name="gulp"></a>gulp

Gulp é um baseados em JavaScript streaming compilação Kit de ferramentas do código do lado do cliente. Ele costuma ser usado para transmitir arquivos do lado do cliente por meio de uma série de processos quando um evento específico é acionado em um ambiente de compilação. Por exemplo, o Gulp pode ser usado para automatizar [empacotamento e minimização](bundling-and-minification.md) ou a limpeza de um ambiente de desenvolvimento antes de uma nova compilação.

Um conjunto de tarefas Gulp é definido em *gulpfile.js*. O JavaScript a seguir inclui módulos de Gulp e especifica os caminhos de arquivo a ser referenciado de tarefas disponível em breve:

```javascript
/// <binding Clean='clean' />
"use strict";

var gulp = require("gulp"),
  rimraf = require("rimraf"),
  concat = require("gulp-concat"),
  cssmin = require("gulp-cssmin"),
  uglify = require("gulp-uglify");

var paths = {
  webroot: "./wwwroot/"
};

paths.js = paths.webroot + "js/**/*.js";
paths.minJs = paths.webroot + "js/**/*.min.js";
paths.css = paths.webroot + "css/**/*.css";
paths.minCss = paths.webroot + "css/**/*.min.css";
paths.concatJsDest = paths.webroot + "js/site.min.js";
paths.concatCssDest = paths.webroot + "css/site.min.css";
```

O código acima Especifica quais módulos de nó são necessários. O `require` função importa cada módulo, para que as tarefas dependentes podem utilizar seus recursos. Cada um dos módulos importados é atribuída a uma variável. Os módulos podem estar localizados por nome ou caminho. Neste exemplo, os módulos denominado `gulp`, `rimraf`, `gulp-concat`, `gulp-cssmin`, e `gulp-uglify` são recuperados por nome. Além disso, uma série de caminhos são criadas para que os locais de arquivos CSS e JavaScript podem ser reutilizados e referência de tarefas. A tabela a seguir fornece descrições dos módulos incluídos no *gulpfile.js*.

| Nome do Módulo | Descrição |
| ----------- | ----------- |
| gulp        | O sistema de compilação streaming Gulp. Para obter mais informações, consulte [gulp](https://www.npmjs.com/package/gulp). |
| rimraf      | Um módulo de exclusão do nó. Para obter mais informações, consulte [rimraf](https://www.npmjs.com/package/rimraf). |
| gulp-concat | Um módulo que concatena arquivos com base em caractere de nova linha do sistema operacional. Para obter mais informações, consulte [gulp concat](https://www.npmjs.com/package/gulp-concat). |
| gulp-cssmin | Um módulo que minimiza arquivos CSS. Para obter mais informações, consulte [gulp cssmin](https://www.npmjs.com/package/gulp-cssmin). |
| uglify gulp | Um módulo que minimiza *. js* arquivos. Para obter mais informações, consulte [uglify gulp](https://www.npmjs.com/package/gulp-uglify). |

Depois que os módulos necessários são importados, as tarefas podem ser especificadas. Aqui, há seis tarefas registrado, representado pelo código a seguir:

```javascript
gulp.task("clean:js", function (cb) {
  rimraf(paths.concatJsDest, cb);
});

gulp.task("clean:css", function (cb) {
  rimraf(paths.concatCssDest, cb);
});

gulp.task("clean", ["clean:js", "clean:css"]);

gulp.task("min:js", function () {
  return gulp.src([paths.js, "!" + paths.minJs], { base: "." })
    .pipe(concat(paths.concatJsDest))
    .pipe(uglify())
    .pipe(gulp.dest("."));
});

gulp.task("min:css", function () {
  return gulp.src([paths.css, "!" + paths.minCss])
    .pipe(concat(paths.concatCssDest))
    .pipe(cssmin())
    .pipe(gulp.dest("."));
});

gulp.task("min", ["min:js", "min:css"]);
```

A tabela a seguir fornece uma explicação das tarefas especificado no código acima:

|Nome da Tarefa|Descrição|
|--- |--- |
|clean:js|Uma tarefa que usa o módulo de exclusão do nó rimraf para remover a versão minimizada do arquivo site.js.|
|Limpar: css|Uma tarefa que usa o módulo de exclusão do nó rimraf para remover a versão minimizada do arquivo site.css.|
|Limpar|Uma tarefa que chama o `clean:js` tarefa, seguida de `clean:css` tarefa.|
|min:js|Uma tarefa que minimiza e concatena todos os arquivos. js na pasta js. A. min.js arquivos serão excluídos.|
|min:CSS|Uma tarefa que minimiza e concatena todos os arquivos. CSS dentro da pasta de css. A. min.css arquivos serão excluídos.|
|min|Uma tarefa que chama o `min:js` tarefa, seguida de `min:css` tarefa.|

## <a name="running-default-tasks"></a>Execução de tarefas padrão

Se você ainda não criou um novo aplicativo Web, crie um novo projeto de aplicativo Web ASP.NET no Visual Studio.

1.  Adicione um novo arquivo JavaScript ao seu projeto e denomine- *gulpfile.js*, em seguida, copie o código a seguir.

    ```javascript
    /// <binding Clean='clean' />
    "use strict";
    
    var gulp = require("gulp"),
      rimraf = require("rimraf"),
      concat = require("gulp-concat"),
      cssmin = require("gulp-cssmin"),
      uglify = require("gulp-uglify");
    
    var paths = {
      webroot: "./wwwroot/"
    };
    
    paths.js = paths.webroot + "js/**/*.js";
    paths.minJs = paths.webroot + "js/**/*.min.js";
    paths.css = paths.webroot + "css/**/*.css";
    paths.minCss = paths.webroot + "css/**/*.min.css";
    paths.concatJsDest = paths.webroot + "js/site.min.js";
    paths.concatCssDest = paths.webroot + "css/site.min.css";
    
    gulp.task("clean:js", function (cb) {
      rimraf(paths.concatJsDest, cb);
    });
    
    gulp.task("clean:css", function (cb) {
      rimraf(paths.concatCssDest, cb);
    });
    
    gulp.task("clean", ["clean:js", "clean:css"]);
    
    gulp.task("min:js", function () {
      return gulp.src([paths.js, "!" + paths.minJs], { base: "." })
        .pipe(concat(paths.concatJsDest))
        .pipe(uglify())
        .pipe(gulp.dest("."));
    });
    
    gulp.task("min:css", function () {
      return gulp.src([paths.css, "!" + paths.minCss])
        .pipe(concat(paths.concatCssDest))
        .pipe(cssmin())
        .pipe(gulp.dest("."));
    });
    
    gulp.task("min", ["min:js", "min:css"]);
    ```

2.  Abra o *Package. JSON* arquivo (Adicionar se não há) e adicione o seguinte.

    ```json
    {
      "devDependencies": {
        "gulp": "3.9.1",
        "gulp-concat": "2.6.1",
        "gulp-cssmin": "0.1.7",
        "gulp-uglify": "2.0.1",
        "rimraf": "2.6.1"
      }
    }
    ```

3.  Em **Solution Explorer**, clique com botão direito *gulpfile.js*e selecione **Explorador do Executador de tarefas**.
    
    ![Abra o Explorador do Executador de tarefas no Gerenciador de soluções](using-gulp/_static/02-SolutionExplorer-TaskRunnerExplorer.png)
    
    **Explorador do Executador de tarefas** mostra a lista de tarefas de Gulp. (Talvez você precise clicar o **atualização** botão que aparece à esquerda do nome do projeto.)
    
    ![Explorador do Executador de tarefas](using-gulp/_static/03-TaskRunnerExplorer.png)

4.  Sob **tarefas** na **Explorador do Executador de tarefas**, clique com botão direito **limpa**e selecione **executar** no menu pop-up.

    ![Tarefa de limpeza de Explorador do Executador de tarefas](using-gulp/_static/04-TaskRunner-clean.png)

    **Explorador do Executador de tarefas** criará uma nova guia chamada **limpa** e execute a tarefa de limpeza, conforme definido em *gulpfile.js*.

5.  Com o botão direito do **limpa** de tarefas e selecione **associações** > **antes de criar**.

    ![Associação BeforeBuild Explorador do Executador de tarefas](using-gulp/_static/05-TaskRunner-BeforeBuild.png)

    O **antes de criar** associação configura a tarefa de limpeza para executar automaticamente antes de cada build do projeto.

As associações que você configurar o com **Explorador do Executador de tarefas** são armazenadas na forma de um comentário na parte superior da sua *gulpfile.js* e entrarão em vigor somente no Visual Studio. É uma alternativa que não requer o Visual Studio configurar a execução automática de tarefas de vez em seu *. csproj* arquivo. Por exemplo, colocar isso em seu *. csproj* arquivo:

```xml
<Target Name="MyPreCompileTarget" BeforeTargets="Build">
  <Exec Command="gulp clean" />
</Target>
```

Agora que a tarefa de limpeza é executada quando você executar o projeto no Visual Studio ou de um prompt de comando usando o [dotnet executar](/dotnet/core/tools/dotnet-run) comando (execute `npm install` primeiro).

## <a name="defining-and-running-a-new-task"></a>Definir e executar uma nova tarefa

Para definir uma nova tarefa Gulp, modificar *gulpfile.js*.

1.  Adicione o seguinte JavaScript ao final da *gulpfile.js*:

    ```javascript
    gulp.task("first", function () {
      console.log('first task! <-----');
    });
    ```

    Essa tarefa é denominada `first`, e simplesmente exibe uma cadeia de caracteres.

2.  Salvar *gulpfile.js*.

3.  Em **Solution Explorer**, clique com botão direito *gulpfile.js*e selecione *Explorador do Executador de tarefas*.

4.  Em **Explorador do Executador de tarefas**, clique com botão direito **primeiro**e selecione **executar**.

    ![Execute a primeira tarefa Explorador do Executador de tarefas](using-gulp/_static/06-TaskRunner-First.png)

    Você verá que o texto de saída é exibido. Se você estiver interessado nos exemplos com base em um cenário comum, consulte Gulp receitas.

## <a name="defining-and-running-tasks-in-a-series"></a>Definindo e tarefas em execução em uma série

Quando você executa várias tarefas, as tarefas executadas simultaneamente por padrão. No entanto, se você precisar executar tarefas em uma ordem específica, você deve especificar quando cada tarefa é concluída, bem como quais tarefas dependentes na conclusão de outra tarefa.

1.  Para definir uma série de tarefas a serem executadas em ordem, substitua o `first` tarefas que você adicionou acima na *gulpfile.js* com o seguinte:

    ```javascript
    gulp.task("series:first", function () {
      console.log('first task! <-----');
    });
 
    gulp.task("series:second", ["series:first"], function () {
      console.log('second task! <-----');
    });
 
    gulp.task("series", ["series:first", "series:second"], function () {});
    ```
 
    Agora você tem três tarefas: `series:first`, `series:second`, e `series`. O `series:second` tarefa inclui um segundo parâmetro que especifica uma matriz de tarefas a ser executado e concluído antes do `series:second` tarefa será executada. Conforme especificado no código acima, somente o `series:first` tarefa deve ser concluída antes do `series:second` tarefa será executada.

2.  Salvar *gulpfile.js*.

3.  Em **Solution Explorer**, clique com botão direito *gulpfile.js* e selecione **Explorador do Executador de tarefas** se ainda não estiver aberto.

4.  Em **Explorador do Executador de tarefas**, clique com botão direito **série** e selecione **executar**.

    ![Explorador do Executador de tarefas executar tarefa de série](using-gulp/_static/07-TaskRunner-Series.png)

## <a name="intellisense"></a>IntelliSense

O IntelliSense oferece conclusão de código, descrições de parâmetro e outros recursos para aumentar a produtividade e diminuir os erros. Gulp tarefas são escritas em JavaScript; Portanto, o IntelliSense pode fornecer assistência durante o desenvolvimento. Conforme você trabalha com JavaScript, IntelliSense listará os objetos, funções, propriedades e parâmetros que estão disponíveis com base em seu contexto atual. Selecione uma opção de codificação na lista pop-up fornecida pelo IntelliSense para concluir o código.

![gulp IntelliSense](using-gulp/_static/08-IntelliSense.png)

Para obter mais informações sobre o IntelliSense, consulte [JavaScript IntelliSense](https://docs.microsoft.com/visualstudio/ide/javascript-intellisense).

## <a name="development-staging-and-production-environments"></a>Ambientes de desenvolvimento, teste e produção

Quando o Gulp é usado para otimizar os arquivos do lado do cliente para produção e preparo, os arquivos processados são salvos em um local de preparo e produção. O *cshtml* arquivo usa o **ambiente** marca auxiliar para fornecer duas versões diferentes de arquivos CSS. É uma versão dos arquivos CSS para o desenvolvimento e a outra versão é otimizada para preparação e produção. No Visual Studio de 2017, quando você altera o **ASPNETCORE_ENVIRONMENT** variável de ambiente `Production`, Visual Studio criará o aplicativo Web e um link para os arquivos CSS minimizados. A marcação a seguir mostra o **ambiente** marca auxiliares que contém as marcações de link para o `Development` CSS arquivos e o minimizada `Staging, Production` arquivos CSS.

```html
<environment names="Development">
    <script src="~/lib/jquery/dist/jquery.js"></script>
    <script src="~/lib/bootstrap/dist/js/bootstrap.js"></script>
    <script src="~/js/site.js" asp-append-version="true"></script>
</environment>
<environment names="Staging,Production">
    <script src="https://ajax.aspnetcdn.com/ajax/jquery/jquery-2.2.0.min.js"
            asp-fallback-src="~/lib/jquery/dist/jquery.min.js"
            asp-fallback-test="window.jQuery"
            crossorigin="anonymous"
            integrity="sha384-K+ctZQ+LL8q6tP7I94W+qzQsfRV2a+AfHIi9k8z8l9ggpc8X+Ytst4yBo/hH+8Fk">
    </script>
    <script src="https://ajax.aspnetcdn.com/ajax/bootstrap/3.3.7/bootstrap.min.js"
            asp-fallback-src="~/lib/bootstrap/dist/js/bootstrap.min.js"
            asp-fallback-test="window.jQuery && window.jQuery.fn && window.jQuery.fn.modal"
            crossorigin="anonymous"
            integrity="sha384-Tc5IQib027qvyjSMfHjOMaLkfuWVxZxUPnCJA7l2mCWNIpG9mGCD8wGNIcPD7Txa">
    </script>
    <script src="~/js/site.min.js" asp-append-version="true"></script>
</environment>
```

## <a name="switching-between-environments"></a>Alternar entre ambientes

Para alternar entre a compilação para ambientes diferentes, modifique o **ASPNETCORE_ENVIRONMENT** valor da variável de ambiente.

1.  Em **Explorador do Executador de tarefas**, verifique o **min** tarefa foi definida para executar **antes de criar**.

2.  Em **Solution Explorer**, clique no nome do projeto e selecione **propriedades**.

    A folha de propriedades para o aplicativo Web é exibida.

3.  Clique na guia **Depurar**.

4.  Definir o valor de **: ambiente de hospedagem** variável de ambiente `Production`.

5.  Pressione **F5** para executar o aplicativo em um navegador.

6.  Na janela do navegador, a página e selecione **Exibir código-fonte** para exibir o HTML da página.

    Observe que os links de folha de estilos apontam para os arquivos CSS minimizados.

7.  Feche o navegador para interromper o aplicativo Web.

8.  No Visual Studio, retorne a folha de propriedades para o aplicativo Web e altere o **: ambiente de hospedagem** de volta para a variável de ambiente `Development`.

9.  Pressione **F5** para executar o aplicativo em um navegador novamente.

10. Na janela do navegador, a página e selecione **Exibir código-fonte** para ver o HTML da página.

    Observe que os links de folha de estilos apontam para as versões unminified dos arquivos CSS.

Para obter mais informações relacionadas a ambientes em ASP.NET Core, consulte [trabalhando com vários ambientes](../fundamentals/environments.md).

## <a name="task-and-module-details"></a>Detalhes da tarefa e o módulo

Uma tarefa Gulp está registrada com um nome de função. É possível especificar dependências se outras tarefas devem ser executadas antes da tarefa atual. Funções adicionais permitem que você execute e observar as tarefas de Gulp, bem como definir a origem (*src*) e de destino (*dest*) dos arquivos que está sendo modificados. Estas são as funções de API Gulp primárias:

|Função gulp|Sintaxe|Descrição|
|---   |--- |--- |
|tarefa  |`gulp.task(name[, deps], fn) { }`|O `task` função cria uma tarefa. O `name` parâmetro define o nome da tarefa. O `deps` parâmetro contém uma matriz de tarefas a serem concluídas antes de executa essa tarefa. O `fn` parâmetro representa uma função de retorno de chamada que executa as operações da tarefa.|
|Inspecionar |`gulp.watch(glob [, opts], tasks) { }`|O `watch` função monitora arquivos e executa tarefas quando ocorre uma alteração de arquivo. O `glob` parâmetro é um `string` ou `array` que determina quais arquivos assistir. O `opts` parâmetro fornece observando as opções de arquivo adicionais.|
|src   |`gulp.src(globs[, options]) { }`|O `src` função fornece os arquivos que correspondem os valores glob. O `glob` parâmetro é um `string` ou `array` que determina quais arquivos para leitura. O `options` parâmetro fornece outras opções de arquivo.|
|dest  |`gulp.dest(path[, options]) { }`|O `dest` função define um local para o qual os arquivos podem ser gravados. O `path` parâmetro é uma cadeia de caracteres ou uma função que determina a pasta de destino. O `options` parâmetro é um objeto que especifica as opções de pasta de saída.|

Para obter informações de referência de API Gulp, consulte [Gulp documentos API](https://github.com/gulpjs/gulp/blob/master/docs/API.md).

## <a name="gulp-recipes"></a>Gulp receitas

A comunidade de Gulp fornece Gulp [receitas](https://github.com/gulpjs/gulp/blob/master/docs/recipes/README.md). Essas receitas consistem em tarefas de Gulp para abordar cenários comuns.

## <a name="additional-resources"></a>Recursos adicionais

* [Documentação de gulp](https://github.com/gulpjs/gulp/blob/master/docs/README.md)
* [Empacotamento e minimização no núcleo do ASP.NET](bundling-and-minification.md)
* [Usando o assistente no núcleo do ASP.NET](using-grunt.md)
