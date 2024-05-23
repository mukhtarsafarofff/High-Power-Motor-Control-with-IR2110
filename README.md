The IR2110 is a popular h-bridge half driver. It is designed to drive high voltage MOSFET and IGBT circuits. It does this without the use of p-channel MOSFETs or photovoltaic opto-couplers. The circuit generates HIGH side MOSFET gate voltage from a diode, bootstrap capacitor combination. The IR2110 is an 14-pin DIP IC and I used 2 for a complete h-bridge. The schematic to each half is identical. I used the 15V MOSFET/IGBT gate voltage with a 5-volt regular to supply the digital voltage (VDD) of 5V. The motor power supply is separate and is limited by the MOSFETs used and motor voltage up to 500-volts. IR2110 illustration :
1) Pin 9 or VDD is the digital logic positive voltage range 3-15V. This enables the use of CMOS or LS TTL logic.
2) The VDD voltage depends on the type of logic connected to the inputs. I used a 5V Arduino so I connected 5V from a voltage regulator.
3) If using 3V logic such as Raspberry Pi GPIO connect to 3V. If using CMOS logic at say 12V connect VDD to 12V.
4) Pin 10 or HIN controls the output on pin 7 labeled HO that controls the HIGH side MOSFET. This input must be pulse-width-modulated as explained below.
5) Pin 12 or LIN controls the LOW side MOSFET connected to pin 1 LO. A HIGH outputs 10-15V on pin 1 turning on the LOW side MOSFET.
6) Pin 11 or SD will disable all outputs when HIGH. This is mostly connected to digital ground.
7) 
