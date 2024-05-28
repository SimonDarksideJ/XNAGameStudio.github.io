# Point sprites – Billboarding

In this chapter, we will be adding bullets to our game. We could use real 3D spheres for these bullets, but this would be asking a lot from our graphics card when there is a better way. Instead, we will be using a very simple 2D image of a fireball and use a new technique from the **effect** file. This technique allows us to specify just the central point of an image in 3D space, and the technique will then render the image so that it is always facing the viewer and scales it to reflect the distance between the viewer and its location in 3D space.  This technique is called billboarding.

Billboarding is used a lot in 3D games, for example, to add trees in a forest.

> For a lot of detailed information and samples on billboarding, [check this article on the openGL forum](http://www.opengl-tutorial.org/intermediate-tutorials/billboards-particles/billboards/).

## Time to fire up some bullets

A **2D image** is also called a **sprite**, and since MonoGame needs only the center point of the image as a 3D location, these 2D sprites when used in 3D, are called **point sprites**.

When being fired, we want the bullets to move forward continuously, thus, for every bullet, we will need to keep track of the bullet's current position and its rotation, to calculate the direction of the bullet, just as we did with our plane. So we are going to define a new struct to help track this information, which you should put at the very top of our code, above our variables:

```csharp
    public struct Bullet
    {
        public Vector3 position;
        public Quaternion rotation;
    }
```

With this struct defined, we can create objects of the type **"Bullet"**. For each object of the type Bullet, we can store a **Vector3** (position) and a **Quaternion** (rotation).

We will also need to keep track of these Bullet objects using a **List**, furthermore, we need a variable to keep track of the last time a bullet was fired, and finally, we will need a **Texture2D** variable to hold the image of the bullet.

So put these lines among our other variable declarations:

```csharp
    private List<Bullet> _bulletList = new List<Bullet>();
    private double _lastBulletTime = 0;
    private Texture2D _bulletTexture;
```

Go ahead and import the **"bullet.jpg"** image from the asset pack into your solution as you have done before. Next, we will load the 2D image into our texture variable. 

This is done by adding this line to the **LoadContent** method:

```csharp
    _bulletTexture = Content.Load<Texture2D>("bullet");
```

Now, every time the user presses the spacebar, we want a new bullet to be created and be added to our bulletList, so add the following code at the bottom of our **ProcessKeyboardmethod**:

```csharp
    if (keys.IsKeyDown(Keys.Space))
    {
        double currentTime = gameTime.TotalGameTime.TotalMilliseconds;
        if (currentTime - _lastBulletTime > 100)
        {
            Bullet newBullet = new Bullet();
            newBullet.position = _xwingPosition;
            newBullet.rotation = _xwingRotation;
            _bulletList.Add(newBullet);

            _lastBulletTime = currentTime;
        }
    }
```

Now when the spacebar is pressed, we compare the **current game time** with the last time our xwing fired a bullet. If our last shot was more than 100 milliseconds ago, we want to fire a new bullet. This comes down to 10 bullets per second.

If it is OK to fire a new bullet, we create a new Bullet object at the current position and rotation of the xwing. This is of course because we want the bullet to travel in the same direction as our xwing was flying the moment the bullet was fired. The last line effectively adds the newly created Bullet object to the **bulletList**.

## Giving the bullet its own movement

Before we move on to the drawing code, let us finish this part by making the bullets move forward. We have already created a method that does all the calculations called **MoveForward**, so we are going to create a new method called **UpdateSpritePositions** which will scroll through our bulletList and update the position of every bullet.

Here is the code:

```csharp
    private void UpdateBulletPositions(float moveSpeed)
    {
        for (int i = 0; i < _bulletList.Count; i++)
        {
            Bullet currentBullet = _bulletList[i];
            MoveForward(ref currentBullet.position, currentBullet.rotation, moveSpeed * 2.0f);
            _bulletList[i] = currentBullet;
        }
    }
```

Pretty straightforward, the position of every bullet in our bulletList is updated, this method will also receive the same moveSpeed as our xwing, but since we multiply it with 2.0f, the bullets will go twice as fast as our xwing.

To finish off, call this method from within the **Update** method, after the collision check but before the "base.Update" call:

```csharp
    UpdateBulletPositions(moveSpeed);
```

## Time to show us the bullets

Next, we want to draw our bullets. I have provided a technique in the Effect file for this series, which requires you to only specify the center point of the bullet, and your graphics card will render the bullet at that location. However, since each bullet is a square image, our graphics card will need two triangles to render each point sprite, therefore, for each bullet, you will need to provide 6 vertices. Luckily, for each vertex you can provide the same position, and the technique will make sure these positions are adjusted correctly.

We will define a new method called **DrawSprites** which draws the bullets stored in our **bulletList**. Let us start with this part:

```csharp
    private void DrawBullets()
    {
        if (_bulletList.Count > 0)
        {
            VertexPositionTexture[] bulletVertices = new VertexPositionTexture[_bulletList.Count * 6];
            int i = 0;
            foreach (Bullet currentBullet in _bulletList)
            {
                Vector3 center = currentBullet.position;

                bulletVertices[i++] = new VertexPositionTexture(center, new Vector2(1, 1));
                bulletVertices[i++] = new VertexPositionTexture(center, new Vector2(0, 0));
                bulletVertices[i++] = new VertexPositionTexture(center, new Vector2(1, 0));

                bulletVertices[i++] = new VertexPositionTexture(center, new Vector2(1, 1));
                bulletVertices[i++] = new VertexPositionTexture(center, new Vector2(0, 1));
                bulletVertices[i++] = new VertexPositionTexture(center, new Vector2(0, 0));
            }
        }
    }
```

For each bullet in our bulletList, this code will add 6 vertices to the bulletVertices array. As you can see, for all vertices we specify the same position, the location of the bullet. The technique in the effect file will use the texture coordinates to make sure two triangles are rendered around this center position, which is required to display the image.
This technique is called **PointSprites**, so we need to select it. We will immediately set all the parameters required by the effect to work.

Add this code at the end of the if-block:

```csharp
    _effect.CurrentTechnique = _effect.Techniques["PointSprites"];
    _effect.Parameters["xWorld"].SetValue(Matrix.Identity);
    _effect.Parameters["xProjection"].SetValue(_projectionMatrix);
    _effect.Parameters["xView"].SetValue(_viewMatrix);
    _effect.Parameters["xCamPos"].SetValue(_cameraPosition);
    _effect.Parameters["xTexture"].SetValue(_bulletTexture);
    _effect.Parameters["xCamUp"].SetValue(_cameraUpDirection);
    _effect.Parameters["xPointSpriteSize"].SetValue(0.1f);
```

As for all objects rendered into a 3D world, you first need to set the World, View, and Projection matrices. Specific to the PointSprites technique, you need to pas the camera position, as well as the camera Up direction, allowing the technique to render the 2 triangles so they’re always facing the camera. Finally, you need to pass the texture and define how large you want it to be.

Obviously, before this can work, we first need to define both camera variables in the Properties section of our code:

```csharp
    private Vector3 _cameraPosition;
    private Vector3 _cameraUpDirection;
```

We have already calculated the values for these variables in our **UpdateCamera** method, so add these lines to the end of that method:

```csharp
    _cameraPosition = cameraPosition;
    _cameraUpDirection = cameraUpDirection;
```

Back to our **DrawBullets** method. With the vertices ready and the effect set up correctly, all we need to do now is render the bullets! Which is done by the following code added to the end of the if block, right after the effect parameter calls:

```csharp
    foreach (EffectPass pass in _effect.CurrentTechnique.Passes)
    {
        pass.Apply();
        _device.DrawUserPrimitives(PrimitiveType.TriangleList, bulletVertices, 0, _bulletList.Count * 2);
    }
```

In this example, the CPU is calculating the new position of each bullet for each frame, then creates an array of vertices from these positions and sends them to the graphics card for each frame. It would be much better and faster to have the GPU calculate all the new positions.

This concludes the method; All we have to do is call this method from the bottom of our **Draw** method:

```csharp
    DrawBullets();
```

Now try to run this code!

You should see a screen like the one below:

![Shooting](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA2-11Shoot1.gif?raw=true)

We will fine-tune our bullets in the next chapter. This will also include detecting collisions between a bullet and an obstacle. In case of a collision, the bullet will be removed from our bulletList, which will free the memory.

## Exercises

You can try these exercises to practice what you have learned:

* Change the size of the point sprites.

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

        public struct Bullet
        {
            public Vector3 position;
            public Quaternion rotation;
        }

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
        private Model _targetModel;
        private const int _maxTargets = 50;
        private List<Bullet> _bulletList = new List<Bullet>();
        private double _lastBulletTime = 0;
        private Texture2D _bulletTexture;

        private List<BoundingSphere> _targetList = new List<BoundingSphere>();
        private Vector3 _cameraPosition;
        private Vector3 _cameraUpDirection;

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

        private void AddTargets()
        {
            int cityWidth = _floorPlan.GetLength(0);
            int cityLength = _floorPlan.GetLength(1);

            Random random = new Random();

            while (_targetList.Count < _maxTargets)
            {
                int x = random.Next(cityWidth);
                int z = -random.Next(cityLength);
                float y = (float)random.Next(2000) / 1000f + 1;
                float radius = (float)random.Next(1000) / 1000f * 0.2f + 0.01f;

                BoundingSphere newTarget = new BoundingSphere(new Vector3(x, y, z), radius);

                if (CheckCollision(newTarget) == CollisionType.None)
                {
                    _targetList.Add(newTarget);
                }
            }
        }

        protected override void LoadContent()
        {
            _spriteBatch = new SpriteBatch(GraphicsDevice);

            // TODO: use this.Content to load your game content here
            _device = _graphics.GraphicsDevice;
            _effect = Content.Load<Effect>("effects");
            _sceneryTexture = Content.Load<Texture2D>("texturemap");
            _bulletTexture = Content.Load<Texture2D>("bullet");

            _xwingModel = LoadModel("xwing");
            _targetModel = LoadModel("target");

            SetUpCamera();
            SetUpVertices();
            SetUpBoundingBoxes();
            AddTargets();
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

            _cameraPosition = cameraPosition;
            _cameraUpDirection = cameraUpDirection;
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

            if (keys.IsKeyDown(Keys.Space))
            {
                double currentTime = gameTime.TotalGameTime.TotalMilliseconds;
                if (currentTime - _lastBulletTime > 100)
                {
                    Bullet newBullet = new Bullet();
                    newBullet.position = _xwingPosition;
                    newBullet.rotation = _xwingRotation;
                    _bulletList.Add(newBullet);

                    _lastBulletTime = currentTime;
                }
            }
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

            for (int i = 0; i < _targetList.Count; i++)
            {
                if (_targetList[i].Contains(sphere) != ContainmentType.Disjoint)
                {
                    _targetList.RemoveAt(i);
                    i--;
                    AddTargets();

                    return CollisionType.Target;
                }
            }

            return CollisionType.None;
        }

        private void UpdateBulletPositions(float moveSpeed)
        {
            for (int i = 0; i < _bulletList.Count; i++)
            {
                Bullet currentBullet = _bulletList[i];
                MoveForward(ref currentBullet.position, currentBullet.rotation, moveSpeed * 2.0f);
                _bulletList[i] = currentBullet;
            }
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

            UpdateBulletPositions(moveSpeed);

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

        private void DrawTargets()
        {
            for (int i = 0; i < _targetList.Count; i++)
            {
                Matrix worldMatrix = Matrix.CreateScale(_targetList[i].Radius) * Matrix.CreateTranslation(_targetList[i].Center);

                Matrix[] targetTransforms = new Matrix[_targetModel.Bones.Count];
                _targetModel.CopyAbsoluteBoneTransformsTo(targetTransforms);
                foreach (ModelMesh modelMesh in _targetModel.Meshes)
                {
                    foreach (Effect currentEffect in modelMesh.Effects)
                    {
                        currentEffect.CurrentTechnique = currentEffect.Techniques["Colored"];
                        currentEffect.Parameters["xWorld"].SetValue(targetTransforms[modelMesh.ParentBone.Index] * worldMatrix);
                        currentEffect.Parameters["xView"].SetValue(_viewMatrix);
                        currentEffect.Parameters["xProjection"].SetValue(_projectionMatrix);
                        currentEffect.Parameters["xEnableLighting"].SetValue(true);
                        currentEffect.Parameters["xLightDirection"].SetValue(_lightDirection);
                        currentEffect.Parameters["xAmbient"].SetValue(0.5f);
                    }

                    modelMesh.Draw();
                }
            }
        }

        private void DrawBullets()
        {
            if (_bulletList.Count > 0)
            {
                VertexPositionTexture[] bulletVertices = new VertexPositionTexture[_bulletList.Count * 6];
                int i = 0;
                foreach (Bullet currentBullet in _bulletList)
                {
                    Vector3 center = currentBullet.position;

                    bulletVertices[i++] = new VertexPositionTexture(center, new Vector2(1, 1));
                    bulletVertices[i++] = new VertexPositionTexture(center, new Vector2(0, 0));
                    bulletVertices[i++] = new VertexPositionTexture(center, new Vector2(1, 0));

                    bulletVertices[i++] = new VertexPositionTexture(center, new Vector2(1, 1));
                    bulletVertices[i++] = new VertexPositionTexture(center, new Vector2(0, 1));
                    bulletVertices[i++] = new VertexPositionTexture(center, new Vector2(0, 0));
                }

                _effect.CurrentTechnique = _effect.Techniques["PointSprites"];
                _effect.Parameters["xWorld"].SetValue(Matrix.Identity);
                _effect.Parameters["xProjection"].SetValue(_projectionMatrix);
                _effect.Parameters["xView"].SetValue(_viewMatrix);
                _effect.Parameters["xCamPos"].SetValue(_cameraPosition);
                _effect.Parameters["xTexture"].SetValue(_bulletTexture);
                _effect.Parameters["xCamUp"].SetValue(_cameraUpDirection);
                _effect.Parameters["xPointSpriteSize"].SetValue(0.1f);

                foreach (EffectPass pass in _effect.CurrentTechnique.Passes)
                {
                    pass.Apply();
                    _device.DrawUserPrimitives(PrimitiveType.TriangleList, bulletVertices, 0, _bulletList.Count * 2);
                }
            }
        }

        protected override void Draw(GameTime gameTime)
        {
            _device.Clear(ClearOptions.Target | ClearOptions.DepthBuffer, Color.DarkSlateBlue, 1.0f, 0);

            DrawCity();
            DrawModel();
            DrawTargets();
            DrawBullets();

            base.Draw(gameTime);
        }
    }
}
```

## Next Steps

[Alpha blending – Bullet collision](Riemers3DXNA2flightsim12alphablending)
