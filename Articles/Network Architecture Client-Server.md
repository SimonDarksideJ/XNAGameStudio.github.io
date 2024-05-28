# Network Architecture: Client/Server

|Area|Submitted|Type|
|-|-|-|
Networking & Web Services|12/17/2007|Code Sample
||||

## Description

This sample implements a simple multiplayer network session by using a client/server network topology. In this topology, each computer sends its player input to the server, which then updates the state of everyone in the session and sends the results back to the clients.

## Sample Overview

This sample shows how to create and join network sessions, and how to exchange data by using a client/server network topology. Each player controls a tank that they can drive around the screen while rotating the turret. The game simulation is run entirely on a single computer that has been designated as the server. Client computers send their player inputs to the server, which then sends the resulting game state back to each client. Game logic only ever runs on the server: the clients are effectively functioning as dumb terminals, responsible only for reading input and rendering the game world as described by the server.

For an alternative way to handle networked game simulation, see the [Peer-to-Peer](https://github.com/simondarksidej/XNAGameStudio/wiki/Network_Architecture_Peer-to-Peer) sample, which implements the same tank movements by using a peer-to-peer network topology.

> All content and source code downloaded from this page are bound to the Microsoft Permissive License (Ms-PL).

![XNA_ClientServer_01_small.jpg](https://github.com/simondarksidej/XNAGameStudio/blob/archive/Images/XNA_ClientServer_01_small.jpg?raw=true)
![XNA_ClientServer_02_small.jpg](https://github.com/simondarksidej/XNAGameStudio/blob/archive/Images/XNA_ClientServer_02_small.jpg?raw=true)

Download | Size | Description
---|---|---|
[ClientServerSample_4_0](https://github.com/simondarksidej/XNAGameStudio/tree/archive/Samples/ClientServerSample_4_0) | 0.10MB | Source code and assets for the Network Architecture: Client/Server Sample (XNA Game Studio 4.0).
[ClientServerSample_4_0.zip](https://github.com/simondarksidej/XNAGameStudioZips/raw/zips/ClientServerSample_4_0.zip) | 0.10MB | Source code and assets for the Network Architecture: Client/Server Sample (XNA Game Studio 4.0).
||||
