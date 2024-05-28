# Calculating Tangents and Binormals/Bitangents


Tangents and binormals are used in advanced shaders like parallax mapping. 
Check out the source below to see how you can calculate these values given a vertex buffer and vertex declaration (DX9)

```cpp
    void ZTanBitan::CalculateTangents(ZRenderable* pRenderable, ZVertexBuffer pTanBitanVB,bool bSaveTan,bool bSaveBitan,int iStream)
    {
        ZVertexDeclaration decl;


        if(pTanBitanVB)
            decl = pTanBitanVB->GetDeclaration();
        else
            decl = pRenderable->GetVB()->GetDeclaration();

        if(decl->GetElementByteOffset(D3DDECLUSAGE_TANGENT,0,iStream)==-1 ||
           decl->GetElementByteOffset(D3DDECLUSAGE_BINORMAL,0,iStream)==-1 )
        {
            
            int iEndOffset=0;

            if(decl->GetElementByteOffset(D3DDECLUSAGE_TANGENT,0,iStream)==-1)
            {
                decl->m_Elements.pop_back();

                ZVertexElement e;
                e.Method = D3DDECLMETHOD_DEFAULT;
                e.Offset = decl->m_Elements.end()->Offset + decl->GetElementSize(decl->m_Elements.size()-1);
                e.Usage = D3DDECLUSAGE_TANGENT;
                e.Stream = iStream;
                e.Type = D3DDECLTYPE_FLOAT3;
                e.UsageIndex = 0;
                decl->m_Elements.push_back(e);
                
                ZVertexElement end;
                decl->m_Elements.push_back(end);

                iEndOffset = e.Offset + decl->GetElementSize(decl->m_Elements.size()-1);
            }

            if(decl->GetElementByteOffset(D3DDECLUSAGE_BINORMAL,0,iStream)==-1)
            {
                decl->m_Elements.pop_back();

                ZVertexElement e;
                e.Method = D3DDECLMETHOD_DEFAULT;
                e.Offset = decl->m_Elements.end()->Offset + decl->GetElementSize(decl->m_Elements.size()-1);
                e.Usage = D3DDECLUSAGE_BINORMAL;
                e.Stream = iStream;
                e.Type = D3DDECLTYPE_FLOAT3;
                e.UsageIndex = 0;
                decl->m_Elements.push_back(e);

                ZVertexElement end;
                decl->m_Elements.push_back(end);

                iEndOffset = e.Offset + decl->GetElementSize(decl->m_Elements.size()-1);
            }

            decl->UpdateDevice();
            
            if(pTanBitanVB)
                pTanBitanVB->ConvertBuffer(decl,iStream);
            else
                pRenderable->GetVB()->ConvertBuffer(decl,iStream);
        }

        char* pVerts=0;
        
        if(pTanBitanVB)
            pVerts = (char*)pTanBitanVB->Lock(0,false,true,false);
        else
            pVerts = (char*)pRenderable->GetVB()->Lock(0,false,true,false);
        
        int iPosOffset = decl->GetElementByteOffset(D3DDECLUSAGE_POSITION,0,iStream);
        int iNormalOffset = decl->GetElementByteOffset(D3DDECLUSAGE_NORMAL,0,iStream);
        int iTexOffset = decl->GetElementByteOffset(D3DDECLUSAGE_TEXCOORD,0,iStream);
        int iTanOffset = decl->GetElementByteOffset(D3DDECLUSAGE_TANGENT,0,iStream);
        int iBitanOffset = decl->GetElementByteOffset(D3DDECLUSAGE_BINORMAL,0,iStream);
        DWORD dwVertSize = decl->GetVertexSize(iStream);

        WORD* pIndices = (WORD*)pRenderable->GetIB()->Lock(true);

        DWORD numVerts = pRenderable->GetVB()->GetNumItems();
        for (DWORD i = 0; i < numVerts; i++)
        {
            if(iTanOffset != -1)
            {
                *(Vector3*)(pVerts+i*dwVertSize+iTanOffset) = Vector3(0,0,0);
            }
            if(iBitanOffset != -1)
            {
                *(Vector3*)(pVerts+i*dwVertSize+iBitanOffset) = Vector3(0,0,0);
            }
        }

        DWORD count = pRenderable->GetIB()->GetNumIndices()/3;
        for (DWORD i = 0; i < count; i++)
        {
            Vector3 v1 = *(Vector3*)(pVerts+pIndices[i*3+0]*dwVertSize+iPosOffset);
            Vector3 v2 = *(Vector3*)(pVerts+pIndices[i*3+1]*dwVertSize+iPosOffset);
            Vector3 v3 = *(Vector3*)(pVerts+pIndices[i*3+2]*dwVertSize+iPosOffset);
            
            Vector2 w1 = *(Vector2*)(pVerts+pIndices[i*3+0]*dwVertSize+iTexOffset);
            Vector2 w2 = *(Vector2*)(pVerts+pIndices[i*3+1]*dwVertSize+iTexOffset);
            Vector2 w3 = *(Vector2*)(pVerts+pIndices[i*3+2]*dwVertSize+iTexOffset);

            float x1 = v2.x - v1.x;
            float x2 = v3.x - v1.x;
            float y1 = v2.y - v1.y;
            float y2 = v3.y - v1.y;
            float z1 = v2.z - v1.z;
            float z2 = v3.z - v1.z;

            float s1 = w2.x - w1.x;
            float s2 = w3.x - w1.x;
            float t1 = w2.y - w1.y;
            float t2 = w3.y - w1.y;

            float r = 1.0f / (s1 * t2 - s2 * t1);
            Vector3 sDir((t2 * x1 - t1 * x2) * r, (t2 * y1 - t1 * y2) * r,
                (t2 * z1 - t1 * z2) * r);
            Vector3 tDir((s1 * x2 - s2 * x1) * r, (s1 * y2 - s2 * y1) * r,
                (s1 * z2 - s2 * z1) * r);


            if(iTanOffset != -1)
            {
                *(Vector3*)(pVerts+pIndices[i*3+0]*dwVertSize+iTanOffset) += sDir;
                *(Vector3*)(pVerts+pIndices[i*3+1]*dwVertSize+iTanOffset) += sDir;
                *(Vector3*)(pVerts+pIndices[i*3+2]*dwVertSize+iTanOffset) += sDir;
            }
            if(iBitanOffset != -1)
            {
                *(Vector3*)(pVerts+pIndices[i*3+0]*dwVertSize+iBitanOffset) += tDir;
                *(Vector3*)(pVerts+pIndices[i*3+1]*dwVertSize+iBitanOffset) += tDir;
                *(Vector3*)(pVerts+pIndices[i*3+2]*dwVertSize+iBitanOffset) += tDir;
            }
        }

        

        count = pRenderable->GetVB()->GetNumItems();
        for (DWORD i = 0; i < count; i++)
        {
            Vector3 n = *(Vector3*)(pVerts+i*dwVertSize+iNormalOffset);
            Vector3 t;
            if(iTanOffset != -1)
            {
                t = *(Vector3*)(pVerts+i*dwVertSize+iTanOffset);
            
                *(Vector3*)(pVerts+i*dwVertSize+iTanOffset) = 
                    (t - n * (n.Dot(t))).Normal();

                if(iBitanOffset != -1)
                {
                    // Calculate handedness
                    float w = (Vector3::CrossProduct(n,t).Dot(*(Vector3*)(pVerts+i*dwVertSize+iBitanOffset)) < 0.0f) ? -1.0f : 1.0f;
                    *(Vector3*)(pVerts+i*dwVertSize+iBitanOffset) = Vector3::CrossProduct(n, t);
                    *(Vector3*)(pVerts+i*dwVertSize+iBitanOffset) *= w;
                    ((Vector3*)(pVerts+i*dwVertSize+iBitanOffset))->Normalize();
                }
            }
            else if(iBitanOffset != -1)
            {
                t = *(Vector3*)(pVerts+i*dwVertSize+iBitanOffset);

                *(Vector3*)(pVerts+i*dwVertSize+iBitanOffset) = 
                    (t - n * (n.Dot(t))).Normal();
            }
        }

        pRenderable->GetVB()->Unlock();
        pRenderable->GetIB()->Unlock();
```