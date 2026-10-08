# DS3231 RTC (optional)

Without it the clock comes from NTP or your phone each boot. With it the board has the right time immediately.

| DS3231 | Board |
|---|---|
| VCC | 3V3 |
| GND | GND |
| SDA | GPIO 38 (shared with touch) |
| SCL | GPIO 39 (shared with touch) |

Then in `src/config.h` set `RTC_DS3231_ENABLED 1`; leave `RTC_SDA_PIN/RTC_SCL_PIN` at -1 (the touch driver already started
that I2C bus). Address 0x68. The RTC holds UTC and is rewritten on every NTP/phone sync. ZS-042-style modules try to charge
a non-rechargeable CR2032: remove the charge resistor/diode or use a LIR2032.
