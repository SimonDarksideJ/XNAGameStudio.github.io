# World Space coordinates and the camera

In the last chapter, we drew a triangle using 'pretransformed' coordinates which are referred to as "Screen Space" coordinates, these coordinates allow you to directly specify their position on the screen, however, you more likely to use 'untransformed' coordinates which are referred to as "World space" coordinates. These allow you to create a whole scene using simple 3D coordinates, and importantly, to position a camera through which the user will look at the scene.

Let us start by redefining our triangle coordinates in 3D World space. Replace the code in our SetUpVertices method with this code:

```csharp
    private void SetUpVertices()
    {
        _vertices = new VertexPositionColor[3];

        _vertices[0].Position = new Vector3(0f, 0f, 0f);
        _vertices[0].Color = Color.Red;
        _vertices[1].Position = new Vector3(10f, 10f, 0f);
        _vertices[1].Color = Color.Yellow;
        _vertices[2].Position = new Vector3(10f, 0f, -5f);
        _vertices[2].Color = Color.Green;
    }
```

As you can see, from here on we will be using the Z-coordinate as well.

Because we are no longer using 'pretransformed' screen coordinates (where x and y coordinates should be in the [-1, 1] region), we need to select a different technique from our effect file. I created a new technique called ‘ColoredNoShading’ to reflect that we will be rendering a Colored 3D image, but did not yet specify any lighting/shading information.

So make this adjustment in our Draw method:

```csharp
    _effect.CurrentTechnique = _effect.Techniques["ColoredNoShading"];
```

When you run this code, you will find that your triangle has disappeared. Why is that? Easy, because you have not told MonoGame yet where to position the camera in your 3D World and where to look at!

## Preparing the camera

To position our camera, we need to define some matrices. **Stop!! matrices?!?**

First, a small word about matrices, we define our points in 3D space because our content is in 3D, however, our screen is in 2D, so logically we need a method that transforms our 3D points somehow into 2D space. This is done by multiplying our 3D positions with a matrix, if you multiply a 3D position with such a matrix, you get the transformed position that can be rendered on screen. (If you want to know more about matrices, you can find more info in the [Extra Reading section](https://github.com/SimonDarksideJ/XNAGameStudio/wiki#as-well-as-several-helpful-short-articles) of this site)

Because there are a lot of properties that need to be defined when transforming our points from 3D world space to our 2D screen, we split this transformation into two steps, first, we define two matrices by adding these variables to the properties section of your class:

```csharp
    private Matrix _viewMatrix;
    private Matrix _projectionMatrix;
```

And then initialize them in this new method:

```csharp
    private void SetUpCamera()
    {
        _viewMatrix = Matrix.CreateLookAt(new Vector3(0, 0, 50), new Vector3(0, 0, 0), new Vector3(0, 1, 0));
        _projectionMatrix = Matrix.CreatePerspectiveFieldOfView(MathHelper.PiOver4, _device.Viewport.AspectRatio, 1.0f, 300.0f);
    }
```

These 2 lines may seem complicated, but all they do is defining the position and the lens of the camera.

![Camera Frustrum](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA1-04Worldspace1.png?raw=true)

The first line creates a matrix that stores the position and orientation of the camera, through which we look at the scene.

* The first argument defines the position of the camera. We position it 50 units on the positive Z axis.
* The next parameter sets the target point the camera is looking at. We will be looking at our (0,0,0) 3D origin.
* Finally, we need to define which direction is "UP", this is crucial as we can still rotate our camera around this axis.

The second line creates a matrix that stores how the camera looks at the scene, much like defining the lens if you will.

* The first argument sets the view angle, 45 degrees in our case.
* Then we set the view aspect ratio, the ratio between the width and height of your screen. In our case of a 500x500 window, this will equal 1, but this will be different for other resolutions.
* The last parameters define the view range. Any objects closer to the camera than 1f will not be shown. Any object further away than 300.0f will not be shown either. These distances are called the near and the far clipping planes since all objects outside of these planes will be clipped (=not drawn).

Now that we have these matrices, we need to pass it to our technique, where they will be combined. This is done by the next lines of code, which we need to add to our Draw method immediately after the existing "_effect.CurrentTechnique" line:

```csharp
    _effect.Parameters["xView"].SetValue(_viewMatrix);
    _effect.Parameters["xProjection"].SetValue(_projectionMatrix);
    _effect.Parameters["xWorld"].SetValue(Matrix.Identity);
```

Although the first 2 lines are explained above, they are discussed in much more detail in [Series 3](Riemers3DXNA3hlsloverview.md). The third line sets another parameter, which is discussed in the next chapter.

Do not forget to call this method in the **LoadContent** method, placing it just after loading the effect file (since we want the camera setup early):

```csharp
    SetUpCamera();
```

Now run the code. You should see the image below: a triangle, of which the bottom-right corner is not exactly below the top-right corner. This is because you have assigned the bottom-right corner a negative Z coordinate, positioning it a bit further away from the camera than the other corners.

## Culling and winding orders

One important thing you should notice before you start experimenting, you will see the green corner of the triangle is on the right side of the window, which seems normal because you defined it on the positive x-axis. So, if you would position your camera on the negative z-axis:

```csharp
    _viewMatrix = Matrix.CreateLookAt(new Vector3(0, 0, -50), new Vector3(0, 0, 0), new Vector3(0, 1, 0));
```

You would expect to see the green point in the left half of the window. Try to run this now.

This might not be exactly what you expected. Something very important has happened.  MonoGame only draws triangles that are facing the camera and triangles that are facing the camera should be defined clockwise relative to the camera. If you position the camera on the negative z-axis, the corner points of the triangle in our vertices array will be defined counter-clockwise relative to the camera, and thus will not be drawn!

Culling can greatly improve performance and the number of triangles to be drawn. However, when designing an application, it is better to turn culling off by putting these lines of code in the beginning of your Draw method:

```csharp
    RasterizerState rs = new RasterizerState();
    rs.CullMode = CullMode.None;
    device.RasterizerState = rs;
```

This will simply draw all triangles, even those not facing the camera. You should note that this **should never be done in a final product** because it slows down the drawing process as all triangles will be drawn, even those not facing the camera! (This should only be used in those cases where you intend to achieve this effect). For now, put the camera back to the positive part of the Z axis.

![Summary](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA1-04Worldspace2.png?raw=true)

## Exercises

You can try these exercises to practice what you have learned:

* Experiment with different positions for your 3D vertices and camera position.
* (advanced) If you have read the 2D section on keyboard input, try adding some controls to "Move" the camera.

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
        private VertexPositionColor[] _vertices;
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
            Window.Title = "Riemer's MonoGame Tutorials -- 3D Series 1";

            base.Initialize();
        }

        private void SetUpVertices()
        {
            _vertices = new VertexPositionColor[3];

            _vertices[0].Position = new Vector3(0f, 0f, 0f);
            _vertices[0].Color = Color.Red;
            _vertices[1].Position = new Vector3(10f, 10f, 0f);
            _vertices[1].Color = Color.Yellow;
            _vertices[2].Position = new Vector3(10f, 0f, -5f);
            _vertices[2].Color = Color.Green;
        }

        private void SetUpCamera()
        {
            _viewMatrix = Matrix.CreateLookAt(new Vector3(0, 0, 50), new Vector3(0, 0, 0), new Vector3(0, 1, 0));
            _projectionMatrix = Matrix.CreatePerspectiveFieldOfView(MathHelper.PiOver4, _device.Viewport.AspectRatio, 1.0f, 300.0f);
        }

        protected override void LoadContent()
        {
            // TODO: use this.Content to load your game content here
            _device = _graphics.GraphicsDevice;

            _effect = Content.Load<Effect>("effects");

            SetUpCamera();

            SetUpVertices();
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

            RasterizerState rs = new RasterizerState();
            rs.CullMode = CullMode.None;
            _device.RasterizerState = rs;

            // TODO: Add your drawing code here
            _effect.CurrentTechnique = _effect.Techniques["ColoredNoShading"];
            _effect.Parameters["xView"].SetValue(_viewMatrix);
            _effect.Parameters["xProjection"].SetValue(_projectionMatrix);
            _effect.Parameters["xWorld"].SetValue(Matrix.Identity);

            foreach (EffectPass pass in _effect.CurrentTechnique.Passes)
            {
                pass.Apply();
                _device.DrawUserPrimitives(PrimitiveType.TriangleList, _vertices, 0, 1, VertexPositionColor.VertexDeclaration);
            }

            base.Draw(gameTime);
        }
    }
}
```

## Next Steps

[Rotations and translations](Riemers3DXNA1Terrain05rotation)
