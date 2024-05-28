# World transformation

In HLSL, you have to perform all transformations yourself. As we’ve already seen, most of them are performed in the vertex shader. To give all people the possibility of completely understanding what they’re doing in this 3rd Series, I’ve decided to write this extra chapter on the World transform.

We’ll be drawing the cars from model files, so you would like to add these variables:

```csharp
 Model carModel;
 Texture2D[] carTextures;
```

As you can see, the car model will contain textures. As always, we need to start with loading the model into our Solution Explorer, like you’ve done many times before. You can download the mesh here.

Next, we need to link our variables to our assets, which we have done before in the chapters “Loading a Model” and “Skybox” of Series2. This method comes straight from the latter:

```csharp
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
```

This code is slightly different, as the texture array has a fixed size of 7. This is because each ModelMesh can have multiple ModelMeshParts, and thus multiple effects. For a detailed description of the Model structure, see Recipes 4-1 and 4-8.

Use this method to load both your Models from within your LoadContent method:

```csharp
 carModel = LoadModel("car", out carTextures);
```

We load the model from file, and save their textures.

OK, with the model loaded, it’s time to draw them. But don’t we first have to set the World transform ?

Remember when the World transform is used? Whenever you want to draw some triangles (from a vertex buffer or from a mesh), you have to set the World transform first. If you wouldn’t, all triangles would be drawn relative to the origin, the (0,0,0) point.

Imagine you want to draw 2 objects from the same mesh, like our 2 lampposts. If you wouldn’t set a different world transform before drawing them, both objects would be drawn at the same place, so you would see only one of them. What you would rather like to do, is tell XNA to draw the first object ‘3 units to the left and 2 units up’, and draw the second object ‘3 units to the right and rotated around the Y axis for 40 degrees and twice as big as the first one’. This ‘transformation’ is called the World transform, and is stored in a matrix. For more info on matrices, what they look like and what you can do with them, you can check out the matrix entries in the Extra Reading section.

Something like the example above is displayed in the image below: the big axes represent our World axes, with its origin in the World (0,0,0) point. Say you would like to draw the first object from a mesh. First, you would have to tell XNA to create a new axis, with origin where you would like the center of the object to be drawn. This new location, as well as its rotation and its scaling, are stored in a matrix M1. When you draw your mesh, the mesh will be drawn around the new axis.

![Transforms](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA3-09WorldTransform1.jpg?raw=true)

The same story for the second object: you first have to set M2 as World transform, so object 2 will be drawn around the correct axis. To obtain this, we have to multiply our vertices with this World matrix in our vertex shader.

The correct order of multiplication would be: World, View and then Projection. We will perform this multiplication in our XNA app, so we need to perform only the multiplication of our vertices with this combined matrix in our vertex shader. So go to our .fx file, and change all instances of xViewProjection to xWorldViewProjection:

```csharp
float4x4 xWorldViewProjection;
```

And

```csharp
Output.Position = mul(inPos, xWorldViewProjection);
```

The first line means HLSL expects the XNA app to set this matrix, and the second line (in the vertex shader) multiplies every vertex with this combined matrix. The result is that the position of each vertex is first transformed to its proper local axis, and then transformed to 2D screen coordinates!

Now we still need to fill this matrix. Go to the Draw method in our XNA code, and replace the line where you fill the xViewProjection matrix by this line:

```csharp
 effect.Parameters["xWorldViewProjection"].SetValue(Matrix.Identity * viewMatrix * projectionMatrix);
```

Because our TriangleStrip containing the street actually needs to be drawn around the (0,0,0) World origin, we don’t need a World matrix for the street. In that case, you can specify Matrix.Identity, which is the unity element in matrix maths: multiplying matrix M by the identity matrix gives M. More applicable for us: multiplying position P with the identity matrix, gives P again.

When you run this code, you should see exactly the same as last chapter, which is OK. Now we’re going to add the first car. The DrawModel method below is based on the DrawSkybox method of last chapter. I have changed it a bit, so it is usable to render any textured Model at any position, as you can specify the Model itself, its textures and its World matrix. Furthermore, as later on we will want to render Models with different techniques, you can also specify the technique to use.

```csharp
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
             currentEffect.CurrentTechnique = currentEffect.Techniques[technique];                                            currentEffect.Parameters["xWorldViewProjection"].SetValue(worldMatrix* viewMatrix*projectionMatrix);
                          currentEffect.Parameters["xTexture"].SetValue(textures[i++]);
         }
         mesh.Draw();
     }
 }
```

There are only a few small differences between this method and the DrawSkybox method, which has been explained in the “Skybox” chapter of Series 2.

Using the DrawModel method, it’s very easy to render a car to our scene. Just put these lines in your Draw method:

```csharp
 Matrix car1Matrix = Matrix.CreateScale(4f) * Matrix.CreateRotationY(MathHelper.Pi) * Matrix.CreateTranslation(-3, 0, -15);
 DrawModel(carModel, carTextures, car1Matrix, "Simplest", false);
```

The first line creates the World matrix for our first car. First, the car is moved to its correct position, where it is rotated so it points into the correct direction. Finally, the car is scaled up a bit so it fits nicely in our scene.

To learn all about the order of matrix multiplication, read Recipe 4-2.

Next, you call the DrawModel method, to which you obviously need to pass the Model, its textures and this World matrix. You also specify the technique, while the last argument indicates the color should be sampled from the textures.

The code that renders the second car is exactly the same, except that it has a different rotation and position:

```csharp
 Matrix car2Matrix = Matrix.CreateScale(4f) * Matrix.CreateRotationY(MathHelper.Pi * 5.0f / 8.0f) * Matrix.CreateTranslation(-28, 0, -1.9f);
 DrawModel(carModel, carTextures, car2Matrix, "Simplest", false);
```

(Previously, I had also two lampposts in the scene, but XNA4.0 doesn’t like them as they didn’t have texture information. If you have 3D models of a textured lamppost (or anything else that could be added to the scene), feel free to send it over!)

When you run this code, you should see 2 nicely textured cars, as the pixel shader will return a black color if no texture has been set (remember this, as it might help you later when debugging your own program).

For some reason, the texture coordinates in the car model are inside the [1,2] region instead of in the [0,1] region. Since we're using the Mirror texture addressing mode, this will result in the textures being applied mirrored on the car. If you want to compensate for this, add this line to the beginning of your pixel shader:

```csharp
PSIn.TexCoords.y--;
```

![Summary](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA3-09WorldTransform2.jpg?raw=true)

The scene looks a bit dull, because only ambient lighting has been used. In the next chapter, we’ll be adding normals to our vertices. These are needed before we can move on to our first light!

> You can try these exercises to practice what you've learned:
>
> - Add another car to your scene, in the middle of the street.
> - Rotate this third car.

# The HLSL code thus far

```csharp
float4x4 xWorldViewProjection;

Texture xTexture;

sampler TextureSampler = sampler_state { texture = <xTexture> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = mirror; AddressV = mirror;};
struct VertexToPixel
{
    float4 Position     : POSITION;    
    float2 TexCoords    : TEXCOORD0;
};

struct PixelToFrame
{
    float4 Color        : COLOR0;
};

VertexToPixel SimplestVertexShader( float4 inPos : POSITION, float2 inTexCoords : TEXCOORD0)
{
    VertexToPixel Output = (VertexToPixel)0;
    
    Output.Position =mul(inPos, xWorldViewProjection);
    Output.TexCoords = inTexCoords;

    return Output;
}

PixelToFrame OurFirstPixelShader(VertexToPixel PSIn)
{
    PixelToFrame Output = (PixelToFrame)0;    

    PSIn.TexCoords.y--;
    Output.Color = tex2D(TextureSampler, PSIn.TexCoords);

    return Output;
}

technique Simplest
{
    pass Pass0
    {
        VertexShader = compile vs_2_0 SimplestVertexShader();
        PixelShader = compile ps_2_0 OurFirstPixelShader();
    }
}
```

## And the XNA code

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
 
         public MyOwnVertexFormat(Vector3 position, Vector2 texCoord)
         {
             this.position = position;
             this.texCoord = texCoord;
         }
 
         public readonly static VertexDeclaration VertexDeclaration = new VertexDeclaration
              (
                  new VertexElement(0, VertexElementFormat.Vector3, VertexElementUsage.Position, 0),
                  new VertexElement(sizeof(float) * 3, VertexElementFormat.Vector2, VertexElementUsage.TextureCoordinate, 0)
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
         Model carModel;
         Texture2D[] carTextures;
 
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
             MyOwnVertexFormat[] vertices = new MyOwnVertexFormat[12];
 
             vertices[0] = new MyOwnVertexFormat(new Vector3(-20, 0, 10), new Vector2(-0.25f, 25.0f));
             vertices[1] = new MyOwnVertexFormat(new Vector3(-20, 0, -100), new Vector2(-0.25f, 0.0f));
             vertices[2] = new MyOwnVertexFormat(new Vector3(2, 0, 10), new Vector2(0.25f, 25.0f));
             vertices[3] = new MyOwnVertexFormat(new Vector3(2, 0, -100), new Vector2(0.25f, 0.0f));
             vertices[4] = new MyOwnVertexFormat(new Vector3(2, 1, 10), new Vector2(0.375f, 25.0f));
             vertices[5] = new MyOwnVertexFormat(new Vector3(2, 1, -100), new Vector2(0.375f, 0.0f));
             vertices[6] = new MyOwnVertexFormat(new Vector3(3, 1, 10), new Vector2(0.5f, 25.0f));
             vertices[7] = new MyOwnVertexFormat(new Vector3(3, 1, -100), new Vector2(0.5f, 0.0f));
             vertices[8] = new MyOwnVertexFormat(new Vector3(13, 1, 10), new Vector2(0.75f, 25.0f));
             vertices[9] = new MyOwnVertexFormat(new Vector3(13, 1, -100), new Vector2(0.75f, 0.0f));
             vertices[10] = new MyOwnVertexFormat(new Vector3(13, 21, 10), new Vector2(1.25f, 25.0f));
             vertices[11] = new MyOwnVertexFormat(new Vector3(13, 21, -100), new Vector2(1.25f, 0.0f));
 
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
 
             base.Update(gameTime);
         }
 
         protected override void Draw(GameTime gameTime)
         {
             device.Clear(ClearOptions.Target | ClearOptions.DepthBuffer, Color.DarkSlateBlue, 1.0f, 0);
 
             effect.CurrentTechnique = effect.Techniques["Simplest"];
             effect.Parameters["xWorldViewProjection"].SetValue(Matrix.Identity * viewMatrix * projectionMatrix);
             effect.Parameters["xTexture"].SetValue(streetTexture);
 
             foreach (EffectPass pass in effect.CurrentTechnique.Passes)
             {
                 pass.Apply();
 
                 device.SetVertexBuffer(vertexBuffer);
                 device.DrawPrimitives(PrimitiveType.TriangleStrip, 0, 10);
             }
 
             Matrix car1Matrix = Matrix.CreateScale(4f) * Matrix.CreateRotationY(MathHelper.Pi) * Matrix.CreateTranslation(-3, 0, -15);
             DrawModel(carModel, carTextures, car1Matrix, "Simplest");
 
             Matrix car2Matrix = Matrix.CreateScale(4f) * Matrix.CreateRotationY(MathHelper.Pi * 5.0f / 8.0f) * Matrix.CreateTranslation(-28, 0, -1.9f);
             DrawModel(carModel, carTextures, car2Matrix, "Simplest");
 
             base.Draw(gameTime);
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
                 }
                 mesh.Draw();
             }
         }
 
     }
 }
```

## Next Steps

[Adding normals](Riemers3DXNA3hlsl10worldnormals)
