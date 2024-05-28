# Specifying the position of an image on the screen

In the last chapter we rendered 2 full-screen images. Obviously, to create a game we will also need to be able to render smaller images to the screen, at the position we define. That is what we’re going to do in this chapter.

This chapter, we are going to render the carriages and cannons for our players. Before we move on to the drawing code, we should define some data about our players, such as their position, color and whether they’re still alive or not. To store all this data nicely together, we should create a struct, which is nothing more than a container able to store some data we define.

## Defining players

Add this struct to the very top of our code (before the *Game1* class), immediately beneath the namespace definition (if you’re unsure where this is, have a look at the bottom of this page):

```csharp
    public struct PlayerData
    {
        public Vector2 Position;
        public bool IsAlive;
        public Color Color;
        public float Angle;
        public float Power;
    }
```

As you can see, an object of the type PlayerData will be able to store:

- A Position, which is a Vector2.
- A Vector2 can store 2 values, which we will use to store the X and Y coordinate of where we want to render the player’s carriage and cannon.
- A boolean (true/false) indicating whether the player is alive (and thus whether it should be drawn or not)
- The color we should draw the carriage and cannon in.
- Finally, for each player we keep track of the current angle of the cannon and of the power the cannonball will be shot with.

We will keep track of an array of **PlayerData** objects (the array can be thought of as a list of objects), allowing us to look at a certain object in the list, or make changes to an object in that list. So add these 2 variables to the **Properties** section in the code:

```csharp
    private PlayerData[] _players;
    private int _numberOfPlayers = 4;
```

When the code is finished, we want to be able to easily adjust the number of players, which the last variable allows.

## Setting up players

To give players options of being different colours, first, we will create a new pre-populated array of available Player colours by adding the following property to the **Properties** section of the code:

```csharp
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
```

Next create a new "SetUpPlayers" method which initializes our array of PlayerData objects, placed after the Initialize method:

```csharp
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
    }
```

We start by defining an array of empty PlayerData objects since I defined numberOfPlayers to be 4, this line will create an array holding 4 empty PlayerData objects.

The for loop then scrolls through each PlayerData object in the array, for each of them we set **IsAlive** to true and assign it a color from the PlayerColors array we defined earlier.

Since the image of the cannon is vertically upwards and we want to start our game with all cannons lying vertically to the right, we need to rotate them over 90 degrees. (will use this later in the series in [Rotation - Drawing the cannon](Riemers2DXNA05rotation)) Finally, we set the initial cannon power to 100.

> In game programming, and thus also in MonoGame, all angles are defined in [radians](http://en.wikipedia.org/wiki/Radian).  This might sound difficult, but if you know that pi (=3.14) radians correspond to 180 degrees, you can find how many radians correspond to any given number of degrees. 
>
> Even better, MonoGame does the calculations for us: we just need to pass in the number of degrees to the MathHelper.ToRadians method and MonoGame will convert it to radians for us.
>
> 90 Degrees corresponds to pi/2, so MathHelper.ToRadians(90) is the same as MathHelper.PiOver2 as used in our code.

There is one important thing we still need to initialize, the position of players on the terrain. As you can see, the foreground image has 4 flat areas, perfectly suited to place a cannon. When we will create a new foreground image ourselves in a later chapter, for now, we know the position of these flat areas and will have to code them by hand.

These are the 4 positions of the 4 flat areas of the terrain (put this code at the end of the **SetUpPlayers** method):

```csharp
    _players[0].Position = new Vector2(100, 193);
    _players[1].Position = new Vector2(200, 212);
    _players[2].Position = new Vector2(300, 361);
    _players[3].Position = new Vector2(400, 164);
```

Now let us not forget to call this method from the end of the **LoadContent** method:

```csharp
    SetUpPlayers();
```

## Adding player content

Now all the data is known about our players, it is time to render them to the screen. The first 3 steps are exactly the same as in the previous chapter:

1. Import the **carriage.png** and **cannon.png** images into the Content project for your solution (you can select multiple images at the same time when importing for convenience).

2. Add the following variables to the **Properties** section of our code:

> Personally, I prefer to group together common properties, so I have placed these next to the other Texture2D variables)

```csharp
    private Texture2D _carriageTexture;
    private Texture2D _cannonTexture;
```

3. Initialize the variables in our **LoadContent** method:

```csharp
    _carriageTexture = Content.Load<Texture2D>("carriage");
    _cannonTexture = Content.Load<Texture2D>("cannon");
```

4. Finally, we should add these images to the SpriteBatch. To keep our Draw method clean, we will again create a new **DrawPlayers** method placed after the *DrawScenery* method:

```csharp
    private void DrawPlayers()
    {
        for (int i = 0; i < _players.Length; i++)
        {
            if (_players[i].IsAlive)
            {
                _spriteBatch.Draw(_carriageTexture, _players[i].Position, Color.White);
            }
        }
    }
```

> Separating different groups of content such as Scenery and Players in to different methods makes managing your code much easier in the future.

Now we must ensure to call this **DrawPlayers** method from our **Draw** method. Since we’re adding images to the SpriteBatch, we need to do this between the calls to SpriteBatch.Begin() and SpriteBatch.End() methods. Because the background image is fully opaque (=not-transparent), we need to do this after the call to the **DrawScenery** method so that our carriages are rendered on top of the scenery and not the other way around:

```csharp
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
```

Now try to run this code! You should see the image below:

![Summary](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/2DXNA03Position1.png?raw=true)

While the X coordinates of the carriages seem to be OK, you should notice 3 large errors:

- The carriages are drawn below the terrain, instead of on top of it.
- The carriages are far too big.
- They are grey.

We will solve these issues in the next chapter.

## Exercises

You can try these exercises to practice what you have learned:

- Try adding more players and positioning them (do not forget to increase the numberOfPlayers variable)
- Add different players with their own images, hint, odd players use carriages, even players use tanks?

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
                    _spriteBatch.Draw(_carriageTexture, _players[i].Position, Color.White);
                }
            }
        }
    }
}
```

## Next Steps

[More powerful features of the SpriteBatch](Riemers2DXNA04spritebatch)
