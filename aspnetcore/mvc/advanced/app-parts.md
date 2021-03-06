---
title: Partes do aplicativo no ASP.NET Core
author: ardalis
description: "Saiba como usar as partes do aplicativo, que são abstrações dos recursos de um aplicativo, para configurar seu aplicativo para descobrir ou evitar o carregamento de recursos de um assembly."
manager: wpickett
ms.author: riande
ms.date: 01/04/2017
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: mvc/extensibility/app-parts
ms.openlocfilehash: 6b855f8725dacc89a7e0607224ef3c19ab9f5676
ms.sourcegitcommit: a510f38930abc84c4b302029d019a34dfe76823b
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/30/2018
---
# <a name="application-parts-in-aspnet-core"></a>Partes do aplicativo no ASP.NET Core

[Exibir ou baixar código de exemplo](https://github.com/aspnet/Docs/tree/master/aspnetcore/mvc/advanced/app-parts/sample) ([como baixar](xref:tutorials/index#how-to-download-a-sample))

Uma *Parte do aplicativo* é uma abstração dos recursos de um aplicativo, da qual recursos de MVC como controladores, componentes de exibição ou auxiliares de marca podem ser descobertos. Um exemplo de uma parte do aplicativo é um AssemblyPart, que encapsula uma referência de assembly e expõe tipos e referências de compilação. *Provedores de recursos* funcionam com as partes do aplicativo para preencher os recursos de um aplicativo do ASP.NET Core MVC. O caso de uso principal das partes do aplicativo é permitir que você configure seu aplicativo para descobrir (ou evitar o carregamento) recursos de MVC de um assembly.

## <a name="introducing-application-parts"></a>Introdução às partes do aplicativo

Aplicativos de MVC carregam seus recursos das [partes do aplicativo](/aspnet/core/api/microsoft.aspnetcore.mvc.applicationparts.applicationpart). Em particular, a classe [AssemblyPart](/aspnet/core/api/microsoft.aspnetcore.mvc.applicationparts.assemblypart#Microsoft_AspNetCore_Mvc_ApplicationParts_AssemblyPart) representa uma parte de aplicativo que é apoiada por um assembly. Você pode usar essas classes para descobrir e carregar recursos de MVC, como controladores, componentes de exibição, auxiliares de marca e fontes de compilação do Razor. O [ApplicationPartManager](/aspnet/core/api/microsoft.aspnetcore.mvc.applicationparts.applicationpartmanager) é responsável por controlar as partes do aplicativo e os provedores de recursos disponíveis para o aplicativo MVC. Você pode interagir com o `ApplicationPartManager` em `Startup` quando configura o MVC:

```csharp
// create an assembly part from a class's assembly
var assembly = typeof(Startup).GetTypeInfo().Assembly;
services.AddMvc()
    .AddApplicationPart(assembly);

// OR
var assembly = typeof(Startup).GetTypeInfo().Assembly;
var part = new AssemblyPart(assembly);
services.AddMvc()
    .ConfigureApplicationPartManager(apm => p.ApplicationParts.Add(part));
```

Por padrão, o MVC pesquisa a árvore de dependência e localiza controladores (mesmo em outros assemblies). Para carregar um assembly arbitrário (por exemplo, de um plug-in que não é referenciado em tempo de compilação), você pode usar uma parte do aplicativo.

Você pode usar as partes do aplicativo para *evitar* pesquisar controladores em um determinado assembly ou local. Você pode controlar quais partes (ou assemblies) ficam disponíveis para o aplicativo modificando a coleção `ApplicationParts` do `ApplicationPartManager`. A ordem das entradas na coleção `ApplicationParts` não é importante. É importante configurar totalmente o `ApplicationPartManager` antes de usá-lo para configurar serviços no contêiner. Por exemplo, você deve configurar totalmente o `ApplicationPartManager` antes de invocar `AddControllersAsServices`. Deixar de fazer isso significará que os controladores em partes do aplicativo adicionadas depois da chamada de método não serão afetados (não serão registrados como serviços), o que pode resultar em um comportamento incorreto de seu aplicativo.

Se tiver um assembly que contém controladores que você não deseja usar, remova-o do `ApplicationPartManager`:

```csharp
services.AddMvc()
    .ConfigureApplicationPartManager(p =>
    {
        var dependentLibrary = p.ApplicationParts
            .FirstOrDefault(part => part.Name == "DependentLibrary");

        if (dependentLibrary != null)
        {
           p.ApplicationParts.Remove(dependentLibrary);
        }
    })
```

Além do assembly de seu projeto e de seus assemblies dependentes, o `ApplicationPartManager` incluirá partes para `Microsoft.AspNetCore.Mvc.TagHelpers` e `Microsoft.AspNetCore.Mvc.Razor` por padrão.

## <a name="application-feature-providers"></a>Provedores de recursos do aplicativo

Os Provedores de recursos do aplicativo examinam as partes do aplicativo e fornece recursos para essas partes. Há provedores de recursos internos para os seguintes recursos de MVC:

* [Controladores](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.controllers.controllerfeatureprovider)
* [Referência de metadados](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.razor.compilation.metadatareferencefeatureprovider)
* [Auxiliares de marcação](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.razor.taghelpers.taghelperfeatureprovider)
* [Componentes da exibição](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.viewcomponents.viewcomponentfeatureprovider)

Provedores de recursos herdam de `IApplicationFeatureProvider<T>`, em que `T` é o tipo do recurso. Você pode implementar seus próprios provedores de recursos para qualquer um dos tipos de recurso do MVC listados acima. A ordem dos provedores de recursos na coleção `ApplicationPartManager.FeatureProviders` pode ser importante, pois provedores posteriores podem reagir às ações tomadas por provedores anteriores.

### <a name="sample-generic-controller-feature"></a>Exemplo: recurso de controlador genérico

Por padrão, o ASP.NET Core MVC ignora controladores genéricos (por exemplo, `SomeController<T>`). Este exemplo usa um provedor de recursos de controlador que é executado depois do provedor padrão e adiciona instâncias de controlador genérico a uma lista de tipos especificados (definidos em `EntityTypes.Types`):

[!code-csharp[Main](./app-parts/sample/AppPartsSample/GenericControllerFeatureProvider.cs?highlight=13&range=18-36)]

Os tipos de entidade:

[!code-csharp[Main](./app-parts/sample/AppPartsSample/Model/EntityTypes.cs?range=6-16)]

O provedor de recursos é adicionado em `Startup`:

```csharp
services.AddMvc()
    .ConfigureApplicationPartManager(p => 
        p.FeatureProviders.Add(new GenericControllerFeatureProvider()));
```

Por padrão, os nomes do controlador genérico usados para roteamento seriam no formato *GenericController'1 [Widget]* em vez de *Widget*. O atributo a seguir é usado para modificar o nome para que ele corresponda ao tipo genérico usado pelo controlador:

[!code-csharp[Main](./app-parts/sample/AppPartsSample/GenericControllerNameConvention.cs)]

A classe `GenericController`:

[!code-csharp[Main](./app-parts/sample/AppPartsSample/GenericController.cs?highlight=5-6)]

O resultado, quando uma rota correspondente é solicitada:

![O exemplo de saída do aplicativo de exemplo lê, "Hello from a generic Sproket controller".](app-parts/_static/generic-controller.png)

### <a name="sample-display-available-features"></a>Exemplo: exibir recursos disponíveis

Você pode iterar nos recursos preenchidos disponíveis para seu aplicativo solicitando um `ApplicationPartManager` por meio da [injeção de dependência](../../fundamentals/dependency-injection.md) e usando-o para preencher as instâncias dos recursos apropriados:

[!code-csharp[Main](./app-parts/sample/AppPartsSample/Controllers/FeaturesController.cs?highlight=16,25-27)]

Saída de exemplo:

![Saída de exemplo do aplicativo de exemplo](app-parts/_static/available-features.png)
