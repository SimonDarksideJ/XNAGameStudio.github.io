# Improving performance by using VertexBuffers and IndexBuffers

Your terrain is fully working, Each frame, all vertices, and indices are being sent over to our graphics card, which means that every frame we are sending over exactly the same data. Obviously, this should be optimized.

## More buffers

We want to send the data over to the graphics card only once, after which the graphics card should store it in its own superfast memory. This can be done by storing our vertices in a **VertexBuffer**, and our indices in an **IndexBuffer**.

Start by declaring these 2 variables at the top of your code in the Properties section:

```csharp
    private VertexBuffer _myVertexBuffer;
    private IndexBuffer _myIndexBuffer;
```

We will initialize and fill the VertexBuffer and IndexBuffer in a new method called **CopyToBuffers**, this code does the trick for the VertexBuffer:

```csharp
    private void CopyToBuffers()
    {
        _myVertexBuffer = new VertexBuffer(_device, VertexPositionColorNormal.VertexDeclaration, _vertices.Length, BufferUsage.WriteOnly);
        _myVertexBuffer.SetData(_vertices);
    }
```

The first line creates the **VertexBuffer**, which allocates a piece of memory on the graphics card that is large enough to store all our vertices, therefore, you need to specify how many bytes we need. This is done by specifying the number of vertices in our array, as well the VertexDeclaration (which contains the size in bytes for one vertex, as we will see in [Series 3](Riemers3DXNA3hlsloverview.md)).

The second line actually copies the data from our local vertices array into the memory on our graphics card.

We need to do the same for our indices, so add this code at the end of the method:

```csharp
    _myIndexBuffer = new IndexBuffer(_device, typeof(int), _indices.Length, BufferUsage.WriteOnly);
    _myIndexBuffer.SetData(_indices);
```

To find out how many bytes to allocate, we pass in the type of each index as well as how many of them we want to store. The second line copies the indices over to the graphics card.

## Finishing and running

Finishing up, you to call the new **CopyToBuffers** method to the end of our **LoadContent** method:

```csharp
    CopyToBuffers();
```

With that done, you only need to instruct your graphics card to fetch the vertex and index data from its own memory using the **DrawIndexedPrimitives** method instead of the *DrawUserIndexedPrimitives* method. Before we call this method, we need to let your graphics card know it should read from the buffers stored in its own memory, by specifying the references of the buffers we have initialized.  So, replace the call to **DrawUserIndexedPrimitives** with the following in the **Draw** method:

```csharp
    _device.Indices = _myIndexBuffer;
    _device.SetVertexBuffer(_myVertexBuffer);
    _device.DrawIndexedPrimitives(PrimitiveType.TriangleList, 0, 0, _indices.Length / 3);
```

This tells the graphics card where it should get its indices and vertices from to render the triangles.

Running this code will give you the same result as in the last chapter. This time, however, all vertex and index data are transferred only once to your graphics card! Much better performance, especially as you start to work with terrains with MILLIONS of vertexes/indexes.

![Summary](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA1-12TerrainLighting1.gif?raw=true)

## Exercises

You can try these exercises to practice what you have learned:

* In this code, we’re still keeping the vertices and indices arrays as global variables, only because we need to know their lengths in the DrawIndexesPrimitives call. So if you remember their lengths you can get rid of them and free some memory.
* Have a play and see if there are any other memory saving options in code and see what you can do.

## The final code

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
        private VertexBuffer _myVertexBuffer;
        private IndexBuffer _myIndexBuffer;

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
            for (short y = 0; y < _terrainHeight - 1; y++)
            {
                for (short x = 0; x < _terrainWidth - 1; x++)
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

        private void CopyToBuffers()
        {
            _myVertexBuffer = new VertexBuffer(_device, VertexPositionColorNormal.VertexDeclaration, _vertices.Length, BufferUsage.WriteOnly);
            _myVertexBuffer.SetData(_vertices);

            _myIndexBuffer = new IndexBuffer(_device, typeof(short), _indices.Length, BufferUsage.WriteOnly);
            _myIndexBuffer.SetData(_indices);
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
            CopyToBuffers();
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

                _device.Indices = _myIndexBuffer;
                _device.SetVertexBuffer(_myVertexBuffer);
                _device.DrawIndexedPrimitives(PrimitiveType.TriangleList, 0, 0, _vertices.Length, 0, _indices.Length / 3);
            }

            base.Draw(gameTime);
        }
    }
}
```
