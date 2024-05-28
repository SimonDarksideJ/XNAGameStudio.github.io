# Adding some color and the Z-Buffer

You might already have a rotating terrain, but it definitely would be better looking filled with some colors instead of just plain white lines. One idea to do this is to use natural colors, like the ones that we find in the mountains, at the bottom we have blue lakes, green trees, brown mountains, and finally snow-topped peaks. This means we will have to extend our **SetUpVertices** method a bit, so it stores the correct colors in each vertex.

## Coloring the map

You cannot expect every image to have a lake at height 0 and a mountain peak at height 255 (the maximum value for a .bmp pixel). Most images height values are only between 50 and 200, this image would then produce a terrain without any lakes or peaks.

To remain as general as possible, we first have to detect the minimum and maximum heights in our image and store these values in the minHeight and maxHeight variables. We will put this code at the top of our **SetUpVertices** method:

```csharp
    float minHeight = float.MaxValue;
    float maxHeight = float.MinValue;
    for (int x = 0; x < _terrainWidth; x++)
    {
        for (int y = 0; y < _terrainHeight; y++)
        {
            if (_heightData[x, y] < minHeight)
                minHeight = _heightData[x, y];
            if (_heightData[x, y] > maxHeight)
                maxHeight = _heightData[x, y];
        }
    }
```

We check for every point of our grid if the current point’s height is below the current minHeight or above the current maxHeight, if it is, store the current height in the corresponding min or max variable.

With these variables filled, you can specify the 4 regions of your colors:

![Color by Height](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA1-10TerrainColor1.jpg?raw=true)

Now when you declare your vertices and their colors, you are going to define the desired colors to the specified height regions, replacing the following in the **SetUpVertices** method:

```csharp
    _vertices[x + y * _terrainWidth].Color = Color.White;
```

With the following:

```csharp
    _vertices[x + y * _terrainWidth].Position = new Vector3(x, _heightData[x, y], -y);

    if (_heightData[x, y] < minHeight + (maxHeight - minHeight) / 4)
    {
        _vertices[x + y * _terrainWidth].Color = Color.Blue;
    }
    else if (_heightData[x, y] < minHeight + (maxHeight - minHeight) * 2 / 4)
    {
        _vertices[x + y * _terrainWidth].Color = Color.Green;
    }
    else if (_heightData[x, y] < minHeight + (maxHeight - minHeight) * 3 / 4)
    {
        _vertices[x + y * _terrainWidth].Color = Color.Brown;
    }
    else
    {
        _vertices[x + y * _terrainWidth].Color = Color.White;
    }
```

Resulting in the following:

![Colored Wireframe](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA1-10Colors2.png?raw=true)

When you run this code, you will indeed see a nicely colored network of lines. When we want to see the whole colored terrain, we just have to remove this line (or set it to FillMode.Solid) in the **Draw** method:

```csharp
    rs.FillMode = FillMode.WireFrame;
```

## Dealing with the ZBuffer

When you execute this, take a few moments to rotate the terrain a couple of times. On some computers, you will see that sometimes the middle peaks get overdrawn by the ‘invisible’ lake behind it. This is because we have not yet defined a ‘Z-buffer’! This Z buffer is nothing more than an array where your video card keeps track of the depth coordinate of every pixel that should be drawn on your screen (so in our case, a 500x500 matrix!). Every time your card receives a triangle to draw, it checks whether the triangle’s pixels are closer to the screen than the pixels already present in the Z-buffer. If they are closer, the Z-buffer’s contents update with the pixels for that region.

Of course, this whole process if fully automated. All we have to do is to initialize our Z buffer with the largest possible distance to start with. So in fact, we have to first fill our buffer with ones. To do this automatically every update of our screen, change this line in the Draw method:

```csharp
    _device.Clear(ClearOptions.Target|ClearOptions.DepthBuffer, Color.Black, 1.0f, 0);
```

> The | is a bitwise OR operator, in this case, it means both the Target (the colors) as well as the DepthBuffer have to be cleared

Now everyone should see the terrain rotating as expected.

![Summary](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA1-10Colors3.gif?raw=true)

Although now we have got some colors in our terrain, it doesn’t really look any better than it did last chapter.. (although I did prefer the wireframe version :P)

## Exercises

You can try these exercises to practice what you have learned:

* Lighting is required to obtain a 3D effect. In the image above, you get a little 3D feeling because you're using multiple colors. Change the colors again back to one single color, and see that the 3D feeling is completely gone!

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
            float minHeight = float.MaxValue;
            float maxHeight = float.MinValue;
            for (int x = 0; x < _terrainWidth; x++)
            {
                for (int y = 0; y < _terrainHeight; y++)
                {
                    if (_heightData[x, y] < minHeight)
                        minHeight = _heightData[x, y];
                    if (_heightData[x, y] > maxHeight)
                        maxHeight = _heightData[x, y];
                }
            }

            _vertices = new VertexPositionColor[_terrainWidth * _terrainHeight];
            for (int x = 0; x < _terrainWidth; x++)
            {
                for (int y = 0; y < _terrainHeight; y++)
                {
                    _vertices[x + y * _terrainWidth].Position = new Vector3(x, _heightData[x, y], -y);
                    _vertices[x + y * _terrainWidth].Position = new Vector3(x, _heightData[x, y], -y);

                    if (_heightData[x, y] < minHeight + (maxHeight - minHeight) / 4)
                    {
                        _vertices[x + y * _terrainWidth].Color = Color.Blue;
                    }
                    else if (_heightData[x, y] < minHeight + (maxHeight - minHeight) * 2 / 4)
                    {
                        _vertices[x + y * _terrainWidth].Color = Color.Green;
                    }
                    else if (_heightData[x, y] < minHeight + (maxHeight - minHeight) * 3 / 4)
                    {
                        _vertices[x + y * _terrainWidth].Color = Color.Brown;
                    }
                    else
                    {
                        _vertices[x + y * _terrainWidth].Color = Color.White;
                    }
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
            _viewMatrix = Matrix.CreateLookAt(new Vector3(60, 80, -80), new Vector3(0, 0, 0), new Vector3(0, 1, 0));
            _projectionMatrix = Matrix.CreatePerspectiveFieldOfView(MathHelper.PiOver4, _device.Viewport.AspectRatio, 1.0f, 300.0f);
        }

        private void LoadHeightData(Texture2D heightMap)
        {
            _terrainWidth = heightMap.Width;
            _terrainHeight = heightMap.Height;

            Color[] heightMapColors = new Color[_terrainWidth * _terrainHeight];
            heightMap.GetData(heightMapColors);

            _heightData = new float[_terrainWidth, _terrainHeight];
            for (int x = 0; x < _terrainWidth; x++)
            {
                for (int y = 0; y < _terrainHeight; y++)
                {
                    _heightData[x, y] = heightMapColors[x + y * _terrainWidth].R / 5.0f;
                }
            }
        }

        protected override void LoadContent()
        {
            // TODO: use this.Content to load your game content here
            _device = _graphics.GraphicsDevice;

            _effect = Content.Load<Effect>("effects");

            SetUpCamera();

            Texture2D heightMap = Content.Load<Texture2D>("heightmap");
            LoadHeightData(heightMap);
            SetUpVertices();
            SetUpIndices();
        }

        protected override void Update(GameTime gameTime)
        {
            if (GamePad.GetState(PlayerIndex.One).Buttons.Back == ButtonState.Pressed || Keyboard.GetState().IsKeyDown(Keys.Escape))
                Exit();

            // TODO: Add your update logic here
            KeyboardState keyState = Keyboard.GetState();
            if (keyState.IsKeyDown(Keys.Left))
            {
                _angle += 0.05f;
            }
            if (keyState.IsKeyDown(Keys.Right))
            {
                _angle -= 0.05f;
            }

            base.Update(gameTime);
        }

        protected override void Draw(GameTime gameTime)
        {
            _device.Clear(ClearOptions.Target | ClearOptions.DepthBuffer, Color.Black, 1.0f, 0);

            RasterizerState rs = new RasterizerState();
            rs.CullMode = CullMode.None;
            rs.FillMode = FillMode.Solid;
            _device.RasterizerState = rs;

            // TODO: Add your drawing code here
            Matrix worldMatrix = Matrix.CreateTranslation(-_terrainWidth / 2.0f, 0, _terrainHeight / 2.0f) * Matrix.CreateRotationY(_angle);
            _effect.CurrentTechnique = _effect.Techniques["ColoredNoShading"];
            _effect.Parameters["xView"].SetValue(_viewMatrix);
            _effect.Parameters["xProjection"].SetValue(_projectionMatrix);
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

[Experimenting with Lights in XNA](Riemers3DXNA1Terrain11lighting)
