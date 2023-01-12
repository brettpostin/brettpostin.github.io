---
layout: post
title:  "TypeScript Support in Visual Studio 2013"
date:   2013-12-12 20:01:17 +0000
categories: Tooling
tags: typescript visual-studio
---

There seems to be quite a lot of confusion surrounding the TypeScript support in Visual Studio 2013, and I believe that is partly down to how the tooling was previously implemented in WebEssentials.

As an early adopter of TypeScript in Visual Studio 2012 using Web Essentials, the development experience was pretty self explanatory.

1. Add a new TypeScript file
2. Edit (with nice preview window) and Save
3. WebEssentials would generate a `.js` and `.min.js` file and automatically add them to the project

When moving to Visual Studio 2013 I was looking for the same functionality from the integrated plugin but found little success. However after much frustration I eventually figured out the differences between the tooling, and I now happen to think the plugin is actually better than the WebEssentials implementation (excluding the lack of preview window!).

In order to get to that stage I first had to do some things to my existing project that had been developed prior to TypeScript 0.8.2.

**Step 1:**

Install the latest version of TypeScript (0.9.5)

**Step 2:**

Unload the project file, edit the .csproj, and add the following:

```xml
<PropertyGroup Condition="'$(Configuration)' == 'Debug'">
    <TypeScriptTarget>ES5</TypeScriptTarget>
    <TypeScriptIncludeComments>true</TypeScriptIncludeComments>
    <TypeScriptSourceMap>true</TypeScriptSourceMap>
</PropertyGroup>
<PropertyGroup Condition="'$(Configuration)' == 'Release'">
    <TypeScriptTarget>ES5</TypeScriptTarget>
    <TypeScriptIncludeComments>false</TypeScriptIncludeComments>
    <TypeScriptSourceMap>false</TypeScriptSourceMap>
</PropertyGroup>
<Import Project="$(MSBuildExtensionsPath32)\Microsoft\VisualStudio\v$(VisualStudioVersion)\TypeScript\Microsoft.TypeScript.targets" />
```

**Step 3:**

Change the build action on all of my existing TypeScript files to `TypeScriptCompile`:

_Properties -> Build Action -> TypeScriptCompile_

_Note: This is set automatically when adding new TypeScript files._

The above changes set the TypeScript files to be compiled on **build**. However this is not ideal during development. The Compile-On-Save feature in the plugin aids the development experience by generating the files on disk upon save.

**However**, this feature differs from WebEssentials in that it does **NOT** add the generated files to the project.

Once I realised that these files were in fact redundant in the project (and in source control), it became clear why the original WebEssentials workflow was the root of my confusion. I was then able to go back and delete all of the previously generated .js and .min.js files in the project.

I don’t think these differences have been communicated particularly well from the TypeScript team and there seems to be a lack of documentation surrounding the Compile-On-Save features. It would also be great to have the editor preview window back!

Links:

[TypeScriptLang.org](http://www.typescriptlang.org/)

[TypeScript for Visual Studio 2012 and 2013](https://www.microsoft.com/en-us/download/details.aspx?id=34790)

[Compile-on-Save](http://typescript.codeplex.com/wikipage?title=Compile-on-Save)