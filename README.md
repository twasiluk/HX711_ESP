#HX711 raw reading library for ESP-8266

Works with Arduino IDE for ESP-8266. Learn more about [installing Arduino Libraries](https://www.arduino.cc/en/Guide/Libraries).

Initially based on code from [http://freedom2000.free.fr](). Read the original author's [post at esp8266.com](http://www.esp8266.com/viewtopic.php?f=29&t=4100).

#### HX711  Strain gauge wiring

The gauges are basically two resistors that change value when
the gauge is strained. One gets longer, the other shorter and
their R difference changes one way or another. You usually get
two gauges mounted in opposition so that the effects are cumulated.
The HX711 will measure the V difference between the two Rs midpoints.

Note that the power consumption of a reading is essentially due to
the power dissipated in the gauge's Rs through the voltage applied
across E-/E+. Gauges Rs are typically in the 1k ohms range. One of
the two Rs may be fixed, although you won't be able to measure it..
<pre>
              RA1                      RA2
            v^v^v^v--------+---------v^v^v^v
           /              (A+)              \
   (E-)---+             [Delta V]            +---(E+)
           \  RB2         (A-)         RB1  /
            v^v^v^v--------+---------v^v^v^v
</pre>
When power is applied by HX711 between E- and E+, if the bridge is
perfectly equilibrated (RA1==RA2==RB1==RB2), V between A+ and A- is 0.

When the A gauge is applied a strain, RA1 will increase while RA2 will
decrease, offsetting up the A+ voltage measured by the HX711.
If at the same time B gauge (mounted in opposite to gauge A) is strained,
RB1 will increase while RB2 will decrease, offsetting down A-, and further
increasing the voltage difference measured by HX711 between A- and A+.

Vendor site: http://www.aviaic.com/enindex.asp
Detailed Spec at http://hivetool.org/w/images/d/d2/Hx711_english.pdf

#### HX711 serial protocol:

Pin PD_SCK and DOUT are used for data retrieval,
input selection, gain selection and power down controls.

When output data is not ready for retrieval, digital
output pin DOUT is HIGH.
Serial clock input PD_SCK should be low.
When DOUT goes to LOW, it indicates data is ready for retrieval.
By applying 25~27 positive clock pulses at the PD_SCK pin,
data is shifted out from the DOUT output pin.
Each PD_SCK pulse shifts out one bit, starting with
the MSB bit first, until all 24 bits are shifted out.
The 25th pulse at PD_SCK input will pull DOUT pin back
to high.

Input and gain selection is controlled by the number of
the input PD_SCK pulses as per table below:
<pre>
 +--------------+---------+------+
 | PD_SCK       | Channel | Gain |
 | Pulses Input |         |      |
 +--------------+---------+------+
 |  25          |   A     | 128  |
 |  26          |   B     |  32  |
 |  27          |   A     |  64  |
 +--------------+---------+------+
</pre>
PD_SCK clock pulses should not be less than 25 or more
than 27 within one conversion period, to avoid causing
serial communication error.

#### Reset and Power-Down

When chip is powered up, on-chip power on rest circuitry
will reset the chip.
Pin PD_SCK input is used to power down the HX711.
When PD_SCK Input is low, chip is in normal working mode
When PD_SCK pin changes from low to high and stays at high
for longer than 60 us, HX711 enters power down mode.
When PD_SCK returns to low, chip will reset and enter normal operation mode.
After a reset or power-down event, input selection is default
to Channel A with a gain of 128.
<pre>
ESP-12 pinout
       | ANT++  |
  REST +--------+ TXD
   ADC |        | RXD
 CH_PD |        | GPIO5
GPIO16 |        | GPIO4
GPIO14 |        | GPIO0
GPIO12 |        | GPIO2
GPIO13 |        | GPIO15
   VCC +--------+ GND
</pre>