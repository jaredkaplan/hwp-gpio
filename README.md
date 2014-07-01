GPIO Hardwarepins Library
=========================

This library provides a basic interface to the GPIO input and output pins on the Kinoma Create, or other devices supporting the Kinoma Hardwarepins API.

Getting Started
---------------

This library can be imported into Kinoma Studio and added to an application's build path settings. You can then use the GPIO module to create objects that operate on input or output pins on the hardware. Interfaces specifically for controlling LED lights and button inputs can be instantiated from the module as well, and provide an additional level of functionality.

The following objects can be created using the GPIO module in this library:

* [Output](#output)
* [OutputGroup](#outputgroup)
* [LED](#led)
* [Input](#input)
* [Button](#button)
* [Switch](#switch)
* [DirectionalButton](#directionalbutton)

Example
-------

The following example demonstrates how to instantiate a new object to control an LED connected to a GPIO pin on the device.

```javascript
// get a reference to the GPIO module
var GPIO = require( "GPIO" );

// create a new LED instance and specify that it is 
// connected to pin 3 on the hardware device
var led = new GPIO.LED( 3 );

// initialize the LED and turn it on
led.init();
led.on();
```

If you need to control a group of GPIO outputs that must be synchronized, then you can create an OutputGroup object and add instances of output objects (such as an LED) to it. All of the output objects that belong to an output group will be set at the same time when the group's sync method is called. Additionally, methods in the OutputGroup object allow you to set the state of all the output objects in the group with one call.

The following example demonstrates how to create a group of LED's that will be synchronized together.

```javascript
// create a new instance of a GPIO OutputGroup
var group = new GPIO.OutputGroup();

// create some LED instances and add them to
// to the LED output group object
group.add( new GPIO.LED( 3 ) );
group.add( new GPIO.LED( 4 ) );
group.add( new GPIO.LED( 5 ) );

// initialize the outputs in the group and 
// turn all of the lights on
group.init();
group.on();
```

It is also possible to poll the state of a GPIO input pin using one of the Input interfaces in the GPIO module. The following program demonstrates how to create a basic button input object that will listen for changes on a GPIO hardware pin:

```xml
<program xmlns="http://www.kinoma.com/kpr/1">
    <require id="GPIO" path="GPIO" />
    <script>
        <![CDATA[
            // Create a new button to poll for changes on
            // GPIO input pin 6. The button is pressed when
            // the input state is set to 0 (low)
            var button = new GPIO.Button( 6, 0 );
            
            // initialize the button and start polling
            button.init();
            button.poll( 50, "milliseconds", "/button/value" );       
        ]]>
    </script>
    <handler path="/button/value">
        <behavior>
            <method id="onInvoke" params="handler, message">
                <![CDATA[
                    var data = message.requestObject;

                    if( data != null )
                        trace( data.value + " (" + data.pressed + ")\n" );
                ]]>
            </method>
        </behavior>
    </handler>
</program>
```

API Reference
-------------

The following reference describes the interfaces defined in the GPIO module.

Output
------

The Output object interface is used to control an output GPIO pin on the hardware device. The interface includes methods for setting the state of the pin by turning it on or off.

Creating a new instance of an Output object:

```javascript
var output = new GPIO.Output( pin, config );
```

Initializing the Output object instance:

```javascript
output.init();
```

Setting the state of the output pin:

```javascript
output.set( value );
output.on();
output.off();
output.toggle();
```

OutputGroup
------

The OutputGroup object interface can be used to synchronize a group of Output objects (including the LED object), that need to all be set at the same time.

Creating a new instance of an OutputGroup object:

```javascript
var group = new GPIO.OutputGroup();
```

Adding an output object to the group:

```javascript
group.add( output );
```

Initializing the group (calls init method on all output objects that have been added to the group):

```javascript
group.init();
```

Synchronize the group of output pins by setting the state of each pin after their having been modified in the corresponding Output object:

```javascript
group.sync();
```

Setting the state of all output pins in the group (this automatically calls sync after changing the state of each Output object in the group):

```javascript
output.set( value );
output.on();
output.off();
output.toggle();
output.randomize();
```

It is possible to access the output objects in the group, and iterate over each item to perform a function, or modify the state of a specific output pin. This is useful for setting individual pin states before syncing the group.

To iterarte over each output object in the group:

```javascript
for( var j = 0; j < group.outputs.length; j++ )
   group.outputs[j].on();
```

To access an output object in the group by its pin number:

```javascript
group.get( pin );
```

LED
------

The LED object extends the Output interface, so you use the same methods to set the state of the LED that you do with an Output object. An LED instance can also be added to an OutputGroup object.

Creating a new instance of an LED object:

```javascript
var led = new GPIO.LED( pin, config );
```

Initializing the LED object instance:

```javascript
led.init();
```

Setting the state of the LED:

```javascript
led.set( value );
led.on();
led.off();
led.toggle();
```

The LED object provides a simulator that helps visualize the state of the LED light when you are running your application on the desktop. When you run an application that instantiates an LED object, a simulator part will be displayed for that LED if you have the Kinoma Create device selected. 

You can customize the simulator part by passing in configuration data to the constructor of the LED object. The LED simulator can be customized by specifying the LED color:

```javascript
var led = new GPIO.LED( 3, {color:"red"} );
```

Input
------

The Input object interface is used to listen for changes to an input GPIO pin on the hardware device. The interface includes methods for getting the current state of the pin, and for polling the pin over time. When polling a GPIO input pin, it will invoke a  callback handler that you specify when the pin state changes.

Creating a new instance of an Input object:

```javascript
var input = new GPIO.Input( pin, config );
```

Initializing the Input object instance:

```javascript
input.init();
```

To request the current state of the input pin, you must specify a callback to receive the result. The callback is the name of a handler that is implemented in your program or module:

```javascript
input.get( callback );
```

To poll for a state change of the input pin, you must specify the polling interval and time units, as well as the callback handler to receive the change event:

```javascript
input.poll( time, units, callback, skipFirst );
```

>The units parameter must be one of the following string literal values:
>
>* milliseconds
>* seconds
>* minutes
>* microseconds
>* nanoseconds
>* hours
>* days

When the Input object's callback handler is invoked, the result will be passed in the message requestObject property. The state of the GPIO input pin will either be 1 (high) or 0 (low). The structure of the result object will be as follows:

```javascript
{ value: 1 }
```

Button
------

The Button object extends the Input interface, and is used in the same way that the Input object is used.

Creating a new instance of a Button object:

```javascript
var button = new GPIO.Button( pin, pressedState, config );
```

Initializing the Button object instance:

```javascript
button.init();
```

Getting the button state:

```javascript
button.get( callback );
button.poll( time, units, callback, skipFirst );
```

Switch
------

The Switch object extends the Input interface, and is used in the same way that the Input object is used.

Creating a new instance of a Switch object:

```javascript
var switch = new GPIO.Switch( pin, config );
```

Initializing the Switch object instance:

```javascript
switch.init();
```

Getting the switch state:

```javascript
switch.get( callback );
switch.poll( time, units, callback, skipFirst );
```

DirectionalButton
------

The DirectionalButton object provides and interface for connecting a digital direction pad or joystick. When the DirectionalButton object is created, the pin numbers for each direction (up, down, left, right, and center) are passed into the constructor.

Creating a new instance of a five-way DirectionalButton object:

```javascript
var dpad = new GPIO.DirectionalButton( up, down, left, right, center, config );
```

Initializing the DirectionalButton object instance:

```javascript
dpad.init();
```

Getting the directional button state:

```javascript
dpad.get( callback );
dpad.poll( time, units, callback, skipFirst );
```

When the DirectionalButton object's callback handler is invoked, the result will be passed in the message requestObject property. The structure of the result object will be as follows:

```javascript
{ up:0, down:1, left:0, right:0, center:0 }
```
