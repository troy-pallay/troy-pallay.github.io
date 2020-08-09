---
layout: post
title: "DC Biasing in Ceramic Capacitors"
author: "Troy Pallay"
date: 2020-08-09 12:30:00 -0400
categories: hardware experiment breadboard
---

When selecting capacitors to use in a board design, multilayer ceramic chip capacitors (MLCCs) are often a good choice. These non-polarized caps are available in relatively high capacitance values and for a cheap price. Their non-polarity can make them more suitable for AC applications, as opposed to their electrolytic brethren. They also tend to have a *much* longer shelf-life. However, there are drawbacks to many ceramic capacitors that you will have to consider before soldering them to your board and calling it a day.

MLCCs are divided into classes based on trade-offs between their stability under different operating conditions and their volumetric efficiency. This is a bit of an oversimplification, but the general idea is that ceramics that offer very high stability in their capacitance over a range of temperatures and voltages require a dialectric material that does not permit large capacitance in a small package (Class I). Conversely, ceramics that use a dialectric that does allow for higher capacitancies will probably not be as stable accross those same temperature and voltage ranges (Class II). Again, the specifics of this are much more complex and nuanced than the few sentences I have spent explaining it. As always, if you want a deep dive into the history, physics, and nomenclature, the wikipedia article on the **[Ceramic Capacitor](https://en.wikipedia.org/wiki/Ceramic_capacitor)** is an interesting read.

In this post however, I specifically want to explore the experimental relationship between the DC voltage bias and capacitance while using caps that utilize an X7R dialectric. This type of dialectric is very common and falls under Class II in the categorization of ceramic caps. As we will soon see, some of these ceramic capacitors have quite a high voltage rating. To the unaware engineer (me), they might assume that this voltage rating signifies a threshold beyond which the capacitor will fail. While this may be true, the reality is that depending on the construction of the component, we may start to see unwanted behavior well before this threshold. And, depending on the sensitivity of our application, this behavour may be catastrophic to our circuit. The results of this experiment should show that going to digikey and selecting a 2.2uF ceramic cap may not be as cut and dry as you might believe.

For our setup, I have created a simple RC circuit to subject a ceramic cap to different voltages (Vapp). By placing the probe of my oscilloscope on the capacitor, we should be able to measure the time constant displayed by the circuit as it charges. This will allow us to calculate the experimental capacitance exhibited by the MLCC. I will be using a 10k resistor to charge the capacitor, and a pushbutton to discharge the circuit so that we can measure the charging ramp again. When discharging the circuit, I press the push-button in order to short the two leads of the capacitor. I have also set up my oscilloscope to trigger on the rising edge. The simple circuit is shown below:

![DC Biasing in Ceramic Capacitors Experiment Schematic](/assets/posts/dc-biasing-in-ceramic-capacitors-01.png)

The capacitors that I am using are rated for 50V. I would have liked to test them over their complete range, but for today I can only test them up to 30V. Even though this does not cover their supposed capability, we should still be able to clearly see the trend I am trying to highlight.

So as I am sure you know, the time constant of an RC circuit is represented as:

𝜏 = R * C

Since we are measuring the rise time of our circuit, this 𝜏 value is also equal to the period of time it takes for the capacitor to charge from zero volts to 63.2% of the applied voltage (Vapp). I will actually be measuring the time it takes to charge from 20% of Vapp up to 63.2% of the remaining voltage rise. For example, for a Vapp of 5V, I would like the start of my interval at 1V. And since 63.2% of the remaining voltage rise between 1V and Vapp is 2.528V, the end of my interval will be at 3.528V.

Vend = .632 * (Vapp - Vstart) + Vstart

Having measured the value of 𝜏 in the circuit, we can use the original equation for the time constant and the resistor value to find the capacitance. Hopefully it will always be 2.2uF? Let's find out.

This table shows the voltage rise over which I am measuring the time constant. I calculate the experimental capacitance as well, using the measured resistance R = 9.77kOhms.
<hr>

| Vapp | Vstart | Vend | 𝜏 | Cexp |
|:----:|:------:|:----:|:-:|:----:|
| 5V | 1V | 3.528V | 21.8ms | 2.17uF |
| 10V | 2V | 7.056V | 21.7ms | 2.16uF |
| 15V | 3V | 10.584V | 19.6ms | 1.95uF |
| 20V | 4V | 14.112V | 15.8ms | 1.57uF |
| 25V | 5V | 17.640V | 13.6ms | 1.35uF |
| 30V | 6V | 21.168V | 10.8ms | 1.08uF |

<hr>

I captured a few of the rise times on the oscilloscope:

![Rise time of capacitor with DC bias of 5V](/assets/posts/dc-biasing-in-ceramic-capacitors-02.png)

![Rise time of capacitor with DC bias of 20V](/assets/posts/dc-biasing-in-ceramic-capacitors-03.png)

![Rise time of capacitor with DC bias of 30V](/assets/posts/dc-biasing-in-ceramic-capacitors-04.png)

As you can see from the waveforms, the rise time of the 30V bias is significantly quicker than that of the 5V bias. I have also put the results in graph form:

![Experimental Capacitance vs DC Bias](/assets/posts/dc-biasing-in-ceramic-capacitors-05.png)

And here is the final graph showing the percentage of the ideal capacitance that is actually present at different bias voltages.

![Percentage of Ideal Capacitance vs DC Bias](/assets/posts/dc-biasing-in-ceramic-capacitors-06.png)

I have done this experiment a few times with multiple capacitors, but this data is only for a single one. The trend seems to be about the same for me with any cap I choose. Since I am not taking data points from many many trials and averaging them all, this data is far from conclusive. Luckily, there are researches and companies who have written extensively on this. If you'd like to read a more, I found this post by KEMET that explains things in a bit more detail: https://ec.kemet.com/blog/mlcc-dielectric-differences/.

Send me an email at mail@troypallay.com if you learned something in this post. Also please let me know if you find something incorrect!
-Troy