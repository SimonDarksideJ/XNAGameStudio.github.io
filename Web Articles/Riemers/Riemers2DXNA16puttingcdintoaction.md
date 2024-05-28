# Putting Collision Detection into practice

For this chapter, we will put all we have learned in the past 2 chapters together and end up with a fully working per-pixel collision detection system. We will create 2 methods, which will detect collision between our rocket and the terrain and players and another that detects whether the rocket is still inside the screen.

## Checking collision with the terrain

Given the past 2 chapters, it is fairly easy to detect for collisions between the rocket and the terrain, so let us begin and add the following method:

```csharp
    private Vector2 CheckTerrainCollision()
    {
        Matrix rocketMat = Matrix.CreateTranslation(-42, -240, 0) *
                           Matrix.CreateRotationZ(_rocketAngle) *
                           Matrix.CreateScale(_rocketScaling) *
                           Matrix.CreateTranslation(_rocketPosition.X, _rocketPosition.Y, 0);
        Matrix terrainMat = Matrix.Identity;
        Vector2 terrainCollisionPoint = TexturesCollide(_rocketColorArray, rocketMat, _foregroundColorArray, terrainMat);
        return terrainCollisionPoint;
    }
```

In this method, we create the transformation matrices for our rocket and foreground textures and pass them together with their color arrays to the TexturesCollide method. This method will then return (-1,-1) if no collision was detected, otherwise it will pass the screen coordinate of the collision. The result of this method will be returned by the **CheckTerrainCollision** method above.

## Checking collision with the players

Next in line is the **CheckPlayersCollision** method, which promises to be a little more complex as there are multiple players and some of them might no longer be alive. Start by adding the following code:

```csharp
    private Vector2 CheckPlayersCollision()
    {
        Matrix rocketMat = Matrix.CreateTranslation(-42, -240, 0) *
                            Matrix.CreateRotationZ(_rocketAngle) *
                            Matrix.CreateScale(_rocketScaling) *
                            Matrix.CreateTranslation(_rocketPosition.X, _rocketPosition.Y, 0);

        for (int i = 0; i < _numberOfPlayers; i++)
        {
            PlayerData player = _players[i];
            if (player.IsAlive)
            {
                if (i != _currentPlayer)
                {
                    int xPos = (int)player.Position.X;
                    int yPos = (int)player.Position.Y;

                    Matrix carriageMat = Matrix.CreateTranslation(0, -_carriageTexture.Height, 0) *
                                            Matrix.CreateScale(_playerScaling) *
                                            Matrix.CreateTranslation(xPos, yPos, 0);
                    Vector2 carriageCollisionPoint = TexturesCollide(_carriageColorArray, carriageMat, _rocketColorArray, rocketMat);
                }
            }
        }
        return new Vector2(-1, -1);
    }
```

1. First, the matrix of the rocket is created as this remains the same for all players.
2. Next, for each player, we check whether the player is alive and if it is not the player that shot the rocket, as otherwise there would be a collision the very moment the rocket was shot.
3. If all of this is true, we create the matrix for the carriage of the current player, as explained in the previous chapter.

The matrices of the rocket and carriage are passed to the **TexturesCollide** method and the result is stored in the **carriageCollisionPoint** Vector2 variable.

Remember that **carriageCollisionPoint** contains **(-1,-1)** if no collision was detected, which is what we check for in the next piece of code which you should put immediately after the **carriageCollisionPoint** variable:

```csharp
    if (carriageCollisionPoint.X > -1)
    {
        _players[i].IsAlive = false;
        return carriageCollisionPoint;
    }
```

If a collision between the rocket and the current carriage is detected (values greater than -1), the **IsAlive** property of the colliding player is set to **false** (they died) and the method returns the collision point, which immediately terminates the method.

If no collision was detected, we should then check for collisions between the rocket and the cannon of the current player. This is done exactly the same way, only this time we create the matrix for the cannon, instead of for the carriage: (put this code after the previous lines)

```csharp
    Matrix cannonMat = Matrix.CreateTranslation(-11, -50, 0) *
                        Matrix.CreateRotationZ(player.Angle) *
                        Matrix.CreateScale(_playerScaling) *
                        Matrix.CreateTranslation(xPos + 20, yPos - 10, 0);

    Vector2 cannonCollisionPoint = TexturesCollide(_cannonColorArray, cannonMat, _rocketColorArray, rocketMat);
    if (cannonCollisionPoint.X > -1)
    {
        _players[i].IsAlive = false;
        return cannonCollisionPoint;
    }
```

Again, if a collision between the rocket and the cannon is detected, the screen position of the collision is returned. If no collision was detected the code continues and the next iteration of the for loop these checks will be done for the next player. In case no collision was found between the rocket and any player, the for loop ends and the CheckPlayersCollision returns (-1,-1).

## Checking for objects going off-screen

Finally, we need a new method to check whether the rocket is still inside the window. This shouldn’t be that difficult:

```csharp
    private bool CheckOutOfScreen()
    {
        bool rocketOutOfScreen = _rocketPosition.Y > _screenHeight;
        rocketOutOfScreen |= _rocketPosition.X < 0;
        rocketOutOfScreen |= _rocketPosition.X > _screenWidth;

        return rocketOutOfScreen;
    }
```

This checks whether:

* The rocket is below the lower boundary OR
* to the left of our window OR
* to the right of our window.

The result is true if any of them is valid and is returned to the calling code.

At this moment, we have created 3 methods that allow us to detect any possible collision. All we need now is a general method that processes the results:

```csharp
    private void CheckCollisions(GameTime gameTime)
    {
        Vector2 terrainCollisionPoint = CheckTerrainCollision();
        Vector2 playerCollisionPoint = CheckPlayersCollision();
        bool rocketOutOfScreen = CheckOutOfScreen();

        if (playerCollisionPoint.X > -1)
        {
            _rocketFlying = false;

            _smokeList = new List<Vector2>();
            NextPlayer();
        }

        if (terrainCollisionPoint.X > -1)
        {
            _rocketFlying = false;

            _smokeList = new List<Vector2>();
            NextPlayer();
        }

        if (rocketOutOfScreen)
        {
            _rocketFlying = false;

            _smokeList = new List<Vector2>();
            NextPlayer();
        }
    }
```

This method calls the three methods and stores their results in three variables. The three if-blocks check whether any of them returned a collision. If this is the case, the rocket will no longer be drawn, the smokelist is reset and the next player is activated. 

> At this moment, the three if-block do exactly the same, but I keep them separated as in later chapters the different collisions will lead to different explosions.

## Some final additions

The last two things we need to do is to make sure to call the new **CheckCollisions** function from our **Update** method as follows:

```csharp
    if (_rocketFlying)
    {
        UpdateRocket();
        CheckCollisions(gameTime);
    }
```

And to add the NextPlayer method so that we can increment the currentPlayer value, and check whether the new player is still alive:

```csharp
    private void NextPlayer()
    {
        _currentPlayer = _currentPlayer + 1;
        _currentPlayer = _currentPlayer % _numberOfPlayers;
        while (!_players[_currentPlayer].IsAlive)
        {
            _currentPlayer = ++_currentPlayer % _numberOfPlayers;
        }
    }
```

First, the currentPlayer value is incremented and since this must not be larger than numberOfPlayers we take the modulus.

> As an example, if numberOfPlayers = 4, when 3 is incremented to 4 this will be reset to 0.

Next, we check whether the new player is alive and if it is not, you increment it again. The line inside the while loop does actually exactly the same as the first two lines together using the ++ before currentPlayer makes sure the value is incremented BEFORE the line is evaluated.

When this method returns, the currentPlayer will hold the next player that is still alive.

Now, when all if this is added to your project in the correct way, your code should be able to detect any collision between the rocket and any objects on its way!

![Summary](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/2DXNA14Collisions1.png?raw=true)

> A quick note on the performance of this method: a lot of calculations and checks need to be done for each pixel of image 1. This means, that when checking for collisions between 2 images, you should pass the smallest as tex1 and the largest as tex2. Furthermore, instead of blindly performing this detailed check between all of your images, you should first check whether the images possible overlap by checking whether their outlines collide. If their outlines don’t collide, you’re sure there is no collision so it’s useless to call the calculation-intensive TexturesCollide method on them.

## Exercises

You can try these exercises to practice what you have learned:

* No exercises this time, but get ready for the next section!

## The code thus far

If you’re unsure about where some part of the code should be placed, have a look at the code below:

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
        private Texture2D _groundTexture;
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
        private Color[,] _rocketColorArray;
        private Color[,] _foregroundColorArray;
        private Color[,] _carriageColorArray;
        private Color[,] _cannonColorArray;

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
            Color[,] groundColors = TextureTo2DArray(_groundTexture);
            Color[] foregroundColors = new Color[_screenWidth * _screenHeight];

            for (int x = 0; x < _screenWidth; x++)
            {
                for (int y = 0; y < _screenHeight; y++)
                {
                    if (y > _terrainContour[x])
                    {
                        foregroundColors[x + y * _screenWidth] = groundColors[x % _groundTexture.Width, y % _groundTexture.Height];
                    }
                    else
                    {
                        foregroundColors[x + y * _screenWidth] = Color.Transparent;
                    }
                }
            }

            _foregroundTexture = new Texture2D(_device, _screenWidth, _screenHeight, false, SurfaceFormat.Color);
            _foregroundTexture.SetData(foregroundColors);

            _foregroundColorArray = TextureTo2DArray(_foregroundTexture);
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

        private Color[,] TextureTo2DArray(Texture2D texture)
        {
            Color[] colors1D = new Color[texture.Width * texture.Height];
            texture.GetData(colors1D);

            Color[,] colors2D = new Color[texture.Width, texture.Height];
            for (int x = 0; x < texture.Width; x++)
            {
                for (int y = 0; y < texture.Height; y++)
                {
                    colors2D[x, y] = colors1D[x + y * texture.Width];
                }
            }

            return colors2D;

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
            _groundTexture = Content.Load<Texture2D>("ground");
            _font = Content.Load<SpriteFont>("myFont");

            _screenWidth = _device.PresentationParameters.BackBufferWidth;
            _screenHeight = _device.PresentationParameters.BackBufferHeight;

            _playerScaling = 40.0f / (float)_carriageTexture.Width;

            GenerateTerrainContour();
            SetUpPlayers();
            FlattenTerrainBelowPlayers();
            CreateForeground();

            _rocketColorArray = TextureTo2DArray(_rocketTexture);
            _carriageColorArray = TextureTo2DArray(_carriageTexture);
            _cannonColorArray = TextureTo2DArray(_cannonTexture);
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

        private Vector2 TexturesCollide(Color[,] tex1, Matrix mat1, Color[,] tex2, Matrix mat2)
        {
            Matrix mat1to2 = mat1 * Matrix.Invert(mat2);
            int width1 = tex1.GetLength(0);
            int height1 = tex1.GetLength(1);
            int width2 = tex2.GetLength(0);
            int height2 = tex2.GetLength(1);

            for (int x1 = 0; x1 < width1; x1++)
            {
                for (int y1 = 0; y1 < height1; y1++)
                {
                    Vector2 pos1 = new Vector2(x1, y1);
                    Vector2 pos2 = Vector2.Transform(pos1, mat1to2);

                    int x2 = (int)pos2.X;
                    int y2 = (int)pos2.Y;
                    if ((x2 >= 0) && (x2 < width2))
                    {
                        if ((y2 >= 0) && (y2 < height2))
                        {
                            if (tex1[x1, y1].A > 0)
                            {
                                if (tex2[x2, y2].A > 0)
                                {
                                    return Vector2.Transform(pos1, mat1);
                                }
                            }
                        }
                    }
                }
            }

            return new Vector2(-1, -1);
        }

        private Vector2 CheckTerrainCollision()
        {
            Matrix rocketMat = Matrix.CreateTranslation(-42, -240, 0) *
                            Matrix.CreateRotationZ(_rocketAngle) *
                            Matrix.CreateScale(_rocketScaling) *
                            Matrix.CreateTranslation(_rocketPosition.X, _rocketPosition.Y, 0);
            Matrix terrainMat = Matrix.Identity;
            Vector2 terrainCollisionPoint = TexturesCollide(_rocketColorArray, rocketMat, _foregroundColorArray, terrainMat);
            return terrainCollisionPoint;
        }

        private Vector2 CheckPlayersCollision()
        {
            Matrix rocketMat = Matrix.CreateTranslation(-42, -240, 0) *
                               Matrix.CreateRotationZ(_rocketAngle) *
                               Matrix.CreateScale(_rocketScaling) *
                               Matrix.CreateTranslation(_rocketPosition.X, _rocketPosition.Y, 0);

            for (int i = 0; i < _numberOfPlayers; i++)
            {
                PlayerData player = _players[i];
                if (player.IsAlive)
                {
                    if (i != _currentPlayer)
                    {
                        int xPos = (int)player.Position.X;
                        int yPos = (int)player.Position.Y;

                        Matrix carriageMat = Matrix.CreateTranslation(0, -_carriageTexture.Height, 0) *
                                             Matrix.CreateScale(_playerScaling) *
                                             Matrix.CreateTranslation(xPos, yPos, 0);
                        Vector2 carriageCollisionPoint = TexturesCollide(_carriageColorArray, carriageMat, _rocketColorArray, rocketMat);

                        if (carriageCollisionPoint.X > -1)
                        {
                            _players[i].IsAlive = false;
                            return carriageCollisionPoint;
                        }

                        Matrix cannonMat = Matrix.CreateTranslation(-11, -50, 0) *
                                           Matrix.CreateRotationZ(player.Angle) *
                                           Matrix.CreateScale(_playerScaling) *
                                           Matrix.CreateTranslation(xPos + 20, yPos - 10, 0);

                        Vector2 cannonCollisionPoint = TexturesCollide(_cannonColorArray, cannonMat, _rocketColorArray, rocketMat);
                        if (cannonCollisionPoint.X > -1)
                        {
                            _players[i].IsAlive = false;
                            return cannonCollisionPoint;
                        }
                    }
                }
            }
            return new Vector2(-1, -1);
        }

        private bool CheckOutOfScreen()
        {
            bool rocketOutOfScreen = _rocketPosition.Y > _screenHeight;
            rocketOutOfScreen |= _rocketPosition.X < 0;
            rocketOutOfScreen |= _rocketPosition.X > _screenWidth;

            return rocketOutOfScreen;
        }

        private void CheckCollisions(GameTime gameTime)
        {
            Vector2 terrainCollisionPoint = CheckTerrainCollision();
            Vector2 playerCollisionPoint = CheckPlayersCollision();
            bool rocketOutOfScreen = CheckOutOfScreen();

            if (playerCollisionPoint.X > -1)
            {
                _rocketFlying = false;

                _smokeList = new List<Vector2>();
                NextPlayer();
            }

            if (terrainCollisionPoint.X > -1)
            {
                _rocketFlying = false;

                _smokeList = new List<Vector2>();
                NextPlayer();
            }

            if (rocketOutOfScreen)
            {
                _rocketFlying = false;

                _smokeList = new List<Vector2>();
                NextPlayer();
            }
        }

        private void NextPlayer()
        {
            _currentPlayer = _currentPlayer + 1;
            _currentPlayer = _currentPlayer % _numberOfPlayers;
            while (!_players[_currentPlayer].IsAlive)
            {
                _currentPlayer = ++_currentPlayer % _numberOfPlayers;
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

            if (_rocketFlying)
            {
                UpdateRocket();
                CheckCollisions(gameTime);
            }

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

[Particles](Riemers2DXNA17particles)
