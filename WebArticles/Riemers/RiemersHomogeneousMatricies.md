# Homogeneous matrices

If you access one of the matrices used by DirectX, you’ll see they have 16 components, which means they consist of 4 rows and 4 columns. This format was chosen, to allow these matrices to also represent translations.

To translate a point, we need to ADD a number to each of its components. By using 3x3 matrices, we can only get coordinates that are multiples of the original coordinates. So next to X,Y and Z, we’ll add a constant to our coordinates. For simplicity, let’s set this to 1. So from now on, we’re going to represent 3D points by 4 coordinates. For example, the point at position (10,5,0) will get coordinates (10,5,0,1). These coordinates are called the homogeneous coordinates of a point.

The scaling and rotation matrices remain the same, but the get an additional row and columns that are 0, except for the point m44, which is 1. Here you can see what the scaling and Y-rotation matrices look like in their homogenous form:

![Example 1](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/RiemersHomogeneousMatrix1.jpg?raw=true)

This looks a little bit more complex, but now we can at last also define a translation matrix:

![Example 2](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/RiemersHomogeneousMatrix2.jpg?raw=true)

Let’s have a look at a small example. Suppose we want to translate the point (10,5,0) into the direction (-8,2,4). This is how it’s done:

![Example 3](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/RiemersHomogeneousMatrix3.jpg?raw=true)

So this gives use the point (2,7,4). This result is of course very obvious, but the point I’m trying to make here is that now we fulfill the 3 basic properties:

1) We have a matrix corresponding to every basic transformation
2) Multiplying such a matrix with the coordinates of a point will give the coordinates of the transformed point
3) Multiplying 2 matrices gives a new matrix, that corresponds to the combined transformations, corresponding to the 2 starting matrices

If you have been following up to this point, you’ll notice the 4th coordinate has absolutely NO geometrical meaning. It’s just there to allow us to define a translation matrix, that has the same shape as a scaling and rotation matrix.

This actually concludes these pages on matrices. One final remark however: sometimes you’ll notice this constant is not 1, as was the case in this theory. In fact, the general rule says you simply have to divide the X,Y and Z coordinate by this 4th coordinate. Let’s call this 4th coordinate W from now on.

Put simply: (20,10,0,2) = (10,5,0,1) both represent the same 3D point (10,5,0). So the simple rule to derive the 3D point out of 4 coordinates is:

![Example 4](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/RiemersHomogeneousMatrix4.jpg?raw=true)
