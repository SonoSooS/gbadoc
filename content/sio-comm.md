# Serial Port Communication

Compared to the 8-bit Game Boy, the GBA has a more versatile serial port system, which means that it can be used for more than just basic 2-player multiplayer communication.

Due to having extra serial communication options included, and also having more direct control over all four pins in the port, it can even be made to do things it was not originally designed to do.

Potential uses are, but not limited to:
- 2-player multiplayer in 8-bit SPI mode, just like 8-bit Game Boy systems
- up to 4-player multiplayer in 16-bit Multiplay mode
- external serial modem connection in UART mode
- fast connection to a GameCube in JOYBUS mode
- debug print output (and possibly serial shell input) in UART mode, connected via an USB-to-serial adapter
- external foot pedal / trigger button interrupt input in GPIO mode
- external passive infrared transceiver with reception start interrupt support

The serial port can be configured into these modes:
- [SPI / "Normal" mode](sio-comm-spi.md)
- [UART mode](sio-comm-uart.md)
- [GPIO mode](sio-comm-gpio.md)
- [Multiplay](sio-comm-multiplay.md)
- [JOYBUS](sio-comm-joybus.md)

<h2 id="sio-comm-port-mode-switch"><a class="header" href="#sio-comm-port-mode-switch">
Port mode switch procedure
</a></h2>

Due to the weird design, you should reset the currently selected serial transceiver to its default state before switching modes. Luckily that's easy to do.

- Clear the low 12 bits in [SCCNT_L](registers.md#REG_SCCNT_L) (`SCCNT_L &= 0xF000`) to disable the currently selected transceiver, regardless of which one is actually selected
- Switch to the desired port mode in [RCNT](registers.md#REG_RCNT)
  - Note: a direct write to `RCNT` is recommended to clear the GPIO direction bits.
    While this is not strictly necessary, it's a good practice to write only to the mode bits, with the rest of the bits cleared to zero, so when switching modes, the two sides won't try to drive output pins against eachother, which would lead to a short.
- If wanting to use SPI mode, UART mode, or Multiplay mode, set the transceiver mode first in SCCNT_L bits 12 and 13 via a direct write, with the low 12 bits cleared to 0
- Configure the selected peripheral
- Enable it if it's needed to be functional immediately
