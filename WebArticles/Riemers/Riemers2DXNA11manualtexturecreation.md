# Defining the colors of a texture per-pixel

At this moment, we have our rockets flying around, but they are not yet colliding with the terrain.

## Generating terrain

Before we can handle whether the rocket has collided with the terrain, we need to know the exact coordinates of our terrain, in this game, this is rather simple, for each X coordinate of our screen the terrain has a certain Y coordinate.

One terribly wrong approach would be to use the 500 Y coordinates for the foreground texture we’re using at the moment. A much better approach is to define the terrain slope ourselves, and from this slope, we will create the foreground texture, which we will do here.

We will implement this using 2 methods:

* GenerateTerrainContour which generates the slope, and
* CreateForeground which creates the foreground texture based on the terrain slope.

We’ll start with easy methods first to get the mechanism up and working, then in the next chapter, we will refine both methods to get a nicer result.

The result of the GenerateTerrainContour will be a simple array of *ints*, storing one Y coordinate for each X coordinate of our screen. We will need this quite often in our code later on, so let us add it to the variables in the Properties section of our code:

```csharp
    private int[] _terrainContour;
```

Next in line is the **GenerateTerrainContour** method, which initializes this array and fills it with values. As was said earlier, we will start with a very basic version:

```csharp
    private void GenerateTerrainContour()
    {
        _terrainContour = new int[_screenWidth];

        for (int x = 0; x < _screenWidth; x++)
        {
            _terrainContour[x] = _screenHeight / 2;
        }
    }
```

The **terrainContour** will need to contain as many values as there are X coordinates, which can be found by using the screenWidth variable defined earlier. Next, for each X coordinate, we store the same Y coordinate. Later on, this will result in a horizontally flat terrain, in the middle of our screen.

Moving on to the **CreateForeground** method, this method should create the foreground texture, based on the contents of the **terrainContour** array:

```csharp
    private void CreateForeground()
    {
        Color[] foregroundColors = new Color[_screenWidth * _screenHeight];

        for (int x = 0; x < _screenWidth; x++)
        {
            for (int y = 0; y < _screenHeight; y++)
            {
                if (y > _terrainContour[x])
                {
                    foregroundColors[x + y * _screenWidth] = Color.Green;
                }
                else
                {
                    foregroundColors[x + y * _screenWidth] = Color.Transparent;
                }
            }
        }
    }
```

1. The first line creates an array, capable of storing Colors. We initialize it so it can store one color for each pixel on our screen.
2. Next, we scroll through each combination of X and Y coordinates, covering each pixel of our screen. For each pixel, we check whether it is above or below the terrain slope. If it is below the terrain (this means the Y coordinate is larger than the Y coordinate stored in the terrainContour array), we store the green color in our foregroundColors array. If the pixel is above the terrain, we store Transparant as color, meaning that the underlying color (our background image) will remain visible at those pixels.
3. At the end of the method, we have an array of colors, but we still need to create a texture out of them.

## Creating a texture manually

Creating new textures in MonoGame is fairly easy, all you need to do is put these 2 lines at the end of our **CreateForeground** method:

```csharp
    _foregroundTexture = new Texture2D(_device, _screenWidth, _screenHeight, false, SurfaceFormat.Color);
    _foregroundTexture.SetData(foregroundColors);
```

The first line creates an empty texture, reserving an area of memory for the graphics card, exactly enough to store one Color for each pixel of our screen.
We pass the graphic device to create the texture for and then in the second and third arguments define how many pixels can be stored in the texture.  The last argument indicates we want to store Color values for each pixel. The third argument ([mipmapping](https://en.wikipedia.org/wiki/Mipmap)) is out of the scope of this tutorial but feel free to research on your own.

The last line copies the color data from the **foregroundcolors** array into the memory on the graphics card.

Finally, as we are no longer using the foreground texture from our content project, go to the **LoadContent** method and remove the following line:

```csharp
    _foregroundTexture = Content.Load<Texture2D>("foreground");
```

> Feel free to also remove the "foreground.png" texture from the Content Project itself too.

And then add these lines to the very bottom of the LoadContent method:

```csharp
    GenerateTerrainContour();
    CreateForeground();
```

Now run this code! You should see the screen shown below. (If it crashes, it’s because you didn’t put the previous lines at the very end of your LoadContent method)

![Summary](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/2DXNA11ManualTexture1.png?raw=true)

This is the same as before, only with the foreground gone and the bottom half fully green. This means, that we have successfully created a texture, of which we have defined the color of each pixel.

In the next chapter, we will make some changes to the slope of the terrain and to the colors of our foreground texture to get rid of the solid green color.

## Exercises

You can try these exercises to practice what you have learned:

* No exercises this time, but get ready for the next section!

## The code thus far

```csharp
using System;
using System.Collections.Generic;
using Microsoft.Xna.Framework;
using Microsoft.Xna.Framework.Graphics;
using Microsoft.Xna.Framework.Input;

namespace Series2D1
{
    public struct PlayerData
    {
        public Vector2 Position;
        public bool IsAlive;
        public Color Color;
        public float Angle;
        public float Power;
    }

    public class Game1 : Game
    {
        //Properties
        private GraphicsDeviceManager _graphics;
        private SpriteBatch _spriteBatch;
        private GraphicsDevice _device;
        private Texture2D _backgroundTexture;
        private Texture2D _foregroundTexture;
        private Texture2D _carriageTexture;
        private Texture2D _cannonTexture;
        private Texture2D _rocketTexture;
        private Texture2D _smokeTexture;
        private SpriteFont _font;
        private int _screenWidth;
        private int _screenHeight;
        private PlayerData[] _players;
        private int _numberOfPlayers = 4;
        private float _playerScaling;
        private int _currentPlayer = 0;
        private bool _rocketFlying = false;
        private Vector2 _rocketPosition;
        private Vector2 _rocketDirection;
        private float _rocketAngle;
        private float _rocketScaling = 0.1f;
        private Color[] _playerColors = new Color[10]
        {
            Color.Red,
            Color.Green,
            Color.Blue,
            Color.Purple,
            Color.Orange,
            Color.Indigo,
            Color.Yellow,
            Color.SaddleBrown,
            Color.Tomato,
            Color.Turquoise
        };
        private List<Vector2> _smokeList = new List<Vector2>();
        private Random _randomizer = new Random();
        private int[] _terrainContour;

        public Game1()
        {
            _graphics = new GraphicsDeviceManager(this);
            Content.RootDirectory = "Content";
        }

        protected override void Initialize()
        {
            // TODO: Add your initialization logic here
            _graphics.PreferredBackBufferWidth = 500;
            _graphics.PreferredBackBufferHeight = 500;
            _graphics.IsFullScreen = false;
            _graphics.ApplyChanges();
            Window.Title = "Riemer's 2D MonoGame Tutorial";

            base.Initialize();
        }

        private void SetUpPlayers()
        {
            _players = new PlayerData[_numberOfPlayers];
            for (int i = 0; i < _numberOfPlayers; i++)
            {
                _players[i].IsAlive = true;
                _players[i].Color = _playerColors[i];
                _players[i].Angle = MathHelper.ToRadians(90);
                _players[i].Power = 100;
            }

            _players[0].Position = new Vector2(100, 193);
            _players[1].Position = new Vector2(200, 212);
            _players[2].Position = new Vector2(300, 361);
            _players[3].Position = new Vector2(400, 164);
        }

        private void GenerateTerrainContour()
        {
            _terrainContour = new int[_screenWidth];

            for (int x = 0; x < _screenWidth; x++)
            {
                _terrainContour[x] = _screenHeight / 2;
            }
        }

        private void CreateForeground()
        {
            Color[] foregroundColors = new Color[_screenWidth * _screenHeight];

            for (int x = 0; x < _screenWidth; x++)
            {
                for (int y = 0; y < _screenHeight; y++)
                {
                    if (y > _terrainContour[x])
                    {
                        foregroundColors[x + y * _screenWidth] = Color.Green;
                    }
                    else
                    {
                        foregroundColors[x + y * _screenWidth] = Color.Transparent;
                    }
                }
            }

            _foregroundTexture = new Texture2D(_device, _screenWidth, _screenHeight, false, SurfaceFormat.Color);
            _foregroundTexture.SetData(foregroundColors);
        }

        protected override void LoadContent()
        {
            _spriteBatch = new SpriteBatch(GraphicsDevice);
            _device = _graphics.GraphicsDevice;

            // TODO: use this.Content to load your game content here
            _backgroundTexture = Content.Load<Texture2D>("background");
            _carriageTexture = Content.Load<Texture2D>("carriage");
            _cannonTexture = Content.Load<Texture2D>("cannon");
            _rocketTexture = Content.Load<Texture2D>("rocket");
            _smokeTexture = Content.Load<Texture2D>("smoke");
            _font = Content.Load<SpriteFont>("myFont");

            _screenWidth = _device.PresentationParameters.BackBufferWidth;
            _screenHeight = _device.PresentationParameters.BackBufferHeight;

            SetUpPlayers();

            _playerScaling = 40.0f / (float)_carriageTexture.Width;

            GenerateTerrainContour();
            CreateForeground();
        }

        private void ProcessKeyboard()
        {
            KeyboardState keybState = Keyboard.GetState();

            if (keybState.IsKeyDown(Keys.Left))
            {
                _players[_currentPlayer].Angle -= 0.01f;
            }
            if (keybState.IsKeyDown(Keys.Right))
            {
                _players[_currentPlayer].Angle += 0.01f;
            }

            if (_players[_currentPlayer].Angle > MathHelper.PiOver2)
            {
                _players[_currentPlayer].Angle = -MathHelper.PiOver2;
            }
            if (_players[_currentPlayer].Angle < -MathHelper.PiOver2)
            {
                _players[_currentPlayer].Angle = MathHelper.PiOver2;
            }

            if (keybState.IsKeyDown(Keys.Down))
            {
                _players[_currentPlayer].Power -= 1;
            }
            if (keybState.IsKeyDown(Keys.Up))
            {
                _players[_currentPlayer].Power += 1;
            }
            if (keybState.IsKeyDown(Keys.PageDown))
            {
                _players[_currentPlayer].Power -= 20;
            }
            if (keybState.IsKeyDown(Keys.PageUp))
            {
                _players[_currentPlayer].Power += 20;
            }

            if (_players[_currentPlayer].Power > 1000)
            {
                _players[_currentPlayer].Power = 1000;
            }
            if (_players[_currentPlayer].Power < 0)
            {
                _players[_currentPlayer].Power = 0;
            }

            if (keybState.IsKeyDown(Keys.Enter) || keybState.IsKeyDown(Keys.Space))
            {
                _rocketFlying = true;
                _rocketPosition = _players[_currentPlayer].Position;
                _rocketPosition.X += 20;
                _rocketPosition.Y -= 10;
                _rocketAngle = _players[_currentPlayer].Angle;
                Vector2 up = new Vector2(0, -1);
                Matrix rotMatrix = Matrix.CreateRotationZ(_rocketAngle);
                _rocketDirection = Vector2.Transform(up, rotMatrix);
                _rocketDirection *= _players[_currentPlayer].Power / 50.0f;
            }
        }

        private void UpdateRocket()
        {
            if (_rocketFlying)
            {
                Vector2 gravity = new Vector2(0, 1);
                _rocketDirection += gravity / 10.0f;
                _rocketPosition += _rocketDirection;
                _rocketAngle = (float)Math.Atan2(_rocketDirection.X, -_rocketDirection.Y);

                for (int i = 0; i < 5; i++)
                {
                    Vector2 smokePos = _rocketPosition;
                    smokePos.X += _randomizer.Next(10) - 5;
                    smokePos.Y += _randomizer.Next(10) - 5;
                    _smokeList.Add(smokePos);
                }
            }
        }

        protected override void Update(GameTime gameTime)
        {
            if (GamePad.GetState(PlayerIndex.One).Buttons.Back == ButtonState.Pressed ||
                Keyboard.GetState().IsKeyDown(Keys.Escape))
            {
                Exit();
            }

            // TODO: Add your update logic here

            ProcessKeyboard();

            UpdateRocket();

            base.Update(gameTime);
        }

        protected override void Draw(GameTime gameTime)
        {
            GraphicsDevice.Clear(Color.CornflowerBlue);

            // TODO: Add your drawing code here

            _spriteBatch.Begin();
            DrawScenery();
            DrawPlayers();
            DrawText();
            DrawRocket();
            DrawSmoke();
            _spriteBatch.End();

            base.Draw(gameTime);
        }

        private void DrawScenery()
        {
            Rectangle screenRectangle = new Rectangle(0, 0, _screenWidth, _screenHeight);
            _spriteBatch.Draw(_backgroundTexture, screenRectangle, Color.White);
            _spriteBatch.Draw(_foregroundTexture, screenRectangle, Color.White);
        }

        private void DrawPlayers()
        {
            for (int i = 0; i < _players.Length; i++)
            {
                if (_players[i].IsAlive)
                {
                    int xPos = (int)_players[i].Position.X;
                    int yPos = (int)_players[i].Position.Y;
                    Vector2 cannonOrigin = new Vector2(11, 50);

                    _spriteBatch.Draw(_carriageTexture, _players[i].Position, null, _players[i].Color, 0, new Vector2(0, _carriageTexture.Height), _playerScaling, SpriteEffects.None, 0);
                    _spriteBatch.Draw(_cannonTexture, new Vector2(xPos + 20, yPos - 10), null, _players[i].Color, _players[i].Angle, cannonOrigin, _playerScaling, SpriteEffects.None, 1);
                }
            }
        }

        private void DrawText()
        {
            PlayerData player = _players[_currentPlayer];
            int currentAngle = (int)MathHelper.ToDegrees(player.Angle);
            _spriteBatch.DrawString(_font, "Cannon angle: " + currentAngle.ToString(), new Vector2(20, 20), player.Color);
            _spriteBatch.DrawString(_font, "Cannon power: " + player.Power.ToString(), new Vector2(20, 45), player.Color);
        }

        private void DrawRocket()
        {
            if (_rocketFlying)
            {
                _spriteBatch.Draw(_rocketTexture, _rocketPosition, null, _players[_currentPlayer].Color, _rocketAngle, new Vector2(42, 240), _rocketScaling, SpriteEffects.None, 1);
            }
        }

        private void DrawSmoke()
        {
            for (int i = 0; i < _smokeList.Count; i++)
            {
                _spriteBatch.Draw(_smokeTexture, _smokeList[i], null, Color.White, 0, new Vector2(40, 35), 0.2f, SpriteEffects.None, 1);
            }
        }
    }
}
```

## Next Steps

[Random terrain generation](Riemers2DXNA12randomterrain)
