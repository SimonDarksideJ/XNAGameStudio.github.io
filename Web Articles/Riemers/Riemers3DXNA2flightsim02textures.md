# Textures

Up till now, the only way we have seen to add some color to our scene is to declare separate vertices for every different color. Of course, this is not the way today’s great games are being made. MonoGame supports a very efficient way of adding color and images to the scene, you can simply apply an image to a triangle. Such images are called textures.

## Painting with textures

As a first example, we are going to draw 1 simple triangle, and cover it with a texture. Import the "riemerstexture.bmp" image from the [Asset pack](https://github.com/SimonDarksideJ/XNAGameStudio/raw/archive/Samples/Riemers/3D%20Series2%20-%20FlightSim%20-%20Assets.zip?raw=true) and add it to the Content project of your MonoGame solution, just as you did earlier for the effect file.

In our code, we are going to add a new variable to hold this texture image. Add this line to the **Properties** your code:

```csharp
    private Texture2D _texture;
```

Now find the **LoadContent** method in your code and add this line after the line where you load your effect file:

```csharp
    _texture = Content.Load<Texture2D>("riemerstexture");
```

This line binds the asset we just loaded in our project to the texture variable!

With our texture loaded into our MonoGame project, it is time to define the 3 vertices which we will be stored in an array. As our vertices will need to be able to store both a 3D position and a texture coordinate (explained below), the vertex format will be **VertexPositionTexture**, so declare this variable in the Properties section of your code:

```csharp
    private VertexPositionTexture[] _vertices;
```

Next, we will be defining the 3 vertices of our triangle, add a new **SetUpVertices** method with the following code:

```csharp
    private void SetUpVertices()
    {
        _vertices = new VertexPositionTexture[3];

        _vertices[0].Position = new Vector3(-10f, 10f, 0f);
        _vertices[0].TextureCoordinate.X = 0;
        _vertices[0].TextureCoordinate.Y = 0;

        _vertices[1].Position = new Vector3(10f, -10f, 0f);
        _vertices[1].TextureCoordinate.X = 1;
        _vertices[1].TextureCoordinate.Y = 1;

        _vertices[2].Position = new Vector3(-10f, -10f, 0f);
        _vertices[2].TextureCoordinate.X = 0;
        _vertices[2].TextureCoordinate.Y = 1;
    }
```

As you see, for every vertex we first define its position in 3D space. Notice again that we have defined our vertices in a clockwise way, so MonoGame will not cull them away (see the ["World Space"](Riemers3DXNA1Terrain04worldspace) chapter in series 1).

The next 2 settings are very important, as they define which point in our texture image we want to correspond with the vertex. These coordinates are simply the X and Y coordinates of the texture, as follows:

* The (0,0) texture coordinate being the top-left point of the texture image
* The (1,0) texture coordinate the top-right point
* And the (1,1) texture coordinate the bottom-right point of the texture.

Don’t forget to call the **SetUpVertices** method from your LoadContent method after the call to **SetUpCamera**:

```csharp
    SetUpVertices();
```

## Time to draw our texture triangle

OK, we have our vertices set up and our texture image loaded into a variable. Let us now draw the triangle!

Go to our **Draw** method and add this code after our call to the Clear method:

```csharp
    Matrix worldMatrix = Matrix.Identity;
    _effect.CurrentTechnique = _effect.Techniques["TexturedNoShading"];
    _effect.Parameters["xWorld"].SetValue(worldMatrix);
    _effect.Parameters["xView"].SetValue(_viewMatrix);
    _effect.Parameters["xProjection"].SetValue(_projectionMatrix);
    _effect.Parameters["xTexture"].SetValue(_texture);

    foreach (EffectPass pass in _effect.CurrentTechnique.Passes)
    {
        pass.Apply();
        _device.DrawUserPrimitives(PrimitiveType.TriangleList, _vertices, 0, 1, VertexPositionTexture.VertexDeclaration);
    }
```

As usual, we indicate which technique we want the graphics card to use to render the triangle to the screen. We need to instruct our graphics card to sample the color of every pixel from the texture image. This is exactly what the **TexturedNoShading** technique of my effect file does, so we set it as an active technique. As we did not specify any normals for our vectors so we cannot expect the effect to do any meaningful shading calculations.

* As explained in Series 1, we need to set the World matrix to identity so the triangles will be rendered where we defined them, and View and Projection matrices so the graphics card can map the 3D positions to 2D screen coordinates.
* Finally, we pass our texture to the technique. Then we draw our triangle from our vertices array, as done before in the first series.

Running this should already give you a textured triangle, displaying half of the texture image!

## The missing half

To display the whole image, we simply have to expand our **SetUpVertices** method by adding the second triangle:

```csharp
    private void SetUpVertices()
    {
        _vertices = new VertexPositionTexture[3];

        _vertices[0].Position = new Vector3(-10f, 10f, 0f);
        _vertices[0].TextureCoordinate.X = 0;
        _vertices[0].TextureCoordinate.Y = 0;

        _vertices[1].Position = new Vector3(10f, -10f, 0f);
        _vertices[1].TextureCoordinate.X = 1;
        _vertices[1].TextureCoordinate.Y = 1;

        _vertices[2].Position = new Vector3(-10f, -10f, 0f);
        _vertices[2].TextureCoordinate.X = 0;
        _vertices[2].TextureCoordinate.Y = 1;

        _vertices[3].Position = new Vector3(10.1f, -9.9f, 0f);
        _vertices[3].TextureCoordinate.X = 1;
        _vertices[3].TextureCoordinate.Y = 1;

        _vertices[4].Position = new Vector3(-9.9f, 10.1f, 0f);
        _vertices[4].TextureCoordinate.X = 0;
        _vertices[4].TextureCoordinate.Y = 0;

        _vertices[5].Position = new Vector3(10.1f, 10.1f, 0f);
        _vertices[5].TextureCoordinate.X = 1;
        _vertices[5].TextureCoordinate.Y = 0;
    }
```

We have simply added another set of 3 vertices for a second triangle to complete the square texture image. We also need to adjust the Draw method so that you now render 2 triangles instead of just 1:

```csharp
    _device.DrawUserPrimitives(PrimitiveType.TriangleList, _vertices, 0, 2, VertexPositionTexture.VertexDeclaration);
```

Now run this code, and you should see the whole texture image, displayed by 2 triangles!

![Status](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA2-02Textures1.jpg?raw=true)

You will notice the small gap between both triangles... This is of course because we defined the positions of the vertices that way, so you can see the image is made out of two separate triangles.

## Exercises

You can try these exercises to practice what you have learned:

* Try to remove the gap between the triangles.
* Play around with the texture coordinates in the SetUpVertices method, it’s worth it!! You can choose any value between 0 and 1.

## The code so far

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
        private Texture2D _texture;
        private VertexPositionTexture[] _vertices;

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

        private void SetUpVertices()
        {
            _vertices = new VertexPositionTexture[6];

            _vertices[0].Position = new Vector3(-10f, 10f, 0f);
            _vertices[0].TextureCoordinate.X = 0;
            _vertices[0].TextureCoordinate.Y = 0;

            _vertices[1].Position = new Vector3(10f, -10f, 0f);
            _vertices[1].TextureCoordinate.X = 1;
            _vertices[1].TextureCoordinate.Y = 1;

            _vertices[2].Position = new Vector3(-10f, -10f, 0f);
            _vertices[2].TextureCoordinate.X = 0;
            _vertices[2].TextureCoordinate.Y = 1;

            _vertices[3].Position = new Vector3(10.1f, -9.9f, 0f);
            _vertices[3].TextureCoordinate.X = 1;
            _vertices[3].TextureCoordinate.Y = 1;

            _vertices[4].Position = new Vector3(-9.9f, 10.1f, 0f);
            _vertices[4].TextureCoordinate.X = 0;
            _vertices[4].TextureCoordinate.Y = 0;

            _vertices[5].Position = new Vector3(10.1f, 10.1f, 0f);
            _vertices[5].TextureCoordinate.X = 1;
            _vertices[5].TextureCoordinate.Y = 0;
        }

        protected override void LoadContent()
        {
            _spriteBatch = new SpriteBatch(GraphicsDevice);

            // TODO: use this.Content to load your game content here
            _device = _graphics.GraphicsDevice;
            _effect = Content.Load<Effect>("effects");
            _texture = Content.Load<Texture2D>("riemerstexture");

            SetUpCamera();
            SetUpVertices();
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
            Matrix worldMatrix = Matrix.Identity;
            _effect.CurrentTechnique = _effect.Techniques["TexturedNoShading"];
            _effect.Parameters["xWorld"].SetValue(worldMatrix);
            _effect.Parameters["xView"].SetValue(_viewMatrix);
            _effect.Parameters["xProjection"].SetValue(_projectionMatrix);
            _effect.Parameters["xTexture"].SetValue(_texture);

            foreach (EffectPass pass in _effect.CurrentTechnique.Passes)
            {
                pass.Apply();
                _device.DrawUserPrimitives(PrimitiveType.TriangleList, _vertices, 0, 2, VertexPositionTexture.VertexDeclaration);
            }

            base.Draw(gameTime);
        }
    }
}
```

## Next Steps

[Creating a floorplan](Riemers3DXNA2flightsim03floorplan)
