# CH570/CH572 Low Power Configuration

## Power Consumption

- In sleep mode with RAM retention, the MCU consumes approximately **0.9 µA**.  
- In sleep mode with RAM retention, LSI and RTC enabled, it consumes arnoud **1.2 µA**.
- In shutdown mode with all functions disabled, it consumes about **0.6 µA**.

CH570 and CH572 have identical low-power modes and configuration options.

---

## Pin Configuration

**Do not leave any pins floating.**  
A floating pin with digital input enabled can draw extra current due to the internal Schmitt trigger.

To avoid unnecessary power consumption, you can either:

1. **Enable all internal pull-up resistors:**
    ```c
    R32_PA_PD_DRV = 0;
    R32_PA_PU = 0xFFFFFFFF;
    R32_PA_DIR = 0;
    ```

2. **Use external pull-up resistors.**

3. **Disable all digital functions on the pins:**
    ```c
    R32_PA_PD_DRV = 0;
    R32_PA_PU = 0;
    R32_PA_DIR = 0;
    R16_PIN_ALTERNATE = 0x0FFF;
    ```

After performing one of the above configurations, you can set up pin functions as needed for your application.

---

## LDO Configuration

The CH570 and CH572 include an integrated 5V LDO. Configuration depends on the power supply source:

### If powered via the V5 pin

Set the `RB_PWR_LDO5V_EN` bit:

```c
sys_safe_access_enable();
R16_POWER_PLAN |= RB_PWR_LDO5V_EN;
sys_safe_access_disable();
```

---

### If powered via the VCC pin

Clear the `RB_PWR_LDO5V_EN` bit:

```c
sys_safe_access_enable();
R16_POWER_PLAN &= ~RB_PWR_LDO5V_EN;
sys_safe_access_disable();
```

---

If not configured correctly, the sleep current may increase by approximately 100 µA.  
Although connecting a 1 MΩ resistor between VCC and V5 can help reduce this excess current, this workaround is not recommended.

## Unofficial test results

- Datasheet asked for a 1.5k resistor between VCC and V5. Priliminary tests suggests that it is not necessary to do so. The MCU works fine without.

- `LowPower_Sleep()` in the SDK tweaks battery voltage detection, clock settings and stuff before entering sleep mode, none is necessary. One can simply configure `R8_SLP_POWER_CTRL`, `R16_POWER_PLAN` and then call `__WFI()`. **But a 40 us delay must be added after the MCU wakes up to wait for HSE and PLL to be stable.**
