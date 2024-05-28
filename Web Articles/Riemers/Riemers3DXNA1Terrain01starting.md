# Starting your MonoGame 4.0 Project

Welcome to the first entry of this 3D MonoGame Tutorial. This tutorial is aimed at people who have not done any 3D programming and would like to see some results in the shortest possible time.

## The history of XNA/MonoGame

Released in December 2004, XNA was a new approach to Game Development built around DirectX in C# (a managed language), which eased game programming in a lot of ways.  MonoGame then took the reigns in 2013 when Microsoft retired XNA, to continue the tradition of making the entry into game development very easy. Since that time MonoGame has grown to become one of the premier C# development frameworks for building games and has given many studios their first entries to the gaming halls of fame.

## Required Software (free)

The software required to start writing your own MonoGame code is completely free to download, please refer to the MonoGame "[Setting up your development environment guide](https://docs.monogame.net/articles/getting_started/0_getting_started.html#setting-up-your-development-environment)" for your respective operating system:

- [Windows](https://docs.monogame.net/articles/getting_started/1_setting_up_your_development_environment_windows.html)
- [macOS](https://docs.monogame.net/articles/getting_started/1_setting_up_your_development_environment_macos.html)
- [Ubuntu](https://docs.monogame.net/articles/getting_started/1_setting_up_your_development_environment_ubuntu.html)

> For more details consult the [MonoGame Getting Started Guide](https://docs.monogame.net/articles/getting_started/0_getting_started.html)

## Starting a new MonoGame Project

With the release of MonoGame 3.8, there are now multiple ways to [create your new MonoGame project](https://docs.monogame.net/articles/getting_started/0_getting_started.html#creating-a-new-project) depending on the operating system you are developing on:

- [Visual Studio 2019](https://docs.monogame.net/articles/getting_started/2_creating_a_new_project_vs.html)
- [Visual Studio for Mac](https://docs.monogame.net/articles/getting_started/2_creating_a_new_project_vsm.html)
- [DotNet core CLI](https://docs.monogame.net/articles/getting_started/2_creating_a_new_project_netcore.html)

Once your project is generated you will see is contains 2 code files:

- Game1.cs
- Program.cs

You can look at the code in the files if you wish by opening them, there you will see the template code for a blank MonoGame project. When you run your program later, your program will start in the Program.cs file and call the "Main" method. This "Main" method simply calls up the "Game" code in the Game1.cs file. There’s nothing we need to change in the Program.cs file.

## Program structure

Open the Game1.cs code file. you will find it is littered with comments in green (feel free to remove them), within the template code we can discover the basic structure of a MonoGame game program:

- The constructor method Game1() is called once at startup. It is used to load some variables needed by the MonoGame framework.
- The Initialize method is also called once on startup. This is the method where we should put our initialization code.
- The LoadContent method is used for importing media (such as images, objects, and audio) as well as data related to the graphics card. In the Game class, this is only called once on startup.
- The Update method is called once every frame, at a rate of exactly 60 times/second. This is where we will put the code that needs to be updated throughout the lifetime of our program, such as the code that reads the keyboard and updates the geometry of our scene.
- As often as your computer (and especially your graphics card) allows, the Draw method is called. This is where we should put the code that actually draws stuff to the screen.

As you can see, there is no code needed to open a window, as this will be done automatically for us when you run your project. if you are running Visual Studio (Windows/Mac) you can press F5 to run your project and will get a nice blue window.

## Setting up the screen

Let us move on, and discuss the graphics device. In short, the ‘device’ is what I will be talking about in the next series of tutorials, this a direct link to your graphical adapter. It is an object in your code that gives you direct access to the piece of hardware inside your computer that controls what you see on your screen. This variable is readily available in our code as well as the GraphicsDevice variable, but we will be using this a lot (really, a lot) and we will make a shortcut for this. First, we’ll declare this variable, by adding this line to the top of our class (Commented as **"Properties"** for easy reference in the code below), exactly above the **Game1()** method:

```csharp
    private GraphicsDevice _device;
```

Obviously, we need to fill this variable. Add this line to your LoadContent method:

```csharp
    _device = _graphics.GraphicsDevice;
```

Let us put it to use, by making the first line in your Draw method a tiny bit shorter and updating it to:

```csharp
    _device.Clear(Color.CornflowerBlue);
```

Next, we are going to specify some extra stuff related to our window such as its size and title. Add this code to the Initialize method:

```csharp
    _graphics.PreferredBackBufferWidth = 500;
    _graphics.PreferredBackBufferHeight = 500;
    _graphics.IsFullScreen = false;
    _graphics.ApplyChanges();
    Window.Title = "Riemer's MonoGame Tutorials -- 3D Series 1";
```

The first line sets the size of our backbuffer which will contain what will be drawn to the screen. We also indicate that we want our program to run in a window. The last line sets the title of our window.

When you run this code, you should see a window of 500x500 pixels, with the title you set, as shown below.

![Blank Screen](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/2DXNA01Starting01.png?raw=true)

After each chapter I will suggest some short exercises, so you practice what you have learned in the chapter. After the exercise, the whole code of the chapter is listed.
As we are focusing purely on 3D for this tutorial, the code surrounding the SpriteBatch has been removed as it is used only to render 2D graphics and we will not need it.

> **Important:** When you copy-paste the code into your Game.cs file, make sure you change the name of the **namespace** in your Program.cs file to **Series3D1**. The namespace by default is the same as the name you specified when creating the new project, so in my case it is Series3D1. You can find the namespace immediately below the using-block at the top of the code file.

## Exercises

You can try these exercises to practice what you have learned:

- Change the size of your window to 800x600.
- Instead of creating a windowed game, switch to full-screen mode (use Alt+F4 to terminate your program).

## Starting code

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
            Window.Title = "Riemer's MonoGame Tutorials -- 3D Series 1";

            base.Initialize();
        }

        protected override void LoadContent()
        {
            // TODO: use this.Content to load your game content here
            _device = _graphics.GraphicsDevice;
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

            base.Draw(gameTime);
        }
    }
}
```

## Next Steps

[The effect file](Riemers3DXNA1Terrain02effect)
