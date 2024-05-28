# Advanced Polygon Clipping

Some times you need to split a polygon across a plane boundary. This is useful when doing a splitting OCTree or BSP tree.

Splitting a polygon becomes more complex when you have a complex polygon structure like in Direct3D. Below is my implementation of a polygon splitter that uses VB data and its corresponding vertex declaration structure:

```cpp
    void ZTexPolyClipper::ClipToPlane(Plane &p,
                    ZVector<char>& vClipPoly,
                    int iNumInVerts,
                    ZVector<char>& verts,
                    int &iNumRetVerts,
                    int iVertPreFillerSize)
    {
        verts.clear();
        iNumRetVerts=0;
        bool bCurrentIn;
        bool bNextIn;

        int iPosOffset = m_Decl->GetElementByteOffset(D3DDECLUSAGE_POSITION);

        DWORD dwVertSize = m_Decl->GetVertexSize();

        float fCurrentDot = p.DotCoord(*(Vector3*)(vClipPoly.begin()+iVertPreFillerSize+iPosOffset));

        bCurrentIn = IsEqual(fCurrentDot,0) || fCurrentDot > 0;
        float fNextDot;

        //get the side of each vert to the plane

        for(int x=0;x<iNumInVerts;x++)
        {
            int i = ((x == iNumInVerts-1) ? 0 : x+1);

            char *vCurVertex = (vClipPoly.begin()+x*iVertPreFillerSize+x*dwVertSize);

            char *vNextVertex = (vClipPoly.begin()+x*iVertPreFillerSize+i*dwVertSize);

            Vector3 *pCurVertexPos = (Vector3*)(vCurVertex+iVertPreFillerSize+iPosOffset);

            Vector3 *pNextVertexPos = (Vector3*)(vNextVertex+iVertPreFillerSize+iPosOffset);

            if(bCurrentIn)
            {
                verts.Grow(verts.numItems + iVertPreFillerSize+dwVertSize);
                memcpy(verts.begin()+verts.numItems,
                       vCurVertex,
                       iVertPreFillerSize+dwVertSize);

                verts.numItems += iVertPreFillerSize+dwVertSize;
                iNumRetVerts++;
            }

            fNextDot = p.DotCoord(*pNextVertexPos);
            bNextIn  = IsEqual(fNextDot,0) || fNextDot > 0;

            if(bNextIn != bCurrentIn)
            {
                Vector3 vPos;
                if(p.IntersectsLineSegment(vPos,
                                *pCurVertexPos,
                                *pNextVertexPos))
                {
                    float fDist = pCurVertexPos->Distance(*pNextVertexPos);
                    float fDist2 = pCurVertexPos->Distance(vPos);

                    float fScalar;

                    if(!IsEqual(fDist,0))
                    {
                        fScalar = fDist2 / fDist;
                    }
                    else
                    {
                        fScalar = 0;
                    }

                    //lerp elements
                    verts.Grow(verts.numItems + iVertPreFillerSize+dwVertSize);

                    for(DWORD j=0;j<m_Decl->m_Elements.size();j++)
                    {
                        ZVertexElement& e = m_Decl->m_Elements[j];

                        if(e.Type == D3DDECLTYPE_UNUSED)
                            break;

                        switch(e.Usage)
                        {
                        case D3DDECLUSAGE_POSITION:
                        case D3DDECLUSAGE_NORMAL:
                            {
                                Vector3 vCur =  *(Vector3*)(vCurVertex+iVertPreFillerSize+e.Offset);

                                Vector3 vNext =  *(Vector3*)(vNextVertex+iVertPreFillerSize+e.Offset);

                                Vector3 vVal = vCur + (vNext - vCur)*fScalar;

                                memcpy(verts.begin()+verts.numItems+iVertPreFillerSize+e.Offset,

                                    &vVal,sizeof(Vector3));
                            }
                            break;
                        case D3DDECLUSAGE_TEXCOORD:
                            {
                                int iNumCoords=0;

                                if(e.Type == D3DDECLTYPE_FLOAT1)
                                    iNumCoords=1;

                                if(e.Type == D3DDECLTYPE_FLOAT2)
                                    iNumCoords=2;

                                if(e.Type == D3DDECLTYPE_FLOAT3)
                                    iNumCoords=3;

                                if(e.Type == D3DDECLTYPE_FLOAT4)
                                    iNumCoords=4;

                                for(int x=0;x<iNumCoords;x++)
                                {
                                    float fCur =  *(float*)(vCurVertex+iVertPreFillerSize+e.Offset+sizeof(float)*x);

                                    float fNext =  *(float*)(vNextVertex+iVertPreFillerSize+e.Offset+sizeof(float)*x);

                                    float fVal = fCur + (fNext - fCur)*fScalar;

                                    memcpy(verts.begin()+verts.numItems+iVertPreFillerSize+e.Offset+sizeof(float)*x,
                                        &fVal,sizeof(float));
                                }
                            }
                            break;
                        default:
                            {
                                memcpy(verts.begin()+verts.numItems+iVertPreFillerSize+e.Offset,
                                    vCurVertex+iVertPreFillerSize+e.Offset,
                                    m_Decl->GetElementSize(j));
                            }
                        }
                    }

                    verts.numItems += iVertPreFillerSize+dwVertSize;
                    iNumRetVerts++;
                }
            }

            fCurrentDot = fNextDot;
            bCurrentIn = bNextIn;
        }

        if(iNumRetVerts < 3)
            iNumRetVerts=0;
    }
```
