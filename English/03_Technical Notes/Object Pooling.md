# **Object Pooling**
---
## **Overview**
This technique is a **performance optimization** technique that **reuses created objects** to avoid frequent memory allocation and deallocation. Similar to [[MemoryPool Library|memory pool]], it focuses on recycling fully initialized object instances. It is ideal for objects that repeatedly appear and disappear.

> ⚠️ Note: This technique improves performance by **reducing CPU overhead** through reuse, though it **may increase memory usage** due to pre-allocation.

---
## **Structure Diagram**
![[object pooling structure diagram_Eng.png]]
 - **White border**: process flow  
 - **Yellow border**: objects container
---
## **Practical Application Examples**

### `1. Project: Infinite Runners`
> *Infinite Runners* is one of my old project, used here to demonstrate object pooling.

In *Infinite Runners*, the `MapGenerator` class uses an **array-based pool structure** to randomly reuse maps through Object Pooling, allowing for varied map sequences.
![[Object Pooling (Media)#^a71532]]
#### `MapGenerator Class - Execution Flow`
- These methods are called once at the start of the game to set up the map pool:
	- [[Object Pooling (Code)#^b06806|Start()]] → [[Object Pooling (Code)#^27dd05|CreateMapPool()]] → [[Object Pooling (Code)#^31cd3b|InitializeFreeIndices()]] → [[Object Pooling (Code)#^302dda|SpawnMapsAtStart()]]
- These methods are triggered during gameplay:
	- [[Object Pooling (Code)#^612f56|OnTriggerEnter(Collider other)]] → [[Object Pooling (Code)#^5232ac|ReplaceMap(int oldIdx)]]

#### `MapGenerator Class - Code Breakdown`
![[Object Pooling (Code)#^27dd05]]
All map objects are instantiated once and stored in an array. They are set inactive by default, and each map is assigned a unique pool index via its `MapBehaviour` component for future tracking.

---
![[Object Pooling (Code)#^31cd3b]]
This function fills a list with all the indices of the pre-instantiated map objects. It acts as a tracker for which maps are currently available for reuse.

---
![[Object Pooling (Code)#^302dda]]
A set number of maps (`m_SpawnAmount`) are randomly taken from the pool, positioned sequentially, and activated to create the starting environment.

---
![[Object Pooling (Code)#^612f56]]
This function is called when a collider enters the trigger zone. If the collider belongs to a map (tagged `"Map"`) and has a `MapBehaviour` component, its pool index is passed to `ReplaceMap()`. Otherwise, the object is destroyed.

---
![[Object Pooling (Code)#^5232ac]]
This function recycles maps by selecting a free index, repositioning and activating a new map, and deactivating the old one while returning its index to the pool for future reuse. It ensures the pool maintains reusable maps without frequent instantiation.

---
### `2. Project: Spaceship Battle`
> *Spaceship Battle* is one of my old project, used here to demonstrate object pooling.
> [**View Repository**](https://github.com/Woo95/Unity_2D_SpaceShipBattle_Automatic_CameraSetup_With_Object_Pooling)

In *SpaceShip Battle*, the `BulletManager` class uses a **queue-based pool structure** to reuse bullets, as there's no need for random selection like in *Infinite Runners*.
![[Object Pooling (Media)#^b6ae16]]
#### `No Pool vs. Pool - Performance Comparison`
![[Object Pooling (Media)#^65f1dd]]
As seen in the images above, the **FPS difference** shows the left side without pooling performs worse, while the right side **with pooling shows a significant performance boost**.

---
## **Conclusion**
> **Object Pooling is a powerful technique** for managing frequently created and destroyed objects at runtime. However, **pre-creating too many objects can lead to significant memory waste**. It's essential to estimate how many objects will be needed during runtime. Based on that estimate, an **appropriate pool size should be selected** to avoid over-allocation. Finally, as seen in my two projects demonstrating object pooling, **selecting the right data structure for the pool is crucial**, depending on the specific use case.