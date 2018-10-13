### ILMerge.Fody is an add-in for [Fody](https://github.com/Fody/Fody/) [![Build status](https://ci.appveyor.com/api/projects/status/3ntf6e6jp0bppo9q?svg=true)](https://ci.appveyor.com/project/tom-englert/ilmerge-fody) [![NuGet Status](http://img.shields.io/nuget/v/ILMerge.Fody.svg?style=flat-square)](https://www.nuget.org/packages/ILMerge.Fody)
![Icon](package_icon.png)

This add-in merges the referenced types of local dependencies as private types into the target assembly, and removes the references to the local dependencies.
This can help you to e.g. escape the DLL-hell.


### NuGet installation

Install the [ILMerge.Fody NuGet package](https://nuget.org/packages/ILMerge.Fody/) and update the [Fody NuGet package](https://nuget.org/packages/Fody/):

```
PM> Install-Package ILMerge.Fody
PM> Update-Package Fody
```

The `Update-Package Fody` is required since NuGet always defaults to the oldest, most buggy version of any dependency.


### Add to FodyWeavers.xml

Add `<ILMerge/>` to [FodyWeavers.xml](https://github.com/Fody/Fody#add-fodyweaversxml)

```xml
<?xml version="1.0" encoding="utf-8" ?>
<Weavers>
  <ILMerge />
</Weavers>
```


## How it works


### Merge IL code from referenced assemblies into the main assembly

This weaver was inspired by the [ILMerge Tool](https://www.microsoft.com/en-us/download/details.aspx?id=17630) by Mike Barnett from Microsoft.

Beside the simplified usage as a Fody add in, it was designed for a slightly different usage: 

- It does not merge complete assemblies, but only the types that are used. This will also save a lot of space if you use only some parts of a large library.
- It hides the merged types by making them private, avoiding conflicts with other modules using the same code.

This way you can make use of 3rd party libraries, without exposing them as references that you have to ship separately. 
You will also avoid the so called "dll hell", since all the code is now private in your assembly.


### Details

This Task performs the following changes

 * Take all assemblies that have been marked as "Copy Local" and merges the IL code into the target assembly.
 * Removes the references to the assemblies that have been merged.


## Configuration Options

All config options are accessed by modifying the `ILMerge` node in FodyWeavers.xml,
or adding the corresponding attribute to the assembly, 
where an attribute in the assembly has precedence of the setting in the FodyWeavers.xml

Default FodyWeavers.xml:

```xml
<Weavers>
  <ILMerge />
</Weavers>
```


### ExcludeAssemblies

A regular expression matching the assembly names to exclude from merging.

Do not include `.exe` or `.dll` in the names.

Can take two forms.

As an XML element:

```xml
<ILMerge>
  <ExcludeAssemblies>xunit|newtonsoft</ExcludeAssemblies>
</ILMerge>
```

Or as an XML attribute:

```xml
<ILMerge ExcludeAssemblies='xunit|newtonsoft' />
```

Or as an attribute in code:

```c#
[assembly: ILMerge.ExcludeAssemblies("xunit|newtonsoft")]
```

### IncludeAssemblies

A regular expression matching the assembly names to include in merging.

Do not include `.exe` or `.dll` in the names.

If combined with `ExcludeAssemblies`, it will only apply to items not already excluded.

Can take two forms. 

As an XML element:

```xml
<ILMerge>
  <IncludeAssemblies>My.*Module|My.*Task</IncludeAssemblies>
</ILMerge>
```

Or as an XML attribute:

```xml
<ILMerge IncludeAssemblies='My.*Module|My.*Task' />
```

Or as an attribute in code:

```c#
[assembly: ILMerge.IncludeAssemblies("My.*Module|My.*Task")]
```

