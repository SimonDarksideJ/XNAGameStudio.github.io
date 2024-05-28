# WinForms Series 2: Content Loading

|Area|Submitted|Type|
|-|-|-|
Games: Content Pipeline, Games: Graphics|1/10/2008|Code Sample
||||

## Description

This sample shows you how to import arbitrary 3D models at run time by running the Content Pipeline importers and processors dynamically on the fly as opposed to building all the content ahead of time in the usual way.

## Sample Overview

This sample builds on the classes described previously in the WinForms Graphics Device sample. It implements a 3D model viewer control that uses a file selector to let the user choose any .fbx or .x format model. Then it imports that file, loads it, and displays it on the screen.

Note: This sample runs only on Windows. WinForms, MSBuild, and the Content Pipeline aren't available on Xbox 360. If you want your game to run on Xbox 360, you must build all your content ahead of time as part of the Visual Studio project. Note also that the XNA Framework redistributable installer doesn't include the Content Pipeline. This sample will run only on computers that have the XNA Game Studio installed.

## Other items in the Winforms Series

* [WinForms Series 1: Graphics Device](https://github.com/simondarksidej/XNAGameStudio/wiki/WinForms_Series_1_Graphics_Device)

> All content and source code downloaded from this page are bound to the Microsoft Permissive License (Ms-PL).

![XNA_Winform-ContentLoader_01_small.JPG](https://github.com/SimonDarksideJ/XNAGameStudio/raw/archive/Images/XNA_Winform-ContentLoader_01_small.JPG?raw=true)
![XNA_Winform-ContentLoader_02_small.JPG](https://github.com/SimonDarksideJ/XNAGameStudio/raw/archive/Images/XNA_Winform-ContentLoader_02_small.JPG?raw=true)
![XNA_Winform-ContentLoader_03_small.JPG](https://github.com/SimonDarksideJ/XNAGameStudio/raw/archive/Images/XNA_Winform-ContentLoader_03_small.JPG?raw=true)

Download | Size | Description
---|---|---|
[WinFormsContentSample_4_0](https://github.com/simondarksidej/XNAGameStudio/tree/archive/Samples/WinFormsContentSample_4_0) | 1.08MB | Source code and assets for the WinForms Series 2: Content Loading Sample (XNA Game Studio 4.0).
[WinFormsContentSample_4_0.zip](https://github.com/simondarksidej/XNAGameStudioZips/raw/zips/WinFormsContentSample_4_0.zip) | 1.08MB | Source code and assets for the WinForms Series 2: Content Loading Sample (XNA Game Studio 4.0).
||||
