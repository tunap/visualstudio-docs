---
title: "How to: Modify a Standard Menu Command in a Domain-Specific Language | Microsoft Docs"
ms.custom: ""
ms.date: "2018-06-30"
ms.prod: "visual-studio-tfs-dev14"
ms.reviewer: ""
ms.suite: ""
ms.tgt_pltfrm: ""
ms.topic: "article"
helpviewer_keywords: 
  - ".vsct files, adding commands to a domain-specific language"
  - "Domain-Specific Language, adding custom commands"
ms.assetid: 9b9d8314-d0d8-421a-acb9-d7e91e69825c
caps.latest.revision: 12
author: gewarren
ms.author: gewarren
manager: "douge"
---
# How to: Modify a Standard Menu Command in a Domain-Specific Language
[!INCLUDE[vs2017banner](../includes/vs2017banner.md)]

The latest version of this topic can be found at [How to: Modify a Standard Menu Command in a Domain-Specific Language](https://docs.microsoft.com/visualstudio/modeling/how-to-modify-a-standard-menu-command-in-a-domain-specific-language).  
  
You can modify the behavior of some of the standard commands that are defined automatically in your DSL. For example, you could modify **Cut** so that it excludes sensitive information. To do this, you override methods in a command set class. These classes are defined in the CommandSet.cs file, in the DslPackage project, and are derived from <xref:Microsoft.VisualStudio.Modeling.Shell.CommandSet>.  
  
 In summary, to modify a command:  
  
1.  [Discover what commands you can modify](#what).  
  
2.  [Create a partial declaration of the appropriate command set class](#extend).  
  
3.  [Override the ProcessOnStatus and ProcessOnMenu methods](#override) for the command.  
  
 This topic explains this procedure.  
  
> [!NOTE]
>  If you want to create your own menu commands, see [How to: Add a Command to the Shortcut Menu](../modeling/how-to-add-a-command-to-the-shortcut-menu.md).  
  
##  <a name="what"></a> What commands can you modify?  
  
#### To discover what commands you can modify  
  
1.  In the `DslPackage` project, open `GeneratedCode\CommandSet.cs`. This C# file can be found in Solution Explorer as a subsidiary of `CommandSet.tt`.  
  
2.  Find classes in this file whose names end with "`CommandSet`", for example `Language1CommandSet` and `Language1ClipboardCommandSet`.  
  
3.  In each command set class, type "`override`" followed by a space. IntelliSense will show a list of the methods that you can override. Each command has a pair of methods whose names begin "`ProcessOnStatus`" and "`ProcessOnMenu`".  
  
4.  Note which of the command set classes contains the command you want to modify.  
  
5.  Close the file without saving your edits.  
  
    > [!NOTE]
    >  Ordinarily, you should not edit files that have been generated. Any edits will be lost the next time that the files are generated.  
  
##  <a name="extend"></a> Extend the appropriate command set class  
 Create a new file that contains a partial declaration of the command set class.  
  
#### To extend the Command Set class  
  
1.  In Solution Explorer, in the DslPackage project, open the GeneratedCode folder and then look under CommandSet.tt and open its generated file CommandSet.cs. Note the namespace and the name of the first class that is defined there. For example, you might see:  
  
     `namespace Company.Language1`  
  
     `{ ...  internal partial class Language1CommandSet : ...`  
  
2.  In **DslPackage**, create a folder named **Custom Code**. In this folder, create a new class file named `CommandSet.cs`.  
  
3.  In the new file, write a partial declaration that has the same namespace and name as the generated partial class. For example:  
  
    ```  
    using System;  
    using System.Collections.Generic;  
    using System.ComponentModel.Design;  
    namespace Company.Language1 /* Make sure this is correct */  
    { internal partial class Language1CommandSet { ...  
    ```  
  
     **Note** If you used the class file template to create the new file, you must correct both the namespace and the class name.  
  
##  <a name="override"></a> Override the command methods  
 Most commands have two associated methods: The method with a name like `ProcessOnStatus`... determines whether the command should be visible and enabled. It is called whenever the user right-clicks the diagram, and should execute quickly and make no changes. `ProcessOnMenu`... is called when the user clicks the command, and should perform the function of the command. You might want to override either one or both of these methods.  
  
### To change when the command appears on a menu  
 Override the ProcessOnStatus... method. This method should set the Visible and Enabled properties of its parameter MenuCommand. Typically the command looks at this.CurrentSelection to determine whether the command applies to the selected elements, and might also look at their properties to determine whether the command can be applied in their current state.  
  
 As a general guide, the Visible property should be determined by what elements are selected. The Enabled property, which determines whether the command appears black or gray on the menu, should depend on the current state of the selection.  
  
 The following example disables the Delete menu item when the user has selected more than one shape.  
  
> [!NOTE]
>  This method does not affect whether the command is available through a keystroke. For example, disabling the Delete menu item does not prevent the command from being invoked through the Delete key.  
  
```  
/// <summary>  
/// Called when user right-clicks on the diagram or clicks the Edit menu.  
/// </summary>  
/// <param name="command">Set Visible and Enabled properties.</param>  
protected override void ProcessOnStatusDeleteCommand (MenuCommand command)  
{  
  // Default settings from the base method.  
  base.ProcessOnStatusDeleteCommand(command);  
  if (this.CurrentSelection.Count > 1)  
  {  
    // If user has selected more than one item, Delete is greyed out.  
    command.Enabled = false;  
  }  
}  
```  
  
 It is good practice to call the base method first, to deal with all the cases and settings with which you are not concerned.  
  
 The ProcessOnStatus method should not create, delete, or update elements in the Store.  
  
### To change the behavior of the command  
 Override the ProcessOnMenu... method. The following example prevents the user from deleting more than one element at a time, even by using the Delete key.  
  
```  
/// <summary>  
/// Called when user presses Delete key   
/// or clicks the Delete command on a menu.  
/// </summary>  
protected override void ProcessOnMenuDeleteCommand()  
{  
  // Allow users to delete only one thing at a time.  
  if (this.CurrentSelection.Count <= 1)  
  {  
    base.ProcessOnMenuDeleteCommand();  
  }  
}  
```  
  
 If your code makes changes to the Store, such as creating, deleting or updating elements or links, you must do so inside a transaction. For more information, see [How to Create and Update model elements](../modeling/how-to-modify-a-standard-menu-command-in-a-domain-specific-language.md).  
  
### Writing the code of the methods  
 The following fragments are frequently useful within these methods:  
  
-   `this.CurrentSelection`. The shape that the user right-clicked is always included in this list of shapes and connectors. If the user clicks on a blank part of the diagram, the Diagram is the only member of the list.  
  
-   `this.IsDiagramSelected()` - `true` if the user clicked a blank part of the diagram.  
  
-   `this.IsCurrentDiagramEmpty()`  
  
-   `this.IsSingleSelection()` - the user did not select multiple shapes  
  
-   `this.SingleSelection` - the shape or diagram that the user right-clicked  
  
-   `shape.ModelElement as MyLanguageElement` - the model element represented by a shape.  
  
 For more information about how to navigate from element to element and about how to create objects and links, see [Navigating and Updating a Model in Program Code](../modeling/navigating-and-updating-a-model-in-program-code.md).  
  
## See Also  
 <xref:System.ComponentModel.Design.MenuCommand>   
 [Writing Code to Customise a Domain-Specific Language](../modeling/writing-code-to-customise-a-domain-specific-language.md)   
 [How to: Add a Command to the Shortcut Menu](../modeling/how-to-add-a-command-to-the-shortcut-menu.md)   
 [Walkthrough: Getting Information from a Selected Link](../misc/walkthrough-getting-information-from-a-selected-link.md)   
 [How VSPackages Add User Interface Elements](../extensibility/internals/how-vspackages-add-user-interface-elements.md)   
 [Visual Studio Command Table (.Vsct) Files](../extensibility/internals/visual-studio-command-table-dot-vsct-files.md)   
 [VSCT XML Schema Reference](../extensibility/vsct-xml-schema-reference.md)   
 [VMSDK – Circuit Diagrams sample. Extensive DSL Customization](http://code.msdn.microsoft.com/Visualization-Modeling-SDK-763778e8)   
 [Sample code: Circuit Diagrams](http://code.msdn.microsoft.com/Visualization-Modeling-SDK-763778e8)



