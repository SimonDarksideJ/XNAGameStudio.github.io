# Cryptography in C#


Here is a simple example of Rijndael Symmetric Encryption and Decryption using C#


```csharp
public class Crypto
{

#region Members

    byte[] key;
    byte[] iv;
    SymmetricAlgorithm algorithm;

    byte[] Key
    {
        get{ return key; }
        set{ key = value; }
    }

    SymmetricAlgorithm Algorithm
    {
        get{ return algorithm; }
        set{ algorithm = value; }
    }
#endregion

#region Encrypt
    public byte[] Encrypt(string str)
    {
        if(algorithm == null)
            algorithm = new RijndaelManaged();

        algorithm.GenerateKey();
        algorithm.GenerateIV();

        iv = algorithm.IV;
        key = algorithm.Key;

        MemoryStream ms = new MemoryStream();
        CryptoStream cs = new CryptoStream(ms,
                  algorithm.CreateEncryptor(),
                  CryptoStreamMode.Write);

        ASCIIEncoding ae = new ASCIIEncoding();
        byte[] plainBytes = ae.GetBytes(str);

        cs.Write(plainBytes,0,plainBytes.Length);
        cs.FlushFinalBlock();

        return ms.ToArray();
    }
#endregion

#region Decrypt
    public string Decrypt(byte[] str)
    {
        if(algorithm == null)
            algorithm = new RijndaelManaged();

        MemoryStream ms = new MemoryStream(str);
        CryptoStream cs = new CryptoStream(ms,
                      algorithm.CreateDecryptor(),
                      CryptoStreamMode.Read);
        algorithm.IV = iv;
        byte[] plainBytes = new byte[str.Length];
        cs.Read(plainBytes,0,str.Length);

        ASCIIEncoding ae = new ASCIIEncoding();
        return ae.GetString(plainBytes);
    }
#endregion

#region Test
    public void Test()
    {
        Crypto c = new Crypto();

        string str = "test";

        Console.WriteLine(str);
        byte[] b = c.Encrypt(str);

        for(int x=0;x<b.Length;x++)
            Console.Write(b[x].ToString("x2") + " ");
        Console.WriteLine("");

        Console.WriteLine( c.Decrypt(b) );
    }
#endregion

}
```