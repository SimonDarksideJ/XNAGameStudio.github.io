# 2D Collision Detection

With our rockets flying through the screen and each pixel of the terrain known to our code, we are ready for the most challenging and thus interesting part of this series, collision detection.

## 2D Collision 101

The general idea behind collision detection between 2 images is very straightforward, for each pixel of the first image, we will check if it is in the same position as (collides) with a pixel of the second image.

However, because an image is always a rectangle does not mean that the image it contains is actually rectangular as most images contain a lot of transparent pixels. This is shown in the left image below. Note that the transparent pixels have a greenish color, while the non-transparent pixels have a soft-red color (don’t you dare to call it pink). We’ll get back to this later.

![Collision](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/2DXNA14Collision1.png?raw=true)

Before we can check for collision between 2 images, we need to make sure they are positioned correctly when they are moved to the correct position on the screen. If we did not first position them correctly before checking for collisions, then the top-left pixels of both images would align and we would almost always have a collision. This is shown in the right part of the image above.

For example, if both images are positioned so that the rocket is positioned in front of the cannon, much like shown in the left part of the image below. Although in the left part the rectangles of both images collide, we see the actual cannon and rocket detail actually do not collide. This is because, in all pairs of overlapping pixels, at least one of both is transparent.

![Collision](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/2DXNA14Collision2.png?raw=true)

When comparing 2 colliding pixels, if either of them is transparent there is no collision between the objects, only when both the pixel of the first and of the second image are non-transparent is there a real collision. This is presented in the table below:

![Chart](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/2DXNA14Collision3.png?raw=true)

An example of a real collision is shown in the right part of the image above. You see that the final 4 pixels of the cannon are non-transparent, while they collide with 4 non-transparent pixels of the rocket.

## 2D collision with rotated and scaled textures

This however, is not all there is to say about 2D collision detection. In the figures above, neither of the 2D images has been scaled or rotated, but when our rocket collides with a cannon it will also be scaled and rotated as shown in the left part of the image below.

![Angle Collision](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/2DXNA14Collision4.png?raw=true)

The right part if the image above shows the most complicated case where both images are both scaled and/or rotated. We will meet this case in our game.  The real question of this chapter is how we can check for collisions in such cases.

The following is some pseudocode for how we will handle this:

```text
 For each pixel in image A:
 {
     Find the screen coordinate occupied by this pixel
     Find the pixel in image B that is also rendered to this position
     If both pixels are non-transparent
     {
         COLLISION!
     }
 }
```

This is illustrated in the image above, the image shows a screen with a rotated cannon and a rotated and scaled rocket as shown in the top-left corner as a reference.

## Using Matrices to simplify detection

Very nice, but how can we ‘Find the screen coordinate occupied by this pixel’? This is done through matrices. Remember though matrices are nothing magical, in fact, they are just a grid of 4x4 numbers, but all you to remember about matrices at this point is this:

* A matrix can store the position, rotation, and scale of an image.

When a matrix holds the position, rotation, and scale of an image, you can use this matrix to find the final screen coordinate of any pixel of this image, also important in this case in the other way around, if we take the inverse of this matrix given any screen coordinate, then we can find the coordinate of the pixel in the original image.

Instead of doing this exercise for all pixels of image A, we will only discuss this for the bottom-right pixel (3,8) of image A. 

* As a first step, we want to find to which screen coordinate this pixel will be drawn which we can find by multiplying (=’transforming’) the pixel’s coordinate (3,8) by the image matrix, this will give us the coordinate of where the pixel will be rendered on the screen!
* The next step is the opposite, given the screen coordinate, we want to find which pixel in image B is drawn there, this is done by transforming the screen coordinate by the inverse of the matrix if image B. This will give us the coordinate in the original image B, which is (13,2) in this case.
* Next, we see whether both the pixel in the original image A and in the original image B are non-transparent, if so both images collide on the screen!

Let’s put this in some code. We will create a method, TexturesCollide, which will return whether 2 images collide. The method will need 4 things in order to work properly:

* The 2D array of colors of image 1
* The matrix of image 1
* The 2D array of colors of image 2
* The matrix of image 2

## The collision detection function

Let us start to implement this with the following method:

```csharp
    private Vector2 TexturesCollide(Color[,] tex1, Matrix mat1, Color[,] tex2, Matrix mat2)
    {
    }
```

> Ignore any errors raised from this new function as we slowly build it up and explain.

The method will eventually return whether there is a collision and if there is one, it will return the screen position of where the collision occurred. If no collision was detected, (-1,-1) will be returned.

Using matrices, we need to transform a coordinate by mat1 and then by the inverse in mat2. We can obtain the matrix that is the combination of both by simply multiplying them. Put this line at the top of the method:

```csharp
    Matrix mat1to2 = mat1 * Matrix.Invert(mat2);
```

Since the mat1to2 matrix is the combination of the mat1 and the inverse(mat2) matrices, transforming a coordinate from Image 1 by this matrix will immediately give us the coordinate in Image 2!

Let us next translate “For each pixel in image A” into C# code, add this to the top of the method:

```csharp
    int width1 = tex1.GetLength(0);
    int height1 = tex1.GetLength(1);
    int width2 = tex2.GetLength(0);
    int height2 = tex2.GetLength(1);

    for (int x1 = 0; x1 < width1; x1++)
    {
        for (int y1 = 0; y1 < height1; y1++)
        {
        }
    }
```

For each pixel of image 1 we first want to find the corresponding screen coordinate, this is done by transforming the original X,Y coordinate with the matrix of image 1 as well as using the inverse of the mat2 matrix to find the corresponding position in image 2. So add these lines inside the double for-loop:

```csharp
    Vector2 pos1 = new Vector2(x1,y1);
    Vector2 pos2 = Vector2.Transform(pos1, mat1to2);
```

At this moment we now have 2 positions from the original images  which we know if they are drawn to the same screen pixel. All we need to do now is check whether they are both non-transparent:

```csharp
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
                    Vector2 screenCoord = Vector2.Transform(pos1, mat1);
                    return screenCoord;
                }
            }
        }
    }
```

Which checks whether the Alpha component of both colors is larger than 0. If both colors are non-transparent, we transform the screen coordinate by the inverse of the matrix from image 2 and return the screen coordinate.

Now, in case no collision is detected we need to return (-1,-1) to the calling code, later on we will update our game so it interprets (-1,-1) as "no collision found". So put this line at the very end of the method:

```csharp
    return new Vector2(-1,-1);
```

This implements the core functionality of the method. While there might be cases that this method survives, in general, it will crash because some pixels of image A will fall outside of image B. In the image above for example, the very first pixel of the rocket (to the left of the top of the rocket) does not correspond to a pixel in image B, so this line will cause or program to crash:

```csharp
    if (tex2[x2, y2].A > 0)
```

## Final 2D collision detection code

As this has been a bit of a rollercoaster ride, here is the full function for reference:

```csharp
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
                                Vector2 screenCoord = Vector2.Transform(pos1, mat1);
                                return screenCoord;
                            }
                        }
                    }
                }
            }
        }

        return new Vector2(-1, -1);
    }
```

> Note also, that we do not use the screen coordinates anymore. As such, when a collision is detected, we still need to transform the current coordinate of Image 1 by mat1 to obtain the screen coordinate, so it can be returned to the calling code.  So the return line can be simplified to:
>
> ```csharp
>   return Vector2.Transform(pos1, mat1);
> ```

There’s no screenshot for this chapter, although we’ve added some powerful functionality to our code. In the next chapter, we will see how we can use this functionality to detect collisions between any two images.

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

[2D Transformation matrices](Riemers2DXNA15collisionmatrices)
