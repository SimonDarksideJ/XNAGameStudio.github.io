# Creating a floorplan

Now that we have seen how we can import simple images into our MonoGame project and have displayed them on triangles, it is not that difficult to create a large number of images. It is, however, more important to find a way to get the computer to define all of the vertices for us.

## The city floor plan

As a small example, let us simply create a [raster](https://en.wikipedia.org/wiki/Raster_graphics) of 3x3 images with the center image missing. This means eight images which will need 16 triangles and 48 vertices. Instead of defining all these vertices manually, let us create a "floorPlan" array in the Properties section of your code. This array will later contain where we want to have buildings in our 3D city:

```csharp
    private int[,] _floorPlan;

    private VertexBuffer _cityVertexBuffer;
```

Since we will need to define the vertices only once, we will store them on the RAM of our graphics card by using them in a VertexBuffer ([as described in Series 1](Riemers3DXNA1Terrain13buffers)) rather than in a simple array as we did in the previous chapter.

We will first create a small **LoadFloorPlan** method that fills the floorPlan array with data:

```csharp
    private void LoadFloorPlan()
    {
        _floorPlan = new int[,]
        {
            {0,0,0},
            {0,1,0},
            {0,0,0},
        };
    }
```

In this data, a 0 means *'draw a floor texture'* and a 1 means to *'leave that tile open'* (Later in this series, a 1 will indicate a building).  This method contains all the flexibility our program needs, simply changing a 0 to a 1, will result in an extra building drawn in our 3D city!

Load this method from within the **Initialize** method:

```csharp
    LoadFloorPlan();
```

## Turning our floorplan into a mesh

Now we will update our **SetUpVertices** method, so it reads the data inside the array and automatically creates the corresponding vertices. In the last chapter you learned how to cover triangles with images, this time, we are going to load one texture image file which is composed of several images next to each other. The leftmost part of the texture will be the floor tile, followed by a wall, and a roofing image for each different type of building.

![Texture Map](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA2-03Floorplan1.bmp?raw=true)

You can remove the ‘riemerstexture’ asset from your Content Project and replace it with add the image you just downloaded to your MonoGame project, as you have done before. In the end, we are going to rename our *'texture'* variable to *'sceneryTexture'*, because later in the game we will be using more than one texture. 

Change the name of the **'_texture'** variable at the top of the code to:

```csharp
    private Texture2D _sceneryTexture;
```

And make sure you update the name of the variable and asset in your **LoadContent** method:

```csharp
    _sceneryTexture = Content.Load<Texture2D>("texturemap");
```

Next, we can delete the contents of the **SetUpVertices** method and replace it with the following code, which is based on the last chapter of Series 1:

```csharp
 private void SetUpVertices()
 {
     int cityWidth = _floorPlan.GetLength(0);
     int cityLength = _floorPlan.GetLength(1);

    List<VertexPositionNormalTexture> verticesList = new List<VertexPositionNormalTexture> ();
    for (int x = 0; x < cityWidth; x++)
    {
        for (int z = 0; z < cityLength; z++)
        {
            //if floorPlan contains a 0 for this tile, add 2 triangles
        }
    }

    _cityVertexBuffer = new VertexBuffer(_device, VertexPositionNormalTexture.VertexDeclaration, verticesList.Count, BufferUsage.WriteOnly);

    _cityVertexBuffer.SetData<VertexPositionNormalTexture> (verticesList.ToArray());
    }
```

> Make sure to also remove the old **"_vertices"** property as we will no longer be using it, else you may see warnings when building your project.

As we are now using a **List** for looping through our collection of vertices, we also need to add a **using** statement so the code knows where to find it, so add the following to the very top of the class with the other **using** statements:

```csharp
    using System.Collections.Generic;
```

This code first retrieves the width and length of our future city, which will be 3x3 in the case of our current floorMap. Next, we create a List capable of storing **VertexPositionNormalTextures**. The main advantage of a List over an array is that you do not have to specify the number of elements you are going to add. You can just add elements, and the List will grow larger automatically.

> Although "List's" are very convenient and useful for setting up data as we have done with our Vertices list, they should not be used in situations that require high-frequency lookups, such as within an **Update** loop. For these cases, it is far better (and performant) to use a **for** loop.

Next, we scroll through the contents of the **floorMap** array, whenever a 0 is found in the floorMap, we want this loop to add 6 vertices to the List (2 triangles). We will finish this off properly later, for now, imagine that when the for loop finishes, the List contains 6 vertices for each 0 tile found in the floorMap.

To store the vertices in the RAM of the graphics card, we create a VertexBuffer that is exactly large enough (see Series 1) to hold the data we intend to fill it with. When setting the VertexBuffer, we transform the List into an array so that we can use the SetData method (which requires an array as an input).

To finish this method, we need to add this code inside the for-loop of the **SetUpVertices** method:

```csharp
    int imagesInTexture = 11;
    if (_floorPlan[x, z] == 0)
    {
        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x, 0, -z), new Vector3(0, 1, 0), new Vector2(0, 1)));
        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x, 0, -z - 1), new Vector3(0, 1, 0), new Vector2(0, 0)));
        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x + 1, 0, -z), new Vector3(0, 1, 0), new Vector2(1.0f / imagesInTexture, 1)));

        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x, 0, -z - 1), new Vector3(0, 1, 0), new Vector2(0, 0)));
        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x + 1, 0, -z-1), new Vector3(0, 1, 0), new Vector2(1.0f / imagesInTexture, 0)));
        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x + 1, 0, -z), new Vector3(0, 1, 0), new Vector2(1.0f / imagesInTexture, 1)));
    }
```

Every time a 0 tile is encountered, two triangles are defined, the normal vectors are pointing upwards (0,1,0) towards the sky, and the correct portion of the texture image is pasted over the image (the rectangle between [0,0] and [1/imagesintexture,1]). In my sample texture map, I have stored 11 images, so that the X coordinates for the first image stretches from 0 to 1/11. Have another look at the texture above to fully understand this.

Right now, we have defined a lot of vertices corresponding to two triangles for each 0 in our floorMap. What is more, is that these vertices have been saved in the RAM on the graphics card.

## Drawing our city mesh

With the "SetUpVertices" method finished, we can move on to the code that renders the triangles. To keep our **Draw** method clean, we will define a new method as follows:

```csharp
    private void DrawCity()
    {
        _effect.CurrentTechnique = _effect.Techniques["Textured"];
        _effect.Parameters["xWorld"].SetValue(Matrix.Identity);
        _effect.Parameters["xView"].SetValue(_viewMatrix);
        _effect.Parameters["xProjection"].SetValue(_projectionMatrix);
        _effect.Parameters["xTexture"].SetValue(_sceneryTexture);

        foreach (EffectPass pass in _effect.CurrentTechnique.Passes)
        {
            pass.Apply();
            _device.SetVertexBuffer(_cityVertexBuffer);
            _device.DrawPrimitives(PrimitiveType.TriangleList, 0, _cityVertexBuffer.VertexCount/3);
        }
    }
```

> This method of separating the actual rendering of 3D elements is very common, ensuring that the correct shader parameters and draw method are used for that content.  Rendering can become quite complex and a fair amount of through has to go into how specific elements of your game are processed and rendered as efficiently as possible, grouping together elements that are rendered the same way. (similar to how 2D content uses [SpriteBatches](Riemers2DXNA04spritebatch))

We will still be using the **Textured** technique in our effect (shader) to render our triangles from our vertices. As always, we need to apply our World, View, and Projection matrices to the effect. As we want the graphics card to also sample the colors from our texture, we need to pass this texture to the graphics card.

The triangles are rendered from the **VertexBuffer**, which are set using our prebuilt buffer in the **SetUpVertices** method, the last argument of the **DrawPrimitives** method automatically determines how many triangles can be rendered from the VertexBuffer, since three vertices define one triangle, we know how many triangles to render!

Do not forget to also call this method from within your **Draw** method by updating it to look as follows:

```csharp
    protected override void Draw(GameTime gameTime)
    {
        _device.Clear(ClearOptions.Target | ClearOptions.DepthBuffer, Color.DarkSlateBlue, 1.0f, 0);

        DrawCity();

        base.Draw(gameTime);
    }
```

This code should be runnable! Although it might be a good idea to reposition our camera a bit in the **SetUpCamera** method:

```csharp
    _viewMatrix = Matrix.CreateLookAt(new Vector3(3, 5, 2), new Vector3(2, 0, -1), new Vector3(0, 1, 0));
```

When running this code, you should see a small square with a hole in the middle, just as you defined in the LoadFloorPlan method.

This should give you the following image:

![Floorplan](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA2-03Floorplan2.jpg?raw=true)

## Exercises

You can try these exercises to practice what you have learned:

* Change the size of the floorPlan array.
* Play around with the texture coordinates in the SetUpVertices method, it’s worth it!! You can choose any value between 0 and 1.
* Change the texture coordinates of our vertices, so your graphics card uses another image from our texture map to cover the floor.

## The code so far

```csharp
using System.Collections.Generic;
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
        private Texture2D _sceneryTexture;
        private int[,] _floorPlan;
        private VertexBuffer _cityVertexBuffer;

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

            LoadFloorPlan();

            base.Initialize();
        }

        private void SetUpCamera()
        {
            _viewMatrix = Matrix.CreateLookAt(new Vector3(3, 5, 2), new Vector3(2, 0, -1), new Vector3(0, 1, 0));
            _projectionMatrix = Matrix.CreatePerspectiveFieldOfView(MathHelper.PiOver4, _device.Viewport.AspectRatio, 0.2f, 500.0f);
        }

        private void SetUpVertices()
        {
            int cityWidth = _floorPlan.GetLength(0);
            int cityLength = _floorPlan.GetLength(1);

            List<VertexPositionNormalTexture> verticesList = new List<VertexPositionNormalTexture>();
            for (int x = 0; x < cityWidth; x++)
            {
                for (int z = 0; z < cityLength; z++)
                {
                    //if floorPlan contains a 0 for this tile, add 2 triangles
                    int imagesInTexture = 11;
                    if (_floorPlan[x, z] == 0)
                    {
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x, 0, -z), new Vector3(0, 1, 0), new Vector2(0, 1)));
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x, 0, -z - 1), new Vector3(0, 1, 0), new Vector2(0, 0)));
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x + 1, 0, -z), new Vector3(0, 1, 0), new Vector2(1.0f / imagesInTexture, 1)));

                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x, 0, -z - 1), new Vector3(0, 1, 0), new Vector2(0, 0)));
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x + 1, 0, -z - 1), new Vector3(0, 1, 0), new Vector2(1.0f / imagesInTexture, 0)));
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x + 1, 0, -z), new Vector3(0, 1, 0), new Vector2(1.0f / imagesInTexture, 1)));
                    }
                }
            }

            _cityVertexBuffer = new VertexBuffer(_device, VertexPositionNormalTexture.VertexDeclaration, verticesList.Count, BufferUsage.WriteOnly);

            _cityVertexBuffer.SetData<VertexPositionNormalTexture>(verticesList.ToArray());
        }

        private void LoadFloorPlan()
        {
            _floorPlan = new int[,]
            {
            {0,0,0},
            {0,1,0},
            {0,0,0},
            };
        }

        protected override void LoadContent()
        {
            _spriteBatch = new SpriteBatch(GraphicsDevice);

            // TODO: use this.Content to load your game content here
            _device = _graphics.GraphicsDevice;
            _effect = Content.Load<Effect>("effects");
            _sceneryTexture = Content.Load<Texture2D>("texturemap");

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

        private void DrawCity()
        {
            _effect.CurrentTechnique = _effect.Techniques["Textured"];
            _effect.Parameters["xWorld"].SetValue(Matrix.Identity);
            _effect.Parameters["xView"].SetValue(_viewMatrix);
            _effect.Parameters["xProjection"].SetValue(_projectionMatrix);
            _effect.Parameters["xTexture"].SetValue(_sceneryTexture);

            foreach (EffectPass pass in _effect.CurrentTechnique.Passes)
            {
                pass.Apply();
                _device.SetVertexBuffer(_cityVertexBuffer);
                _device.DrawPrimitives(PrimitiveType.TriangleList, 0, _cityVertexBuffer.VertexCount / 3);
            }
        }

        protected override void Draw(GameTime gameTime)
        {
            _device.Clear(ClearOptions.Target | ClearOptions.DepthBuffer, Color.DarkSlateBlue, 1.0f, 0);

            DrawCity();

            base.Draw(gameTime);
        }
    }
}
```

## Next Steps

[Drawing the buildings](Riemers3DXNA2flightsim04creatingcity)
