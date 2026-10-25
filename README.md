# cyw43439-nv

A driver for Infineon's CYW43439, the wireless chip of the Raspberry Pi
Pico W and Pico 2 W, on its gSPI bus.  It powers the chip on, brings its
bus up, downloads the chip's firmware, NVRAM and CLM blob, reads the
firmware's version through its ioctl path, and drives the chip's own
GPIO pins, of which GPIO0 is the Pico W boards' LED.  The Wi-Fi station
and Bluetooth are not in this release.

The package is in the novo-lang tree at `orbit/cyw43439-nv/` and is not
published.

## Using it

A board's device table binds the chip to the driver, the way
`orbit/bsp/raspberrypi/rp2350/pico2w/src/bsp.nv` does:

```novo norun:fragment
    board
        wl_on    23
        wl_dio   24
        wl_cs    25
        wl_clk   29
    devices
        wifi  Cyw43439  from "cyw43439-nv"  on spi0  cs wl_cs  irq wl_dio  en wl_on
```

and a program reaches it as `bsp.board.wifi()`:

```novo norun:needs-pkg
use cyw43439

@tier(embedded)
fn main() [hw, mutate]
    let wifi = bsp.board.wifi()
    if wifi.bring_up() == 0
        var on = true
        while true
            let _ok = wifi.led_set(on)
            on = not on
            hal.timer.delay_ms(500)
```

On the Pico 2 W the board's `hal.gpio` also reaches the chip's pins, so
`examples/intro/blink.nv` blinks the LED with `--target=pico2w`.

| Method | What it does |
|---|---|
| `Cyw43439.attach(bus, pins, w)` | the chip on the board's SPI bus with the chip select `w.cs` and the power pin `w.en`, as the device table builds it; nothing is sent |
| `bring_up()` | powers the chip on through its WL_REG_ON pin, the wiring's `en`, and brings it up; answers `UP` (0) or the step that failed (`FAIL_BUS_TEST` 1 to `FAIL_NO_CS` 13, and `FAIL_NO_POWER` 14 for a wiring with no `en`) |
| `is_up()` | whether `bring_up` has run to its end |
| `version(var out)` | the firmware's version string, the `ver` iovar |
| `gpio_set(pin, on)` | sets the chip's GPIO 0 to 2 through the `gpioout` iovar |
| `gpio_out(pin)` | the level ChipCommon's GPIO output register holds for the pin, read through the backplane |
| `led_set(on)` | `gpio_set(0, on)`, the Pico W boards' LED |
| `device_id()` | devices-nv's `Identifiable`: the chip id (43439) and its revision |

The power pin is the device table's `en`, which devices-nv 0.3.0's
wiring record carries.  The driver is for one chip in a program: its
state is module state.

## The bus

The chip's gSPI is a half-duplex SPI with one data line.  The driver
speaks it over embedded-hal-nv's `SpiBus` and clocks each byte it reads
with `transfer(-1)`: a three-wire controller releases the data line for
it, as the Raspberry Pi PIO driver `rp_pio_spi3` does, and a four-wire
controller clocks out 0xFF.  The chip select is a GPIO pin the driver
drives.  The bring-up's order, its constants and the checks it keeps
are those of Embassy's `cyw43` crate, read for its fixes and not
ported; `src/cyw43439.nv` lists the steps and names each fix.

## The firmware

`blobs/` holds the chip's firmware (`43439A0.bin`, 231077 bytes,
version 7.95.61), its CLM blob (`43439A0_clm.bin`, 984 bytes) and the
Pico W boards' NVRAM (`nvram_rp2040.txt`, the pico-sdk's
`wifi_nvram_43439.h` as text), as Embassy's `cyw43-firmware` carries
them, taken from georgerobotics/cyw43-driver.  They are Infineon's,
redistributed under the **Infineon Permissive Binary License 1.0**,
whose text is `blobs/LICENSE-permissive-binary-license-1.0.txt`: binary
redistribution without modification, with the notice and the
disclaimer, and no reverse engineering.  The driver's own code is under
the package's licence, Apache-2.0.

The manifest's `[assets]` table names the three files, and the build
writes them as the constants of the module `cyw43439_blobs`
([the manifest's `[assets]`](../../docs/manifest.md#assets)).  On a
device they are constant data in the image's flash, about 233 KB.

## Tests

`novo test tests` checks the command words and the blobs on the host.
The driver is measured on a board: the Pico 2 W package's suite,
`orbit/bsp/raspberrypi/rp2350/pico2w/tests/run.sh`, brings the chip up,
reads its version and drives its LED, and reads back the line coverage
of this package's modules: on 2026-10-07 `cyw43_gspi` 119 of 122
lines, `cyw43_ioctl` 91 of 99 and `cyw43439` 171 of 197, every line
not run marked at its site with its reason, a failure path only a
failing chip takes.  The registry's coverage figure is the host's
measure, which cannot reach the bring-up, and is lower by design; the
board's is the one that counts for this package.
