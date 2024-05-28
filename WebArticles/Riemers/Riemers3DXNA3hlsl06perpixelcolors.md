# Experimenting with shaders

So now our XNA program passes vertices to our vertex shader, which transform their coordinates to 2D screen coordinates and sends these coordinates together with the color to the pixel shader, which takes the color and draws the pixel on the screen.

Let’s have another look at our flowchart, as last chapter we rushed things a bit to finally get something on the screen. You’ll notice the rasterizer between the vertex and the pixel shader. This rasterizer determines which pixels on the screen are occupied by our triangle, and makes sure these pixels are also sent to the pixel shader. Without this rasterizer, only the 3 points corresponding to the vertices would be sent to the pixel shader.

And now something important: what would be the color of these extra pixels at the interior of the triangle, which the pixel shader receives at its input? The interpolator next to the rasterizer calculates this value, by interpolating the color value of the corner points. This means that a pixel exactly in the middle between a blue and a red corner point, will get the color that is exactly in the middle of these colors. In the example of our program, this means for all pixels the triangle occupies, the colors are nicely shaded from corner to corner. Our next code will demonstrate this, and how this can cause problems.

So you can be asking yourself “What’s the use of all this ??”. Even at this point, you can see we are able to perform some extra manipulations. For example, you could change the position or color of each vertex. You could also do this in your XNA app, but then your CPU would have to perform those calculations, which would lower your framerate. Now you can have these calculations done by the GPU, which is A LOT faster at it, leaving your CPU free to perform more important calculations.

Using the vertex shader, you could also adjust the color, which we’ve done before (we made our whole triangle white) and which we’ll do now again as little exercise. So for this example, we will throw away the color information provided to us by the vertex stream, and define our own colors. Say, we want our vertex shader make the red color component indicate the X coordinate of each vertex, the green component the Y coordinate, and the blue color component indicate the z coordinate. To do this, insert this line of code into your vertex shader:

```csharp
Output.Color.rgb = inPos.xyz;
```

Note that for any color the value 0 and below means ‘none’, and 1 and above means ‘full’. You’ll notice I have chosen the coordinates of our vertexes in the XNA app between the [-2,2] range, which I did especially for this example. So, for every color, you could expect that the color is not present in the [-2,0] region, then shaded from ‘none’ to full color in the [0,1] region, to remain at full color in the [1,2] region. For example, the point (0,0,0), which is part of our triangle, should be drawn completely black (0 red component, 0 green component and 0 blue component). (If you fail miserably at imagining this, you can have a peek at the bottom image of this page, where the correct colors are shown.)

Try to run the code with the new line instead of the Output.Color = inColor; line in our vertex shader. The colors will be different from the previous chapter: this time, the corner towards you (pos Z coordinate) will be blue, the upper corner (pos Y coordinate) will be green and the point to the right will be red (pos X coordinate).

However, you’ll see the colors remains nicely shaded over the full size of the triangle, which isn’t what we wanted (for example, the (0,0,0) point in the very middle of our triangle had to be black).

So what is happening? Before the colors are passed to the interpolator, the 3 color values of the 3 vertices are being clipped to the [0,1] region. For example, the (-2,-2,2) vertex should have -2, -2 and 2 as rgb color values, but it gets 0, 0 and 1 as color values. Next, the interpolator simply gives every pixel in the triangle the color that is the interpolation of the 3 clipped color values in the vertices. So, for example, the (0,0,0) point gets a color value that is an interpolation of color values between the [0,1] region, and thus will never be completely 0,0,0 (=black).

In short: when only using a vertex shader, the colors of a triangle can only change linearly, and then again only with outer values in the [0,1] range.

![HLSL Overview](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA3-02Hlsl1.jpg?raw=true)

When you take a look at the flowchart, you’ll notice there are 2 arrows going from the vertex shader to the pixel shader. The left one is necessary, as it is the position of the pixel in 2D screen coordinates. The rasterizer and interpolator, as well as the pixel shader, need it to do their work. One remark: by default you can NOT use this position as input to your pixel shader. The bigger arrow is not necessary, but you’ll always want to use it, as it contains the data the pixel shader uses as input. This can be, for example, the interpolated color, interpolated normal, interpolated texture coordinates, etc. I guess by now you noticed the emphasis I put on the word ‘Interpolated’. If you want, you can also pass a copy of the 2D screen position in this arrow, so your pixel shader can use it as input.

When you look at the flowchart, you’ll see the pixel shader eventually sends its output to the frame buffer. This is where multiple renderings get blended in case of alpha blending.

Now it’s time to show you something specific about pixel shaders: we’re going to program the color of each of the pixels separately. We’re going to do the same exercise again, only this time using a pixel shader. Remember, we want each of the 3 color channels to take the values of each of the three 3D-coordinates of the pixel.

Because we’re going to derive the color from the 3D position, we need the 3D position as input to our pixel shader. The only coordinate currently being passed to the pixel shader is the 2D screen position (remember we cannot use this value in our pixel shader, as it is the left arrow in our flowchart). So what we need to do is pass the 3D coordinate from our vertex shader to our pixel shader. First redefine the output structure of your vertex shader:

```csharp
struct VertexToPixel
{
    float4 Position     : POSITION;
    float4 Color        : COLOR0;
    float3 Position3D    : TEXCOORD0;
};
```

You see we added a member to store the 3D position. As semantic, we used TEXCOORD0. You can use TEXCOORD0 to TEXCOORD16 safely to pass float4 values from your vertex shader to your pixel shader.

Next we’ll update our vertex shader so it routes the 3D position it receives from the vertex stream to its output. To do this, we simply have to add the following line to our vertex shader:

```csharp
Output.Position3D = inPos;
```

This will have our 3D position sent to the interpolator, which will interpolate the correct 3D position of every pixel in the triangle. For each pixel, this interpolated 3D coordinate will be sent to the pixel shader. This position we receive in the pixel shader will be exact in each pixel, for two reasons:

Only colors are clipped to the [0,1] range, TEXCOORD0 semantics can have any value
The interpolator performs linear interpolator, and our triangle is linear, as it’s flat.

Now we will have our pixel shader set the color components to the value of this 3D coordinate. In your pixel shader, put this line as color definition:

```csharp
Output.Color.rgb = PSIn.Position3D.xyz;
```

By now you should know what this does: it sets the value of the X coordinate as the red color component, and so on. Now run the code. When you did everything well, you shouldn’t see any error messages. You should see the same as the image below: a simple triangle, with pixels that display color. So we have programmed the color of every pixel individually! As a quick check, you can see that now the (0,0,0) middle point of our triangle is black.

![Summary](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA3-06Summary1.jpg?raw=true)

Note we haven’t changed anything to our XNA code, only the HLSL part has been changed. This means our CPU still has the same workload. Only the GPU will have to work a bit harder, but this still is nothing compared to the capabilities of the GPUs present on today’s Radeon and GeForce boards.

> You can try these exercises to practice what you've learned:
>
> - In your pixel shader, add 1 to the Red color component of each pixel.

## The HLSL code

```csharp
float4x4 xViewProjection;

struct VertexToPixel
{
    float4 Position     : POSITION;
    float4 Color        : COLOR0;
    float3 Position3D : TEXCOORD0;
};

struct PixelToFrame
{
    float4 Color        : COLOR0;
};

VertexToPixel SimplestVertexShader( float4 inPos : POSITION, float4 inColor : COLOR0)
{
    VertexToPixel Output = (VertexToPixel)0;
    
    Output.Position = mul(inPos, xViewProjection);
    Output.Color = inColor;
    Output.Position3D = inPos;

    return Output;
}

PixelToFrame OurFirstPixelShader(VertexToPixel PSIn)
{
    PixelToFrame Output = (PixelToFrame)0;    

    Output.Color = PSIn.Color;    
    Output.Color.rgb = PSIn.Position3D.xyz;

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

## Next Steps

[Texturing our triangle using the Pixel Shader](Riemers3DXNA3hlsl07texturedtriangle)
