# PWM-generation-with-STM32F030C8T6
PWM (Pulse Width Modulation) is a type of digital electrical signa; which is periodic in nature, with a triangular waveform. In this guide, I will explain PWM configuration and coding for STM32F030C8T6. 

## What is PWM? 
PWM (Pulse Width Modulation) is a type of digital electrical signal which is periodic in nature, with a triangular waveform. Each PWm graph acts as wave. It has the period, which is the time it takes to repeat the same waveform, and the duty cycle, which is the time for which the signal is a logic 1 by the total time period. 

If $`T_{on}`$ represents the duty cycle, and $`T_{p}`$ represents the period, the the duty cycle is defined by 
``` math
Duty Cycle = \frac{T_{on}}{T_{p}} \cdot 100
```

Frequency is the number of times the waveform repeats in a second. So: 
``` math 
F = \frac{1}{T_{p}}
```
## Application of PWM
These are a few applications of PWM:
* Variable voltage generator: if you apply the PWM signal to electrical components, when the duty cycle is varied, the components act as if they receive analog signal. The voltage that they receive has a linear relation with the duty cycle:
  - 0% duty cycle = 0V
  - 25% duty cycle = 1.25V
  - 50% duty cycle = 2.5V
  - 75% duty cycle = 3.75V
  - 100% duty cycle = 5V
 
This application can be used to control speed of DC motors, nrightness of LEDs, or amplitude of buzzer, etc. 

* Control signal: Some electrical devices analyze the PWM signal to give coressponding output. Changes in the duty cycle is reflected on the output. This application can be used on servr motors, Electronic Sopeed controllers, etc.

## PWM signal generation on STM32 
STM32 CPUs have built-in timer. Timers are built-in circuit in the MCUs which can measure the passing of time. They count up to a certain number and upon reaching that number, they change the value of a certauin registor to indicate that the timer has counted up to that number. The time it takes to increment this count by 1 is deteremined by the clock frequency. If the frequency is 1KHz, it means the time period that will be taken by the timer to increment the value is 1 milisecond. This is how the ```HAL_Delay()``` function works. 

However, depending on the register size, the timer has maximum limit. These are the limits on STM32 MCUs timers:
* 8-bit timer = 256
* 10-bit timer = 1024
* 16-bit timer = 65536
* 32-bit timer = 4294967296

## Configuration 
Here's how to configure the clock frequency:
* After creating the project, enable the HSE clock. Go to System>Core>RCC, then look to the right side of that panel, ther will be a panel where we can configure the RCC. Go to HSE, click the drop down, and choose Crystal/Ceramic Resonator
* Go to Clock Configuration tab, and then set the HCLK to 48 and the APB1 Prescalar to /2
* Go back to Pinout and Configuration tab, and then click the Timer drop down. Select TIM1, and set the Clock Source to Internal Clock and Channel2 to PWM Generation CH2
* Go to Parameters setting, set the Prescalar to 48 - 1 (as the prescalar automatically add 1 to the input value) and Counter Period to 10000 (It's 16-bit, you can choose any value you want, just don't exceed 65536), then generate code

Note that we set the HCLK to 48, and as:  
``` math
Timer Frequency = \frac{Timer Peripheral Clock Frequency}{Prescalar}
```
This means: 
``` math
Timer Frquency = \frac{48}{48} = 1Mhz
```
And 
``` math
PWM Frequency = \frac{Timer Frequency}{ARR value}
```

ARR (Auto Reload Registor) decides the value at which the timer starts counting again. Our ARR value is 10000, so:
``` math
PWM Frequency = \frac{1Mhz}{10000} = 100Hz
```
## Coding
Now, as we already configured the clock, we will start writign code. This is the code for PWM generation: 
``` C
void Set_Duty_Cycle(int DutyCycle) {

	if (DutyCycle < 0) DutyCycle = 0;
	if (DutyCycle > 100) DutyCycle = 100;

	TIM1->CCR2 = (DutyCycle * htim1.Init.Period) / 100;
	HAL_TIM_PWM_Start(&htim1, TIM_CHANNEL_2);
}

 while (1)
  {
	  for (int i = 1; i <= 100; i++){
	  		  Set_Duty_Cycle(i);
	  		  HAL_Delay(20);
	  }

	  for (int i = 100; i >= 0; i--) {
	              Set_Duty_Cycle(i);
	              HAL_Delay(20);
	  }
  }
```

The function ```Set_Duty_Cycle(int DutyCycle)``` limits the duty cycle within the valide range, then change the CCR value. 

``` C
if (DutyCycle < 0) DutyCycle = 0;
if (DutyCycle > 100) DutyCycle = 100;
```
This code limits the duty cyle range by setting it to 0 if it's lesser than 0, and if it's bigger than 100, the code sets it to 100, so the duty cycle stays within the valid range. 

``` C
TIM1->CCR2 = (DutyCycle * htim1.Init.Period) / 100;
```
This code changes the CRR value by calculate using this equation: 
``` math
CCR2 = \frac{Duty Cycle \cdot the Period It May Take To Complete One Duty Cycle}{100}
```
The calculation is done to ensure to set the register to the appropriate value for the desire duty cycle. 

``` C
HAL_TIM_PWM_Start(&htim1, TIM_CHANNEL_2);
```
This code start the PWM signal on TIM1 Channel 2. 

The main loop, increment and decrement the duty cycle. 

``` C
for (int i = 1; i <= 100; i++){
     Set_Duty_Cycle(i);
     HAL_Delay(20);
}
```
This for loop, increment the duty cycle to increase the brightness of the LED gradually, and then delay the code execution by 20 milisecond. 

``` C
for (int i = 100; i >= 0; i--) {
     Set_Duty_Cycle(i);
     HAL_Delay(20);
}
```
This for loop, decrement the duty cycle to decrease the brightness of the LED gradually, and then delay the code execution by 20 milisecond.
