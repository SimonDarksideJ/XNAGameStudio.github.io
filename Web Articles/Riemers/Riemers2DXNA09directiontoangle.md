# Making the rocket move

This chapter, we will see how we can propagate the rocket. It’s fairly easy to just make it move forward since we’ve already calculated its direction in the previous chapter. However, we also want its trajectory to be influenced by the gravity, so what you shoot upwards will eventually come down again. As a result, since the direction of the rocket will be adjusted, we will update the angle at which the rocket should be drawn each time its direction is changed.

## Updating the rocket

Let us create a new method to handle this stuff. Since it doesn’t render anything but does need to be called frequently, we will call it from the Update method. This makes sure it is called exactly 60 times each second.

```csharp
    private void UpdateRocket()
    {
        if (_rocketFlying)
        {
            _rocketPosition += _rocketDirection;
        }
    }
```

Each time this method is called and a rocket has been shot, the rocketDirection will be added to the current position of the rocket. Since the rocketDirection remains the same, this will make the rocket move in a straight line. Call this method from our Update method:

```csharp
    UpdateRocket();
```

Now run the code, and launch a rocket! The rocket will fly in a straight line, at a speed depending on the current Power.

## Adding gravity

Quite an achievement, but it doesn’t look too real as it isn’t coming down, even if you shoot it at a very low power. Therefore, we will add some gravitational influence. The gravitational force is a downward influence based on the direction of the rocket. 

Replace the contents inside the if-block of the **UpdateRocket** with this code:

```csharp
    Vector2 gravity = new Vector2(0, 1);
    _rocketDirection += gravity / 10.0f;
    _rocketPosition += _rocketDirection;
```

The gravity direction is along the positive Y axis (meaning downward), each time the method is called (being 60 times per second) a portion of this downward direction is added to the direction of the rocket.

Now run this code again and try out the changes! You see the rocket makes a more realistic curve, but there’s another problem: the rocket doesn’t rotate accordingly!

## Rotating rocket along the direction

The rocket does not rotate because the SpriteBatch renders the image rotated by the rocketAngle value, and when we change the rocketDirection we don’t change this value yet. So what we want to know is: given a direction, what is the angle corresponding to this direction?

To find this angle, take a look at the left image below:

![Direction to Angle](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/2DXNA09Angle01.jpg?raw=true)

Now back in high school, there were a few very basic rules that you should have stored in the back of your head. This is one of them:

> “In a triangle having a corner of 90 degrees, you can find the angle of a corner by dividing the length of the opposite side by the length of the neighboring side, and take the arctangent of this division.”

This follows immediately from the definition of the sine, cosine, and tangent.

Applied to our example, you can find the corresponding angle between our Direction and the Right (X) axis by taking the arctangent of Direction.Y/Direction.X. This is illustrated in the left part of the image above: we’re starting from the Direction shown there, and we want to find the Angle between this direction and the X axis. This would be given by the following code:

```csharp
    Math.Atan(Direction.Y/Direction.X)
```

That’s the general case, which is usually taught in high-school maths. In the case of our rocket, there are 2 slight differences which I’ve put in bold:
We’re looking for the angle between the direction and the Y axis, in MonoGame, the Up axis is the negative Y axis
The first one is solved by changing the places of X and Y, and the latter by negating the Y value. So this is what we get.

Put the following code immediately after the previous line inside the **UpdateRocket** method.

```csharp
    rocketAngle = (float)Math.Atan2(_rocketDirection.X, -_rocketDirection.Y);
```

You will also have to add an additional "Using" statement to the top of the file ow we are using the additional builtin Atan2 function:

```csharp
    using System;
```

Instead of using the Atan method, you’ll usually want to use the Atan2 method, the difference being that with the Atan method you need to specify the division of the components as an argument. Since we all know that –X/-Y is the same as X/Y, this would require an additional if-check, this if-check is contained in the Atan2 method allowing us to specify the 2 components separately.

That is it for this chapter, run the code, and notice how your rocket is rotated corresponding to its direction!

![Direction to Angle](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/2DXNA09Angle02.png?raw=true)

In this chapter and the previous chapter, you have seen how to go from angle to direction, and from direction to angle. You will need this quite often in both 2D and 3D game programming.

## Exercises

You can try these exercises to practice what you have learned:

* No exercises this time, but get ready for the next section!

## The code thus far

```csharp
using System;
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
            _rocketTexture = Content.Load<Texture2D>("rocket");
            _font = Content.Load<SpriteFont>("myFont");

            _screenWidth = _device.PresentationParameters.BackBufferWidth;
            _screenHeight = _device.PresentationParameters.BackBufferHeight;

            SetUpPlayers();

            _playerScaling = 40.0f / (float)_carriageTexture.Width;
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
    }
}
```

## Next Steps

[Adding the smoke trail](Riemers2DXNA10smoketrail)
