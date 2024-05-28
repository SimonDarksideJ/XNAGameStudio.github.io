# Introduction


C# is a great cross-platform language.

C# is compiled into Microsoft Intermediate Language (MISL) instructions and when the .exe or .dll is run, the MISL is compiled into the appropriate byte code for your processor.

View more information on MISL

Compiling a C# program requires the csc compiler. This comes with the .net framework.

A simple example would be the following:

> HelloWorld.cs

```csharp
using System;

public class HelloWorld
{
    public static void Main()
    {
        Console.WriteLine("Hello World");
    }
}
```

This program is compiled with the following command:
csc.exe HelloWorld.cs /out:HelloWorld.exe

You can leave out the /out parameter if the resulting exe is the same name as the source file:

This produces the same results as above
csc.exe HelloWorld.cs

