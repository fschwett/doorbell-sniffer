# Doorbell sniffer

## Idea:

I can't hear my doorbell everywhere in my house and I wanted to change that without changing too much of the electrical installation. Also I didn't want to use any external power and instead only use the doorbell's power, thus the term "sniffer".

## Concept:

Doorbells in Germany usually operate using SELV (safety extra low voltage) which I take advantage of. This reduces the need for a big and complicated power supply. In my specific case the transformer is rated 8 VAC. Due to soft regulation characteristics the actual output is 16 VAC if not connected to a load. This has to be taken into account when choosing parts. Using a full bridge rectifier I charge a capacitor bank which then powers a small RF transmitter. The receiver side can just be connected to anything. In my case a small MCU making a buzzer beep.

## Implementation:

In order to choose components it's necessary to know the voltage I'm dealing with along with some other limitations:

The transformer is rated for a maximum current of 2 A and the bell itself might draw up to 1 amp max. With 16 VAC RMS the maximum DC voltage will be

`16 V * 1.414 ≈ 22.6 V`

I decided to use VS-10MQ040-M3 diodes for the rectifier. It's a Schottky diode with a forward voltage of around .5 volts and it's rated for 40 V and 1 A continuous current. It also happened to be what I had on hand. Since connecting this directly to a capacitor would result in a high current spike, a charging resistor is needed.

<img width="344" height="219" alt="image" src="https://github.com/user-attachments/assets/29e36b77-c0fa-4667-9389-d69cd4578e56" />

The RF module I'm using is a TX118S which needs at least 3 VDC to operate and starts transmitting after .4 s if powered with 5 VDC. I went with this module, because it is simple, small, requires no long startup (unlike an MCU) and can operate on up to 24 VDC. This eliminates the need for a voltage regulator, which increases efficiency and reduces heat dissipation and component count.

Knowing that, the capacitor and charging resistor values can be calculated. The resistor has to limit the current so the total load stays below the transformer's 2 A rating. This means the device should stay well below 1 A to avoid overloading the transformer. I used a bit of safety margin and calculated the resistance for .6 A:

`R = U / I = 22.6 V / 0.6 A ≈ 38 Ω`

I didn't have the correct resistor with a high enough power rating though so I used two 68 Ω resistors in parallel resulting in 34 Ω. This will still keep it safe even in worst case scenarios. The power will theoretically be

`P = I^2 * R = (0.6 A)^2 * 34 Ω ≈ 14 W`

Using a 14 W resistor would be overkill though, because the current will only last a fraction of a second and realistically the voltage will drop to around 10 VAC, which leaves me with 13 VDC after rectification. The resistor's parallel setup is enough to handle the current while keeping thermal stress low. It will result in a slightly higher current than the intended .6 A (≈ 0.66 A), but it is still in a safe range.

<img width="324" height="219" alt="image" src="https://github.com/user-attachments/assets/64a08326-7e33-471e-a2b7-b3b4a39704e2" />

The transmitter will draw around 10 mA. To keep it safe I used 15 mA in my calculations. The usable voltage range is from 3 V to 13 V after the bridge resulting in a usable voltage range of 10 V. The needed capacity is then calculated:

`C = (I * t) / ΔU = (0.015 A * 0.4 s) / 10 V = 0.0006 F = 600 µF`

This has no safety margin though. I chose to use three 1000 µF 35 V caps. This gives me plenty of time and safety when it comes to voltage and again it's what I had on hand. Using three of them also gives me a lower ESR, less internal heating and a longer lifespan. If anyone is very quick and only rings for 0.1 s, the caps will be charged to this voltage:

`τ = 34 Ω * 0.003 F = 0.102 s`

`U(t) = U_max * ​(1 − e^(-t/τ)) = 13 V * ​(1 − e^(-0.1 / 0.102)) ≈ 13 V * (1 − 0.375) ≈ 8.1 V`

The remaining usable voltage range is 5.1 V which will give us the following runtime:

`t = (C * ΔU) / I = (0.003 F * 5.1 V) / 0.015 A = 1.02 s`

That's more than enough time and that's the **worst case**. Additionally, there is a 4.7 µF capacitor and a 100 nF ceramic capacitor in parallel with the other three capacitors for filtering.

<img width="328" height="343" alt="image" src="https://github.com/user-attachments/assets/5ba28aa6-18ba-4119-9832-4ae65f3536e1" />

For the actual design I used perfboard. I used two dual screw terminals for power in and out which are connected in parallel. This makes it easier to install. I tried to place the filter caps as close to the module as possible. The module itself is soldered in place parallel to the perfboard. I used two plastic spacers from pin headers to fit it above the resistors and diodes. Also I included a double row of pin headers to choose between the four signals the transmitter can send with a jumper in case I want to change it some day for whatever reason. The jumpers pull the connected pin to ground and that way activate the transmitter. Since it's supposed to transmit immediately after powering up, the input can just be hardwired.

<img width="328" height="255" alt="image" src="https://github.com/user-attachments/assets/36c2932e-e4ca-4fe8-8fd5-8b133fb9a233" />

That's everything that went into transmitter design. The reciever design is not really worth mentioning, it's just an RX480E feeding into an ATTiny 85. I literally used the good ol' blink sketch with an added if-statement. It then feeds an N-channel MOSFET which drives a simple buzzer. Nothing too special. You could also use an ESP32 to send push notifications to a mobile phone but my current version actually didn't even outgrow the breadboarding stage.
