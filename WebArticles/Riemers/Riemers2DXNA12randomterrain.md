# Random terrain generation

The result of the last chapter does not meet the vision of rocky terrain as it is completely flat, as a first improvement, we are going to add slopes to the terrain. This doesn’t introduce any new MonoGame features but will show us how randomness can be added to your game to improve replayability.

## New terrain generator

Instead of a flat terrain, let us change our code so it generates a wave for terrain slope. Replace the **GenerateTerrainContour** method with the following code:

```csharp
    private void GenerateTerrainContour()
    {
        _terrainContour = new int[_screenWidth];

        float offset = _screenHeight / 2;
        float peakheight = 100;
        float flatness = 50;
        for (int x = 0; x < _screenWidth; x++)
        {
            double height = peakheight * Math.Sin((float)x / flatness) + offset;
            _terrainContour[x] = (int)height;
        }
    }
```

If you now run this code, you should see the terrain as shown in the image below. The Y coordinates are calculated using a [Sine function](https://en.wikipedia.org/wiki/Sine), which is the mathematical equivalent of a wave. There are a few parameters we can change to randomize this a bit further:

* The offset is the easiest one, as it simply sets the position of the midheight of the wave.
* The peakheight value is multiplied by the output of the Sine function, so it defines how high the wave will be.
* Finally, the flatness value has an impact on the influence of X, which slows down or speeds up the wave. This increases or decreases the wavelength of our wave.

![Terrain Height](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/2DXNA12Terrain1.jpg?raw=true)

A simple wave like this still looks artificial, this can be solved by mixing a second wave, tor example, we can add in a wave with a smaller height, and shorter wavelength. The result would look like this:

![Generated Terrain](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/2DXNA12Terrain2.jpg?raw=true)

## Adding waves to randomize terrain

In our code, we will also add in a third wave to improve the resolution of our random terrain. Also, to make the terrain look different each time, we are going to influence the 3 parameters of each wave with a random value. This is what our **GenerateTerrainContour** will finally look like:

```csharp
    private void GenerateTerrainContour()
    {
        _terrainContour = new int[_screenWidth];

        double rand1 = _randomizer.NextDouble() + 1;
        double rand2 = _randomizer.NextDouble() + 2;
        double rand3 = _randomizer.NextDouble() + 3;

        float offset = _screenHeight / 2;
        float peakheight = 100;
        float flatness = 70;

        for (int x = 0; x < _screenWidth; x++)
        {
            double height = peakheight / rand1 * Math.Sin((float)x / flatness * rand1 + rand1);
            height += peakheight / rand2 * Math.Sin((float)x / flatness * rand2 + rand2);
            height += peakheight / rand3 * Math.Sin((float)x / flatness * rand3 + rand3);
            height += offset;
            _terrainContour[x] = (int)height;
        }
    }
```

For each wave:

* We first draw a random value between 0 and 1 together with an offset making the first wave between the [0,1] range
* We add a second wave between the [1,2] range
* And finally, the last wave between the [2,3] range.

For our 3 waves will divide the peakheight and the flatness so that wave 3 will be lower and shorter than waves 1 and 2. Furthermore, you can see we are also adding random values inside the Sine method, otherwise all 3 waves would start exactly at the same Y coordinate specified by ‘offset’, making this point fixed each time we restart the game.

Now when you try to run this code, you will get a much nicer terrain slope, as shown in the final screenshot of this chapter.

## placing the players dynamically

One thing that immediately catches the eye when running this code, is that the players are not positioned on the terrain, this is simply because we are still using fixed positions for our players based on the original fixed foreground texture we used a few chapters ago.

To solve this, find the **SetUpPlayers** method and replace it with the following code, which we will use to calculate the player's new positions inside the for-loop of that method:

```csharp
    private void SetUpPlayers()
    {
        _players = new PlayerData[_numberOfPlayers];
        for (int i = 0; i < _numberOfPlayers; i++)
        {
            _players[i].IsAlive = true;
            _players[i].Color = _playerColors[i];
            _players[i].Angle = MathHelper.ToRadians(90);
            _players[i].Power = 100;
            _players[i].Position = new Vector2();
            _players[i].Position.X = _screenWidth / (_numberOfPlayers + 1) * (i + 1);
            _players[i].Position.Y = _terrainContour[(int)_players[i].Position.X];
        }
    }
```

The X coordinate of the player can easily be found by dividing the width of our screen by the number of players. The "+ 1"s are needed to make sure no player is positioned at the left or right border of the screen. The player's Y coordinate is found simply by looking up the terrain heigh at that X coordinate.

However, for this to work, the **SetUpPlayers** method must have access to the finalized **terrainContour** array, this means that in our **LoadContent** method, we need to call the **SetUpPlayers** method after we call our **GenerateTerrainContour** method, so move the call to "SetUpPlayers" as follows:

```csharp
    GenerateTerrainContour();
    SetUpPlayers();
    CreateForeground();
```

## making room for the player

When you now run this code, you will see the bottom-left point of the cannons are positioned on the terrain, but the terrain underneath them doesn’t match. To solve this, we will simply extend the terrain underneath the players by adding this simple method:

```csharp
    private void FlattenTerrainBelowPlayers()
    {
        foreach (PlayerData player in _players)
        {
            if (player.IsAlive)
            {
                for (int x = 0; x < 40; x++)
                {
                    _terrainContour[(int)player.Position.X + x] = _terrainContour[(int)player.Position.X];
                }
            }
        }
    }
```

For each active player, the terrain underneath the cannon is levelled to the same height as the bottom-left point. We need to call this method from within our **LoadContent** method but we need to keep some things in mind:

* The FlattenTerrainBelowPlayers must be called after the **GenerateTerrainContour** and **SetUpPlayers** methods, since it uses data defined in these methods.
* On the other hand, it must be called before the **CreateForeground** method is called, as that method uses the **terrainContour** variable, which is changed by the FlattenTerrainBelowPlayers method:

```csharp
    GenerateTerrainContour();
    SetUpPlayers();
    FlattenTerrainBelowPlayers();
    CreateForeground();
```

When you run the code at this moment, the slope of the terrain should look just fine.

![Summary](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/2DXNA12Terrain3.png?raw=true)

In the next chapter we will do something about the solid green color of our terrain.

## Exercises

You can try these exercises to practice what you have learned:

* Alter the waves logic to mix up the terrain more
* Try using [other mathematical waves](https://en.wikipedia.org/wiki/Trigonometric_functions#cos) such as cosine or tan to mix things up further, go wild

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
                _players[i].Position = new Vector2();
                _players[i].Position.X = _screenWidth / (_numberOfPlayers + 1) * (i + 1);
                _players[i].Position.Y = _terrainContour[(int)_players[i].Position.X];
            }
        }

        private void GenerateTerrainContour()
        {
            _terrainContour = new int[_screenWidth];

            double rand1 = _randomizer.NextDouble() + 1;
            double rand2 = _randomizer.NextDouble() + 2;
            double rand3 = _randomizer.NextDouble() + 3;

            float offset = _screenHeight / 2;
            float peakheight = 100;
            float flatness = 70;

            for (int x = 0; x < _screenWidth; x++)
            {
                double height = peakheight / rand1 * Math.Sin((float)x / flatness * rand1 + rand1);
                height += peakheight / rand2 * Math.Sin((float)x / flatness * rand2 + rand2);
                height += peakheight / rand3 * Math.Sin((float)x / flatness * rand3 + rand3);
                height += offset;
                _terrainContour[x] = (int)height;
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

        private void FlattenTerrainBelowPlayers()
        {
            foreach (PlayerData player in _players)
            {
                if (player.IsAlive)
                {
                    for (int x = 0; x < 40; x++)
                    {
                        _terrainContour[(int)player.Position.X + x] = _terrainContour[(int)player.Position.X];
                    }
                }
            }
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

            _playerScaling = 40.0f / (float)_carriageTexture.Width;

            GenerateTerrainContour();
            SetUpPlayers();
            FlattenTerrainBelowPlayers();
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

[Texture to an array of colors](Riemers2DXNA13texturetocolors)
