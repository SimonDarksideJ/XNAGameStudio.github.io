# Rotate your terrain using the keyboard

Using MonoGame, it is very easy to read in the current state of your keyboard (gamepads and Touchscreens are also just as easy). The correct libraries, *Microsoft.Xna.Framework.Input*, is already linked to by default when you created your MonoGame project, so we can immediately move on to the code that reads in the keyboard input.

Replace the line that updates your angle with the following piece of code in your **Update** method, which is called 60 times per second:

From:

```csharp
    _angle += 0.005f;
```

To:

```csharp
    KeyboardState keyState = Keyboard.GetState();
    if (keyState.IsKeyDown(Keys.Left))
    {
        _angle += 0.05f;
    }
    if (keyState.IsKeyDown(Keys.Right))
    {
        _angle -= 0.05f;
    }
```

Here you put the current state of the keyboard into a variable **'keyState'**. Using this variable, you can immediately read which keys are currently being pressed! When the user presses the Left or Right keys, the value of the ‘angle’ variable will be adjusted.

In our **Draw** method, we need to make sure that this rotation is also taken into account in our World matrix:

```csharp
    Matrix worldMatrix = Matrix.CreateTranslation(-_terrainWidth / 2.0f, 0, _terrainHeight / 2.0f) * Matrix.CreateRotationY(_angle);
```

The terrain will now rotate around the Y Up axis before it is moved to the (0,0,0) 3D origin.

Et voila! When you run the code, you can rotate your terrain simply by pressing the Delete and PageDown buttons!

![Summary](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA1-09Keyboard1.gif?raw=true)

## Exercises

You can try these exercises to practice what you have learned:

* Add some code that reads in the Up and Down Arrow keys, then adjust your world matrix so the terrain is moved into the direction of the Arrow key being pressed.

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
            _device.Clear(Color.Black);

            RasterizerState rs = new RasterizerState();
            rs.CullMode = CullMode.None;
            rs.FillMode = FillMode.WireFrame;
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

[Adding some color and the Z-Buffer](Riemers3DXNA1Terrain10colors)
