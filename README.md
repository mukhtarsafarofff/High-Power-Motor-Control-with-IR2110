(EN) The IR2110 is a popular h-bridge half driver. It is designed to drive high voltage MOSFET and IGBT circuits. It does this without the use of p-channel MOSFETs or photovoltaic opto-couplers. The circuit generates HIGH side MOSFET gate voltage from a diode, bootstrap capacitor combination. The IR2110 is an 14-pin DIP IC and I used 2 for a complete h-bridge. The schematic to each half is identical. I used the 15V MOSFET/IGBT gate voltage with a 5-volt regular to supply the digital voltage (VDD) of 5V. The motor power supply is separate and is limited by the MOSFETs used and motor voltage up to 500-volts. IR2110 illustration :
1) Pin 9 or VDD is the digital logic positive voltage range 3-15V. This enables the use of CMOS or LS TTL logic.
2) The VDD voltage depends on the type of logic connected to the inputs. I used a 5V Arduino so I connected 5V from a voltage regulator.
3) If using 3V logic such as Raspberry Pi GPIO connect to 3V. If using CMOS logic at say 12V connect VDD to 12V.
4) Pin 10 or HIN controls the output on pin 7 labeled HO that controls the HIGH side MOSFET. This input must be pulse-width-modulated as explained below.
5) Pin 12 or LIN controls the LOW side MOSFET connected to pin 1 LO. A HIGH outputs 10-15V on pin 1 turning on the LOW side MOSFET.
6) Pin 11 or SD will disable all outputs when HIGH. This is mostly connected to digital ground.
7) Pins 10, 11, and 12 all have Schmitt trigger inputs and internal pull down resistors.
8) Pin 13 or VSS is the digital power supply negative ground. This is often connected to motor supply HV voltage ground COM or pin 2.
9) Pins 8 and 14 no connection.
Now comes the tricky part the output. The IR2110 uses to "bootstrap" capacitor to supply the gate turn on voltage for MOSFET Q1. MOSFETs Q1 and Q2 are connected in a "totem pole" configuration. Pin 7 supplies the gate drive voltage for HIGH side MOSFET Q1. Pin 1 supplies the gate drive voltage for LOW MOSFET Q2. Notice the internal "totem pole" outputs from LO and HO within the IR2110. VCC pin 3 is 10-15V gate drive voltage. Each have internal under voltage (UV) detectors. When the gate voltage drops below ~8.5V outputs are turned off. Capacitor Cb is charged through a diode from VCC to 15V. Internal transistor "A" will switch the capacitor voltage to gate Q1. Cb can no longer charge from VCC until Q1 is turned off. It is the charge-discharge of Cb connected across Q1 that keeps Q1 turned on. RL represents the load (motor for example) and the LOW side MOSFET turned on at the other half of the h-bridge. Capacitor Cb in my circuit is a 0.22uF or 220n. It is charged to 15V through diode D1. In my case I used a 1N4007. For high speed switching a UF4007 high speed diode. must be used. D1 also serves to block the motor drive voltage from 15V VCC. The capacitor charges to 15V across VB and VS when Q1 is off. VS is connected to Q1 source connection. When a HIGH is input to HIN the voltage at VB is switched to pin HO turning on Q1. Once Q1 is turned on the motor voltage appears at Q1 source preventing any further charging of Cb. Cb will discharge the rate related to MOSFET leakage, etc. Most internet sites connect a 1K resistor across Q1 gate-source, I removed it. They also connected a 22uF electrolytic across Cb I removed that. They also failed to bypass VCC so I added 22uF capacitor to ground. I removed a 22uF bypass capacitor across VDD as not needed. Diode D2 was a 1N4007 in the test circuit. I used a 1N4148 high speed diode in the final circuit. MOSFETs tested: IRFZ40 - VDS = 60V; ID = 36A and IRFZ44N - VDS = 55V; ID = 49A. The following from IR application note AN978-b. VDD is 3-15V, the output stage is implemented either with two N-Channel MOSFETs in totem pole configuration. Each IR2110 MOSFET can sink or source gate currents up to 2A, depending on the MGD. MGD stands for MOS-gate-driver. To quote in regards to VB, VS, and HO: The gate charge for the high side MOSFET is provided by the bootstrap capacitor which is charged by the 15V supply through the bootstrap diode during the time when the device is off. The use of pulses greatly reduces the power dissipation associated with the level translation. The pulse discriminator filters the set/reset pulses from fast dv/dt transients appearing on the VS node so that switching rates as high as 50V/ns in the power devices will not adversely affect the operation of the MGD. This channel has its own undervoltage lockout ( on some MGDs) which blocks the gate drive if the voltage between VB and VS, i.e., the voltage across the upper totem pole is below its limits (typically 8.7/8.3V). The operation of the UV lockout differs from the one on VCC in one detail: the first pulse after the UV lockout has released the channel changes the state of the output. The high voltage level translator circuit is designed to function properly even when the VS node swings below the COM pin by a voltage indicated in the data sheet, typically 5 V. The following in my view is nonsense. I used a single 15V supply for VCC and the motor supply voltage. 12V should also work. It worked just fine with my h-bridge. Gate voltage must be 10-15V higher than the drain voltage. Being a high side switch, such gate voltage would have to be higher than the rail voltage, which is frequently the highest voltage available in the system. Code is here :

 #define INAHI 5 // pwm input A HI MOSFET
#define INBHI 6 // pwm input B HI MOSFET
#define INALO 7 // input A LO MOSFET 
#define INBLO 8 // input B LO MOSFET

#define POT 0

#define SW1 2
#define SW2 3

int val;

void setup() {
  pinMode(INAHI, OUTPUT);
  pinMode(INBHI, OUTPUT);
  pinMode(INALO, OUTPUT);
  pinMode(INBLO, OUTPUT);

  digitalWrite(INAHI, LOW);
  digitalWrite(INBHI, LOW);
  digitalWrite(INALO, LOW);
  digitalWrite(INBLO, LOW);

  pinMode(SW1, INPUT);
  pinMode(SW2, INPUT);

  pinMode(SW1, INPUT_PULLUP);
  pinMode(SW2, INPUT_PULLUP);
}

void loop() {

  analogWrite(INAHI, 0);
  analogWrite(INBHI, 0);
  
  // INAHI HIGH and INBLO HIGH
  // INALO LOW and INBHI LOW
  while (!digitalRead(SW1))   {
    digitalWrite(INALO, LOW);
    digitalWrite(INBHI, LOW);
    val = analogRead(POT)/ 4 ;
    analogWrite(INAHI, val);
    digitalWrite(INBLO, HIGH); 
  }

  // INAHI LOW and INBLO LOW
  // INALO HIGH and INBHI HIGH
  while (!digitalRead(SW2))   {
    digitalWrite(INAHI, LOW);
    digitalWrite(INBLO, LOW);
    val = analogRead(POT)/ 4;
    analogWrite(INBHI, val);
    digitalWrite(INALO, HIGH); 
  }

} // end loop


(DE)  Der IR2110 ist ein beliebter Halbbrückentreiber. Er ist dafür ausgelegt, Hochspannungs-MOSFET- und IGBT-Schaltungen zu steuern. Dies geschieht ohne die Verwendung von P-Kanal-MOSFETs oder photovoltaischen Optokopplern. Die Schaltung erzeugt die Gate-Spannung für den HIGH-Side-MOSFET aus einer Kombination von Diode und Bootstrap-Kondensator
