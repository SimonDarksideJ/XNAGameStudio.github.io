# High Performance Counters in C#


We need to use hi performance counters when calculating time intervals since the standard timer is only accurate up to one millisecond. The QueryPerformanceCounter function returns a much more accurate time accurate to 64 bits!


```csharp
public unsafe class HighPerformanceTimer
{
    #region DLL Imports

    [DllImport("kernel32.dll")]
    protected static extern bool QueryPerformanceCounter( ref long 
        lpPerformanceCount );
 
    [DllImport("kernel32.dll")]
    protected static extern bool QueryPerformanceFrequency( ref long 
        lpFrequency );

     #endregion
 
    #region Member Variables
    protected long  m_counterStart = 0;
    protected long  m_counterElapsed = 0;
    protected long  m_counterFrequency = 0;
    #endregion

    #region Constructor
    public HighPerformanceTimer()
    {
        QueryPerformanceCounter( ref m_counterStart );
        m_counterElapsed = m_counterStart;
        QueryPerformanceFrequency( ref m_counterFrequency );
    }
    #endregion
 
    #region Elapse (returns the elapsed time since las call to Elapsed)
    public double Elapsed
    {
        get
        {
            long currentTime = 0;
            QueryPerformanceCounter( ref currentTime );
 
            long elapsedTime = currentTime - m_counterElapsed;
            double elapsedSeconds = (double) elapsedTime / (double) m_counterFrequency;
 
            m_counterElapsed = currentTime;
            return( elapsedSeconds );
        }
    }
    #endregion

    #region Frequency (returns the value of QPF)
    public long Frequency
    {
        get
        {
            return m_counterFrequency;
        }
    }
    #endregion

   #region Now (returns the current time)
    public long Now
    {
        get
        {
            long currentTime = 0;
            QueryPerformanceCounter( ref currentTime );
            return( currentTime );
        }
    }
    #endregion

}

```