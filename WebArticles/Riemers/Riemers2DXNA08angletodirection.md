# Launching the rocket

In this chapter, we will extend our code so the cannon will launch a rocket whenever the spacebar is pressed.

## Adding the Rocket

Start by importing the **Rocket.png** image into your content project and then add a new variable into the **Properties** section of our code to store the Texture data:

```csharp
    private Texture2D _rocketTexture;
```

And initialize it in our LoadContent method:

```csharp
    _rocketTexture = Content.Load<Texture2D> ("rocket");
```

Next, add these additional variables to the Properties section of our code for tracking various states of a rocket:

```csharp
    private bool _rocketFlying = false;
    private Vector2 _rocketPosition;
    private Vector2 _rocketDirection;
    private float _rocketAngle;
    private float _rocketScaling = 0.1f;
```

These variables can be described as follows:

* The first one keeps track of whether our rocket is in the air, and thus whether it should be drawn.
* Obviously, at all times we need to know the position of the rocket.
* The direction and angle of the rocket are needed when calculating the next position of the rocket and is closely related to the angle. In fact, the angle can be calculated from the direction, but because we need both quite frequently we want both to be a easily accessible variable.
* Finally, the scaling variable will remain a fixed constant, but since we will need it a few times in our code we also add it as a variable.

## Launching the rocket from the keyboard

We want to launch the rocket on a press of the spacebar (and the Enter key for fun). So add this code to the end of the ProcessKeyboard method:

```csharp
    if (keybState.IsKeyDown(Keys.Enter) || keybState.IsKeyDown(Keys.Space))
    {
        _rocketFlying = true;
        _rocketPosition = _players[_currentPlayer].Position;
        _rocketPosition.X += 20;
        _rocketPosition.Y -= 10;
        _rocketAngle = _players[_currentPlayer].Angle;
    }
```

The above code will:

* Set the rocketFlying value to true
* Sets the position and angle for the rocket
* Moves the rocket to appear just beyond the end of the cannon

For the position, you start at the bottom-left corner of the carriage and add 20 pixels to the right and 10 pixels up, corresponding to the center of the carriage. For the starting angle of the rocket, we should simply take the current angle of the cannon.

## Firing the rocket

A bit more complicated is to find the direction of the rocket. A direction should have an X and Y component, indicating how many pixels horizontally and vertically the rocket should be moved each time. Obviously, this can be found from the angle of the rocket.

One approach would be to take the sine and cosine of this angle and use this as Y and X components, but MonoGame allows us to use a much more elegant solution. We will start from the (0,-1) Up direction, and ask MonoGame to rotate this over an angle we specify. This is illustrated in the image below:

![Angle to Direction](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/2DXNA08Direction01.jpg?raw=true)

Let’s start by defining the Up vector which is along the negative Y axis, as its coordinates are (0,-1): (put this code after the _rocketAngle line shown above)

```csharp
    Vector2 up = new Vector2(0, -1);
```

Next, we want MonoGame to rotate this vector by having MonoGame construct a rotation matrix, this sounds very complex but it is not (a matrix is something you can multiply with a vector to obtain a transformed version of that vector). If MonoGame can give us a matrix containing our rotation, we can use this matrix to obtain the rotated version of our vector for the rocket.

This is how we can obtain the rotation matrix:

```csharp
    Matrix rotMatrix = Matrix.CreateRotationZ(_rocketAngle);
```

To explain the line above, think of how you would rotate the Up vector shown in the image above, you would take it between 2 fingers, and rotate it. Important here is that your fingers are pointing to the screen.

This helps to understand the line of code above, it creates a matrix containing a rotation around the Z axis which provides us with what we need. If the X axis is pointing to the right and the Y axis is pointing down, the Z axis is sticking out of the screen!

Now we have this matrix containing our rotation, we can ask MonoGame to transform our Up vector with this rotation:

```csharp
    _rocketDirection = Vector2.Transform(up, rotMatrix);
```

This will take the original up direction, transform it with the rotation around the Z axis and store the result in the rocketDirection variable! Now all we need to do is scale this direction up or down, depending on the Power the rocket was shot:

```csharp
    _rocketDirection *= _players[_currentPlayer].Power / 50.0f;
```

## Drawing the rocket

At this moment, we have stored everything we need of our rocket in variables, so let us create a small method that renders the rocket to the screen:

```csharp
    private void DrawRocket()
    {
        if (_rocketFlying)
        {
            _spriteBatch.Draw(_rocketTexture, _rocketPosition, null, _players[_currentPlayer].Color, _rocketAngle, new Vector2(42, 240), _rocketScaling, SpriteEffects.None, 1);
        }
    }
```

The first line checks whether we fired a rocket. The second line issues the SpriteBatch to draw:

* The rocket image
* The rocketPosition on the screen
* The whole image, that is
* The color of the current player
* Rotated over an angle stored in rocketAngle
* The rotational and positional origin (42, 240) (look this position up in the original rocket image for reference)
* Scale down 10 the image by 10% using the _rocketScaling variable
* No effects, e.g. Not mirrored
* Layer with distance 1

Don’t forget to call this method from within our Draw method:

```csharp
    DrawRocket();
```

Now when you run this code, whenever you press the spacebar or the enter key, you should see a rocket positioned at the cannon! Since we are not yet updating the position, the rocket will not move yet, but this is easy to solve in the next chapter as we’ve already calculated the direction of the rocket!

![Angle to Direction](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/2DXNA08Direction02.png?raw=true)

## Exercises

You can try these exercises to practice what you have learned:

* No exercises this time, but get ready for the next section!

## The code thus far

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
            _rocketTexture = Content.Load<Texture2D> ("rocket");
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

        protected override void Update(GameTime gameTime)
        {
            if (GamePad.GetState(PlayerIndex.One).Buttons.Back == ButtonState.Pressed ||
                Keyboard.GetState().IsKeyDown(Keys.Escape))
            {
                Exit();
            }

            // TODO: Add your update logic here

            ProcessKeyboard();

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

[Making the rocket move](Riemers2DXNA09directiontoangle)
