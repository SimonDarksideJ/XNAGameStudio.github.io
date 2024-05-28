# Terrain creation basics

At last, we have seen enough topics to start creating our terrain.

## Your first terrain

Let us start small by connecting 4x3 specified points. However, we will make our engine dynamic, so that in the next chapter we can load a larger number of points easily.

To do this, we have to create 2 new variables in our class in the Properties section:

```csharp
    private int _terrainWidth = 4;
    private int _terrainHeight = 3;
```

We will approximate that there will be 4x3 points and that they are equidistant. So the only thing we do not know about our points is the Z coordinate. We will use a multi-dimensional array to hold this information which we will also add to the Properties section of our class as well:

```csharp
    private float[,] _heightData;
```

For now, use this method to fill the array :

```csharp
    private void LoadHeightData()
    {
        _heightData = new float[4, 3];
        _heightData[0, 0] = 0;
        _heightData[1, 0] = 0;
        _heightData[2, 0] = 0;
        _heightData[3, 0] = 0;

        _heightData[0, 1] = 0.5f;
        _heightData[1, 1] = 0;
        _heightData[2, 1] = -1.0f;
        _heightData[3, 1] = 0.2f;

        _heightData[0, 2] = 1.0f;
        _heightData[1, 2] = 1.2f;
        _heightData[2, 2] = 0.8f;
        _heightData[3, 2] = 0;
    }
```

And do not forget to call it from within our **LoadContent** method, just make sure you call it before the **SetUpVertices** method, as that method will use the contents of the heightData we are populating.

```csharp
    LoadHeightData();
```

With our height array filled we can now create our vertices. Since we have defined a 4x3 sized terrain, 12 (=terrainWidth*terrainHeight) vertices will do. The points are equidistant (the distance between them is the same), so we can easily change our **SetUpVertices** method.

To begin with, we will not be using the Z coordinate so that you can see the difference later on in this chapter.

```csharp
    private void SetUpVertices()
    {
        _vertices = new VertexPositionColor[_terrainWidth * _terrainHeight];
        for (int x = 0; x < _terrainWidth; x++)
        {
            for (int y = 0; y < _terrainHeight; y++)
            {
                _vertices[x + y * _terrainWidth].Position = new Vector3(x, 0, -y);
                _vertices[x + y * _terrainWidth].Color = Color.White;
            }
        }
    }
```

Nothing magical going on here, you simply define your 12 points and make them white.

> Note that the terrain will grow into the positive X direction (Right) and into the negative Z direction (Forward).

Next comes the difficult part, defining the indices which define the required triangles to connect the 12 vertices. 

## Stitching the terrain together

The best way to do this is by creating two sets of vertices:

![Indices](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA1-7Terrain1.jpg?raw=true)

We will start by drawing the set of triangles drawn in solid lines (as indicated in the image above). To do this, change our **SetUpIndices** method like this:

```csharp
    private void SetUpIndices()
    {
        _indices = new int[(_terrainWidth - 1) * (_terrainHeight - 1) * 3];
        int counter = 0;
        for (int y = 0; y < _terrainHeight - 1; y++)
        {
            for (int x = 0; x < _terrainWidth - 1; x++)
            {
                int lowerLeft = x + y * _terrainWidth;
                int lowerRight = (x + 1) + y * _terrainWidth;
                int topLeft = x + (y + 1) * _terrainWidth;
                int topRight = (x + 1) + (y + 1) * _terrainWidth;

                _indices[counter++] = topLeft;
                _indices[counter++] = lowerRight;
                _indices[counter++] = lowerLeft;
            }
        }
    }
```

> Remember that terrainWidth and terrainHeight are the horizontal and vertical number of vertices in the terrain. We will need 2 rows of 3 triangles, giving us 6 triangles. These will require ***3*2 * 3 = 18 indices (=(terrainWidth-1)*(terrainHeight-1)*3)**.

The first line creates an array capable of storing exactly this amount of integers, then you fill your array with indices. You scan the X and Y coordinates row by row, and this time you create your triangles. During the first row, where y=0, you need to create 6 triangles based on vertices 0 (bottom-left) till 7 (middle-right). Next, y becomes 1, and 1*terrainWidth=4 is added to each index: this time we create our 6 triangles on vertices 4 (middle-left) till 11 (top-right).

To make things easier, I have defined 4 shortcuts for the 4 corner indices of a quad, for each quad you store 3 indices, defining one triangle. Remember culling? It requires us to define the vertices in clockwise order, so you first define the top-left vertex, then the bottom-down vertex, and finally the bottom-left vertex.

The counter variable is an easy way to store vertices to an array, as we increment it each time an index has been added to the array. When the method finishes, the array will contain all indices required to render all bottom-left triangles.

We have coded our Draw method in such a way that  MonoGame draws a number of triangles specified by the length of our indices array, so you can immediately run this code!

You should note the triangles look tiny, so try positioning your camera at (0,10,0) and rerun the program. You should see 6 triangles in the right half of your window, every point of every triangle at the same Z coordinate. Now change the height of your points according to your **heightData** array within the **SetUpVertices** method:

```csharp
    _vertices[x + y * _terrainWidth].Position = new Vector3(x, _heightData[x,y], -y);
```

Running this, you will notice the triangles are no longer positioned in the same plane.

## Drawing the "other" half

Remember you’re still rendering only the bottom-left triangles. So when you would render the triangles with their solid colors instead of only their wireframes, 50% of your grid would not be covered. To fix this, let us define some more indices to render the top-right triangles also. We are using the same vertices, so the only thing we have to change is the **SetUpIndices** method:

```csharp
    private void SetUpIndices()
    {
        _indices = new short[(_terrainWidth - 1) * (_terrainHeight - 1) * 6];
        int counter = 0;
        for (int y = 0; y < _terrainHeight - 1; y++)
        {
            for (int x = 0; x < _terrainWidth - 1; x++)
            {
                short lowerLeft = (short)(x + y * _terrainWidth);
                short lowerRight = (short)((x + 1) + y * _terrainWidth);
                short topLeft = (short)(x + (y + 1) * _terrainWidth);
                short topRight = (short)((x + 1) + (y + 1) * _terrainWidth);

                _indices[counter++] = topLeft;
                _indices[counter++] = lowerRight;
                _indices[counter++] = lowerLeft;

                _indices[counter++] = topLeft;
                _indices[counter++] = topRight;
                _indices[counter++] = lowerRight;
            }
        }
    }
```

We will now be drawing twice as many vertices now, that is why the "* 3" has been replaced by "* 6" when specifying the number of indices. You can also see the second set of triangles being drawn clockwise relative to the camera, first the top-left corner, then the top-right, and finally the bottom-right.

Running this code will give you a better 3-dimensional view. We have especially taken care to only use the variables **terrainWidth** and **terrainHeight**, so that these are the only things we need change to increase the size of our map, together with the contents of the heightData array. It would be nice to find a mechanism to fill this last one automatically, which we will do in the next chapter.

![Summary](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA1-07Terrain1.png?raw=true)

> By default MonoGame runs in what is referred to as "Reach" mode, aimed at targeting the lowest common denominator graphics platform, [you can read more about Graphics Profiles here](https://docs.microsoft.com/en-us/previous-versions/windows/xna/ff604995(v=xnagamestudio.42)).
>
> If you only wish to target high-end systems, you can upgrade your implementation to use the "HiDef" profile and then update the Index/Vertex arrays to use **ints** instead of **shorts** and by filling it like this:
>
> ```csharp
>     private void SetUpIndices()
>     {
>         _indices = new int[(_terrainWidth - 1) * (_terrainHeight - 1) * 6];
>         int counter = 0;
>         for (int y = 0; y < _terrainHeight - 1; y++)
>         {
>             for (int x = 0; x < _terrainWidth - 1; x++)
>             {
>                 int lowerLeft = x + y * _terrainWidth;
>                 int lowerRight = (x + 1) + y * _terrainWidth;
>                 int topLeft = x + (y + 1) * _terrainWidth;
>                 int topRight = (x + 1) + (y + 1) * _terrainWidth;
>
>                 _indices[counter++] = topLeft;
>                 _indices[counter++] = lowerRight;
>                 _indices[counter++] = lowerLeft;
>
>                 _indices[counter++] = topLeft;
>                 _indices[counter++] = topRight;
>                 _indices[counter++] = lowerRight;
>             }
>         }
>     }
> ```
>
> Don't forget to also update the Index/Vertex Array data types, as well as the Width/Height variables too.

## Exercises

You can try these exercises to practice what you have learned:

* Play around with the values of your heightData array.
* Try to add an extra row to your grid.

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
        private Matrix _viewMatrix;
        private Matrix _projectionMatrix;
        private float _angle = 0f;
        private short[] _indices;
        private int _terrainWidth = 4;
        private int _terrainHeight = 3;
        private float[,] _heightData;

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
            _vertices = new VertexPositionColor[_terrainWidth * _terrainHeight];
            for (int x = 0; x < _terrainWidth; x++)
            {
                for (int y = 0; y < _terrainHeight; y++)
                {
                    _vertices[x + y * _terrainWidth].Position = new Vector3(x, _heightData[x, y], -y);
                    _vertices[x + y * _terrainWidth].Color = Color.White;
                }
            }
        }

        private void SetUpIndices()
        {
            _indices = new short[(_terrainWidth - 1) * (_terrainHeight - 1) * 6];
            int counter = 0;
            for (int y = 0; y < _terrainHeight - 1; y++)
            {
                for (int x = 0; x < _terrainWidth - 1; x++)
                {
                    short lowerLeft = (short)(x + y * _terrainWidth);
                    short lowerRight = (short)((x + 1) + y * _terrainWidth);
                    short topLeft = (short)(x + (y + 1) * _terrainWidth);
                    short topRight = (short)((x + 1) + (y + 1) * _terrainWidth);

                    _indices[counter++] = topLeft;
                    _indices[counter++] = lowerRight;
                    _indices[counter++] = lowerLeft;

                    _indices[counter++] = topLeft;
                    _indices[counter++] = topRight;
                    _indices[counter++] = lowerRight;
                }
            }
        }

        private void SetUpCamera()
        {
            _viewMatrix = Matrix.CreateLookAt(new Vector3(0, 10, 0), new Vector3(0, 0, 0), new Vector3(0, 0, -1));
            _projectionMatrix = Matrix.CreatePerspectiveFieldOfView(MathHelper.PiOver4, _device.Viewport.AspectRatio, 1.0f, 300.0f);
        }

        private void LoadHeightData()
        {
            _heightData = new float[4, 3];
            _heightData[0, 0] = 0;
            _heightData[1, 0] = 0;
            _heightData[2, 0] = 0;
            _heightData[3, 0] = 0;

            _heightData[0, 1] = 0.5f;
            _heightData[1, 1] = 0;
            _heightData[2, 1] = -1.0f;
            _heightData[3, 1] = 0.2f;

            _heightData[0, 2] = 1.0f;
            _heightData[1, 2] = 1.2f;
            _heightData[2, 2] = 0.8f;
            _heightData[3, 2] = 0;
        }

        protected override void LoadContent()
        {
            // TODO: use this.Content to load your game content here
            _device = _graphics.GraphicsDevice;

            _effect = Content.Load<Effect>("effects");

            SetUpCamera();

            LoadHeightData();
            SetUpVertices();
            SetUpIndices();
        }

        protected override void Update(GameTime gameTime)
        {
            if (GamePad.GetState(PlayerIndex.One).Buttons.Back == ButtonState.Pressed || Keyboard.GetState().IsKeyDown(Keys.Escape))
                Exit();

            // TODO: Add your update logic here
            _angle += 0.005f;

            base.Update(gameTime);
        }

        protected override void Draw(GameTime gameTime)
        {
            _device.Clear(Color.DarkSlateBlue);

            RasterizerState rs = new RasterizerState();
            rs.CullMode = CullMode.None;
            rs.FillMode = FillMode.WireFrame;
            _device.RasterizerState = rs;

            // TODO: Add your drawing code here
            _effect.CurrentTechnique = _effect.Techniques["ColoredNoShading"];
            _effect.Parameters["xView"].SetValue(_viewMatrix);
            _effect.Parameters["xProjection"].SetValue(_projectionMatrix);
            Matrix worldMatrix = Matrix.Identity;
            _effect.Parameters["xWorld"].SetValue(worldMatrix);

            foreach (EffectPass pass in _effect.CurrentTechnique.Passes)
            {
                pass.Apply();
                _device.DrawUserIndexedPrimitives(PrimitiveType.TriangleList, _vertices, 0, _vertices.Length, _indices, 0, _indices.Length / 3, VertexPositionColor.VertexDeclaration);
            }

            base.Draw(gameTime);
        }
    }
}
```

## Next Steps

[Terrain creation from file](Riemers3DXNA1Terrain08terrainfile)
