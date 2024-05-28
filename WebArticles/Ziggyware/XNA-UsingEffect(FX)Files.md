# Using Effect Files in XNA


Using Effect (.FX) files in XNA is very easy.

We start by declaring an Effect:

```csharp
    Effect effectNormalMapping;
```


Now lets Load an .FX file:

We'll use the Content pipeline to load an instance of the Effect class:

```csharp
    effectNormalMapping = Content.Load<Effect>("NormalMap");
```


Now lets set some Parameters in the effect, this generally needs to go in the update loop as they change each frame:

```csharp
    Matrix worldViewProj = world * view * proj;
    effectNormalMapping.Parameters["World"].SetValue(world);
    effectNormalMapping.Parameters["WorldI"].SetValue(Matrix.Invert(world));
    effectNormalMapping.Parameters["WorldViewProjection"].SetValue(worldViewProj);
```


And render using the effect:

(We assume there is only one technique and pass for this effect. We can make this more complex to handle multiple techniques and passes easily :) )

```csharp
    effectNormalMapping.Techniques[0].Passes[0].Begin();

    graphicsDevice.DrawUserPrimitives<VertexPosTexNormalTanBitan>(
        PrimitiveType.TriangleStrip, 2, diceVerts);
```

Note the use of a custom vertex during this render. 

Consult the custom vertex article if you would like to see how to [create your own custom vertices](https://github.com/simondarksidej/XNAGameStudio/wiki/XNA-CustomVertices).