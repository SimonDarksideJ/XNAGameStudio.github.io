# The water technique

Now we have removed every single pixel of solid background color in our screen, it’s time we move on to the most complex technique of this series: the water. It’s not the most difficult one; it just takes a few steps to get it right. But it’s more than worth the effort: the addition of realistic water to a 3D scene will greatly enhance its reality.

Because there are quite a few steps involved, instead of jumping into the first step I think it’s a better idea to first quickly go through the whole sequence of steps, to get a better overview.

For 3D games, you roughly have 2 kinds of water: ocean water and lake water. The difference of course is the height and length of the waves: for ocean water, you need a lot of vertices of which the height is adjusted in the vertex shader.

See Recipe 5-17 on how to create a compelling 3D ocean.

For lake water, as in our case, we will do with a completely flat surface, in which we create the illusion of ripples. Because of this, to draw the flat surface of our water we’ll only need 2 triangles! The effect of water is completely created in the pixel shader, as we’ll see below.

Remember, in the pixel shader, the main question we’re after is: for each pixel of the water, what will its color be? You could simply take a texture of some waves, and put it over the surface. But in reality, the color of water depends completely on its surroundings: it is partly a mirror, so we see the reflections of the surrounding scene in the water. But it is also partly transparent, so we also see a bit of the underlying sand shining through, which is called the refractive color. So the final color of the water is a smart combination of the reflective and the refractive color.

I’ve made a flowchart of the steps we’ll go through in the process, so each chapter you can situate it in the bigger picture:

![Flowchart](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA4-07Water1.jpg?raw=true)

To start with, we’ll need to know the reflective and refractive color for each pixel, so we’ll start by rendering these maps into 2 textures. Now imagine for each pixel we would only take the reflection color. This would give a perfect mirror. But real water never looks like a real mirror, it is always deformed by ripples. So before our pixel shader will sample the color from both maps, we will add a small deformation to the texture coordinates of each pixel, which will simulate these ripples.

Next, both colors are blended together, in such a way that when you look straight into the water, all you see is the refractive color (the underlying sand). And when you look over the water, all you see is the reflective color. The blendfactor is called the Fresnel term.

The color we get at this moment would be the color of perfectly pure and clean water. To make it a bit more realistic, at the end we’ll blend in a bit of gray-blue. (As a small extension, you can also sample the heightmap of the terrain in the pixel shader, so deeper patches of water only have deep-blue as refractive color.)

That’s it for the start, but we’ll go a bit further and throw in some kind of bump mapping, so we can add specular reflections (where the sunlight is directly reflected in the water).

But that’s for later, let’s first try to finish the flowchart displayed above. In the screenshot below, you can see the different steps: close to the camera, where we look directly into the water, we see the refractive color (the sand). When the angle at which we look into the water gets more sharp, we see that reflections are getting more important.

Also important to note, is how the reflections of the mountain are deformed by the ripples. And as a last note: the water very close to the border is a bit blue-ish, which is due to the last step.

![Summary](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA4-07Water2.jpg?raw=true)

Don’t worry if you’re not too sure about the whole process; each step will be explained in greater detail in the following chapters. Let’s go!

## Next Steps

[Rendering the refractive map](Riemers3DXNA4advterrain08refraction)
