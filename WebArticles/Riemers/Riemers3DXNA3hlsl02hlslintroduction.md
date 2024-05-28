High Level Shader Language (HLSL) Introduction
Welcome to this introduction on XNA and HLSL.

- HSL – What ?!?
- HLSL, the High Level Shader Language
- But I don’t care about this HLSL, just show me some more XNA code I can copy-paste into my own application!

Well of course I could go on showing you more and more XNA commands, defining some more renderstates, etc. Looking a bit further, it’s clear that all of these commands are at some point translated into commands for the hardware, the graphical card in your pc.

Before 2002, game programmers could only use the Fixed Function Pipeline, meaning the commands provided by DirectX. Since DirectX 8, a lot of flexibility has been added to the way programmers can control their graphics cards. Since then, it’s possible to directly program the vertex and pixel shaders in the GPU, the Graphical Processing Unit. This way, programmers are able to program every graphical effect they could think of, thus bypassing the limited set of DirectX/XNA instructions.

- So what you’re saying is that I can throw away everything I’ve learnt about XNA programming and start learning HLSL??

By all means, no. We’re still going to need a full 100% of what we’ve seen up till now. The difference is that this time we’re going to write our own effects. In a few chapters, you’ll see what is happening in there, how vertices are transformed, etc

- Why would I care about all these low-level commands? The nice thing about XNA is that it takes care of all the maths for us!

The more you can do manually, the more power you have about what is actually drawn on the screen. Hey, this is the 3rd series, it’s time we move on to something more advanced! It is still a ‘high level’ language, so you won’t be seeing any low-level commands, like assembler.

- So, in a nutshell, why would I want to start using this HLSL?

HLSL is used not to improve the gameplay, but to enhance the quality of the final image. Every vertex that is drawn will pass through your vertex shader, and even every pixel drawn will have passed through your pixel shader. The shaders can perform pretty much any manipulation you can think of on their data! HLSL is the only missing link between XNA code and what you see on the screen, so no doubt you’ll benefit from this knowledge. It is also incredibly useful when debugging, and absolutely necessary to add some cool visual effects to your game.

To demonstrate the use of HLSL and shaders, I have written this 3rd Series of XNA tutorials. Have a look at the lighting on one of the screenshots. You can see all lights cast shadows. This is a nice example of something that would be quite impossible to achieve without shaders. As with the previous Series, we’ll start by showing the basics, and gradually build up our application. In the end, you’ll have a complete overview of the meaning of shaders, and have a good understanding of what you can do with them! Pretty much what a tutorial should do, I guess..

So much for this introduction to HLSL. You might still be wondering where HLSL fits into the big picture. The image below demonstrates this, and will be explained while writing our first vertex and pixel shader in the next 2 chapters.

![HLSL Overview](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA3-02Hlsl1.jpg?raw=true)

## Next Steps

[Defining our own vertex format](Riemers3DXNA3hlsl03vertexformat)
