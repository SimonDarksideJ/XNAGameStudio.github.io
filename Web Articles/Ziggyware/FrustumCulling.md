# Frustum Culling


In order to speed up the rendering of your game you need to use frustum culling

Frustum culling is just a fancy word for making your game engine not render objects that are not visible to the camera

In order to do frustum culling you can use the the six planes that make up the camera viewport:

```cpp
void ZCullManager::UpdateCullInfo(ZCamera pCam)
    {
        Matrix4 mat;
 
        mat = pCam->GetViewProjMatrix();
        mat.Inverse();
 
        m_CullInfo.vecFrustum[0] = Vector3(-1.0f, -1.0f,  0.0f); // xyz
        m_CullInfo.vecFrustum[1] = Vector3( 1.0f, -1.0f,  0.0f); // Xyz
        m_CullInfo.vecFrustum[2] = Vector3(-1.0f,  1.0f,  0.0f); // xYz
        m_CullInfo.vecFrustum[3] = Vector3( 1.0f,  1.0f,  0.0f); // XYz
        m_CullInfo.vecFrustum[4] = Vector3(-1.0f, -1.0f,  1.0f); // xyZ
        m_CullInfo.vecFrustum[5] = Vector3( 1.0f, -1.0f,  1.0f); // XyZ
        m_CullInfo.vecFrustum[6] = Vector3(-1.0f,  1.0f,  1.0f); // xYZ
        m_CullInfo.vecFrustum[7] = Vector3( 1.0f,  1.0f,  1.0f); // XYZ
 
        for( int i = 0; i < 8; i++ )
        {
            m_CullInfo.vecFrustum[i].Transform(mat);
            if(i==0)
            {
                m_CullInfo.m_AABB.m_vMinWorld  = m_CullInfo.vecFrustum[i];
                m_CullInfo.m_AABB.m_vMaxWorld  = m_CullInfo.m_AABB.m_vMinWorld;
            }
            else
            {
                m_CullInfo.m_AABB.m_vMinWorld.Minimize(m_CullInfo.vecFrustum[i]);
                m_CullInfo.m_AABB.m_vMaxWorld.Maximize(m_CullInfo.vecFrustum[i]);
            }
        }
 
 
 
                                            //Near
        m_CullInfo.planeFrustum[0].FromPoints(m_CullInfo.vecFrustum[0],
                                            m_CullInfo.vecFrustum[1],
                                            m_CullInfo.vecFrustum[2]);
 
                                            //Far
        m_CullInfo.planeFrustum[1].FromPoints(m_CullInfo.vecFrustum[6],
                                            m_CullInfo.vecFrustum[7],
                                            m_CullInfo.vecFrustum[5]);
 
                                            //Left
        m_CullInfo.planeFrustum[2].FromPoints(m_CullInfo.vecFrustum[2],
                                            m_CullInfo.vecFrustum[6],
                                            m_CullInfo.vecFrustum[4]);
 
                                            //Right
        m_CullInfo.planeFrustum[3].FromPoints(m_CullInfo.vecFrustum[7],
                                            m_CullInfo.vecFrustum[3],
                                            m_CullInfo.vecFrustum[5]);
 
                                            //Top
        m_CullInfo.planeFrustum[4].FromPoints(m_CullInfo.vecFrustum[2],
                                            m_CullInfo.vecFrustum[3],
                                            m_CullInfo.vecFrustum[6]);
 
                                            //Bottom
        m_CullInfo.planeFrustum[5].FromPoints(m_CullInfo.vecFrustum[1],
                                            m_CullInfo.vecFrustum[0],
                                            m_CullInfo.vecFrustum[4]);
 
    }
```


Here we calculated the six planes that make up the view frustum. Now lets use it to check the visibility of a sphere:

```cpp
    bool ZCullManager::SphereInFrustum( Vector3& p, float radius )
    {
        int i=0;
        Plane* pl = m_CullInfo.planeFrustum;
        while(i<6)
        {
            if( (pl+i)->DotCoord(p) <= -radius )
                return false;
            i++;
        }
        return true;
    }
```

Here we checked to see if the sphere is in front all six planes of the viewport. If the sphere is touching one of the planes or is in front of all six planes, the function returns true.
