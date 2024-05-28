# Network Prediction

|Area|Submitted|Type|
|-|-|-|
Networking & Web Services|12/27/2007|Code Sample
||||

## Description

This sample shows how to use prediction and smoothing algorithms to compensate for network lag, making remotely controlled objects appear to move smoothly even when there is a significant delay in packets being delivered over the network.

## Sample Overview

The [Network Architecture: Client/Server](https://github.com/simondarksidej/XNAGameStudio/wiki/Network_Architecture_Client-Server) and [Network Architecture: Peer-to-Peer](https://github.com/simondarksidej/XNAGameStudio/wiki/Network_Architecture_Peer-to-Peer) samples demonstrate two different network topologies, using an example of a tank that the player can drive around the screen. In both samples, tank data are sent over the network every frame, 60 times per second. That's a lot of data! When playing over a local network, these packets are delivered quickly enough to achieve smooth and continuous movement, but things don't work so well over the Internet. Most Internet connections don't have sufficient bandwidth to send data so often, and are slow enough that players will see delays and jerkiness in the movement of the tank.

This sample shows how to make the tank example from the [Network Architecture: Peer-to-Peer](https://github.com/simondarksidej/XNAGameStudio/wiki/Network_Architecture_Peer-to-Peer) sample work over the Internet. It uses the *NetworkSession.SimulatedLatency* and *NetworkSession.SimulatedPacketLoss* properties to artificially emulate a typical Internet connection, so you can see the effects of lag even when testing over a fast local network. It then applies prediction and smoothing algorithms to compensate for this lag, making the tanks move smoothly even though the underlying network data is far from smooth.

> All content and source code downloaded from this page are bound to the Microsoft Permissive License (Ms-PL).

[XNA_NetworkPrediction_01_small.jpg](https://github.com/SimonDarksideJ/XNAGameStudio/raw/archive/Images/XNA_NetworkPrediction_01_small.jpg?raw=true)
![XNA_NetworkPrediction_02_small.jpg](https://github.com/simondarksidej/XNAGameStudio/blob/archive/Images/XNA_NetworkPrediction_02_small.jpg?raw=true)

Download | Size | Description
---|---|---|
[NetworkPredictionSample_4_0](https://github.com/simondarksidej/XNAGameStudio/tree/archive/Samples/NetworkPredictionSample_4_0) | 0.25MB | Source code and assets for the Network Prediction Sample (XNA Game Studio 4.0).
[NetworkPredictionSample_4_0.zip](https://github.com/simondarksidej/XNAGameStudioZips/raw/zips/NetworkPredictionSample_4_0.zip) | 0.25MB | Source code and assets for the Network Prediction Sample (XNA Game Studio 4.0).
||||
