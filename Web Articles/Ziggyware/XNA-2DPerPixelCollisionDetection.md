# 2D Sprite Collision Detection


2D Collision detection using the alpha channels is pretty easy to do.

Lets define a function to check the collision between two Rectangles: (MonoGame does already have this function as part of the Rectangle class, it's included here so you can see it working)

If we define a Sprite having both a Position, Texture and a Rectangle Bounding box as follows:

```csharp
    public class Sprite
    {
        private Texture2D m_Texture;
        private Rectangle m_Bounds;
        private Point m_Position;

        public Texture2D Texture
        {
            get { return m_Texture; }
            set
            {
                //Set texture and reset bounding volume
                m_Texture = value;
                m_Bounds = new Rectangle(m_Position.X, m_Position.Y, value.Width, value.Height);
            }
        }

        public Rectangle Bounds
        {
            get { return m_Bounds; }
        }

        public Point Position
        {
            get { return m_Position; }
            set
            {
                //Move Sprite and update Bounding position
                m_Position = value;
                m_Bounds.X = value.X;
                m_Bounds.Y = value.Y;
            }
        }
    }
```

Then we can check if the two sprites intersect using the Rectangle Bounding volume

```csharp
public class Collision
{
    public static bool Intersects(Rectangle a, Rectangle b)
    {
        // check if two Rectangles intersect
        return (a.Right > b.Left && a.Left < b.Right && 
                a.Bottom > b.Top && a.Top < b.Bottom);
    }
}
```


Now the tricky part is checking the collision between two alpha masked textures:

```csharp
public static bool Intersects(Sprite a,Sprite b)
{
```


Lets first check their bounding Rectangles to see if they intersect:

```csharp
    if (Collision.Intersects(a.Bounds, b.Bounds))
    {
```

And grab the pixel data from the textures

```csharp
        uint[] bitsA = new uint[a.Texture.Width * a.Texture.Height];
        a.Texture.GetData<uint>(bitsA);

        uint[] bitsB = new uint[b.Texture.Width * b.Texture.Height];
        b.Texture.GetData<uint>(bitsB);
```


Now lets get the overlapping regions for each rectangle:

```csharp
        int x1 = Math.Max(a.Bounds.X, b.Bounds.X);
        int x2 = Math.Min(a.Bounds.X + a.Bounds.Width, b.Bounds.X + b.Bounds.Width);

        int y1 = Math.Max(a.Bounds.Y, b.Bounds.Y);
        int y2 = Math.Min(a.Bounds.Y + a.Bounds.Height, b.Bounds.Y + b.Bounds.Height);
```


Now that we know where the overlapping pixels are for each texture, we simply loop through the pixels and test for common pixels that are non transparent

```csharp
        for (int y = y1; y < y2; ++y)
        {
            for (int x = x1; x < x2; ++x)
            {
```


Check the alpha bytes to see if they are greater than 20 (just a fudge factor to make sure any noise in the texture doesn't affect the collision detection)

```csharp
                if (((bitsA[(x - a.Bounds.X) + 
                            (y - a.Bounds.Y) * a.Texture.Width] & 
                            0xFF000000)>>24) > 20 &&
                    ((bitsB[(x - b.Bounds.X) + 
                            (y - b.Bounds.Y) * b.Texture.Width] & 
                            0xFF000000)>>24) > 20) 
                {
                    return true;
                }
            }
        }
    }
```


At this point, there was no collision. Lets return false
```csharp
    return false;
}
```


