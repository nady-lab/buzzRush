# buzz rush

a little bee game for the **STM32 Nucleo-F031K6**, written in C with PlatformIO. you're a bee, there are eight flowers, and only one of them is the right one. pick correctly and your score goes up, pick wrong and you lose a life (you get five).

<!-- add a photo or gif of the board running here, e.g. ![buzz rush gameplay](assets/demo.gif) -->

## a lil about this
my first time ever wiring a microcontroller, and somehow my favorite part of my CS course<br/>
i wrote the movement, drew the sprites in LibreSprite, added sound, and got a real taste of hardware<br/>
it ended up covering GPIO input, SPI display control, UART, timers, interrupts and sound generation, all in one project

## what you need

an STM32 Nucleo-F031K6 (the on-board ST-Link handles flashing and serial over the same USB cable), a 128 x 160 SPI LCD, five push buttons, a green and a red LED, a small buzzer, and [PlatformIO](https://platformio.org/) (VS Code extension or CLI).

| part | pin |
|------|-----|
| up button | PA8 |
| down button | PA11 |
| left button | PB5 |
| right button | PB4 |
| pause button | PB0 |
| green LED | PA9 |
| red LED | PA10 |
| buzzer | PB1 (Timer14 PWM) |
| LCD reset / CS / D/C | PA3 / PA4 / PA6 |
| UART TX / RX | PA2 / PA15 (ST-Link virtual COM port) |

buttons are active low with the internal pull-ups on, so wire each one between its pin and GND. the LCD clock and data go over SPI1.

## build and flash

```bash
git clone https://github.com/nady-lab/buzzRush.git
cd buzzRush

# build
pio run

# flash the board over USB (ST-Link)
pio run -t upload
```

the project uses the `nucleo_f031k6` environment with the CMSIS framework, so there are no extra libraries to install.

the game prints debug messages (score, lives, events) over UART at **9600 baud**:

```bash
pio device monitor -b 9600
```

## controls

| input | action |
|-------|--------|
| up / down / left / right | move the bee (up also starts and restarts the game) |
| pause button (PB0) | pause / resume |
| `Q` / `q` in the serial monitor | quit |

## how it works

the start screen waits for you to press up. each round picks a random correct flower (`rand() % 8`) and scatters random flower sprites around the screen. touch the right one and you get the green LED and a happy jingle, touch a wrong one and you get the red LED and a sad sound. run out of lives and you get a game over tune, then a full reset if you want another go. pause freezes everything and puts it back exactly how it was when you resume.

### under the hood:

**game loop:** an outer loop handles restarts and an inner loop runs the active game.<br/>
**timing:** a 1 ms SysTick interrupt drives delays and game timing.<br/>
**collisions:** bounding box checks with `isInside()` detect when the bee overlaps a flower, then the score or lives update, the sound and LED play, and the flowers get repositioned.<br/>
**rendering:** sprites and pixels are drawn over SPI, and the screen only redraws when something moves, which cuts down flicker.<br/>
**sound:** Timer14 runs at 65536 Hz and the note frequency sets the auto-reload value, giving a 50% duty cycle PWM on PB1.

## making your own sprites

sprites live in `assets/sprites` as `.bmp` files. `assets/bmptoh.py` turns a bitmap into the 16-bit pixel values used in `src/sprites.h`:

```bash
pip install pillow
python assets/bmptoh.py assets/sprites/bee/beeUp.bmp
```

## what's next

difficulty scaling (faster movement, more flowers), saving high scores to non-volatile memory, frame-based animation, background music, and maybe moving to FreeRTOS for task management.
