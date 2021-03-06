## 基于STM32的电子钟

单片机：STM32指南者，lcd9341显示屏，红外遥控器。

#### 显示界面

三个显示模式：

- Time show 正常时间显示
- Time set 时间设置模式
- Alarm set 闹钟设置模式

左下角标记模式，右下有Alarmed标识，表明闹钟使能开启。

#### 红外控制

| 遥控板按键 | 效果                 |
| ---------- | -------------------- |
| 电源键     | 熄屏与亮屏           |
| 播放键     | 切换至下一个模式     |
| 回退键     | 切换至上一个模式     |
| test键     | 闹钟使能开关         |
| 加减键     | 数字增大减小         |
| 左右快进键 | 切换小时和分钟设置位 |
| 数字键     | 直接为设置位输入数字 |
| c键        | 设置位归零           |

#### 功能实现

- 时钟断电走时

> STM32 的 RTC 外设（Real Time Clock），实质是一个掉电后还继续运行的定时器。从定时器的角度来说，相对于通用定时器 TIM 外设，它十分简单，只有很纯粹的计时和触发中断的功能；但 从掉电还继续运行的角度来说，它却是 STM32 中唯一一个具有如此强大功能的外设。

它是一个32位的计数器，但在本程序中，并没有作为传统意义上的时间戳来使用，只记录时和分，即使源码中所含的日期有关代码，也并不以1970年为开始，而且范围是2000到2099。但对于简易电子钟来说足够了。

<img src="Doc\RTC外设.jpg" style="zoom: 80%;" />

- 闹钟使能

即通过RTC外设，设置一个闹钟事件，在走时计数器与闹钟寄存器中的值相等时，触发闹钟中断。

- 闹钟值断电保存

在开发过程中，通过RTC外设设置的闹钟寄存器中的值，在断电重新上电后，会被意外修改为6：28，暂时没有发现原因。转而考虑了其他方式实现闹钟断电保存，于是通过flash寄存，在空余的扇区内写入了闹钟值，在一次load初始化后，再次load后，即可实现上下电后保存闹钟的值。