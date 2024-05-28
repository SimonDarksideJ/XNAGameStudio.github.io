# Normal Mapping

|Area|Submitted|Type|
|-|-|-|
Games: 3D Graphics, Games: Content Pipeline, Games: Shaders|2/19/2007|Code Sample
||||

## Description

This sample builds on the concepts illustrated in the [Custom Model Effect](https://github.com/simondarksidej/XNAGameStudio/wiki/Custom_Model_Effect) sample, and it applies a normal mapping effect to a mesh. The sample also uses a custom content processor to demonstrate two additional concepts: adding and removing per-vertex data from a mesh, and reading opaque data.

## Sample Overview

The sample demonstrates how to use a custom effect to render a model with a normal map, creating the appearance of a bumpy surface without needing to render additional geometry.

A custom content pipeline processor applies the normal mapping effect to the model during the content build process. The processor also creates additional per-vertex data channels for the binormal and tangent information, which the normal mapping effect uses. To ensure that the run-time vertex declaration matches the inputs to the vertex shader, the processor removes all vertex channels that the shader doesn't use. Finally, the processor uses opaque data to determine which normal map to use, and then applies it to the material.

To make rendering more efficient at runtime, a second custom processor changes the normal map pixel format from an encoded unsigned format to a signed format.

> All content and source code downloaded from this page are bound to the Microsoft Permissive License (Ms-PL).

![](https://github.com/simondarksidej/XNAGameStudio/blob/archive/Images/XNA_NormalMappingEffect_01_small.jpg?raw=true)
![](https://github.com/simondarksidej/XNAGameStudio/blob/archive/Images/XNA_NormalMappingEffect_02_small.jpg?raw=true)
![](https://github.com/simondarksidej/XNAGameStudio/blob/archive/Images/XNA_NormalMappingEffect_03_small.jpg?raw=true)

Download | Size | Description
---|---|---|
[NormalMappingSample_4_0](https://github.com/simondarksidej/XNAGameStudio/tree/archive/Samples/NormalMappingSample_4_0) | 8.81MB | Source code and assets for the Normal Mapping Sample (XNA Game Studio 4.0)
[NormalMappingSample_4_0.zip](https://github.com/simondarksidej/XNAGameStudioZips/raw/zips/NormalMappingSample_4_0.zip) | 8.81MB | Source code and assets for the Normal Mapping Sample (XNA Game Studio 4.0)
||||
