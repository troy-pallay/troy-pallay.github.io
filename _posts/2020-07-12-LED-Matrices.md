---
layout: post
title: "LED Matrices"
author: "Troy Pallay"
date: 2020-07-12 12:23:00 -0400
categories: hardware experiment breadboard arduino
---

_This is an old post migrated from my previous site._

Many years ago I attempted to create a circuit that drove a large number of LEDs mounted in a sign. It essentially consisted of an astable multivibrator made of some discrete components and two groups of LEDs in parallel. My goal was to blink these two banks of LEDs so the sign caught the eye. However, I soon realized that my large number of LEDs were taxing the 9V battery I was using with quite the current draw when they were switched on. That meant that my sign could only be powered for a very short time from a mobile source.

That’s because I wasn’t using an individually addressable LED matrix! Apparently, I wasn’t very bright back then (and neither were my diodes). But not today! I’m much wiser in my ways, and when we couple Arduino magic with a few blinky bois, we can accomplish some beautiful results.

So, let’s get down to brass tacks and check out a circuit diagram to understand just how these savings in current work (shoutout to [EasyEDA](https://easyeda.com) for the quick and easy schematic software!).

<object
  data="/assets/posts/led-matrices-01.pdf"
  width="100%"
  height="500"
  type='application/pdf'>
PDF not viewable</object>

As you can see, I have a very simple setup. For each column shown in the diagram, six LEDs are connected by their cathodes (C[1-6]). For each row, six LEDs are connected by their anodes (A[1-6]). I have also shown the pin on the Arduino to which I have connected each net of either anodes or cathodes. By connecting the LEDs in this way, we have created a matrix. And this is important because it means we can think of each LED in this matrix as having its home at an address. For example, the diode in the top left of the diagram is connected to cathode grouping 1 as well as anode grouping 6. Let’s think of this component as having an address of (1, 6). In the same way, we can think of the diode in the bottom right (cathode 6 and anode 1) as having an address of (6, 1).

Well, that’s all well and good, but what does it really accomplish? Let’s take a step back and remember how LEDs (light-emitting diodes) really work. The LED in its simplest form is a two-pin component. It has a positive end called the anode and a negative end called the cathode. In order to turn the LED on and produce light, we need to send current into the anode and out through the cathode. And, following the basic principles of electricity, we then know that the anode needs to be at a higher voltage than the cathode in order for current to flow in this direction. This is called a forward-bias. If we do the opposite and connect the cathode to a higher voltage than the anode, we would be subjecting the LED to a reverse-bias. In the case of LEDs, this reverse biasing does nothing. The diode will be off and unlit. In a similar manner, when both the anode and cathode are at the same voltage, the diode is also off because no current is flowing.

Here’s a table that summarizes the behavior:

<hr>

|Anode | Cathode | Result |
|:----:|:-------:|:------:|
|Anode LOW  | Cathode LOW  | OFF |
|Anode LOW  | Cathode HIGH | OFF |
|Anode HIGH | Cathode LOW  | ON |
|Anode HIGH | Cathode HIGH | OFF |

<hr>

Notice there is only a single state where the LED is lit.

So, back to the problem at hand! Why would we want to address the LEDs with all of these banks of anodes and cathodes? For individual control of course. Let’s assume our default starting state is to reverse-bias all of the diodes in the matrix. In that case, we would be setting all of the anode banks to logic low (0V) and all of the cathode banks to logic high (5V). Now all of the lights are off. What happens if we flip the logic levels of a single anode bank and a single cathode bank?

Take anode bank 2 and cathode bank 1. Now all of the LEDs connected to anode bank 2 have their anodes high and all of the LEDs connected to cathode bank 1 have their cathodes low. That means that at the intersection between anode 2 and cathode 1, LED(2, 1) is in the single ‘on’ state and all other LEDs in the matrix are in one of the other three possible ‘off’ states. Taking this a step further, we can now see that beginning with all of the anodes low and cathodes high, then toggling any anode and cathode, we turn on the LED that is at their intersection.

If you still can’t quite picture what is going on, I don’t blame you. Check out this [interactive javascript app](/pages/led-matrix-simulator.html) that I made to further illustrate the behavior. Please excuse the slightly underwhelming graphics…

The second concept that we should review before actually implementing this topic into a circuit is ‘persistence of vision’. I won’t get into the nitty-gritty of this, but it is a very important phenomenon in relation to lights and displays. The applicable idea in this situation is a simple statement: We can blink an LED at such a high frequency that the human brain perceives it to be simply ‘on’. It’s a simple enough idea, but it has big consequences for our matrix. If we can blink a single LED fast enough to make it appear ‘on’, then that means we can blink an entire matrix of LEDs so that they all appear ‘on’ at the same time. We do this by turning one LED on, then off. Then the next light in the matrix is turned on, then off. And so on and so forth until we have pulsed all of the lights on and off. We continue looping through each diode in the matrix, giving them a pulse. It is done so quickly that all of the LEDs appear to simply be ‘on’.

We have two ideas now. The first is that by connecting LEDs in rows and columns by anodes and cathodes, we can produce a circuit that has the capability of driving single LEDs at any given time. The second is that we can pulse a large number of LEDs so quickly that they all appear to be ‘on’ at the same time. Combining these abilities results in a matrix of LEDs that can light in any combo we want! It’s almost as if we’ve created a very simple monitor or display… And we have! Old displays used this same idea. The technology is called a passive-matrix control scheme, or PMOLED. In the case of LCD screens, each pixel isn’t actually creating light. There is a backlight that shines through the colored pixels, and that’s how we see the display. However, the idea of controlling those individual pixels in a vast array is the same!

Perfect! Now we have a better understanding of the underlying mechanics of this LED matrix. Let’s actually implement the idea on the breadboard and hook up the Arduino. I’ve taken the liberty of loading up the code I’ve written to run the matrix through a few paces. But don’t worry, we’ll soon be going over that.

<div class="embed-container">
  <iframe
      src="https://www.youtube.com/embed/4ez2iLf2AL0?feature=oembed&amp;wmode=transparent"
      width="100%"
      height="500"
      frameborder="0"
      allowfullscreen="">
  </iframe>
</div>

As you can see in the video, I have six anode banks and six cathode banks. I stopped at six because I was running out of digital IO on that side of the Arduino microcontroller. But with just twelve total signals coming from the Arduino, we already have a matrix comprised of thirty-six LEDs! I tried to wire the breadboard neatly so that you can clearly see all of the LEDs.

You might also notice that when only single LEDs are lit, they are much brighter than when multiple LEDs are on at one time. This is due to the duty cycle of the LED. When only a single diode is lit, all electrical power is being used by just that diode. But when multiple diodes are lit, they all have to share the power being output over that amount of time. In our case, when a single LED is illuminated, it has a 100% duty cycle. It is simply on. But as we light an entire column or row, we are splitting the ‘on’ time for any single given LED between five others. So each LED is only getting about 16% duty cycle (it’s only on for 1/6th of the time). As with most things, Wikipedia has an excellent explanation of the concept of [duty cycle](https://en.wikipedia.org/wiki/Duty_cycle).

But enough of that. It’s time to dig into the code! **View the entire source on [Github](https://troypallay.com/wp-content/uploads/2019/10/LEDmatrix.txt)**. In the next few sections, I will be explaining the code piece by piece:

```
#define ROW1 8
#define ROW2 9
#define ROW3 10
#define ROW4 11
#define ROW5 12
#define ROW6 13

#define COL1 2
#define COL2 3
#define COL3 4
#define COL4 5
#define COL5 6
#define COL6 7

int ROW[] = { ROW1, ROW2, ROW3, ROW4, ROW5, ROW6 };
int COL[] = { COL1, COL2, COL3, COL4, COL5, COL6 };
```
The beginning of the file starts with some #define statements. These statements are especially useful in the case of Arduino because they allow us to map names to pin numbers. And that’s what I’m doing here. I also create two arrays that hold the numbers of all the pins used as rows (anodes) and columns (cathodes). This is just a time-saving device that will allow me to perform actions on many pins at a time with fewer statements. You’ll see this in the next function.

From now on, we can go function by function. Peep my setup().

```
void setup() {

  for( int i = 0; i < 6; i++ ) {
    digitalWrite( ROW[i], LOW );
    pinMode( ROW[i], OUTPUT );
  }

  for( int i = 0; i < 6; i++ ) {
    digitalWrite( COL[i], HIGH );
    pinMode( COL[i], OUTPUT );
  }

  randomSeed(analogRead(0));

}
```

First I write all of the anode pins low and then set them as output pins. I usually try to write my pins high or low before I actually assert them as outputs just so that I know exactly how they are behaving before sending signals into my circuit. As you can see, since I placed all of the pins in arrays, it is much easier to perform repetitive tasks on all of them in a loop. Next, I write all of the cathode pins high and set them as outputs as well. Finally, I retrieve a random seed from one of the Arduino’s analog pins. This is another thing that we will use in a later function. It will allow us to generate random numbers when we need them.

Next up is the LED pulse function.

```
/*
 * Function: led_pulse
 * --------------------
 * Illuminates a chosen LED for a specified amount of time
 *
 * row: row that the LED occupies
 * col: column that the LED occupies
 * t : length of time that the LED will be illuminated in MICROSECONDS
 */
void led_pulse( int row, int col, unsigned long t ) {

  int multiple = t / 16383;
  int r = t % 16383;
  if( t > 16383 ) t = 16383;

  digitalWrite( ROW[row - 1], HIGH );
  digitalWrite( COL[col - 1], LOW );

  while( multiple ) {
    delayMicroseconds( t );
    multiple--;
  }
  delayMicroseconds( r );

  digitalWrite( ROW[row - 1], LOW );
  digitalWrite( COL[col - 1], HIGH );

}
```

This is probably the most complicated looking function in the whole sketch, but even then, it’s really pretty simple. To start off, the function is taking in three arguments: row, col, and t. row and col are self-explanatory. They refer to the row and column (or the address) at which you want to light an LED. Lastly, t is how long the LED will be illuminated in microseconds. The reason why we want to be able to specify the length of time that the LED is lit in such small increments is that when we illuminate a large bank of LEDs, they will need to all be blinking very rapidly to create the persistence of vision effect. We want to turn the LED on, wait a very short period, then turn it off. To create these short time delays, we use the delayMicroseconds() function. It takes an unsigned integer value for how many microseconds of delay you need. However, we have a problem. The Arduino reference material states that 16383 is the largest value that should be given to this function (see here). Beyond that, the function may not reliably work. This creates a problem for us because it means that the longest delay we can reliably produce is only 0.016383 seconds!!! Sure, we want fast blinking to make it appear as though many LEDs are lit at once, but what if we want to only light one LED for a long period? We could repeatedly call this function many times in a row, but I’d like to make it so that we can specify much longer periods of time without calling led_pulse() hundreds of times.

My solution to this is as follows: We take in the number t. We also create two variables multiple and r. multiple becomes t divided by the max value 16383 so that we know how many times we need to call the delay function with that max value to add up to a total delay of t. However, since we are dealing with integer division, we should also catch the remainder of this operation so that we stay accurate to the requested t delay. If you are not familiar with the mathematical operator to do this, it is called modulo. It is related to the division operator, but instead of giving us the quotient, it gives us the remainder. We end up with the situation described below:

requested t = (multiple * 16383) + r

If the requested t value is less than the max value, multiple will be 0 and r will be equal to t. If the requested t value was greater than the max value, then multiple will be a positive integer and r will be the positive remainder value. However, in that situation, we should also set t to the max value so that the delay function is not called with a number that is too large. We no longer need t to represent the requested t value because using the above function, we can now recover it using multiple and the max value 16383.

Beginning on line 16 of the snippet we then write the row high and the column low. Using ROW[row – 1] allows the input of the function row to be a row number 1 through 6. subtracting one from that number and using it as the index of our ROW[] matrix translates it into the number corresponding with that row’s pin. The same is done with the column to write it low. Now we wait… Using the values that we calculated earlier, we use the delayMicroseconds() function to wait for the proper amount of time. If the requested time was less than the max value, we wait for r microseconds (which would be equal to t). If the requested value was greater than the maximum allowed value, we loop multiple times, each time waiting for the maximum number of microseconds. After the loop is exhausted, we then complete the waiting period with r microseconds to ensure the total requested time was observed. Now that we have waited for the proper amount of time, the LED is turned off by asserting the row low and the column high.

As an aside, notice that we use an unsigned long data type for the t argument. since we are treating this as the number of microseconds, that means that the largest argument we can pass is 4,294,967,295 (32 bits or 2^32 – 1). That many microseconds convert to about 70 minutes.

Now let’s pulse rows and columns of LEDs all at one time!

```
/*
 * Function: row_pulse
 * --------------------
 * Illuminates a chosen row of LEDs for a specified amount of time
 *
 * row: row to be illuminated
 * row_t: length of time that the row will be illuminated in MILLISECONDS
 */
void row_pulse( int row, unsigned long row_t ) {

  unsigned long start = millis();
  while( millis() - start < row_t ) {

    for( int col = 1; col < 7; col++) led_pulse( row, col, 10 );

  }

}

/*
 * Function: col_pulse
 * --------------------
 * Illuminates a chosen column of LEDs for a specified amount of time
 *
 * col: column to be illuminated
 * col_t: length of time that the column will be illuminated in MILLISECONDS
 */
void col_pulse( int col, unsigned long col_t ) {

  unsigned long start = millis();
  while( millis() - start < col_t ) {

    for( int row = 1; row < 7; row++) led_pulse( row, col, 10 );

  }

}
```

These two functions operate by making use of the led_pulse() function. Each function takes either a row or col, as well as a time value. Notice that it is in milliseconds and not microseconds. This time value will specify how long each row or column is lit. A different method for keeping track of this time is shown here. Instead of using specific time delays that add up to the total requested period, the current time in milliseconds is recorded at the beginning of the function call. Then, the current time is repeatedly called until the correct amount of time has passed. At that point, the function exits.

In order to actually light an entire row or column, a for loop is used. In the row_pulse() function, led_pulse() is used with a constant row value, but an incrementing col value. This means that by using a small value like 10 microseconds, all of the LEDs in a row appear lit. The same principle is used in col_pulse(). And, since we are running through six LEDs at a time, this means we are checking the current time against the requested time every 60 microseconds because that is how long it takes to fully run through the loop.

```
/*
 * Function: all_on
 * --------------------
 * Illuminates all LEDs in the matrix for a specified amount of time
 *
 * all_t: length of time that all LEDs will be illuminated in MILLISECONDS
 */
void all_on( unsigned long all_t ) {

  unsigned long start = millis();
  int flag = 1;
  while( flag ) {

    for( int row = 1; row < 7; row++) {
      for( int col = 1; col < 7; col++) {
        led_pulse( row, col, 20 );
        if( millis() - start > all_t ) flag = 0;
      }
    }

  }

}
```

The all_on() function behaves exactly as named. It illuminates all of the LEDs in the 6×6 matrix for a specified amount of time. Again, this function takes its argument in milliseconds. And there is yet again a different method for keeping track of time. This one is quite similar to the previous method in that it checks the runtime of the method against the requested time. But, in this case, the check is performed after every single pulse of a single LED. To do this, a flag integer is used to check that the runtime is still less than the requested time. All of the LED pulsing happens within a while loop, so when the correct amount of time has elapsed, the flag is simply set to 0, or false, and the while loop exits along with the function. And, in order to flash all LEDs in the matrix one after another, two for loops run through each row and each column, lighting all the LEDs along the way. This time, 20 microseconds is used as the ‘on’ time for led_pulse().

```
/*
 * Function: zig_rows
 * --------------------
 * Illuminates rows one by one, in a back and forth pattern.
 *    This starts with row 1, progreses to row 6, then ends back at row 2
 *
 * repeat: number of times that the pattern will be repeated
 * row_t: length of time that each individual row will be illuminated in MILLISECONDS
 */
void zig_rows( int repeat, unsigned long row_t ) {

  while( repeat ) {

    for( unsigned long i = 2345654321; i > 0; i = i / 10 )
      row_pulse( i % 10, row_t );

    repeat--;
  }

}

/*
 * Function: zig_cols
 * --------------------
 * Illuminates columns one by one, in a back and forth pattern.
 *    This starts with column 1, progreses to column 6, then ends back at column 2
 *
 * repeat: number of times that the pattern will be repeated
 * col_t: length of time that each individual column will be illuminated in MILLISECONDS
 */
void zig_cols( int repeat, unsigned long col_t ) {

  while( repeat ) {

    for( unsigned long i = 2345654321; i > 0; i = i / 10 )
      col_pulse( i % 10, col_t );

    repeat--;
  }

}
```

I like these functions quite a lot. They are for creating a certain pattern using the row and column pulse functions. The for loop might look a little strange, but it’s nothing crazy. I start with a counter integer 2345654321. On the first pass of the loop, either row or col_pulse is called. If we take zig_rows(), for example, the row that is called is i % 10. By using modulo ten, we are calculating the remainder of a number divided by ten. And anything divided by ten gives a remainder of whatever was in that number’s one’s place. This means that the row that is pulsed corresponds to the one’s place of our counter integer i. On the first pass, it would be 1, since 1 is in the one’s place of 2345654321. Our loop then divides the counter by 10. Since it is integer division, this just chops off the current digit in the one’s place. The counter is now 234565432. This repeats until i is divided away to 0, meaning that each row is called in a back and forth pattern (rows 1, 2, 3, 4, 5, 6, then 5, 4, 3, and finally 2 again). You’ll also note that the counter i is of the data type unsigned long. That’s because it is such a large number, we need a large data type to hold it.

This entire process is enclosed in a while loop that also has a counter that is decrementing. It uses the repeat argument that is passed to the function in order to loop the pattern for the requested number of times. The function also takes an argument that it uses in either the col_pulse() or row_pulse() function for the length of time each row or column should be lit for. Note that this value is in milliseconds.

```
/*
 * Function: led_rand
 * --------------------
 * Illuminates a specified number of random LEDs in the matrix, each for a specified amount of time
 *
 * repeat: number of LEDs to be illuminated, one after another
 * t: length of time that the random LED will be illuminated in MILLISECONDS
 */
void led_rand( int repeat, unsigned long t ) {

  while( repeat ) {
    led_pulse( random( 1, 7 ), random( 1, 7 ), t * 1000 );
    repeat--;
  }

}
```

This is my favorite function. It makes use of the random seed that we created earlier in order to flash a random LED in the matrix. Its arguments are repeat and t. Just like the last two functions, repeat is how many times the pattern should repeat. And in this case, the pattern is just picking a random LED to pulse. t is how long each random LED should be illuminated. Notice that this function also takes its time argument in milliseconds. However, the led_pulse() function takes its time in microseconds. In order to convert, we simply multiply by 1000 before calling led_pulse().

In order to pick a random LED, random() is used with two arguments. 1 is given as a minimum value (inclusive), and 7 is given as a maximum value (exclusive).

```
void loop() {

  zig_rows( 2, 200 );
  all_on( 500 );

  led_rand( 20, 200 );
  all_on( 500 );

  zig_cols( 2, 200 );
  all_on( 500 );

  led_rand( 20, 200 );
  all_on( 500 );

}
```

Finally, we finish up with the actual Arduino loop() function! All it consists of is calling some of our created functions with appropriate values. Play around with these values to see how it changes the patterns and timings! You could also change the ‘on’ time values in the row_pulse(), col_pulse(), and all_on() functions in order to experiment with different values than 10 and 20 microseconds when calling led_pulse(). Have fun!

-Troy
