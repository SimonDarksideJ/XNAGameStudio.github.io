# Particle engine

At this moment, whenever the rocket hits the terrain or a player, some particles are created and added to a list and rendered to the screen using additive alpha blending. However, the particles are not moving yet, which is what we’re going to solve this chapter.

## Evaluating live particles

The current position of any object can be calculated from 3 properties:

* The initial position: this is the position of the particle at the very beginning of its life, the center of our explosion, the position on the screen where the collision happened.
* The initial speed: the speed defines how much the position should be changed each second. (in 2D also called direction or velocity), for an explosion, the speed of the particles should be very at the beginning. In a 2D game, since the position has an X and Y component, the speed also has an X and Y component.
* The acceleration: the acceleration defines how much the speed should be changed each second since the particles start at a high speed, we want them to slow down over time so their speed reaches 0 at the end of the particle’s life. As you should feel, and as shown later, this means that the direction of the speed should be the opposite of the direction of the speed.

As I mentioned earlier, based on the initial position the initial speed, and the acceleration, you can find the current position of the particle over time. This is done using the formula below:

![Formula](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/2DXNA19ParticleEngine1.png?raw=true)

We will code a new **UpdateParticles** method which scrolls through our List of particles and updates their positions accordingly. Starting with this code:

```csharp
    private void UpdateParticles(GameTime gameTime)
    {
        float now = (float)gameTime.TotalGameTime.TotalMilliseconds;
        for (int i = particleList.Count - 1; i >= 0; i--)
        {
            ParticleData particle = _particleList[i];
            float timeAlive = now - particle.BirthTime;

            if (timeAlive > particle.MaxAge)
            {
                _particleList.RemoveAt(i);
            }
            else
            {
                //update current particle
            }
        }
    }
```

This method gets the current game time 'now' in Milliseconds (a float). Next, it scrolls through all particles and finds the current age of each particle, if the particle is older than its maximum age then we delete it from our List.

> Note that the List is scrolled backwards. This has to do with deleting an object in a List: if you delete an object (i), all objects following i jump one place forward, so i+1 becomes i and i+2 becomes i+1. Now, if you would be scrolling forward, the next iteration you check object i+1, which means you have skipped one object.

## How to update the particles over time

Now, let us replace the comment line (**update current particle**) with some actual code, the following code should all be placed within the "Else" clock indicated by the comment above.

If the particle is still active, we should update its position using the simple formula above. However, when working with time, we usually want to scale the time to a value between 0 and 1, where 0 means ‘begin’ and 1 means ‘end’, which will make a lot of the calculations easier as you’ll see at the end of the chapter. We’ll call this value between 0 and 1 the ‘relative time’, relTime in short, as follows:

```csharp
    float relAge = timeAlive / particle.MaxAge;
```

We will use this value as the time in the formula above, which becomes:

```csharp
    particle.Position = 0.5f * particle.Acceleration * relAge * relAge + particle.Direction * relAge + particle.OriginalPosition;
```

This code will correctly update the position of each particle each time the method is called, but that is not all we want to change, we also want to update the transparency and scale of our image. As the particle gets closer to its end, we want the particle to fade away and to grow larger, both effects combined together will make sure the particles fade smoothly away.

We can make the particles fade away by reducing their color strength, remembering that the colors of our particles are all added together and then added to the scene. Now, before this is being done we will decrease their color values, by multiplying them with a value smaller than 1, between 1 and 0.

At the beginning of a particle's life, we will multiply its colors by the modulation color (1,1,1,1) white, then when it is halfway through, we will multiply it by color (0.5, 0.5, 0.5, 0.5) and at the very end we will multiply it by color (0,0,0,0), which makes sure the effect has absolutely zero effect in the final screen.

For this, we need to find this value that starts at 1 and ends at 0, which is easy to find since we already have a time value that starts at 0 and ends at 1:

```csharp
    float invAge = 1.0f - relAge;
    particle.ModColor = new Color(new Vector4(invAge, invAge, invAge, invAge));
```

Now in our **DrawExplosion** method, we need to update the Particle drawing so that each particle is now using the ModColor as modulation color:

```csharp
    spriteBatch.Draw(_explosionTexture, particle.Position, null, particle.ModColor, i, new Vector2(256, 256), particle.Scaling, SpriteEffects.None, 1);
```

Back to our **UpdateParticles** method, we next need to make sure the particle grows as it nears its end, which we will make its size depend on its distance from the origin, the further away from the origin the larger the particle should be.

```csharp
    Vector2 positionFromCenter = particle.Position - particle.OriginalPosition;
    float distance = positionFromCenter.Length();
    particle.Scaling = (50.0f + distance) / 200.0f;
```

The first 2 lines calculate the distance between the current and the original position of the particle, the distance is used to determine the current size of the particle. The 50.0f pixels offset is required, otherwise the scaling would be 0 at the beginning where distance equals 0, whereas the actual values of 50 and 200 were found to be better when experimenting. When the distance is 150 pixels, the scaling will be equal 1.

To finish off this method, we still need to save the updated particle back into our List:

```csharp
    _particleList[i] = particle;
```

That’s it for the logic of our **UpdateParticles** method! Each time the method is called, the position, modulation color, and size of each particle is updated.

## Updating the particles from Update

Let us also call this method from within our **Update** method by adding the following code just before the "base.Update" call, this enables the particles to be updated exactly 60 times each second:

```csharp
    if (_particleList.Count > 0)
    {
        UpdateParticles(gameTime);
    }
```

And as a minor adjustment, let us also lock our keyboard when the rocket is flying and when there is an active explosion.  This will make sure the player cannot change his cannon angle or power after he’s launched the rocket. In the **Update** method, remove the original "ProcessKeyboard()" line and add the following code just before "base.Update".

```csharp
    if (!_rocketFlying && _particleList.Count == 0)
    {
        ProcessKeyboard();
    }
```

When you run this code, your explosions should look finished and the player can only fire one rocket at any time!

## Refining the math

However, when you take a close look at them (especially the larger explosions), you’ll notice some things are not 100% correct: towards the end of the explosion, your particles are going faster and faster! Even worse, their final positions don’t respect the explosion size you specified.

This is because in our **AddExplosionParticle** method, we chose the Direction and Acceleration of our particles without thinking about them, so we just need to think them over again and everything will be nice.

There are 2 constraints that help us find the correct values. At the end of our explosions (when relAge=1), we want:

* the final speed to be exactly 0
* the final position should be the starting position + the “displacement” vector calculated in our AddExplosionParticle

Let us put this in some formulas. Since acceleration defines the change in speed over time, and relAge = 1 at the end of the particle, we can write the first constraint like this:

![Formula](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/2DXNA19ParticleEngine2.png?raw=true)

Or

![Formula](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/2DXNA19ParticleEngine3.png?raw=true)

Which means we just need to find the initial speed. Let’s write the second constraint in a formula:

![Formula](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/2DXNA19ParticleEngine4.png?raw=true)

We can drop the posInitial at both sides, and replace the acceleration with our first constraint:

![Formula](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/2DXNA19ParticleEngine5.png?raw=true)

Or

![Formula](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/2DXNA19ParticleEngine6.png?raw=true)

That’s all we needed to know! Go to our **AddExplosionParticle**, and replace these lines so they reflect what we just calculated:

Replacing:

```csharp
    particle.Direction = displacement;
    particle.Acceleration = 3.0f * particle.Direction;
```

with the following:

```csharp
    particle.Direction = displacement * 2.0f;
    particle.Acceleration = - particle.Direction;
```

Now when you run this code, you’ll notice your explosion grows as large as we defined in the explosionSize variable, and slows down until its speed reaches 0 at the end of its life!

![Summary](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/2DXNA19ParticleEngine7.png?raw=true)

## Exercises

You can try these exercises to practice what you have learned:

* Now that explosions are a little more realistic, try swapping out the explosion texture with another and try some different effects.

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

            particle.Direction = displacement * 2.0f;
            particle.Acceleration = -particle.Direction;

            _particleList.Add(particle);
        }

        private void UpdateParticles(GameTime gameTime)
        {
            float now = (float)gameTime.TotalGameTime.TotalMilliseconds;
            for (int i = _particleList.Count - 1; i >= 0; i--)
            {
                ParticleData particle = _particleList[i];
                float timeAlive = now - particle.BirthTime;

                if (timeAlive > particle.MaxAge)
                {
                    _particleList.RemoveAt(i);
                }
                else
                {
                    //update current particle
                    float relAge = timeAlive / particle.MaxAge;
                    particle.Position = 0.5f * particle.Acceleration * relAge * relAge + particle.Direction * relAge + particle.OriginalPosition;

                    float invAge = 1.0f - relAge;
                    particle.ModColor = new Color(new Vector4(invAge, invAge, invAge, invAge));

                    Vector2 positionFromCenter = particle.Position - particle.OriginalPosition;
                    float distance = positionFromCenter.Length();
                    particle.Scaling = (50.0f + distance) / 200.0f;

                    _particleList[i] = particle;
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

            if (_rocketFlying)
            {
                UpdateRocket();
                CheckCollisions(gameTime);
            }

            if (_particleList.Count > 0)
            {
                UpdateParticles(gameTime);
            }

            if (!_rocketFlying && _particleList.Count == 0)
            {
                ProcessKeyboard();
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

            _spriteBatch.Begin(SpriteSortMode.Deferred, BlendState.Additive);
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

[Adding explosion craters](Riemers2DXNA20explosion)
