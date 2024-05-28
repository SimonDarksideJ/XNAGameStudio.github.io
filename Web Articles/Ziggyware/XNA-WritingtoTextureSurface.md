# Writing to a Texture in MonoGame

In this tutorial you will be writing to a texture's memory to create a texture on the fly :)

Declare the standard assemblies for creating an XNA game:

```csharp
using System;
using System.Collections.Generic;
using Microsoft.Xna.Framework;
using Microsoft.Xna.Framework.Content;
using Microsoft.Xna.Framework.Graphics;
using Microsoft.Xna.Framework.Input;
```



The game template defines the default namespace and Game1 class, as follows:

```csharp
namespace WriteToTexture
{
    public class Game1 : Microsoft.Xna.Framework.Game
    {
```


The GraphicsDeviceManager object lets you change video modes and change the window size. Use the PreferredBackBufferWidth and PreferredBackBufferHeight values of the GraphicsDeviceManager in the Game1 constructor if you would like to change the window size.

```csharp
        GraphicsDeviceManager graphics;
```


In order to render the texture once you have written to it you will need a SpriteBatch object

```csharp
        SpriteBatch spriteBatch;
        Texture2D texture;
```


MonoGame creates our default Game constructor and Initialize methods for us as shown here:

```csharp
        public Game1()
        {
            graphics = new GraphicsDeviceManager(this);
            Content.RootDirectory = "Content";
        }

  
        protected override void Initialize()
        {
            base.Initialize();
        }
```


The template also provides a sprite batch in the LoadContent method
```csharp
        protected override void LoadContent()
        {
            // Create a new SpriteBatch, 
            //which can be used to draw textures.
            spriteBatch = new SpriteBatch(GraphicsDevice);
```


Now you need to create a texture that you can modfiy.

Set the width and height to the back buffer width and height

The fourth parameter is the number of mip levels. 0 means as many as the device can handle. For this example you are not using mip maps so set it to 1. This creates only one surface in memory.

The fifth parameter sets the texture usage. You do not need to use this right now so set it to TextureUsage.None.

The last parameter is the format of the texture surface. XNA Uses SurfaceFormat.Color as the most generic format.

```csharp
            //lets create an empty texture
            texture = new Texture2D(GraphicsDevice,
                graphics.PreferredBackBufferWidth,
                graphics.PreferredBackBufferHeight,
                1, TextureUsage.None, SurfaceFormat.Color);
```


Now define an array to hold the pixels of the texture and populate it from the surface

```csharp
            Color[] rgba = new Color[texture.Width * texture.Height];
            texture.GetData<Color>(rgba);
```

Now you need to put some color into this texture :)

Create a point at the middle of the texture and calculate the distance to each pixel

This produces a light map of sorts

```csharp
            //set a point that is above and in the middle of the texture
            Vector3 vLightPos = new Vector3(
                graphics.PreferredBackBufferWidth / 2,
                graphics.PreferredBackBufferHeight / 2,
                0);
            
            for (int x = 0; x < texture.Width; x++)
            {
                for (int y = 0; y < texture.Height; y++)
                {
                    Vector3 v = new Vector3(x, y, 0);
                    float f = Vector3.Distance(v, vLightPos);
```


You need to clamp the distance value between 0 and 255

```csharp
                    f = f > 255 ? 255 : f < 0 ? 0 : f;
```


Here i reverse the color to make it fade from the nearest point outward and divide by 255 to make the number between 0 and 1

```csharp
                    f = (255 - f) / 255.0f;
```


And finally set the color at the correct position in the texture:

```csharp
                    Color c = new Color(f, f, f);
                    rgba[x + y * texture.Width] = c;
                }
            }
```


Now that you're done modifying the pixels, put them back into the surface

```csharp
            texture.SetData<Color>(rgba);
        }
```


Make sure you dispose of the texture during UnloadContent. Since this texture was created without using the Content pipeline you are responsible for disposing the resources.

```csharp
        protected override void UnloadContent()
        {
            texture.Dispose();
        }
```


Now you just need to draw the texture on the screen and see what results you got! :)

```csharp
        protected override void Draw(GameTime gameTime)
        {
            GraphicsDevice.Clear(Color.CornflowerBlue);

            spriteBatch.Begin();

            spriteBatch.Draw(texture, Vector2.Zero, Color.White);

            spriteBatch.End();

            base.Draw(gameTime);
        }
    }
}
```
