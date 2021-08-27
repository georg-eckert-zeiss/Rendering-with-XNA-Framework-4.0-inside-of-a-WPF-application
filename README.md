# Overview

This is a repository preserving example code by [Nick Gravelyn](https://www.nickgravelyn.com/) did some blog post on XNA in the Microsoft Docs. As long as it's there, the original post can be found in the [Microsoft Docs Blog Archive](https://docs.microsoft.com/en-us/archive/blogs/nicgrave/rendering-with-xna-framework-4-0-inside-of-a-wpf-application). Originally there was no license included. By mail and in [this issue at MerjTek's WPF integration](https://github.com/MerjTek/MerjTek.WpfIntegration/issues/1#issuecomment-902622749) he kindly answered the code may be considered MIT or Microsoft Public License, so I put it here under MIT.

This repository allows for using the code without legal concerns which may occur with the missing license in the blog post.


# Example Code

There was more example code included in the blogpost, which is preserved here.


## Example 1

```csharp
 // What importers or processors should we load?
private const string xnaVersion = 
    ", Version=4.0.0.0, PublicKeyToken=6d5c3888ef60e27d";
private static readonly string[] pipelineAssemblies =
{
    "Microsoft.Xna.Framework.Content.Pipeline.AudioImporters" + xnaVersion,
    "Microsoft.Xna.Framework.Content.Pipeline.EffectImporter" + xnaVersion,
    "Microsoft.Xna.Framework.Content.Pipeline.FBXImporter" + xnaVersion,
    "Microsoft.Xna.Framework.Content.Pipeline.TextureImporter" + xnaVersion,
    "Microsoft.Xna.Framework.Content.Pipeline.VideoImporters" + xnaVersion,
    "Microsoft.Xna.Framework.Content.Pipeline.XImporter" + xnaVersion,
};
```


## Example 2

```csharp
using Microsoft.Build.Evaluation;
```


## Example 3

```csharp
// MSBuild objects used to dynamically build content.
private ProjectCollection projectCollection;
private Project msBuildProject;
```

## Example 4

```csharp
/// <summary>
/// Creates a temporary MSBuild content project in memory.
/// </summary>
void CreateBuildProject()
{
    string projectPath = 
        Path.Combine(buildDirectory, "content.contentproj");
    string outputPath = Path.Combine(buildDirectory, "bin");

    // Create the project collection
    projectCollection = new ProjectCollection();

    // Hook up our custom error logger.
    errorLogger = new ErrorLogger();
    projectCollection.RegisterLogger(errorLogger);

    // Create the build project.
    msBuildProject = new Project(projectCollection);
    msBuildProject.FullPath = projectPath;

    // set up the properties we care about
    msBuildProject.SetProperty("XnaPlatform", "Windows");
    msBuildProject.SetProperty("XnaFrameworkVersion", "v4.0");
    msBuildProject.SetProperty("XnaProfile", "HiDef");
    msBuildProject.SetProperty("Configuration", "Release");
    msBuildProject.SetProperty("OutputPath", outputPath);

    // Register any custom importers or processors.
    foreach (string pipelineAssembly in pipelineAssemblies)
    {
        msBuildProject.AddItem("Reference", pipelineAssembly);
    }

    // Include the standard targets file that defines
    // how to build XNA Framework content.
    msBuildProject.Xml.AddImport(
        "$(MSBuildExtensionsPath)\\Microsoft\\XNA Game Studio\\v4.0\\" +
        "Microsoft.Xna.GameStudio.ContentPipeline.targets");
}
```


## Example 5

```csharp
/// <summary>
/// Adds a new content file to the MSBuild project. The importer and
/// processor are optional: if you leave the importer null, it will
/// be autodetected based on the file extension, and if you leave the
/// processor null, data will be passed through without any processing.
/// </summary>
public void Add(
    string filename, string name, 
    string importer, string processor)
{
    // set up the metadata for this item
    var metadata = new SortedList<string,string>();
    metadata.Add("Link", Path.GetFileName(filename));
    metadata.Add("Name", name);

    if (!string.IsNullOrEmpty(importer))
        metadata.Add("Importer", importer);

    if (!string.IsNullOrEmpty(processor))
        metadata.Add("Processor", processor);

    // add the item
    msBuildProject.AddItem("Compile", filename, metadata);
}
```


## Example 6

```csharp
/// <summary>
/// Removes all content files from the MSBuild project.
/// </summary>
public void Clear()
{
    // select all compiled objects in the project and remove them
    var compileObjects = from i in project.Items 
                         where i.ItemType == "Compile" 
                         select i;
    project.RemoveItems(compileObjects);
}
```