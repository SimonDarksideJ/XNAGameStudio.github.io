# Recycling vertices using indices

The triangle was nice, but what about a lot of triangles?

## Making a mountain out of a molehill

To make more triangles, we simply would need to specify 3 vertices for every triangle we need. Consider the following example:

![Triangles](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA1-6indices1.jpg?raw=true)

Only 4 out of 6 vertices are unique. So the other 2 are simply a waste of bandwidth to your graphics card! It would be better to define the 4 vertices in an array from 0 to 3, and to define triangle one as vertices 1, 2, and 3 and then triangle two as vertices 2, 3, and 4. This way, the complex vertex data is not duplicated. 

This is exactly the idea behind indices. Suppose we would like to draw these 2 triangles :

![Triangles](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA1-6indices2.jpg?raw=true)

Normally we would have to define 6 vertices, now we will define only 5. This might not sound like much of a cost-saving, but consider if we had MILLIONS of triangles, it all adds up.

To get started, change our SetUpVertices method as follows (I have compacted the code to make it more manageable):

```csharp
    private void SetUpVertices()
    {
        _vertices = new VertexPositionColor[5]
        {
        new VertexPositionColor() {Position = new Vector3(0f, 0f, 0f), Color = Color.White},
        new VertexPositionColor() {Position = new Vector3(5f, 0f, 0f), Color = Color.White},
        new VertexPositionColor() {Position = new Vector3(10f, 0f, 0f), Color = Color.White},
        new VertexPositionColor() {Position = new Vector3(5f, 0f, -5f), Color = Color.White},
        new VertexPositionColor() {Position = new Vector3(10f, 0f, -5f), Color = Color.White}
        };
    }
```

Vertices 0 to 2 are positioned on the positive X axis. Vertices 3 and 4 have a negative Z component, as  MonoGame considers the negative Z axis to be ‘Forward’. As your vertices are defined along the Right and the Forward directions, the resulting triangles will be lying flat on the ground.

Next, we will create a list of indices. As discussed earlier, indices reference specific vertices defined in our array of vertices. The indices build the triangles, so for two triangles, we will need to define 6 indices. Start by defining the array at the top of your class. Since indices are integer numbers, you will define an array capable of storing **int's**.

Let us start by adding a new variable to track our list of Indices in the Properties section of our class:

```csharp
    private int[] _indices;
```

And create another small method that fills the array of indices.

```csharp
    private void SetUpIndices()
    {
        _indices = new int[6];

        _indices[0] = 3;
        _indices[1] = 1;
        _indices[2] = 0;
        _indices[3] = 4;
        _indices[4] = 2;
        _indices[5] = 1;
    }
```

Or in shorthand:

```csharp
    private void SetUpIndices()
    {
        _indices = new int[6] {3,1,0,4,2,1};
    }
```

As you can see, this method defines six indices that make up two triangles. Vertex number one is referred to twice, which was our initial goal as you can see in the image above. In this case, the profit is rather small, but in bigger applications (as you will see soon ;) ) this is the way to go.

> Also, note that the triangles have been defined in a **clockwise order** again, so MonoGame will see them as facing the camera and will not cull them away.

Make sure to call this method from our **LoadContent** method after the call to SetUpVertices:

```csharp
    SetUpIndices();
```

All that's left for this chapter is to draw the triangles from our buffer! Replace the following line in your Draw method:

```csharp
    _device.DrawUserIndexedPrimitives(PrimitiveType.TriangleList, _vertices, 0, _vertices.Length, _indices, 0, _indices.Length / 3, VertexPositionColor.VertexDeclaration);
```

Instead of using the **DrawUserPrimitives** method, this time we call the **DrawUserIndexedPrimitives** method which allows us to specify both an **array of vertices** and an **array of indices**. The second-last argument specifies how many triangles are defined by the indices. Since one triangle is defined by 3 indices, we specify the number of indices divided by 3.

Before you try this code, make your triangles stop rotating by resetting their World matrix to the unity matrix. The Identity matrix is the unity matrix, so your original World space coordinates will be used.

```csharp
    Matrix worldMatrix = Matrix.Identity;
```

That's it! When you run the program, however, there will be not too much to see. This is because both your triangles and your camera are positioned on the floor! We’ll have to reposition our camera so the triangles are in sight of the camera. So go to our **SetUpCamera** method, and change the position of the camera so it is positioned above our (0,0,0) 3D origin:

```csharp
    _viewMatrix = Matrix.CreateLookAt(new Vector3(0, 50, 0), new Vector3(0, 0, 0), new Vector3(0, 0, -1));
```

We've positioned our camera 50 units above the (0,0,0) 3D origin, as the Y axis is considered as Up axis by  MonoGame. However, because the camera is looking down, you can no longer specify the (0,1,0) Up vector as Up vector for the camera! Therefore, you specify the (0,0,-1) Forward vector as Up vector for the camera.

Now when you run this code, you should see both triangles, but they’re still solid.

![Solid Triangles](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA1-06Indices1.png?raw=true)

## Seeing how the sausage is made

Another way to look at our rendered content is to tell the graphics card to NOT fill in the spaces inside the triangle, try changing this property to your **RasterizerState**:

```csharp
    rs.FillMode = FillMode.WireFrame;
```

This will only draw the edges of our triangles, instead of solid triangles.

![Wireframe](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA1-06Indices2.png?raw=true)

> By default MonoGame runs in what is referred to as "Reach" mode, aimed at targeting the lowest common denominator graphics platform, [you can read more about Graphics Profiles here](https://docs.microsoft.com/en-us/previous-versions/windows/xna/ff604995(v=xnagamestudio.42)).
>
> If you only wish to target high-end systems, you can upgrade your implementation to use the "HiDef" profile and then update the variables to use **ints** instead of **shorts** and by filling it like this:
>
> ```csharp
>     int[] indices;
> ```
>
> And in your SetUpIndices method, initialize it like this:
>
> ```csharp
>     indices = new int[6];
> ```
>
> This should work, even on pcs with lower-end graphics cards.

The benefit of using indices in this example is not very big as we are only passing 5 vertices instead of 6. In the next chapter, however, the benefit will be a lot larger.

## Exercises

You can try these exercises to practice what you have learned:

* Try to render the triangle that connects vertices 1, 3, and 4. The only changes you need to make are in the **SetUpIndices** method.
* Use another vector, such as (1,0,0) as Up vector for your camera’s view matrix.

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
        private short[] _indices;

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
            _vertices = new VertexPositionColor[5]
            {
                new VertexPositionColor() {Position = new Vector3(0f, 0f, 0f), Color = Color.White},
                new VertexPositionColor() {Position = new Vector3(5f, 0f, 0f), Color = Color.White},
                new VertexPositionColor() {Position = new Vector3(10f, 0f, 0f), Color = Color.White},
                new VertexPositionColor() {Position = new Vector3(5f, 0f, -5f), Color = Color.White},
                new VertexPositionColor() {Position = new Vector3(10f, 0f, -5f), Color = Color.White}
            };
        }

        private void SetUpIndices()
        {
            _indices = new short[6] { 3, 1, 0, 4, 2, 1 };
        }

        private void SetUpCamera()
        {
            _viewMatrix = Matrix.CreateLookAt(new Vector3(0, 50, 0), new Vector3(0, 0, 0), new Vector3(0, 0, -1));
            _projectionMatrix = Matrix.CreatePerspectiveFieldOfView(MathHelper.PiOver4, _device.Viewport.AspectRatio, 1.0f, 300.0f);
        }

        protected override void LoadContent()
        {
            // TODO: use this.Content to load your game content here
            _device = _graphics.GraphicsDevice;

            _effect = Content.Load<Effect>("effects");

            SetUpCamera();

            SetUpVertices();
            SetUpIndices();
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
            rs.FillMode = FillMode.WireFrame;
            _device.RasterizerState = rs;

            // TODO: Add your drawing code here
            _effect.CurrentTechnique = _effect.Techniques["ColoredNoShading"];
            _effect.Parameters["xView"].SetValue(_viewMatrix);
            _effect.Parameters["xProjection"].SetValue(_projectionMatrix);
            Matrix worldMatrix = Matrix.Identity;
            _effect.Parameters["xWorld"].SetValue(worldMatrix);

            foreach (EffectPass pass in _effect.CurrentTechnique.Passes)
            {
                pass.Apply();
                _device.DrawUserIndexedPrimitives(PrimitiveType.TriangleList, _vertices, 0, _vertices.Length, _indices, 0, _indices.Length / 3, VertexPositionColor.VertexDeclaration);
            }

            base.Draw(gameTime);
        }
    }
}
```

## Next Steps

[Terrain creation basics](Riemers3DXNA1Terrain07terrainbasics)
