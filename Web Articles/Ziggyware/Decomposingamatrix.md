# Decomposing a matrix


Matrix decomposition is very useful when you would like to get the translation, scale or rotation from a matrix. This should not be done very often however since it is an expensive operation.

Here is a Matrix Decomposition function declaration

```cpp
Parameters: const Matrix& mat
```

The input matrix to decompose into its translation, scale and rotation elements

```cpp
Vector3& vTrans
```

The output decomposed translation Vector

```cpp
Vector3& vScale
```

The output decomposed scale Vector

```cpp
Matrix4& mRot
```

The output decomposed rotation Matrix

```cpp
    inline void MatrixDecompose(const Matrix4& mat, 
                         Vector3& vTrans,
                         Vector3& vScale,
                         Matrix4& mRot)
    {
```

Retrieving the translation is as easy as getting the x,y,z values from the thrid row: 

```cpp
        vTrans.x = mat.m_val[3][0];
        vTrans.y = mat.m_val[3][1];
        vTrans.z = mat.m_val[3][2];
```


We create a temporary Vector3 array to store the 3x3 matrix that contains the scaling and rotation. We will then take this information and seperate it into its scale and rotation components. 

```cpp
        Vector3 vCols[3] = {
                Vector3(mat.m_val[0][0],mat.m_val[0][1],mat.m_val[0][2]),
                Vector3(mat.m_val[1][0],mat.m_val[1][1],mat.m_val[1][2]),
                Vector3(mat.m_val[2][0],mat.m_val[2][1],mat.m_val[2][2])
        };
```

Retrieving the scale is done by gathering the legnth of the vectors for each column of the 3x3 matrix. 

```cpp
        vScale.x = vCols[0].Length();
        vScale.y = vCols[1].Length();
        vScale.z = vCols[2].Length();
```


The 3x3 rotation matrix can be obtained by dividing each column of the 3x3 rotation/scale matrix by the retrieved scalar component: 

```cpp
        if(vScale.x != 0)
        {
            vCols[0].x /= vScale.x;
            vCols[0].y /= vScale.x;
            vCols[0].z /= vScale.x;
        }
        if(vScale.y != 0)
        {
            vCols[1].x /= vScale.y;
            vCols[1].y /= vScale.y;
            vCols[1].z /= vScale.y;
        }
        if(vScale.z != 0)
        {
            vCols[2].x /= vScale.z;
            vCols[2].y /= vScale.z;
            vCols[2].z /= vScale.z;
        }

        //unroll the loop to increase the speed

        /*for(int x=0;x<3;x++)
        {
        mRot.m_val[0][x] = vCols[x].x;
        mRot.m_val[1][x] = vCols[x].y;
        mRot.m_val[2][x] = vCols[x].z;
        mRot.m_val[x][3] = 0;
        mRot.m_val[3][x] = 0;
        }*/
        mRot.m_val[0][0] = vCols[0].x;
        mRot.m_val[1][0] = vCols[0].y;
        mRot.m_val[2][0] = vCols[0].z;
        mRot.m_val[0][3] = 0;
        mRot.m_val[3][0] = 0;
        mRot.m_val[0][1] = vCols[1].x;
        mRot.m_val[1][1] = vCols[1].y;
        mRot.m_val[2][1] = vCols[1].z;
        mRot.m_val[1][3] = 0;
        mRot.m_val[3][1] = 0;
        mRot.m_val[0][2] = vCols[2].x;
        mRot.m_val[1][2] = vCols[2].y;
        mRot.m_val[2][2] = vCols[2].z;
        mRot.m_val[2][3] = 0;
        mRot.m_val[3][2] = 0;  
        mRot.m_val[3][3] = 1;
    }
```