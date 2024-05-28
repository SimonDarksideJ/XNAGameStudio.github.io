# Rendering our scene into a texture – Displaying 2D images

> This chapter again has nothing to do with HLSL, so anyone only interested in the Render-To-Texture technique will be able to follow.

Except for this paragraph :) As I told you last chapter, the second step of the shadow mapping algorithm involves comparing values in our depth map. To do this, we will save the screen of last chapter (the depth map) into a texture, so we can use this the data in this texture during the third and final step.

So we’ll be drawing our scene not to the screen, but into a texture instead. This can be useful for example to create a mirror in your scene: first you render the scene as seen by the mirror into the texture, afterwards you draw your scene while texturing two triangles with the rendered texture. See Recipe 3-13 on how to make a mirror.

Thanks to XNA and some of its helper classes, this rendering of our scene into a texture is made pretty easy. We’ll need to add these variables to our code:

```csharp
 RenderTarget2D renderTarget;
 Texture2D shadowMap;
```

The first line is the rendertarget where we’ll be drawing to. The default rendertarget is your screen, this is a piece of memory we can render to. The second variable is the texture where we will store the result of the rendertarget to.

To learn all details about custom render targets, see Recipe 3-8.

We only need to initialize the rendertarget, so put this code in the LoadContent method:

```csharp
 PresentationParameters pp = device.PresentationParameters;
 renderTarget = new RenderTarget2D(device, pp.BackBufferWidth, pp.BackBufferHeight, true, device.DisplayMode.Format, DepthFormat.Depth24);
```

This creates a rendertarget of exactly the same size and data format as our original target, the backbuffer to your screen. The fourth argument specifies the number of mipmap levels in the resulting texture, see the note in Recipe 3-7 about mipmaps.

Now we’ve got our variables ready, it’s time to modify the Draw method a bit. It’s very easy: we simply need to specify the active rendertarget before drawing. So put this line as the very first line in your Draw method:

```csharp
 device.SetRenderTarget(renderTarget);
```

From that line on, the rendertarget will get cleared, and our shadow map will be drawn to it. When you run your code now, there shouldn’t be drawn anything to the screen. If there is something drawn, it’s because this was still present in the backuffer. When the last thing you saw in your form was the result of last chapter, chances are you’ll still see it. But when you run another 3D program, or you reboot your computer, you’ll generally see a blank screen. Simply because you’re not longer rendering to your backbuffer!

At the end of our Draw method, after everything has been drawn, we need to store the contents of our custom render target into the texture. Before we can access its contents, though, we need to de-activate the custom render target. This can be done by activating another one, for example the default backbuffer as done here:

```csharp
 device.SetRenderTarget(null);
 shadowMap = (Texture2D)renderTarget;
```

The last line retrieves the contents of the rendertarget and puts it in our texture! It’s really that easy.

OK, that’s all very nice, but as a difficult audience you of course want proof of all this. One way would be to simply display the texture on the screen, as a 2D image. For this, we first need to set our form as active rendertarget, and use a SpriteBatch to render the 2D image. Warning: the following code is VERY dirty, as it creates a new SpriteBatch every frame! You should never create such an object each frame, rather you want to keep one as variable in your class and reuse it every frame. Yet I’m doing this anyway, as we’ll remove this code in 2 minutes:

```csharp
 device.Clear(ClearOptions.Target | ClearOptions.DepthBuffer, Color.DarkSlateBlue, 1.0f, 0);
 using (SpriteBatch sprite = new SpriteBatch(device))
 {
     sprite.Begin();
     sprite.Draw(shadowMap, new Vector2(0, 0), null, Color.White, 0, new Vector2(0, 0), 0.4f, SpriteEffects.None, 1);
     sprite.End();
 }
```

The first line cleans our backbuffer. Next, we create a SpriteBatch object, which we use to draw our texture (scaled down) to the screen!

To render the texture, we need to specify these arguments: the texture itself, the screen position where we want the texture to be drawn (we specify the top-left corner), which part of the texture to draw (null means the whole image), with which color of light to shine on the texture (white means normal colors), the rotation, where to start drawing from the texture, and the scaling. I put it in bold, because it is the only important argument for our case.

To learn more about the (advanced) functionality and performance optimizations when using the SpriteBatch, read Recipes 3-1 to 3-4.

That’s it! When you run this code, you should see the texture, scaled by a factor 0.4f, as in the image below.

![Summary](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA3-13RenderTexture1.jpg?raw=true)

Not a lot nicer than the result of last chapter, but you’ve learned a lot: you have drawn the scene into a texture, and drawn this 2D image to the screen afterwards! Now remove the code that displays the texture before moving on to the next chapter.

Our HLSL code hasn’t been changed, so I’ll only list the XNA code:

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
     public struct MyOwnVertexFormat
     {
         public Vector3 position;
         private Vector2 texCoord;
         private Vector3 normal;
 
         public MyOwnVertexFormat(Vector3 position, Vector2 texCoord, Vector3 normal)
         {
             this.position = position;
             this.texCoord = texCoord;
             this.normal = normal;
         }
 
         public readonly static VertexDeclaration VertexDeclaration = new VertexDeclaration
              (
                  new VertexElement(0, VertexElementFormat.Vector3, VertexElementUsage.Position, 0),
                  new VertexElement(sizeof(float) * 3, VertexElementFormat.Vector2, VertexElementUsage.TextureCoordinate, 0),
                  new VertexElement(sizeof(float) * (3+2), VertexElementFormat.Vector3, VertexElementUsage.Normal, 0)
              );
     }
 
     public class Game1 : Microsoft.Xna.Framework.Game
     {
         GraphicsDeviceManager graphics;
         GraphicsDevice device;
 
         Effect effect;
         Matrix viewMatrix;
         Matrix projectionMatrix;
         VertexBuffer vertexBuffer;
         Vector3 cameraPos;
         Texture2D streetTexture;
         Model lamppostModel;
         Texture2D[] lamppostTextures;
         Model carModel;
         Texture2D[] carTextures;
         Vector3 lightPos;
         float lightPower;
         float ambientPower;
         Matrix lightsViewProjectionMatrix;
         RenderTarget2D renderTarget;
         Texture2D shadowMap;
 
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
 

            effect = Content.Load<Effect> ("OurHLSLfile");            SetUpVertices();
            SetUpCamera();


            streetTexture = Content.Load<Texture2D> ("streettexture");            carModel = LoadModel("car", out carTextures);

            PresentationParameters pp = device.PresentationParameters;
            renderTarget = new RenderTarget2D(device, pp.BackBufferWidth, pp.BackBufferHeight, true, device.DisplayMode.Format, DepthFormat.Depth24);
        }

        private Model LoadModel(string assetName, out Texture2D[] textures)
        {

            Model newModel = Content.Load<Model> (assetName);
            textures = new Texture2D[7];
            int i = 0;
            foreach (ModelMesh mesh in newModel.Meshes)
                foreach (BasicEffect currentEffect in mesh.Effects)
                    textures[i++] = currentEffect.Texture;

            foreach (ModelMesh mesh in newModel.Meshes)
                foreach (ModelMeshPart meshPart in mesh.MeshParts)
                    meshPart.Effect = effect.Clone();

            return newModel;
        }

        private void SetUpVertices()
        {
            MyOwnVertexFormat[] vertices = new MyOwnVertexFormat[18];

            vertices[0] = new MyOwnVertexFormat(new Vector3(-20, 0, 10), new Vector2(-0.25f, 25.0f), new Vector3(0, 1, 0));
            vertices[1] = new MyOwnVertexFormat(new Vector3(-20, 0, -100), new Vector2(-0.25f, 0.0f), new Vector3(0, 1, 0));
            vertices[2] = new MyOwnVertexFormat(new Vector3(2, 0, 10), new Vector2(0.25f, 25.0f), new Vector3(0, 1, 0));
            vertices[3] = new MyOwnVertexFormat(new Vector3(2, 0, -100), new Vector2(0.25f, 0.0f), new Vector3(0, 1, 0));
            vertices[4] = new MyOwnVertexFormat(new Vector3(2, 0, 10), new Vector2(0.25f, 25.0f), new Vector3(-1, 0, 0));
            vertices[5] = new MyOwnVertexFormat(new Vector3(2, 0, -100), new Vector2(0.25f, 0.0f), new Vector3(-1, 0, 0));
            vertices[6] = new MyOwnVertexFormat(new Vector3(2, 1, 10), new Vector2(0.375f, 25.0f), new Vector3(-1, 0, 0));
            vertices[7] = new MyOwnVertexFormat(new Vector3(2, 1, -100), new Vector2(0.375f, 0.0f), new Vector3(-1, 0, 0));
            vertices[8] = new MyOwnVertexFormat(new Vector3(2, 1, 10), new Vector2(0.375f, 25.0f), new Vector3(0, 1, 0));
            vertices[9] = new MyOwnVertexFormat(new Vector3(2, 1, -100), new Vector2(0.375f, 0.0f), new Vector3(0, 1, 0));
            vertices[10] = new MyOwnVertexFormat(new Vector3(3, 1, 10), new Vector2(0.5f, 25.0f), new Vector3(0, 1, 0));
            vertices[11] = new MyOwnVertexFormat(new Vector3(3, 1, -100), new Vector2(0.5f, 0.0f), new Vector3(0, 1, 0));
            vertices[12] = new MyOwnVertexFormat(new Vector3(13, 1, 10), new Vector2(0.75f, 25.0f), new Vector3(0, 1, 0));
            vertices[13] = new MyOwnVertexFormat(new Vector3(13, 1, -100), new Vector2(0.75f, 0.0f), new Vector3(0, 1, 0));
            vertices[14] = new MyOwnVertexFormat(new Vector3(13, 1, 10), new Vector2(0.75f, 25.0f), new Vector3(-1, 0, 0));
            vertices[15] = new MyOwnVertexFormat(new Vector3(13, 1, -100), new Vector2(0.75f, 0.0f), new Vector3(-1, 0, 0));
            vertices[16] = new MyOwnVertexFormat(new Vector3(13, 21, 10), new Vector2(1.25f, 25.0f), new Vector3(-1, 0, 0));
            vertices[17] = new MyOwnVertexFormat(new Vector3(13, 21, -100), new Vector2(1.25f, 0.0f), new Vector3(-1, 0, 0));

            vertexBuffer = new VertexBuffer(device, MyOwnVertexFormat.VertexDeclaration, vertices.Length, BufferUsage.WriteOnly);
            vertexBuffer.SetData(vertices);
        }

        private void SetUpCamera()
        {
            cameraPos = new Vector3(-25, 13, 18);
            viewMatrix = Matrix.CreateLookAt(cameraPos, new Vector3(0, 2, -12), new Vector3(0, 1, 0));
            projectionMatrix = Matrix.CreatePerspectiveFieldOfView(MathHelper.PiOver4, device.Viewport.AspectRatio, 1.0f, 200.0f);
        }

        protected override void UnloadContent()
        {
        }

        protected override void Update(GameTime gameTime)
        {
            if (GamePad.GetState(PlayerIndex.One).Buttons.Back == ButtonState.Pressed)
                this.Exit();

            UpdateLightData();

            base.Update(gameTime);
        }

        private void UpdateLightData()
        {
            ambientPower = 0.2f;

            lightPos = new Vector3(-18, 5, -2);
            lightPower = 1.0f;

            Matrix lightsView = Matrix.CreateLookAt(lightPos, new Vector3(-2, 3, -10), new Vector3(0, 1, 0));
            Matrix lightsProjection = Matrix.CreatePerspectiveFieldOfView(MathHelper.PiOver2, 1f, 5f, 100f);

            lightsViewProjectionMatrix = lightsView * lightsProjection;
        }


        protected override void Draw(GameTime gameTime)
        {

             device.SetRenderTarget(renderTarget);            
             device.Clear(ClearOptions.Target | ClearOptions.DepthBuffer, Color.Black, 1.0f, 0);
 
             DrawScene("ShadowMap");
             device.SetRenderTarget(null);
             shadowMap = (Texture2D)renderTarget;
 
             device.Clear(ClearOptions.Target | ClearOptions.DepthBuffer, Color.DarkSlateBlue, 1.0f, 0);
             using (SpriteBatch sprite = new SpriteBatch(device))
             {
                 sprite.Begin();
                 sprite.Draw(shadowMap, new Vector2(0, 0), null, Color.White, 0, new Vector2(0, 0), 0.4f, SpriteEffects.None, 1);
                 sprite.End();
             }
 
             base.Draw(gameTime);
         }
 
         private void DrawScene(string technique)
         {
             effect.CurrentTechnique = effect.Techniques[technique];
             effect.Parameters["xWorldViewProjection"].SetValue(Matrix.Identity * viewMatrix * projectionMatrix);
             effect.Parameters["xTexture"].SetValue(streetTexture);
             effect.Parameters["xWorld"].SetValue(Matrix.Identity);
             effect.Parameters["xLightPos"].SetValue(lightPos);
             effect.Parameters["xLightPower"].SetValue(lightPower);
             effect.Parameters["xAmbient"].SetValue(ambientPower);
             effect.Parameters["xLightsWorldViewProjection"].SetValue(Matrix.Identity * lightsViewProjectionMatrix);
 
             foreach (EffectPass pass in effect.CurrentTechnique.Passes)
             {
                 pass.Apply();
 
                 device.SetVertexBuffer(vertexBuffer);
                 device.DrawPrimitives(PrimitiveType.TriangleStrip, 0, 16);
             }
 
             Matrix car1Matrix = Matrix.CreateScale(4f) * Matrix.CreateRotationY(MathHelper.Pi) * Matrix.CreateTranslation(-3, 0, -15);
             DrawModel(carModel, carTextures, car1Matrix, technique);
 
             Matrix car2Matrix = Matrix.CreateScale(4f) * Matrix.CreateRotationY(MathHelper.Pi * 5.0f / 8.0f) * Matrix.CreateTranslation(-28, 0, -1.9f);
             DrawModel(carModel, carTextures, car2Matrix, technique);
         }
 
         private void DrawModel(Model model, Texture2D[] textures, Matrix wMatrix, string technique)
         {
             Matrix[] modelTransforms = new Matrix[model.Bones.Count];
             model.CopyAbsoluteBoneTransformsTo(modelTransforms);
             int i = 0;
             foreach (ModelMesh mesh in model.Meshes)
             {
                 foreach (Effect currentEffect in mesh.Effects)
                 {
                     Matrix worldMatrix = modelTransforms[mesh.ParentBone.Index] * wMatrix;
                     currentEffect.CurrentTechnique = currentEffect.Techniques[technique];
                     currentEffect.Parameters["xWorldViewProjection"].SetValue(worldMatrix * viewMatrix * projectionMatrix);
                     currentEffect.Parameters["xTexture"].SetValue(textures[i++]);
                     currentEffect.Parameters["xWorld"].SetValue(worldMatrix);
                     currentEffect.Parameters["xLightPos"].SetValue(lightPos);
                     currentEffect.Parameters["xLightPower"].SetValue(lightPower);
                     currentEffect.Parameters["xAmbient"].SetValue(ambientPower);
                     currentEffect.Parameters["xLightsWorldViewProjection"].SetValue(worldMatrix * lightsViewProjectionMatrix);
                 }
                 mesh.Draw();
             }
         }
 
     }
 }
```

## Next Steps

[Transforming vertices](Riemers3DXNA3hlsl14projectivetexturing)
