# The maths behind Matrices

The fact you’re reading this page means you already have a basic idea what matrices are used for by DirectX, but you want to go a little further. So let’s start by showing what a matrix actually looks like.

A matrix is nothing more than a table of numbers. Much like your basic Excel sheet; or a 2D array of floats, if you’re more into that. To process 3D coordinates, matrices of 3 columns and 3 rows are used, much like you can see here:

![Matrix](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/RiemersMatrixMath1.jpg?raw=true)

If you want, you can represent the 3 coordinates of a 3D points as a matrix of 1 column and 3 rows. Then multiplication between such a point and a matrix is defined like this:

![Matrix Multiplication](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/RiemersMatrixMath2.jpg?raw=true)

Now you know everything that’s going on there! And as you can see, nothing difficult. Maybe it’s time for a small example, so you can see this in action. Imagine we have a point in 3D space with coordinates (6,18,9.5). Let’s start with a simple example: we want to enlarge our scene, so all objects will look twice as large. This means we’re going to scale by factor 2. In general, a scaling matrix simply looks like this:

![Scale](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/RiemersMatrixMath3.jpg?raw=true)

Which gives

![Scaled](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/RiemersMatrixMath4.jpg?raw=true)

In the case of our small example, s = 2, thus the coordinates of our transformed point are (12, 36, 19).

Now we’ve seen the scaling matrix, let’s move on and have a look at the rotation matrices. These look a little more complex, but remember that all sin and cos are in fact a number between -1 and +1. These 3 matrices represent the rotation around the X,Y and Z axis respectively over an angle theta:

![Translation](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/RiemersMatrixMath5.jpg?raw=true)

A small example would be in place I guess. Say we want to rotate the point (10,5,0) around the Z axis over 45 degrees. Our translation matrix would be:

![Example](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/RiemersMatrixMath6.jpg?raw=true)

Since pi=3.14 corresponds to 180 degrees, 45 degrees corresponds to pi/4.
This gives us:

![Example](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/RiemersMatrixMath7.jpg?raw=true)

So when you rotate the point (10,5,0) for 45 degrees, you get the point (10.6065, -3.5355, 0)! Luckily, DirectX takes care of all these maths for us, but now you have an idea of how this all works.

There is only one problem left. There is no such matrix you can multiply with a point to get the translated version of this point. To solve this, DirectX uses homogeneous matrices, which we’ll see in page 3.
