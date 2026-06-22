## 目标

* 依据 `/d:/HuaweiMoveData/Users/fengl/Desktop/Obsidian/编程/UML/设计原则/shotDuck.md` 的类图，用面向对象方式实现两个场景（Hot、Cold）。

* 严格遵守 `/d:/HuaweiMoveData/Users/fengl/Desktop/Obsidian/编程/UML/设计原则/GRASP.md` 的原则。

* 输出为命令行字符串（队列），通过 `map<string, queue<string>>` 和 `produceAnimate(file, description)` 打印。

## 技术选择

* 语言：C++17。

* 单文件或少量 `.hpp/.cpp` 文件均可；默认先用单文件 `main.cpp` 保持简单。

* 使用 `std::unique_ptr` 管理对象生命周期；对外接口遵循类图方法签名与命名。

## 核心类与接口

* `Weather`：枚举，包含 `HOT`、`COLD`。

* `MoveBehavior`：纯虚接口，`move(name)` 返回 `std::string`（统一运动策略）。

  * 实现：`FlyWithWings`、`FlyWithPropulsion`、`FlyWithPropeller`、`WalkOnLand`、`SwimInWater`、`FlyNoWay`、`StopOnGround`。

* `QuackBehavior`：纯虚接口，`quack(name)` 返回 `std::string`。

  * 实现：`Quack`、`Squick`、`MuteQuack`。

* `ClothingBehavior`：纯虚接口，`wear(personName)` 返回 `std::string`。

  * 实现：`HotWear`、`ColdWear`（具体衣物文案自行设定）。

* `Duck`：

  * 成员：`moveBehavior`、`quackBehavior`、`isAlive`、`name`。

  * 方法：`performFly()`、`performSwim()`、`performQuack()`、`display()`、`setMoveBehavior(MoveBehavior)`、`setQuackBehavior(QuackBehavior)`、`die()`（均返回 `std::string` 或变更状态）。

  * 子类：`MallardDuck`、`RedHeadDuck`、`RubberDuck`、`DecoyDuck`（构造时设定默认策略，如 `RubberDuck` 用 `FlyNoWay` + `Squick`）。

* `Person`：

  * 成员：`clothing`、`moveBehavior`、`name`。

  * 方法：`performWalk()`、`display()`、`setClothing(ClothingBehavior)`、`setMoveBehavior(MoveBehavior)`。

  * 子类：`Hunter`（新增 `shoot(Duck&)`，返回 `std::string` 并令目标 `die()`）、`Boy`、`Girl`。

* `Aircraft`：

  * 成员：`moveBehavior`、`name`。

  * 方法：`performFly()`、`performStopOnGround()`、`takeOff()`、`display()`、`setMoveBehavior(MoveBehavior)`。

  * 子类：`Boeing`、`Apache`。

* `SimulationController`：

  * 成员：`weather`。

  * 方法：`setWeather(Weather)`、`runScene()`，返回 `std::map<std::string, std::queue<std::string>>`。内部根据 `weather` 调度两种场景。

* `produceAnimate(file, description)`：按场景键遍历队列并打印到命令行（保留签名参数中的文件名，但仅打印）。

## 输出规则

* 所有 `performX`/`wear`/`display`/`shoot` 返回形如：

  * `Name.Action` 或 `Name.ActionWithStrategy`，例如：`MallardDuck0.FlyWithWings`、`RubberDuck0.SwimInWater`、`Hunter0.WalkOnLand`、`Boeing0.FlyWithPropulsion`、`Hunter0.Shoots MallardDuck0`、`MallardDuck0.Dead`。

* 队列顺序严格按照场景描述。

## 场景编排

* 场景1（Hot）：

  * 创建 `Hunter0`，着 `HotWear`，`WalkOnLand`；`performWalk()`。

  * 创建 `MallardDuck0`、`RedHeadDuck0`、`RubberDuck0` 依次出现并 `performSwim()`（均为 `SwimInWater`）。

  * 一段时间后，`MallardDuck0` 设为 `FlyWithWings` 并 `performFly()`。

  * 再过一会，`Hunter0.shoot(MallardDuck0)`；记录 `MallardDuck0.Dead`。

* 场景2（Cold）：

  * 创建 `Boy0`、`Girl0`，着 `ColdWear`，`WalkOnLand`；依次 `performWalk()`。

  * 创建 `Boeing0`（`FlyWithPropulsion`）、`Apache0`（`FlyWithPropeller`）；调用 `takeOff()` 后 `performFly()`。

## main 流程

* 创建 `SimulationController`。

* `setWeather(Weather::HOT)`，获取 `description` 并调用 `produceAnimate("SceneHot.json", description)` 打印。

* `setWeather(Weather::COLD)`，同样处理并打印。

## GRASP 映射

* Controller：`SimulationController` 统一接收场景请求与编排。

* Information Expert：行为由拥有信息的对象自身返回文本；例如鸭子的游泳/飞行、人的穿衣/行走、飞机的起飞/飞行。

* Low Coupling & High Cohesion：通过策略接口隔离具体实现；`Controller` 仅协调。

* Polymorphism：`MoveBehavior`、`QuackBehavior`、`ClothingBehavior` 的多态分发。

* Indirection & Pure Fabrication：输出由各对象方法生成字符串；`produceAnimate` 充当纯粹制造者以打印描述。

* Protected Variations：客户端依赖接口，不依赖具体实现差异。

## 交付物

* `main.cpp`：包含所有类与 `main()`，可直接编译运行。

* 编译命令：`g++ -std=c++17 -O2 -Wall main.cpp -o shotDuck`。

* 运行：`./shotDuck`（Windows 下 `shotDuck.exe`）。

## 关键假设

* `Duck` 仅持有一个 `MoveBehavior`，通过在不同动作前动态切换行为实现 `performFly/performSwim` 的语义。

* `performX` 与 `shoot` 等方法均返回字符串以便入队。

* `produceAnimate` 只打印到命令行，不进行文件写入。

