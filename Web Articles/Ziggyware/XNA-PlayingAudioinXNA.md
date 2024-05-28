# XNA - Playing Audio in XNA

> *Note, this article references XNA V1 - MonoGame update to follow

Using XNA to create audio effects is a breeze.

XNA uses XACT to create audio within XNA. This is a powerful tool that requires very little time to get up to speed:


First thing is you should read the online help from within Visual Studio:


Once you've read up on how to create a wav bank, sound bank and cue you should be half way there :)


Here is a little tutorial on how to put the knowledge gained to use:




```csharp
using System;
using System.Collections.Generic;
using System.IO;
using Microsoft.Xna.Framework;
using Microsoft.Xna.Framework.Audio;
using Microsoft.Xna.Framework.Components;
using Microsoft.Xna.Framework.Graphics;
using Microsoft.Xna.Framework.Input;
using Microsoft.Xna.Framework.Storage;
```

First we create a generic project:


```csharp
namespace SimpleXNA
{
    partial class Game1 : Microsoft.Xna.Framework.Game
    {
```

Now we must declare an AudioEngine, WaveBank and SoundBank:



```csharp
        AudioEngine audioEngine;
        WaveBank waveBank;
        SoundBank soundBank;
```

Create a Play function that will start playing a particular audio file giving its Cue name:




```csharp
        public Cue Play(string Name)
        {
            Cue returnValue = soundBank.GetCue(Name);
            returnValue.Play();
            return returnValue;
        }
```

Being able to stop the playing of a file is also useful:




```csharp
        public static void Stop(Cue cue)
        {
            cue.Stop(AudioStopOptions.Immediate);
        }
```

Don't forget to clean up the objects you create!


        
```csharp
        private void WindowsGame_Exiting(object sender, GameEventArgs e)
        {
            soundBank.Dispose();
            waveBank.Dispose();
            audioEngine.Dispose();
        }
```

In the Starting event we can create our audio objects giving the files that were output from XACT



```csharp
        private void WindowsGame_Starting(object sender, GameEventArgs e)
        {
            graphics.ApplyChanges();
```

Here i am loading the audio data in Windows format. For XBOX you would want to use the XBOX directories instead



```csharp
            FileStream audioParameters = new FileStream(@"Win\Audio.xgs", FileMode.Open, FileAccess.Read);

            audioEngine = new AudioEngine(audioParameters);
            waveBank = new WaveBank(audioEngine, @"Win\Wave Bank.xwb");
            soundBank = new SoundBank(audioEngine, @"Win\Sound Bank.xsb");

            audioParameters.Close();
        }
```

No need to modify the default constructor


```csharp
        public Game1()
        {
            InitializeComponent();
        }
```

I have this set up to play an explosion sound every five seconds:




```csharp
        float f = 0.0f;

        protected override void Update()
        {
            // The time since Update was called last
            float elapsed = (float)ElapsedTime.TotalSeconds;
            
            // TODO: Add your game logic here

            
            f += elapsed;

            if(f >= 5.0f)
            {
                f=0;
                Play("explode4");
            }
            
            

            // Let the GameComponents update
            UpdateComponents();
        }
```

We are not drawing anything in this sample so I have just left the default Draw() method




```csharp
        protected override void Draw()
        {
            // Make sure we have a valid device
            if (!graphics.EnsureDevice())
                return;

            
            graphics.GraphicsDevice.Clear(Color.CornflowerBlue);
            graphics.GraphicsDevice.BeginScene();

            // TODO: Add your drawing code herej

            // Let the GameComponents draw
            DrawComponents();

            graphics.GraphicsDevice.EndScene();
            graphics.GraphicsDevice.Present();
        }

    }

}
```

Thats all you need for basic audio using XNA! What a breeze!