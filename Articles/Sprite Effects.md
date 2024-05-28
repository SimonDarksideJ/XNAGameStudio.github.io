# Sprite Effects

|Area|Submitted|Type|
|-|-|-|
Games: 2D Graphics, Games: Content Pipeline, Games: Graphics, Games: Postprocessing, Games: Shaders|2/19/2007|Code Sample
||||

## Description

This sample shows how shaders can be used to implement special effects when drawing 2D graphics with SpriteBatch. It uses two custom content pipeline processors to prepare the sprite textures in special ways.

## Sample Overview

The sample includes five different rendering techniques. To switch to the next technique, press the A button on your gamepad, or SPACEBAR on your keyboard.

The first custom content pipeline processor merges color and alpha information from two separate source files into a single texture. This allows the alpha data to be drawn as a simple grayscale image, which can be convenient if you're using a paint program that doesn't properly support images with embedded alpha channels.

The second custom processor converts an image containing grayscale height values into a normal-map texture.

All content and source code downloaded from this page are bound to the Microsoft Permissive License (Ms-PL).

![XNA_SpriteEffects_01_small.jpg](https://github.com/simondarksidej/XNAGameStudio/blob/archive/Images/XNA_SpriteEffects_01_small.jpg?raw=true)
![XNA_SpriteEffects_02_small.jpg](https://github.com/simondarksidej/XNAGameStudio/blob/archive/Images/XNA_SpriteEffects_02_small.jpg?raw=true)
![XNA_SpriteEffects_03_small.jpg](https://github.com/simondarksidej/XNAGameStudio/blob/archive/Images/XNA_SpriteEffects_03_small.jpg?raw=true)

Download | Size | Description
---|---|---|
[SpriteEffectsSample_4_0](https://github.com/simondarksidej/XNAGameStudio/tree/archive/Samples/SpriteEffectsSample_4_0) | 0.30MB | Source code and assets for the Sprite Effects Sample (XNA Game Studio 4.0).
[SpriteEffectsSample_4_0.zip](https://github.com/simondarksidej/XNAGameStudioZips/raw/zips/SpriteEffectsSample_4_0.zip) | 0.30MB | Source code and assets for the Sprite Effects Sample (XNA Game Studio 4.0).
||||
