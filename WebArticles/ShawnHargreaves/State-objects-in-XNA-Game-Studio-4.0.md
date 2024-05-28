# State objects in XNA Game Studio 4.0

*Originally posted to Shawn Hargreaves Blog on MSDN, Friday, April 2, 2010*

The [most-often-linked-to article](https://shawnhargreaves.com/blog/spritebatch-and-renderstates.html) I ever wrote is about renderstates, so it should come as no surprise that we tried to improve this area in Game Studio 4.0.

There are fundamentally only two sane ways to manage renderstates:

1. Assume nothing: explicitly set everything you depend on before drawing
2. Assume fixed defaults: if you change anything, you must put it back when you are done

Previous versions of Game Studio supported both approaches, but neither worked particularly well:

1. Our API exposed over 70 different states, so explicitly setting them all was awkward and slow
2. We provided StateBlock and SaveStateMode to help with the "put it back when you are done" approach, but these were extremely slow

Luckily for us, our colleagues over in the native DirectX team grappled with this very issue a few years earlier, and came up with a great solution. For Game Studio 4.0, we basically just borrowed the same state objects design that is used in DirectX 10 and 11.

## Change Summary

* Replaced RenderState class with three new state objects: BlendState, DepthStencilState, and RasterizerState
* Replaced the old SamplerState class with a new SamplerState state object (same name, different behavior)
* Removed StateBlock and SaveStateMode

This new API makes it easy and efficient to explicitly set all state before every drawing operation. We don't provide built-in support for the "save and restore state" pattern, but state objects make it easy to implement that yourself if you want it. Generally, though, we believe that explicitly setting all state is a better and more robust approach than saving and restoring.

## Using State Objects

First, we create a state object:

```csharp
    BlendState blendSubtract = new BlendState();
```

Then we set its properties:

```csharp
    blendSubtract.ColorSourceBlend = Blend.SourceAlpha;
    blendSubtract.ColorDestinationBlend = Blend.One;
    blendSubtract.ColorBlendFunction = BlendFunction.ReverseSubtract;

    blendSubtract.AlphaSourceBlend = Blend.SourceAlpha;
    blendSubtract.AlphaDestinationBlend = Blend.One;
    blendSubtract.AlphaBlendFunction = BlendFunction.ReverseSubtract;
```

Finally, we tell the graphics device to use this custom state:

```csharp
    GraphicsDevice.BlendState = blendSubtract;
```

The first time we bind a state object to the device, it becomes immutable, so we cannot change its properties after we have rendered with it. If we later want a different combination of properties, we must create a different state object instance.

## Using State Objects Wisely

The advantage of state objects is that a single object can atomically specify a whole family of related state settings. This is handy for the developer, and also for the graphics runtime because it gives us the chance to process these state values just once (the first time the state object is bound to the device), rather than having to repeat this work on every draw call.

To get great performance from state objects, you should create all the objects you are going to need ahead of time, so you drawing code is just setting existing state objects onto the graphics device. C# object initializer syntax comes in handy for this:

```csharp
    static class MyStateObjects
    {
        public static BlendState BlendSubtract = new BlendState()
        {
            ColorSourceBlend = Blend.SourceAlpha,
            ColorDestinationBlend = Blend.One,
            ColorBlendFunction = BlendFunction.ReverseSubtract,

            AlphaSourceBlend = Blend.SourceAlpha,
            AlphaDestinationBlend = Blend.One,
            AlphaBlendFunction = BlendFunction.ReverseSubtract,
        };
    }
```

Avoid:

* Creating new state objects every frame
* Creating many duplicate state object instances that all contain the same property settings

## Built-in State Objects

Certain combinations of states are so common that we built them right into the framework, saving you the need of creating custom state objects at all. These built-in states are:

* BlendState
  * Opaque
  * AlphaBlend
  * Additive
  * NonPremultiplied
* DepthStencilState
  * None
  * Default
  * DepthRead
* RasterizerState
  * CullNone
  * CullClockwise
  * CullCounterClockwise
* SamplerState
  * PointWrap
  * PointClamp
  * LinearWrap
  * LinearClamp
  * AnisotropicWrap
  * AnisotropicClamp

Using these built-in states, the graphics device can be reset to default values with just four lines of code:

```csharp
    GraphicsDevice.BlendState = BlendState.Opaque;
    GraphicsDevice.DepthStencilState = DepthStencilState.Default;
    GraphicsDevice.RasterizerState = RasterizerState.CullCounterClockwise;
    GraphicsDevice.SamplerStates[0] = SamplerState.LinearWrap;
``` 

## BlendState

The BlendState object controls how colors are blended into the framebuffer: source and destination blend mode, BlendFunction, BlendFactor, ColorWriteChannels, and MultiSampleMask.

You may notice that there is no more AlphaBlendEnable property. This was redundant, as you can achieve the same result just by setting source = 1 and destination = 0.

There is also no more SeparateAlphaBlendEnabled property. Again, this was redundant: if you want the color and alpha channels to use the same blend mode, you can just configure their properties with the same values.

We renamed some of the blend control properties from previous versions:

|Game Studio 3.1|Game Studio 4.0|
|-|-
|BlendFunction|ColorBlendFunction|
|SourceBlend|ColorSourceBlend|
|DestinationBlend|ColorDestinationBlend|
|AlphaBlendOperation|AlphaBlendFunction|
|AlphaSourceBlend|AlphaSourceBlend|
|AlphaDestinationBlend|AlphaDestinationBlend|

This fixes a common confusion, where people would think the Alpha* properties referred to alpha blending as a whole, not realizing these are only for the alpha channel, while there is another set of properties controlling the blend behavior of the red, green, and blue channels.

We removed the Blend.BothSourceAlpha and Blend.BothInverseSourceAlpha enum values, because these were not especially useful and not universally supported by hardware.

We also removed the alpha test renderstates, mostly because alpha test is no longer supported by DirectX 10 and above. As of Game Studio 4.0, alpha testing can be implemented in the pixel shader, or by using the built-in AlphaTestEffect.

## DepthStencilState

The DepthStencilState object controls the behavior of the depth buffer and stencil buffer. Its properties are obvious and directly match the old renderstates, so this is a short paragraph!

## RasterizerState

The RasterizerState object controls how triangles are turned into pixels: CullMode, FillMode, DepthBias, MultiSampleAntiAlias, and ScissorTestEnable.

We removed the FillMode.Point enum value and point size renderstates, because Game Studio 4.0 does not support point sprites.

We also removed the Fog* and Wrap* states, which are not useful (and in some cases not even supported) on modern shader hardware.

## SamplerState

The SamplerState object controls how data is fetched from textures: AddressU, AddressV, AddressW, Filter, MaxAnisotropy, MaxMipLevel, and MipMapLevelOfDetailBias.

We removed the TextureAddressMode.Border and TextureAddressMode.MirrorOnce enum values, because these are not consistently supported by all our target hardware.

The old MinFilter, MagFilter, and MipFilter properties are collapsed into a single Filter property, which specifies all three options using a single TextureFilter enum value. The old API allowed too many permutations of filter values, many of which were not actually legal. For instance you cannot set MinFilter = None and MipFilter = Anisotropic, but that didn't stop many beginners from trying! With the new API, the TextureFilter enum only contains legal filter settings, so it is no longer possible to get this wrong.

## High Frequency States

There are three special state values, which are duplicated as properties directly on the GraphicsDevice:

* BlendFactor
* MultiSampleMask
* ReferenceStencil

These states are unique because it is often useful to animate them over a range of continuously varying values, for instance to fade out an object by changing BlendFactor. It would be ridiculous if we had to create hundreds of otherwise identical BlendState objects in order to animate this one value! Instead, we can write:

```csharp
    GraphicsDevice.BlendState = BlendState.AlphaBlend;
    GraphicsDevice.BlendFactor = CurrentFadeAmount;
```

Assigning to GraphicsDevice.BlendState overrides the current GraphicsDevice.BlendFactor setting with the BlendFactor value from the specified state object, so order is important. Set the state object first, and the high-frequency property second.