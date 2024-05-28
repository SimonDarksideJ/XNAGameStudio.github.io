# Adding lighting to our city

This will be a pretty short chapter, as we have already explored the basics of lighting in the [first 3D Series](Riemers3DXNA1Terrainoverview).

## Bring out the sunlight

As the first type of lighting, we are simply going to use a directional light, much as we have used in [Series 1](Riemers3DXNA1Terrain11lighting). All the vertices of the objects in our scene already contain normal data, we added it explicitly to the vertices of our 3D city in the **SetUpVertices** method, and the .x file we loaded the Model from, already contained normal information related to the model. So all we have to do is define the direction of the light, and explain the techniques used in order to take lighting into account!

Start by defining the direction of the light by adding this variable to the top of your code:

```csharp
    private Vector3 _lightDirection = new Vector3(3, -2, 5);
```

As explained in Series 1, we need to make sure the total length of the direction of the light is equal to 1. We can do this by normalizing the direction, so put this line into the **Initialize** method:

```csharp
    _lightDirection.Normalize();
```

Now that we have defined the direction of the light, go to our **DrawCity** and **DrawModel** methods to provide the lighting information to the effects:

This line to the effect parameter configuration of the **DrawCity** method:

```csharp
    _effect.Parameters["xEnableLighting"].SetValue(true);
    _effect.Parameters["xLightDirection"].SetValue(_lightDirection);
```

And these lines to the effect parameter configuration of the **DrawModel** method:

```csharp
    currentEffect.Parameters["xEnableLighting"].SetValue(true);
    currentEffect.Parameters["xLightDirection"].SetValue(_lightDirection);
```

Now when you run this code, you should see that some of the sides of our buildings are more lit than others. However, the sides that are on the opposite side of the light do not receive any light, and are therefore completely black!

![Lighting](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA2-06Lighting1.jpg?raw=true)

## Adding Ambiance

As in normal life, you don't just get completely darkened sides of buildings just because the sun is not shining on them, so we need to also add a bit of ambient lighting to all of the pixels of our city in the **DrawCity** method:

```csharp
    _effect.Parameters["xAmbient"].SetValue(0.5f);
```

And in our **DrawModel** method:

```csharp
    currentEffect.Parameters["xAmbient"].SetValue(0.5f);
```

Now when you run this code, you should see that there are no more dark sides on our buildings. Most sides are still pretty dark, but this is only because we are looking at the shadowed side of our city. You should already see some sides on the left side of our city that is being lit by our sunlight, but really, they should appear brighter.

![Lighting](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA2-06Lighting2.jpg?raw=true)

It is entirely possible to achieve better-looking lighting effects by experimenting with other types of lights, however, I did not include these in the effects file, simply because in the 3rd series you will learn more about how to do this yourself.

## Exercises

You can try these exercises to practice what you have learned:

* Change the intensity of the ambient lighting.
* Change the direction of the sunlight.
* Adjust the position of the camera to get a feeling of the impact of a directional light on your city.
* (advanced) if you're feeling lucky, also try adding keyboard input to manipulate the lighting values so you can see the changes in real-time.

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
        private Vector3 _lightDirection = new Vector3(3, -2, 5);

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

[Using quaternions for rotations](Riemers3DXNA2flightsim07quaternioncamera)
