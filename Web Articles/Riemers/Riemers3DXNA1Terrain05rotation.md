# Rotations and translations

In this chapter, we will make our triangle spin around (exciting stuff) and because we are using world space coordinates this is very easy to do.

## Make me spin

Let us first add a variable **angle** to our class to store the current rotation angle. Just add this to the Properties section of the class.

```csharp
    private float _angle = 0f;
```

Now, we should increase this variable with 0.05f every frame. The Update method is an excellent place to put this code, as it is called 60 times each second:

```csharp
    _angle += 0.005f;
```

With our angle increasing automatically, all we have to do now is to rotate the world coordinates for the triangle. I hope you remember from your math class this is done using transformation matrices ;) Luckily, all you have to do is specify the rotation axis and the rotation angle. All the rest is done by MonoGame!

The rotation is stored in what is called the **World** matrix. In your **Draw** method, replace the line where you set your xWorld parameter with this code:

```csharp
    Matrix worldMatrix = Matrix.CreateRotationY(3 * _angle);
    _effect.Parameters["xWorld"].SetValue(worldMatrix);
```

The first line creates our World matrix, which holds a rotation around the Y axis. The second line passes this World matrix to the effect, which it needs to perform its job. From now on, everything we draw will be rotated along the Y axis by the amount currently stored in 'angle'!

## Moving 3D content as well as rotating

When you run the application, you will see that your triangle is spinning around its (0,0,0) point! This is of course because the Y axis runs through this point, so the (0,0,0) point is the only point of our triangle that remains the same. Now imagine we would like to spin it through the center of the triangle. One possibility is to redefine the triangle so the (0,0,0) would be in the center of our triangle, but the better solution would be to first move (=translate) the triangle a bit to the left and down, and then rotate it.

To do this, simply first multiply your World matrix with a new translation matrix (updating the line you just added):

```csharp
    Matrix worldMatrix = Matrix.CreateTranslation(-20.0f/3.0f, -10.0f / 3.0f, 0) * Matrix.CreateRotationZ(_angle);
```

This will move the triangle so its center point is in our (0,0,0) 3D World origin. Next, our triangle is rotated around this point, along the Z axis, giving us the desired result.

> **Note the order of transformations**. Go ahead and place the translation AFTER the rotation. You will see a triangle rotating around one point, moved to the left and below. This is because in matrix multiplications **M1*M2 is NOT the same as M2*M1**!

You can easily change the code to make the triangle rotate around the X or Y axis. Remember that one point of our triangle has a Z coordinate of -5, which explains why the triangle won’t seem to rotate symmetrically sometimes.

## Stepping up the translation

A bit more complex is the Matrix CreateFromAxisAngle, where you can specify your own custom rotation axis :

```csharp
 Vector3 rotAxis = new Vector3(3*angle, angle, 2*angle);
 rotAxis.Normalize();
 Matrix worldMatrix = Matrix.CreateTranslation(-20.0f/3.0f, -10.0f / 3.0f, 0) * Matrix.CreateFromAxisAngle(rotAxis, angle);
```

This will make our triangle spin around an ever-changing axis. The first line defines this axis (which is changed every frame as it depends on the angle variable). The second line normalizes this axis, which is needed to make the CreateFromAxisAngle method work properly (Normalize() changes the coordinates of the vector, so the distance between the vector and the (0, 0, 0) point is exactly 1).

![Summary](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA1-05Rotation1.gif?raw=true)

## Exercises

You can try these exercises to practice what you have learned:

* Try to rotate your triangle 180 degrees over its bottom edge.
* Try adding a second triangle and rotating it the opposite way. (hint, you will need to duplicate "some" code in a separate draw loop)

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
        private float _angle = 0f;

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
            _angle += 0.005f;

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
            Matrix worldMatrix = Matrix.CreateTranslation(-20.0f / 3.0f, -10.0f / 3.0f, 0) * Matrix.CreateRotationZ(_angle);
            _effect.Parameters["xWorld"].SetValue(worldMatrix);

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

[Recycling vertices using indices](Riemers3DXNA1Terrain06indices)
