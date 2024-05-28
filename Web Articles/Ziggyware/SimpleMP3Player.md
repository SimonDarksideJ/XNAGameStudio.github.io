# Simple MP3 Player


Playing an MP3 file with DirectX

```cpp
class ZMp3
    {
    private:
        IBaseFilter   *  pif;
        IGraphBuilder *  pigb;
        IMediaControl *  pimc;
        IMediaEventEx *  pimex;
 
        bool    ready;
 
    public:
        ZMp3();
        ~ZMp3();
 
        void LoadMp3(LPSTR filename);
        void CleanupMp3();
 
        void PlayMp3();
        void PauseMp3();
        void StopMp3();
    };
 
    ZMp3::ZMp3()
    {
        pif = NULL;
        pigb = NULL;
        pimc = NULL;
        pimex = NULL;
 
        ready = false;
    }
 
    ZMp3::~ZMp3()
    {
        CleanupMp3();
    }
 
    void ZMp3::CleanupMp3()
    {
        if (pimc)
            pimc->Stop();
 
        if(pif)
        {
            pif->Release();
            pif = NULL;
        }
 
        if(pigb)
        {
            pigb->Release();
            pigb = NULL;
        }
 
        if(pimc)
        {
            pimc->Release();
            pimc = NULL;
        }
 
        if(pimex)
        {
            pimex->Release();
            pimex = NULL;
        }
    }
 
    void ZMp3::LoadMp3(LPSTR szFile)
    {
        WCHAR wFile[MAX_PATH];
        MultiByteToWideChar(CP_ACP, 0, szFile, -1, wFile, MAX_PATH);
 
        if (SUCCEEDED(CoCreateInstance( CLSID_FilterGraph,
            NULL,
            CLSCTX_INPROC_SERVER,
            IID_IGraphBuilder,
            (void **)&this->pigb)))
        {
            pigb->QueryInterface(IID_IMediaControl, (void **)&pimc);
            pigb->QueryInterface(IID_IMediaEventEx, (void **)&pimex);
 
            if (SUCCEEDED(pigb->RenderFile(wFile, NULL)))
            {
                ready = true;
            }
        }
    }
 
    void ZMp3::PlayMp3()
    {
        if (ready)
        {
            pimc->Run();
        }
    }
 
    void ZMp3::PauseMp3()
    {
        if (ready)
        {
            pimc->Pause();
        }
    }
 
    void ZMp3::StopMp3()
    {
        if (ready)
        {
            pimc->Stop();
        }
    }
```