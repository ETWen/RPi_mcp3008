# RPi_mcp3008
RPi_mcp3008 is a library to listen to the MCP3008 A/D converter chip with a RPi.
This library implements the example communication protocol described in the datasheet.
https://www.adafruit.com/datasheets/MCP3008.pdf

Communication is made trhough RPi SPI port using SpiDev
https://github.com/doceme/py-spidev

## Wiring
Connect the SPI data cables in the tables bellow. Choose either CE0# or CE1# to connect to CS.

### RPi SPI GPIOs

| RPi GPIO  | Mode |
|-----------|:-----|
| GPIO 07   | CE1# |
| GPIO 08   | CE0# |
| GPIO 09   | MISO |
| GPIO 10   | MOSI |
| GPIO 11   | SCLK |


### MCP3008 Pinout

| Pin | Description | Pin | Description |
|-----|:------------|:----|:------------|
| 01  |     CH0     | 09  | Vdd - Supply voltage (2.7V - 5.5V) |
| 02  |     CH1     | 10  | Vref - Reference voltage |
| 03  |     CH2     | 11  | AGND - Analog ground |
| 04  |     CH3     | 12  | CLK - SPI Clock (SCLK) |
| 05  |     CH4     | 13  | Dout - Data out (MISO) |
| 06  |     CH5     | 14  | Din - Data in (MOSI) |
| 07  |     CH6     | 15  | CS - Chip select (CE0# or CE1#) |
| 08  |     CH7     | 16  | DGND - Digital ground |

## Usage

RPi_mcp3008 uses the `with` statement to properly handle the cleanup.
```python
import mcp3008
with mcp3008.MCP3008() as adc:
    print adc.read()
```
It's possible instantiate the object normally, but it's necessary to call the close method before terminating the program.
```python
import mcp3008
adc = mcp3008.MCP3008()
print adc.read()
adc.close()
```
The initialization arguments are `MCP3008(bus=0, device=0)` where:
`MCP3008(X, Y)` will open /dev/spidev-X.Y, same as spidev.SpiDev.open(X, Y)

### Methods
Currently there are two implemented methods:
```python
def read(self, mode, norm=False):
    '''
    Returns the raw value (0 ... 1024) of the reading.
    mode is the mode of operation (e.g. mcp3008.CH0)
    norm is a normalization factor, usually Vref
    '''
```

```python
def read_all(self, norm=False):
    '''
    Returns a list with the readings of all the channels and modes
    Order:
    [DF0, DF1, DF2, DF3, DF4, DF5, DF6, DF7,
     CH0, CH1, CH2, CH3, CH4, CH5, CH6, CH7]
    norm is a normalization factor, usually Vref
    '''
```
* The mode argument must be one of 16 listed bellow (CH0 - CH7, DF0 - DF7)
* The norm argument is a normalization factor that rescales the raw data, usually Vref

## MCP3008 Operation modes
MCP3008 has 16 different operation modes:
It can listen to each of the channels individually **Single Ended** or in a pseudo-differential mode **Differential**

| Single Ended | Differential |
|--------------|:-------------|
| CH0  | DF0  (CH0 = IN+; CH1 = IN-) |
| CH1  | DF0  (CH0 = IN-; CH1 = IN+) |
| CH2  | DF0  (CH2 = IN+; CH3 = IN-) |
| CH3  | DF0  (CH2 = IN-; CH3 = IN+) |
| CH4  | DF0  (CH4 = IN+; CH5 = IN-) |
| CH5  | DF0  (CH4 = IN-; CH5 = IN+) |
| CH6  | DF0  (CH6 = IN+; CH7 = IN-) |
| CH7  | DF0  (CH6 = IN-; CH7 = IN+) |
Use the table above as the operation mode when calling `MCP3008.read(mode)`