# Rotated Sprite Per Pixel Collision Detection


Lets look into per pixel collision detection on rotated sprites.

This can be quite a task to implement. Trust me, I've spent the entire week researching and receiving guidance from our community (namely Kukona and Dr^Nick from #XNA on Efnet).

Lets start with a basic sprite rendering class:

```csharp
public class Sprite
{
    private Vector2 pos;
    public Color color;
    public Texture2D texture;
    private float rotation = 0;
    public Vector2 RotationOrigin;
```


Now lets see what else we need to do collision detection on a sprite.

First lets examine the possibilities of solving the problem of sprite collisions.

First, we could check each pixel of both textures against each other in screen space to see if any of the non alpha pixels overlap however this would potentially have you looking at every pixel in the entire image :(

Another option would be to build an outline of the sprite and use the outlines to check for overlapping. This is a much nicer approach since you only need to store an outline of the sprite and use it for the collision :)

Since we will be using this "bounding hull" method, lets go ahead and declare some variables to help in this process.

We will get into each of these later on.

```csharp
    private bool updateHull = true;
    private List<Point> hull;
    private Dictionary<Point, bool> hullTransformed;
    private int minX, minY;
    private int maxX, maxY;

    public BoundingSphere boundingSphere;
```
Next we need some properties to properly interact with the class.
When the sprite rotates we need to transform its hull back into view space. Lets flag this to be done and get to that further down the tutorial.

```csharp
    public float Rotation
    {
        get { return rotation; }
        set { rotation = value; updateHull = true; }
    }
```


Whenever the sprite's position changes, we need to update its bounding sphere

```csharp
    public Vector2 Pos
    {
        get { return pos; }
        set { pos = value; boundingSphere.Center = new Vector3(pos,0); }
    }
```

Don't forget to dispose of your resources!

```csharp
    public void Dispose()
    {
        texture.Dispose();
    }
```


Lets create a simple sprite constructor and pre-process it for its bounding hull and sphere

```csharp
    public Sprite(Texture2D Tex, Vector2 ?vPos)
    {
        texture = Tex;
        if (vPos != null)
            Pos = (Vector2)vPos;
        color = Color.White;
        CalcHull();  //<- Will add this shortly
        
        boundingSphere.Center = new Vector3(pos,0);
        boundingSphere.Radius = 
                    (float)Math.Sqrt(texture.Width * texture.Width +
                            texture.Height * texture.Height)/2.0f;
    }
```

Our render function is a basic call to SpriteBatch.Draw

```csharp
    public void Render(SpriteBatch sb)
    {
        sb.Draw(texture, pos, null, color, Rotation, 
                    RotationOrigin, 1.0f,SpriteEffects.None, 0.5f);
    }
```


Lets look at how the collision hull is calculated:

```csharp
    public void CalcHull()
    {
```


Lets get the pixel information from the sprites texture

```csharp
    uint[] bits = new uint[texture.Width * texture.Height];
    texture.GetData<uint>(bits);
```


Now lets build a list of all the pixels at the border of an alpha pixel. 

This will be used during the collision process

```csharp
    hull = new List<Point>();

    for (int x = 0; x < texture.Width; x++)
    {
        for (int y = 0; y < texture.Height; y++)
        {
```


Lets skip any alpha pixels. These are not needed during collision

```csharp
    if ((bits[x + y * texture.Width] & 0xFF000000) >> 24 <= 20)
        continue;
```


Lets check the pixels all around this pixel to see if any of them are alpha. If so then this pixel is a border pixel and will be part of the bounding hull.

```csharp
    bool bSilouette = false;

    //check for an alpha next to a color to make sure this is an edge
    //right
    if (x < texture.Width - 1)
    {
        if ((bits[x + 1 + y * texture.Width] & 0xFF000000) >> 24 <= 20)
            bSilouette = true;
    }
    //left
    if (!bSilouette && x > 0)
    {
        if ((bits[x - 1 + y * texture.Width] & 0xFF000000) >> 24 <= 20)
            bSilouette = true;
    }
    //top
    if (!bSilouette && y > 0)
    {
        if ((bits[x + (y-1) * texture.Width] & 0xFF000000) >> 24 <= 20)
            bSilouette = true;
    }
    //bottom
    if (!bSilouette && y < texture.Height-1)
    {
        if ((bits[x + (y + 1) * texture.Width] & 0xFF000000) >> 24 <= 20)
            bSilouette = true;
    }
    //bottom left
    if (!bSilouette && x > 0 && y < texture.Height-1)
    {
        if ((bits[x - 1 + (y + 1) * texture.Width] & 0xFF000000) >> 24 <= 20)
            bSilouette = true;
    }
    //bottom right
    if (!bSilouette && x < texture.Width - 1 && y < texture.Height-1)
    {
        if ((bits[x + 1 + (y + 1) * texture.Width] & 0xFF000000) >> 24 <= 20)
            bSilouette = true;
    }
    //top left
    if (!bSilouette && x > 0 && y > 0)
    {
        if ((bits[x - 1 + (y - 1) * texture.Width] & 0xFF000000) >> 24 <= 20)
            bSilouette = true;
    }
    //top right
    if (!bSilouette && x < texture.Width && y > 0)
    {
        if ((bits[x + 1 + (y - 1) * texture.Width] & 0xFF000000) >> 24 <= 20)
            bSilouette = true;
    }
```


If any of the border pixels are alpha then add this point to the hull:

```csharp
                Point p = new Point(x, y);
                if(bSilouette && !hull.Contains(p))    
                    hull.Add(p);
            }
        }
    }
```


In order to do collision tests we will need to transform the collision hull into screen space for faster comparisons in the collision process

```csharp
    public void UpdateHull()
    {
```


Lets create a dictionary to store the transformed hull coordinates in screen space. In order to gain efficiency, this object re-uses its previous size during re-construction. This has a significant performance boost!

```csharp
    hullTransformed = new Dictionary<Point, bool>(
                hullTransformed == null ? 0 : hullTransformed.Count);
```


Lets calculate the bounding box that surrounds this transformed hull

```csharp
    minX = texture.Width;
    minY = texture.Height;
    maxX = -texture.Width;
    maxY = -texture.Height;

    int centerX = texture.Width / 2;
    int centerY = texture.Height / 2;
```


Lets pre-compute the cos and sin of the angle of rotation. This is not something you would want to do in a tight loop.

```csharp
    float cos = (float)Math.Cos(rotation);
    float sin = (float)Math.Sin(rotation);
```


Loop thru the points on the hull and transform them to the screen

```csharp
    foreach (Point p in hull)
    {
        int newX = (int)((p.X - centerX) * cos - (p.Y - centerY) * sin);
        int newY = (int)((p.Y - centerY) * cos + (p.X - centerX) * sin);
```


Update the bounding box

```csharp
    if (minX > newX) minX = newX;
    if (minY > newY) minY = newY;
    if (maxX < newX) maxX = newX;
    if (maxY < newY) maxY = newY;
```


Before we insert into the hullTransformed dictionary object, lets check to see if this new point already exists. If it does exist, we will get an error. If it doesnt, lets add it.

```csharp
        Point newP = new Point(newX, newY);
        if(!hullTransformed.ContainsKey(newP))
        {
            hullTransformed.Add(newP, true);
        }                    
        
    }
```


The hull has been updated. Lets reset the dirty flag

```csharp
        updateHull = false;
    }
```


Lets test to see if two rotated sprites overlap

```csharp
    public static bool Intersects(Sprite a, Sprite b)
    {
```


If the bounding spheres dont overlap then the pixels surely dont :)

```csharp
    if (!a.boundingSphere.Intersects(b.boundingSphere))
        return false;
```


Looks like these two sprites are close. Lets update the screen space bounding hulls if they are not already calculated since the last rotation of the object.

```csharp
    if (a.updateHull)
        a.UpdateHull();

    if (b.updateHull)
        b.UpdateHull();
```


Lets check to see if the newly calculated screen space bounding boxes overlap

```csharp
    int minX1 = a.minX + (int)a.pos.X;
    int maxX1 = a.maxX + (int)a.pos.X;
    int minY1 = a.minY + (int)a.pos.Y;
    int maxY1 = a.maxY + (int)a.pos.Y;

    int minX2 = b.minX + (int)b.pos.X;
    int maxX2 = b.maxX + (int)b.pos.X;
    int minY2 = b.minY + (int)b.pos.Y;
    int maxY2 = b.maxY + (int)b.pos.Y;

    if (maxX1 < minX2 || minX1 > maxX2 ||
        maxY1 < minY2 || minY1 > maxY2)
        return false;
```


Looks like there is an overlap. Time to check for a collision.

Since we used a dictionary object to hold the screen space hulls, lets iterate thru object A's list of hull coordinates and check to see if that same location exists in object B. If they have any matching hull coordinates then the sprites intersect.

```csharp
        Dictionary<Point,bool>.Enumerator enumer = a.hullTransformed.GetEnumerator();
        while(enumer.MoveNext())
        {
            Point point1 = enumer.Current.Key;
            int x1 = point1.X + (int)a.pos.X;
            int y1 = point1.Y + (int)a.pos.Y;

            if (x1 < minX2 || x1 > maxX2 ||
                y1 < minY2 || y1 > maxY2)
                continue;

            if (b.hullTransformed.ContainsKey(new Point(x1 - (int)b.pos.X, y1 - (int)b.pos.Y)))
                return true;
        }
        return false;
    }
}
```

