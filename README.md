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
Now comes the tricky part the output. The IR2110 uses to "bootstrap" capacitor to supply the gate turn on voltage for MOSFET Q1. MOSFETs Q1 and Q2 are connected in a "totem pole" configuration. Pin 7 supplies the gate drive voltage for HIGH side MOSFET Q1. Pin 1 supplies the gate drive voltage for LOW MOSFET Q2. Notice the internal "totem pole" outputs from LO and HO within the IR2110. VCC pin 3 is 10-15V gate drive voltage. Each have internal under voltage (UV) detectors. When the gate voltage drops below ~8.5V outputs are turned off. Capacitor Cb is charged through a diode from VCC to 15V. Internal transistor "A" will switch the capacitor voltage to gate Q1. Cb can no longer charge from VCC until Q1 is turned off. It is the charge-discharge of Cb connected across Q1 that keeps Q1 turned on. RL represents the load (motor for example) and the LOW side MOSFET turned on at the other half of the h-bridge. Capacitor Cb in my circuit is a 0.22uF or 220n. It is charged to 15V through diode D1. In my case I used a 1N4007. For high speed switching a UF4007 high speed diode must be used. D1 also serves to block the motor drive voltage from 15V VCC. The capacitor charges to 15V across VB and VS when Q1 is off. VS is connected to Q1 source connection. When a HIGH is input to HIN the voltage at VB is switched to pin HO turning on Q1. Once Q1 is turned on the motor voltage appears at Q1 source preventing any further charging of Cb. Cb will discharge the rate related to MOSFET leakage, etc. Most internet sites connect a 1K resistor across Q1 gate-source, I removed it. They also connected a 22uF electrolytic across Cb I removed that. They also failed to bypass VCC so I added 22uF capacitor to ground. I removed a 22uF bypass capacitor across VDD as not needed. Diode D2 was a 1N4007 in the test circuit. I used a 1N4148 high speed diode in the final circuit. MOSFETs tested: IRFZ40 - VDS = 60V; ID = 36A and IRFZ44N - VDS = 55V; ID = 49A. The following from IR application note AN978-b. VDD is 3-15V, the output stage is implemented either with two N-Channel MOSFETs in totem pole configuration. Each IR2110 MOSFET can sink or source gate currents up to 2A, depending on the MGD. MGD stands for MOS-gate-driver. To quote in regards to VB, VS, and HO: The gate charge for the high side MOSFET is provided by the bootstrap capacitor which is charged by the 15V supply through the bootstrap diode during the time when the device is off. The use of pulses greatly reduces the power dissipation associated with the level translation. The pulse discriminator filters the set/reset pulses from fast dv/dt transients appearing on the VS node so that switching rates as high as 50V/ns in the power devices will not adversely affect the operation of the MGD. This channel has its own undervoltage lockout ( on some MGDs) which blocks the gate drive if the voltage between VB and VS, i.e., the voltage across the upper totem pole is below its limits (typically 8.7/8.3V). The operation of the UV lockout differs from the one on VCC in one detail: the first pulse after the UV lockout has released the channel changes the state of the output. The high voltage level translator circuit is designed to function properly even when the VS node swings below the COM pin by a voltage indicated in the data sheet, typically 5 V. The following in my view is nonsense. I used a single 15V supply for VCC and the motor supply voltage. 12V should also work. It worked just fine with my h-bridge. Gate voltage must be 10-15V higher than the drain voltage. Being a high side switch, such gate voltage would have to be higher than the rail voltage, which is frequently the highest voltage available in the system. Code is here :

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


(DE)  Der IR2110 ist ein beliebter Halbbrückentreiber. Er ist dafür ausgelegt, Hochspannungs-MOSFET- und IGBT-Schaltungen zu steuern. Dies geschieht ohne die Verwendung von P-Kanal-MOSFETs oder photovoltaischen Optokopplern. Die Schaltung erzeugt die Gate-Spannung für den HIGH-Side-MOSFET aus einer Kombination von Diode und Bootstrap-Kondensator. Der IR2110 ist ein 14-poliges DIP-IC, und ich habe zwei davon für eine vollständige H-Brücke verwendet. Der Schaltplan für jede Hälfte ist identisch. Ich habe die 15V MOSFET/IGBT-Gate-Spannung mit einem 5-Volt-Regler verwendet, um die digitale Spannung (VDD) von 5V zu liefern. Die Motorstromversorgung ist separat und wird durch die verwendeten MOSFETs und die Motorspannung bis zu 500 Volt begrenzt. Illustration des IR2110: 
1) Pin 9 oder VDD ist die positive Logikspannung im Bereich von 3-15V. Dies ermöglicht die Verwendung von CMOS- oder LS-TTL-Logik.
2) Die VDD-Spannung hängt von der Art der Logik ab, die mit den Eingängen verbunden ist. Ich habe einen 5V-Arduino verwendet, daher habe ich 5V von einem Spannungsregler angeschlossen. Wenn Sie 3V-Logik wie Raspberry Pi GPIO verwenden, schließen Sie VDD an 3V an.
3) Wenn Sie CMOS-Logik mit beispielsweise 12V verwenden, schließen Sie VDD an 12V an.
4) Pin 10 oder HIN steuert den Ausgang an Pin 7, bezeichnet als HO, der den HIGH-Side-MOSFET steuert. Dieser Eingang muss wie unten erläutert pulsweitenmoduliert werden.
5) Pin 12 oder LIN steuert den LOW-Side-MOSFET, der an Pin 1 LO angeschlossen ist. Ein HIGH gibt 10-15V an Pin 1 aus und schaltet den LOW-Side-MOSFET ein.
6) Pin 11 oder SD deaktiviert alle Ausgänge, wenn er auf HIGH gesetzt ist. Dieser ist meistens mit der digitalen Masse verbunden.
7) Pins 10, 11 und 12 haben alle Schmitt-Trigger-Eingänge und interne Pull-Down-Widerstände.
8) Pin 13 oder VSS ist die negative Masse der digitalen Stromversorgung. Diese ist oft mit der Masse der Motorversorgung (HV) verbunden, COM oder Pin 2.
9) Pins 8 und 14 sind nicht verbunden.
Nun kommt der knifflige Teil: der Ausgang. Der IR2110 verwendet einen "Bootstrap"-Kondensator, um die Einschaltspannung für das Gate des MOSFET Q1 bereitzustellen. Die MOSFETs Q1 und Q2 sind in einer "Totem-Pole"-Konfiguration verbunden. Pin 7 liefert die Gate-Ansteuerspannung für den HIGH-Side-MOSFET Q1. Pin 1 liefert die Gate-Ansteuerspannung für den LOW-Side-MOSFET Q2. Beachten Sie die internen "Totem-Pole"-Ausgänge von LO und HO innerhalb des IR2110. Der VCC-Pin 3 ist eine Gate-Ansteuerspannung von 10-15V. Jeder hat interne Unterspannung (UV)-Detektoren. Wenn die Gate-Spannung unter etwa 8,5V fällt, werden die Ausgänge abgeschaltet. Der Kondensator Cb wird über eine Diode von VCC auf 15V aufgeladen. Der interne Transistor "A" schaltet die Kondensatorspannung zum Gate von Q1. Cb kann sich nicht mehr von VCC aufladen, bis Q1 ausgeschaltet ist. Es ist das Lade-Entlade-Verhalten von Cb, der über Q1 verbunden ist, das Q1 eingeschaltet hält. RL stellt die Last dar (zum Beispiel einen Motor), und der LOW-Side-MOSFET ist in der anderen Hälfte der H-Brücke eingeschaltet. Der Kondensator Cb in meiner Schaltung ist 0,22 µF oder 220 nF. Er wird über die Diode D1 auf 15V aufgeladen. In meinem Fall habe ich eine 1N4007 verwendet. Für schnelles Schalten muss eine UF4007 Hochgeschwindigkeitsdiode verwendet werden. D1 dient auch dazu, die Motortreiberspannung von der 15V-VCC zu blockieren. Der Kondensator lädt sich auf 15V über VB und VS auf, wenn Q1 ausgeschaltet ist . VS ist mit dem Source-Anschluss von Q1 verbunden. Wenn ein HIGH-Signal an HIN angelegt wird, wird die Spannung an VB auf den Pin HO geschaltet und schaltet Q1 ein. Sobald Q1 eingeschaltet ist, erscheint die Motorspannung am Source-Anschluss von Q1 und verhindert eine weitere Aufladung von Cb. Cb entlädt sich mit einer Rate, die von der Leckstrom des MOSFETs und anderen Faktoren abhängt. Die meisten Internetseiten verbinden einen 1K-Widerstand zwischen Gate und Source von Q1, den habe ich entfernt. Sie haben auch einen 22uF-Elektrolytkondensator parallel zu Cb geschaltet, den habe ich ebenfalls entfernt. Sie haben VCC nicht gebypasst, also habe ich einen 22uF-Kondensator zur Erdung hinzugefügt. . Ich habe einen 22uF-Bypasskondensator parallel zu VDD entfernt, da er nicht benötigt wird. Diode D2 war im Testschaltkreis eine 1N4007. Ich habe in der endgültigen Schaltung eine 1N4148-Hochgeschwindigkeitsdiode verwendet. Getestete MOSFETs: IRFZ40 - VDS = 60V; ID = 36A und IRFZ44N - VDS = 55V; ID = 49A. Das Folgende stammt aus der IR-Anwendungsnote AN978-b. VDD beträgt 3-15V, die Ausgangsstufe wird entweder mit zwei N-Kanal-MOSFETs in Totem-Pole-Konfiguration implementiert. Jeder IR2110 MOSFET kann Gate-Ströme bis zu 2A aufnehmen oder abgeben, abhängig vom MGD. MGD steht für MOS-Gate-Treiber. Zitat zu VB, VS und HO: Die Gate-Ladung für den High-Side-MOSFET wird durch den Bootstrap-Kondensator bereitgestellt, der durch die 15V-Versorgung über die Bootstrap-Diode während der Zeit aufgeladen wird, in der das Gerät ausgeschaltet ist. Die Verwendung von Impulsen reduziert die Leistungsabgabe, die mit der Pegelübersetzung verbunden ist, erheblich. Der Impulsdiskriminator filtert die Set/Reset-Impulse von schnellen dv/dt-Übergängen, die am VS-Knoten auftreten, sodass Schaltgeschwindigkeiten von bis zu 50V/ns in den Leistungsbauteilen die Funktion des MGD nicht negativ beeinflussen. Dieser Kanal hat eine eigene Unterspannungssperre (bei einigen MGDs), die den Gate-Antrieb blockiert, wenn die Spannung zwischen VB und VS, d.h. die Spannung über dem oberen Totem-Pole, unterhalb ihrer Grenzwerte liegt (typischerweise 8,7/8,3V). Die Funktionsweise der UV-Sperre unterscheidet sich in einem Detail von derjenigen bei VCC: : Der erste Impuls nach dem Freigeben der UV-Sperre ändert den Zustand des Ausgangs. Die Hochspannungs-Pegelübersetzungsschaltung ist so ausgelegt, dass sie ordnungsgemäß funktioniert, selbst wenn der VS-Knoten unterhalb des COM-Pins um eine im Datenblatt angegebene Spannung, typischerweise 5V, schwingt. Das Folgende halte ich für Unsinn.  Ich habe eine einzige 15V-Versorgung für VCC und die Motorspannung verwendet.  12V sollte auch funktionieren. Es funktionierte einwandfrei mit meiner H-Brücke. Die Gate-Spannung muss 10-15V höher als die Drain-Spannung sein.  Als High-Side-Schalter muss eine solche Gate-Spannung höher als die Rail-Spannung sein, die häufig die höchste im System verfügbare Spannung ist.

#define INAHI 5
// pwm input A HI MOSFET
#define INBHI 6
// pwm input B HI MOSFET #define INALO 7
// input A LO MOSFET #define INBLO 8
// input B LO MOSFET

#define POT 0

#define SW1 2 #define SW2 3

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
while (!digitalRead(SW1)) {
digitalWrite(INALO, LOW);
digitalWrite(INBHI, LOW); 
val = analogRead(POT)/4;
analogWrite(INAHI, val); 
digitalWrite(INBLO, HIGH); }

// INAHI LOW and INBLO LOW 
//INALO HIGH and INBHI HIGH 
while (!digitalRead(SW2)) 
{ digitalWrite(INAHI, LOW); 
digitalWrite(INBLO, LOW); 
val = analogRead(POT)/ 4;
analogWrite(INBHI, val); 
digitalWrite(INALO, HIGH); }

} 
// end loop
