# Bloom Postprocess

|Area|Submitted|Type|
|-|-|-|
Games: Graphics, Games: Postprocessing, Games: Shaders|4/26/2007|Code Sample
||||

## Description

This sample shows how to use bloom post-processing filters to add a glow effect over the top of an existing scene.

## Sample Overview

Bloom post-processing emulates the visual effect of bright lights and glowing objects. It does this by extracting the brightest parts of an image to a custom render target, blurring these bright areas, and then adding the blurred result back into the original image.

Because bloom is implemented entirely as a post-process, it can easily be used over the top of any other 2D or 3D rendering techniques. In addition to the more extreme glowing effects, when used subtly it provides a useful softening that can make computer graphics look more organic and help to hide artifacts from elsewhere in your rendering. Particle systems, for example, often look better with a subtle bloom applied over the top.

> All content and source code downloaded from this page are bound to the Microsoft Permissive License (Ms-PL).

![](https://github.com/simondarksidej/XNAGameStudio/blob/archive/Images/XNA_Bloom_01_small.jpg?raw=true)
![](https://github.com/simondarksidej/XNAGameStudio/blob/archive/Images/XNA_Bloom_02_small.jpg?raw=true)

Download | Size | Description
---|---|---|
[BloomSample_4_0](https://github.com/simondarksidej/XNAGameStudio/tree/archive/Samples/BloomSample_4_0) | 11.48MB | Source code and assets for the Bloom Postprocessing Sample (XNA Game Studio 4.0)
[BloomSample_4_0.zip](https://github.com/simondarksidej/XNAGameStudioZips/raw/zips/BloomSample_4_0.zip) | 11.48MB | Source code and assets for the Bloom Postprocessing Sample (XNA Game Studio 4.0)
||||
