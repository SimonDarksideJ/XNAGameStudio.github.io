# Importing 3D Models from file

The city is nice, but how are we supposed to draw an airplane into our scene? Do we have to program every single vertex of this object? Luckily, we can also load models that have been generated in 3D creation programs (such as Blender or 3DS Max) and exported to a file type supported by MonoGame. In short, a Model.

## The anatomy of a model file

A Model is a file structure that holds all the necessary information to draw an object. It contains the position of the required vertices, as well as the normal data, color info, and if needed, the texture coordinates. It stores its geometrical data in Vertexbuffers and indexbuffers, which we can simply load from the file.

But that is not all, a Model can also contain multiple parts that describe one object (or several in the case of an entire scene, such as a room). Imagine we would load the model for a tank model from a file. This file could contain:

* One part that describes the hull of the tank
* Another part that describes the turret
* Another one for the door
* And two more parts for each of the caterpillars.

The Model stores all vertices in one big vertexbuffer, and each part of the model holds the indices that refer to vertices in the big vertexbuffer. The model also stores all the indexbuffers for each part of the model, one after another, in one big indexbuffer. As an example of this, you can see these large buffers at the top right of the image below.

![Model Structure](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA2-05Importing1.jpg?raw=true)

Each part of the model simply contains the data that indicates which section of the larger indexbuffer belongs to which part.

Each part also contains an effect, and if applicable, the texture image, that should be used for that part of the model. This way, you can let your graphics card draw the turret as a shiny reflective metal, and draw the caterpillars using another effect that uses a different texture.

## Importing 3D content

Enough theory, let us see how this works in practice. We will be loading a spaceship into our scene, Import the *"xwing.x"* file from the [Asset Pack](https://github.com/SimonDarksideJ/XNAGameStudio/raw/archive/Samples/Riemers/3D%20Series2%20-%20FlightSim%20-%20Assets.zip?raw=true) into your Content project for your Solution, the same way as you did with images! You should now have an asset called ‘xwing’.

Next, we will create a variable to assign the Model to, so add this to the Properties section of the code:

```csharp
    private Model _xwingModel;
```

Next, we will be adding a small method called **LoadModel** that loads the model asset and returns a reference to that model, which we will then store in our variable:

```csharp
    private Model LoadModel(string assetName)
    {
        Model newModel = Content.Load<Model>(assetName);
        return newModel;
    }
```

The method takes in the name of the asset and loads all the Model data from the file into the newly created **mod** object, the method then returns this filled Model to the calling code. For now, this method loads a Model in exactly the same way as you have been loading Textures.

This is nice but at this moment the Model default to what is referred to as the **"BasicEffect"** effect, the default 'effect' assigned by MonoGame, as shown in the image below.

![Model Properties](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA2-05Importing2.jpg?raw=true)

To apply our own effect, we have to copy our own effect into every part of the model to override how it should be drawn! This is done by simply extending our new **LoadModel** method like this:

```csharp
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
```

For each part of each mesh in our model, we store a copy of our own effect into the part. Now our model has been loaded and initialized completely!

Next, call this method from our LoadContent method to load the xwing on our Model variable:

```csharp
    _xwingModel = LoadModel("xwing");
```

## Drawing model content

We have seen quite a lot of theory above, but luckily this did not result in a lot of difficult code. With our model loaded into our xwingModel variable, we can move on to the code that renders the Model.

To keep things clean, we will create a separate **DrawModel** method:

```csharp
    private void DrawModel()
    {
        Matrix worldMatrix = Matrix.CreateScale(0.0005f, 0.0005f, 0.0005f) *
                                Matrix.CreateRotationY(MathHelper.Pi) *
                                Matrix.CreateTranslation(new Vector3(19, 12, -5));

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
            }
            mesh.Draw();
        }
    }
```

This code has some important differences with the code that renders the city. First of all, you need a World matrix that is different from the Identity matrix. This is required because when you would render the xwing immediately, you would notice 2 problems:

* The xwing would be WAY too large
* It would be rotated for 180 degrees, with its nose into the positive Z direction, while MonoGame takes the negative Z direction as Forward

These problems can easily be compensated for by rendering the xwing with a World matrix containing a scaled-down operation as well as a rotation. Have a look at the definition of the World matrix:

* The model is scaled down with a factor 0.0005, making it 2000 times smaller!
* Furthermore, the model is rotated over Pi radians (Pi radians equal 180 degrees).
* Finally, the Model is moved to position (19,12,-5) in our 3D city, just to make sure it is not inside a building.

The second difference is that you need to take the Bone matrices of the Model into account, so that the different parts of the Model are rendered at their correct positions.

The remainder of the method configures the effect to render the Model. Since the vertices of our xwing only contain Color information, we use the **Colored** technique to render it.

And let us not forget to call this method from within our **Draw** method:

```csharp
    protected override void Draw(GameTime gameTime)
    {
        device.Clear(ClearOptions.Target | ClearOptions.DepthBuffer, Color.DarkSlateBlue, 1.0f, 0);

        DrawCity();
        DrawModel();

        base.Draw(gameTime);
    }
```

This should give you the final image:

![Status](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA2-05Importing2.jpg?raw=true)

This does not look that realistic yet because of the blue background, and because we are not yet using any lighting. More on this in a few chapters!

## Exercises

You can try these exercises to practice what you have learned:

* Move the xwing to a different position.
* Set a different rotation and scaling before rendering the xwing.

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
        }

        protected override void Update(GameTime gameTime)
        {
            if (GamePad.GetState(PlayerIndex.One).Buttons.Back == ButtonState.Pressed ||
                Keyboard.GetState().IsKeyDown(Keys.Escape))
                Exit();

            // TODO: Add your update logic here

            base.Update(gameTime);
        }

        private void DrawCity()
        {
            _effect.CurrentTechnique = _effect.Techniques["Textured"];
            _effect.Parameters["xWorld"].SetValue(Matrix.Identity);
            _effect.Parameters["xView"].SetValue(_viewMatrix);
            _effect.Parameters["xProjection"].SetValue(_projectionMatrix);
            _effect.Parameters["xTexture"].SetValue(_sceneryTexture);

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
                                 Matrix.CreateTranslation(new Vector3(19, 12, -5));

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

[Adding lighting to our city](Riemers3DXNA2flightsim06ambientanddiffuse)
