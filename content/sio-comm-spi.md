# SPI / Normal mode

In this mode, the serial port acts like the Game Boy serial port.

> Note: While there is some on-board protection to prevent damage during accidents, connecting an 8-bit Game Boy or a GBA in 5V mode to a GBA in 3.3V mode is not recommended.

However, unlike the Game Boy, there are two unit sizes selectable: 8-bit, or 32-bit.

> Note: Try to select one unit size, and use that. While it is possible to easily switch between 8-bit and 32-bit mode, it just creates more problems than it solves.

This mode is strictly for communicating between two systems, or to communicate with an attached peripheral.  
Due to how this mode works electrically, there has to be a mechanism present in the game program to let the player select between hosting the game or connecting to the game, and they should select the correct mode for the communication to work.

<h2 id="sio-comm-spi-setup"><a class="header" href="#sio-comm-spi-setup">
Initial setup
</a></h2>

- Switch the port mode to SIO in [RCNT](registers.md#REG_RCNT), according to the [mode switch procedure](sio-comm.md#sio-comm-port-mode-switch)
- Set the SIO mode to SPI in [SCCNT_L](registers.md#REG_SCCNT_L), bit13 = 0, then 8-bit size (bit12 = 0) or 32-bit size (bit12 = 1)
- Set SO output to high (SCCNT_L bit3 = 1).
  This is not strictly necessary, but it's recommended to be used for flow control. Read about this in a later section.
- Set clock generation mode (in SCCNT_L bit0) to output (1=host), or input (0=client).
  Note: this takes effect immediately! So make sure that you have a way in your game for players to select who is the host, and who connects!
  For this reason, set clock mode to input (bit0 = 0) when the communication is closed.
- Set clock speed (in SCCNT_L bit1, no effect on client).
  You probably want 256kiHz (=0) for multiplayer due to signal integrity issues. 2MiHz mode (=1) is for external peripherals with a short cable that can handle the higher speed.
- (Optional) Enable SIO interrupt (in SCCNT_L bit14 = 1) that gets triggered on transfer completion (IRQ #7).
  You probably want to use this for asynchronous communication, as spinwaiting for transfer completion is a bad idea in the long run.

<h2 id="sio-comm-spi-flow-control"><a class="header" href="#sio-comm-spi-flow-control">
Software flow control
</a></h2>

Official games (including the BIOS) implement software flow control, so the host can know when to start clocking the client. This is necessary, as only one side is allowed to have control over the clock. If you try clocking the client while it's not ready, the data will be lost, or the client will get stuck partially receiving the data due to missed clock pulses.

To combat this, the SO pin is used for flow control in the client side, where the host can read the status of that on its SI pin.

Official games use the "active low" approach, where flow=0 means "ready", and flow=1 means "not ready".


<h2 id="sio-comm-spi-tx-host"><a class="header" href="#sio-comm-spi-tx-host">
Transmit data (host)
</a></h2>

- If flow control is enabled, wait for SI=0 (you can read it from [SCCNT_L](registers.md#REG_SCCNT_L) bit2, or [RCNT](registers.md#REG_RCNT) bit2, depending on the flow control wait method)
  - Make sure to have a timeout for this! Disconnected serial cable reads as SI=1 (not ready).
- Load data to be sent (8-bit data goes into [SCCNT_H](registers.md#REG_SCCNT_H), 32-bit data goes into [SCD32](registers.md#REG_SCD32)).
- Set the START bit (SCCNT_L bit7 = 1)
- Wait for the START bit to be cleared by the hardware
  - If enabled, an interrupt will fire when the START bit clears
- Read the received data from the same register that was written for the transmit data

<h2 id="sio-comm-spi-tx-client"><a class="header" href="#sio-comm-spi-tx-client">
Transmit data (client)
</a></h2>

TODO
