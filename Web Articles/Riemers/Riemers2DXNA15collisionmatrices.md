# 2D Transformation matrices

In this chapter we will see how we can make use of the **TexturesCollide method** created in the last chapter, from the method definition, you can see that it requires the 2D color array and the transformation matrix for both images to compare.

## Preparing the color data

Let us start with the 2D color array and extract the 2D color arrays from our images using the **Textureto2DArray** method we created earlier and store them in a variable. 

Begin by adding these variable to the Properties section of our code:

```csharp
    private Color[,] _rocketColorArray;
    private Color[,] _foregroundColorArray;
    private Color[,] _carriageColorArray;
    private Color[,] _cannonColorArray;
```

> Note that these are the only images in this game that we need to check for possible collisions, checking between our rocket, the terrain and the players.

Next, initialize each of the loaded images into arrays at the end of our LoadContent method as follows:

```csharp
    _rocketColorArray = TextureTo2DArray(_rocketTexture);
    _carriageColorArray = TextureTo2DArray(_carriageTexture);
    _cannonColorArray = TextureTo2DArray(_cannonTexture);
```

The 2D color array of the foreground needs to be extracted every time the **CreateForeground** method is called, so add this line at the end of the **CreateForeground** method:

```csharp
    _foregroundColorArray = TextureTo2DArray(_foregroundTexture);
```

Now that we have the 2D color arrays of our textures readily available there is one more thing we need, the transformation matrices of our images. These matrices describe how images are transformed (moved, rotated, scaled) before they are rendered to the screen. We will construct the 4 transformation images, starting with the easiest: the matrix for the foreground image.

## Calculating the foreground matrix

> The rest of this chapter is theory only, we will implement it in the next chapter

For the foreground image, this is rather easy. Remember that we created this texture so it already has exactly the same size as the screen, so the foreground image can be rendered to the screen exactly the way it is, this means we need a transformation that actually does not change the image.

To specify the Identity matrix as a transformation matrix of our foreground image, the Identity matrix makes sure the image remains identical.

> You can compare it to the identity element of the "+" operation, which is 0, [a number] + 0 = [a number].
>
> For the "*" operation, the identity element is 1, [a number] * 1 = [a number].
>
> Now for matrices: [any image] * Matrix.Identity = [an image].

So this line would create the matrix for foreground texture:

```csharp
    Matrix foregroundMat = Matrix.Identity;
```

## Calculating the carriage matrix

Next in line is the matrix for one of the carriages. Before they are rendered to the screen they are moved to their correct position, scaled-down, and then moved again to its origin position. For each of these separate transformations, we can construct the global transformation matrix which nothing more than the multiplication of all these separate matrices.

> There is one very important rule that you need to keep in mind, **the order of matrix multiplications is important**. In matrix multiplications, you should translate "*" to "after".

With that said, let us go over the transformations that MonoGame does before rendering a carriage to the screen.

> **Very important: these operations happen in the exact order specified here.**

The steps correspond to the image underneath them.

1. If we would render the image just like it is (or, with the Identity transformation), it would be rendered in its original size in the top-left corner of the screen.
2. First, the carriage image is moved so its top-left point is at the position specified as the second argument in the *SpriteBatch.Draw* method.
3. Then, everything is scaled down. You see this includes the images, as well as its own X and Y axis
4. Finally, and this is the most challenging step: the image is moved over the Y axis, since in our SpriteBatch.Draw method we’ve specified (0, carriageTexture.Height) as the origin.

> **Very important: the texture is moved over its own Y axis, which has been scaled down. So instead of being moved over 39 screen pixels, the carriages will be moved vertically over 39*0.4=16 pixels (since carriageTexture.Height = 39 and playerScaling = 0.4)**

![Collision checks](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/2DXNA15Collision1.png?raw=true)

> Note that things would be different if step 2 and step 3 were interchanged, first the image would be moved vertically over 39 screen pixels and AFTER that, the image and axis would be scaled down which would result in our carriages floating in thin air!

Let us rewrite this, in matrices:

1. Matrix.Identity;
2. Matrix.CreateTranslation(xPos, yPos, 0)
3. Matrix.CreateScale(playerScaling)
4. Matrix.CreateTranslation(0, -carriage.Height, 0)

The **global transformation matrix** is obtained by multiplying them all together:

```csharp
    Matrix carriageMat = Matrix.CreateTranslation(0, -carriage.Height, 0) *
                         Matrix.CreateScale(_playerScaling) *
                         Matrix.CreateTranslation(xPos, yPos, 0) *
                         Matrix.Identity;
```

Remember our discussion about the Identity matrix being the “1” in multiplication, this means we can also leave it away in the line above (since for example a*b*c*1 = a*b*c).

## Calculating the rocket matrix

Next, we approach the most general example possible, it is time to construct the transformation matrix for the rocket. Let us go over the different transformations that are applied to the rocket while taking a look at the corresponding image below them:

1: If we would render the image just like it is (or: with the Identity transformation), it would be rendered in its original size in the top-left corner of the screen.
2: First, the rocket image is moved so its top-left point is at the position specified as the second argument in the SpriteBatch.Draw method.
3: Then, everything is scaled down. You see this includes the image, as well as its own X and Y axis
4: Then, everything is rotated. Again, this includes the image as well as its own X and Y axis.
5: Finally, the resulting image is moved for 42 pixels over its transformed X axis, and 240 pixels over its transformed Y axis. To find out how it will be moved relative to the screen we check which transformations have already had an impact on the X and Y axis. In step 2, the axis has been scaled down factor 10 (since rocketScaling is 0.1). In step 3, these downscaled axis are rotated over an angle specified in rocketAngle. This means with rocketAngle=0, the image would be moved over (4.2, 24) pixels. In the case of a rotation, the image will be moved (42, 240) pixels over the axis, which has been scaled down and rotated. As an example: in step 2, we have scaled our rocket image down by factor 10. Let’s say we have not set a rotation in step 3. As a result, the image will be moved over (4.2, 24) pixels.

Since this is the most general case, you can find the transformation matrix of any image you draw using this rule.

![Collision checks](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/2DXNA15Collision2.png?raw=true)

Note that this yields the result we wanted: the rotational center of the rocket is at the position we specified in the SpriteBatch.Draw method and the rocket is rotated and scaled as we wanted.

I already mentioned the order of transformations is important. As an example, the image below shows what would happen when steps 4 and 5 would be swapped:

![Collision checks](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/2DXNA15Collision3.png?raw=true)

After the coordinate system is rotated it ends up at a totally different position, this results in the rocket being rendered partially off-screen.

Let us rewrite the correct way of transformations, this time in matrices. 

> Note that not using the Identity matrix as the first step

1. First: Matrix.CreateTranslation(rocketPosition.X, rocketPosition.Y, 0)
2. Then : Matrix.CreateScale(rocketScaling)
3. Then : Matrix.CreateRotationZ(rocketAngle)
4. Finally: Matrix.CreateTranslation(-42, -240, 0)

And since we need to combine all of them together, we multiply them together:

```csharp
    Matrix rocketMat = Matrix.CreateTranslation(-42, -240, 0) *
                       Matrix.CreateRotationZ(_rocketAngle) *
                       Matrix.CreateScale(_rocketScaling) *
                       Matrix.CreateTranslation(_rocketPosition.X, _rocketPosition.Y, 0);
```

This is the most general transformation you will find when creating any 2D game.

## Comparing SpriteBatch draw to matrices

If we compare these matrices to the arguments used on your SpriteBatch.Draw, the left-most matrix contains the origin point you specified in the SpriteBatch.Draw method, then the angle, then the scaling and finally the position on the screen which you specified as the second argument in the SpriteBatch.Draw call.

As an example, we will compare the transformation matrix of the cannon to our existing cannon SpriteBatch.Draw call:

```csharp
    _spriteBatch.Draw(_cannonTexture, new Vector2(xPos + 20, yPos - 10), null, _players[i].Color, _players[i].Angle, cannonOrigin, _playerScaling, SpriteEffects.None, 1);
```

Which translates to the following matrix: (is the same as)

```csharp
    Matrix cannonMat = Matrix.CreateTranslation(cannonOrigin) *
                       Matrix.CreateRotationZ(_players[i].Angle) *
                       Matrix.CreateScale(_playerScaling) *
                       Matrix.CreateTranslation(xPos + 20, yPos - 10, 0);
```

That will be all for this chapter. At this moment, we have the 2D arrays of colors of our images and the templates for the transformation matrices of each texture that needs them. We will put all of them together in the next chapter.

The Matrix's discussed in this chapter:

```csharp
    Matrix foregroundMat = Matrix.Identity;

    Matrix carriageMat = Matrix.CreateTranslation(0, -carriage.Height, 0) *
                         Matrix.CreateScale(_playerScaling) *
                         Matrix.CreateTranslation(xPos, yPos, 0) *
                         Matrix.Identity;

    Matrix rocketMat = Matrix.CreateTranslation(-42, -240, 0) *
                       Matrix.CreateRotationZ(_rocketAngle) *
                       Matrix.CreateScale(_rocketScaling) *
                       Matrix.CreateTranslation(_rocketPosition.X, _rocketPosition.Y, 0);

    Matrix cannonMat = Matrix.CreateTranslation(cannonOrigin) *
                       Matrix.CreateRotationZ(_players[i].Angle) *
                       Matrix.CreateScale(_playerScaling) *
                       Matrix.CreateTranslation(xPos + 20, yPos - 10, 0);
```

## Next Steps

[Putting Collision Detection into practice](Riemers2DXNA16puttingcdintoaction)
