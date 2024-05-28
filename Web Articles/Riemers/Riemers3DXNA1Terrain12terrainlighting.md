# Adding normals to our Terrain – Intuitive approach

Following on from the last theory chapter, we will be adding normal data to all vertices for our terrain, this will enable our graphics card to perform some lighting calculations on it. As seen in the previous chapter, we will need to add a normal to each of our vertices.

## Updating the Vertex declaration

This normal should be perpendicular to the triangle of the vertex. In cases where the vertex is shared among multiple triangles (as in our terrain), you should find the normal of all triangles that use the vertex, and store the average of those normals in the vertex.

First, we have to reload our code from the main terrain code in the ['Adding colors'](Riemers3DXNA1Terrain10colors) chapter, after which we need to add the struct (which allows normal data to be added to our vectors) to the top of our class:

```csharp
    public struct VertexPositionColorNormal
    {
        public Vector3 Position;
        public Color Color;
        public Vector3 Normal;

        public readonly static VertexDeclaration VertexDeclaration = new VertexDeclaration
        (
            new VertexElement(0, VertexElementFormat.Vector3, VertexElementUsage.Position, 0),
            new VertexElement(sizeof(float) * 3, VertexElementFormat.Color, VertexElementUsage.Color, 0),
            new VertexElement(sizeof(float) * 3 + 4, VertexElementFormat.Vector3, VertexElementUsage.Normal, 0)
        );
    }
```

As well as update our vertices declaration:

```csharp
    private VertexPositionColorNormal[] _vertices;
```

And the instantiation of this array in the **SetUpVertices** method:

```csharp
    _vertices = new VertexPositionColorNormal[_terrainWidth * _terrainHeight];
```

As well as the line that actually renders your triangles in our **Draw** method:

```csharp
    _device.DrawUserIndexedPrimitives(PrimitiveType.TriangleList, _vertices, 0, vertices.Length, _indices, 0, indices.Length / 3, VertexPositionColorNormal.VertexDeclaration);
```

## Time to calculate the normals

Now we are ready to calculate the normals, which will be done in a new method called **CalculateNormals**. We start by clearing the normals in each of your vertices:

```csharp
    private void CalculateNormals()
    {
        for (int i = 0; i < _vertices.Length; i++)
        {
            _vertices[i].Normal = new Vector3(0, 0, 0);
        }
    }
```

Next, you are going to scroll through each of the triangles, for each triangle you will calculate its normal and then add this normal to the normal each of the triangle’s three vertices, add this to the end of our new **CalculateNormals** method:

```csharp
    for (int i = 0; i < _indices.Length / 3; i++)
    {
        int index1 = _indices[i * 3];
        int index2 = _indices[i * 3 + 1];
        int index3 = _indices[i * 3 + 2];

        Vector3 side1 = _vertices[index1].Position - _vertices[index3].Position;
        Vector3 side2 = _vertices[index1].Position - _vertices[index2].Position;
        Vector3 normal = Vector3.Cross(side1, side2);

        _vertices[index1].Normal += normal;
        _vertices[index2].Normal += normal;
        _vertices[index3].Normal += normal;
    }
```

Here we look up the indices for the three vertices for a triangle, if you know 2 sides of a triangle, you can find its normal by taking the cross product of the two sides. Given any 2 vectors, their cross product gives you the vector that is perpendicular to both vectors.

We start with finding two sides of the triangle, subtract the position from one corner from the position of another, once you know the normal for the triangle, you add it to the normal of each of the three vertices.

At this point, all vertices contain huge normal vectors, while they need to be of unity length. So we normalizing all of them to ensure they are uniform, by finishing our **CalculateNormals** method with this code:

```csharp
    for (int i = 0; i < vertices.Length; i++)
    {
        _vertices[i].Normal.Normalize();
    }
```

## Finishing touches

That is it for the normals! Do not forget to call this method at the end of our **LoadContent** method:

```csharp
    CalculateNormals();
```

In your **Draw** method, tell your graphics card to use the correct effect:

```csharp
    effect.CurrentTechnique = effect.Techniques["Colored"];
```

And turn on the light, by adding the following after the other effect "Parameters" (before the Pass foreach loop):

```csharp
    Vector3 lightDirection = new Vector3(1.0f, -1.0f, -1.0f);
    lightDirection.Normalize();
    _effect.Parameters["xLightDirection"].SetValue(lightDirection);
    _effect.Parameters["xAmbient"].SetValue(0.1f);
    _effect.Parameters["xEnableLighting"].SetValue(true);
```

The last line turns on ambient lighting, so even the parts of the terrain that are not lit by the directional light, still receive some lighting. 

When you run this code, you should see the image below:

![Summary](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA1-12TerrainLighting1.gif?raw=true)

As you can see, adding correct lighting dramatically adds more 3D feeling to your scene.

## Exercises

You can try these exercises to practice what you have learned:

* Try adding keyboard input to rotate the light as well as the terrain, just make your own day/night effect.
* Play with the light strength settings, to see the effect the ambient has on your terrain.

## The code so far

```csharp
using Microsoft.Xna.Framework;
using Microsoft.Xna.Framework.Graphics;
using Microsoft.Xna.Framework.Input;

namespace Series3D1
{
    public class Game1 : Game
    {
        public struct VertexPositionColorNormal
        {
            public Vector3 Position;
            public Color Color;
            public Vector3 Normal;

            public readonly static VertexDeclaration VertexDeclaration = new VertexDeclaration
            (
                new VertexElement(0, VertexElementFormat.Vector3, VertexElementUsage.Position, 0),
                new VertexElement(sizeof(float) * 3, VertexElementFormat.Color, VertexElementUsage.Color, 0),
                new VertexElement(sizeof(float) * 3 + 4, VertexElementFormat.Vector3, VertexElementUsage.Normal, 0)
            );
        }

        //Properties
        private GraphicsDeviceManager _graphics;
        private GraphicsDevice _device;
        private Effect _effect;
        private VertexPositionColorNormal[] _vertices;
        private Matrix _viewMatrix;
        private Matrix _projectionMatrix;
        private float _angle = 0f;
        private short[] _indices;
        private int _terrainWidth = 4;
        private int _terrainHeight = 3;
        private float[,] _heightData;

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
            Window.Title = "Riemer's MonoGame Tutorials -- 3D Series 1";

            base.Initialize();
        }

        private void SetUpVertices()
        {
            float minHeight = float.MaxValue;
            float maxHeight = float.MinValue;
            for (int x = 0; x < _terrainWidth; x++)
            {
                for (int y = 0; y < _terrainHeight; y++)
                {
                    if (_heightData[x, y] < minHeight)
                        minHeight = _heightData[x, y];
                    if (_heightData[x, y] > maxHeight)
                        maxHeight = _heightData[x, y];
                }
            }

            _vertices = new VertexPositionColorNormal[_terrainWidth * _terrainHeight];
            for (int x = 0; x < _terrainWidth; x++)
            {
                for (int y = 0; y < _terrainHeight; y++)
                {
                    _vertices[x + y * _terrainWidth].Position = new Vector3(x, _heightData[x, y], -y);
                    _vertices[x + y * _terrainWidth].Position = new Vector3(x, _heightData[x, y], -y);

                    if (_heightData[x, y] < minHeight + (maxHeight - minHeight) / 4)
                    {
                        _vertices[x + y * _terrainWidth].Color = Color.Blue;
                    }
                    else if (_heightData[x, y] < minHeight + (maxHeight - minHeight) * 2 / 4)
                    {
                        _vertices[x + y * _terrainWidth].Color = Color.Green;
                    }
                    else if (_heightData[x, y] < minHeight + (maxHeight - minHeight) * 3 / 4)
                    {
                        _vertices[x + y * _terrainWidth].Color = Color.Brown;
                    }
                    else
                    {
                        _vertices[x + y * _terrainWidth].Color = Color.White;
                    }
                }
            }
        }

        private void SetUpIndices()
        {
            _indices = new short[(_terrainWidth - 1) * (_terrainHeight - 1) * 6];
            int counter = 0;
            for (int y = 0; y < _terrainHeight - 1; y++)
            {
                for (int x = 0; x < _terrainWidth - 1; x++)
                {
                    short lowerLeft = (short)(x + y * _terrainWidth);
                    short lowerRight = (short)((x + 1) + y * _terrainWidth);
                    short topLeft = (short)(x + (y + 1) * _terrainWidth);
                    short topRight = (short)((x + 1) + (y + 1) * _terrainWidth);

                    _indices[counter++] = topLeft;
                    _indices[counter++] = lowerRight;
                    _indices[counter++] = lowerLeft;

                    _indices[counter++] = topLeft;
                    _indices[counter++] = topRight;
                    _indices[counter++] = lowerRight;
                }
            }
        }

        private void CalculateNormals()
        {
            for (int i = 0; i < _vertices.Length; i++)
            {
                _vertices[i].Normal = new Vector3(0, 0, 0);
            }

            for (int i = 0; i < _indices.Length / 3; i++)
            {
                int index1 = _indices[i * 3];
                int index2 = _indices[i * 3 + 1];
                int index3 = _indices[i * 3 + 2];

                Vector3 side1 = _vertices[index1].Position - _vertices[index3].Position;
                Vector3 side2 = _vertices[index1].Position - _vertices[index2].Position;
                Vector3 normal = Vector3.Cross(side1, side2);

                _vertices[index1].Normal += normal;
                _vertices[index2].Normal += normal;
                _vertices[index3].Normal += normal;
            }
            for (int i = 0; i < _vertices.Length; i++)
            {
                _vertices[i].Normal.Normalize();
            }
        }

        private void SetUpCamera()
        {
            _viewMatrix = Matrix.CreateLookAt(new Vector3(60, 80, -80), new Vector3(0, 0, 0), new Vector3(0, 1, 0));
            _projectionMatrix = Matrix.CreatePerspectiveFieldOfView(MathHelper.PiOver4, _device.Viewport.AspectRatio, 1.0f, 300.0f);
        }

        private void LoadHeightData(Texture2D heightMap)
        {
            _terrainWidth = heightMap.Width;
            _terrainHeight = heightMap.Height;

            Color[] heightMapColors = new Color[_terrainWidth * _terrainHeight];
            heightMap.GetData(heightMapColors);

            _heightData = new float[_terrainWidth, _terrainHeight];
            for (int x = 0; x < _terrainWidth; x++)
            {
                for (int y = 0; y < _terrainHeight; y++)
                {
                    _heightData[x, y] = heightMapColors[x + y * _terrainWidth].R / 5.0f;
                }
            }
        }

        protected override void LoadContent()
        {
            // TODO: use this.Content to load your game content here
            _device = _graphics.GraphicsDevice;

            _effect = Content.Load<Effect>("effects");

            SetUpCamera();

            Texture2D heightMap = Content.Load<Texture2D>("heightmap");
            LoadHeightData(heightMap);
            SetUpVertices();
            SetUpIndices();
            CalculateNormals();
        }

        protected override void Update(GameTime gameTime)
        {
            if (GamePad.GetState(PlayerIndex.One).Buttons.Back == ButtonState.Pressed || Keyboard.GetState().IsKeyDown(Keys.Escape))
                Exit();

            // TODO: Add your update logic here
            KeyboardState keyState = Keyboard.GetState();
            if (keyState.IsKeyDown(Keys.Left))
            {
                _angle += 0.05f;
            }
            if (keyState.IsKeyDown(Keys.Right))
            {
                _angle -= 0.05f;
            }

            base.Update(gameTime);
        }

        protected override void Draw(GameTime gameTime)
        {
            _device.Clear(ClearOptions.Target | ClearOptions.DepthBuffer, Color.Black, 1.0f, 0);

            RasterizerState rs = new RasterizerState();
            rs.CullMode = CullMode.None;
            rs.FillMode = FillMode.Solid;
            _device.RasterizerState = rs;

            // TODO: Add your drawing code here
            Matrix worldMatrix = Matrix.CreateTranslation(-_terrainWidth / 2.0f, 0, _terrainHeight / 2.0f) * Matrix.CreateRotationY(_angle);
            _effect.CurrentTechnique = _effect.Techniques["Colored"];
            _effect.Parameters["xView"].SetValue(_viewMatrix);
            _effect.Parameters["xProjection"].SetValue(_projectionMatrix);
            _effect.Parameters["xWorld"].SetValue(worldMatrix);

            Vector3 lightDirection = new Vector3(1.0f, -1.0f, -1.0f);
            lightDirection.Normalize();
            _effect.Parameters["xLightDirection"].SetValue(lightDirection);
            _effect.Parameters["xAmbient"].SetValue(0.1f);
            _effect.Parameters["xEnableLighting"].SetValue(true);

            foreach (EffectPass pass in _effect.CurrentTechnique.Passes)
            {
                pass.Apply();
                _device.DrawUserIndexedPrimitives(PrimitiveType.TriangleList, _vertices, 0, _vertices.Length, _indices, 0, _indices.Length / 3, VertexPositionColorNormal.VertexDeclaration);
            }

            base.Draw(gameTime);
        }
    }
}
```

## Next Steps

[Improving performance by using VertexBuffers and IndexBuffers](Riemers3DXNA1Terrain13buffers)
