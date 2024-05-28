# Terrain creation from file

It is time to finally create a nice looking landscape. Instead of manually entering the data into our heightData array, we are going to fill it from a file. To do this, we are going to load a 128x128 pixel grayscale image, and we are going to use the 'white value' of every pixel as the height coordinate for each corresponding pixel!

## Preparing for the height data

Once have unpacked the "heightmap.bmp" from the asset package for this series and imported it into your content project, just as you did for the .fx file, we can begin using its magical data to build our terrain.

The image file should be loaded in the **LoadContent** method. The .fx file was loaded into an Effect variable, an image should be loaded into a **Texture2D** variable, this should replace the old "LoadHeightData" function we used earlier when we were manually creating our heightmap:

```csharp
    Texture2D heightMap = Content.Load<Texture2D>("heightmap");
    LoadHeightData(heightMap);
```

By using the Content Pipeline, it does not matter whether you are using a .bmp, .jpg, or .pgn file. The second line then calls the LoadHeightData, which we will adjust so it receives this texture. Update the LoadHeightData method as follows:

```csharp
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
```

As you can see, this method receives the image as an argument, and instead of using a predefined width and height for your terrain, we are now using the resolution of your image as the size of the terrain. The first 2 lines read the width and height of the image and stores them as width and height for the rest of our program, which will cause the rest of our code to automatically generate enough vertices and indices to render enough triangles.

Because you also want to access the color values of the pixels of the image, we create an array of Color objects into which we store the color of each pixel from the image. This is done in 2 easy lines.

To finish, we reshape the 1D array of Colors into a 2D array of floats, first by creating a 2D matrix capable of storing just enough floats, next, you select the Red value (which is a value between 0 (no red) and 255 (fully red)) before storing this value inside the 2D array. You scale it a bit down, otherwise your terrain would be too steep.

> You may note we are only using the Red pixel for the height and ignoring the rest. This is a useful trick when you need to store multiple types of float data in an image, using each color channel to store the different data.  Not all images are for just looking at.

At this point, you have a 2D array containing the height for each point of your terrain, we then need to call the **SetUpVertices** method to generate the vertexes for each point of the array. The **SetUpIndices** method will then generate three indices for each triangle that needs to be drawn to completely cover your terrain. Finally, your Draw method will render many triangles as your indices array allows. So when you run this code, you should see your terrain!

## Has the island sunk again?

Bad luck, again. The solution is simple, again. The corner of the terrain is positioned above your camera! So if you increase the height of your camera to **40**, you should see your terrain.

However, when you run your program, you will only see one corner of a huge terrain, so you will need to move your terrain so that its center is shifted to the (0,0,0) 3D origin point. This can be done in the Draw method:

```csharp
    Matrix worldMatrix = Matrix.CreateTranslation(-_terrainWidth / 2.0f, 0, _terrainHeight / 2.0f);
    _effect.CurrentTechnique = _effect.Techniques["ColoredNoShading"];
    _effect.Parameters["xView"].SetValue(_viewMatrix);
    _effect.Parameters["xProjection"].SetValue(_projectionMatrix);
    _effect.Parameters["xWorld"].SetValue(worldMatrix);
```

This will bring your terrain to the center of your screen.

As the camera is still looking straight on the terrain, we will get a nicer look by repositioning the camera a bit in the **SetUpCamera** method as follows:

```csharp
    _viewMatrix = Matrix.CreateLookAt(new Vector3(60, 80, -80), new Vector3(0, 0, 0), new Vector3(0, 1, 0));
```

> If you set your graphics clearing color to Color.Black, you should get a clearer look at the terrain mesh.

![Summary](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA1-08TerrainFile1.png?raw=true)

## Exercises

You can try these exercises to practice what you have learned:

* This code scales the Red color value down by five. Try out some other scaling values.
* Open the heightmap.bmp image in Paint, and adjust some pixel values. Make some pixels pure red, others pure blue. Load this into your program.
* Find another image file and load it into your program. Look for other images with a small resolution, as this approach will not allow you to render **huge** terrains.

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
            _angle += 0.005f;

            base.Update(gameTime);
        }

        protected override void Draw(GameTime gameTime)
        {
            _device.Clear(Color.Black);

            RasterizerState rs = new RasterizerState();
            rs.CullMode = CullMode.None;
            rs.FillMode = FillMode.WireFrame;
            _device.RasterizerState = rs;

            // TODO: Add your drawing code here
            Matrix worldMatrix = Matrix.CreateTranslation(-_terrainWidth / 2.0f, 0, _terrainHeight / 2.0f);
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

[Rotate your terrain using the keyboard](Riemers3DXNA1Terrain09keyboard)
