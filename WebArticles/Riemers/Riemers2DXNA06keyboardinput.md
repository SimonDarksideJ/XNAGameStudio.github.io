# Reading out the Keyboard

In this chapter, we’ll see how easy it is in MonoGame to read keyboard input. For this game whenever the user presses the left or right button, the angle of the cannon of the current player will be adjusted.

## Getting Keyboard state

Start by adding this variable at the Properties section of our code, so we know which cannon to adjust:

```csharp
    private int _currentPlayer = 0;
```

As the keyboard needs to be checked frequently it is added to the update logic our game, so we can update the angles of the cannon.

To keep our Update method clean, we will add a new method called **ProcessKeyboard** which we will then call from the Update method.

```csharp
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
    }
```

This code is basically all we need!

* The first line retrieves the current state of the keyboard, holding which keys are currently pressed.
* We use this KeyboardState object to the whether the Left and Right arrow keys are currently down, and if this is the case we change the angle of the current player.

Now we need to call this method from our Update method (before *base.Update*), ensuring the keyboard is read exactly 60 times each second:

```csharp
    ProcessKeyboard();
```

## Rotation validation (do not hit the floor)

Now run this code! When you press the left and right arrow keys, the cannon of the first (red) player will rotate. However, at this moment you are able to rotate the cannon so it’s colliding with the terrain, so we should add this little logic to the end of the **ProcessKeyboard** method:

```csharp
    if (_players[_currentPlayer].Angle > MathHelper.PiOver2)
    {
        _players[_currentPlayer].Angle = -MathHelper.PiOver2;
    }
    if (_players[_currentPlayer].Angle < -MathHelper.PiOver2)
    {
        _players[_currentPlayer].Angle = MathHelper.PiOver2;
    }
```

In this code, you need to remember that Pi = 3.14 radians which correspond to 180 degrees, so PiOver2 radians corresponds to 90 degrees. Then it is easy:

* If the cannon is at 90 degrees (meaning horizontally to the right), and the right button is pressed, the angle changes to -90 degrees so it is pointing to the left.
* The last 2 lines make this true for the other way around.

have a go and see what this does when running the code!

## Adding more keys

Now we will add a last block of code to our **ProcessKeyboard** method, which allows the player to adjust the Power of the cannon:

```csharp
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
```

When the Up or Down arrows are pressed, the power is incremented or decremented by one. Since this way it can take a while to reach a high power, if the PageUp or PageDown keys are pressed the power will increment and decrement in much larger steps. The last 4 lines simply limit the Power value between 0 and 1000.

![Summary](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/2DXNA06Keyboard01.png?raw=true)

That is it for this chapter! You can see the cannon rotate on your keyboard input, but it’s hard to see the Power is changing. Therefore, next chapter we will see how to render text to the screen, so we can print the Angle and Power of the current player to the screen!

## Exercises

You can try these exercises to practice what you have learned:

* Add more keys for increasing/decreasing the power
* Try and make the rotation lock in place instead of flip to the other side of the carriage (like the power)

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
        private int _screenWidth;
        private int _screenHeight;
        private PlayerData[] _players;
        private int _numberOfPlayers = 4;
        private float _playerScaling;
        private int _currentPlayer = 0;
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

[Writing text to the screen in XNA](Riemers2DXNA07writingtext)
