# Using the Mouse


Here is a nice little class that encapsulates the mouse. There isn't much to say about it since you should be getting used to creating, rendering and disposing of objects after you've read the previous tutorials :)

Feel free to check out the tutorial on creating a texture to familiarize yourself with initializing and disposing of objects in XNA here

```csharp

using Microsoft.Xna.Framework;
using Microsoft.Xna.Framework.Graphics;
using Microsoft.Xna.Framework.Input;

    public class XNAMouse
    {
        public MouseState mouseState;
        SpriteBatch mouseSprite;
        Texture2D mouseTexture;

        public XNAMouse(GraphicsDevice device, Texture2D texture)
        {
            mouseSprite = new SpriteBatch(device);

            mouseTexture = texture;
        }

        public void Render()
        {
```


You should set the BlendState to AlphaBlend and set the alpha channel for your cursor texture appropriately.

> More on BlendStates and other spritebatch options in a later tutorial

```csharp
            mouseSprite.Begin(blendState:BlendState.AlphaBlend);
            mouseSprite.Draw(mouseTexture, new Rectangle(mouseState.X, mouseState.Y,
                mouseTexture.Width, mouseTexture.Height), Color.White);
            mouseSprite.End();
        }

        public void Update()
        {
            mouseState = Mouse.GetState();
        }

        public void Dispose()
        {
            mouseTexture.Dispose();
            mouseSprite.Dispose();
        }
    }
```

With this in place, consuming it in your game is very easy.  Simply create a reference to the XNAMouse in your game as follows:

```csharp
    XNAMouse mouse;
```
Ensure you load a cursor texture and then initialise the mouse in LoadContent as shown here:

```csharp
    var mouseTexture = Content.Load<Texture2D>("mouse-pointer-texture-name");
    mouse = new XNAMouse(GraphicsDevice, mouseTexture);
```

Ensure the mouse is updated, so that the current cursor position is obtained in the Update method:

```csharp
    mouse.Update();
```

Then finally, add a call to draw the mouse on the screen:

> *Note, this should really be the last draw call to ensure your mouse is drawn on top of everything else (unless you have screen effects that will effect the mouse cursor, more on that later)

```csharp
    mouse.Render();
```

All done, simples.