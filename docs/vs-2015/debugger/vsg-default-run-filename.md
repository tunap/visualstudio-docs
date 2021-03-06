---
title: "VSG_DEFAULT_RUN_FILENAME | Microsoft Docs"
ms.custom: ""
ms.date: "2018-06-30"
ms.prod: "visual-studio-dev14"
ms.reviewer: ""
ms.suite: ""
ms.technology: 
  - "vs-ide-debug"
ms.tgt_pltfrm: ""
ms.topic: "article"
ms.assetid: ea549d2f-c857-458c-93c7-bc5a2d11d15d
caps.latest.revision: 7
author: "mikejo5000"
ms.author: "mikejo"
manager: "ghogen"
---
# VSG_DEFAULT_RUN_FILENAME
[!INCLUDE[vs2017banner](../includes/vs2017banner.md)]

The latest version of this topic can be found at [VSG_DEFAULT_RUN_FILENAME](https://docs.microsoft.com/visualstudio/debugger/graphics/vsg-default-run-filename).  
  
Defines the default file name of the graphics log file.  
  
## Syntax  
  
```cpp  
#define VSG_DEFAULT_FILENAME filename  
```  
  
#### Parameters  
 `filename`  
 The file name given by default to the graphics log file when graphics information is captured programmatically.  
  
## Value  
 A string literal that represents the file name of the graphics log file. By default, L"default.vsglog".  
  
```cpp  
#define VSG_DEFAULT_FILENAME L"default.vsglog"  
```  
  
## Remarks  
 If the preprocessor symbol `DONT_SAVE_VSGLOG_TO_TEMP` is defined, then the file name is relative to the current directory of the captured app, or is an absolute path; otherwise, it's relative to the user's temporary files directory and can't be an absolute path.  
  
 To change the defined file name, you must redefine it before you include `vsgcapture.h` in your program.  
  
## Example  
 This example shows how to change the capture file's default file name:  
  
```cpp  
// Redefine the default capture filename before including vsgcapture.h  
#define VSG_DEFAULT_FILENAME L"capture.vsglog"  
  
#include <vsgcapture.h>  
```  
  
## See Also  
 [DONT_SAVE_VSGLOG_TO_TEMP](../debugger/dont-save-vsglog-to-temp.md)



