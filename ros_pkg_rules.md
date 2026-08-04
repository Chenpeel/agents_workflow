# 📐 ROS2 工程强约束 Rules（具身智能工程规范）

> 机器可读版本见 [`ros_pkg_rules.yaml`](./ros_pkg_rules.yaml)，两份文档内容必须保持一致。

## 一、总体架构原则（强制）

所有 ROS 包必须采用三层分离结构：

- **Node 层** → ROS 接口与 IO 适配
- **Manager 层** → 系统编排与状态调度
- **Algorithm 层** → 纯算法与模型（无 ROS 依赖）

### 分层依赖规则（必须遵守）

- Node 层只能依赖 Manager 层
- Manager 层只能依赖 Algorithm 层
- Algorithm 层不得依赖 ROS / rclcpp / ROS msg

### ❌ 严禁以下行为

- 中间层向上 include
- Node 层跨层直接调用 Algorithm
- 上层访问下层内部数据结构
- 通过成员变量共享跨层状态

### ✅ 层间通信规范

- 层间通信必须通过函数接口完成
- 下层对上层提供接口，上层不得依赖实现细节

## 二、Node 层职责边界（ROS 接口层）

### ✅ Node 层只允许做

- topic / service / action 通信
- 参数读取与动态参数
- timer
- ROS msg ↔ 内部数据结构转换
- TF 发布与监听

> Node 是 IO 适配器，不是系统控制中心。

### ❌ Node 层严禁出现

- 状态机
- 模式判断
- 控制逻辑
- 算法参数管理
- 多模块协调逻辑

### ❌ 禁止出现如下结构

```cpp
if (mode == AUTO) {
   do_auto_control();
} else {
   do_manual_control();
}
```

> 模式决策必须在 Manager 层完成。

## 三、Manager / System 层职责（系统编排核心）

### ✅ Manager 层必须负责

**模式管理 / 状态机**

- teleop / auto
- relative / absolute
- safe / emergency

```cpp
switch (state_) {
  case TELEOP: ...
  case AUTO: ...
}
```

**多输入源选择与融合**

- VR + 按钮
- 自动规划 + 人工修正
- 多传感器融合

> Node 只接收数据，Manager 决定用谁。

**数据缓存与时间对齐**

- 最近一帧缓存
- 插值
- 滤波前缓冲

**算法调度**

Manager 不实现算法，但必须决定：

- 何时调用
- 使用哪个算法实例
- 使用什么参数

### ❌ Manager 层严禁

- ROS topic/service 通信
- 发布 ROS msg
- TF 操作
- 参数服务器访问

> 否则视为 ROS 强耦合，违反可复用与可测试原则。

## 四、Algorithm / Core 层原则（纯算法层）

### ✅ Algorithm 层必须满足

- 不 include rclcpp
- 不使用任何 ROS msg
- 仅使用：
  - Eigen
  - STL
  - 自定义 struct

示例：

```cpp
struct Pose {
  Eigen::Vector3d p;
  Eigen::Quaterniond q;
};

Pose process(const Pose& raw);
```

- 必须支持脱离 ROS 的单元测试
- 不包含 IO、不包含线程、不包含调度

## 五、跨层接口与耦合控制

- 所有跨层交互必须通过函数接口
- 禁止返回内部容器引用
- 禁止上层修改下层内部状态
- 下层负责数据一致性与边界检查

## 六、ROS 包级别设计规范

### 1. 功能单一原则

- 一个 ROS 包只实现一个独立功能
- 同一包内不得出现两个系统级关键节点

### 2. Node 命名规则

- Node 层类名与可执行文件名必须为：包名
- 包名必须准确描述功能，不允许泛化命名

### 3. 启动方式强制规范

- 所有 Node 必须通过 launch 文件启动
- ❌ 禁止使用 `ros2 run` 作为正式运行方式

## 七、C++ 文件结构强制规范

### 1. 头源文件分离（强制）

- 每个 `.cpp` 必须有对应 `.hpp`
- `.hpp`：只允许声明
- `.cpp`：只允许实现

### 2. 头文件实现限制

`.hpp` 中允许存在：

- inline
- 极小型桥接函数
- 单个实现不得超过 5 行

❌ 禁止：

- 在 `.cpp` 中写声明
- 在 `.hpp` 中实现业务逻辑
- 模板滥用导致实现暴露

## 八、允许的结构简化规则

- 可按需求省略 Manager 层
- 但一旦存在系统模式、调度或多源选择逻辑，必须引入 Manager 层
- 禁止 Node 直接承担 Manager 职责

## 九、与大模型交互时的执行规则

当用户请求修改或新增功能时：

- 必须先判断其所在层级
- 只能在对应层级内修改
- 不得因实现方便破坏分层
- 若必须跨层改动，必须先说明并等待授权

## 十、强制工程一致性

所有代码交付必须同时满足：

- 架构分层正确
- 头源文件规范
- ROS2 构建系统可通过 `colcon build` 无错误

> 违反任一条即视为不可接受实现。
