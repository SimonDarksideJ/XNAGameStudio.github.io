# Point Inside Polygon


Detecting when a point is inside a triangle is one of the base steps in a collision system.
I have listed below one implementation of such a function:

```cpp
bool ZCollision::PointInsidePoly(const Vector3& vA,const Vector3& vB,const Vector3& vC, const Vector3 &p)
    {
        int	pos = 0;
        int	neg = 0;
        const Vector3 verts[3] = {vA,vB,vC};
        Plane pl;
        pl.FromPoints(vA,vB,vC);
        unsigned int	v0 = 3 - 1;
        for (unsigned int v1 = 0; v1 < 3; v0 = v1, ++v1)
        {
            const Vector3 &p0 = verts[v0];
            const Vector3 &p1 = verts[v1];
 
            // Generate a normal for this edge
 
            Vector3 n = (p1 - p0).Cross(pl.abc);
 
            // Which side of this edge-plane is the point?
 
            float	halfPlane = (p.Dot(n)) - (p0.Dot(n));
 
            // Keep track of positives & negatives (but not zeros -- which means it's on the edge)
 
            if (halfPlane > ZCollision::Epsilon) pos++;
            else if (halfPlane < -ZCollision::Epsilon) neg++;
 
            // Early-out
 
            if (pos && neg) return false;
        }
 
        // If they're ALL positive, or ALL negative, then it's inside
 
        if (!pos || !neg) return true;
 
        // Must not be inside, because some were pos and some were neg
 
        return false;
    }
```