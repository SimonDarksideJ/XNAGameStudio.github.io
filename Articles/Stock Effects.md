# Stock Effects

|Area|Submitted|Type|
|-|-|-|
Games: Shaders|7/28/2010|Code Sample
||||

## Description

Stock effects provides source code for the five effects (BasicEffect, SkinnedEffect, EnvironmentMapEffect, DualTextureEffect, and AlphaTestEffect), and the default shader used by SpriteBatch (SpriteEffect), built into the XNA Framework. There's also a command-line utility (CompileEffect) that uses the Content Pipeline to compile a .fx source file into a binary blob that can be passed directly to the XNA Framework Effect class constructor.

## Sample Overview

This code is provided for educational purposes. It may be a useful starting point when you create more advanced shaders.

Although the built-in effect classes are simple and easy to use, the shader code behind them is complex because these effects support many rendering options within a single shader. BasicEffect, for example, allows you to toggle on or off texturing, vertex colors, and lighting, and to select per-vertex or per-pixel lighting. To support all possible permutations of these options, BasicEffect ends up needing no less than 20 different vertex shaders and 10 pixel shaders, all of which are combined within a single effect file.

If you want an easy starting point to learn shader programming, use the [*Shader Series*](https://github.com/simondarksidej/XNAGameStudio/wiki/Shader_Series_Introduction) samples. The shader samples are simpler to use than the effects described here because they don't include so many adjustable options.

> All content and source code downloaded from this page are bound to the Microsoft Permissive License (Ms-PL).

Download | Size | Description
---|---|---|
[StockEffectsSample_4_0](https://github.com/simondarksidej/XNAGameStudio/tree/archive/Samples/StockEffectsSample_4_0) | 0.07MB | Source code and assets for the Stock Effects Sample (XNA Game Studio 4.0).
[StockEffectsSample_4_0.zip](https://github.com/simondarksidej/XNAGameStudioZips/raw/zips/StockEffectsSample_4_0.zip) | 0.07MB | Source code and assets for the Stock Effects Sample (XNA Game Studio 4.0).
||||
