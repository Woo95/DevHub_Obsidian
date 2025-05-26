# RealTimeMovement Block References
---
## GIF
![[RealTimeMovement Multiplayer.gif|500]] ^5a7686

## Code

### Server Side
#### GameLogic.cs
```csharp
public class GameLogic : MonoBehaviour
{
	public int m_PlayerSeed = 1000;
	public List<PlayerData> m_ConnectedPlayers = new List<PlayerData>();

	void Start() { NetworkServerProcessing.SetGameLogic(this); }
	
	public PlayerData Add(int clientConnectionID);
	public PlayerData Remove(int clientConnectionID);
	public PlayerData Search(int playerSeed);
}
```

^201a55

```csharp
public PlayerData Add(int clientConnectionID)
{
	PlayerData playerData = new PlayerData(clientConnectionID, m_PlayerSeed++);
	m_ConnectedPlayers.Add(playerData);

	return playerData;
}
```

^fc9c2b

```csharp
public PlayerData Remove(int clientConnectionID)
{
	PlayerData removedPlayerData = null;
	foreach (PlayerData playerData in m_ConnectedPlayers)
	{
		if (playerData.m_ClientConnectionID == clientConnectionID)
		{
			removedPlayerData = playerData;
			m_ConnectedPlayers.Remove(removedPlayerData);
			break;
		}
	}
	return removedPlayerData;
}
```

^e4f3d1

```csharp
public PlayerData Search(int playerSeed)
{
	PlayerData foundPlayerData = null;
	foreach (PlayerData playData in m_ConnectedPlayers)
	{
		if (playData.m_Seed == playerSeed)
		{
			foundPlayerData = playData;
			break;
		}
	}
	return foundPlayerData;
}
```

^52ae28

#### NetworkServerProcessing.cs

```csharp
static public class NetworkServerProcessing
{
	static public void ReceivedMessageFromClient(string msg, int clientConnectionID, TransportPipeline pipeline);
	static public void SendMessageToClient(string msg, int clientConnectionID, TransportPipeline pipeline);
	
	static public void ConnectionEvent(int clientConnectionID);
	static public void DisconnectionEvent(int clientConnectionID);
}
```

^d082db

```csharp
static public void ReceivedMessageFromClient(string msg, int clientConnectionID, TransportPipeline pipeline)
{
    string[] csv = msg.Split(',');
    int signifier = int.Parse(csv[0]);
    
    switch (signifier)
    {
		case ClientToServerSignifiers.PTC_PLAYER_MOVE:
		{
		    string seed = csv[1];
		    string posX = csv[2], posY = csv[3], posZ = csv[4];
		
		    // Update server-side position data
		    gameLogic.Search(int.Parse(seed))?.SetData(posX, posY, posZ);
		
		    // Forward updated position to all connected clients
		    string msgOut = $"{ServerToClientSignifiers.PTS_PLAYER_MOVE},{seed},{posX},{posY},{posZ}";
		    foreach (PlayerData data in gameLogic.m_ConnectedPlayers)
		        SendMessageToClient(msgOut, data.m_ClientConnectionID, TransportPipeline.ReliableAndInOrder);
		}
		break;
		
		case ClientToServerSignifiers.PTC_PLAYER_MOVE2:
			// [Omitted] Input-based version of the above.
			break;
	}
}
```

^e38f3b

### Client Side
#### GameLogic.cs
```csharp
public class GameLogic : MonoBehaviour
{
	public Player m_prefabPlayer;
	public Player m_prefabOthers;
	public List<Player> m_ConnectedPlayers = new List<Player>();

	void Start() { NetworkClientProcessing.SetGameLogic(this); }

	public void SpawnMySelf(int mySeed, Vector3 position);
	public void SpawnOthers(int otherSeed, Vector3 position);
	public void MovePlayer(int movedPlayerSeed, Vector3 targetPos);
	public void MovePlayer(int movedPlayerSeed, Vector3 targetPos, Vector2 inputKeys);
	public void OtherPlayerLeft(int leftPlayerSeed);
}
```

^ae6818

```csharp
public void SpawnMySelf(int mySeed, Vector3 position)
{
	Player mySelf = Instantiate(m_prefabPlayer, position, Quaternion.identity);
	PlayerData myData = new PlayerData(mySelf, mySeed, true);
	
	mySelf.InitData(myData);
	m_ConnectedPlayers.Add(mySelf);
}
```

^d888d2

```csharp
public void SpawnOthers(int otherSeed, Vector3 position)
{
	Player other = Instantiate(m_prefabOthers, position, Quaternion.identity);
	PlayerData otherData = new PlayerData(other, otherSeed, false);

	other.InitData(otherData);
	m_ConnectedPlayers.Add(other);
}
```

^1f211d

```csharp
public void MovePlayer(int movedPlayerSeed, Vector3 targetPos)
{
	foreach (Player player in m_ConnectedPlayers)
	{
		if (player.m_PlayerData.m_IsMe)
			continue;

		if (player.m_PlayerData.m_Seed == movedPlayerSeed)
		{
			player.SetMovePosition(targetPos);
		}
	}
}
```

^3cb25e

```csharp
public void MovePlayer(int movedPlayerSeed, Vector3 targetPos, Vector2 inputKeys)
{
	foreach (Player player in m_ConnectedPlayers)
	{
		if (player.m_PlayerData.m_IsMe)
			continue;

		if (player.m_PlayerData.m_Seed == movedPlayerSeed)
		{
			player.SetMovePosition(targetPos, inputKeys);
		}
	}
}
```

^a84f81

```csharp
public void OtherPlayerLeft(int leftPlayerSeed)
{
	foreach (Player player in m_ConnectedPlayers)
	{
		if (player.m_PlayerData.m_Seed == leftPlayerSeed)
		{
			m_ConnectedPlayers.Remove(player);
			player.RemoveData();
			break;
		}
	}
}
```

^068ad0

#### NetworkClientProcessing.cs
```csharp
static public class NetworkClientProcessing
{
	static public void ReceivedMessageFromServer(string msg, TransportPipeline pipeline);
	static public void SendMessageToServer(string msg, TransportPipeline pipeline);

	static public void ConnectionEvent();
	static public void DisconnectionEvent();
}
```

^ac80a7

```csharp
static public void ReceivedMessageFromServer(string msg, TransportPipeline pipeline)
{
	string[] csv = msg.Split(',');
	int signifier = int.Parse(csv[0]);

	switch (signifier)
	{
		case ServerToClientSignifiers.PTS_CONNECTED_NEW_PLAYER:
			// [Omitted] Initialize and spawn the local player
			break;
		case ServerToClientSignifiers.PTS_CONNECTED_NEW_PLAYER_RECEIVE_DATA:
			// [Omitted] Receive and spawn existing players from server
			break;
		case ServerToClientSignifiers.PTS_CONNECTED_PLAYERS_RECEIVE_NEW_PLAYER_DATA:
			// [Omitted] Spawn newly joined remote player on other clients
			break;
		case ServerToClientSignifiers.PTS_PLAYER_MOVE:
			// [Omitted] Update remote player’s position (Type A)
			break;
		case ServerToClientSignifiers.PTS_PLAYER_MOVE2:
			// [Omitted] Update remote player’s position & input (Type B)
			break;
		case ServerToClientSignifiers.PTS_PLAYER_LEFT:
			// [Omitted] Remove disconnected player from scene
			break;
	}
}
```

^c3ff2f
