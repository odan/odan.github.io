---
title: Visual Studio Code Setup for ASP.NET Core
layout: post
comments: true
published: true
description:
keywords: asp dotnet csharp vscode
image: 
---

## Introduction

As a developer, you know that with the right tools and settings, you can significantly increase your productivity and make your coding experience more enjoyable. In this article, we will explore the essential [Visual Studio Code](https://code.visualstudio.com/) extensions that every C# and ASP.NET Core developer should have in their toolkit. These extensions will provide powerful features for debugging and testing with code coverage.

## Requirements

First, you need to have VS Code installed on your computer. 
If you don't have it installed yet, don't worry! 

It's a free and lightweight code editor that is good for C# and ASP.NET Core development. 

Go to the official [Visual Studio Code website](https://code.visualstudio.com/Download), download the installer for your operating system and follow the installation instructions.

ASP.NET is an open-source framework for building modern web applications and APIs with C#. To build .NET apps you also have to install the .NET SDK (Software Development Kit) on your machine. 

You can install it by visiting the official [.NET](https://dotnet.microsoft.com/en-us/download) website, downloading the  installer, and following the installation instructions specific to your operating system.

Once you have installed VSCode and ASP.NET, you can proceed with setting up the extensions.

## Extensions

Launch VSCode and click on the "Extensions" icon in the left sidebar.

### Essential Extensions

These are the most common extensions I would recommend:

* The [C# extension](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp) extension provides C# language support, code snippets, debugging, and more.

* Install the [.NET Core Tools](https://marketplace.visualstudio.com/items?itemName=formulahendry.dotnet) extension to enhance your .NET Core development experience by providing commands for running, testing, and publishing your ASP.NET Core applications.

### Extensions for Testing

To enable testing your ASP.NET Core applications, I would like to share the following useful extension with you:

[Test Explorer UI](https://marketplace.visualstudio.com/items?itemName=hbenl.vscode-test-explorer): Allows you to discover and run your unit tests from within Visual Studio Code.

[.NET Core Test Explorer](https://marketplace.visualstudio.com/items?itemName=formulahendry.dotnet-test-explorer): This extension integrates with the Test Explorer UI extension and provides test discovery and execution capabilities specifically for .NET Core projects.

[Test Adapter Converter](https://marketplace.visualstudio.com/items?itemName=ms-vscode.test-adapter-converter): Converts from the Test Explorer UI API into native VS Code testing.

As a useful helper, the [Test Explorer Status Bar](https://marketplace.visualstudio.com/items?itemName=connorshea.vscode-test-explorer-status-bar)
adds some some test statistics into the status bar.

The result should look and work pretty awesome:

<img src="https://github.com/odan/slim4-skeleton/assets/781074/bc296a9d-59b4-44ed-aa49-e0ea73ea6eee" loading="lazy" alt="Test Explorer UI">

**Code Coverage**

[Coverage Gutters](https://marketplace.visualstudio.com/items?itemName=ryanluker.vscode-coverage-gutters): Helps you visualize code coverage in your project by displaying colored gutters in the editor, indicating which lines are covered by tests.

To make Coverage Gutters work, you should configure the "Test Explorer UI"
key `dotnet-test-explorer.testArguments` to `/p:CollectCoverage=true /p:CoverletOutputFormat=lcov /p:CoverletOutput=./lcov.info`.

Example: `.vscode/settings.json`

```json
{
  "dotnet-test-explorer.testProjectPath": "**/*Tests.csproj",
  "dotnet-test-explorer.testArguments": "/p:CollectCoverage=true /p:CoverletOutputFormat=lcov /p:CoverletOutput=./lcov.info",
  "coverage-gutters.showLineCoverage": true,
  "coverage-gutters.showRulerCoverage": true,
  "coverage-gutters.showGutterCoverage": true,
}
```

Coverage Gutters in action:

<img src="https://github.com/odan/slim4-skeleton/assets/781074/664ba642-bacc-4f18-9049-1ee608051000" loading="lazy" alt="Coverage Gutters">


#### Useful Extensions

Besides the the essential extensions, I would like to recommend other
very useful extensions, that will give you a even better developer experience.

[Thunder Client](https://marketplace.visualstudio.com/items?itemName=rangav.vscode-thunder-client) is a lightweight REST API client that allows you to make HTTP requests and view responses within VS Code.

[Auto-Using for C#](https://marketplace.visualstudio.com/items?itemName=Fudge.auto-using) automatically adds missing using statements for C# code, saving you time and reducing manual effort.

[C# Namespace Autocompletion](https://marketplace.visualstudio.com/items?itemName=adrianwilczynski.namespace) provides intelligent autocompletion for C# namespaces, making it easier and faster to import the namespaces you need.

[Studio Icons](https://marketplace.visualstudio.com/items?itemName=jtlowe.vscode-icon-theme): Official icons from the Visual Studio Image Library. Optimized to work well for dark, light, and high contrast themes.

Once you have installed all these extensions, you'll have a comprehensive development environment for C# and ASP.NET Core in Visual Studio Code. Make sure to enable any necessary settings or configurations for specific extensions, and explore their documentation for more details on how to use their features effectively.

## Read more

* <https://www.hanselman.com/blog/automatic-unit-testing-in-net-core-plus-code-coverage-in-visual-studio-code>

