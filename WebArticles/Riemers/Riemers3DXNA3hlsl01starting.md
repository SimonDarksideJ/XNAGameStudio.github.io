# Starting point for the 3rd series of XNA Tutorials

Welcome to this 3rd installment of my Tutorials on XNA. Because this Series will cover a lot of ground, I would like to take a jumpstart by starting from the code presented below. As you can see on the screenshot below, it will only draw a simple triangle. There is nothing in this code that hasn’t been covered yet in the previous 2 series.

At this moment, my standard effects.fx file is loaded so we are able to render the triangle, but soon my effect file will be replaced by one of your own.

> Use the Effects file provided and add it to your Content Project as normal.

That’s about all there is to say about our starting code. If you have been following my tutorials up to this point, simply copy-paste this code into a new XNA Game Studio 4.0 project. Remember, you might have to change my namespace to yours (or vice versa). The only requirement is that both namespaces in the Game1.cs and Program.cs files are the same.

![Starting](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA3-01Starting1.jpg?raw=true)

## Starting Code

```csharp
 using System;
 using System.Collections.Generic;
 using Microsoft.Xna.Framework;
 using Microsoft.Xna.Framework.Audio;
 using Microsoft.Xna.Framework.Content;
 using Microsoft.Xna.Framework.GamerServices;
 using Microsoft.Xna.Framework.Graphics;
 using Microsoft.Xna.Framework.Input;
 using Microsoft.Xna.Framework.Net;
 using Microsoft.Xna.Framework.Storage;
 
 namespace XNAseries3
 {
     public class Game1 : Microsoft.Xna.Framework.Game
     {
         GraphicsDeviceManager graphics;
         GraphicsDevice device;
 
         Effect effect;
         Matrix viewMatrix;
         Matrix projectionMatrix;
         VertexBuffer vertexBuffer;
         Vector3 cameraPos;
 
         public Game1()
         {
             graphics = new GraphicsDeviceManager(this);
             Content.RootDirectory = "Content";
         }
 
         protected override void Initialize()
         {
             graphics.PreferredBackBufferWidth = 500;
             graphics.PreferredBackBufferHeight = 500;
             graphics.IsFullScreen = false;
             graphics.ApplyChanges();
             Window.Title = "Riemer's XNA Tutorials -- Series 3";
 
             base.Initialize();
         }
 
         protected override void LoadContent()
         {
             device = GraphicsDevice;
 

            effect = Content.Load<Effect> ("effects"); SetUpVertices();            SetUpCamera();
        }

        private void SetUpVertices()
        {
            VertexPositionColor[] vertices = new VertexPositionColor[3];

            vertices[0] = new VertexPositionColor(new Vector3(-2, 2, 0), Color.Red);
            vertices[1] = new VertexPositionColor(new Vector3(2, -2, -2), Color.Green);
            vertices[2] = new VertexPositionColor(new Vector3(0, 0, 2), Color.Yellow);

            vertexBuffer = new VertexBuffer(device, VertexPositionColor.VertexDeclaration, vertices.Length, BufferUsage.WriteOnly);
            vertexBuffer.SetData(vertices);
        }

        private void SetUpCamera()
        {
            cameraPos = new Vector3(0, 5, 6);
            viewMatrix = Matrix.CreateLookAt(cameraPos, new Vector3(0, 0, 1), new Vector3(0, 1, 0));
            projectionMatrix = Matrix.CreatePerspectiveFieldOfView(MathHelper.PiOver4, device.Viewport.AspectRatio, 1.0f, 200.0f);
        }

        protected override void UnloadContent()
        {
        }

        protected override void Update(GameTime gameTime)
        {
            if (GamePad.GetState(PlayerIndex.One).Buttons.Back == ButtonState.Pressed)
                this.Exit();

            base.Update(gameTime);
        }

        protected override void Draw(GameTime gameTime)
        {
            device.Clear(ClearOptions.Target | ClearOptions.DepthBuffer, Color.DarkSlateBlue, 1.0f, 0);

            effect.CurrentTechnique = effect.Techniques["ColoredNoShading"];
            effect.Parameters["xView"].SetValue(viewMatrix);
            effect.Parameters["xProjection"].SetValue(projectionMatrix);
            effect.Parameters["xWorld"].SetValue(Matrix.Identity);

            foreach (EffectPass pass in effect.CurrentTechnique.Passes)
            {
                pass.Apply();

                device.SetVertexBuffer(vertexBuffer);
                device.DrawPrimitives(PrimitiveType.TriangleList, 0, 1);
            }

            base.Draw(gameTime);
        }
    }
}
```

## Next Steps

[HLSL Introduction](Riemers3DXNA3hlsl02hlslintroduction)
