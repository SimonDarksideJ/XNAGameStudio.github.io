# XNA Tutorial using C# and HLSL Series 4 – Overview

Welcome to this 4th instalment of my XNA Tutorials for C#. In the first series, we’ve seen how to create a terrain. This Series we’ll go over that again – only this time using a totally different approach. Since we’ve learned (a bit beyond) the basics of HLSL in Series 3, let’s put that knowledge to good use.

How excellent your latest idea for a new game might be, you’ll hardly impress anyone if you can only move some self-drawn crosses or dots over a 2D board. In this Series, we’ll be creating a terrain that you can use immediately as a start for your game. As this series relies heavily on HLSL, you will like to go through Series 3 before you start reading on this one.

Because a terrain looks a lot nicer fullscreen that it does in a window, you can have a look at this fullscreenshot or at this one. And yes, the water moves and the clouds change shape ;)
So, in a nutshell, what will be covered in this series?

- First-person mouse camera
- Multitexturing with high resolution textures close to the camera
- Realistic, moving water with reflections-refractions
- Bump mapping
- Specular reflections
- Perlin noise
- Shape-changing clouds
- HLSL cylindrical billboarding
- Region growing

During this series you will need to download the following resources:
Starting code: Series4Effects.fx
Starting code: heightmap.bmp
Textured terrain: grass.dds
Multitexturing: sand.dds
Multitexturing: rocks.dds
Multitexturing: snow.dds
Skydome: dome.x
Skydome: cloudMap.jpg
Ripples: waterbump.dds
Billboarding: bbEffect.fx
Billboarding: tree.dds
Region growing: treeMap.jpg

Before you move on to the first chapter, you can find a few more screenshots below.

![Overview](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA4-00Overview1.jpg?raw=true)

![Overview](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA4-00Overview2.jpg?raw=true)

![Overview](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA4-00Overview3.jpg?raw=true)

## Chapters

1. [Starting point](Riemers3DXNA4advterrain01starting)
2. [Adding a first-person mouse camera](Riemers3DXNA4advterrain02mousecamera)
3. [Texturing our terrain](Riemers3DXNA4advterrain03texturedterrain)
4. [Multitexturing](Riemers3DXNA4advterrain04multitexturing)
5. [Adding more detail](Riemers3DXNA4advterrain05addingdetail)
6. [Drawing a simple skydome](Riemers3DXNA4advterrain06skydome)
7. [The water technique](Riemers3DXNA4advterrain07watertechnique)
8. [Rendering the refractive map](Riemers3DXNA4advterrain08refraction)
9. [Rendering the Reflection map](Riemers3DXNA4advterrain09reflection)
10. [Perfect mirror](Riemers3DXNA4advterrain10perfectmirror)
11. [Rippling water](Riemers3DXNA4advterrain11ripples)
12. [Blending in refractions using the Fresnel term](Riemers3DXNA4advterrain12fresnel)
13. [Making our water move](Riemers3DXNA4advterrain13movingwater)
14. [Specular Highlights](Riemers3DXNA4advterrain14specularhighlights)
15. [Cylindrical Billboarding](Riemers3DXNA4advterrain15billboarding)
16. [Region growing](Riemers3DXNA4advterrain16regiongrowing)
17. [Billboarding](Riemers3DXNA4advterrain17billboardingrendrstates)
18. [2D Perlin noise](Riemers3DXNA4advterrain18perlinnoise)
19. 
