# Collision detection

As you have probably already guessed, in this chapter we are going to detect when our plane has collided with an element of the scene. 

## Bouncing off walls

For the type of collision, we will implement in this chapter, we will define 3 kinds of collisions:

* Building: When the player has crashed against a building of our 3D city
* Boundary: When the xwing is outside the city, below the ground or too high in the sky
* Target: If the xwing has crashed against one of the targets we will add in the next chapter.

The first thing to do is to add an enumeration to the top of your code, for example, above your variable declarations:

> Normally you would define enumerations in their own class for easy maintenance and reusability, but we are keeping the code contained for this tutorial series.

```csharp
    public enum CollisionType { None, Building, Boundary, Target }
```

To detect collisions, we are going to model a sphere around our xwing, an abstraction which is good enough for this purpose and simplifies the calculations needed to detect if the xwing "hits" something, and a box around each of our buildings and the arena to see if the ships "sphere" passes within them.  As shown below:

![Collision](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA2-09Collision1.jpg?raw=true)

> The sphere is the simplest and most efficient form of collision detection, as you only need to measure the distance between a colliding object (xwing) and a collision object (say a building).  Other collision volumes give more detailed collision information but can get more and more costly, with mesh collision (where we use the triangles of a model) being the most expensive.  Always with collision, it is a trade-off between the quality of a collision (knowing specifically where a collision took place) and performance, and in most cases, multiple methods are used with increasing quality of detection to reduce the calculations needed.
>
> For a deeper understanding of collisions, you can try reading this article ["Physics - Collision Detection"](https://research.ncl.ac.uk/game/mastersdegree/gametechnologies/physicstutorials/4collisiondetection/Physics%20-%20Collision%20Detection.pdf)

By modeling our xwing as a sphere and using a box for a building (since buildings are actually box-shaped), we can use the built-in functionality of MonoGame to detect for collisions.

For each building of our city, we will create a [**BoundingBox**](https://docs.monogame.net/api/Microsoft.Xna.Framework.BoundingBox.html) object. Next, if we create a [**BoundingSphere**](https://docs.monogame.net/api/Microsoft.Xna.Framework.BoundingSphere.html) object for our xwing, we can use the **Contains** method to check if they collide!

## Bounding those boxes

First, we need to create a BoundingBox for each building of our city, and then add all BoundingBoxes together in one big array to mark out the city arena.

We start by defining the following array of **BoundingBox's** for our buildings and an extra **Bounding Box** for our arena/city in the Properties section of our code:

```csharp
    private BoundingBox[] _buildingBoundingBoxes;
    private BoundingBox _completeCityBox;
```

The **completeCityBox** will be used to check whether the player has flown out of the city.

Next, add this method to your code:

```csharp
    private void SetUpBoundingBoxes()
    {
        int cityWidth = _floorPlan.GetLength(0);
        int cityLength = _floorPlan.GetLength(1);

        List<BoundingBox> bbList = new List<BoundingBox> ();
        for (int x = 0; x < cityWidth; x++)
        {
            for (int z = 0; z < cityLength; z++)
            {
                int buildingType = _floorPlan[x, z];
                if (buildingType != 0)
                {
                    int buildingHeight = _buildingHeights[buildingType];
                    Vector3[] buildingPoints = new Vector3[2];
                    buildingPoints[0] = new Vector3(x, 0, -z);
                    buildingPoints[1] = new Vector3(x + 1, buildingHeight, -z - 1);
                    BoundingBox buildingBox = BoundingBox.CreateFromPoints(buildingPoints);
                    bbList.Add(buildingBox);
                }
            }
        }
        _buildingBoundingBoxes = bbList.ToArray();
    }
```

The method defined works as follows:

* We start by retrieving the width and length of your city and creating a **List** capable of storing the collective BoundingBoxes.
* Next, we scroll through the **floorPlan** and check for the type of building for each tile of the city. If there is a building, we create an array of 2 Vector3's, one indicating the lower-back-left point of the building, and one indicating the upper-front-right point of the building. These 2 points indicate a box!
* Next, we ask MonoGame to construct a **BoundingBox** object based on these 2 points using the **BoundingBox.CreateFromPoints** method.
* Once the BoundingBox for the current building has been created, you add it to the List.
* In the end, you convert the List to an array and store this array in the **buildingBoundingBoxes** variable.

We still need to create the BoundingBox that contains the whole city, that allows us to detect whether the xwing has left the city. This is done using the same approach:

* You create an array of 2 points.
* The first point is the lower-back-left point of the city
* The other is the upper-front-right point of the city.
* From these points, you create a BoundingBox which you store in the **completeCityBox** variable.

Put this code at the end of the **SetUpBoundingBoxes** method:

```csharp
    Vector3[] boundaryPoints = new Vector3[2];
    boundaryPoints[0] = new Vector3(0, 0, 0);
    boundaryPoints[1] = new Vector3(cityWidth, 20, -cityLength);
    _completeCityBox = BoundingBox.CreateFromPoints(boundaryPoints);
```

These BoundingBoxes only need to be defined once, so call this method from the end of our **LoadContent** method:

```csharp
    SetUpBoundingBoxes();
```

## Check those corners

With our **BoundingBoxes** defined, we can now add the CheckCollision method:

```csharp
    private CollisionType CheckCollision(BoundingSphere sphere)
    {
        for (int i = 0; i < _buildingBoundingBoxes.Length; i++)
        {
            if (_buildingBoundingBoxes[i].Contains(sphere) != ContainmentType.Disjoint)
            {
                return CollisionType.Building;
            }
        }

        if (_completeCityBox.Contains(sphere) != ContainmentType.Contains)
        {
            return CollisionType.Boundary;
        }

        return CollisionType.None;
    }
```

This method expects a BoundingSphere object from the calling code, this BoundingSphere will be the sphere around our xwing.

* First, we check if there is a collision between our xwing and one of the BoundingBoxes of the building by calling the **Contains** method. If the result is not Disjoint (not touching), then there is a collision with a building, so we return **CollisionType.Building**.
* If the xwing does not collide with the buildings, we check the **Contains** method between our xwing sphere and the box surrounding our city. This time, the box should completely Contain our xwing. If not, our xwing is too low, too high or outside the city. Which means we return **CollisionType.Boundary**.
* If there is no collision between the xwing and the buildings, and if the xwing is inside the city box, then everything is OK and we return **CollisionType.None**.

Now all we need to do is create a **BoundingSphere** around our xwing, and pass this BoundingSphere to our **CheckCollision** method! So add this code to our **Update** method:

```csharp
    BoundingSphere xwingSpere = new BoundingSphere(_xwingPosition, 0.04f);
    if (CheckCollision(xwingSpere) != CollisionType.None)
    {
        _xwingPosition = new Vector3(8, 1, -3);
        _xwingRotation = Quaternion.Identity;
        _gameSpeed /= 1.1f;
    }
```

When checking if the xwing has collided in each update, we:

* First, create a **BoundingSphere** with its origin at the current position of our xwing and set its radius to 0.04f, which somewhat corresponds to the size of our Model.
* Next, we pass this **BoundingSphere** to the **CheckCollision** method. If the returned **CollisionType is not None**, we reset the position and rotation of our xwing back to the start (along with the camera following the xwing) and decrease the speed of the game as things were clearly going too fast for the player.

Now run this code, and make sure you crash as soon as possible to try out your new code!

![Collision](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA2-09Collision2.gif?raw=true)

What good is a flightsim without some targets to shoot at? Let us add some in the next chapter.

## Exercises

You can try these exercises to practice what you have learned:

* Alter the Radius of the xwing's BoundingSphere and check the results.
* See if you can merge the Building creation method with the bounding box creation and make the code more streamlined.  Not essential but good practice in refactoring (cleaning up code).  There is no spoon.

## The code so far

```csharp
using System;
using System.Collections.Generic;
using Microsoft.Xna.Framework;
using Microsoft.Xna.Framework.Graphics;
using Microsoft.Xna.Framework.Input;

namespace Series3D2
{
    public class Game1 : Game
    {
        public enum CollisionType { None, Building, Boundary, Target }

        //Properties
        private GraphicsDeviceManager _graphics;
        private SpriteBatch _spriteBatch;
        private GraphicsDevice _device;
        private Effect _effect;

        private Matrix _viewMatrix;
        private Matrix _projectionMatrix;
        private Texture2D _sceneryTexture;
        private int[,] _floorPlan;
        private VertexBuffer _cityVertexBuffer;
        private int[] _buildingHeights = new int[] { 0, 2, 2, 6, 5, 4 };
        private Model _xwingModel;
        private Vector3 _lightDirection = new Vector3(3, -2, 5);
        private Vector3 _xwingPosition = new Vector3(8, 1, -3);
        private Quaternion _xwingRotation = Quaternion.Identity;
        private float _gameSpeed = 1.0f;
        private BoundingBox[] _buildingBoundingBoxes;
        private BoundingBox _completeCityBox;

        public Game1()
        {
            _graphics = new GraphicsDeviceManager(this);
            Content.RootDirectory = "Content";
            IsMouseVisible = true;
        }

        protected override void Initialize()
        {
            // TODO: Add your initialization logic here
            _graphics.PreferredBackBufferWidth = 500;
            _graphics.PreferredBackBufferHeight = 500;
            _graphics.IsFullScreen = false;
            _graphics.ApplyChanges();
            Window.Title = "Riemer's MonoGame Tutorials -- 3D Series 2";

            LoadFloorPlan();

            _lightDirection.Normalize();

            base.Initialize();
        }

        private void SetUpCamera()
        {
            _viewMatrix = Matrix.CreateLookAt(new Vector3(20, 13, -5), new Vector3(8, 0, -7), new Vector3(0, 1, 0));
            _projectionMatrix = Matrix.CreatePerspectiveFieldOfView(MathHelper.PiOver4, _device.Viewport.AspectRatio, 0.2f, 500.0f);
        }

        private void SetUpVertices()
        {
            int differentBuildings = _buildingHeights.Length - 1;
            float imagesInTexture = 1 + differentBuildings * 2;

            int cityWidth = _floorPlan.GetLength(0);
            int cityLength = _floorPlan.GetLength(1);

            List<VertexPositionNormalTexture> verticesList = new List<VertexPositionNormalTexture>();
            for (int x = 0; x < cityWidth; x++)
            {
                for (int z = 0; z < cityLength; z++)
                {
                    int currentBuilding = _floorPlan[x, z];

                    //floor or ceiling
                    verticesList.Add(new VertexPositionNormalTexture(new Vector3(x, _buildingHeights[currentBuilding], -z), new Vector3(0, 1, 0), new Vector2(currentBuilding * 2 / imagesInTexture, 1)));
                    verticesList.Add(new VertexPositionNormalTexture(new Vector3(x, _buildingHeights[currentBuilding], -z - 1), new Vector3(0, 1, 0), new Vector2((currentBuilding * 2) / imagesInTexture, 0)));
                    verticesList.Add(new VertexPositionNormalTexture(new Vector3(x + 1, _buildingHeights[currentBuilding], -z), new Vector3(0, 1, 0), new Vector2((currentBuilding * 2 + 1) / imagesInTexture, 1)));

                    verticesList.Add(new VertexPositionNormalTexture(new Vector3(x, _buildingHeights[currentBuilding], -z - 1), new Vector3(0, 1, 0), new Vector2((currentBuilding * 2) / imagesInTexture, 0)));
                    verticesList.Add(new VertexPositionNormalTexture(new Vector3(x + 1, _buildingHeights[currentBuilding], -z - 1), new Vector3(0, 1, 0), new Vector2((currentBuilding * 2 + 1) / imagesInTexture, 0)));
                    verticesList.Add(new VertexPositionNormalTexture(new Vector3(x + 1, _buildingHeights[currentBuilding], -z), new Vector3(0, 1, 0), new Vector2((currentBuilding * 2 + 1) / imagesInTexture, 1)));

                    if (currentBuilding != 0)
                    {
                        //front wall
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x + 1, 0, -z - 1), new Vector3(0, 0, -1), new Vector2((currentBuilding * 2) / imagesInTexture, 1)));
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x, _buildingHeights[currentBuilding], -z - 1), new Vector3(0, 0, -1), new Vector2((currentBuilding * 2 - 1) / imagesInTexture, 0)));
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x, 0, -z - 1), new Vector3(0, 0, -1), new Vector2((currentBuilding * 2 - 1) / imagesInTexture, 1)));

                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x, _buildingHeights[currentBuilding], -z - 1), new Vector3(0, 0, -1), new Vector2((currentBuilding * 2 - 1) / imagesInTexture, 0)));
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x + 1, 0, -z - 1), new Vector3(0, 0, -1), new Vector2((currentBuilding * 2) / imagesInTexture, 1)));
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x + 1, _buildingHeights[currentBuilding], -z - 1), new Vector3(0, 0, -1), new Vector2((currentBuilding * 2) / imagesInTexture, 0)));

                        //back wall
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x + 1, 0, -z), new Vector3(0, 0, 1), new Vector2((currentBuilding * 2) / imagesInTexture, 1)));
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x, 0, -z), new Vector3(0, 0, 1), new Vector2((currentBuilding * 2 - 1) / imagesInTexture, 1)));
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x, _buildingHeights[currentBuilding], -z), new Vector3(0, 0, 1), new Vector2((currentBuilding * 2 - 1) / imagesInTexture, 0)));

                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x, _buildingHeights[currentBuilding], -z), new Vector3(0, 0, 1), new Vector2((currentBuilding * 2 - 1) / imagesInTexture, 0)));
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x + 1, _buildingHeights[currentBuilding], -z), new Vector3(0, 0, 1), new Vector2((currentBuilding * 2) / imagesInTexture, 0)));
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x + 1, 0, -z), new Vector3(0, 0, 1), new Vector2((currentBuilding * 2) / imagesInTexture, 1)));

                        //left wall
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x, 0, -z), new Vector3(-1, 0, 0), new Vector2((currentBuilding * 2) / imagesInTexture, 1)));
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x, 0, -z - 1), new Vector3(-1, 0, 0), new Vector2((currentBuilding * 2 - 1) / imagesInTexture, 1)));
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x, _buildingHeights[currentBuilding], -z - 1), new Vector3(-1, 0, 0), new Vector2((currentBuilding * 2 - 1) / imagesInTexture, 0)));

                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x, _buildingHeights[currentBuilding], -z - 1), new Vector3(-1, 0, 0), new Vector2((currentBuilding * 2 - 1) / imagesInTexture, 0)));
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x, _buildingHeights[currentBuilding], -z), new Vector3(-1, 0, 0), new Vector2((currentBuilding * 2) / imagesInTexture, 0)));
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x, 0, -z), new Vector3(-1, 0, 0), new Vector2((currentBuilding * 2) / imagesInTexture, 1)));

                        //right wall
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x + 1, 0, -z), new Vector3(1, 0, 0), new Vector2((currentBuilding * 2) / imagesInTexture, 1)));
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x + 1, _buildingHeights[currentBuilding], -z - 1), new Vector3(1, 0, 0), new Vector2((currentBuilding * 2 - 1) / imagesInTexture, 0)));
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x + 1, 0, -z - 1), new Vector3(1, 0, 0), new Vector2((currentBuilding * 2 - 1) / imagesInTexture, 1)));

                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x + 1, _buildingHeights[currentBuilding], -z - 1), new Vector3(1, 0, 0), new Vector2((currentBuilding * 2 - 1) / imagesInTexture, 0)));
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x + 1, 0, -z), new Vector3(1, 0, 0), new Vector2((currentBuilding * 2) / imagesInTexture, 1)));
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x + 1, _buildingHeights[currentBuilding], -z), new Vector3(1, 0, 0), new Vector2((currentBuilding * 2) / imagesInTexture, 0)));
                    }
                }
            }

            _cityVertexBuffer = new VertexBuffer(_device, VertexPositionNormalTexture.VertexDeclaration, verticesList.Count, BufferUsage.WriteOnly);
            _cityVertexBuffer.SetData<VertexPositionNormalTexture>(verticesList.ToArray());
        }

        private void LoadFloorPlan()
        {
            _floorPlan = new int[,]
            {
                {1,1,1,1,1,1,1,1,1,1,1,1,1,1,1},
                {1,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
                {1,0,0,1,1,0,0,0,1,1,0,0,1,0,1},
                {1,0,0,1,1,0,0,0,1,0,0,0,1,0,1},
                {1,0,0,0,1,1,0,1,1,0,0,0,0,0,1},
                {1,0,0,0,0,0,0,0,0,0,0,1,0,0,1},
                {1,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
                {1,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
                {1,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
                {1,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
                {1,0,1,1,0,0,0,1,0,0,0,0,0,0,1},
                {1,0,1,0,0,0,0,0,0,0,0,0,0,0,1},
                {1,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
                {1,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
                {1,0,0,0,0,1,0,0,0,0,0,0,0,0,1},
                {1,0,0,0,0,1,0,0,0,1,0,0,0,0,1},
                {1,0,1,0,0,0,0,0,0,1,0,0,0,0,1},
                {1,0,1,1,0,0,0,0,1,1,0,0,0,1,1},
                {1,0,0,0,0,0,0,0,1,1,0,0,0,1,1},
                {1,1,1,1,1,1,1,1,1,1,1,1,1,1,1},
            };

            Random random = new Random();
            int differentBuildings = _buildingHeights.Length - 1;
            for (int x = 0; x < _floorPlan.GetLength(0); x++)
            {
                for (int y = 0; y < _floorPlan.GetLength(1); y++)
                {
                    if (_floorPlan[x, y] == 1)
                    {
                        _floorPlan[x, y] = random.Next(differentBuildings) + 1;
                    }
                }
            }
        }

        private Model LoadModel(string assetName)
        {
            Model newModel = Content.Load<Model>(assetName);
            foreach (ModelMesh mesh in newModel.Meshes)
            {
                foreach (ModelMeshPart meshPart in mesh.MeshParts)
                {
                    meshPart.Effect = _effect.Clone();
                }
            }
            return newModel;
        }

        private void SetUpBoundingBoxes()
        {
            int cityWidth = _floorPlan.GetLength(0);
            int cityLength = _floorPlan.GetLength(1);

            List<BoundingBox> bbList = new List<BoundingBox>();
            for (int x = 0; x < cityWidth; x++)
            {
                for (int z = 0; z < cityLength; z++)
                {
                    int buildingType = _floorPlan[x, z];
                    if (buildingType != 0)
                    {
                        int buildingHeight = _buildingHeights[buildingType];
                        Vector3[] buildingPoints = new Vector3[2];
                        buildingPoints[0] = new Vector3(x, 0, -z);
                        buildingPoints[1] = new Vector3(x + 1, buildingHeight, -z - 1);
                        BoundingBox buildingBox = BoundingBox.CreateFromPoints(buildingPoints);
                        bbList.Add(buildingBox);
                    }
                }
            }
            _buildingBoundingBoxes = bbList.ToArray();

            Vector3[] boundaryPoints = new Vector3[2];
            boundaryPoints[0] = new Vector3(0, 0, 0);
            boundaryPoints[1] = new Vector3(cityWidth, 20, -cityLength);
            _completeCityBox = BoundingBox.CreateFromPoints(boundaryPoints);
        }

        protected override void LoadContent()
        {
            _spriteBatch = new SpriteBatch(GraphicsDevice);

            // TODO: use this.Content to load your game content here
            _device = _graphics.GraphicsDevice;
            _effect = Content.Load<Effect>("effects");
            _sceneryTexture = Content.Load<Texture2D>("texturemap");

            _xwingModel = LoadModel("xwing");

            SetUpCamera();
            SetUpVertices();
            SetUpBoundingBoxes();
        }

        private void UpdateCamera()
        {
            Vector3 cameraPosition = new Vector3(0, 0.1f, 0.6f);
            cameraPosition = Vector3.Transform(cameraPosition, Matrix.CreateFromQuaternion(_xwingRotation));
            cameraPosition += _xwingPosition;
            Vector3 cameraUpDirection = new Vector3(0, 1, 0);
            cameraUpDirection = Vector3.Transform(cameraUpDirection, Matrix.CreateFromQuaternion(_xwingRotation));

            _viewMatrix = Matrix.CreateLookAt(cameraPosition, _xwingPosition, cameraUpDirection);
            _projectionMatrix = Matrix.CreatePerspectiveFieldOfView(MathHelper.PiOver4, _device.Viewport.AspectRatio, 0.2f, 500.0f);
        }

        private void ProcessKeyboard(GameTime gameTime)
        {
            float leftRightRotation = 0;
            float upDownRotation = 0;

            float turningSpeed = (float)gameTime.ElapsedGameTime.TotalMilliseconds / 1000.0f;
            turningSpeed *= 1.6f * _gameSpeed;

            KeyboardState keys = Keyboard.GetState();

            if (keys.IsKeyDown(Keys.Right))
            {
                leftRightRotation += turningSpeed;
            }
            if (keys.IsKeyDown(Keys.Left))
            {
                leftRightRotation -= turningSpeed;
            }
            if (keys.IsKeyDown(Keys.Down))
            {
                upDownRotation += turningSpeed;
            }
            if (keys.IsKeyDown(Keys.Up))
            {
                upDownRotation -= turningSpeed;
            }

            Quaternion additionalRotation = Quaternion.CreateFromAxisAngle(new Vector3(0, 0, -1), leftRightRotation) * Quaternion.CreateFromAxisAngle(new Vector3(1, 0, 0), upDownRotation);
            _xwingRotation *= additionalRotation;
        }

        private void MoveForward(ref Vector3 position, Quaternion rotationQuat, float speed)
        {
            Vector3 addVector = Vector3.Transform(new Vector3(0, 0, -1), rotationQuat);
            position += addVector * speed;
        }

        private CollisionType CheckCollision(BoundingSphere sphere)
        {
            for (int i = 0; i < _buildingBoundingBoxes.Length; i++)
            {
                if (_buildingBoundingBoxes[i].Contains(sphere) != ContainmentType.Disjoint)
                {
                    return CollisionType.Building;
                }
            }

            if (_completeCityBox.Contains(sphere) != ContainmentType.Contains)
            {
                return CollisionType.Boundary;
            }

            return CollisionType.None;
        }

        protected override void Update(GameTime gameTime)
        {
            if (GamePad.GetState(PlayerIndex.One).Buttons.Back == ButtonState.Pressed ||
                Keyboard.GetState().IsKeyDown(Keys.Escape))
                Exit();

            // TODO: Add your update logic here
            UpdateCamera();
            ProcessKeyboard(gameTime);

            float moveSpeed = gameTime.ElapsedGameTime.Milliseconds / 500.0f * _gameSpeed;
            MoveForward(ref _xwingPosition, _xwingRotation, moveSpeed);

            BoundingSphere xwingSpere = new BoundingSphere(_xwingPosition, 0.04f);
            if (CheckCollision(xwingSpere) != CollisionType.None)
            {
                _xwingPosition = new Vector3(8, 1, -3);
                _xwingRotation = Quaternion.Identity;
                _gameSpeed /= 1.1f;
            }

            base.Update(gameTime);
        }

        private void DrawCity()
        {
            _effect.CurrentTechnique = _effect.Techniques["Textured"];
            _effect.Parameters["xWorld"].SetValue(Matrix.Identity);
            _effect.Parameters["xView"].SetValue(_viewMatrix);
            _effect.Parameters["xProjection"].SetValue(_projectionMatrix);
            _effect.Parameters["xTexture"].SetValue(_sceneryTexture);
            _effect.Parameters["xEnableLighting"].SetValue(true);
            _effect.Parameters["xLightDirection"].SetValue(_lightDirection);
            _effect.Parameters["xAmbient"].SetValue(0.5f);

            foreach (EffectPass pass in _effect.CurrentTechnique.Passes)
            {
                pass.Apply();
                _device.SetVertexBuffer(_cityVertexBuffer);
                _device.DrawPrimitives(PrimitiveType.TriangleList, 0, _cityVertexBuffer.VertexCount / 3);
            }
        }

        private void DrawModel()
        {
            Matrix worldMatrix = Matrix.CreateScale(0.0005f, 0.0005f, 0.0005f) *
                                 Matrix.CreateRotationY(MathHelper.Pi) *
                                 Matrix.CreateFromQuaternion(_xwingRotation) *
                                 Matrix.CreateTranslation(_xwingPosition);

            Matrix[] xwingTransforms = new Matrix[_xwingModel.Bones.Count];
            _xwingModel.CopyAbsoluteBoneTransformsTo(xwingTransforms);

            foreach (ModelMesh mesh in _xwingModel.Meshes)
            {
                foreach (Effect currentEffect in mesh.Effects)
                {
                    currentEffect.CurrentTechnique = currentEffect.Techniques["Colored"];
                    currentEffect.Parameters["xWorld"].SetValue(xwingTransforms[mesh.ParentBone.Index] * worldMatrix);
                    currentEffect.Parameters["xView"].SetValue(_viewMatrix);
                    currentEffect.Parameters["xProjection"].SetValue(_projectionMatrix);
                    currentEffect.Parameters["xEnableLighting"].SetValue(true);
                    currentEffect.Parameters["xLightDirection"].SetValue(_lightDirection);
                    currentEffect.Parameters["xAmbient"].SetValue(0.5f);
                }
                mesh.Draw();
            }
        }

        protected override void Draw(GameTime gameTime)
        {
            _device.Clear(ClearOptions.Target | ClearOptions.DepthBuffer, Color.DarkSlateBlue, 1.0f, 0);

            DrawCity();
            DrawModel();

            base.Draw(gameTime);
        }
    }
}
```

## Next Steps

[Adding targets](Riemers3DXNA2flightsim10addingtargets)
