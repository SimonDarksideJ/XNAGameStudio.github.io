# XNA - Simple Overhead Shoot-em-up


Check out this simple overhead shoot-em-up game I wrote today!

> *Note the code is for example only.  Sample seems to be missing some game classes like Player, Ghost, Projectile, etc
> Keys style still uses XNA 3 style keyState, should use KeyState.IsKeyDown(Keys.A) for example.

Main Source File

```csharp
using System;
using System.Collections.Generic;
using Microsoft.Xna.Framework;
using Microsoft.Xna.Framework.Content;
using Microsoft.Xna.Framework.Graphics;
using Microsoft.Xna.Framework.Input;

namespace SampleOverheadShootemup
{

    public class Game1 : Microsoft.Xna.Framework.Game
    {
        GraphicsDeviceManager graphics;
        SpriteBatch spriteBatch;
        SpriteFont gameFont;


        public enum GameState
        {
            StateMainMenu,
            StatePlaying,
            StateGameOver,
            StateHelp,
        }
        GameState currentState = GameState.StateMainMenu;

        KeyboardState kbState;
        Keys[] pressedKeys;


        List<Ghost> Enemies = new List<Ghost>();
        List<Projectile> Projectiles = new List<Projectile>();
        List<Ghost> DeadGhosts = new List<Ghost>();

        Player Soldier;

        float nextShootTime = 0.01f;
        float nextEnemy = 0.850f;


        Random rand = new Random();

        Sprite MouseCursor;
        int Score = 0;


        public Game1()
        {
            graphics = new GraphicsDeviceManager(this);
            Content.RootDirectory = "Content";
        }


        protected override void LoadContent()
        {
            // Create a new SpriteBatch, which can be used to draw textures.
            spriteBatch = new SpriteBatch(GraphicsDevice);

            gameFont = Content.Load<SpriteFont>("GameFont");


            MouseCursor = new Sprite(Content.Load<Texture2D>("cursor"), 
                                        new Vector2(0, 0));

            Soldier = new Player(
                new Sprite(Content.Load<Texture2D>("soldier"),
                new Vector2(graphics.PreferredBackBufferWidth / 2,
                            graphics.PreferredBackBufferHeight - 70)));

            kbState = Keyboard.GetState();


            //cache textures in ContentManager
            Content.Load<Texture2D>("bullet");
            Content.Load<Texture2D>("ghost");
            Content.Load<Texture2D>("ghostdie1");
            Content.Load<Texture2D>("ghostdie2");
            Content.Load<Texture2D>("ghostdie3");
            Content.Load<Texture2D>("ghostdie4");
            Content.Load<Texture2D>("ghostdie5");
            Content.Load<Texture2D>("ghostdie6");
            Content.Load<Texture2D>("ghostdie7");
            Content.Load<Texture2D>("ghostdie8");
        }

     
        protected override void Update(GameTime gameTime)
        {
            // Allows the game to exit
            if (GamePad.GetState(PlayerIndex.One).Buttons.Back == 
                ButtonState.Pressed)
                this.Exit();


            // The time since Update was called last
            float elapsed = (float)gameTime.ElapsedGameTime.TotalSeconds;

            nextEnemy -= elapsed;

            if (nextEnemy <= 0)
            {
                nextEnemy = 0.25f;

                float r = rand.Next((int)(2 * 3.14159 * 1000)) / 1000.0f;
                Vector2 vPos;
                vPos.X = graphics.PreferredBackBufferWidth / 2 - 
                    (float)(Math.Cos(r) - Math.Sin(r)) * 500;
                vPos.Y = graphics.PreferredBackBufferHeight / 2 - 
                    (float)(Math.Cos(r) + Math.Sin(r)) * 500;


                Enemies.Add(
                    new Ghost(
                            new Sprite(
                                Content.Load<Texture2D>("Ghost"),
                                vPos), 
                                Vector2.Normalize(
                                    (new Vector2(Soldier.RenderElement.bounds.X, 
                                         Soldier.RenderElement.bounds.Y)) - vPos), 
                            150));


                Enemies[Enemies.Count - 1].content = Content;

                Enemies[Enemies.Count - 1].RenderElement.color = Color.White;
                if (rand.Next(100) < 5)
                {
                    Enemies[Enemies.Count - 1].RenderElement.color = Color.Red;
                    Enemies[Enemies.Count - 1].Velocity += 250;
                }
                if (rand.Next(50) < 5)
                {
                    Enemies[Enemies.Count - 1].RenderElement.color = Color.DarkRed;
                    Enemies[Enemies.Count - 1].Velocity += 150;
                }

            }

            List<Ghost> InvalidEnemies = new List<Ghost>();
            foreach (Ghost en in Enemies)
            {
                en.Update(this, rand, Soldier, elapsed, Projectiles);

                if (en.RenderElement.bounds.X < -500 ||
                    en.RenderElement.bounds.Y < -500 ||
                    en.RenderElement.bounds.X + en.RenderElement.texture.Width > 
                    graphics.PreferredBackBufferWidth + 500 ||
                    en.RenderElement.bounds.Y + en.RenderElement.texture.Height > 
                    graphics.PreferredBackBufferHeight + 500)
                {
                    InvalidEnemies.Add(en);
                }
            }

            List<Ghost> InvalidDeadEnemies = new List<Ghost>();
            foreach (Ghost en in DeadGhosts)
            {
                en.Update(this, rand, Soldier, elapsed, Projectiles);

                if ((int)en.fGhostTime - 7 > 7)
                {
                    InvalidDeadEnemies.Add(en);
                }
            }

            foreach (Ghost en in InvalidDeadEnemies)
            {
                DeadGhosts.Remove(en);
            }

            foreach (Ghost en in InvalidEnemies)
            {
                en.bDead = true;
                DeadGhosts.Add(en);
            }

            kbState = Keyboard.GetState();

            Keys[] newKeys = (Keys[])kbState.GetPressedKeys().Clone();
            
            if (pressedKeys != null)
            {
                foreach (Keys k in newKeys)
                {
                    bool bFound = false;

                    foreach (Keys k2 in pressedKeys)
                    {
                        if (k == k2)
                        {
                            bFound = true;
                            break;
                        }
                    }

                    if (!bFound)
                    {
                        OnKeyPressed(k);
                    }
                    else if (currentState == GameState.StatePlaying)
                    {
                        if (k == Keys.Space)
                        {
                            nextShootTime -= elapsed;
                            if (nextShootTime <= 0)
                                OnKeyPressed(k);
                        }
                    }
                }
            }

            pressedKeys = newKeys;



            if (currentState == GameState.StatePlaying)
            {

                MouseCursor.bounds.X = Mouse.GetState().X;
                MouseCursor.bounds.Y = Mouse.GetState().Y;

                Vector2 vObjToMouse = 
                    (new Vector2(MouseCursor.bounds.X, MouseCursor.bounds.Y)) - 
                    (new Vector2(Soldier.RenderElement.bounds.X, 
                                 Soldier.RenderElement.bounds.Y));
                if (vObjToMouse.LengthSquared() > 0)
                    vObjToMouse.Normalize();

                //todo
                Soldier.Rotation = (float)Math.Atan2(vObjToMouse.Y, 
                                        vObjToMouse.X) + 3.14150f / 2;


                Vector2 vDirection = new Vector2(0, 0);

                foreach (Keys k in newKeys)
                {
                    if (k == Keys.W)
                    {
                        vDirection += new Vector2(0, 1);
                    }
                    else if (k == Keys.S)
                    {
                        vDirection -= new Vector2(0, 1);
                    }
                    if (k == Keys.A)
                    {
                        vDirection += new Vector2(1, 0);

                    }
                    else if (k == Keys.D)
                    {
                        vDirection -= new Vector2(1, 0);
                    }
                }

                if (vDirection.LengthSquared() > 0)
                    vDirection.Normalize();

                Vector2 v =
                    new Vector2(
                        (new Vector2(Soldier.RenderElement.bounds.X,
                            Soldier.RenderElement.bounds.Y)).X - 
                            vDirection.X * elapsed * 100,
                        (new Vector2(Soldier.RenderElement.bounds.X, 
                            Soldier.RenderElement.bounds.Y)).Y - 
                            vDirection.Y * elapsed * 100);

                Soldier.RenderElement.bounds.X = (int)v.X;
                Soldier.RenderElement.bounds.Y = (int)v.Y;
            }

            List<Projectile> invalidProjectiles = new List<Projectile>();

            foreach (Projectile p in Projectiles)
            {
                for (int x = 0; x < 10; x++)
                {
                    p.Update(elapsed / 10.0f);


                    if (p.Source.GetType() == typeof(Player))
                    {
                        bool bHitEnemy = false;
                        foreach (Ghost e in Enemies)
                        {
                            if (Sprite.Intersects(p.RenderElement, 
                                e.RenderElement))
                            {
                                bHitEnemy = true;
                                e.bDead = true;
                                DeadGhosts.Add(e);

                                Score += (int)e.Velocity;

                                Enemies.Remove(e);


                                invalidProjectiles.Add(p);
                                break;
                            }
                        }
                        if (bHitEnemy)
                            continue;
                    }

                    if (p.Source.GetType() == typeof(Ghost))
                    {
                        if (Sprite.Intersects(p.RenderElement, 
                            Soldier.RenderElement))
                        {
                            currentState = GameState.StateGameOver;
                            break;
                        }
                    }



                    if (p.RenderElement.bounds.X < -50 ||
                        p.RenderElement.bounds.Y < -50 ||
                        p.RenderElement.bounds.X + 
                            p.RenderElement.texture.Width > 
                            graphics.PreferredBackBufferWidth + 50 ||
                        p.RenderElement.bounds.Y + 
                            p.RenderElement.texture.Height > 
                            graphics.PreferredBackBufferHeight + 50)
                    {
                        invalidProjectiles.Add(p);
                    }
                }

            }

            foreach (Ghost en in Enemies)
            {
                if (Sprite.Intersects(Soldier.RenderElement, en.RenderElement))
                {
                    currentState = GameState.StateGameOver;
                    break;
                }
            }

            foreach (Projectile p in invalidProjectiles)
            {
                Projectiles.Remove(p);
            }


            base.Update(gameTime);
        }

      
        protected override void Draw(GameTime gameTime)
        {
            GraphicsDevice.Clear(Color.CornflowerBlue);

            spriteBatch.Begin(SpriteSortMode.Immediate,
                                BlendState.AlphaBlend);

            switch (currentState)
            {
                case GameState.StateMainMenu:
                    {
                        spriteBatch.DrawString(gameFont,
                            "Press Enter to begin", 
                            new Vector2(graphics.PreferredBackBufferWidth / 2 - 
                                gameFont.MeasureString("Press Enter to begin").X / 2,
                                        graphics.PreferredBackBufferHeight / 2), 
                            Color.White);
                    }
                    break;
                case GameState.StatePlaying:
                    {
                        spriteBatch.DrawString(gameFont, 
                            "Score: " + Score.ToString(), 
                            new Vector2(10, 10), Color.White);

                        foreach (Ghost e in DeadGhosts)
                        {
                            e.Render(spriteBatch);
                        }

                        foreach (Ghost e in Enemies)
                        {
                            e.Render(spriteBatch);
                        }


                        foreach (Projectile p in Projectiles)
                        {
                            p.Render(spriteBatch);
                        }




                        Soldier.Render(spriteBatch);

                        MouseCursor.Render(spriteBatch);

                    }
                    break;
                case GameState.StateGameOver:
                    {
                        foreach (Ghost e in DeadGhosts)
                        {
                            e.Render(spriteBatch);
                        }

                        foreach (Ghost e in Enemies)
                        {
                            e.Render(spriteBatch);
                        }

                        foreach (Projectile p in Projectiles)
                        {
                            p.Render(spriteBatch);
                        }


                        Soldier.Render(spriteBatch);

                        spriteBatch.DrawString(gameFont, 
                            "Score: " + Score.ToString(), 
                            new Vector2(10, 10), Color.White);
                        spriteBatch.DrawString(gameFont, 
                            "Game Over", 
                            new Vector2(graphics.PreferredBackBufferWidth / 2 - 150, 
                                        graphics.PreferredBackBufferHeight / 2), 
                            Color.LightBlue);
                        spriteBatch.DrawString(gameFont, 
                            "Press Return to Continue", 
                            new Vector2(graphics.PreferredBackBufferWidth / 2 - 300, 
                                        graphics.PreferredBackBufferHeight / 2 + 50),
                            Color.LightBlue);
                    }
                    break;
            }

            spriteBatch.End();

            base.Draw(gameTime);
        }


        public void OnKeyPressed(Keys k)
        {

            if ((currentState == GameState.StateMainMenu ||
                 currentState == GameState.StateGameOver)
                && (k == Keys.Enter || k == Keys.Space))
            {
                currentState = GameState.StatePlaying;

                Enemies.Clear();
                DeadGhosts.Clear();
                Projectiles.Clear();


                Score = 0;
            }
            else if (currentState == GameState.StatePlaying)
            {
                if (k == Keys.Space && nextShootTime <= 0)
                {
                    Projectile p = new Projectile(
                        new Sprite(Content.Load<Texture2D>("Bullet"),
                                    new Vector2(Soldier.RenderElement.bounds.X,
                                                Soldier.RenderElement.bounds.Y)),
                                    Vector2.Normalize(
                                        (new Vector2(MouseCursor.bounds.X, 
                                             MouseCursor.bounds.Y)) - 
                                            (new Vector2(Soldier.RenderElement.bounds.X, 
                                              Soldier.RenderElement.bounds.Y))), 
                                    1000.0f, 
                                    Soldier);

                    Projectiles.Add(p);

                    nextShootTime = 0.1f;
                }
            }

            if (k == Keys.Escape)
            {
                this.Exit();
            }
        }
    }

}
```