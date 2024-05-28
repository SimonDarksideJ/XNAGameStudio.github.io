# Drawing your first Triangle

This chapter will cover the basics of drawing. First, a few things you should know about.

## Creating 3D content

Every object drawn in 3D is drawn using triangles. Even spheres can be represented using triangles if you use enough of them. Surprisingly enough, a triangle is defined by 3 points. Every point is defined by a vector, specifying the X, Y, and Z coordinates of the point. However, just knowing the coordinates of a point might not be enough, for example, you might also want to define a color for a given point as well, this is where a vertex (pl. vertices) comes in as it represents a list of properties for a given point, including the position, color and so on.

MonoGame has a structure that fits perfectly to hold our vertex information, the **VertexPositionColor** struct, a vertex of this type can hold a position and a color, which is perfect to begin with. To define a triangle, we need 3 of these vertices, which we will store in an array.

So let us declare this variable at the top of our class in the Properties section:

```csharp
    private VertexPositionColor[] _vertices;
```

Next, we will add a small method to our code called **SetUpVertices** which will fill this array with 3 vertices:

```csharp
    private void SetUpVertices()
    {
        _vertices = new VertexPositionColor[3];

        _vertices[0].Position = new Vector3(-0.5f, -0.5f, 0f);
        _vertices[0].Color = Color.Red;
        _vertices[1].Position = new Vector3(0, 0.5f, 0f);
        _vertices[1].Color = Color.Green;
        _vertices[2].Position = new Vector3(0.5f, -0.5f, 0f);
        _vertices[2].Color = Color.Yellow;
    }
```

The array is initialized to hold 3 vertices which we then fill. For now, we are just using coordinates that are relative to the screen:

* The (0,0,0) point would be the middle of our screen
* The (-1, -1, 0) point bottom-left
* And the (1, 1, 0) point top-right.

So in the example above, the first point is halfway to the bottom left of the window and the second point is halfway to the top in the middle of our screen.

> As these points are not really 3D coordinates, they do not need to be transformed to 2D coords. Hence, the name of the technique: ‘Pretransformed’

The **'f'** behind some of the numbers indicates the values are floats, the format of preference when working with MonoGame. We also set each of the vertices to different colors for variety.

To finish, we still need to call this **SetUpVertices** method, as it uses the device we need to call it at the end of the LoadContent method which is called after **Initialize**, where the graphics card is setup:

```csharp
    SetUpVertices();
```

## Drawing primitives

All we have to do now is tell the device to draw the triangle! Go to our Draw method, where we should draw the triangle after the call to pass.Apply (within the foreach loop):

```csharp
    _device.DrawUserPrimitives(PrimitiveType.TriangleList, _vertices, 0, 1, VertexPositionColor.VertexDeclaration);
```

This line actually tells our graphics card to draw the triangle:

* We want to draw 1 triangle from the vertices array, starting at vertex 0.
* TriangleList means that our vertices array contains a list of triangles (in our case, a list of only 1 triangle).
* If you would want to draw 4 triangles, you would need an array of 12 vertices.
* Another possibility is to use a TriangleStrip, which can perform a lot faster but is only useful to draw triangles that are connected to each other.
* The last parameter specifies the VertexDeclaration, which is quite important. We have stored the position and color data for 3 vertices inside an array.

When we instruct MonoGame to render a triangle based on this data, MonoGame will put all this data into a byte stream and send it over to the graphics card. Our graphics card receives the byte stream but does not know what is in there and that is where the **VertexDeclaration** comes in, this object tells the graphics device what kind of vertices it can expect.

The **VertexDeclaration** is very important, as you will always need it before you can render any triangle to the screen. By specifying the **VertexPositionColor.VertexDeclaration**, we inform the graphics card that there are vertices coming that contain Position and Color data.

> We will see how we can create our own vertex types later on in this series.

Running this code should give you this window:

![Summary](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA1-03Triangles1.png?raw=true)

That is all there is to it! Running this code will already give you a colorful triangle on a blue background. Feel free to experiment with the colors and the coordinates.

## Exercises

You can try these exercises to practice what you have learned:

* Make the bottom-left corner of the triangle collide with the bottom-left corner of your window.
* Render 2 triangles, covering your whole window.

## The code so far

```csharp
using Microsoft.Xna.Framework;
using Microsoft.Xna.Framework.Graphics;
using Microsoft.Xna.Framework.Input;

namespace Series3D1
{
    public class Game1 : Game
    {
        //Properties
        private GraphicsDeviceManager _graphics;
        private GraphicsDevice _device;
        private Effect _effect;
        private VertexPositionColor[] _vertices;

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
            _vertices = new VertexPositionColor[3];

            _vertices[0].Position = new Vector3(-0.5f, -0.5f, 0f);
            _vertices[0].Color = Color.Red;
            _vertices[1].Position = new Vector3(0, 0.5f, 0f);
            _vertices[1].Color = Color.Green;
            _vertices[2].Position = new Vector3(0.5f, -0.5f, 0f);
            _vertices[2].Color = Color.Yellow;
        }

        protected override void LoadContent()
        {
            // TODO: use this.Content to load your game content here
            _device = _graphics.GraphicsDevice;

            _effect = Content.Load<Effect>("effects");

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
            _device.Clear(Color.DarkSlateBlue);

            // TODO: Add your drawing code here
            _effect.CurrentTechnique = _effect.Techniques["Pretransformed"];

            foreach (EffectPass pass in _effect.CurrentTechnique.Passes)
            {
                pass.Apply();
                _device.DrawUserPrimitives(PrimitiveType.TriangleList, _vertices, 0, 1, VertexPositionColor.VertexDeclaration);
            }

            base.Draw(gameTime);
        }
    }
}
```

## Next Steps

[World Space coordinates and the camera](Riemers3DXNA1Terrain04worldspace)
