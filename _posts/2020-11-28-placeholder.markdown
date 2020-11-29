---
layout: post
title: Placeholder
date: 2020-11-28 20:33:38 -0600
categories: test
---

This article is a placeholder.

**English** / [日本語](https://github.com/ChenCMD/datapack-helper-plus-JP/blob/master/README.md) / [简体中文](https://github.com/SPGoding/vscode-datapack-helper-plus/blob/master/README-zh_Hans.md)

Data-pack Helper Plus can provide many heavy language features for documents in your datapack, including advancements, dimensions, dimension types, functions, loot tables, predicates, recipes, all kinds of tags, and worldgen settings.

If you like this extension, please consider [sponsoring](https://github.com/sponsors/SPGoding) me. You can also report bugs, suggest features, and help translations! See [CONTRIBUTING.md](https://github.com/SPGoding/datapack-language-server/blob/master/CONTRIBUTING.md) for more information.

- [Features](#features)
   - [Workspace Support](#workspace-support)

# Features

## Workspace Support

Please use the root folder of your datapack (where the `data` folder and the `pack.mcmeta` file are) as a root folder of your workspace, so that DHP can provide you with the best functionalities.

Moreover, DHP fully supports VSCode's [multi-root workspace feature](https://code.visualstudio.com/docs/editor/multi-root-workspaces). Every root which contains a `data` folder and `pack.mcmeta` file will be considered as a datapack and will be used for computing completions. Other root folders will not be affected by DHP.

You can access any content of any root as long as they are in the same workspace. The order of the roots in your workspace will affect the priority of these datapacks in DHP. The root at the beginning will be loaded at first, and the root at the end will be loaded at last, which means that the **earlier** the root is, the **lower** priority in DHP it has. This is exactly how Minecraft loads datapacks and decide which one overrides another one if a file has the same namespaced ID and is in the same category. For example, if your multi-root workspace looks like this:

```
─── (Root) Datapack A
   ├── data
   |   └── spgoding
   |       └── functions
   |           └── foo.mcfunction
   └── pack.mcmeta
─── (Root) Datapack B
   ├── data
   |   └── spgoding
   |       └── functions
   |           └── foo.mcfunction
   └── pack.mcmeta
```

And then you use `F2` in a mcfunction file to renamed the mcfunction `spgoding:foo` to `wtf:foo`, only the file in Datapack B (`Datapack B/data/spgoding/functions/foo.mcfunction`) will be moved to `Datapack B/data/wtf/functions/foo.mcfunction`, even if there's a function with the same namespaced ID in Datapack A (`Datapack A/data/spgoding/functions/foo.mcfunction`).

If you try to execute these commands in Minecraft, you can also noticed that the function in Datapack B is executed.
```mcfunction
datapack enable "file/Datapack A" first
datapack enable "file/Datapack B" last
function spgoding:foo
```

By acting like this, DHP ensures that the order it handling datapacks is consistent with Minecraft.

**Note**: you can drag and put the root folders in VSCode to sort them, and DHP will update the priority of them in DHP accordingly, which is really handy.
