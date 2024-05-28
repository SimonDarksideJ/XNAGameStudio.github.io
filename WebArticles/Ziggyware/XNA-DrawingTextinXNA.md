# Drawing Text in MonoGame

In MonoGame, like XNA 4.0, drawing text is very quick and easy with the built-in SpriteFont system

You simply need to create a new SpriteFont (SpriteFontDescriptor) in the content pipeline, load it and then use the DrawString methods of the SpriteBatch class.

If you open the Content Pipeline editor (by double clicking on the Content.mgcb file in Visual Studio, or launching it manually and opening your content project). 
Once open you simple need to add a new SpriteFont using the following command in the menu:

> File -> New File -> SpriteFont Description (.spritefont)

This will create a default SpriteFont XML file for your project using the name provided.

If you select Right-click -> Open and open it with your favourite XML editor or notepad.  Once opened it should look as follows:

```xml
<XnaContent xmlns:Graphics="Microsoft.Xna.Framework.Content.Pipeline.Graphics">
  <Asset Type="Graphics:FontDescription">

    <FontName>Arial</FontName>

    <Size>12</Size>

    <Spacing>0</Spacing>

    <UseKerning>true</UseKerning>

    <Style>Regular</Style>

    <CharacterRegions>
      <CharacterRegion>
        <Start>&#32;</Start>
        <End>&#126;</End>
      </CharacterRegion>
    </CharacterRegions>
  </Asset>
</XnaContent>

```

This file simply describes the Font you wish to produce for use in your game, the result of which will be a bitmap image of all the characters to be used.  By updating the XML file you can change:

* The Font name (has to be a font that is currently installed on your workstation)
* The size of the Font
* Character spacing
* Font style (check the source font for supported styles)
* The character regions, allowing you to restrict the font character set to only those characters you intend to use. This saves space and makes the output bitmap smaller

With this done (and saved!), loading the SpriteFont and using it is very easy.

Create a reference for your SpriteFont:

```csharp
    SpriteFont font;
```

Then load the asset file we created earlier, remembering to use the same name as you used in the Editor tool (no file extension)

```csharp
    font = Content.Load<SpriteFont>("myFont");
```

Then finally in your draw call, we call the *SpriteBatch.DrawString()* method passing the font we want to use, the text and finally the colour you want it drawn in.
Here I simply broke out the test to draw to make for easier reading.

```csharp
    string message = "Some really nice text to draw on my screen";

    spriteBatch.Begin();
    spriteBatch.DrawString(font,message,Vector2.Zero,Color.White);
    spriteBatch.End();
```

And that's it.

Check the later tutorials where you can customise the Font building process to tweak your text bitmap.