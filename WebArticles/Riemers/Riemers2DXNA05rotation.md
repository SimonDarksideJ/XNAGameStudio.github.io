# Rotation – Drawing the cannon

We’ve almost covered every aspect of 2D rendering but one, rotation. In this chapter, we will render our cannons on top of the carriages, according to the rotation angle stored in the PlayerData array.

## Rotating the cannon

As mentioned in the previous chapters, by default MonoGame takes the top-left corner of the 2D image as the origin. This is important for 2 reasons, of which we’ve covered the first one in the previous chapter:

* MonoGame will position the image on the screen so the origin point is at the position you specify in the SpriteBatch.Draw method.
* When you rotate the image, MonoGame rotates the image around its origin point. This is a direct consequence of the previous point, this way the origin point stays at the position you specify, no matter what the rotation of the image is.

The image below illustrates this:

![Canon Rotation](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/2DXNA05Cannons01.jpg?raw=true)

In the upper case, the cannon is rendered with the top-left corner as the origin. Now when we would apply a rotation to the cannon, such as in the upper-right image, the cannon would be rotated around this origin point. Now imagine the cannon is rotated over 90 degrees as in the upper-right image, and the cannon would be totally outside the carriage!

What we need to do is specify the origin point in the center of rotation, which is the one point that should remain fixed while being rotated. This point is illustrated by the blue dot in the bottom part of the image. Now when the cannon is rotated over 90 degrees, the result would be much nicer, as shown in the bottom-right image.

So how can we code this in MonoGame? Fairly easy, update the DrawPlayers method to add some additional variables to get the Player position, set the origin and draw the Cannon:

```csharp
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
```

We need to specify 2 positions:

* The new origin point of the cannon image
* The screen position indicating where this origin point should be positioned.

The cannon image is an image of 70x20 pixels, so I chose (11,50) as the origin point. If you open the image file and locate this position, you’ll see it is at the rotational center of the cannon.

> Note that this position needs to be defined based on the original size of the image prior to any scaling.

Next, we need to define where on the screen this cannon origin should be positioned. We want to position it 20 pixels to the right and 10 pixels up, relative to the player’s position, which was the top-left pixel of the carriage.

Finally, we set the rotation angle using the angle stored in our PlayerData array, as well as setting the cannon color like the carriage.

Now when you run this code, all the player's cannons should be pointing to the right, as shown in the image below, as the cannon image is actually drawn upward, but because we initialized all of the angles to 90 degrees they are turned to the right.

![Summary](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/2DXNA05Cannons02.png?raw=true)

Next chapter we’ll learn how we can read in keyboard input, so we can adjust the angles while our program is running!

## Exercises

You can try these exercises to practice what you have learned:

* Try setting the cannon angle for each player to something different using MathHelper.ToRadians function with various values of degrees.
* Try playing with the SpriteEffects when drawing the carriages / cannons to see what they do (go crazy).

## The code so far

```csharp
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
        private int _screenWidth;
        private int _screenHeight;
        private PlayerData[] _players;
        private int _numberOfPlayers = 4;
        private float _playerScaling;
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

        protected override void LoadContent()
        {
            _spriteBatch = new SpriteBatch(GraphicsDevice);
            _device = _graphics.GraphicsDevice;

            // TODO: use this.Content to load your game content here
            _backgroundTexture = Content.Load<Texture2D>("background");
            _foregroundTexture = Content.Load<Texture2D>("foreground");
            _carriageTexture = Content.Load<Texture2D>("carriage");
            _cannonTexture = Content.Load<Texture2D>("cannon");

            _screenWidth = _device.PresentationParameters.BackBufferWidth;
            _screenHeight = _device.PresentationParameters.BackBufferHeight;

            SetUpPlayers();

            _playerScaling = 40.0f / (float)_carriageTexture.Width;
        }

        protected override void Update(GameTime gameTime)
        {
            if (GamePad.GetState(PlayerIndex.One).Buttons.Back == ButtonState.Pressed ||
                Keyboard.GetState().IsKeyDown(Keys.Escape))
            {
                Exit();
            }

            // TODO: Add your update logic here

            base.Update(gameTime);
        }

        protected override void Draw(GameTime gameTime)
        {
            GraphicsDevice.Clear(Color.CornflowerBlue);

            // TODO: Add your drawing code here

            _spriteBatch.Begin();
            DrawScenery();
            DrawPlayers();
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
    }
}
```

## Next Steps

[Reading out the Keyboard](Riemers2DXNA06keyboardinput)
