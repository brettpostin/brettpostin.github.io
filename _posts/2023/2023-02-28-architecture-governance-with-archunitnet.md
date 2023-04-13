---
title:  "Architecture Governance with ArchUnitNET"
date:   2023-02-28 21:01:17 +0000
categories: Development
---

Maintaining a clean, well-structured, and consistent architecture is crucial for the long-term success and maintainability of any software project. However, as a project grows and evolves, it becomes increasingly difficult to enforce architectural rules and best practices. 

Over the years I have casually explored ways to automate the monitoring and enforcement of architectural designs. Having revisited it again recently, I came across a great library called ArchUnitNET. 

Firstly let's look a little deeper at the problem...

# Challenges in governance

1. Complex systems can be difficult to understand. There may be many components and relationships in the system that need to be understood and maintained. Good documentation and training can help of course. However developers have a lot to contend with! Processes, procedures, deadlines etc. Expecting developers to read, process, and recollect that knowledge on a daily basis is optimistic.

2. New hires and junior developers may need time to get up to speed. You want developers to be productive as quickly as possible, whilst minimising risk that your architectural designs will be violated.

3. Time pressures and deadlines can result in shortcuts or compromises being considered that challenge your designs. 

4. Tooling can work against you! Visual Studio and R# refactorings can make it trivial to introduce mistakes through the introduction of undesirable references and other suggestions. 

# ArchUnitNET

ArchUnitNET is an open-source library that provides a simple, fluent API for defining and validating architectural rules in .NET using unit tests. 

Inspired by the Java library ArchUnit, ArchUnitNET helps developers enforce architectural constraints and best practices by automatically checking the codebase for violations. 

This ensures that your project adheres to the desired architectural principles and makes it easier to maintain and scale the code over time.

# Getting Started

## Installation

To get started with ArchUnitNET, you'll need to add the ArchUnitNET NuGet package to your test project:

```powershell
Install-Package ArchUnitNET
```

It is also recommended to install the extension package for your specific unit test framework. There are extensions for xUnit, NUnit and MSTestV2.

```powershell
Install-Package ArchUnitNET.xUnit
Install-Package ArchUnitNET.NUnit
Install-Package ArchUnitNET.MSTestV2
```

## Architecture Definition

Once you've installed the package, you can start writing unit tests to enforce architectural rules.

The first step is to define your architecture by loading all relevant assemblies using the `ArchLoader()` class. This builds up the metadata about your types within ArchUnitNET. 

```csharp
private static readonly Architecture Architecture =
    new ArchLoader().LoadAssemblies(
        typeof(ExampleClass).Assembly, 
        typeof(ForbiddenClass).Assembly
    ).Build();
```

It should only be done once to optimise performance.

## Test Cases

With your architecture parsed, you can write test cases using ArchUnitNET's Fluent API. Here are some examples:

_Assembly dependencies_
```csharp
Types().That().ResideInAssembly("A").Should()
.OnlyDependOn(Types().That().ResideInAssembly("B"));
```
Govern project references to ensure unwanted project dependencies do not get introduced. Useful if you organise your architectural layers using project boundaries.

_Namespace dependencies_
```csharp
Types().That().ResideInNamespace("A").Should()
.OnlyDependOn(Types().That().ResideInAssembly("B"));
```
Similar to project references but at a more granular level. Useful if you organise your architectural layers within a single project.

_Inheritance checks_
```csharp
Types().That().AreAssignableTo(typeof(Controller)).Should()
.ResideInNamespace("Controllers");
```
Check various architectural rules around inherited types.

_Naming conventions_
```csharp
Types().That().ResideInNamespace("Controllers").Should()
.HaveNameEndingWith("Controller");
```
Enforce code conventions to maintain consistency throughout your codebase.

These examples are by no means exhaustive. The ArchUnitNET Fluent API is fairly extensive, allowing for checks against many possible scenarios.

# Evaluation

## Pros

- By integrating ArchUnitNET into your development workflow and CI/CD pipeline, you can ensure that your team consistently adheres to the defined architectural rules. 

- It raise awareness of design violations and prevent unwanted changes or mistakes from creeping into your codebase.

- Simple to get up and running.

- Free.

- As it runs within unit tests, there is no impact on build or IDE performance, unlike some alternatives.

- Another unintentional benefit I discovered was package governance. By defining what dependencies are allowed in your projects, your ArchUnitNET test cases can raise awareness of new third-party dependencies being introduced. This can then prompt a discussion with the team if necessary. 

## Cons

- Test time developer feedback is not as "real-time" as some alternatives.

- Test run durations may be impacted. Though I have found this to be insignificant.

- Official documentation could be better. Though the Fluent API is fairly self explanatory.

## Some Alternatives

#### Custom Roslyn Analyser(s)

Custom Roslyn analysers offer a flexible and powerful means of providing real-time developer feedback within the IDE. However it is non-trivial to get up and running. This could be a barrier to getting developer buy-in on your automated governance. 

It is also possible to write poorly performing analysers, which could impact IDE performance. So you need to be careful with your implementation, especially with large solutions.

#### NsDepCop

This is a cool NuGet package to define and enforce namespace dependencies. It's free and simple to get going, and offers compile time developer feedback. 

However it is limited in the architectural rules you can enforce. 

# Summary 

In summary, enforcing software architecture design can be challenging due to the complexity of software systems, changes over time, communication and collaboration issues, limited resources, and lack of automation. 

However, tools such as ArchUnitNET can help to address some of these challenges by automating the process of enforcing the architecture and providing a common language and understanding of the system.
