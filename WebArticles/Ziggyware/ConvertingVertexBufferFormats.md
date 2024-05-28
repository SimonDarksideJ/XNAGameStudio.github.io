# Converting Vertex Buffer Formats


Converting vertex buffers from one format to another is useful when you need to add more UV coordinates or tangent and binormals to a vertex buffer. Below is an implementation I wrote that handles this for me: 

```cpp
    void ZVertexBufferData::ConvertBuffer(ZVertexDeclaration decl,int iStream)
    {
        DWORD dwNumItems = this->GetNumItems();
        bool bWriteOnly = this->m_bWriteOnly;
        bool bPoints = this->m_bPoints;
        ZVertexDeclaration vdVertexDecl = this->m_Declaration;
        ZVector<char> data;
        ZVector<int> vertMap;
        DWORD dwFVFSize = this->GetFVFSize();
 
        if(!m_bWriteOnly)
        {
            //find mappings
            for(DWORD x=0;x< m_Declaration->m_Elements.size();x++)
            {
                ZVertexElement& thisElem = m_Declaration->m_Elements[x];
                bool bFound=false;
                DWORD i=0;
                for(i=0;i< decl->m_Elements.size();i++)
                {
                    ZVertexElement& elem = decl->m_Elements[i];
                    if(elem.Stream == iStream)
                        if(thisElem == elem)
                        {
                            bFound = true;
                            break;
                        }
                }
                if(bFound)
                {
                    vertMap.push_back(i);
                }
                else
                {
                    vertMap.push_back(-1);
                }
            }
            data.Grow(this->GetLength());
            memcpy(data.begin(),Lock(0,true),this->GetLength());
            this->Unlock();
        }
 
        this->Invalidate();
        this->Initialize(decl);
        this->Create(dwNumItems,bWriteOnly,bPoints);
 
        if(!bWriteOnly)
        {
            char* pData = (char*)this->Lock();
 
            for(DWORD x=0;x<vertMap.size();x++)
            {
                int i = vertMap[x];
 
                if(i != -1)
                {
                    ZVertexElement& elem    = vdVertexDecl->m_Elements[x];
                    ZVertexElement& elemNew = decl->m_Elements[i];
 
                    for(DWORD k=0; k < this->GetNumItems(); k++)
                    {
                        memcpy(pData+(k*m_fvfSize)+elemNew.Offset,
                            data.begin() + (k*dwFVFSize) + elem.Offset,
                            vdVertexDecl->GetElementSize(x));
                    }
                }
            }
            this->Unlock();
        }
    }
```