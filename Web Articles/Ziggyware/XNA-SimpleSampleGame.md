# XNA - Simple Sample Game


In this sample we will look through the source code to a simple game of dropping letters.  
To survive the player must tap the lowest key to remove it from the screen. Only the last key can be collected however, with time / speed not on your side.


First off, when the game is running, we need to know what state the game is currently in. So I create an enumeration that can hold the current state of the game.
Create a new C# class and replace its contents with the following:

```csharp
    public enum GameState
    {
        StateMainMenu,
        StatePlaying,
        StateGameOver,
        StateHelp,
    }
```

Now we need somewhere to hold the information about the characters that are falling down the screen.

The class will need to hold the character's position on the screen (x,y coordinates) as well as the character it represents (A thru Z).

I also added in a color variable so the text can change colors to make it look cooler :)


```csharp
using Microsoft.Xna.Framework;

    public class FallingCharacter
    {
        public float X, Y;
        public Color color;
        public char character;
```


Inside the constructor the member variables should be set


```csharp
        public FallingCharacter(float x, float y, Color c,char character)
        {
            this.X = x;
            this.Y = y;
            this.color = c;
            this.character = character;
        }
```

Enough boilerplate, let's start building the game.  Turning to your Game1.CS class, we will start adding the game logic itself.

For this game to work we need to create some variables to store the current state of the game.  Add these to the top of the Game Class

```csharp
    public class Game1 : Game
    {
        GameState currentState = GameState.StateMainMenu; //current state of the game
        SpriteFont font;         //Text for our Character
        string WorkingDirectory; //Current Directory for debugging purposes
        KeyboardState kbState; //Holds the state of the keyboard keys
        Keys[] pressedKeys;     //An array of the currently held down keys on the keyboard
        Queue<FallingCharacter> Letters = new Queue<FallingCharacter>(); //List of falling letters

        Random rand = new Random();//Random number generator
        float nextLetterTime = 0.6f; //Number of seconds until next letter falls
        int NextSpeedup = 20; //Speed up the game every 20 correct answers
        int iSpeed = 2; //Speed at which the letters fall

        int Score=0; //The current player's score

        int GameScreenWidth = 800; //The width of the screen
        int GameScreenHeight = 600; //The Height of the screen
```

We will look into how these variables are used soon enough.

When the game starts, lets set the screen size and to keep things simple, we will use the two public properties (shown above) to control this, update the Game Initialize method as follows:


```csharp
    void Initialize()
    {
        graphics.PreferredBackBufferWidth = 800;
        graphics.PreferredBackBufferHeight = 600;
        graphics.ApplyChanges();
```

So we can display the characters on the screen as well as the player score, we are going to need a Font. So add one to your content project and load it in the LoadContent method as shown below.
> [See this article about getting started using Fonts in MonoGame](https://github.com/simondarksidej/XNAGameStudio/wiki/XNA-DrawingTextinXNA)


```csharp
    protected override void LoadContent()
    {
        font=Content.Load<SpriteFont>("myFont");
```

Lets get the keyboard state. The Keyboard class is an MonoGame class. This class has a public static function GetState() that returns the current state of the Keyboard. we could check here for errors however this was left out for brevity.  We recommend any input code should always be in its own method so you can always find it easier.  
In this case we have called it "HandleInput" (a good a name as any), so add the following new method to your game class:


```csharp
        public void HandleInput()
        {
            kbState = Keyboard.GetState();

```

The GetPressedKeys() of the keyboard state (kbState) returns an array of Keys (an enumeration of keyboard keys) for all keys pressed in the current frame/game loop. We grab this so we have a copy of the keys (which is important to this style of game)

> Normally you would just do the following to check if a key is pressed:
> ```csharp
>   if (kbState.IsKeyDown(Keys.A))
>   {
>       // Do something if A is pressed
>   }
> ```
> Which will tell us whether the A key was pressed or not.  For this game however we need all the keys at once to know what letter was pressed


```csharp
        Keys[] newKeys = (Keys[])kbState.GetPressedKeys();
```

Lets check the keys to see if any are new key presses. This can be done by comparing the new state of pressed keys with the old set of pressed keys. If the new set contains a key that is held down and the old set does not contain this key, then it was a new keypress and we should fire an event.


```csharp
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
                    OnKeyPressed(k); //handle this key press
                }
            }
        }
```

Now lets save the set of keys that are pressed so we can compare it with our results next time and see if any new keys are pressed at that time.


```csharp
        pressedKeys = newKeys;
```

Lets create the OnKeyPressed method to do somethign with any pressed keys


```csharp
    public void OnKeyPressed(Keys k)
    {
```

If the user is at the main menu or at the game over screen, lets have them press Enter when they would like to start playing.


```csharp
        if ((currentState == GameState.StateMainMenu ||
             currentState == GameState.StateGameOver)
            && k == Keys.Enter)
        {
            //Reset the state of the game

            currentState = GameState.StatePlaying;
            nextLetterTime = 0.0f;
            NextSpeedup = 20; 
            iSpeed = 1;
            Score = 0;
        }
```

If the player is currently playing the game, lets check to see if the key they pressed is the lowest letter on the screen. If it is then we can remove that letter and increase the players score :) (also prevents cheating and pressing all the keys at the same time, it won't save you!)

```csharp
        else if (currentState == GameState.StatePlaying)
        {
            string sName = Enum.GetName(typeof(Keys),k);
            if (Letters.Count > 0 &&
                String.Compare(Letters.Peek().character.ToString(), sName) == 0)
            {
                Score += 100;
                Letters.Dequeue();  //<- will get to this shortly

                NextSpeedup--;

                if (NextSpeedup <= 0)
                {
                    iSpeed++;

                    NextSpeedup = 20;
                }
            }
        }
    }
```

Finally, we need to call HandleInput each update, so that it keeps checking if we are pressing keys or not, so add the following line to the Update Method

```csharp
    protected override void Update(GameTime gameTime)
    {
        // The time since Update was called last
        float elapsed = (float)ElapsedTime.TotalSeconds;
        HandleInput();
```


Now that we have input, let's have the game actually do something. While the game is playing, Lets update the falling character's positions.

Continuing in the Update method, let's add the following: 

```csharp
        if (currentState == GameState.StatePlaying)
        {
            
            foreach (FallingCharacter fc in Letters)
            {
```

This updates the Y coordinate for characters on the screen. This number is increasing since the Y is calculated from top of the window to the bottom of the window.


```csharp
                fc.Y += (elapsed * (float)(iSpeed * 40));
```

If the character has gone into the bottom of the screen then the game is over


```csharp
                if (fc.Y + 50 > GameScreenHeight)
                {
                    currentState = GameState.StateGameOver;

                    Letters.Clear();
                    break;
                }
            }
```


While the game is playing and after a certain (small) amount of time, we need to throw in a new character to fall down the screen.


```csharp
            nextLetterTime -= elapsed;

            if (nextLetterTime <= 0 && currentState != GameState.StateGameOver)
            {
```

Lets set the time the next letter will appear. This will be based on the current speed of the game and fudged by a couple numbers to produce some good results.

```csharp
                nextLetterTime = 0.75f - (iSpeed * .03f);
```

Lets set the new character at a random place along the width of the screen, minus a fudge number so the letter does not go over the right side of the screen.

The letter should also start above the visible screen so it drops in properly without "appearing"

Lets go ahead and make the text Light Green and give it a random character

> The Letters property is using a Queue, this is a special kind of list which is a FIFO (First In First Out) operator.  
> In this game, when you add new letters and they fall down the screen, but you can only type the lowest letter. So only the last letter in the Queue can be removed first.

```csharp
                Letters.Enqueue(new FallingCharacter(
                     rand.Next(GameScreenWidth - 30), -30, 
                     Color.LightGreen, (char)((int)'A' + rand.Next(25))));

            }
        }
    }
```

Rendering the game is pretty straight forward


```csharp
    protected override void Draw(GameTime gameTime)
    {
        //Clear the screen from the last render
        GraphicsDevice.Clear(Color.CornflowerBlue);
        
        //Depending on the current state of the game, render something different
        switch (currentState)
        {
```

On the Main Menu lets display some text to the user to let them know how to start playing

> using "\n" in text will move the text to a new line

```csharp
        case GameState.StateMainMenu:
            {
                spriteBatch.DrawString(font,"Press Enter to begin\n Tap the keys as they appear and don't let them hit the bottom!",new Vector2(
                            GameScreenWidth / 4,
                            GameScreenHeight / 2), Color.White);
            }
            break;
```

As the game is playing, the score and characters must be rendered


```csharp
            case GameState.StatePlaying:
                {
                    textWriter.OutputText("Score: " + Score.ToString(), 
                                                     10, 10, Color.White);
                    //Draw each character in our Letters Queue to the screen in its current position
                    foreach (FallingCharacter fc in Letters)
                    {
                        spriteBatch.DrawString(font, fc.character.ToString(), new Vector2(fc.X, fc.Y), fc.color);
                    }
                }
                break;
```

If the game is over, lets display "Game Over" to the user.


```csharp
            case GameState.StateGameOver:
                {

                    textWriter.OutputText("Score: " + Score.ToString(), 
                                                     10, 10, Color.White);

                    textWriter.OutputText("Game Over", 
                                                    GameScreenWidth / 3, 
                                                    GameScreenHeight / 2,
                                                    Color.LightBlue);

                    textWriter.OutputText("Press Return to Continue", 
                                                     GameScreenWidth / 6, 
                                                     GameScreenHeight / 2 + 50, 
                                                     Color.LightBlue);
                }
                break;
        }
    }
```

Thats all there is to it :)

Check out this game!

Characters fall from the sky. 

You must type in the letter that is closest to the bottom of the screen.

Uses Keyboard input and the fonts explained in my [Font article](https://github.com/simondarksidej/XNAGameStudio/wiki/XNA-DrawingTextinXNA).