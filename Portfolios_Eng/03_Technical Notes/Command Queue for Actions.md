# **Command Queue for Actions**
---
## **Overview**
A **Command Queue** is a design pattern that uses a **queue-based structure** (FIFO) to **collect commands each frame** and **process them consistently**, separating game systems from object behavior. It organizes game logic around queued commands and **helps control actions in a clean and flexible way**. (e.g., used in [[Portfolios_Eng/03_Technical Notes/D3D12 FlightDemo|D3D12 FlightDemo]])

> ⚠️ Note: This pattern improves structure and modularity, but may introduce **overhead due to deferred execution**, and can be harder to debug if commands are misrouted or delayed unexpectedly.

[**View Repository**](https://github.com/Woo95/Command-Queue-for-Actions)

---
## **Structure Diagram**
![[CommandQueue Structure Diagram_Eng.png]]
- **White border**: process flow
- **Yellow border**: commands container

---
## **Class Descriptions**
### `Command Struct - Code Breakdown`
![[Command Queue for Actions (Code)#^27cad2]]
Wraps a callable `action` (usually a lambda or function object) that takes `float deltaTime` as its parameter. This allows the command to handle time-dependent logic.

---
### `CommandQueue Class - Code Breakdown`
![[Command Queue for Actions (Code)#^50c1f1]]
The `CCommandQueue` manages a FIFO list of `FCommand` and handles their execution. It decouples command creation from execution timing, allowing cleaner and more modular game logic.

---
## **Code Breakdown**
### `Usage Example`
![[Command Queue for Actions (Code)#^8d7160]]
This example simulates a 5-frame loop where each frame generates a command using a lambda that captures [[Command Queue for Actions (Code)#^bc13cb|player]], [[Command Queue for Actions (Code)#^55138a|monster]], and `frame`. Each command is pushed to the [[Command Queue for Actions (Code)#^6c2790|scene]] via `PushCommand()` and executed during `Scene::Update()`.

---
#### `CScene::Update()`
![[Command Queue for Actions (Code)#^5edefe]]
This method dequeues from `mCommandQueue`, then executes all pending commands in order.

---
## **Conclusion**
> The command queue pattern **can clearly separate logic from execution, making frame-based system more modular and maintainable, while also improving structure, timing control, and batch processing optimization**. 
>
> On the other hand, it **may introduce overhead due to deferred execution, complicate command ordering and debugging, and increase memory usage** if commands are continuously retained in the queue.
>
> Therefore, it's important to consider these carefully and decide whether to adopt it based on the specific requirements of the project.