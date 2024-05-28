# Adding more detail close to the camera

When you look at the terrain from a distance, it looks pretty nice. But when you move the camera very close to the terrain, you値l see the individual pixels of each texture are spread out over quite a large area. This is bad, because in a game, the camera will most of the time be placed close to the terrain.

So how can we solve this? One easy approach would be to enlarge our texture coords, so much more of our textures would be put on our terrain, like this:

```csharp
 terrainVertices[x + y * terrainWidth].TexCoords.Z = x / (float)10;
 terrainVertices[x + y * terrainWidth].TexCoords.W = y / (float)10;
```

Instead of:

```csharp
 terrainVertices[x + y * terrainWidth].TexCoords.Z = x / (float)30;
 terrainVertices[x + y * terrainWidth].TexCoords.W = y / (float)30;
```

When running this, you will see that this gives good results when the camera is close to the terrain, but when you move the camera further away, the textures will be too small so you see the repeating pattern. So the problem is now the other way around.

Instead of choosing one of these approaches, we値l combine both. This is done in our pixel shader: when the pixel is far away from our camera, we値l use the big textures. When the pixel is very close to the camera, we値l use the smaller, more detailed textures. For the region in between, we値l blend between these two. This whole operation can be done in the pixel shader, so the load on our CPU will remain the same.

As you can see, for each pixel we will need the distance to our camera. So add this line to the struct MTVertexToPixel struct:

```csharp
float    Depth            : TEXCOORD4;
```

This distance is nothing more than the z coordinate of the position in camera space. Remember, because this was the result of a 4x4 matrix multiplication, we first need to divide it by the w component before we can use it:

```csharp
Output.Depth = Output.Position.z/Output.Position.w;
```

Now we can access this in the pixel shader, we know the distance between each pixel and the camera. Let痴 go to our pixel shader, where we値l define 2 values concerning the blending: the distance to the camera at which blending will begin, and the width of the blending border:

```csharp
float blendDistance = 0.99f;
float blendWidth = 0.005f;
float blendFactor = clamp((PSIn.Depth-blendDistance)/blendWidth, 0, 1);
```

The last line is our blending function. All pixels will have a Depth value between 0 and 1, where 0 corresponds to the near clipping plane distance and 1 to the far clipping plane distance (as set in your projectionMatrix). Using this function, all pixels closer to the camera than 0.99 will get blendfactor 0, all pixel further than 0.99+0.005=0.995 will get blendfactor 1 and all pixels in between will get a linearly interpolated blendfactor. I have visualized this in the next image:

![Blending](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA4-05Detail1.jpg?raw=true)

Where completely blue means blendfactor 0, and completely red blendfactor 1. Pixels with blendfactor 0 will use the highly detailed textures, and the red ones will use the standard textures.

First, we will calculate both the high-detail color for each pixel as well as the standard color:

```csharp
float4 farColor;
farColor = tex2D(TextureSampler0, PSIn.TextureCoords)*PSIn.TextureWeights.x;
farColor += tex2D(TextureSampler1, PSIn.TextureCoords)*PSIn.TextureWeights.y;
farColor += tex2D(TextureSampler2, PSIn.TextureCoords)*PSIn.TextureWeights.z;
farColor += tex2D(TextureSampler3, PSIn.TextureCoords)*PSIn.TextureWeights.w;

float4 nearColor;
float2 nearTextureCoords = PSIn.TextureCoords*3;
nearColor = tex2D(TextureSampler0, nearTextureCoords)*PSIn.TextureWeights.x;
nearColor += tex2D(TextureSampler1, nearTextureCoords)*PSIn.TextureWeights.y;
nearColor += tex2D(TextureSampler2, nearTextureCoords)*PSIn.TextureWeights.z;
nearColor += tex2D(TextureSampler3, nearTextureCoords)*PSIn.TextureWeights.w;
```

Where the first block of code is taken straight from the previous chapter. For the near color, we multiply our texture c0ords by 3, so the textures a 3 times smaller, and thus 3 times more detailed.

Now we have both our near and far colors for the pixel, we need to mix them according to the blendfactor:

```csharp
Output.Color = lerp(nearColor, farColor, blendFactor);
Output.Color *= lightingFactor;
```

This is simple linear interpolation, useful when the blendfactor is between 0 and 1. When you multiply this with the lighting factor, you should get the same result as below:

![Summary](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA4-05Detail2.jpg?raw=true)

The chapter is the last one of this series that will handle the texturing of our terrain. Next chapter we could start with the water technique, but this will yield poor results because we don稚 have any clouds yet that can be reflected in the water. So first we値l load a skysphere and decorate it with a cloud texture.

> You can try these exercises to practice what you've learned:
>
> - Try playing around with the blendDistance and blendWidth values
> - Try adding a 3rd level of detail for the textures

## Shader code

```csharp
//----------------------------------------------------
//-- --
//-- www.riemers.net --
//-- Series 4: Advanced terrain --
//-- Shader code --
//-- --
//----------------------------------------------------

//------- Constants --------
float4x4 xView;
float4x4 xProjection;
float4x4 xWorld;
float3 xLightDirection;
float xAmbient;
bool xEnableLighting;

//------- Texture Samplers --------
Texture xTexture;

sampler TextureSampler = sampler_state { texture = <xTexture> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = mirror; AddressV = mirror;};Texture xTexture0;

sampler TextureSampler0 = sampler_state { texture = <xTexture0> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = wrap; AddressV = wrap;};Texture xTexture1;

sampler TextureSampler1 = sampler_state { texture = <xTexture1> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = wrap; AddressV = wrap;};Texture xTexture2;

sampler TextureSampler2 = sampler_state { texture = <xTexture2> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = mirror; AddressV = mirror;};Texture xTexture3;

sampler TextureSampler3 = sampler_state { texture = <xTexture3> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = mirror; AddressV = mirror;};
//------- Technique: Textured --------
struct TVertexToPixel
{
float4 Position     : POSITION;
float4 Color        : COLOR0;
float LightingFactor: TEXCOORD0;
float2 TextureCoords: TEXCOORD1;
};

struct TPixelToFrame
{
float4 Color : COLOR0;
};

TVertexToPixel TexturedVS( float4 inPos : POSITION, float3 inNormal: NORMAL, float2 inTexCoords: TEXCOORD0)
{
    TVertexToPixel Output = (TVertexToPixel)0;
    float4x4 preViewProjection = mul (xView, xProjection);
    float4x4 preWorldViewProjection = mul (xWorld, preViewProjection);

    Output.Position = mul(inPos, preWorldViewProjection);
    Output.TextureCoords = inTexCoords;

    float3 Normal = normalize(mul(normalize(inNormal), xWorld));
    Output.LightingFactor = 1;
    if (xEnableLighting)
        Output.LightingFactor = saturate(dot(Normal, -xLightDirection));

    return Output;
}

TPixelToFrame TexturedPS(TVertexToPixel PSIn)
{
    TPixelToFrame Output = (TPixelToFrame)0;

    Output.Color = tex2D(TextureSampler, PSIn.TextureCoords);
    Output.Color.rgb *= saturate(PSIn.LightingFactor + xAmbient);

    return Output;
}

technique Textured_2_0
{
    pass Pass0
    {
        VertexShader = compile vs_2_0 TexturedVS();
        PixelShader = compile ps_2_0 TexturedPS();
    }
}

technique Textured
{
    pass Pass0
    {
        VertexShader = compile vs_1_1 TexturedVS();
        PixelShader = compile ps_1_1 TexturedPS();
    }
}

//------- Technique: Multitextured --------
struct MTVertexToPixel
{
    float4 Position         : POSITION;
    float4 Color            : COLOR0;
    float3 Normal            : TEXCOORD0;
    float2 TextureCoords    : TEXCOORD1;
    float4 LightDirection    : TEXCOORD2;
    float4 TextureWeights    : TEXCOORD3;

     float Depth                : TEXCOORD4;

};

struct MTPixelToFrame
{
    float4 Color : COLOR0;
};

MTVertexToPixel MultiTexturedVS( float4 inPos : POSITION, float3 inNormal: NORMAL, float2 inTexCoords: TEXCOORD0, float4 inTexWeights: TEXCOORD1)
{    
    MTVertexToPixel Output = (MTVertexToPixel)0;
    float4x4 preViewProjection = mul (xView, xProjection);
    float4x4 preWorldViewProjection = mul (xWorld, preViewProjection);

    Output.Position = mul(inPos, preWorldViewProjection);
    Output.Normal = mul(normalize(inNormal), xWorld);
    Output.TextureCoords = inTexCoords;
    Output.LightDirection.xyz = -xLightDirection;
    Output.LightDirection.w = 1;    
    Output.TextureWeights = inTexWeights;

     Output.Depth = Output.Position.z/Output.Position.w;


    return Output;
}

MTPixelToFrame MultiTexturedPS(MTVertexToPixel PSIn)
{
    MTPixelToFrame Output = (MTPixelToFrame)0;        
    
    float lightingFactor = 1;
    if (xEnableLighting)
        lightingFactor = saturate(saturate(dot(PSIn.Normal, PSIn.LightDirection)) + xAmbient);

         
     float blendDistance = 0.99f;
     float blendWidth = 0.005f;
     float blendFactor = clamp((PSIn.Depth-blendDistance)/blendWidth, 0, 1);
         
     float4 farColor;
     farColor = tex2D(TextureSampler0, PSIn.TextureCoords)*PSIn.TextureWeights.x;
     farColor += tex2D(TextureSampler1, PSIn.TextureCoords)*PSIn.TextureWeights.y;
     farColor += tex2D(TextureSampler2, PSIn.TextureCoords)*PSIn.TextureWeights.z;
     farColor += tex2D(TextureSampler3, PSIn.TextureCoords)*PSIn.TextureWeights.w;
     
     float4 nearColor;
     float2 nearTextureCoords = PSIn.TextureCoords*3;
     nearColor = tex2D(TextureSampler0, nearTextureCoords)*PSIn.TextureWeights.x;
     nearColor += tex2D(TextureSampler1, nearTextureCoords)*PSIn.TextureWeights.y;
     nearColor += tex2D(TextureSampler2, nearTextureCoords)*PSIn.TextureWeights.z;
     nearColor += tex2D(TextureSampler3, nearTextureCoords)*PSIn.TextureWeights.w;
 
     Output.Color = lerp(nearColor, farColor, blendFactor);
     Output.Color *= lightingFactor;

    
    return Output;
}

technique MultiTextured
{
    pass Pass0
    {
        VertexShader = compile vs_1_1 MultiTexturedVS();
        PixelShader = compile ps_2_0 MultiTexturedPS();
    }
}
```

## Next Steps

[Drawing a simple skydome](Riemers3DXNA4advterrain06skydome)
