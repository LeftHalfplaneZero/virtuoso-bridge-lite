# 【用Agent做了一个配合Layout EXL的插件来偷懒】AutoPorts：EXL(EMX)自动打端口并配对参考，并一键调用仿真和plot结果

**中文** | [English](introduction.en.md)

![AutoPorts 2.0 功能总览：生成顶层 pin、配对 EMX 端口、运行仿真、计算与绘图](01-overview.png)

在使用 Cadence Virtuoso 和 EMX 进行电感、电容、变压器等无源器件仿真时，端口准备往往需要重复完成很多操作：

- 从器件实例中整理并生成顶层 pin；
- 为每个信号端口创建参考 pin；
- 在 EMX 模型中生成端口，并逐个设置 Reference（特别是EXL版本中，每次重新生成端口都要把参考重新设一遍，非常烦躁）；
- 仿真结束后查找 S 参数文件；

**AutoPorts 的目标，就是把这套重复流程集中到一个 Virtuoso 插件中，让端口生成、EMX 仿真和结果分析更直接。**

## AutoPorts 能做什么？

### 1. 自动生成顶层 pin

AutoPorts 可以读取源 Cell 或 PCell 中已有的 pin，并根据实例的位置、旋转和镜像关系，在指定顶层版图中生成对应的顶层 pin。

它也支持一定深度的层级读取，适合端口位于子层实例中的设计。

### 2. 自动创建参考 pin

用户可以为不同源 Cell 指定参考金属层。插件会自动生成与信号 pin 对应的 `_ref` 参考 pin，例如：

```text
PLUS      → PLUS_ref
MINUS     → MINUS_ref
```

参考 pin 会保持与信号 pin 相同的位置和形状，减少手工绘制与命名错误。

### 3. 自动生成并配对 EMX 端口

在当前 EMX model 中，AutoPorts 可以：

- 根据顶层 pin 生成 EMX ports；
- 自动将信号端口与对应的 `_ref` 端口配对；
- 将参考端口设置为 Disabled；
- 保存当前模型设置。

如果端口已经由 Cadence 原生功能生成，也可以只执行 Reference 配对。

![信号端口与参考端口的对应关系，以及 EMX Ports 表的实机截图](02-reference-pairing.png)

*上方展示配对关系，下方是实机端口表：PLUS 对应 PLUS_ref，MINUS 对应 MINUS_ref，两个参考端口均为 Disabled。*

### 4. 直接启动 EMX 仿真

AutoPorts 2.0 可以调用当前顶层和当前模型的原生 EMX 仿真设置，并提供实时日志窗口，用于查看：

- 仿真进度；
- 错误信息；
- 结果文件路径；
- 仿真耗时和内存使用情况。

仿真完成后，插件会尝试自动找到实际生成的 S 参数文件并回填，无需手动在目录中查找。

![AutoPorts 2.0 的 EM 页面和计算结果表实机截图](03-em-and-results.png)

*左侧是包含 Simulate、Stop、Calculate 和 Plot 的 EM 页面；右侧是较早 2.0 版本读取已有 S 参数后的结果表，展示电感与 Q 的计算结果。*

### 5. 自动计算和绘制无源器件参数

AutoPorts 2.0 支持读取 Touchstone 或 Spectre ASCII PSF S 参数结果，并计算常用指标：

- 电感：L、Q；
- 电容：C、Q；
- 变压器：Lp、Qp、Ls、Qs、耦合系数 k；
- 带中心抽头变压器：整圈与各半圈的 L、Q。

目前支持八种器件模式，包括单端/差分电感、单端/差分电容、普通变压器，以及带初级、次级或双中心抽头的变压器。

计算结果可以：

- 在插件表格中查看；
- 导出为 CSV / TSV；
- 生成独立 SVG 图；
- 在 ViVA 中显示曲线。

![ViVA 中分条带显示的电感和 Q 曲线](04-viva-curves.png)

*ViVA 实机绘图示例：上方为电感，下方为 Q。曲线来自已有测试模型，仅用于展示绘图功能。*

### 6. 保存和恢复配置

AutoPorts 可以保存当前工程设置，包括：

- 顶层 Library / Cell / View；
- 已启用的源 Cell；
- 参考层；
- 层级读取深度；
- EM 器件模式；
- 端口角色映射；
- S 参数文件路径。

下次打开工程时，可以直接恢复配置，减少重复设置。

## 一个典型的使用流程

```text
选择顶层版图
    ↓
选择源 Cell 和参考层
    ↓
生成顶层 pin 与参考 pin
    ↓
生成 EMX ports 并自动配对 Reference
    ↓
使用原生 EMX 设置运行仿真
    ↓
自动获取 S 参数文件
    ↓
计算并绘制 L / C / Q / k
```

## AutoPorts 适合谁？

AutoPorts 主要适合以下场景：

- 经常使用 Virtuoso Layout EXL / EMX 进行无源器件仿真；
- 需要重复建立顶层测试版图和 EMX 端口；
- 需要处理电感、电容、变压器及中心抽头变压器；
- 希望减少端口命名、Reference 配对和结果文件选择错误；
- 希望将端口准备、仿真和结果计算放在同一个界面中完成。

## 使用前需要注意

- AutoPorts 不会安装 Cadence、PDK、EMX 或相关许可证；
- 插件沿用用户已经配置好的 EMX model、工艺文件和仿真设置；
- `Generate pins` 会清除所选顶层原有的顶层 pin，再根据已启用的源实例重新生成，首次使用建议先在测试版图中验证；
- 插件负责自动化流程，但参考层选择、回流路径、模型范围和最终电磁性能仍需要设计人员确认；
- AutoPorts 2.0 的安装可不依赖 Python，但 EM 结果计算、绘图预处理和部分运行控制功能需要可用的 Python 3.8+ 环境。

## 总结

AutoPorts 并不是替代 EMX，而是帮助用户更高效地使用 EMX。

它把原本分散在版图、端口表、仿真窗口和结果脚本中的重复操作连接起来，让整个流程从：

> 手工建 pin、手工配端口、手工找结果、手工跑计算

变成：

> 配置一次，按流程生成、仿真和分析

如果你正在使用 Virtuoso / EMX 开发电感、电容或变压器，希望 AutoPorts 能帮助你减少重复操作，把更多时间放在器件设计和结果分析上。
