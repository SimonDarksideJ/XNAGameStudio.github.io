# A* Path Finding in C#


Here is a port of my path finding algorithm from C++ to C# and XNA

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using Microsoft.Xna.Framework;
using Microsoft.Xna.Framework.Audio;
using Microsoft.Xna.Framework.Content;
using Microsoft.Xna.Framework.GamerServices;
using Microsoft.Xna.Framework.Graphics;
using Microsoft.Xna.Framework.Input;
using Microsoft.Xna.Framework.Media;
using Microsoft.Xna.Framework.Net;
using Microsoft.Xna.Framework.Storage;

namespace Ziggyware
{
    class PathFinding
    {
        
        //takes three points on a triangle where point 2 is the center point
        //and creates a new point in the direction the 'arrow' of points makes
        public static Vector3 GetTriPoint(Vector3 p1, Vector3 p2, Vector3 p3)
        {
            //vector from p2 to p1
            Vector3 p2p1, p3p1;
            Vector3 temp;

            Vector3.Subtract(ref p1, ref p2, out p2p1);
            Vector3.Normalize(ref p2p1, out temp);
            p2p1 = temp;

            //vector from p3 to p1
            Vector3.Subtract(ref p1, ref p3, out p3p1);

            Vector3.Normalize(ref p3p1, out temp);
            p3p1 = temp;

            //add the vectors together to produce new vector
            Vector3.Add(ref p2p1, ref p3p1, out temp);

            //normalize the vector to reduce the length to 1 unit
            Vector3.Normalize(ref temp, out temp);

            // scale the vector... no good if its < 1. : )
            Vector3.Multiply(ref temp, 5.0f, out temp);

            //add the new vector to p1 (offset it)
            Vector3.Add(ref temp, ref p1, out temp);

            return temp;
        }


        //Taken from comp.graphics.algorithms FAQ
        public static bool IntersectionOfTwoLines(
                                Vector3 a, Vector3 b, Vector3 c,
                                Vector3 d, out Vector3 result)
        {
            result = Vector3.Zero;

            float r, s;

            float denominator = (b.X - a.X) * (d.Y - c.Y) - (b.Y - a.Y) * (d.X - c.X);

            // If the denominator in above is zero, AB & CD are colinear
            if (denominator == 0)
                return false;

            float numeratorR = (a.Y - c.Y) * (d.X - c.X) - (a.X - c.X) * (d.Y - c.Y);
            //  If the numerator above is also zero, AB & CD are collinear.
            //  If they are collinear, then the segments may be projected to the x- 
            //  or y-axis, and overlap of the projected intervals checked.

            r = numeratorR / denominator;

            float numeratorS = (a.Y - c.Y) * (b.X - a.X) - (a.X - c.X) * (b.Y - a.Y);

            s = numeratorS / denominator;

            //  If 0 < = r < = 1 & 0 < = s < = 1, intersection exists
            //  r < 0 or r > 1 or s < 0 or s >1 line segments do not intersect
            if (r < 0 || r > 1 || s < 0 || s > 1)
                return false;

            
            //    Note:
            //    If the intersection point of the 2 lines are needed (lines in this
            //    context mean infinite lines) regardless whether the two line
            //    segments intersect, then
            //
            //    If r > 1, P is located on extension of AB
            //    If r < 0, P is located on extension of BA
            //    If s > 1, P is located on extension of CD
            //    If s < 0, P is located on extension of DC
            

            // Find intersection point
            result.X = a.X + (r * (b.X - a.X));
            result.Y = a.Y + (r * (b.Y - a.Y));

            return true;
        }
        

        public class Obstacle
        {
            public List<Vector3> Points = new List<Vector3>();
        }




        class BeenTo
        {
            public int BeenToObject;
            public List<int> BeenToVertex = new List<int>();
        }

        public class PathFinder
        {
            public List<Obstacle> ObstacleList = new List<Obstacle>();

            public Vector3 Direction;
            public Vector3 Location;
            public float Speed;

            public bool FoundPath;

            public int NextPointIndex;
            public Vector3 NextPoint;

            List<BeenTo> BeenToList = new List<BeenTo>();
            public List<Vector3> Path = new List<Vector3>();

            public bool UseOptimizedPath;
            public int ClosestObjectIndex;
            public int ClosestVertexIndex;

            public float Dist;

            public Vector3 StartPoint = Vector3.Zero;
            public Vector3 EndPoint = Vector3.One;

            public PathFinder()
            {
                UseOptimizedPath = true;

                ClosestObjectIndex = 0;
                ClosestVertexIndex = 0;

                Dist = 0.0f;

                FoundPath = false;
                NextPointIndex = 0;
                Speed = 1.0f;
                SetStartPoint();
            }

            public bool HaveBeenToObjPoint(int objIndex, int vertexIndex)
            {
                bool SkipVertex = false;
                //make shure we havent already tried this point
                for (int i = 0; i < BeenToList.Count; i++)
                {
                    BeenTo bt = BeenToList[i];

                    if (bt.BeenToObject == objIndex)
                    {
                        for (int x = 0; x < bt.BeenToVertex.Count; x++)
                        {
                            if (bt.BeenToVertex[x] == vertexIndex)
                            {
                                SkipVertex = true;
                                break;
                            }
                        }
                    }
                }

                return SkipVertex;
            }

            public bool GetClosestVisiblePointToGoal(Vector3 fromPoint)
            {
                float fClosest = float.MinValue;

                bool didfind = false;

                for (int i = 0; i < ObstacleList.Count; i++)
                {

                    if (ObstacleList[i].Points.Count > 2)
                        for (int j = 0; j < ObstacleList[i].Points.Count; j++)
                        {
                            if (HaveBeenToObjPoint(i, j) == true)
                            {
                                continue;
                            }

                            Obstacle o = ObstacleList[i];

                            // get prev point
                            Vector3 temp = 
                                o.Points[j == 0 ? o.Points.Count - 1 : j - 1];
                            
                            //get next point
                            Vector3 temp2 = 
                                o.Points[j == o.Points.Count - 1 ? 0 : j + 1];

                            Vector3 v = GetTriPoint(o.Points[j], temp, temp2);

                            if (CheckObstacles(fromPoint, v) == false)
                            {

                                Dist = Vector3.Distance(o.Points[j], EndPoint);

                                if (Dist < fClosest)
                                {
                                    NextPointIndex = 1;
                                    ClosestObjectIndex = i;
                                    ClosestVertexIndex = j;
                                    fClosest = Dist;
                                    didfind = true;

                                    if (CheckObstacles(v, EndPoint) == false)
                                    {
                                        NextPoint = 
                                            ObstacleList[ClosestObjectIndex].
                                                            Points[ClosestVertexIndex];

                                        return false;
                                    }
                                }
                            }
                        }
                }
                if (didfind)
                {
                    NextPoint = 
                        ObstacleList[ClosestObjectIndex].Points[ClosestVertexIndex];
                        
                    BeenTo bt = new BeenTo();
                    bt.BeenToObject = ClosestObjectIndex;
                    bt.BeenToVertex.Add(ClosestVertexIndex);
                    BeenToList.Add(bt);
                }
                return didfind;
            }

            public void ClearPath()
            {
                FoundPath = false;
                Path.Clear();
                BeenToList.Clear();
            }

            public void DrawPath()
            {
                //not yet implemented, here is how i did it in GDI
                
                //POINT lastPoint;
                //if(Path.Count)
                //for(int i=0;i<Path.Count-1;i++)
                //{
                //   SelectObject(hdc,GetRedPen());
                //
                //   MoveToEx(hdc,Path[i].x,Path[i].y,&lastPoint);
                //   LineTo(hdc,Path[i+1].x,Path[i+1].y);
                //}
            }


            public void GetPath()
            {
                if (FoundPath == false)
                {
                    ClearPath();
                    if (CheckObstacles(Location, EndPoint) == false)
                    {
                        FoundPath = true;
                        Path.Add(Location);
                        Path.Add(EndPoint);
                        NextPointIndex = Path.Count - 1;
                    }
                    else
                    {
                        Path.Add(Location);
                        NextPoint = Location;

                        while (true)
                        {
                            while (GetClosestVisiblePointToGoal(NextPoint))
                            {
                                Obstacle closest = ObstacleList[ClosestObjectIndex];

                                // get prev point
                                Vector3 temp = 
                                        closest.Points[ClosestObjectIndex == 0 ? 
                                                            closest.Points.Count - 1 : 
                                                            ClosestVertexIndex - 1];
                                                            
                                //get next point
                                Vector3 temp2 = 
                                        closest.Points[ClosestObjectIndex == 
                                                            closest.Points.Count - 1 ? 
                                                            0 : 
                                                            ClosestVertexIndex + 1];

                                Vector3 v = GetTriPoint(NextPoint, temp, temp2);

                                Path.Add(v);
                                NextPoint = Path[Path.Count - 1];
                            }

                            if (Path.Count > 1)
                            {

                                if (ObstacleList.Count > 0)
                                    if (ObstacleList[0].Points.Count > 2)
                                    {
                                        Obstacle clostestObstacle = 
                                            ObstacleList[ClosestObjectIndex];
                                            
                                        // get prev point
                                        Vector3 temp = 
                                            clostestObstacle.Points[C
                                                losestVertexIndex == 0 ? 
                                                    clostestObstacle.Points.Count - 1 : 
                                                    ClosestVertexIndex - 1];
                                                    
                                        //get next point
                                        Vector3 temp2 = 
                                            clostestObstacle.Points[
                                                ClosestVertexIndex == 
                                                    clostestObstacle.Points.Count - 1 ? 
                                                        0 : 
                                                        ClosestVertexIndex + 1];

                                        Vector3 v = GetTriPoint(NextPoint, temp, temp2);

                                        Path.Add(v);
                                        NextPoint = Path[Path.Count - 1];
                                    }


                                if (CheckObstacles(Path[Path.Count - 1], EndPoint) == false)
                                {
                                    Path.RemoveAt(Path.Count - 1);
                                    break;
                                }
                                else
                                {
                                    Path.RemoveAt(Path.Count - 1);
                                    Path.RemoveAt(Path.Count - 1);
                                    NextPoint = Path[Path.Count - 1];
                                }
                            }
                            else
                            {
                                break;
                            }


                        }

                        DoCleanPath();


                        if (ObstacleList.Count > 0)
                            if (ObstacleList[0].Points.Count > 2)
                            {
                                Obstacle obstacle = ObstacleList[ClosestObjectIndex];
                                // get prev point
                                Vector3 temp = obstacle.Points[
                                        ClosestVertexIndex == 0 ? 
                                        obstacle.Points.Count - 1 : 
                                        ClosestVertexIndex - 1];
                                        
                                //get next point
                                Vector3 temp2 = obstacle.Points[
                                        ClosestVertexIndex == obstacle.Points.Count - 1 ? 
                                        0 : 
                                        ClosestVertexIndex + 1];

                                Vector3 v = GetTriPoint(NextPoint, temp, temp2);
                                Path.Add(v);

                                NextPoint = Path[Path.Count - 1];
                            }

                        Path.Add(EndPoint);

                        FoundPath = true;
                    }
                }
            }

            public void DoCleanPath()
            {
                if (UseOptimizedPath)
                    while (!CleanPath()) { }
            }

            public bool CleanPath()
            {
                int foundAt = -1;
                float foundAtDist = float.MinValue;

                //make shure Path is the best possible
                for (int i = 0; i < Path.Count - 1; i++)
                {
                    for (int j = i + 1; j < Path.Count; j++)
                    {
                        if (CheckObstacles(Path[i], Path[j]) == false)
                        {
                            if (j > i + 1)
                            {
                                float temp = Vector3.Distance(Path[j], EndPoint);
                                if (temp < foundAtDist)
                                {
                                    foundAt = j;
                                    foundAtDist = temp;
                                }
                            }
                        }
                    }
                    if (foundAt != -1)
                    {
                        for (int k = foundAt - 1; k > i; k--)
                        {
                            Path.RemoveAt(k);
                        }
                        return false;
                    }
                }
                return true;
            }


            public void UpdatePosition(bool RunPath)
            {
                if (RunPath)
                {
                    if (FoundPath == true)
                    {

                        if (Location != Path[NextPointIndex])
                        {
                            Direction = Path[NextPointIndex] - StartPoint;
                            Direction.Normalize();

                            if (Vector3.Distance(Location, 
                                                 Path[NextPointIndex]) <= Speed)
                            {
                                Location = Path[NextPointIndex];
                                NextPointIndex++;

                            }
                            else
                            {
                                Location += Direction * Speed;
                            }
                        }
                        else
                        {
                            Location = Path[NextPointIndex];
                            NextPointIndex++;
                        }

                    }
                    if (NextPointIndex == Path.Count)
                        RunPath = false;

                    StartPoint = Location;

                }
            }

            public void SetStartPoint()
            {
                Location = StartPoint;
                Direction = EndPoint - StartPoint;
                Direction.Normalize();
            }

            public bool CheckObstacles(Vector3 point, Vector3 dest)
            {
                for (int j = 0; j < ObstacleList.Count; j++)
                {
                    Obstacle ob = ObstacleList[j];

                    for (int i = 0; i < ob.Points.Count; i++)
                    {
                        Vector3 a = ob.Points[i];
                        Vector3 b = ob.Points[i == ob.Points.Count - 1 ? 0 : i + 1];
                        Vector3 result;
                        if (IntersectionOfTwoLines(a, b, point, dest, out result))
                        {
                            return true;
                        }
                    }
                }
                return false;
            }
        }

        public class PathFinderManager
        {
            public List<PathFinder> PathFinders = new List<PathFinder>();
        }
    }
}
```