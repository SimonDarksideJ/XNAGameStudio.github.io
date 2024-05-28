# Particles

With collision detection added to our project, the functionality of our game is almost complete. There are, however, some things lacking, such as a nice explosion whenever the rocket hits the terrain or another cannon.

A few years ago, 2D explosions were nothing more than pre-defined animations, similar to .gif image sequences, these contained a number of frames that were being cycled through.

Today, since the available processing power has increased dramatically, both 2D as 3D explosions are built up from many small very faint fireball images that are rendered on top of each other, these small images all start at the explosion origin and travel in random directions while they fade away over time. These small images are called particles, and the engine that updates them is called a particle system.

The particle we are going to use as a building block for our explosions is shown below (explosion.png), and can be found in the assets pack you downloaded previously. Import it into our project, add a variable in the Properties section:

```csharp
    private Texture2D _explosionTexture;
```

And load it in our LoadContent method:

```csharp
    _explosionTexture = Content.Load<Texture2D>("explosion");
```

> Notice that the image is mainly black, with a faint fireball image in the center. This will be further discussed at the end of this chapter.

![Particle](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/2DXNA17Particles1.png?raw=true)

Before we can render the particles there are a few things about them that we need to keep track of. For example, we need to store the particle's current positions and scaling factors. To enable us to calculate their next position, we will need their direction, but that’s not all, we want the particles to also grow larger and fade out as time goes by, which means for each particle we need to store when it was created and how long the particle should stay alive before totally disappearing. Finally, since the particles should move very fast in the beginning and slow down at the end, we will need to store its acceleration.

To store this data in a nice way, we will define a new struct called **ParticleData**, which can store all of this data. Add this at the very top of our file after the **PlayerData** struct:

```csharp
    public struct ParticleData
    {
        public float BirthTime;
        public float MaxAge;
        public Vector2 OriginalPosition;
        public Vector2 Acceleration;
        public Vector2 Direction;
        public Vector2 Position;
        public float Scaling;
        public Color ModColor;
    }
```

TO keep track of all the particles being rendered we will need a List, so add this variable to our code in the Properties section:

```csharp
    List<ParticleData> _particleList = new List<ParticleData>();
```

Next, we will create a small method called **AddExplosion** which will generate a few particles:

```csharp
    private void AddExplosion(Vector2 explosionPos, int numberOfParticles, float size, float maxAge, GameTime gameTime)
    {
        for (int i = 0; i < numberOfParticles; i++)
        {
            AddExplosionParticle(explosionPos, size, maxAge, gameTime);
        }
    }
```

Whenever we start a new explosion there are a few things we want to control. The approach explained in this series gives us full control over the explosion, we will be able to determine:

* Where the explosion starts
* How many particles we want for one explosion
* The maximum size of the explosion
* And how long the explosion should take!

The method above simply calls the **AddExplosionParticle** method a couple of times which is where the particles are actually being added to the List, so let us add the missing method:

```csharp
    private void AddExplosionParticle(Vector2 explosionPos, float explosionSize, float maxAge, GameTime gameTime)
    {
        ParticleData particle = new ParticleData();

        particle.OriginalPosition = explosionPos;
        particle.Position = particle.OriginalPosition;

        particle.BirthTime = (float)gameTime.TotalGameTime.TotalMilliseconds;
        particle.MaxAge = maxAge;
        particle.Scaling = 0.25f;
        particle.ModColor = Color.White;
    }
```

The method starts by creating a new, empty **ParticleData** object. There are a few things we can immediately store inside this object. The center position of the explosion can be stored in the **OriginalPosition** property and the current (first) position the particle, which should be the same center of the explosion.

Next, we store the "birth time" of the particle, which will be needed each frame when we need to determine the age of the particle. The maximum age will determine when the particle should be completely gone. The scaling property will determine the size of the particle, while the ModColor will allow us to make the particle fade away towards the end of its life.

There is one very important property missing: the direction of the particle, if you want to make a more realistic particle system there is one very important word, randomness. Although all particles for one explosion should start in the same position, they should move towards different directions and the only way to obtain a nice result is to make these directions totally random.

The following code will generate a random direction. Not only the direction itself is random, but also the length:

```csharp
    float particleDistance = (float)_randomizer.NextDouble() * explosionSize;
    Vector2 displacement = new Vector2(particleDistance, 0);
    float angle = MathHelper.ToRadians(_randomizer.Next(360));
    displacement = Vector2.Transform(displacement, Matrix.CreateRotationZ(angle));

    particle.Direction = displacement;
    particle.Acceleration = 3.0f * particle.Direction;
```

First, we will start by determining how far the current particle should have traveled at the end of its life, we want a random value between 0 and the maximum size of the explosion.

We will use this **particleDistance** variable as the length of our direction, the only thing we need to do is rotate this over a random angle. This is done with the next three lines, first, we define a Down vector of the length found in the first line. Next, we generate a random angle, and use this random angle to create a rotation matrix, as explained in the **"Angle to Direction"** chapter, which is used to rotate the Down vector over the random angle.

At this moment, we do not know exactly how the particle should be displaced at the end of its life, for now, we use this displacement vector as Direction and Acceleration, which is totally wrong as we will see in the next chapter.

Finally, this new particle should be added to the particleList at the end of the **AddExplosionParticle** method:

```csharp
    _particleList.Add(particle);
```

That is it for the **AddExplosionParticle** method, remember this method is called by the **AddExplosion** method which we should call whenever the rocket collides with the terrain or a player. This means the ideal place to call the AddExplosion method is inside our **CheckCollisions** method where we have already defined 3 separate if-blocks, each corresponding to a different type of rocket collision.

Go to our **CheckCollisions** method, and find the first "if-block" which is executed in case a collision between the rocket and a player was detected. Inside this "if-block", add this line (between smokelist and NextPlayer):

```csharp
    AddExplosion(playerCollisionPoint, 10, 80.0f, 2000.0f, gameTime);
```

This will cause 10 new particles to be added to our particleList. The size and duration of the explosion don’t really matter in this chapter as we will be updating our particles in the next chapter.

In the second if-block, add the following line which will start a smaller explosion each time the rocket hits the terrain:

```csharp
    AddExplosion(terrainCollisionPoint, 4, 30.0f, 1000.0f, gameTime);
```

To finish off this chapter, we will simply draw our particles to the screen, since we are not yet updating their positions, they will remain fixed at the collision point. Add the following method to the very end of our code:

```csharp
    private void DrawExplosion()
    {
        for (int i = 0; i < _particleList.Count; i++)
        {
            ParticleData particle = _particleList[i];
            _spriteBatch.Draw(_explosionTexture, particle.Position, null, particle.ModColor, i, new Vector2(256, 256), particle.Scaling, SpriteEffects.None, 1);
        }
    }
```

And best not to forget to call this new **DrawExplosion** method from our **Draw** method, immediately before our call to **SpriteBatch.End**:

```csharp
    DrawExplosion();
```

Now run this code! Whenever your rocket hits the terrain or a player, you should see the particles being rendered to your screen. Note that the result depends on whether you hit the terrain or a player, which is exactly as we defined in our CheckCollisions method.

![Summary](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/2DXNA17Particles2.png?raw=true)

I agree that this final image is not really what we expect of an explosion. But we’re definitely on the right track, and the result will be refined in the next 2 chapters.

## Exercises

You can try these exercises to practice what you have learned:

* Try playing with the explosion variables from the CheckCollisions method to alter the explosions
* I wonder what happens if you do actually shoot a rocket off the screen? Global Thermonuclear War? How big is that explosion?

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

    public struct ParticleData
    {
        public float BirthTime;
        public float MaxAge;
        public Vector2 OriginalPosition;
        public Vector2 Acceleration;
        public Vector2 Direction;
        public Vector2 Position;
        public float Scaling;
        public Color ModColor;
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
        private Texture2D _explosionTexture;
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
        List<ParticleData> _particleList = new List<ParticleData>();


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
            _explosionTexture = Content.Load<Texture2D>("explosion");
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
                AddExplosion(playerCollisionPoint, 10, 80.0f, 2000.0f, gameTime);
                NextPlayer();
            }

            if (terrainCollisionPoint.X > -1)
            {
                _rocketFlying = false;

                _smokeList = new List<Vector2>();
                AddExplosion(terrainCollisionPoint, 4, 30.0f, 1000.0f, gameTime);
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

        private void AddExplosion(Vector2 explosionPos, int numberOfParticles, float size, float maxAge, GameTime gameTime)
        {
            for (int i = 0; i < numberOfParticles; i++)
            {
                AddExplosionParticle(explosionPos, size, maxAge, gameTime);
            }
        }

        private void AddExplosionParticle(Vector2 explosionPos, float explosionSize, float maxAge, GameTime gameTime)
        {
            ParticleData particle = new ParticleData();

            particle.OriginalPosition = explosionPos;
            particle.Position = particle.OriginalPosition;

            particle.BirthTime = (float)gameTime.TotalGameTime.TotalMilliseconds;
            particle.MaxAge = maxAge;
            particle.Scaling = 0.25f;
            particle.ModColor = Color.White;

            float particleDistance = (float)_randomizer.NextDouble() * explosionSize;
            Vector2 displacement = new Vector2(particleDistance, 0);
            float angle = MathHelper.ToRadians(_randomizer.Next(360));
            displacement = Vector2.Transform(displacement, Matrix.CreateRotationZ(angle));

            particle.Direction = displacement;
            particle.Acceleration = 3.0f * particle.Direction;

            _particleList.Add(particle);
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
            DrawExplosion();
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

        private void DrawExplosion()
        {
            for (int i = 0; i < _particleList.Count; i++)
            {
                ParticleData particle = _particleList[i];
                _spriteBatch.Draw(_explosionTexture, particle.Position, null, particle.ModColor, i, new Vector2(256, 256), particle.Scaling, SpriteEffects.None, 1);
            }
        }
    }
}
```

## Next Steps

[Additive alpha blending](Riemers2DXNA18alphablending)
