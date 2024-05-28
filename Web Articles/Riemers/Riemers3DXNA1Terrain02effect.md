# The effect file

One of the main differences between DirectX with C++ and MonoGame is that we need an effect for everything we draw in 3D. So what exactly is an effect?

## Effect files 101

In 3D programming, all objects are represented using triangles. Even spheres can be represented using triangles if you use enough of them. An effect is some code that instructs your hardware (the graphics card) how it should display these triangles. An effect file contains one or more techniques, for example, technique A and technique B, drawing triangles using technique A will draw them semi-transparent, while drawing them using technique B will draw all objects using only blue-gray colors as seen in some horror movies.

Do not worry too much about this, as this is already more advanced stuff which we will handle in [Series 3](Riemers3DXNA3hlsloverview.md).

## Your first effect

MonoGame needs an effect file to draw even the simplest triangles, so I have written an effect file that contains some very basic techniques. If you have not done so already, you can download the [asset pack that contains the "effect.fx" file here](https://github.com/SimonDarksideJ/XNAGameStudio/raw/archive/Samples/Riemers/3D%20Series1%20-%20Terrain%20-%20Assets.zip?raw=true). Right-click on the link, and select "Save As". You should put the file on your hard drive, for example in the same folder as your code files.

Now that you have downloaded the effect file (effects.fx) to the same folder as your code files, we will import the file into our MonoGame project.

Open the "Content" project (Content.mgcb) contained in the Content folder of your project by double-clicking on it, which will open the **MonoGame Content Builder editor tool** (MGCB-Editor)

> If the editor does not open, please check the installation instructions on the [MonoGame documentation site here](https://docs.monogame.net/articles/tools/mgcb_editor.html). It should automatically be installed with Visual Studio but needs manual installation/registration when using .NET Core CLI.

Now select ***Edit-> Add -> Existing Item***, as shown in the image below.

![MGCB-Editor Add existing](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA1-02Effect1.png?raw=true)

In the window that opens, browse to the location where you saved the background image to and select the "effects.fx" file by clicking on it and clicking the **Open** button. You will be prompted whether you want to "Copy" or "Link" the content in your project, for now, select the **Copy** option and click **Add** to place the file inside your Content folder.

![mgcb-editor Add Content Item](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA1-02Effect2.png?raw=true)

If you select the file in the MGCB-Editor, you can see its properties listed below.

![mgcb-editor Add Content Item](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA1-02Effect3.png?raw=true)

## Adding effects into code

Next, we will link this effect file to a variable so that we can actually use it in our code. We will declare a new **Effect** object by adding this line at the beginning of your class: (in the code at the end of this section, you will notice I have added a comment called "Properties" to refer to this section of code)

```csharp
    private Effect _effect;
```

In your **LoadContent** method, add this line to have MonoGame load the .fx file into the effect variable for you:

```csharp
    _effect = Content.Load<Effect>("effects");
```

> The name **"effects"** refers to the part of the filename without the .fx extension, as shown in the properties window screenshot above.

## Applying an effect

With the necessary variable loaded, we can now concentrate on the **Draw** method. You will notice the first line starts with a Clear command, this line clears the buffer of our window to a specified color. Let us set this to DarkSlateBlue, just for fun:

```csharp
    _device.Clear(Color.DarkSlateBlue);
```

MonoGame uses a buffer to draw to instead of drawing directly to the window, at the end of the **Draw** method, the contents of the buffer is drawn on the screen all at once. In this way, the screen will not flicker as it would when we would draw each part of our scene separately to the screen.

Running this code will already give you the image you see below, but I would first like to add some additional code. As discussed above, to draw something, we first need to specify a **technique** from an **Effect** object, so we will immediately activate our effect so that in the next chapter we are ready to render something to the screen! 

Add this line to the **Draw** method right before the "base.Draw" call (which should always be last):

```csharp
    _effect.CurrentTechnique = _effect.Techniques["Pretransformed"];
```

You can see we have selected the **Pretransformed** technique from the effects.fx file, this technique will be used and discussed in the next chapter.

A technique can be made up of multiple passes, so we need to iterate through them by adding this code below the code you just entered:

```csharp
    foreach (EffectPass pass in _effect.CurrentTechnique.Passes)
    {
        pass.Apply();
    }
```

> All your drawing code must be put after your call to pass.Apply().

Finally, we are through the initialization part! If you’re not yet 100% clear on effects and techniques, there is no need to worry as we will discuss them in more detail in [Series 3](Riemers3DXNA3hlsloverview.md). With all of this code set up, we are finally ready to start drawing things on the screen, which is what we will do in the next chapter.

![Summary](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA1-02Effect4.png?raw=true)

## Exercises

You can try these exercises to practice what you have learned:

- No homework today

## The code so far

```csharp
using Microsoft.Xna.Framework;
using Microsoft.Xna.Framework.Graphics;
using Microsoft.Xna.Framework.Input;

namespace Series3D1
{
    public class Game1 : Game
    {
        //Properties
        private GraphicsDeviceManager _graphics;
        private GraphicsDevice _device;
        private Effect _effect;

        public Game1()
        {
            _graphics = new GraphicsDeviceManager(this);
            Content.RootDirectory = "Content";
            IsMouseVisible = true;
        }

        protected override void Initialize()
        {
            // TODO: Add your initialization logic here
            _graphics.PreferredBackBufferWidth = 500;
            _graphics.PreferredBackBufferHeight = 500;
            _graphics.IsFullScreen = false;
            _graphics.ApplyChanges();
            Window.Title = "Riemer's MonoGame Tutorials -- 3D Series 1";

            base.Initialize();
        }

        protected override void LoadContent()
        {
            // TODO: use this.Content to load your game content here
            _device = _graphics.GraphicsDevice;

            _effect = Content.Load<Effect>("effects");
        }

        protected override void Update(GameTime gameTime)
        {
            if (GamePad.GetState(PlayerIndex.One).Buttons.Back == ButtonState.Pressed || Keyboard.GetState().IsKeyDown(Keys.Escape))
                Exit();

            // TODO: Add your update logic here

            base.Update(gameTime);
        }

        protected override void Draw(GameTime gameTime)
        {
            _device.Clear(Color.DarkSlateBlue);

            // TODO: Add your drawing code here
            _effect.CurrentTechnique = _effect.Techniques["Pretransformed"];

            foreach (EffectPass pass in _effect.CurrentTechnique.Passes)
            {
                pass.Apply();
            }

            base.Draw(gameTime);
        }
    }
}
```

## Next Steps

[Drawing your first Triangle](Riemers3DXNA1Terrain03triangles)
