# **Command Queue for Actions**
---
## **Overview**
A **Command Queue** is a design pattern that uses a **queue-based structure** (FIFO) to **collect commands each frame** and **process them consistently**, separating game systems from object behavior. It organizes game logic around queued commands and **helps control actions in a clean and flexible way**. (e.g., used in [[D3D12 FlightDemo]])

> ⚠️ Note: This pattern improves structure and modularity, but may introduce **overhead due to deferred execution**, and can be harder to debug if commands are misrouted or delayed unexpectedly.

[**View Repository**](https://github.com/Woo95/Command-Queue-for-Actions)

---
## **Structure Diagram**
![[CommandQueue Structure Diagram.png]]
- **White border**: process flow
- **Yellow border**: commands container

---
## **Class Descriptions**
### `Command Struct - Code Breakdown`
![[Command Queue for Actions Block References#^27cad2]]
Wraps a callable `action` (usually a lambda or function object) that takes `float deltaTime` as its parameter. This allows the command to handle time-dependent logic.

---
### `CommandQueue Class - Code Breakdown`
![[Command Queue for Actions Block References#^50c1f1]]
The `CCommandQueue` manages a FIFO list of `Command` and handles their execution. It decouples command creation from execution timing, allowing cleaner and more modular game logic.

---
## **Code Breakdown**
### `Usage Example`
![[Command Queue for Actions Block References#^8d7160]]
This example simulates a 5-frame loop where each frame generates a command using a lambda that captures `player`, `monster`, and `frame`. Each command is pushed to the scene via `PushCommand()` and executed during `Scene::Update()`.

---
#### `CScene::Update()`
![[Command Queue for Actions Block References#^5edefe]]
This method dequeues and executes all pending commands in order.

---
## **Conclusion**
> As a **design pattern**, the **command queue** cleanly separates **logic** from **execution**, making **frame-based systems** more **modular** and **maintainable**. While it improves **structure** and **timing control**, it may introduce minor **overhead** due to **deferred execution**.