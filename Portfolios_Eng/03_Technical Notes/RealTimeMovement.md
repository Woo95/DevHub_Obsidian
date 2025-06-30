# **RealTimeMovement**
---
## **Overview**
Developed as part of my coursework, this project extends a provided **networking framework** with custom implementations of **client-server movement synchronization**, **input handling**, and **command send-receive systems** on both the server and client sides. The focus was on building a responsive, modular architecture suitable for **real-time multiplayer interaction**.

![[RealTimeMovement (Media)#^5a7686]]

[**View Repository**](https://github.com/Woo95/RealTimeMovement)

---
## **Client-Server Command Processing System Diagram**
![[Client-Server Command Processing System Diagram_Eng.png]]
- **White border**: process flow
- **Yellow border**: network related classes
---
## **🖧 Server-Side Core Modules**

### `1. GameLogic Class`
**Handles player session state on the server by exposing core utility methods** that are invoked by `NetworkServerProcessing`. Rather than processing messages directly, it maintains a list of active players, assigns unique seeds to each new client, and supports data lookup for runtime updates such as connection, disconnection, and movement.

**Provides server-side methods** for registering, removing, and searching players.
![[RealTimeMovement (Code)#^201a55]]

- **Key Methods**
	- [[RealTimeMovement (Code)#^fc9c2b|PlayerData Add(int clientConnectionID)]]:
		- Called when a new client connects. Assigns a unique `seed` and registers the player in the session list. Invoked by `NetworkServerProcessing` during initialization.
	- [[RealTimeMovement (Code)#^e4f3d1|PlayerData Remove(int clientConnectionID)]]:
		- Called on client disconnection. Finds and removes the player from the active session list. Used by `NetworkServerProcessing` to update the game state.
	- [[RealTimeMovement (Code)#^52ae28|PlayerData Search(int playerSeed)]]:
		- Returns a reference to a player based on their unique `seed`. Used by the server to apply updates like movement or input synchronization during gameplay.

---
### `2. NetworkServerProcessing Class`
Acts as the core **command dispatcher** on the server side. This static class **receives messages from connected clients**, **analyzes protocol signifiers**, and **dispatches appropriate logic**. It coordinates with `GameLogic` to **apply updates** such as connection, disconnections, or movement events.

**Provides centralized methods** for analyzing, routing, and distributing protocol messages.
![[RealTimeMovement (Code)#^d082db]]

- **Key Method** 
	![[RealTimeMovement (Code)#^e38f3b]]
Handles **incoming messages from clients**, analyzes signifiers, and dispatches real-time server-side commands. Closely tied to `GameLogic` for applying session and movement updates.

---
## **🎮 Client-Side Core Modules**
### `1. GameLogic Class`
**Handles player object state on the client** by exposing core gameplay handlers invoked by `NetworkClientProcessing`. Rather than interpreting messages directly, it manages spawned players, updates positions, and removes them on disconnection in response to server instructions.

**Provides client-side methods** for spawning, updating, and removing players.
![[RealTimeMovement (Code)#^ae6818]]

- **Key Methods**
	- [[RealTimeMovement (Code)#^d888d2|void SpawnMySelf(int mySeed, Vector3 position)]]:
		- Instantiates the local player at the given position and assigns a unique `seed`.
	- [[RealTimeMovement (Code)#^1f211d|void SpawnOthers(int otherSeed, Vector3 position)]]:
		- Instantiates a remote player for another client and assigns the given `seed`.
	- [[RealTimeMovement (Code)#^3cb25e|void MovePlayer(int movedPlayerSeed, Vector3 targetPos)]]:
		- Updates the position of a remote player using server-synced coordinates.  
		- **Type A**: movement updates are sent at **fixed intervals**, regardless of input state.
	- [[RealTimeMovement (Code)#^a84f81|void MovePlayer(int movedPlayerSeed, Vector3 targetPos, Vector2 inputKeys)]]:
		- Updates both the position and directional input of a remote player.  
		- **Type B**: updates only when input changes, **reducing network traffic**.
	- [[RealTimeMovement (Code)#^068ad0|void OtherPlayerLeft(int leftPlayerSeed)]]:
	    - Removes a disconnected player's object from the scene using its `seed` identifier.
---
### `2. NetworkClientProcessing Class`
Acts as the core **command handler** on the client side. This static class receives messages from the server, **analyzes protocol signifiers**, and **invokes appropriate game logic**. It works alongside `GameLogic` to instantiate, update, or remove players based on server-driven state changes.

**Provides centralized methods** for analyzing, routing, and distributing protocol messages.
![[RealTimeMovement (Code)#^ac80a7]]
- **Key Method**
  ![[RealTimeMovement (Code)#^c3ff2f]]
Handles **incoming messages from the server**, analyzes signifiers, and applies client-side updates. Closely tied to `GameLogic` for spawning, movement, and player removal.
> ⚠️ Note: Logics are omitted. Full implementation available on the [**repository**](https://github.com/Woo95/RealTimeMovement).

---
## **Conclusion**
> This project showcases a **modular**, **protocol-driven** approach to **real-time multiplayer synchronization**. With **mirrored server-client logic** and **clear message handling**, it provides a solid base for **scalable** and **responsive player movement systems**. This project reflects my understanding of **client-server architecture** and **real-time game design fundamentals**.