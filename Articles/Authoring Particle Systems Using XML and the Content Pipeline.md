# Authoring Particle Systems Using XML and the Content Pipeline

|Area|Submitted|Type|
|-|-|-|
Games: 3D Graphics, Games: Content Pipeline, Games: Graphics, Games: Shaders|6/28/2007|Tutorial|
||||

---

## Description

This tutorial teaches you how to extend the [Particles 3D](https://github.com/simondarksidej/XNAGameStudio/wiki/Particles_3D) sample so the particle systems are defined by XML files loaded with the XNA Content Pipeline.

## Sample Overview

In the original version of the sample, each particle system was defined by using a separate class that derived from the abstract ParticleSystem base class, and overloaded the InitializeSettings method to specify how the particle system should be displayed. That meant you had to create a new class any time you wanted to add a new type of particle, and had to recompile your game any time you modified one of these classes to alter the particle behavior. Using external XML files allows you to create and tweak any number of different particle systems without needing to change the main game executable.

> All content and source code downloaded from this page are bound to the Microsoft Permissive License (Ms-PL).

![](https://github.com/simondarksidej/XNAGameStudio/blob/archive/Images/XNA_Particle3D_01_small.jpg?raw=true)
![](https://github.com/simondarksidej/XNAGameStudio/blob/archive/Images/XNA_Particle3D_02_small.jpg?raw=true)
![](https://github.com/simondarksidej/XNAGameStudio/blob/archive/Images/XNA_Particle3D_03_small.jpg?raw=true)

Download | Size | Description
---|---|---|
[XmlParticles_4_0](https://github.com/simondarksidej/XNAGameStudio/tree/archive/Samples/XmlParticles_4_0) | 0.40MB | Source code and assets for the XML Particle Systems Tutorial (XNA Game Studio 4.0).
[XmlParticles_4_0.zip](https://github.com/simondarksidej/XNAGameStudioZips/raw/zips/XmlParticles_4_0.zip) | 0.40MB | Source code and assets for the XML Particle Systems Tutorial (XNA Game Studio 4.0).
||||
