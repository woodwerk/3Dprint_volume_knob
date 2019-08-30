# 3Dprint_volume_knob
A combo project - a 3D printed base and knob, containing a rotary encoder and Arduino Pro Micro

This project was inspired by model files, schematics and code shared on the [Prusa printers site](https://www.prusaprinters.org/prints/3592-media-control-volume-knob-abstract-body). The instructions provided there are fairly complete, so limiting content here to photos of the finished product and the inclusion of a shell for the Arduino that was also printed and then super-glued to the base. A set of small self-adhesive rubber button feed were added to the bottom of the assembly so it stays put on my desk.

The photo below shows the individual printed parts:
* Knob
* Base
* Bottom/cover

![parts](https://github.com/woodwerk/3Dprint_volume_knob/blob/master/media/printed_parts.png)

Also, printed a small box to contain the Arduino.

![arduino_box](https://github.com/woodwerk/3Dprint_volume_knob/blob/master/media/arduino_box.png)

Next is a view of the rotary encoder wired to the Ardunio Pro micro mounted in its box. 

![](https://github.com/woodwerk/3Dprint_volume_knob/blob/master/media/circuit.png)

The finished product

![](https://github.com/woodwerk/3Dprint_volume_knob/blob/master/media/assembled.png)

The original code needed a little tweaking to smooth the volume changes.

```
// Read the full tutorial at prusaprinters.org

#include <ClickEncoder.h>
#include <TimerOne.h>
#include <HID-Project.h>

#define ENCODER_CLK A2 // Change A0 to, for example, A5 if you want to use analog pin 5 instead
#define ENCODER_DT A1
#define ENCODER_SW A0

ClickEncoder *encoder; // variable representing the rotary encoder
int16_t last, value; // variables for current and last rotation value

void timerIsr() {
  encoder->service();
}

void setup() {
  Serial.begin(9600); // Opens the serial connection used for communication with the PC. 
  Consumer.begin(); // Initializes the media keyboard
  encoder = new ClickEncoder(ENCODER_DT, ENCODER_CLK, ENCODER_SW); // Initializes the rotary encoder with the mentioned pins

  Timer1.initialize(1000); // Initializes the timer, which the rotary encoder uses to detect rotation
  Timer1.attachInterrupt(timerIsr); 
  last = -1;
} 

void loop() {  
  value += encoder->getValue();
  // This part of the code is responsible for the actions when you rotate the encoder
//  if (value != last) { // New value is different than the last one, that means to encoder was rotated
    if (abs(value-last) > 2) { // New value more than 2 different than the last one (desenstize vs default); encoder was rotated
    if(last<value) // Detecting the direction of rotation
      Consumer.write(MEDIA_VOLUME_UP); // Replace this line to have a different function when rotating counter-clockwise
      else
      Consumer.write(MEDIA_VOLUME_DOWN); // Replace this line to have a different function when rotating clockwise
    last = value; // Refreshing the "last" varible for the next loop with the current value
    Serial.print("Encoder Value: "); // Text output of the rotation value used manily for debugging (open Tools - Serial Monitor to see it)
    Serial.println(value);
  }

  // This next part handles the rotary encoder BUTTON
  ClickEncoder::Button b = encoder->getButton(); // Asking the button for it's current state
  if (b != ClickEncoder::Open) { // If the button is unpressed, we'll skip to the end of this if block
    //Serial.print("Button: "); 
    //#define VERBOSECASE(label) case label: Serial.println(#label); break;
    switch (b) {
      case ClickEncoder::Clicked: // Button was clicked once
        Consumer.write(MEDIA_PLAY_PAUSE); // Replace this line to have a different function when clicking button once
      break;      
      
      case ClickEncoder::DoubleClicked: // Button was double clicked
         Consumer.write(MEDIA_NEXT); // Replace this line to have a different function when double-clicking
      break;      
    }
  }

  delay(10); // Wait 10 milliseconds, we definitely don't need to detect the rotary encoder any faster than that
  // The end of the loop() function, it will start again from the beginning (the beginning of the loop function, not the whole document)
}



```
