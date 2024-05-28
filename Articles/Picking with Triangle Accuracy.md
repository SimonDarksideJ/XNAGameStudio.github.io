# Picking with Triangle Accuracy

|Area|Submitted|Type|
|-|-|-|
Games: Collision, Games: Graphics, Games: Physics|6/28/2007|Code Sample
||||

## Description

This sample shows how to access vertex position data from a model. It then uses that information to implement a per-triangle picking test.

## Sample Overview

This sample builds on the ideas presented in the [Picking](https://github.com/simondarksidej/XNAGameStudio/wiki/Picking) sample. If you're unfamiliar with the process of converting between world space and screen space, and checking for intersection between rays and bounding primitives, you should start with the Picking sample, which explains these basic concepts.

This sample extends the approximate bounding box tests from the [Picking](https://github.com/simondarksidej/XNAGameStudio/wiki/Picking) sample to perform an accurate per-triangle picking test. It loops over all the triangles in the model, performing a ray-to-triangle intersection test for each one. It uses a custom content processor to extract the model vertex position data into a format that can be easily accessed at runtime.

> All content and source code downloaded from this page are bound to the Microsoft Permissive License (Ms-PL).

![](https://github.com/simondarksidej/XNAGameStudio/blob/archive/Images/XNA_PickingTriangle_01_small.jpg?raw=true)
![](https://github.com/simondarksidej/XNAGameStudio/blob/archive/Images/XNA_PickingTriangle_02_small.jpg?raw=true)
![](https://github.com/simondarksidej/XNAGameStudio/blob/archive/Images/XNA_PickingTriangle_03_small.jpg?raw=true)

Download | Size | Description
---|---|---|
[TrianglePickingSample_4_0](https://github.com/simondarksidej/XNAGameStudio/tree/archive/Samples/TrianglePickingSample_4_0) | 3.77MB | Source code and assets for the Picking with Triangle Accuracy Sample (XNA Game Studio 4.0).
[TrianglePickingSample_4_0.zip](https://github.com/simondarksidej/XNAGameStudioZips/raw/zips/TrianglePickingSample_4_0.zip) | 3.77MB | Source code and assets for the Picking with Triangle Accuracy Sample (XNA Game Studio 4.0).
||||
