# Experimenting with Lights in XNA

Even when using colors and a Z buffer, your terrain seems to miss some depth detail when you turn on the Solid FillMode. By adding some lighting we can make it look a lot better. In this chapter, we will see the impact of a light on 2 simple triangles so we can have a better understanding of how lights work in MonoGame. We will be using the code from the 'World space' chapter, so reload that code now.

## Calculating normals

For this chapter, we will be using a directional light. Imagine this as the sunlight, which is light that travels in one singular direction. To calculate the effect of light hitting a triangle, MonoGame needs to understand how the light will "bounce" off the triangle. To do this we calculate what is called a 'normal' for every vertex. Consider the next figure:

![Normals](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA1-11Lighting1.jpg?raw=true)

If you have a light source a), and you shine a light on the 3 surfaces shown, how is MonoGame supposed to know that surface 1 should be lit more intensely than surface 3? If you look at the thin red lines in figure b), you will notice that their length is a nice indication of how much light you would want to be reflected (and thus seen) on every surface. So how can we calculate the length of these lines? Actually, MonoGame does the job for us, all we have to do is to provide the blue arrow perpendicular (with an angle of 90 degrees, the thin blue lines) to every surface and MonoGame does the rest (a simple cosine projection) for us!

This is why we need to add normals (the perpendicular blue lines) to our vertex data. The **VertexPositionColor** will no longer do, as it does not allow us to store a normal for each vertex, and unfortunately, MonoGame does not offer a structure that can contain a position, a color, and a normal. But that is not a problem as we can easily create one of our own.

> The main reason that MonoGame does not provide such a structure is that you would normally be using a texture with a model, and provides a vertex structure for that common use.  But thanks to the extensibility in MonoGame, you can create any structure you meet your needs and it will still "just work".

Let us put this code at the top of our class, immediately above our variable declarations:

```csharp
    public struct VertexPositionColorNormal
    {
        public Vector3 Position;
        public Color Color;
        public Vector3 Normal;

        public readonly static VertexDeclaration VertexDeclaration = new VertexDeclaration
        (
            new VertexElement(0, VertexElementFormat.Vector3, VertexElementUsage.Position, 0),
            new VertexElement(sizeof(float) * 3, VertexElementFormat.Color, VertexElementUsage.Color, 0),
            new VertexElement(sizeof(float) * 3 + 4, VertexElementFormat.Vector3, VertexElementUsage.Normal, 0)
        );
    }
```

This might look complicated, but I am sure you can understand the first 3 lines which is a new struct can hold a position, a color, and a normal, exactly what we need! The bottom of the struct is a little more complex, and we will discuss it in its full detail in [Series 3](Riemers3DXNA3hlsloverview.md). For now, think of it as a manual for the graphics card to understand what kind of data is contained inside each vertex.

This allows us to change our vertex variable declaration to, so replace this in the Properties section:

```csharp
    private VertexPositionColorNormal[] _vertices;
```

Now we could start defining triangles with normals, but first, let us have a look at the next picture, where the arrows at the top represent the direction of the light and the color bar below the drawing represents the color of every pixel along our surface:

![Surfaces](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA1-11Lighting2.jpg?raw=true)

If we simply define the perpendicular vectors, it is easy to see there will be an 'edge' in the lighting (see the bar directly above the ‘a’)). This is because the right surface is lit more than the left surface. So it will be easy to see the surface is made of separate triangles, however, if we place in the shared top vertex, a 'normal' as shown in figure b), MonoGame automatically interpolates the lighting in every point of our surface! This will give a much smoother effect, as you can see in the bar above the b). This vector is the average of the 2 top vectors of a).

As always, the average of 2 vectors can be found by summing them and by dividing them by two.

To demonstrate this example, we will first reset the camera position in the **SetUpCamera** method:

```csharp
    _viewMatrix = Matrix.CreateLookAt(new Vector3(0, -40, 100), new Vector3(0, 50, 0), new Vector3(0, 1, 0));
```

Next, we will update our **SetUpVertices** method with 6 vertices that define the 2 triangles of the example above using our new **VertexPositionColorNormal** declaration:

```csharp
    private void SetUpVertices()
    {
        _vertices = new VertexPositionColorNormal[6];

        _vertices[0].Position = new Vector3(0f, 0f, 50f);
        _vertices[0].Color = Color.Blue;
        _vertices[0].Normal = new Vector3(1, 0, 1);
        _vertices[1].Position = new Vector3(50f, 0f, 00f);
        _vertices[1].Color = Color.Blue;
        _vertices[1].Normal = new Vector3(1, 0, 1);
        _vertices[2].Position = new Vector3(0f, 50f, 50f);
        _vertices[2].Color = Color.Blue;
        _vertices[2].Normal = new Vector3(1, 0, 1);

        _vertices[3].Position = new Vector3(-50f, 0f, 0f);
        _vertices[3].Color = Color.Blue;
        _vertices[3].Normal = new Vector3(-1, 0, 1);
        _vertices[4].Position = new Vector3(0f, 0f, 50f);
        _vertices[4].Color = Color.Blue;
        _vertices[4].Normal = new Vector3(-1, 0, 1);
        _vertices[5].Position = new Vector3(0f, 50f, 50f);
        _vertices[5].Color = Color.Blue;
        _vertices[5].Normal = new Vector3(-1, 0, 1);

        for (int i = 0; i <_vertices.Length; i++)
        {
            _vertices[i].Normal.Normalize();
        }
    }
```

This defines the 2 surfaces of the picture above. By adding a Z coordinate (other than 0), the triangles are now 3D. You can notice that I have defined the "Normal" vectors perpendicular to the triangles, as to reflect example a) of the image above.

In the end, we 'normalize our normals'. These 2 words have absolutely nothing to do with each other, it simply means we scale our normal vectors so their lengths become exactly one, this is required for correct lighting, as the length of the normals has an impact on the amount of lighting.

All that there is left to do is change the Draw method a bit:

```csharp
    _device.DrawUserPrimitives(PrimitiveType.TriangleList, _vertices, 0, 2, VertexPositionColorNormal.VertexDeclaration);
```

And let the graphics card know that from now on it has to use the Colored technique. This technique works exactly the same as the ColoredNoShading technique, but also adds correct lighting (in case you provided correct normals!):

```csharp
    effect.CurrentTechnique = effect.Techniques["Colored"];
```

When you run this code, you should see an arrow (our 2 triangles), but you don’t see any shading because we haven’t yet defined the light! We can define this by setting these additional parameters of our effect:

```csharp
    _effect.Parameters["xEnableLighting"].SetValue(true);
    Vector3 lightDirection = new Vector3(0.5f, 0, -1.0f);
    lightDirection.Normalize();
    _effect.Parameters["xLightDirection"].SetValue(lightDirection);
```

This instructs our technique to enable lighting calculations, now that the technique needs the normals, and we set the direction of our light. Note again that you need to normalize this vector so that its lengths become one, otherwise the length of this vector influences the strength of the shading, while you want the shading to depend solely on the direction of the incoming light. You might also want to change the background color to black, to get a better view.

Now if you run this code and you will see what I mean with 'edged lighting', the light shines brightly on the left panel, yet the right panel is darker. You can clearly see the difference between the two triangles! This is what was shown in the left part of the example image above.

Now it is time to combine the vectors on the edge that is shared by the 2 triangles from (-1,0,1) and (1,0,1) to (-1+1,0,1+1)/2 = (0,0,1):

```csharp
    _vertices[0].Position = new Vector3(0f, 0f, 50f);
    _vertices[0].Color = Color.Blue;
    _vertices[0].Normal = new Vector3(0, 0, 1);
    _vertices[1].Position = new Vector3(50f, 0f, 00f);
    _vertices[1].Color = Color.Blue;
    _vertices[1].Normal = new Vector3(1, 0, 1);
    _vertices[2].Position = new Vector3(0f, 50f, 50f);
    _vertices[2].Color = Color.Blue;
    _vertices[2].Normal = new Vector3(0, 0, 1);

    _vertices[3].Position = new Vector3(-50f, 0f, 0f);
    _vertices[3].Color = Color.Blue;
    _vertices[3].Normal = new Vector3(-1, 0, 1);
    _vertices[4].Position = new Vector3(0f, 0f, 50f);
    _vertices[4].Color = Color.Blue;
    _vertices[4].Normal = new Vector3(0, 0, 1);
    _vertices[5].Position = new Vector3(0f, 50f, 50f);
    _vertices[5].Color = Color.Blue;
    _vertices[5].Normal = new Vector3(0, 0, 1);
```

When you run this code, you'll see that the reflection is nicely distributed from the dark right tip to the brighter left panel. It's not difficult to imagine that this effect will give a much nicer and smoother effect on a large number of triangles, such as our terrain.

![Summary](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA1-11Lighting1.png?raw=true)

Also, you can see that since the middle vertices use the SAME normal, we could again combine the 2x2 shared vertices to 2x1 vertices using an index buffer. Notice however, that in the case you really WANT to create an edge, you need to specify the 2 separate normal vectors.

## Exercises

You can try these exercises to practice what you have learned:

* Try moving the normals a bit to the left and right in both the separate and shared case.

## Code for this example (just for test)

```csharp
using Microsoft.Xna.Framework;
using Microsoft.Xna.Framework.Graphics;
using Microsoft.Xna.Framework.Input;

namespace Series3D1
{
    public class Game1 : Game
    {
        public struct VertexPositionColorNormal
        {
            public Vector3 Position;
            public Color Color;
            public Vector3 Normal;

            public readonly static VertexDeclaration VertexDeclaration = new VertexDeclaration
            (
                new VertexElement(0, VertexElementFormat.Vector3, VertexElementUsage.Position, 0),
                new VertexElement(sizeof(float) * 3, VertexElementFormat.Color, VertexElementUsage.Color, 0),
                new VertexElement(sizeof(float) * 3 + 4, VertexElementFormat.Vector3, VertexElementUsage.Normal, 0)
            );
        }

        //Properties
        private GraphicsDeviceManager _graphics;
        private GraphicsDevice _device;
        private Effect _effect;
        private VertexPositionColorNormal[] _vertices;
        private Matrix _viewMatrix;
        private Matrix _projectionMatrix;

        public Game1()
        {
            _graphics = new GraphicsDeviceManager(this);
            Content.RootDirectory = "Content";
            IsMouseVisible = true;
        }

        protected override void Initialize()
        {
            // TODO: Add your initialization logic here
            _graphics.PreferredBackBufferWidth = 500;
            _graphics.PreferredBackBufferHeight = 500;
            _graphics.IsFullScreen = false;
            _graphics.ApplyChanges();
            Window.Title = "Riemer's MonoGame Tutorials -- 3D Series 1";

            base.Initialize();
        }

        private void SetUpVertices()
        {
            _vertices = new VertexPositionColorNormal[6];

            _vertices[0].Position = new Vector3(0f, 0f, 50f);
            _vertices[0].Color = Color.Blue;
            _vertices[0].Normal = new Vector3(0, 0, 1);
            _vertices[1].Position = new Vector3(50f, 0f, 00f);
            _vertices[1].Color = Color.Blue;
            _vertices[1].Normal = new Vector3(1, 0, 1);
            _vertices[2].Position = new Vector3(0f, 50f, 50f);
            _vertices[2].Color = Color.Blue;
            _vertices[2].Normal = new Vector3(0, 0, 1);

            _vertices[3].Position = new Vector3(-50f, 0f, 0f);
            _vertices[3].Color = Color.Blue;
            _vertices[3].Normal = new Vector3(-1, 0, 1);
            _vertices[4].Position = new Vector3(0f, 0f, 50f);
            _vertices[4].Color = Color.Blue;
            _vertices[4].Normal = new Vector3(0, 0, 1);
            _vertices[5].Position = new Vector3(0f, 50f, 50f);
            _vertices[5].Color = Color.Blue;
            _vertices[5].Normal = new Vector3(0, 0, 1);

            for (int i = 0; i < _vertices.Length; i++)
            {
                _vertices[i].Normal.Normalize();
            }
        }

        private void SetUpCamera()
        {
            _viewMatrix = Matrix.CreateLookAt(new Vector3(0, -40, 100), new Vector3(0, 50, 0), new Vector3(0, 1, 0));
            _projectionMatrix = Matrix.CreatePerspectiveFieldOfView(MathHelper.PiOver4, _device.Viewport.AspectRatio, 1.0f, 300.0f);
        }

        protected override void LoadContent()
        {
            // TODO: use this.Content to load your game content here
            _device = _graphics.GraphicsDevice;

            _effect = Content.Load<Effect>("effects");

            SetUpCamera();

            SetUpVertices();
        }

        protected override void Update(GameTime gameTime)
        {
            if (GamePad.GetState(PlayerIndex.One).Buttons.Back == ButtonState.Pressed || Keyboard.GetState().IsKeyDown(Keys.Escape))
                Exit();

            // TODO: Add your update logic here

            base.Update(gameTime);
        }

        protected override void Draw(GameTime gameTime)
        {
            _device.Clear(Color.Black);

            RasterizerState rs = new RasterizerState();
            rs.CullMode = CullMode.None;
            _device.RasterizerState = rs;

            // TODO: Add your drawing code here
            _effect.CurrentTechnique = _effect.Techniques["Colored"];
            _effect.Parameters["xView"].SetValue(_viewMatrix);
            _effect.Parameters["xProjection"].SetValue(_projectionMatrix);
            _effect.Parameters["xWorld"].SetValue(Matrix.Identity);

            _effect.Parameters["xEnableLighting"].SetValue(true);
            Vector3 lightDirection = new Vector3(0.5f, 0, -1.0f);
            lightDirection.Normalize();
            _effect.Parameters["xLightDirection"].SetValue(lightDirection);

            foreach (EffectPass pass in _effect.CurrentTechnique.Passes)
            {
                pass.Apply();
                _device.DrawUserPrimitives(PrimitiveType.TriangleList, _vertices, 0, 2, VertexPositionColorNormal.VertexDeclaration);
            }

            base.Draw(gameTime);
        }
    }
}
```

## Next Steps

[Adding normals to our Terrain – Intuitive approach](Riemers3DXNA1Terrain12terrainlighting)
