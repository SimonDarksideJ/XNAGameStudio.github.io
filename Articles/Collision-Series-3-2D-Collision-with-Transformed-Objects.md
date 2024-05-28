# Collision Series 3: 2D Collision with Transformed Objects

|Area|Submitted|Type|
|-|-|-|
Games: 2D Graphics, Games: Collision|6/28/2007|Tutorial
||||

## Description

This tutorial explains how to perform per-pixel collision detection in two dimensions on sprites that use linear transformations such as rotation or scale.

This tutorial builds on the previous tutorial in this series, [Collision Series 2: 2D Per-Pixel Collision](https://github.com/simondarksidej/XNAGameStudio/wiki/Collision_Series_2_2D_Per-Pixel_Collision). Follow the steps in the previous tutorials before starting this one.

## Tutorial Overview

In the previous tutorial, you enhanced your simple obstacle avoidance game by adding per-pixel collisions, which are more accurate than the previously existing bounding rectangle test. The per-pixel technique presented in tutorial 2 accommodates only positioned sprites, without any other transformations. For many games, this is completely sufficient. However, if your game requires objects that are rotated, scaled, or otherwise linearly transformed, you are going to need a more sophisticated per-pixel collision test.

## Other items in the Collision Series

* [Collision Series 1: 2D Rectangle Collision](https://github.com/simondarksidej/XNAGameStudio/wiki/Collision_Series_1_2D_Rectangle_Collision)

* [Collision Series 2: 2D Per-Pixel Collision](https://github.com/simondarksidej/XNAGameStudio/wiki/Collision_Series_2_2D_Per-Pixel_Collision)

* [Collision Series 4: Collision with a Heightmap](https://github.com/simondarksidej/XNAGameStudio/wiki/Collision_Series_4_Collision_with_a_Heightmap)

* [Collision Series 5: Heightmap Collision with Normals](https://github.com/simondarksidej/XNAGameStudio/wiki/Collision_Series_5_Heightmap_Collision_with_Normals)

> All content and source code downloaded from this page are bound to the Microsoft Permissive License (Ms-PL).

![XNA_Collision3_2D_Transform_01_small.jpg](https://github.com/simondarksidej/XNAGameStudio/blob/archive/Images/XNA_Collision3_2D_Transform_01_small.jpg?raw=true)
![XNA_Collision3_2D_Transform_02_small.jpg](https://github.com/simondarksidej/XNAGameStudio/blob/archive/Images/XNA_Collision3_2D_Transform_02_small.jpg?raw=true)

Download | Size | Description
---|---|---|
[TransformedCollisionSample_4_0](https://github.com/simondarksidej/XNAGameStudio/tree/archive/Samples/TransformedCollisionSample_4_0) | 0.18MB | Source code and assets for the Collision Series 3: 2D Transformed Per-Pixel Tutorial (XNA Game Studio 4.0).
[TransformedCollisionSample_4_0.zip](https://github.com/simondarksidej/XNAGameStudioZips/raw/zips/TransformedCollisionSample_4_0.zip) | 0.18MB | Source code and assets for the Collision Series 3: 2D Transformed Per-Pixel Tutorial (XNA Game Studio 4.0).
||||
