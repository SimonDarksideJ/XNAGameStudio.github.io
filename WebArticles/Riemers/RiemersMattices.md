# Matrices and DirectX

What exactly are those matrices we keep using in our DirectX programs? Maybe you already have an idea about the concept behind them, but want full control. Maybe you don’t care about the maths, but want the geometric meaning behind them. In both cases, these 3 pages are for you.

The first page we’ll talk about the geometric reasoning behind matrices, which can be summarized in 3 easy-to-understand properties. The second page we’ll cover the basic maths behind them and we’ll finish with homogeneous matrices, which are the matrices actually used by DirectX.

Transformations
You all know transformations, these transform the coordinates of any 3D point into the coordinates of another 3D point. Below you see an example of the 3 basic transformations:

![Translations](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/RiemersMatrices1.jpg?raw=true)

The first transformation will simply move all points in 3D space down and to the left. Such a ‘movement’ transformation is called a translation. The second transformation you see is the rotation, where all points in 3D space are rotated around a specified axis, in this case around the Z-axis. The last basic transformation is the scaling, where the coordinates of all points are multiplied by a certain number.

I call these the ‘basic’ transformations, because any transformation you can think of can be described as a combination of these 3 basic transformations. Below you can see the result of a translation, followed by a rotation:

![Rotation](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/RiemersMatrices2.jpg?raw=true)

This actually is the transformation we use in Series1, where we first translate the middle of our terrain to our origin, and then rotate it when the user presses a key.

## The 3 properties

### Property 1

A matrix can be thought of as a special (but not complicated) element, that can describe any of above transformations. So imagine you have 3 matrices; Mtrans, Mrot and Mscal; that each correspond to one of the transformations above.

### Property 2

If you multiply such a matrix with the coordinates of any 3D point, you get the coordinates of the transformed point. As an illustration:

![Multiply](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/RiemersMatrices3.jpg?raw=true)

This means for any point, you can find its projection by simply multiplying its coordinates with a matrix! This of course also holds for the rotation and scaling transformations.

### Property 3
If you multiply 2 matrices M1 and M2, you get a new matrix M3. The transformation that corresponds to this new matrix M3, is the combination of the transformations corresponding to the first 2 matrices M1 and M2.
This third property is also EXTREMELY useful. Imagine the example above where we have a translation followed by a rotation. One way to calculate the positions of the projected points, would be to first calculate the points, transformed by the translation matrix M1, and then to calculate the rotated projections of these points by multiplying them with M2. This implies you would have to calculate a projection for each point TWICE. This is illustrated by the following equations:

![Combinations](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/RiemersMatrices4.jpg?raw=true)

Using the third property, you simply first multiply M1 and M2 to get M3. Now you only need to multiply your points with this matrix M3 to get the rotated and translated coordinates of your points! Multiplication of 2 matrices happens very fast (64 multiplications and 48 sums) while transforming all the points in your scene takes MUCH longer (16 multiplications and 12 sums for each point)

![Translation](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/RiemersMatrices5./jpg?raw=true)

These 3 properties conclude this first page on the geometric meaning behind matrices. Next page we’re going to see what matrices exactly look like.
