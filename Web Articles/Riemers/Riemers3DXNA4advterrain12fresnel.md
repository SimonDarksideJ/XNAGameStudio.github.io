# Blending in refractions using the Fresnel term

Now we have added some ripples to our water, it’s time we use our refraction map to blend in the color of the bottoms of the river. We will use the same rippling effect of last chapter and we already know the reflective and refractive color, so the only remaining question is: for each pixel, how much of each color do we have to use?

The answer is in the image below. The horizontal flat line represents our water. The vector pointing upward is the normal vector of the pixel. The other vector, called the eyevector, is the vector going from the camera to the pixel. The amount of green on the normal vector indicates the amount of reflection the current pixel should have, and the amount of red represents the amount of refraction.

![Wavelengths](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA4-12Fresnel1.jpg?raw=true)

So how can we find the lengths of the green and red bars? We need to project the eyevector on the normal vector. For those of you that have always wondered what a dot product is, well, this is exactly what is does: when you dot product the eyevector and the normal vector, you get the length of the red bar. The green bar then equals (1- length of red bar).

That’s enough theory for now, let’s go on to the HLSL code. As you can see, we’ll need the position of our camera in the pixel shader, so we need to add this variable:

```csharp
float3 xCamPos;
```

Before we really start with the Fresnel stuff in our shader, let’s first make sure we calculate the refractive color for our water. This is done exactly the same is for the reflective water: first we need the projective textures, add some ripple perturbation to them, and sample the refraction map. To know the projective textures, we again need the 2D screen coords for that pixel, as seen by the camera that created the refraction map. This camera was our normal camera, so add this to the vertex shader and its output struct:

```csharp
struct WVertexToPixel
{
    float4 Position                 : POSITION;
    float4 ReflectionMapSamplingPos    : TEXCOORD1;
    float2 BumpMapSamplingPos        : TEXCOORD2;
    float4 RefractionMapSamplingPos : TEXCOORD3;
    float4 Position3D                : TEXCOORD4;
};

Output.RefractionMapSamplingPos = mul(inPos, preWorldViewProjection);
Output.Position3D = mul(inPos, xWorld);
```

For each pixel, we’ll also need its 3D position to calculate the eyevector, so add the last line to our vertex shader. The vertices of the water have already been defined in absolute World space, but to stay general we multiply their positions with the World matrix.

Next, in our pixel shader, we first need to save the reflective color we obtained last chapter into a variable:

```csharp
float4 reflectiveColor = tex2D(ReflectionSampler, perturbatedTexCoords);
```

Now we do exactly the same as we did last chapter, but this time for the refraction map:

```csharp
float2 ProjectedRefrTexCoords;
ProjectedRefrTexCoords.x = PSIn.RefractionMapSamplingPos.x/PSIn.RefractionMapSamplingPos.w/2.0f + 0.5f;
ProjectedRefrTexCoords.y = -PSIn.RefractionMapSamplingPos.y/PSIn.RefractionMapSamplingPos.w/2.0f + 0.5f;
float2 perturbatedRefrTexCoords = ProjectedRefrTexCoords + perturbation;
float4 refractiveColor = tex2D(RefractionSampler, perturbatedRefrTexCoords);
```

Maybe you can have a look at the rippled refraction map first. To do this, route the refractiveColor to Output.Color and run your code.

Now we know both the reflective and the refractive color, it’s time to blend them together according to the Fresnel term. As explained a above, to obtain the Fresnel term we first need to find the eyevector:

```csharp
float3 eyeVector = normalize(xCamPos - PSIn.Position3D);
```

Usually the normal is passed by the XNA app to the vertex shader, which passes it on to the pixel shader. In this simple case however, we’ll say the normal in each pixel of our water points upwards:

```csharp
float3 normalVector = float3(0,1,0);
```

Now we know both vectors, we can find the Fresnel term as explained above:

```csharp
float fresnelTerm = dot(eyeVector, normalVector);
```

And finally we can blend our both colors:

```csharp
Output.Color = lerp(reflectiveColor, refractiveColor, fresnelTerm);
```

Which interpolates between the refractiveColor and reflectiveColor.

That’s it for the HLSL code, in our XNA we only need to pass in the camera position in the DrawWater method:

```csharp
 effect.Parameters["xCamPos"].SetValue(cameraPosition);
```

Now when you run this code you should get some nice water that has both a reflective and refractive color!

Let’s immediately finish the look of our water, by blending in a dull water color. Save the color we got thus far as combinedColor:

```csharp
float4 combinedColor = lerp(reflectiveColor, refractiveColor, fresnelTerm);
```

And define a dull water color, this one is blueish gray:

```csharp
float4 dullColor = float4(0.3f, 0.3f, 0.5f, 1.0f);
```

And blend it in!

```csharp
Output.Color = lerp(combinedColor, dullColor, 0.2f);
```

This line takes 20% of the dull color, 80% of the combinedColor, add them together and routes them to the output!

This should give you the following image:

![Summary](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA4-12Fresnel2.jpg?raw=true)

And there you have it! When you run this code, you should see some really nice looking water. You have reflections, refractions, and to make it a bit more realistic a dull color has been blended in.

There are a few more advanced things we could add. For example, we’ve said the normal is pointing upwards for all waves, which is not the case with ripples. And what about lighting? Lighting is affected by the direction of the normal. This will all be added at the end of the series, but let’s increase the realism of our water with a huge step: by making the ripples move through the water!

> You can try these exercises to practice what you've learned:
>
> - Try out some more dull water colors, and blending factors.
> - I’m not going to list the entire XNA code, as all we added was the xCamPos XNA-to-HLSL variable.

## Here is our HLSL code

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
float4x4 xReflectionView;
float4x4 xProjection;
float4x4 xWorld;
float3 xLightDirection;
float xAmbient;
bool xEnableLighting;
float xWaveLength;
float xWaveHeight;

 float3 xCamPos;


//------- Texture Samplers --------
Texture xTexture;

sampler TextureSampler = sampler_state { texture = <xTexture> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = mirror; AddressV = mirror;};Texture xTexture0;

sampler TextureSampler0 = sampler_state { texture = <xTexture0> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = wrap; AddressV = wrap;};Texture xTexture1;

sampler TextureSampler1 = sampler_state { texture = <xTexture1> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = wrap; AddressV = wrap;};Texture xTexture2;

sampler TextureSampler2 = sampler_state { texture = <xTexture2> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = mirror; AddressV = mirror;};Texture xTexture3;

sampler TextureSampler3 = sampler_state { texture = <xTexture3> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = mirror; AddressV = mirror;};Texture xReflectionMap;

sampler ReflectionSampler = sampler_state { texture = <xReflectionMap> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = mirror; AddressV = mirror;};Texture xRefractionMap;

sampler RefractionSampler = sampler_state { texture = <xRefractionMap> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = mirror; AddressV = mirror;};Texture xWaterBumpMap;

sampler WaterBumpMapSampler = sampler_state { texture = <xWaterBumpMap> ; magfilter = LINEAR; minfilter = LINEAR; mipfilter=LINEAR; AddressU = mirror; AddressV = mirror;};
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

//------- Technique: Water --------
struct WVertexToPixel
{
    float4 Position                 : POSITION;
    float4 ReflectionMapSamplingPos    : TEXCOORD1;
    float2 BumpMapSamplingPos        : TEXCOORD2;

     float4 RefractionMapSamplingPos : TEXCOORD3;
     float4 Position3D                : TEXCOORD4;

};

struct WPixelToFrame
{
    float4 Color : COLOR0;
};

WVertexToPixel WaterVS(float4 inPos : POSITION, float2 inTex: TEXCOORD)
{    
    WVertexToPixel Output = (WVertexToPixel)0;

    float4x4 preViewProjection = mul (xView, xProjection);
    float4x4 preWorldViewProjection = mul (xWorld, preViewProjection);
    float4x4 preReflectionViewProjection = mul (xReflectionView, xProjection);
    float4x4 preWorldReflectionViewProjection = mul (xWorld, preReflectionViewProjection);

    Output.Position = mul(inPos, preWorldViewProjection);
    Output.ReflectionMapSamplingPos = mul(inPos, preWorldReflectionViewProjection);
    Output.BumpMapSamplingPos = inTex/xWaveLength;

     Output.RefractionMapSamplingPos = mul(inPos, preWorldViewProjection);
     Output.Position3D = mul(inPos, xWorld);


    return Output;
}

WPixelToFrame WaterPS(WVertexToPixel PSIn)
{
    WPixelToFrame Output = (WPixelToFrame)0;        
    
    float4 bumpColor = tex2D(WaterBumpMapSampler, PSIn.BumpMapSamplingPos);
    float2 perturbation = xWaveHeight*(bumpColor.rg - 0.5f)*2.0f;
    
    float2 ProjectedTexCoords;
    ProjectedTexCoords.x = PSIn.ReflectionMapSamplingPos.x/PSIn.ReflectionMapSamplingPos.w/2.0f + 0.5f;
    ProjectedTexCoords.y = -PSIn.ReflectionMapSamplingPos.y/PSIn.ReflectionMapSamplingPos.w/2.0f + 0.5f;        
    float2 perturbatedTexCoords = ProjectedTexCoords + perturbation;
    float4 reflectiveColor = tex2D(ReflectionSampler, perturbatedTexCoords);
    

     float2 ProjectedRefrTexCoords;
     ProjectedRefrTexCoords.x = PSIn.RefractionMapSamplingPos.x/PSIn.RefractionMapSamplingPos.w/2.0f + 0.5f;
     ProjectedRefrTexCoords.y = -PSIn.RefractionMapSamplingPos.y/PSIn.RefractionMapSamplingPos.w/2.0f + 0.5f;    
     float2 perturbatedRefrTexCoords = ProjectedRefrTexCoords + perturbation;    
     float4 refractiveColor = tex2D(RefractionSampler, perturbatedRefrTexCoords);
     
     float3 eyeVector = normalize(xCamPos - PSIn.Position3D);
     float3 normalVector = float3(0,1,0);
     float fresnelTerm = dot(eyeVector, normalVector);    
     float4 combinedColor = lerp(reflectiveColor, refractiveColor, fresnelTerm);
     
     float4 dullColor = float4(0.3f, 0.3f, 0.5f, 1.0f);
     
     Output.Color = lerp(combinedColor, dullColor, 0.2f);

    
    return Output;
}

technique Water
{
    pass Pass0
    {
        VertexShader = compile vs_1_1 WaterVS();
        PixelShader = compile ps_2_0 WaterPS();
    }
}
```

## Next Steps

[Making our water move](Riemers3DXNA4advterrain13movingwater)
