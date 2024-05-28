# Drawing a full-screen background image

The first step in creating a 2D game would be to render a 2D image. In MonoGame, this is very easy to do. As a first example, we’re going to render 2 full-screen images to the screen. The first will be the background image, containing the mountain scene. On top of that, we’ll render the foreground image, containing the terrain.

## Adding Content

Let’s start by downloading the below files, you can find them by clicking on this link and saving the asset and then extract the contents. Make sure you know the location where you save it.

- [MonoGame 2D series assets](https://github.com/SimonDarksideJ/XNAGameStudio/raw/archive/Samples/Riemers/2D%20Series%20-%20Shooters%20-%20Assets.zip)

Next open the "Content" project (Content.mgcb) contained in the Content folder of your project by double-clicking on it, which will open the **MonoGame Content Builder editor tool** (MGCB-Editor)

> If the editor does not open, please check the installation instructions on the [MonoGame documentation site here](https://docs.monogame.net/articles/tools/mgcb_editor.html). It should automatically be installed with Visual Studio but needs manual installation/registration when using .NET Core CLI.

Now select ***Edit-> Add -> Existing Item***, as shown in the image below.

![mgcb-editor Add Content Item](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/2DXNA02Background01.png?raw=true)

In the window that opens, browse to the location where you saved the background image to and select the "Background.jpg" image file by clicking on it and clicking the **Open** button. You will be prompted whether you want to "Copy" or "Link" the content in your project, for now, select the **Copy** option and click **Add** to place the file inside your Content folder.

![mgcb-editor Add Content Item](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/2DXNA02Background02.png?raw=true)

If you select the file in the MGCB-Editor, you can see its properties listed below.

![mgcb-editor Add Content Item](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/2DXNA02Background03.png?raw=true)

## Adding images into code

With the image loaded into our MonoGame project, we can now create a variable in our code that we will link to the image. The variable is required, so we can access the image from within our code.

Add this variable to the Properties section at the top of our code:

```csharp
    private Texture2D _backgroundTexture;
```

The variable is of the type Texture2D which is an image type. An image is called a Texture or a Sprite in game programming, so that’s where the name comes from. A variable of the Texture2D type can store image data, and can as such be used to render the image or access the color data inside the image (as we’ll do in one of the next chapters).

Obviously, we need to import the image file we just added into our MonoGame Content project. This is done by putting the following line in our LoadContent method:

```csharp
    _backgroundTexture = Content.Load<Texture2D> ("background");
```

This easy line takes care of loading the image from disk and copying the image into memory. The “background” argument specified at the end, points to the **name** of the file without the extension (as shown in the **Properties** screenshot above), which by default is "background" (for background.jpg).

> You can change the **Name** of the asset to anything you like in the mgcb-editor, but it is best to leave it as the default.  Whatever the name is, that is the reference you need to specify in your Content.Load call.

It doesn’t matter whether your image is in the JPG, BMP, PNG, DDS or any other image format as MonoGame handles the type automatically. Just import the image into your MonoGame project and link it to a variable, what matters is that when you call "Content.Load" you have specify the type of data you are loading, in this case, a **Texture2D**.

## Handling screen size rendering

With our variable populated with the image data, we are ready to draw the image to the screen. MonoGame provides a very easy, and very powerful tool for this: the **SpriteBatch**. This is also a variable, which is already present in your code the moment you start a new MonoGame project. Find it in your code: it is defined at the top of your code, and initialized in your LoadContent method.

The **SpriteBatch** is what its name implies: it allows us to render a collection of sprites/2D images to the screen all at once (which saves on calls to the graphics card for performance). For now, we will only render one image, but we can use a single SpriteBatch to render a huge amount of images. Best of all, this is done using the hardware acceleration of your video card!

Since the spriteBatch object is already initialized we can immediately start drawing our image. This should be done in the Draw method. To organize our code a little better, we will create a new method for this. The main reason for this is because the Draw method would look very cluttered at the end of this series of tutorials when drawing lots of images.

To keep things a bit clean, add this new empty method to the end of our code, immediately below the Draw method:

```csharp
    private void DrawScenery()
    {
    }
```

When we want to draw a 2D image to the screen, we basically have 2 options:

1. Specify the position of the top-left corner of the image - Or
2. Specify a rectangle on the screen where we want the image to be rendered into. In case the size of your image doesn’t match the size of the rectangle, MonoGame will take care that the image is scaled so it fits in perfectly.

Since we want our background image to cover the whole screen, we will go for the second option. We will use the first option in the next chapter.

When we want to specify the rectangle that corresponds to the whole screen, we need to know the width and height of the screen. Obviously, since we set the ***screensize*** to 500*500 pixels, we already know the size. But of course, we want our game to adapt itself automatically whenever we change the ***screensize*** later on.

Since we will be using the width and height of our screen a lot throughout this series, we will add them as 2 variables to the **Properties** section of our code:

```csharp
    private int _screenWidth;
    private int _screenHeight;
```

Next, put the following code at the end of our **LoadContent** method, which stores the correct values into these variables:

```csharp
    _screenWidth = _device.PresentationParameters.BackBufferWidth;
    _screenHeight = _device.PresentationParameters.BackBufferHeight;
```

> We do this in **LoadContent** and not Initialize because LoadContent is called AFTER the game has started and we know the actual drawn dimensions on screen.

## Drawing using a SpriteBatch

With the size of our screen readily available, we can go back to our **DrawScenery** method and define a rectangle that covers the entire window:

```csharp
    private void DrawScenery()
    {
        Rectangle screenRectangle = new Rectangle(0, 0, _screenWidth, _screenHeight);
        _spriteBatch.Draw(_backgroundTexture, screenRectangle, Color.White);
    }
```

The first line creates a rectangle, with its upper-left corner at the (0,0) pixel and having a width and height equal to the width and height of the screen.

> **Important:** in graphics programming, the (0,0) point corresponds to the top-left corner of the screen. The positive **X direction** is to the **right**, while the positive **Y direction** is **down**.

The last line asks the SpriteBatch to draw the backgroundTexture image, spanning the entire window. We will discuss the last argument in the next chapter, for now, understand that **Color.White** means that the image should be rendered in its original colors. The code line adds the background image to the list of images the SpriteBatch has to draw (a list of only 1 image).

This will, however, not immediately draw the image to the screen. To make a SpriteBatch work:

1. First, you need to start the Batch, specifying any additional arguments you need to setup the batch (we are just using the default for now)
2. Next, you need to add images to the Batch.  

> **Importantly** understand that the order you add them the batch is also the order they will be drawn to the screen, from back to front.

3. Finally, you should **End** the SpriteBatch to make it render its images.

Go to our Draw method, and update it as follows:

```csharp
    protected override void Draw(GameTime gameTime)
    {
        GraphicsDevice.Clear(Color.CornflowerBlue);

        // TODO: Add your drawing code here

        _spriteBatch.Begin();
        DrawScenery();
        _spriteBatch.End();

        base.Draw(gameTime);
    }
```

- The first line clears the window to a color of our choice. 
- Next, we start our SpriteBatch, so we can add images to its list of images to draw.
- Then we call our DrawScenery method, which adds one image to the list of the SpriteBatch.
- Finally, we End the SpriteBatch, which effectively asks the SpriteBatch to draw the images in its list to the screen.
- The last line calls the Draw method of any GameComponents you’ve attached to your Game.

When you run this code, the background image should be shown, nicely stretched so it perfectly fits your window! Also when you set a different window size, the image will adapt itself automatically.

## Adding the Foreground

With the background image in place, let’s render the foreground to the screen. In a later chapter, we’ll create the terrain dynamically so we get a new terrain each time we start our program. For now, we will simply load one I saved to file. Start by going through the same 3 steps:

1. Import the foreground.png image into the Content Project, repeating the previous steps for the Background image

2. Add the following variable to the top of your code:

```csharp
    private Texture2D _foregroundTexture;
```

3. Initialize it on your LoadContent method:

```csharp
    _foregroundTexture = Content.Load<Texture2D>("foreground");
```

To render this image on top of our background, simply add the following line to the end of our **DrawScenery** method:

```csharp
    _spriteBatch.Draw(_foregroundTexture, screenRectangle, Color.White);
```

Which adds the foregroundTexture to the SpriteBatch and draws it covering the entire screen as with the background, in its original colors.

When you open the foreground.png image in windows explorer, you see that the part above the terrain is transparent, this is why I’ve used the PNG format instead of the JPG format (The JPG format cannot store transparency information). Since we’ve first added the background image to the list of the SpriteBatch, and then the foreground image, the background image will be drawn first with the foreground on top.

This is what you should see on your screen:

![mgcb-editor Add Content Item](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/2DXNA02Background04.png?raw=true)

## Exercises

You can try these exercises to practice what you have learned:

- Try changing the order of the images drawn and watch your foreground disappear.
- Play with adding more images on top of each other (just remove them before the next exercise)

## The code so far

```csharp
using Microsoft.Xna.Framework;
using Microsoft.Xna.Framework.Graphics;
using Microsoft.Xna.Framework.Input;

namespace Series2D1
{
    public class Game1 : Game
    {
        //Properties
        private GraphicsDeviceManager _graphics;
        private SpriteBatch _spriteBatch;
        private GraphicsDevice _device;
        private Texture2D _backgroundTexture;
        private Texture2D _foregroundTexture;
        private int _screenWidth;
        private int _screenHeight;

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

        protected override void LoadContent()
        {
            _spriteBatch = new SpriteBatch(GraphicsDevice);
            _device = _graphics.GraphicsDevice;

            // TODO: use this.Content to load your game content here
            _backgroundTexture = Content.Load<Texture2D>("background");
            _foregroundTexture = Content.Load<Texture2D>("foreground");

            _screenWidth = _device.PresentationParameters.BackBufferWidth;
            _screenHeight = _device.PresentationParameters.BackBufferHeight;
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
            _spriteBatch.End();

            base.Draw(gameTime);
        }

        private void DrawScenery()
        {
            Rectangle screenRectangle = new Rectangle(0, 0, _screenWidth, _screenHeight);
            _spriteBatch.Draw(_backgroundTexture, screenRectangle, Color.White);
            _spriteBatch.Draw(_foregroundTexture, screenRectangle, Color.White);
        }
    }
}
```

## Next Steps

[Specifying the position of an image on the screen](Riemers2DXNA03imageposition)
