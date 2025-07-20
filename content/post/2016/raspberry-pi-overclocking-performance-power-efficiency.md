---
title: "Performance vs Power: Overclocking the Raspberry Pi"
description: "A study on the relationship between Raspberry Pi CPU frequency and power consumption, efficiency and voltage."
date: 2016-08-10T15:42:40+08:00
categories:
  - Tinkering
tags:
  - Raspberry Pi
---

## Preparation

- Raspberry Pi 2 Model B.
- A power meter bought on Taobao for 37 RMB.
- Power supply from my Nubia Z9 Max, 5V 1.5A.

## Testing Method

- Modify `config.txt` to overclock via `arm_freq`.
- Change `scaling_governor` from `powersave` to `performance` to ensure the CPU runs at maximum frequency.
- Use `aircrack-ng -S` to measure power consumption and performance under full load.

## Data Table

| Frequency (MHz) | Idle Power (W) | Full Load Power (W) | Performance (k/s) | Performance/Freq | Performance/Power |
|-----|-----|-----|-----|-----|-----|
|600|1.75|2.31|339|0.56500|146.75|
|700|1.82|2.65|397|0.56714|149.81|
|800|1.83|2.81|455|0.56875|161.92|
|900|1.84|2.94|514|0.57111|174.83|
|1000|1.83|3.08|571|0.57100|185.39|
|1050|1.84|3.13|600|0.57143|191.69|
|1100|1.89|3.37|629|0.57182|186.65|

Note: At 1100MHz, the system could not run stably under full load at default voltage. After testing, setting `over_voltage=2` made it stable.

## Conclusion

- With `governor` set to `powersave`, idle power consumption is 1.75W.
- With `governor` set to `performance`, idle power consumption remains almost unchanged across frequencies, around 1.84W.
- Performance per frequency slightly improves as frequency increases.
- With `over_voltage=0`, performance per watt improves as frequency increases.

**Therefore, to improve performance per watt, we can overclock as much as possible without increasing voltage, provided the system remains stable.**

Moreover, even with overvolting at 1100MHz, its performance per watt is still better than 1000MHz at default voltage. Methods to improve low-frequency performance per watt are discussed in the postscript.

## Postscript

These are experiments I did after completing the experiments above.

### Unexpected Findings

- After overclocking, even when the CPU frequency stays at 600MHz, there is a slight increase in power consumption and performance under full load.
- At maximum frequency 1000MHz but actual frequency 600MHz, full load power is 2.33W, performance is 341k/s. Idle power remains unchanged (original idle power 2.31W, performance 339k/s).

**This indicates that overclocking slightly increases full-load power consumption and performance even at the lowest frequency (600MHz).**

### Stability Testing with Overvolting

```
arm_freq=1050
over_voltage=3
```

- No difference in idle power consumption with `powersave` governor.
- With `performance` governor, idle power is 1.91W, full load power is 3.38W, performance is 599k/s.
- At the same frequency, overvolting increases power consumption, does not improve performance, causes more heat, and reduces performance per watt.

**Thus, if the system is stable, overvolting is not recommended.**

### Methods to Improve Performance per Watt

An unexpected discovery: I set `over_voltage` to a negative value, hoping to undervolt. I thought it wouldn't boot, but surprisingly it worked! This means we can improve performance per watt by lowering the voltage.

```
arm_freq=800
over_voltage=-3
```

With `performance` governor, idle power is 1.78W, full load power is 2.63W, performance is 456k/s, performance per watt = 173.38 (original idle power 1.83W, full load power 2.81W, performance 455k/s).

This suggests that as long as the system remains stable, we can lower the voltage to improve performance per watt. (In fact, my laptop i7-4710MQ CPU has been undervolted by 0.1V for a long time.)

**Therefore, if you find your Raspberry Pi’s performance sufficient (i.e., you've reached your target frequency), you can reduce the voltage as much as possible while keeping the system stable to improve performance per watt.**