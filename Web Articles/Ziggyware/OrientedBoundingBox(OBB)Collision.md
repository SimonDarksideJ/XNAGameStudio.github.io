# Oriented Bounding Box (OBB) Collision


Here is my implementation of OBB collision between two oriented bounding boxes.

I read the algorithm in the book: Dynamic Simulations of Multibody Systems. This is an excellent source of information on rigid body physics and the mathematics behind collision detection between arbitrary shapes as well as multiple simultaneous contacts tons of valuable content for anyone interested in physics programming :) This is a must-have for any game developer!

```cpp


   ZCollision::ZCollisionType ZCollision::OOBBCollision(ZOOBB &a,ZOOBB &b)
    {
        Matrix4 matB = b.m_matRot*a.m_matRotInverse;
        Vector3 vPosB = a.m_matRotInverse * (b.m_vCenter - a.m_vCenter);


        Vector3 XAxis(matB._11,matB._21,matB._31);
        Vector3 YAxis(matB._12,matB._22,matB._32);
        Vector3 ZAxis(matB._13,matB._23,matB._33);
 
        //15 tests
        //1 (Ra)x
        if(fabs(vPosB.x) > a.m_vBounds.x + b.m_vBounds.x * fabs(XAxis.x) + b.m_vBounds.y * fabs(XAxis.y) + b.m_vBounds.z * fabs(XAxis.z))
            return ZCollisionType::None;
        //2 (Ra)y
        if(fabs(vPosB.y) > a.m_vBounds.y + b.m_vBounds.x * fabs(YAxis.x) + b.m_vBounds.y * fabs(YAxis.y) + b.m_vBounds.z * fabs(YAxis.z))
            return ZCollisionType::None;
        //3 (Ra)z
        if(fabs(vPosB.z) > a.m_vBounds.z + b.m_vBounds.x * fabs(ZAxis.x) + b.m_vBounds.y * fabs(ZAxis.y) + b.m_vBounds.z * fabs(ZAxis.z))
            return ZCollisionType::None;
 
        //4 (Rb)x
        if(fabs(vPosB.x*XAxis.x+vPosB.y*YAxis.x+vPosB.z*ZAxis.x) >
            (b.m_vBounds.x+a.m_vBounds.x*fabs(XAxis.x) + a.m_vBounds.y * fabs(YAxis.x) + a.m_vBounds.z*fabs(ZAxis.x)))
            return ZCollisionType::None;
        //5 (Rb)y
        if(fabs(vPosB.x*XAxis.y+vPosB.y*YAxis.y+vPosB.z*ZAxis.y) >
            (b.m_vBounds.y+a.m_vBounds.x*fabs(XAxis.y) + a.m_vBounds.y * fabs(YAxis.y) + a.m_vBounds.z*fabs(ZAxis.y)))
            return ZCollisionType::None;
        //6 (Rb)z
        if(fabs(vPosB.x*XAxis.z+vPosB.y*YAxis.z+vPosB.z*ZAxis.z) >
            (b.m_vBounds.z+a.m_vBounds.x*fabs(XAxis.z) + a.m_vBounds.y * fabs(YAxis.z) + a.m_vBounds.z*fabs(ZAxis.z)))
            return ZCollisionType::None;
 
        //7 (Ra)x X (Rb)x
        if(fabs(vPosB.z*YAxis.x-vPosB.y*ZAxis.x) > a.m_vBounds.y*fabs(ZAxis.x) + 
            a.m_vBounds.z*fabs(YAxis.x) + b.m_vBounds.y*fabs(XAxis.z) + b.m_vBounds.z*fabs(XAxis.y))
            return ZCollisionType::None;
        //8 (Ra)x X (Rb)y
        if(fabs(vPosB.z*YAxis.y-vPosB.y*ZAxis.y) > a.m_vBounds.y*fabs(ZAxis.y) + 
            a.m_vBounds.z*fabs(YAxis.y) + b.m_vBounds.x*fabs(XAxis.z) + b.m_vBounds.z*fabs(XAxis.x))
            return ZCollisionType::None;
        //9 (Ra)x X (Rb)z
        if(fabs(vPosB.z*YAxis.z-vPosB.y*ZAxis.z) > a.m_vBounds.y*fabs(ZAxis.z) + 
            a.m_vBounds.z*fabs(YAxis.z) + b.m_vBounds.x*fabs(XAxis.y) + b.m_vBounds.y*fabs(XAxis.x))
            return ZCollisionType::None;
 
        //10 (Ra)y X (Rb)x
        if(fabs(vPosB.x*ZAxis.x-vPosB.z*XAxis.x) > a.m_vBounds.x*fabs(ZAxis.x) + 
            a.m_vBounds.z*fabs(XAxis.x) + b.m_vBounds.y*fabs(YAxis.z) + b.m_vBounds.z*fabs(YAxis.y))
            return ZCollisionType::None;
        //11 (Ra)y X (Rb)y
        if(fabs(vPosB.x*ZAxis.y-vPosB.z*XAxis.y) > a.m_vBounds.x*fabs(ZAxis.y) + 
            a.m_vBounds.z*fabs(XAxis.y) + b.m_vBounds.x*fabs(YAxis.z) + b.m_vBounds.z*fabs(YAxis.x))
            return ZCollisionType::None;
        //12 (Ra)y X (Rb)z
        if(fabs(vPosB.x*ZAxis.z-vPosB.z*XAxis.z) > a.m_vBounds.x*fabs(ZAxis.z) + 
            a.m_vBounds.z*fabs(XAxis.z) + b.m_vBounds.x*fabs(YAxis.y) + b.m_vBounds.y*fabs(YAxis.x))
            return ZCollisionType::None;
 
        //13 (Ra)z X (Rb)x
        if(fabs(vPosB.y*XAxis.x-vPosB.x*YAxis.x) > a.m_vBounds.x*fabs(YAxis.x) + 
            a.m_vBounds.y*fabs(XAxis.x) + b.m_vBounds.y*fabs(ZAxis.z) + b.m_vBounds.z*fabs(ZAxis.y))
            return ZCollisionType::None;
        //14 (Ra)z X (Rb)y
        if(fabs(vPosB.y*XAxis.y-vPosB.x*YAxis.y) > a.m_vBounds.x*fabs(YAxis.y) + 
            a.m_vBounds.y*fabs(XAxis.y) + b.m_vBounds.x*fabs(ZAxis.z) + b.m_vBounds.z*fabs(ZAxis.x))
            return ZCollisionType::None;
        //15 (Ra)z X (Rb)z
        if(fabs(vPosB.y*XAxis.z-vPosB.x*YAxis.z) > a.m_vBounds.x*fabs(YAxis.z) + 
            a.m_vBounds.y*fabs(XAxis.z) + b.m_vBounds.x*fabs(ZAxis.y) + b.m_vBounds.y*fabs(ZAxis.x))
            return ZCollisionType::None;
        return ZCollisionType::Embedded;
    }
```