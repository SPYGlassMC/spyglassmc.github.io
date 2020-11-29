---
layout: page
title: Usage
permalink: /usage/
---

# To Use the Data Pack Language Server

The Data Pack Language Server can provide language features like code completions and refactors for making Minecraft: Java Edition data packs.
It can be installed to any [text editors that support the LSP](https://microsoft.github.io/language-server-protocol/implementors/tools/).
We only provide instructions for several typical editors here. Please join the [discuss space][discuss] if you need any help.

## For Visual Studio Code

[Visual Studio Code][vs-code] is the primarily supported text editor by the SPYGlass development team. You just need to install the [VS Code extension][vs-code-ext]
to enjoy all the features provided by the Language Server.

## For Sublime Text 3

The process of setting up the Language Server for Sublime Text is quite tedious. We will look into improving it in the future.

1. Install [Node.js][node-js] if you haven't.
2. Execute `npm i -g @spgoding/datapack-language-server` in your command line to install the Language Server.
3. Install [Package Control](https://packagecontrol.io/installation) if you haven't.
4. Install [Arcensoth](https://github.com/Arcensoth)'s language-mcfunction package by following the [instructions](https://github.com/Arcensoth/language-mcfunction#installing-the-sublimetext-package) if you haven't.
5. Install [LSP](https://packagecontrol.io/packages/LSP) package.
6. Open the Command Palette and select `Preferences: LSP Settings`.
7. Configure LSP to add the Language Server. Here's one example:
```json
{
  "clients": {
    "datapack-language-server": {
      "command": [
        "datapack-language-server",
        "--stdio"
      ],
      "enabled": true,
      "languages": [
        {
          "languageId": "mcfunction",
          "scopes": [
            "source.mcfunction"
          ],
          "syntaxes": [
            "Packages/language-mcfunction/mcfunction.tmLanguage"
          ]
        },
        {
          "languageId": "json",
          "scopes": [
            "source.json"
          ],
          "syntaxes": [
            "Packages/JavaScript/JSON.sublime-syntax"
          ]
        }
      ]
    }
  },
  "only_show_lsp_completions": true
}
```
8. Open the Command Palette, select `LSP: Enable Language Server Globally`, and choose `datapack-language-server`.
9. Enjoy. Do note that you need to execute the command in step 2 manually if you want to update the Language Server.

[discuss]: /discuss
[node-js]: https://nodejs.org/
[vs-code]: https://code.visualstudio.com/
[vs-code-ext]: https://marketplace.visualstudio.com/items?itemName=SPGoding.datapack-language-server
