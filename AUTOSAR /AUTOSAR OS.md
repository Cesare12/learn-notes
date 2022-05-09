# AUTOSAR OS

## 1. 交互模块

### 1.1 RTE

1. 将 SWC 的 runnables 映射到操作系统调度的（一个或多个）task；

2. 一个 task 中的所有 runnables 共享相同的保护边界；

3. runnables 通过 AUTOSAR RTE 访问硬件来源的数据；

4. RTE 提供 runnables 和 BSW 模块之间的运行时接口；

5. IOC 提供 OS Applications 之间的通信。IOC 的生成基于 RTE 生成器生成的配置信息。 另一方面，RTE 使用 IOC 生成的函数来传输数据；

6. AUTOSAR 软件模块还包括由操作系统使用操作系统抽象层 (OSAL) 调度
的许多 task 和 ISR。

### 1.2 EcuM/BswM

OS 的启动：

C 代码初始化 -> EcuM_Init() -> StartPreOS Sequence -> StartOS() -> StatupHook() -> ActiveTask() -> EcuM_StartupTwo() -> StartPostOS Sequence

OS 关闭

BswM 调用 EcuM_SelectShutdownTarget() -> EcuM ref OffPreOS Sequence -> ShutdownOS() -> ShutdownHook() -> EcuM_Shutdown() -> EcuM ref OffPostOS Sequence 

## 2. 基础背景

AUTOSAR OS 是由 C 语言实现，它是静态配置的，有几个基本特性：

1. 静态配置和缩放
2. 事件触发的操作系统，可以进行实时性能推理
3. 提供优先级调度策略
4. 在运行时提供保护（存储，时间，等等）
5. 可以低端控制器运行，无需额外资源

Scalability class 1: OS-Applications 里的基本对象

Scalability class 2: 比 SC1 多了时间保护

Scalability class 3: 比 SC1 多了内存保护

Scalability class 4: SC1 + SC2 + SC3

## 3. 调度表

调度表通过配置 expiry point 来解决同步问题。每个 expiry point 都可以配置多个 Task 或 Event。在运行时，OS 会遍历调度表处理每个 expiry point。遍历是被 counter 驱动的。

也就是 OS 自己有个 OS Counter 一直在运行，schedule table 在 Counter 的上一层，跟 OS Counter 做同步。

## 4. OS Application

Tasks, ISRs, Alarms, Schedule tables, Counters 构成一个功能单元 叫 OS Application。

1. 可信 OS Application：受信任的操作系统应用程序可以在运行时禁用监控或保护功能的情况下运行。 它们可以不受限制地访问内存、操作系统模块的 API，并且不需要在运行时强制执行它们的计时行为。 当处理器支持时，它们可以在特权模式下运行。
2. 不可信 OS Application：在运行时禁用监控或保护功能的情况下，不允许运行不受信任的操作系统应用程序。 它们对内存的访问受到限制，对操作系统模块的 API 的访问受到限制，并在运行时强制执行它们的计时行为。 当处理器支持时，它们不允许在特权模式下运行。

资源不属于任何 OS Application，需要明确对资源访问的访问权限。

OS Application 有三种状态：
1. Active and accessible（APPLICATION_ACCESSIBLE）：OS Application 可以从其它 OS Application 访问。这是启动时的默认状态。
2. 目前处于重启阶段（APPLICATION_RESTART）。不能从其他 OS Application 访问操作系统对象。在 OS Application 调用 AllowAccess() 之前，状态一直有效。
3. Terminated and not accessible（APPLICATION_TERMINATED）：不能从其他 OS Applications 访问操作系统对象。 状态不会改变，不能再切换其他状态了。

## 5. 存储保护

保护对象仅限于 Task 和 2类中断。

内存保护由硬件支持，基于段（data, code and stack）。

1. 堆栈：根据定义，Task/ISR 的堆栈仅属于所有者对象，因此即使这些 Task/ISR 属于同一个 OS Application，也无需在这些 Task/ISR 之间共享堆栈数据。
2. 数据：OS Application 的私有数据部分被属于该 OS Application 的所有 Task/ISR 共享。
3. 代码：代码部分要么是 OS Application 私有的，要么可以在所有 OS Applications 之间共享（以使用共享库）。 在不使用代码保护的情况下，执行不正确的代码最终会导致内存、时序或服务违规。

如果检测到内存访问违规，操作系统模块将调用状态码为 E_OS_PROTECTION_MEMORY 的 Protection Hook。

## 6. 时间保护

时序保护不是直接 监控 最后期限。

Task/ISRs 执行时间，有上界，叫做 执行预算。
Task/ISRs 被低优先级 Tasks/ISRs 占用共享资源 或者 被关中断 的阻塞时间，有上界，叫做 阻塞预算。
Task/ISRs 的到达间隔率，有下界，叫做 时间帧。

## 7. Service 保护

由于 OS Application 可以通过服务与 OS 进行交互，因此服务调用不会 OS 模块本身是至关重要的。 服务保护可在运行时防止此类损坏。

服务保护需要考虑多种情况：

1. 具有无效或超出范围值处理。
2. 在错误的上下文中，例如 在 StartupHook() 中调用 ActivateTask()。
3. 或错误的进行 API 调用导致 OSEK 操作系统处于未定义的状态，例如 它在没有 ReleaseResource() 调用的情况下就终止了。
4. 影响系统中所有其他 OS Application 的行为，例如 ShutdownOS()。
5. 操作属于另一个 OS Application 的系统对象（对其没有权限），例如 OS Application 尝试在它不拥有的 Task 上执行 ActivateTask()。

## 8. 硬件保护

OS 模块的服务需要在特权模式下执行，可以使用 MPU、定时器单元、中断控制器等的控制寄存器，因此有必要保护这些寄存器免受不受信任的 OS Application 的影响。

OS Application 可以调用由（另一个）受信任的 OS Application 提供的受信任功能。这可能需要从非特权模式切换到特权模式。

从不受信任的 OS Application 对受信任服务的调用正在使用 OS 提供的机制（例如陷阱/软件中断）。服务作为标识符传递，用于在可信环境中确定是否可以调用服务。

## 9. 多核

AUTOSAR 多核 OS 源自现有的 AUTOSAR OS，这是一个共享相同配置和大部分代码的 OS，但针对每个内核运行不同的数据结构。

ID（例如 TASKID、RESOURCEID 等）在内核中是唯一的。

由于多个内核真正并行运行，因此可以同时执行多个 TASK。

TASK 和 ISR 不能通过调度算法动态改变内核。

可定位实体：必须完全位于一个核上。 (OsApplicationCoreRef)。

在单核系统上，OS Applications 仅适用于 SC3 和 SC4，因为该机制用于支持内存保护并暗示使用扩展模式。在多核系统中，操作系统应用程序始终可用，独立于内存保护，并且在 SC1 标准模式下应该是可能的。

启动：AUTOSAR 多核操作系统规范要求系统具有主从启动行为，或者由硬件直接支持或在软件中模拟。主核被定义为不需要软件激活的核，而从核需要通过软件激活。
 
AUTOSAR 规范不支持同步阶段的超时。

StartNonAutosarCore 可用于启动不受 AUTOSAR OS 控制的核。

OS-Applications 终止：与 TASK 类似，当 OS-Application 的任何 TASK 占用自旋锁时，不能终止 OS-Application。

等待事件：必须适应新的多核自旋锁。
调用 WaitEvent 时可能会取消调度一个 TASK，在这种情况下它将无法释放自旋锁。 因此，WaitEvent 必须检查调用的 TASK 是否持有自旋锁。 与 RESOURCES 一样，自旋锁不能被处于等待状态的 TASK 占用。

调用重新调度：调度 API 服务必须以与 WaitEvent 相同的方式适应新的多核自旋锁功能。

RESOURCE 占用：优先级上限协议（由 GetResource 使用）临时更改 TASK 的优先级。 这种方法在多核系统上失败了，因为优先级对于每个核心都是本地的。 因此，上限协议不足以保护关键部分免受来自不同核心的访问。

## 10. IOC

IOC 负责 OS-Applications 之间的通信，特别是跨内核或内存保护边界的通信。

RTE 使用 IOC 服务跨越这些边界进行通信。
不支持通过 BSW 模块直接访问 IOC 服务，但允许 CDD 和其他模块（如果不可避免）。

IOC 仅提供 sender/receiver（信号传递）通信。 RTE（或本规范未来版本中的适配 BSW 模块）将 client/server 调用和响应传输转换为 sender/receiver 通信。

IOC 支持 1:1、N:1 和 N:M（仅限非排队）通信。

提供非排队（Last-is-Best，数据语义）或排队（先进先出，事件语义）通信操作。

一旦传输的数据可用于接收方访问，IOC 通过调用由通信用户提供的配置回调函数，可选择通知接收方。

IOC 函数不可重入。

## 11. IOC 配置和生成：

1. 在 ECU 配置描述文件中指定 分配到 OS-Applications 和内核的 SWC 的所有信息。
2. 生成 RTE。 RTE 生成器创建数据元素特定的 IOC 服务调用和相应的 IOC 配置描述块（XML 格式），以指定每个数据元素的通信关系。
3. 根据 IOC 配置描述（步骤 2）生成 IOC 代码，同时考虑硬件描述文件。 此外，生成一个头文件 (Ioc.h) 以包含在 RTE.c 中，以提供定义、函数原型和宏。
