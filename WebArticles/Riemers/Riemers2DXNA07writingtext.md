# Writing text to the screen in XNA

In this chapter, we will learn how to render text to the screen so we can print the Power and Angle of the current player’s cannon.

Text is very easy in MonoGame as it uses TrueType fonts to generate the textures needed to render text.  Other options are available but let us just go through the basics for now.

## Importing fonts into your project

First, we need to add a font to our project and bind it to a variable in our code, much like how we would add an image to a project. Then we can simply render it using the SpriteBatch. We even don’t need to create a separate SpriteBatch: we can simply use the SpriteBatch we’re using to render our terrain and cannons.

Let’s start by adding the font to our project. To do this, find the Content project in your Solution Explorer at the top-right of your screen, and right-click on it. From the drop-down list, select Add -> New Item, as shown in the image below.

![MGCB-Editor Add Spritefont](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/2DXNA07Text01.png?raw=true)

In the dialog that opens, select “SpriteFont Description (.spritefont)” and give it a name at the top, I have used **myFont.spritefont** for this tutorial.  

![MGCB-Editor Add Spritefont](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/2DXNA07Text02.png?raw=true)

When you are ready, click the Add button and you should see the myFont.spritefont file has been added to the Content project of our project.

If you double-click the new myFont.spritefont file you can look at its contents (you made need to select an app to view it, for which I recommend VSCode). Once open, find the **FontName** entry in the file and confirm it is set to Arial (if not, simply enter "Arial" in the field). This will define the font family to use for the text, you can choose from any font installed on your system.  In this text file you can also change the size of your font, by default, the size is set to 12 points.

```xml
<?xml version="1.0" encoding="utf-8"?>
<!--
This file contains an xml description of a font, and will be read by the XNA
Framework Content Pipeline. Follow the comments to customize the appearance
of the font in your game, and to change the characters which are available to draw
with.
-->
<XnaContent xmlns:Graphics="Microsoft.Xna.Framework.Content.Pipeline.Graphics">
  <Asset Type="Graphics:FontDescription">

    <!--
    Modify this string to change the font that will be imported.
    -->
    <FontName>Arial</FontName>

    <!--
    Size is a float value, measured in points. Modify this value to change
    the size of the font.
    -->
    <Size>12</Size>

    <!--
    Spacing is a float value, measured in pixels. Modify this value to change
    the amount of spacing in between characters.
    -->
    <Spacing>0</Spacing>

    <!--
    UseKerning controls the layout of the font. If this value is true, kerning information
    will be used when placing characters.
    -->
    <UseKerning>true</UseKerning>

    <!--
    Style controls the style of the font. Valid entries are "Regular", "Bold", "Italic",
    and "Bold, Italic", and are case sensitive.
    -->
    <Style>Regular</Style>

    <!--
    If you uncomment this line, the default character will be substituted if you draw
    or measure text that contains characters which were not included in the font.
    -->
    <!-- <DefaultCharacter>*</DefaultCharacter> -->

    <!--
    CharacterRegions control what letters are available in the font. Every
    character from Start to End will be built and made available for drawing. The
    default range is from 32, (ASCII space), to 126, ('~'), covering the basic Latin
    character set. The characters are ordered according to the Unicode standard.
    See the documentation for more information.
    -->
    <CharacterRegions>
      <CharacterRegion>
        <Start>&#32;</Start>
        <End>&#126;</End>
      </CharacterRegion>
    </CharacterRegions>
  </Asset>
</XnaContent>
```

> The selected Font needs to be installed on the development machine in order to build the Font.  I recommend you keep a copy of the Font file in the Content folder (not in the project) to ensure you always have it available.  The Last thing you want after rebuilding or moving machines it to have to hunt for the Fonts used in your project.

## Adding Text to your project

Now we have our content, the process for getting it into your game works very similar to images except we are importing **SpriteFont** data instead of Textures. Open the Game1.cs file add this variable to the *Properties* section of our code:

```csharp
    private SpriteFont _font;
```

A Texture2D object can handle an image file, in the same way a SpriteFont object can handle a .spritefont file. Load the content into the new variable in our LoadContent method:

```csharp
    _font = Content.Load<SpriteFont> ("myFont");
```

## Drawing Text

With the font loaded and the SpriteBatch already active, we are ready to draw some text. Once again, we will not put this directly in our Draw method, instead, we will create a separate method:

```csharp
    private void DrawText()
    {
        _spriteBatch.DrawString(_font, "Cannon power: 100", new Vector2(20, 45), Color.White);
    }
```

This line of code asks the SpriteBatch to render some text using our ‘font’ object. We specify the position, color and the text to draw, which in this case is the current power of the cannon. This will be rendered 20 pixels to the right and 45 down from the top-left corner of the screen.

Don’t forget to call this method from within our Draw method:

```csharp
    protected override void Draw(GameTime gameTime)
    {
        GraphicsDevice.Clear(Color.CornflowerBlue);

        // TODO: Add your drawing code here

        _spriteBatch.Begin();
        DrawScenery();
        DrawPlayers();
        DrawText();
        _spriteBatch.End();

        base.Draw(gameTime);
    }
```

Now when you run this code, you will see the current Power indicated at the top-left corner!

Obviously, the number printed to the screen does not correspond to the actual power when you change it because this is fixed text. Instead, change our **DrawText** method to this:

```csharp
    private void DrawText()
    {
        PlayerData player = _players[_currentPlayer];
        int currentAngle = (int)MathHelper.ToDegrees(player.Angle);
        _spriteBatch.DrawString(_font, "Cannon angle: " + currentAngle.ToString(), new Vector2(20, 20), player.Color);
        _spriteBatch.DrawString(_font, "Cannon power: " + player.Power.ToString(), new Vector2(20, 45), player.Color);
    }
```

This looks up the data of the current player in our PlayerData array and from this data, we use the Angle, Power and Color values while drawing the text to the screen.

That’s it! When you run the code you should see the screen below. Try to change the angle or the power: the values printed on the screen should change accordingly.

![MGCB-Editor Add Spritefont](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/2DXNA07Text03.png?raw=true)

## Exercises

You can try these exercises to practice what you have learned:

* Try changing the Font and size by editing the .spritefont file. (just make sure you have the font installed)
* Try adding some more text to show the players name for the current player. Will mean you need to also update the PlayerData struct.

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
        private SpriteFont _font;
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
    }
}
```

## Next Steps

[Launching the rocket](Riemers2DXNA08angletodirection)
