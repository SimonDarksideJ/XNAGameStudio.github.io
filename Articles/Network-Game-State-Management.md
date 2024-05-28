# Network Game State Management

|Area|Submitted|Type|
|-|-|-|
Games: Gameplay, Networking & Web Services|12/17/2007|Code Sample
||||

## Description

This sample shows how to implement the user interface for a multiplayer networked game. It provide menus for creating, finding, and joining sessions, a lobby screen, and robust network error handling.

## Sample Overview

This sample builds on the Game State Management sample, adding the user interface screens needed by a multiplayer networked game. If you're not familiar with the underlying concepts of the ScreenManager and GameScreen classes, you should start out by reading the documentation for the [Game State Management sample](https://github.com/simondarksidej/XNAGameStudio/wiki/Game-State-Management-(Mango,-C%23VB)).

At the main menu, players can choose between Single Player, LIVE, or System Link game modes. If they choose a networked mode, they'll be prompted to sign in a suitable player profile (if one isn't already signed in), and then asked whether to create a new session or search for existing sessions. The sample displays an animated busy indicator whenever a network operation is in progress, and robustly handles errors by catching network exceptions and turning them into message box popups. Once in the lobby, a list of gamers is displayed along with icons indicating who is currently talking and who has marked themselves as ready. When all the gamers are ready, the sample loads the gameplay screen, at which point the rest is up to you: there is no actual game code included here!

All content and source code downloaded from this page are bound to the Microsoft Permissive License (Ms-PL).

![XNA_NetGameStateManagement_01_small.jpg](https://github.com/simondarksidej/XNAGameStudio/blob/archive/Images/XNA_NetGameStateManagement_01_small.jpg?raw=true)

Download | Size | Description
---|---|---|
[NGSMSample_4_0](https://github.com/simondarksidej/XNAGameStudio/tree/archive/Samples/NGSMSample_4_0) | 0.23MB | Source code and assets for the Network Game State Management Sample (XNA Game Studio 4.0).
[NGSMSample_4_0.zip](https://github.com/simondarksidej/XNAGameStudioZips/raw/zips/NGSMSample_4_0.zip) | 0.23MB | Source code and assets for the Network Game State Management Sample (XNA Game Studio 4.0).
||||
