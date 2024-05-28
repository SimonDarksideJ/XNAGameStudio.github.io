# Rendering a Texutre in XNA


XNA 1.0
For XNA 3.1 scroll down


First off you should create a new project using the Windows Game (XNA) application wizard in Visual C# 2005 express with XNA

A new application will be created.

Execute this program and see how it renders a blank window with a light blue background.


Examine the primary code in this sample that was generated.


Look at Game1.cs (Right click on the Game1.cs file in the solution explorer and select "view code")

As you can see there are several assembly imports for the XNA framework:

```csharp
using System;
using System.Collections.Generic;
using Microsoft.Xna.Framework;
using Microsoft.Xna.Framework.Content;
using Microsoft.Xna.Framework.Graphics;
using Microsoft.Xna.Framework.Input;
```


The App Wizard also creates a simple game interface:

```csharp
namespace DrawTexture
{
    public class Game1 : Microsoft.Xna.Framework.Game
    {
```


I have modified this sample to render a picture inside the window so lets add the following code right here:

```csharp
       Texture2D fredTexture;
```


MonoGame provides a LoadContent method to load your assets when the project starts, here I've added code to Load my texture and then size the camera / screen to the size of that Texture:

```csharp
        protected override void LoadContent()
        {
            // Create a new SpriteBatch, 
            //which can be used to draw textures.
            spriteBatch = new SpriteBatch(GraphicsDevice);

            //do not put the texture extension
            fredTexture = Content.Load<Texture2D>("fred");


            //resize the window to be the size of the texture
            graphics.PreferredBackBufferWidth = fredTexture.Width;
            graphics.PreferredBackBufferHeight = fredTexture.Height;
            graphics.ApplyChanges();
        }
```


The project template also adds an Update function which is called by the game for each loop of your game.

This is where you would place any logic that should be handled each frame of the games rendering. I've added some simple for to this just for reference but it's not actually needed for this sample:

```csharp
        protected override void Update()
        {
            // The time since Update was called last
            float elapsed = (float)ElapsedTime.TotalSeconds;

            // TODO: Add your game logic here
        }
```


The Draw() method is the perfect place to render to the screen :)

I have utilised the spriteBatch provided for us to render the image in side the game window:

```csharp
        protected override void Draw(GameTime gameTime)
        {
            GraphicsDevice.Clear(Color.CornflowerBlue);

            spriteBatch.Begin();

            spriteBatch.Draw(fredTexture, Vector2.Zero, Color.White);

            spriteBatch.End();

            base.Draw(gameTime);
        }
    }
}
```

Last thing we need to do is add a Texture to our Content project.  In the default solution you should find a "Content" folder and within that the default "Content.mgcb" file.

So if you open up the content project and either drag / drop a texture in to the editor and save it, it will be available to your project.
If your texture is called something other than "fred", then simply update the name of your asset in the following line in the **LoadContent** method

```csharp
            //do not put the texture extension
            fredTexture = Content.Load<Texture2D>("fred");
```

Thats it! Run the project and you should see your image full screen.

MonoGame makes coding games very easy :)




We will look into some of the other features in the next tutorial set
