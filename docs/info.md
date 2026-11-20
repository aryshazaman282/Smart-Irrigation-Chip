## How it works

This chip is a low-power irrigation controller for a smart plant monitoring system. It makes the simple, repetitive watering decisions on its own, so a microcontroller (such as an Arduino) can sleep most of the time and only wake up when something needs attention.

It has three parts:

1. **Timer (sensor duty cycling):** a 3-bit counter built from D flip-flops counts 0 to 7 on every clock tick. When it reaches 7, the SENSOR_EN output turns on for one tick. This powers the soil moisture sensor for only 1 out of every 8 ticks, which saves power.
2. **Pump memory (hysteresis):** one D flip-flop remembers whether the pump should be on. It only updates while SENSOR_EN is high: DRY sets it (pump on) and WET clears it (pump off). If neither is active, it keeps its previous value, so the pump keeps running until the soil is properly wet instead of switching on and off rapidly. If DRY and WET are both high, DRY wins.
3. **Dry-run protection and wake:** PUMP_ON is the pump memory ANDed with WATER_OK, so the pump can never run when the water tank is empty. LOW_WATER is the inverse of WATER_OK. WAKE is PUMP_ON OR LOW_WATER, a signal that tells the microcontroller to wake up.

Reset is synchronous: while rst_n is low, an AND gate in front of each flip-flop forces a 0 into it on the next clock edge, clearing the timer and the pump memory.

## How to test

Use a slow clock (1 Hz or manual stepping) and hold reset for a few clock cycles first.

1. Set ui[2] (WATER_OK) low. LOW_WATER and WAKE go high, PUMP_ON stays low.
2. Set ui[2] high. LOW_WATER and WAKE go low.
3. Watch uo[1] (SENSOR_EN): it pulses high for one clock out of every 8.
4. Set ui[0] (DRY) high. On the next SENSOR_EN pulse, PUMP_ON and WAKE go high.
5. Set ui[0] low. PUMP_ON stays high (hysteresis).
6. Set ui[1] (WET) high. On the next SENSOR_EN pulse, PUMP_ON and WAKE go low.
7. With PUMP_ON high, set ui[2] low. PUMP_ON goes low immediately (dry-run protection).

## External hardware

For testing on the demo board, the DIP switches and LEDs are enough. In the full system: DRY and WET come from a capacitive soil moisture sensor through a microcontroller or comparators, WATER_OK comes from a float switch in the water tank, PUMP_ON drives a relay module that switches a 5V water pump, SENSOR_EN powers the soil sensor, and WAKE connects to a wake-up pin on the microcontroller.
