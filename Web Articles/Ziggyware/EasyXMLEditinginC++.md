# Easy XML Editing in C++


By using your own XML classes instead of those provided by MSXML you can get more readable code and reliability when manipulating complex XML data:

I highly recommend staying away from using MSXML directly as i've found this to be very difficult to read when modifying other peoples code.

Below is a simple way to extract XML from the MSXML structure into your own data format:

```cpp
    IXMLDOMDocument* Load(const char* pStrXML)
    {
        IXMLDOMDocument* pDoc = NULL;
 
        CoInitialize(NULL);
 
        ZString strXML = pStrXML;
        if(FAILED(CoCreateInstance(CLSID_DOMDocument, NULL, CLSCTX_INPROC_SERVER,
                                    IID_IXMLDOMDocument, (void**)&pDoc)))
        {
            pDoc = NULL;
        }
        else
        {
            VARIANT_BOOL bOK = 0;
 
            LPWSTR b = strXML.AllocWideChar();
 
		    HRESULT hr;
            if(FAILED(hr = pDoc->loadXML((BSTR)b,&bOK)) || !bOK)
            {
                pDoc->Release();
                pDoc = NULL;
            }
 
            strXML.FreeWideChar(b);
        }
 
        return pDoc;
    }
 
    IXMLNode WalkNodes(IXMLDOMNode* pNode)
    {
        IXMLNode pRet;
 
        //get node name
        BSTR bNodeName;
        ZString strNodeName;
        pNode->get_nodeName(&bNodeName);
        strNodeName = bNodeName;
        ::SysFreeString(bNodeName);
 
        if(strNodeName.GetLength() && strNodeName.m_Data[0] != '#')
        {
            //get node type
            tagDOMNodeType NodeType;
            pNode->get_nodeType(&NodeType);
 
	        if (NodeType == NODE_ELEMENT)
	        {
                pRet.Initialize();
                pRet->SetName(strNodeName);
 
                // Element
		        BSTR bstrNodeText;
		        pNode->get_text(&bstrNodeText);
                ZString str;
                str = bstrNodeText;
		        pRet->SetValue(str);
                ::SysFreeString(bstrNodeText);
 
                IXMLDOMNamedNodeMap* pAttribMap = NULL;
                pNode->get_attributes(&pAttribMap);
 
                long lNumAttribs=0;
                pAttribMap->get_length(&lNumAttribs);
 
                for(int x=0;x<lNumAttribs;x++)
                {
                    IXMLDOMNode *pAttribNode = NULL;
                    pAttribMap->get_item(x,&pAttribNode);
 
                    IXMLNode pAttrib = pRet->Attributes.AllocItem();
 
                    BSTR attribName;
                    pAttribNode->get_nodeName(&attribName);
                    str = attribName;
                    pAttrib->SetName(str);
                    ::SysFreeString(attribName);
 
                    BSTR attribValue;
                    pAttribNode->get_text(&attribValue);
                    str = attribValue;
				    pAttrib->SetValue(str);
                    ::SysFreeString(attribValue);
 
                    pAttribNode->Release();
                }
 
                pAttribMap->Release();
	        }
        }
 
 
        IXMLDOMNode* pChild = 0;
	    pNode->get_firstChild(&pChild);
	    while (pChild)
	    {
            IXMLNode p = WalkNodes(pChild);
 
            if(!p)
            {
                if(pRet)
                {
                    pChild->Release();
                    break;
                }
            }
 
            if(!pRet)
                pRet = p;
            else
                pRet->AppendChild(p);
 
            IXMLDOMNode* pNext = 0;
            pChild->get_nextSibling(&pNext);
            pChild->Release();
		    pChild = pNext;
	    }
 
        return pRet;
    }
 
    IXMLNode Convert(IXMLDOMDocument* pDoc)
    {
        IXMLNode pRet;
        IXMLDOMNode* pNode = NULL;
 
        if(SUCCEEDED(pDoc->QueryInterface(IID_IXMLDOMNode,(void**)&pNode)))
        {
            pRet = WalkNodes(pNode);
 
            pNode->Release();
        }
 
        return pRet;
    }
 
    IXMLNode MSXMLConverter::Convert(const char* strXML)
    {
        IXMLNode pRet;
        IXMLDOMDocument* pDoc = Load(strXML);
        if(pDoc)
        {
            pRet = WalkNodes(pDoc);
            pDoc->Release();
            pDoc = 0;
        }
 
        return pRet;
    }
```