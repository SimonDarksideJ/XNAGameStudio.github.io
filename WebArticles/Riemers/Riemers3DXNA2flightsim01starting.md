# Initialization of our Project

Hi there! Glad you made it to this second series of the 3D MonoGame Tutorials. We are going to cover some new MonoGame features, put them together into one project, and end up with a real flight simulator! Again, the main goal is to show you some principles of MonoGame, so do not expect complete realistic flight physics, such as gravity, Coriolis, and others. This would add far too much maths, and draw our attention away from the MonoGame implementation.

The sole purpose of this first chapter is to set up our starting code. The code should be very simple to understand if you have finished the [first tutorial in the 3D series](Riemers2DXNAoverview). This is what the starting code does:

- Loading the effect
- Positioning the camera
- Clearing the window and Z buffer in the Draw method

So open a new MonoGame project (I used the DesktopGL template, but you can whichever suits your target platform) as described in [chapter one of the first series](Riemers3DXNA1Terrain01starting), I named my project "**Series3D2**". You are free to give your project a different name, but then you must replace the **namespace** of my code with your project name. This line is the first line under your using-block in my code.

If you have not done this already, download the assets for this series and add my [standard effect file](https://github.com/SimonDarksideJ/XNAGameStudio/raw/archive/Samples/Riemers/3D%20Series2%20-%20FlightSim%20-%20Assets.zip?raw=true), which you need to import into your MonoGame project as explained in ["The effect file"](Riemers3DXNA1Terrain02effect) in series 1. This effect file contains all techniques we are going to need in this second series.

> Remember, you will learn everything you need to know about effect files in the [third series](Riemers3DXNA3hlsloverview). For now, simply copy-paste the code below into your Game1.cs file.

Compiling and running the code should give you an empty window, cleared to a color of your choice by MonoGame:

![Starting](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA2-01Starting1.png?raw=true)

We are ready to start our second project.

## Code so far

```csharp
using Microsoft.Xna.Framework;
using Microsoft.Xna.Framework.Graphics;
using Microsoft.Xna.Framework.Input;

namespace Series3D2
{
    public class Game1 : Game
    {
        //Properties
        private GraphicsDeviceManager _graphics;
        private SpriteBatch _spriteBatch;
        private GraphicsDevice _device;
        private Effect _effect;

        private Matrix _viewMatrix;
        private Matrix _projectionMatrix;

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
            Window.Title = "Riemer's MonoGame Tutorials -- 3D Series 2";

            base.Initialize();
        }

        private void SetUpCamera()
        {
            _viewMatrix = Matrix.CreateLookAt(new Vector3(0, 0, 30), new Vector3(0, 0, 0), new Vector3(0, 1, 0));
            _projectionMatrix = Matrix.CreatePerspectiveFieldOfView(MathHelper.PiOver4, _device.Viewport.AspectRatio, 0.2f, 500.0f);
        }

        protected override void LoadContent()
        {
            _spriteBatch = new SpriteBatch(GraphicsDevice);

            // TODO: use this.Content to load your game content here
            _device = _graphics.GraphicsDevice;
            _effect = Content.Load<Effect>("effects");

            SetUpCamera();
        }

        protected override void Update(GameTime gameTime)
        {
            if (GamePad.GetState(PlayerIndex.One).Buttons.Back == ButtonState.Pressed || 
                Keyboard.GetState().IsKeyDown(Keys.Escape))
                Exit();

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

[Textures](Riemers3DXNA2flightsim02textures)
