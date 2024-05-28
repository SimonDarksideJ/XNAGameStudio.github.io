# More powerful features of the SpriteBatch

The final image of the previous chapter showed 3 large flaws, which we will solve now:

* The carriages are drawn below the terrain, instead of on top of it.
* The carriages are far too big.
* They are grey.

## Fixing the Carriage draw position

We will start by resolving the first issue. When you use the SpriteBatch.Draw() method, MonoGame will render the top-left corner of the image at the position you specify. When you run the program again you can see the position the image is drawn starts from its top-left.

We can update the way we draw in two ways:

* We could specify the image should be drawn at a higher Y coordinate.
* We could specify the bottom-left corner should be put at the position we specify, instead of the default top-left corner.

In this case, we’re going for the latter. The *SpriteBatch.Draw()* method has a lot of different shapes (=overloads) that support more arguments. Replace the line of interest in the **DrawPlayers** method with this line:

```csharp
    _spriteBatch.Draw(_carriageTexture, _players[i].Position, null, Color.White, 0, new Vector2(0, _carriageTexture.Height), 1, SpriteEffects.None, 0);
```

Let’s discuss the multiple arguments in short:

1. The image to draw
2. The target screen pixel position
3. The third argument allows us to specify which part of the image should be drawn. This is useful in case you have stored multiple images inside one large image. By specifying null, we indicate we want to render the whole image.
4. The modulation color which we will be discussing below when dealing with each individual players color.
5. The fifth argument allows us to rotate the image before it is rendered to the screen, this will be used in the next chapter to render the cannon with the correct rotation.
6. **We’ll use the 6th argument to solve our positioning issue. The 6th argument allows us to specify what MonoGame calls the ‘origin’ of the image. By default, this is the top-left corner of the image. By setting this argument we will override the ‘origin’ point of the image.**
7. The 7th argument allows us to scale the image. We specify 1, indicating we want to render the image at its original size.
8. The 8th argument can be used to mirror the image horizontally of vertically before drawing it to the screen using the SpriteEffects enum.
9. The final argument can be used to specify the layer the image should be drawn on. This is useful for advanced games having multiple transparent images stacked on top of each other.

As the sixth argument, we are specifying the bottom-left corner of the image by specifying ***Vector2(0, carriageTexture.height)***. This position is indicated by the red dot in the image below. MonoGame will make sure this pixel is positioned at the position stored in player.Position.

![Gun Carriage](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/2DXNA04SpriteBatch01.jpg?raw=true)

Now when you run this code you should see the bottom-left corners of the carriage are better positioned!

## Scaling textures

Let’s move on and try to correctly scale the carriage. This is fairly easy, as we simply need to set the scaling factor as 7th argument of the **SpriteBatch.Begin** method. You can try to set for example 0.5f, which will cut the size in 2. However, since we will need this scaling factor later on throughout our code, we will store it as another variable in the *Properties* section of our code:

```csharp
    private float _playerScaling;
```

And then initialize it at the bottom of the LoadContent method:

```csharp
    _playerScaling = 40.0f / (float)_carriageTexture.Width;
```

Since the width of each flat area on the terrain is 40 pixels, this scaling factor should scale the carriage so it fits on the flat area.

Now we can use this new variable as the scaling argument of our SpriteBatch.Draw method in the **DrawPlayers** method:

```csharp
    _spriteBatch.Draw(_carriageTexture, _players[i].Position, null, Color.White, 0, new Vector2(0, _carriageTexture.Height), _playerScaling, SpriteEffects.None, 0);
```

Now when you run this code, the carriage will be scaled down so they fit on the flat areas, next we just need to add some color to them!

Or actually: remove some color, as in graphics programming the color ‘white’ is the combination of all colors together. Every possible color is made up of 3 components: the Red, Green and Blue (RGB) components. If you add all of them together, you get white. If you only mix red and blue together, you get purple. If you use none of them, you get black.

## Coloring the player sprites

The Color argument in the SpriteBatch.Draw() method is a useful tool to use before the image is rendered to the screen MonoGame looks up the color of each of the pixels of the image and applies the color you specified in the SpriteBatch.Draw() method. Next, the Red components of both colors are multiplied with each other, the same is true for both Green and Blue components. The resulting color is then rendered to the screen!

Let us say you want to render a white pixel to the screen which has full Red, Green and Blue components (RGB = 1,1,1). If you specified *Blue* as *Color* argument of the SpriteBatch, the Draw method having RGB = 0,0,1 would result in a color that is:

* Red 1 * 0
* Green 1 * 0
* Blue 1 * 1
* = 0,0,1 = Blue

A second example is that you want to render the color ***0.8,0.6,1*** to the screen and you have specified ***0.5,0.2,0.4*** as *Color* argument. 

The resulting color would be:

* Red 0.8 * 0.5
* Green 0.6 * 0.2
* Blue 1 * 0.4
* = 0.4, 0.12, 0.4

Sounds very complicated, but our example will show it is not. The image of our carriage contains only white or grey pixels. Now set the **Color** argument to **Color.Blue** like this:

```csharp
    _spriteBatch.Draw(_carriageTexture, _players[i].Position, null, Color.Blue, 0, new Vector2(0, _carriageTexture.Height), _playerScaling, SpriteEffects.None, 0);
```

Now if you run this code, you will notice the cannons have been rendered in **Blue**. As explained above, this is simply because the Red and Green color components have been stripped away from the original colors, and the completely white pixels have been replaced by fully blue pixels, while the grey pixels have been replaced by dark-blue pixels:

> Grey = 0.5,0.5,0.5 which becomes 0,0,0.5 = Dark Blue.

Now since we don’t want all our carriages to be rendered in blue, we retrieve the correct color from our PlayerData array:

```csharp
    _spriteBatch.Draw(_carriageTexture, _players[i].Position, null, _players[i].Color, 0, new Vector2(0, _carriageTexture.Height), _playerScaling, SpriteEffects.None, 0);
```

Now if you run this code, all of the carriages should be drawn at the correct position, scaled nicely and in the correct color!

![Summary](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/2DXNA04SpriteBatch1.png?raw=true)

## Exercises

You can try these exercises to practice what you have learned:

* Try to mirror the carriages horizontally by changing SpriteEffects.None in the SpriteBatch.Draw method.

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
                    _spriteBatch.Draw(_carriageTexture, _players[i].Position, null, _players[i].Color, 0, new Vector2(0, _carriageTexture.Height), _playerScaling, SpriteEffects.None, 0);
                }
            }
        }
    }
}
```

## Next Steps

[Rotation – Drawing the cannon](Riemers2DXNA05rotation)
