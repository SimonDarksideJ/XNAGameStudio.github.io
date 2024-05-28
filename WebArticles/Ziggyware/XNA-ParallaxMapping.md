# Parallax Mapping in XNA


Here is a sample of using Parallax mapping in XNA using the sample shader provided in the August 2006 DirectX SDK from Microsoft ([Here](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/directx9_c/ParallaxOcclusionMapping_Sample.asp))

First off lsets define the assemblies we will be requiring:

```csharp
using System;
using System.Xml;
using System.Collections.Generic;
using Microsoft.Xna.Framework;
using Microsoft.Xna.Framework.Audio;
using Microsoft.Xna.Framework.Components;
using Microsoft.Xna.Framework.Graphics;
using Microsoft.Xna.Framework.Input;
using Microsoft.Xna.Framework.Storage;
```


Here begins the main namespace for our game:

```csharp
namespace FarkleXNA
{

    public class Game1 : Microsoft.Xna.Framework.Game
    {
```


Lets declare a HighPerformanceTimer.

(see the previous tutorials for explanations of these items)

```csharp
        HighPerformanceTimer timer;
        Vector3 vCamPos;
        Vector3 vCamDir;

        Matrix viewMatrix, projMatrix;

        Die testDie;

        public Game1()
        {
            InitializeComponent();
        }
```

Initialize your objects in the initialize function:

```csharp
        protected override void Initialize()
        {
            timer = new HighPerformanceTimer();

            vCamPos = new Vector3(1, 1,1.5f);
            vCamDir = Vector3.Normalize(vCamPos);

            viewMatrix = Matrix.CreateLookAt(vCamPos, Vector3.Zero, Vector3.Up);
                        
            projMatrix = Matrix.CreatePerspectiveFieldOfView(
                (float)Math.PI / 2.0f, 
                (float)Window.ClientWidth / (float)Window.ClientHeight, 
                0.1f, 
                1000.0f);
            graphics.AllowMultiSampling = false;
            graphics.ApplyChanges();

            testDie = new Die(graphics.GraphicsDevice);
        }
```


In the Update method you should do any per-frame calculations that are needed

```csharp
        protected override void Update()
        {
            // The time since Update was called last
            float elapsed = timer.Elapsed;

            // TODO: Add your game logic here


            testDie.Update(elapsed);

            // Let the GameComponents update
            UpdateComponents();
        }
```


The draw method is where we will render the Parallax mapped dice

```csharp
        protected override void Draw()
        {
            // Make sure we have a valid device
            if (!graphics.EnsureDevice())
                return;

            graphics.GraphicsDevice.Clear(Color.CornflowerBlue);
            graphics.GraphicsDevice.BeginScene();

            // TODO: Add your drawing code here
```


Pass in the current camera position, camera direction, view and projection matrices( for the shader)

```csharp
            testDie.Render(graphics.GraphicsDevice,
                vCamPos,
                vCamDir, 
                viewMatrix, 
                projMatrix);

         
             // Let the GameComponents draw
            DrawComponents();

            

            graphics.GraphicsDevice.EndScene();
            graphics.GraphicsDevice.Present();
        }
    }
```


Here is the Die class for rendering dice:

```csharp
    public class Die
    {
        Texture2D[,] Textures;
        public VertexBuffer vb;
        static Vector3[] vNormals = new Vector3[6];

        Effect effectNormalMapping;
        Matrix world;
        float fRot=0;
```


Always have a dispose method to clean up any data that is created for this object

```csharp
        public void Dispose()
        {
            foreach (Texture2D tex in Textures)
                tex.Dispose();
            Textures = null;

            vb.Dispose();

            effectNormalMapping.Dispose();
        }
```


The constructor 

```csharp
        public Die(GraphicsDevice device)
        {

```


Populate the vertices

```csharp
            //one
            verts[0] = new VertexPosTexNormalTanBitan(
                new Vector3(-1, 1, -1), new Vector2(0, 0), 
                new Vector3(0, 1, 0), new Vector3(1, 0, 0), new Vector3(0, 0, 1));
            verts[1] = new VertexPosTexNormalTanBitan(
                new Vector3(1, 1, -1), new Vector2(1, 0), 
                new Vector3(0, 1, 0), new Vector3(1, 0, 0), new Vector3(0, 0, 1));
            verts[2] = new VertexPosTexNormalTanBitan(
                new Vector3(1, 1, 1), new Vector2(1, 1), 
                new Vector3(0, 1, 0), new Vector3(1, 0, 0), new Vector3(0, 0, 1));
            verts[3] = new VertexPosTexNormalTanBitan(
                new Vector3(-1, 1, 1), new Vector2(0, 1), 
                new Vector3(0, 1, 0), new Vector3(1, 0, 0), new Vector3(0, 0, 1));

            //two
            verts[4] = new VertexPosTexNormalTanBitan(
                new Vector3(1,-1,-1), new Vector2(0, 0), 
                new Vector3(1, 0, 0), new Vector3(0, 0, 1), new Vector3(0, 1, 0));
            verts[5] = new VertexPosTexNormalTanBitan(
                new Vector3(1,-1, 1), new Vector2(1, 0), 
                new Vector3(1, 0, 0), new Vector3(0, 0, 1), new Vector3(0, 1, 0));
            verts[6] = new VertexPosTexNormalTanBitan(
                new Vector3(1, 1, 1), new Vector2(1, 1), 
                new Vector3(1, 0, 0), new Vector3(0, 0, 1), new Vector3(0, 1, 0));
            verts[7] = new VertexPosTexNormalTanBitan(
                new Vector3(1, 1,-1), new Vector2(0, 1), 
                new Vector3(1, 0, 0), new Vector3(0, 0, 1), new Vector3(0, 1, 0));

            //three
            verts[8] = new VertexPosTexNormalTanBitan(
                new Vector3(1, -1, 1), new Vector2(0, 0), 
                new Vector3(0, 0, 1), new Vector3(-1, 0, 0), new Vector3(0, 1, 0));
            verts[9] = new VertexPosTexNormalTanBitan(
                new Vector3(-1,-1, 1), new Vector2(1, 0), 
                new Vector3(0, 0, 1), new Vector3(-1, 0, 0), new Vector3(0, 1, 0));
            verts[10] = new VertexPosTexNormalTanBitan(
                new Vector3(-1, 1,  1), new Vector2(1, 1), 
                new Vector3(0, 0, 1), new Vector3(-1, 0, 0), new Vector3(0, 1, 0));
            verts[11] = new VertexPosTexNormalTanBitan(
                new Vector3(1, 1,  1), new Vector2(0, 1), 
                new Vector3(0, 0, 1), new Vector3(-1, 0, 0), new Vector3(0, 1, 0));
            
            //four
            verts[12] = new VertexPosTexNormalTanBitan(
                new Vector3(-1,-1, -1), new Vector2(0, 0), 
                new Vector3(0, 0, -1), new Vector3(1, 0, 0), new Vector3(0, 1, 0));
            verts[13] = new VertexPosTexNormalTanBitan(
                new Vector3( 1,-1, -1), new Vector2(1, 0), 
                new Vector3(0, 0, -1), new Vector3(1, 0, 0), new Vector3(0, 1, 0));
            verts[14] = new VertexPosTexNormalTanBitan(
                new Vector3( 1, 1, -1), new Vector2(1, 1), 
                new Vector3(0, 0, -1), new Vector3(1, 0, 0), new Vector3(0, 1, 0));
            verts[15] = new VertexPosTexNormalTanBitan(
                new Vector3(-1, 1, -1), new Vector2(0, 1), 
                new Vector3(0, 0, -1), new Vector3(1, 0, 0), new Vector3(0, 1, 0));
            
            

            //five
            verts[16] = new VertexPosTexNormalTanBitan(
                new Vector3(-1, -1, 1), new Vector2(0, 0), 
                new Vector3(-1, 0, 0), new Vector3(0, 0, -1), new Vector3(0, 1, 0));
            verts[17] = new VertexPosTexNormalTanBitan(
                new Vector3(-1, -1,-1), new Vector2(1, 0), 
                new Vector3(-1, 0, 0), new Vector3(0, 0, -1), new Vector3(0, 1, 0));
            verts[18] = new VertexPosTexNormalTanBitan(
                new Vector3(-1, 1,-1), new Vector2(1, 1), 
                new Vector3(-1, 0, 0), new Vector3(0, 0, -1), new Vector3(0, 1, 0));
            verts[19] = new VertexPosTexNormalTanBitan(
                new Vector3(-1, 1, 1), new Vector2(0, 1), 
                new Vector3(-1, 0, 0), new Vector3(0, 0, -1), new Vector3(0, 1, 0));
            
            

            //six
            verts[20] = new VertexPosTexNormalTanBitan(
                new Vector3( 1, -1, -1), new Vector2(0, 0), 
                new Vector3(0, -1, 0), new Vector3(-1, 0, 0), new Vector3(0, 0, 1));
            verts[21] = new VertexPosTexNormalTanBitan(
                new Vector3(-1, -1,-1), new Vector2(1, 0), 
                new Vector3(0, -1, 0), new Vector3(-1, 0, 0), new Vector3(0, 0, 1));
            verts[22] = new VertexPosTexNormalTanBitan(
                new Vector3(-1, -1, 1), new Vector2(1, 1), 
                new Vector3(0, -1, 0), new Vector3(-1, 0, 0), new Vector3(0, 0, 1));
            verts[23] = new VertexPosTexNormalTanBitan(
                new Vector3(1, -1, 1), new Vector2(0, 1), 
                new Vector3(0, -1, 0), new Vector3(-1, 0, 0), new Vector3(0, 0, 1));
```


Pre-compute the side normals so we can optimize the rendering to exclude non visible sides of the dice

```csharp
            for (int x = 0; x < 6; x++)
            {
                vNormals[x] = verts[x * 4].Normal;
            }
```


Setup the new vertex buffer and pass it the data we defined above:

```csharp
            vb = new VertexBuffer(device, typeof(VertexPosTexNormalTanBitan), verts.Length, BufferUsage.WriteOnly);
            vb.SetData(verts);
```


Lets load up the ParallaxOcclusionMapping effect, a new LoadContent method, along with the textures needed (see previous tutorial on HLSL effect files)

```csharp
        public void LoadContent(GraphicsDevice device, ContentManager content)
        {
            effectNormalMapping = content.Load<Effect>("ParallaxOcclusionMapping");
            Textures = new Texture2D[6, 2];

            for (int x = 0; x < 6; x++)
            {
                Textures[x, 0] = content.Load<Texture2D>("textures\\" + (x + 1).ToString());
                    
                Textures[x, 1] = content.Load<Texture2D>("textures\\" + (x + 1).ToString() + ".Normal");

            }
        }
```


Initialize the Die's world transform to Identity

```csharp
            world = Matrix.Identity;
        }
```


On Update() I am incrementing a counter to be used to rotate the object around to see all sides of the die


```csharp
        public void Update(float fTime)
        {
            fRot += fTime;
        }
```


Heres the render method 

```csharp
        public void Render(GraphicsDevice device,
                            Vector3 vCamPos,
                            Vector3 vCamDir, 
                            Matrix view,
                            Matrix proj)
        {
```


Create a rotation matrix to spin the Die around

```csharp
            world = Matrix.CreateRotationZ(fRot);
            world = world * Matrix.CreateRotationX((float)Math.Cos((float)fRot) * 1.5f);
```

Set the default properties in the Effect

```csharp
            effectNormalMapping.Parameters["g_materialAmbientColor"].SetValue(
                new Vector4(.35f, .35f, .35f, 0));
            effectNormalMapping.Parameters["g_materialDiffuseColor"].SetValue(
                new Vector4(1, 1, 1, 1));
            effectNormalMapping.Parameters["g_materialSpecularColor"].SetValue(
                new Vector4(1, 1, 1, 1));

            effectNormalMapping.Parameters["g_fSpecularExponent"].SetValue(120.0f);
            effectNormalMapping.Parameters["g_bAddSpecular"].SetValue(true);

            effectNormalMapping.Parameters["g_LightDir"].SetValue(Vector3.Normalize(
                new Vector3(0, 0, 1)));
            effectNormalMapping.Parameters["g_LightDiffuse"].SetValue(
                new Vector4(1, 1, 1, 1));

            effectNormalMapping.Parameters["g_vEye"].SetValue(new Vector4(vCamPos, 1));

            effectNormalMapping.Parameters["g_fBaseTextureRepeat"].SetValue(1.0f);
            effectNormalMapping.Parameters["g_fHeightMapScale"].SetValue(0.17f);

            effectNormalMapping.Parameters["g_mWorldViewProjection"].SetValue(
                world * view * proj);
            effectNormalMapping.Parameters["g_mWorld"].SetValue(world);
            effectNormalMapping.Parameters["g_mView"].SetValue(view);

            effectNormalMapping.Parameters["g_bVisualizeLOD"].SetValue(false);
            effectNormalMapping.Parameters["g_bVisualizeMipLevel"].SetValue(false);
            effectNormalMapping.Parameters["g_bDisplayShadows"].SetValue(true);

            effectNormalMapping.Parameters["g_vTextureDims"].SetValue(
                        new Vector2(Textures[0, 0].Width, Textures[0, 0].Height));

            effectNormalMapping.Parameters["g_nLODThreshold"].SetValue(5);
            effectNormalMapping.Parameters["g_fShadowSoftening"].SetValue(0.58f);
            effectNormalMapping.Parameters["g_nMinSamples"].SetValue(10);
            effectNormalMapping.Parameters["g_nMaxSamples"].SetValue(100);
```


transform the camera direction into model space for faster comparisons

```csharp
            Vector3 vCamDirLocal = Vector3.TransformNormal(vCamDir, Matrix.Invert(world));
            for (int x = 0; x < 6; x++)
            {
```


check the angle between the camera direction and the face normal to see if it should be rendered or not 

```csharp
                if (Vector3.Dot(vNormals[x], vCamDirLocal) < 0)
                    continue;
```


This polygon is facing the camera so lets go ahead and set the diffuse and normal map's

```csharp
                effectNormalMapping.Parameters["g_baseTexture"].SetValue(Textures[x, 0]);
                effectNormalMapping.Parameters["g_nmhTexture"].SetValue(Textures[x, 1]);
            }
```

With everything defined, all we need to do is setup the Vertex Buffer and set it drawing, as follows: Set the device's VertexDeclaration
```csharp
            device.SetVertexBuffer(vb);

            foreach (EffectPass pass in effectNormalMapping.CurrentTechnique.Passes)
            {
                pass.Apply();
                device.DrawPrimitives(PrimitiveType.TriangleList, 0, 12);
            }
        }
}
```

ANd here is the ParallaxOcclusionMapping effect file for adding in to your content project.
```hlsl
#if OPENGL
	#define SV_POSITION POSITION
	#define VS_SHADERMODEL vs_3_0
	#define PS_SHADERMODEL ps_3_0
#else
	#define VS_SHADERMODEL vs_4_0_level_9_1
	#define PS_SHADERMODEL ps_4_0_level_9_1
#endif
//--------------------------------------------------------------------------------------
// File: ParallaxOcclusionMapping.fx
//
// Parallax occlusion mapping implementation
//                                        
// Implementation of the algorithm as described in "Dynamic Parallax Occlusion
// Mapping with Approximate Soft Shadows" paper, by N. Tatarchuk, ATI Research, 
// to appear in the proceedings of ACM Symposium on Interactive 3D Graphics and Games, 2006.
//                                                                               
// For examples of use in a real-time scene, see ATI X1K demo "ToyShop":         
//    http://www.ati.com/developer/demos/rx1800.html                             
//
// Copyright (c) ATI Research, Inc. All rights reserved.
//--------------------------------------------------------------------------------------

//--------------------------------------------------------------------------------------
// Global variables
//--------------------------------------------------------------------------------------

// Base color texture
texture g_baseTexture;              
// Normal map and height map texture pair
texture g_nmhTexture;               

// Material's ambient color
float4 g_materialAmbientColor;      
// Material's diffuse color
float4 g_materialDiffuseColor;      
// Material's specular color
float4 g_materialSpecularColor;     

// Material's specular exponent
float  g_fSpecularExponent;         
// Toggles rendering with specular or without
bool   g_bAddSpecular;              

// Light parameters:

// Light's direction in world space
float3 g_LightDir;                  
// Light's diffuse color
float4 g_LightDiffuse;              
// Light's ambient color
float4 g_LightAmbient;              

// Camera's location
float4   g_vEye;                    
// The tiling factor for base and normal map textures
float    g_fBaseTextureRepeat;      
// Describes the useful range of values for the height field
float    g_fHeightMapScale;         

// Matrices:

// World matrix for object
float4x4 g_mWorld;                  
// World * View * Projection matrix
float4x4 g_mWorldViewProjection;    
// View matrix 
float4x4 g_mView;                   

// Toggles visualization of level of detail colors
bool     g_bVisualizeLOD;           
// Toggles visualization of mip level
bool     g_bVisualizeMipLevel;      
// Toggles display of self-occlusion based shadows
bool     g_bDisplayShadows;         

// Specifies texture dimensions for computation of mip level at 
// render time (width, height)
float2   g_vTextureDims;            

// The mip level id for transitioning between the full computation
// for parallax occlusion mapping and the bump mapping computation
int      g_nLODThreshold;           

// Blurring factor for the soft shadows computation
float    g_fShadowSoftening;        

// The minimum number of samples for sampling the height field profile
int      g_nMinSamples;             
// The maximum number of samples for sampling the height field profile
int      g_nMaxSamples;             



//--------------------------------------------------------------------------------------
// Texture samplers
//--------------------------------------------------------------------------------------
sampler tBase = 
sampler_state
{
    Texture = < g_baseTexture >;
    MipFilter = LINEAR;
    MinFilter = LINEAR;
    MagFilter = LINEAR;
};
sampler tNormalHeightMap = 
sampler_state
{
    Texture = < g_nmhTexture >;
    MipFilter = LINEAR;
    MinFilter = LINEAR;
    MagFilter = LINEAR;
};


//--------------------------------------------------------------------------------------
// Vertex shader output structure
//--------------------------------------------------------------------------------------
struct VS_OUTPUT
{
    float4 position          : POSITION;
    float2 texCoord          : TEXCOORD0;
    float3 vLightTS          : TEXCOORD1;   // light vector in tangent space, denormalized
    float3 vViewTS           : TEXCOORD2;   // view vector in tangent space, denormalized
    float2 vParallaxOffsetTS : TEXCOORD3;   // Parallax offset vector in tangent space
    float3 vNormalWS         : TEXCOORD4;   // Normal vector in world space
    float3 vViewWS           : TEXCOORD5;   // View vector in world space
    
};  


//--------------------------------------------------------------------------------------
// This shader computes standard transform and lighting
//--------------------------------------------------------------------------------------
VS_OUTPUT RenderSceneVS( float4 inPositionOS  : POSITION, 
                         float2 inTexCoord    : TEXCOORD0,
                         float3 vInNormalOS   : NORMAL,
                         float3 vInBinormalOS : BINORMAL,
                         float3 vInTangentOS  : TANGENT )
{
    VS_OUTPUT Out;
        
    // Transform and output input position 
    Out.position = mul( inPositionOS, g_mWorldViewProjection );
       
    // Propagate texture coordinate through:
    Out.texCoord = inTexCoord * g_fBaseTextureRepeat;

    // Transform the normal, tangent and binormal vectors from object space 
    // to homogeneous projection space:
    float3 vNormalWS   = mul( vInNormalOS,   (float3x3) g_mWorld );
    float3 vTangentWS  = mul( vInTangentOS,  (float3x3) g_mWorld );
    float3 vBinormalWS = mul( vInBinormalOS, (float3x3) g_mWorld );
    
    // Propagate the world space vertex normal through:   
    Out.vNormalWS = vNormalWS;
    
    vNormalWS   = normalize( vNormalWS );
    vTangentWS  = normalize( vTangentWS );
    vBinormalWS = normalize( vBinormalWS );
    
    // Compute position in world space:
    float4 vPositionWS = mul( inPositionOS, g_mWorld );
                 
    // Compute and output the world view vector (unnormalized):
    float3 vViewWS = g_vEye - vPositionWS;
    Out.vViewWS = vViewWS;

    // Compute denormalized light vector in world space:
    float3 vLightWS = g_LightDir;
       
    // Normalize the light and view vectors and transform it to the tangent space:
    float3x3 mWorldToTangent = float3x3( vTangentWS, vBinormalWS, vNormalWS );
       
    // Propagate the view and the light vectors (in tangent space):
    Out.vLightTS = mul( vLightWS, mWorldToTangent );
    Out.vViewTS  = mul( mWorldToTangent, vViewWS  );
       
    // Compute the ray direction for intersecting the height field profile with 
    // current view ray. See the above paper for derivation of this computation.
         
    // Compute initial parallax displacement direction:
    float2 vParallaxDirection = normalize(  Out.vViewTS.xy );
       
    // The length of this vector determines the furthest amount of displacement:
    float fLength         = length( Out.vViewTS );
    float fParallaxLength = sqrt( fLength * fLength - 
                                Out.vViewTS.z * Out.vViewTS.z ) / Out.vViewTS.z; 
       
    // Compute the actual reverse parallax displacement vector:
    Out.vParallaxOffsetTS = vParallaxDirection * fParallaxLength;
       
    // Need to scale the amount of displacement to account for different height ranges
    // in height maps. This is controlled by an artist-editable parameter:
    Out.vParallaxOffsetTS *= g_fHeightMapScale;

   return Out;
}   


//--------------------------------------------------------------------------------------
// Pixel shader output structure
//--------------------------------------------------------------------------------------
struct PS_OUTPUT
{
    float4 RGBColor : COLOR0;  // Pixel color    
};

struct PS_INPUT
{
   float2 texCoord          : TEXCOORD0;
   float3 vLightTS          : TEXCOORD1;   // light vector in tangent space, denormalized
   float3 vViewTS           : TEXCOORD2;   // view vector in tangent space, denormalized
   float2 vParallaxOffsetTS : TEXCOORD3;   // Parallax offset vector in tangent space
   float3 vNormalWS         : TEXCOORD4;   // Normal vector in world space
   float3 vViewWS           : TEXCOORD5;   // View vector in world space
};


//--------------------------------------------------------------------------------------
// Function:    ComputeIllumination
// 
// Description: Computes phong illumination for the given pixel using its attribute 
//              textures and a light vector.
//--------------------------------------------------------------------------------------
float4 ComputeIllumination( float2 texCoord, float3 vLightTS, 
                            float3 vViewTS, float fOcclusionShadow )
{
   // Sample the normal from the normal map for the given texture sample:
   float3 vNormalTS = normalize( tex2D( tNormalHeightMap, texCoord ) * 2 - 1 );
   
   // Sample base map:
   float4 cBaseColor = tex2D( tBase, texCoord );
   
   // Compute diffuse color component:
   float3 vLightTSAdj = float3( vLightTS.x, -vLightTS.y, vLightTS.z );
   
   float4 cDiffuse = saturate( dot( vNormalTS, vLightTSAdj )) * g_materialDiffuseColor;
   
   // Compute the specular component if desired:  
   float4 cSpecular = 0;
   if ( g_bAddSpecular )
   {
      float3 vReflectionTS = normalize( 2 * dot( vViewTS, vNormalTS ) * vNormalTS - vViewTS );
           
      float fRdotL = saturate( dot( vReflectionTS, vLightTSAdj ));
      cSpecular = saturate( pow( fRdotL, g_fSpecularExponent )) * g_materialSpecularColor;
   }
   
   // Composite the final color:
   float4 cFinalColor = (( g_materialAmbientColor + cDiffuse ) * 
                                cBaseColor + cSpecular ) * fOcclusionShadow; 
   
   return cFinalColor;  
}   
 

//--------------------------------------------------------------------------------------
// Parallax occlusion mapping pixel shader
//
// Note: this shader contains several educational modes that would not be in the final
//       game or other complicated scene rendering. The blocks of code in various "if"
//       statements for turning off visual qualities (such as visual level of detail
//       or specular or shadows, etc), can be handled differently, and more optimally.
//       It is implemented here purely for educational purposes.
//--------------------------------------------------------------------------------------
float4 RenderScenePS( PS_INPUT i ) : COLOR0
{ 

   //  Normalize the interpolated vectors:
   float3 vViewTS   = normalize( i.vViewTS  );
   float3 vViewWS   = normalize( i.vViewWS  );
   float3 vLightTS  = normalize( i.vLightTS );
   float3 vNormalWS = normalize( i.vNormalWS );
     
   float4 cResultColor = float4( 0, 0, 0, 1 );

   // Adaptive in-shader level-of-detail system implementation. Compute the 
   // current mip level explicitly in the pixel shader and use this information 
   // to transition between different levels of detail from the full effect to 
   // simple bump mapping. See the above paper for more discussion of the approach
   // and its benefits.
   
   // Compute the current gradients:
   float2 fTexCoordsPerSize = i.texCoord * g_vTextureDims;

   // Compute all 4 derivatives in x and y in a single instruction to optimize:
   float2 dxSize, dySize;
   float2 dx, dy;

   float4( dxSize, dx ) = ddx( float4( fTexCoordsPerSize, i.texCoord ) );
   float4( dySize, dy ) = ddy( float4( fTexCoordsPerSize, i.texCoord ) );
                  
   float  fMipLevel;      
   float  fMipLevelInt;    // mip level integer portion
   float  fMipLevelFrac;   // mip level fractional amount for blending in between levels

   float  fMinTexCoordDelta;
   float2 dTexCoords;

   // Find min of change in u and v across quad: compute du and dv magnitude across quad
   dTexCoords = dxSize * dxSize + dySize * dySize;

   // Standard mipmapping uses max here
   fMinTexCoordDelta = max( dTexCoords.x, dTexCoords.y );

   // Compute the current mip level  (* 0.5 is effectively computing a square root before )
   fMipLevel = max( 0.5 * log2( fMinTexCoordDelta ), 0 );
    
   // Start the current sample located at the input texture coordinate, which would correspond
   // to computing a bump mapping result:
   float2 texSample = i.texCoord;
   
   // Multiplier for visualizing the level of detail (see notes for 'nLODThreshold' variable
   // for how that is done visually)
   float4 cLODColoring = float4( 1, 1, 3, 1 );

   float fOcclusionShadow = 1.0;

   if ( fMipLevel <= (float) g_nLODThreshold )
   {
      //===============================================//
      // Parallax occlusion mapping offset computation //
      //===============================================//

      // Utilize dynamic flow control to change the number of samples per ray 
      // depending on the viewing angle for the surface. Oblique angles require 
      // smaller step sizes to achieve more accurate precision for computing displacement.
      // We express the sampling rate as a linear function of the angle between 
      // the geometric normal and the view direction ray:
      int nNumSteps = (int) lerp( g_nMaxSamples, g_nMinSamples, dot( vViewWS, vNormalWS ) );

      // Intersect the view ray with the height field profile along the direction of
      // the parallax offset ray (computed in the vertex shader. Note that the code is
      // designed specifically to take advantage of the dynamic flow control constructs
      // in HLSL and is very sensitive to specific syntax. When converting to other examples,
      // if still want to use dynamic flow control in the resulting assembly shader,
      // care must be applied.
      // 
      // In the below steps we approximate the height field profile as piecewise linear
      // curve. We find the pair of endpoints between which the intersection between the 
      // height field profile and the view ray is found and then compute line segment
      // intersection for the view ray and the line segment formed by the two endpoints.
      // This intersection is the displacement offset from the original texture coordinate.
      // See the above paper for more details about the process and derivation.
      //

      float fCurrHeight = 0.0;
      float fStepSize   = 1.0 / (float) nNumSteps;
      float fPrevHeight = 1.0;
      float fNextHeight = 0.0;

      int    nStepIndex = 0;
      bool   bCondition = true;

      float2 vTexOffsetPerStep = fStepSize * i.vParallaxOffsetTS;
      float2 vTexCurrentOffset = i.texCoord;
      float  fCurrentBound     = 1.0;
      float  fParallaxAmount   = 0.0;

      float2 pt1 = 0;
      float2 pt2 = 0;
       
      float2 texOffset2 = 0;

      while ( nStepIndex < nNumSteps ) 
      {
         vTexCurrentOffset -= vTexOffsetPerStep;

         // Sample height map which in this case is stored in the alpha channel of the normal map:
         fCurrHeight = tex2Dgrad( tNormalHeightMap, vTexCurrentOffset, dx, dy ).a;

         fCurrentBound -= fStepSize;

         if ( fCurrHeight > fCurrentBound ) 
         {   
            pt1 = float2( fCurrentBound, fCurrHeight );
            pt2 = float2( fCurrentBound + fStepSize, fPrevHeight );

            texOffset2 = vTexCurrentOffset - vTexOffsetPerStep;

            nStepIndex = nNumSteps + 1;
            fPrevHeight = fCurrHeight;
         }
         else
         {
            nStepIndex++;
            fPrevHeight = fCurrHeight;
         }
      }   

      float fDelta2 = pt2.x - pt2.y;
      float fDelta1 = pt1.x - pt1.y;
      
      float fDenominator = fDelta2 - fDelta1;
      
      // SM 3.0 requires a check for divide by zero, since that operation will generate
      // an 'Inf' number instead of 0, as previous models (conveniently) did:
      if ( fDenominator == 0.0f )
      {
         fParallaxAmount = 0.0f;
      }
      else
      {
         fParallaxAmount = (pt1.x * fDelta2 - pt2.x * fDelta1 ) / fDenominator;
      }
      
      float2 vParallaxOffset = i.vParallaxOffsetTS * (1 - fParallaxAmount );

      // The computed texture offset for the displaced point on the pseudo-extruded surface:
      float2 texSampleBase = i.texCoord - vParallaxOffset;
      texSample = texSampleBase;

      // Lerp to bump mapping only if we are in between, transition section:
        
      cLODColoring = float4( 1, 1, 1, 1 ); 

      if ( fMipLevel > (float)(g_nLODThreshold - 1) )
      {
         // Lerp based on the fractional part:
         fMipLevelFrac = modf( fMipLevel, fMipLevelInt );

         if ( g_bVisualizeLOD )
         {
            // For visualizing: lerping from regular POM-resulted color 
            // through blue color for transition layer:
            cLODColoring = float4( 1, 1, max( 1, 2 * fMipLevelFrac ), 1 ); 
         }

         // Lerp the texture coordinate from parallax occlusion mapped 
         // coordinate to bump mapping
         // smoothly based on the current mip level:
         texSample = lerp( texSampleBase, i.texCoord, fMipLevelFrac );

     }  
      
     if ( g_bDisplayShadows == true )
     {
        float2 vLightRayTS = vLightTS.xy * g_fHeightMapScale;
      
        // Compute the soft blurry shadows taking into account self-occlusion for 
        // features of the height field:
   
        float sh0 =  tex2Dgrad( tNormalHeightMap, texSampleBase, dx, dy ).a;
        float shA = (tex2Dgrad( tNormalHeightMap, texSampleBase + 
            vLightRayTS * 0.88, dx, dy ).a - sh0 - 0.88 ) *  1 * g_fShadowSoftening;
        float sh9 = (tex2Dgrad( tNormalHeightMap, texSampleBase + 
            vLightRayTS * 0.77, dx, dy ).a - sh0 - 0.77 ) *  2 * g_fShadowSoftening;
        float sh8 = (tex2Dgrad( tNormalHeightMap, texSampleBase + 
            vLightRayTS * 0.66, dx, dy ).a - sh0 - 0.66 ) *  4 * g_fShadowSoftening;
        float sh7 = (tex2Dgrad( tNormalHeightMap, texSampleBase + 
            vLightRayTS * 0.55, dx, dy ).a - sh0 - 0.55 ) *  6 * g_fShadowSoftening;
        float sh6 = (tex2Dgrad( tNormalHeightMap, texSampleBase + 
            vLightRayTS * 0.44, dx, dy ).a - sh0 - 0.44 ) *  8 * g_fShadowSoftening;
        float sh5 = (tex2Dgrad( tNormalHeightMap, texSampleBase + 
            vLightRayTS * 0.33, dx, dy ).a - sh0 - 0.33 ) * 10 * g_fShadowSoftening;
        float sh4 = (tex2Dgrad( tNormalHeightMap, texSampleBase + 
            vLightRayTS * 0.22, dx, dy ).a - sh0 - 0.22 ) * 12 * g_fShadowSoftening;
   
        // Compute the actual shadow strength:
        fOcclusionShadow = 1 - max( max( max( max( max( max( shA, 
                                    sh9 ), sh8 ), sh7 ), sh6 ), sh5 ), sh4 );
      
        // The previous computation overbrightens the image, let's adjust for that:
        fOcclusionShadow = fOcclusionShadow * 0.6 + 0.4;         
     }       
   }   

   // Compute resulting color for the pixel:
   cResultColor = ComputeIllumination( texSample, vLightTS, vViewTS, fOcclusionShadow );
              
   if ( g_bVisualizeLOD )
   {
      cResultColor *= cLODColoring;
   }
   
   // Visualize currently computed mip level, tinting the color blue if we are in 
   // the region outside of the threshold level:
   if ( g_bVisualizeMipLevel )
   {
      cResultColor = fMipLevel.xxxx;      
   }   

   // If using HDR rendering, make sure to tonemap the resuld color prior to outputting it.
   // But since this example isn't doing that, we just output the computed result color here:
   return cResultColor;
}   


//--------------------------------------------------------------------------------------
// Bump mapping shader
//--------------------------------------------------------------------------------------
float4 RenderSceneBumpMapPS( PS_INPUT i ) : COLOR0
{ 
   //  Normalize the interpolated vectors:
   float3 vViewTS   = normalize( i.vViewTS  );
   float3 vLightTS  = normalize( i.vLightTS );
     
   float4 cResultColor = float4( 0, 0, 0, 1 );

   // Start the current sample located at the input texture coordinate, which would correspond
   // to computing a bump mapping result:
   float2 texSample = i.texCoord;

   // Compute resulting color for the pixel:
   cResultColor = ComputeIllumination( texSample, vLightTS, vViewTS, 1.0f );
              
   // If using HDR rendering, make sure to tonemap the resuld color prior to outputting it.
   // But since this example isn't doing that, we just output the computed result color here:
   return cResultColor;
}   


//--------------------------------------------------------------------------------------
// Apply parallax mapping with offset limiting technique to the current pixel
//--------------------------------------------------------------------------------------
float4 RenderSceneParallaxMappingPS( PS_INPUT i ) : COLOR0
{ 
   const float sfHeightBias = 0.01;
   
   //  Normalize the interpolated vectors:
   float3 vViewTS   = normalize( i.vViewTS  );
   float3 vLightTS  = normalize( i.vLightTS );
   
   // Sample the height map at the current texture coordinate:
   float fCurrentHeight = tex2D( tNormalHeightMap, i.texCoord ).a;
   
   // Scale and bias this height map value:
   float fHeight = fCurrentHeight * g_fHeightMapScale + sfHeightBias;
   
   // Perform offset limiting if desired:
   fHeight /= vViewTS.z;
   
   // Compute the offset vector for approximating parallax:
   float2 texSample = i.texCoord + vViewTS.xy * fHeight;
   
   float4 cResultColor = float4( 0, 0, 0, 1 );

   // Compute resulting color for the pixel:
   cResultColor = ComputeIllumination( texSample, vLightTS, vViewTS, 1.0f );
              
   // If using HDR rendering, make sure to tonemap the resuld color prior to outputting it.
   // But since this example isn't doing that, we just output the computed result color here:
   return cResultColor;
}   


//--------------------------------------------------------------------------------------
// Renders scene to render target
//--------------------------------------------------------------------------------------
technique RenderSceneWithPOM
{
    pass P0
    {          
        
        VertexShader = compile VS_SHADERMODEL RenderSceneVS();
        PixelShader = compile PS_SHADERMODEL RenderScenePS(); 
    }
}

technique RenderSceneWithBumpMap
{
    pass P0
    {          
        VertexShader = compile VS_SHADERMODEL RenderSceneVS();
        PixelShader = compile PS_SHADERMODEL RenderSceneBumpMapPS(); 
    }
}
technique RenderSceneWithPM
{
    pass P0
    {          
        VertexShader = compile VS_SHADERMODEL RenderSceneVS();
        PixelShader = compile PS_SHADERMODEL RenderSceneParallaxMappingPS(); 
    }
}
```

