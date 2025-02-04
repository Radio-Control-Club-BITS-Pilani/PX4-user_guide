# 云台配置

本页面介绍了如何配置及控制一个安装了相机（或其他任务载荷）的云台。

## 概述

PX4包含支持不同输入输出方法的通用云台驱动程序：

- **输入方法**定义了被PX4管理的云台指令协议。 该输入可能是一个RC遥控器，通过GCS发送的MAVLink命令，或者两者兼备并可自动切换。
- **输出方法**定义了PX4如何与已连接的云台通讯。 推荐使用Mavlink v2协议，但是你也可以直接通过PWM输出端口与非空链接。

PX4接收输入信号并将其经过路由/转换发送至输出。 任何输入方法都可以被用来驱动任何输出。

输入和输出都使用参数进行配置。 输入通过参数[MNT_MODE_IN](../advanced_config/parameter_reference.md#MNT_MODE_IN)进行设置。 默认情况下设置为 `Disabled (-1)` 既驱动不运行。 在选择输入模式后，请重新启动飞行器以启动云台驱动程序。

`MNT_MODE_IN` 应该被设置为下列选项中的其中一个： `RC (1)` ，`MAVlink gimbal protocol v2 (4)` 或 `Auto (0)` (其他选项已废弃)。 如果选择 `Auto(0)`，则云台将基于最新地输入自动选择 RC 或 MAVLink 作为输入。 请注意，从 MAVLink 到 RC 的自动切换需要一个大幅度地杆量操作！

输出通过参数[MNT_MODE_OUT](../advanced_config/parameter_reference.md#MNT_MODE_OUT)进行设置。 默认情况下，输出被设置为 PXM 端口(`AUX (0)`)。 如果云台支持 [MAVLink Gimbal Protocol v2](https://mavlink.io/en/services/gimbal_v2.html) ，应该选择 `MAVLink gimbal protocl v2 (2)`。

云台驱动的完整参数列表可在 [参数 > Mount](../advanced_config/parameter_reference.md#mount) 中找到。 下面介绍了一些通用的云台相关设置。

## MAVLink 云台(MNT_MODE_OUT=MAVLINK)

系统上的每个物理云台装置必须有自己的高级云台管理器， 地面站通过使用MAVLink云台协议发现它。 地面站将高级 [MAVLink Gimbal Manager](https://mavlink.io/en/services/gimbal_v2.html#gimbal-manager-messages) 命令发送给它想要控制的云台管理器。 而管理器则会发送适当的较低级别的“云台设备”命令来控制云台。

PX4可以配置为云台管理器以控制耽搁云台设备（可以是物理连接的设备，或者实现云台设备接口[gimbal device interface](https://mavlink.io/en/services/gimbal_v2.html#gimbal-device-messages)的MAVLink云台

要启用 MAVLink云台 ，首先设置参数 [MNT_MODE_IN](../advanced_config/parameter_reference.md#MNT_MODE_IN) 为 `MAVlink gimbal protocol v2` 和 [MNT_MODE_OUT](../advanced_config/parameter_reference.md#MNT_MODE_OUT) 为 `MAVlink gimbal protocol v2`

云台使用[MAVLink Peripherals (GCS/OSD/Companion)](../peripherals/mavlink_peripherals.md)中的指导可以连接到_任意未用串口_（参见[串口配置](../peripherals/serial_configuration.md#serial-port-configuration)） 例如， 如果飞行控制器上的 `TELEM2` 端口未被使用，您可以将其连接到云台并设置下面的 PX4 参数：

- [MAV_1_CONFIG](../advanced_config/parameter_reference.md#MAV_1_CONFIG)为**TELEM2**（如果`MAV_1_CONFIG`已经用于连接机载计算机，使用`MAV_2_CONFIG`）。
- [MAV_1_MODE](../advanced_config/parameter_reference.md#MAV_1_MODE)为**NORMAL**
- [SER_TEL2)BAUD](../advanced_config/parameter_reference.md#SER_TEL2_BAUD)设置为厂家建议的波特率。

### 多云台支持

PX4 可以自动为已连接的 PWM 云台或第一个在任何接口上检测到相同的系统 id的MAVLink 云台设备创建一个云台管理器。 它不会自动为它检测到的其他MAVLink云台设备创建云台管理器。

您可以支持额外的云台 ，但他们必须：

- 实现 云台 _管理器_ 协议
- 在 MAVLink 网络上对地面站和 PX4 可见。 这可能需要在PX4、GCS和云台之间配置流量转接。
- 每个云台必须有一个独特的ID。 对于已连接的 PWM 云台，这将是飞控系统的组件 ID

## 飞控PWM 输出上的云台(MNT_MODE_OUT=AUX)

可以通过设置输出模式为`MNT_MODE_OUT=AUX`，把云台连接到飞控的 3个PWM 端口。

用于控制云台的输出引脚设置在 [Acuator 配置 > 输出](../config/actuators.md#actuator-outputs) 中通过选择任何三个未使用的驱动输出并赋予它们以下输出功能：

- `Gimbal Roll<0>：输出控制云台滚动</li>
<li><code>Gimbal Pitch`：输出控制俯仰
- `Gimbal Yaw`：输出控制云台转动

For example, you might have the following settings to assign the gimbal roll, pitch and yaw to AUX1-3 outputs.

![Gimbal Actuator config](../../assets/config/actuators/qgc_actuators_gimbal.png)

The PWM values to use for the disarmed, maximum and minimum values can be determined in the same way as other servo, using the [Actuator Test sliders](../config/actuators.md#actuator-testing) to confirm that each slider moves the appropriate axis, and changing the values so that the gimbal is in the appropriate position at the disarmed, low and high position in the slider. The values may also be provided in gimbal documentation.

## SITL

The [Gazebo Classic](../sim_gazebo_classic/README.md) simulation [Typhoon H480 model](../sim_gazebo_classic/vehicles.md#typhoon-h480-hexrotor) comes with a preconfigured simulated gimbal.

要运行它，请使用：

```sh
make px4_sitl gazebo-classic_typhoon_h480
```

To just test the [gimbal driver](../modules/modules_driver.md#gimbal) on other models or simulators, make sure the driver runs (using `gimbal start`), then configure its parameters.

## 测试

The driver provides a simple test command. 接下来描述了在SITL中的测试方式，但是这些指令也可以在真实的设备中使用。

使用下面这条指令开始仿真（不需要修改任何参数）：

```sh
make px4_sitl gazebo-classic_typhoon_h480
```

确保无人机是上锁状态，例如使用`命令行 takeoff`， 然后用下面的命令来控制云台（例如）：

```sh
gimbal test yaw 30
```

注意模拟的云台自身稳定，因此如果发送 MAVLink 命令，设置`stabilize`标志为`false`。

![Gazebo 云台仿真](../../assets/simulation/gazebo_classic/gimbal-simulation.png)
