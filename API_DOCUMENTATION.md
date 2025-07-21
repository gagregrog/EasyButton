# EasyButton Library - Comprehensive API Documentation

## Table of Contents

1. [Overview](#overview)
2. [Installation](#installation)
3. [Quick Start](#quick-start)
4. [Core Classes](#core-classes)
   - [EasyButton](#easybutton)
   - [EasyButtonTouch](#easybuttontouch)
   - [EasyButtonVirtual](#easybuttonvirtual)
   - [Sequence](#sequence)
5. [API Reference](#api-reference)
6. [Examples](#examples)
7. [Advanced Usage](#advanced-usage)
8. [Platform Support](#platform-support)

## Overview

**EasyButton** is a comprehensive Arduino library for debouncing momentary contact switches like tactile buttons. It provides event-driven programming with callbacks for various button states and supports advanced features like sequence detection, long press detection, and interrupt-based operation.

### Key Features

- **Debouncing**: Automatic debouncing with configurable timing
- **Event-driven**: Callback functions for button events
- **Multiple Button Types**: Physical buttons, touch sensors, and virtual buttons
- **Advanced Detection**: Single press, long press, and sequence detection
- **Interrupt Support**: Hardware interrupt support for better performance
- **Cross-platform**: Supports Arduino, ESP8266, and ESP32

## Installation

### Arduino IDE
1. Open Arduino IDE
2. Go to **Sketch > Include Library > Manage Libraries**
3. Search for "EasyButton"
4. Install the library by Evert Arias

### PlatformIO
Add to your `platformio.ini`:
```ini
lib_deps = evert-arias/EasyButton@^2.0.3
```

### Manual Installation
1. Download the library from [GitHub](https://github.com/evert-arias/EasyButton)
2. Extract to your Arduino libraries folder

## Quick Start

```cpp
#include <EasyButton.h>

#define BUTTON_PIN 2

EasyButton button(BUTTON_PIN);

void onButtonPressed() {
    Serial.println("Button pressed!");
}

void setup() {
    Serial.begin(115200);
    button.begin();
    button.onPressed(onButtonPressed);
}

void loop() {
    button.read();
}
```

## Core Classes

### EasyButton

The main class for handling physical buttons connected to digital pins.

#### Constructor

```cpp
EasyButton(uint8_t pin, uint32_t debounce_time = 35, bool pullup_enable = true, bool active_low = true)
```

**Parameters:**
- `pin`: Arduino pin number where the button is connected
- `debounce_time`: Debounce time in milliseconds (default: 35ms)
- `pullup_enable`: Enable internal pull-up resistor (default: true)
- `active_low`: Button logic - true for active low, false for active high (default: true)

#### Public Methods

##### Initialization

```cpp
void begin()
```
Initializes the button object and configures the pin. Must be called before using the button.

**Example:**
```cpp
EasyButton button(2);
void setup() {
    button.begin();
}
```

##### Reading Button State

```cpp
bool read()
```
Reads and updates the button state. Returns the current debounced state (true = pressed, false = released).

**Example:**
```cpp
void loop() {
    bool currentState = button.read();
    if (currentState) {
        // Button is currently pressed
    }
}
```

##### Event Callbacks

```cpp
void onPressed(callback_t callback)
```
Sets a callback function to be called when the button is pressed and then released (single press).

**Example:**
```cpp
void buttonPressed() {
    Serial.println("Single press detected");
}

void setup() {
    button.begin();
    button.onPressed(buttonPressed);
}
```

```cpp
void onPressedFor(uint32_t duration, callback_t callback)
```
Sets a callback function to be called when the button is held for at least the specified duration.

**Parameters:**
- `duration`: Minimum hold time in milliseconds
- `callback`: Function to call when the button is held

**Example:**
```cpp
void buttonHeld() {
    Serial.println("Button held for 2 seconds");
}

void setup() {
    button.begin();
    button.onPressedFor(2000, buttonHeld);
}
```

```cpp
void onSequence(uint8_t sequences, uint32_t duration, callback_t callback)
```
Sets a callback function to be called when a specific sequence of button presses is detected.

**Parameters:**
- `sequences`: Number of presses in the sequence
- `duration`: Maximum time window for the sequence (in milliseconds)
- `callback`: Function to call when sequence is matched

**Example:**
```cpp
void doubleClick() {
    Serial.println("Double click detected");
}

void setup() {
    button.begin();
    button.onSequence(2, 1000, doubleClick); // Double click within 1 second
}
```

##### State Query Methods

```cpp
bool isPressed()
```
Returns true if the button is currently pressed.

```cpp
bool isReleased()
```
Returns true if the button is currently released.

```cpp
bool wasPressed()
```
Returns true if the button was just pressed (state changed from released to pressed).

```cpp
bool wasReleased()
```
Returns true if the button was just released (state changed from pressed to released).

```cpp
bool pressedFor(uint32_t duration)
```
Returns true if the button has been pressed for at least the specified duration.

**Parameters:**
- `duration`: Time in milliseconds

```cpp
bool releasedFor(uint32_t duration)
```
Returns true if the button has been released for at least the specified duration.

**Parameters:**
- `duration`: Time in milliseconds

##### Interrupt Support

```cpp
bool supportsInterrupt()
```
Returns true if the button pin supports hardware interrupts.

```cpp
void enableInterrupt(callback_t callback)
```
Enables interrupt-based reading for better performance.

**Parameters:**
- `callback`: ISR function to handle pin changes

**Example:**
```cpp
void buttonISR() {
    button.read();
}

void setup() {
    button.begin();
    if (button.supportsInterrupt()) {
        button.enableInterrupt(buttonISR);
    }
}
```

```cpp
void disableInterrupt()
```
Disables interrupt-based reading and returns to polling mode.

```cpp
void update()
```
Updates button pressed time. Only needed when using interrupts to handle long press detection.

### EasyButtonTouch

Specialized class for ESP32 touch sensors.

**Note:** Only available on ESP32 platforms with touch sensor support.

#### Constructor

```cpp
EasyButtonTouch(uint8_t pin, uint32_t debounce_time = 35, uint16_t threshold = 50)
```

**Parameters:**
- `pin`: Touch-capable pin number
- `debounce_time`: Debounce time in milliseconds (default: 35ms)
- `threshold`: Touch sensitivity threshold (default: 50)

#### Public Methods

```cpp
void begin()
```
Initializes the touch button with default threshold.

```cpp
void begin(int threshold)
```
Initializes the touch button with specified threshold.

```cpp
void setThreshold(int threshold)
```
Sets the touch sensitivity threshold.

**Example:**
```cpp
#include <EasyButtonTouch.h>

EasyButtonTouch touchButton(T0); // GPIO 4 on ESP32

void touchDetected() {
    Serial.println("Touch detected!");
}

void setup() {
    Serial.begin(115200);
    touchButton.begin(40); // Lower threshold = more sensitive
    touchButton.onPressed(touchDetected);
}

void loop() {
    touchButton.read();
}
```

### EasyButtonVirtual

Class for creating virtual buttons using boolean variables instead of physical pins.

#### Constructor

```cpp
EasyButtonVirtual(bool &button_abstraction, bool active_low = true)
```

**Parameters:**
- `button_abstraction`: Reference to a boolean variable representing the button state
- `active_low`: Button logic (default: true)

#### Public Methods

```cpp
void begin()
```
Initializes the virtual button.

```cpp
bool read()
```
Reads and updates the virtual button state from the referenced boolean variable.

**Example:**
```cpp
#include <EasyButtonVirtual.h>

bool virtualButtonState = false;
EasyButtonVirtual vButton(virtualButtonState);

void virtualPressed() {
    Serial.println("Virtual button pressed!");
}

void setup() {
    Serial.begin(115200);
    vButton.begin();
    vButton.onPressed(virtualPressed);
}

void loop() {
    vButton.read();
    
    // Simulate button press from external source
    // (e.g., from I2C port expander, network, etc.)
    if (someExternalCondition()) {
        virtualButtonState = true;
    } else {
        virtualButtonState = false;
    }
}
```

### Sequence

Internal class for handling button sequence detection. Not directly instantiated by users.

#### Key Features
- Tracks multiple button presses within a time window
- Automatically resets after timeout or successful match
- Supports up to 5 different sequences per button

## API Reference

### Callback Function Types

```cpp
// For platforms with functional support (ESP8266, ESP32)
typedef std::function<void()> callback_t;

// For other Arduino platforms
typedef void (*callback_t)();
```

### Constants

```cpp
#define EASYBUTTON_READ_TYPE_INTERRUPT 0
#define EASYBUTTON_READ_TYPE_POLL 1
#define MAX_SEQUENCES 5
```

### Compilation Options

```cpp
// Disable sequence functionality to save memory
#define EASYBUTTON_DO_NOT_USE_SEQUENCES

// Disable functional support even on supported platforms
#undef EASYBUTTON_FUNCTIONAL_SUPPORT
```

## Examples

### Basic Button Press Detection

```cpp
#include <EasyButton.h>

#define BUTTON_PIN 2

EasyButton button(BUTTON_PIN);

void onPressed() {
    Serial.println("Button pressed!");
}

void setup() {
    Serial.begin(115200);
    button.begin();
    button.onPressed(onPressed);
}

void loop() {
    button.read();
}
```

### Long Press Detection

```cpp
#include <EasyButton.h>

#define BUTTON_PIN 2

EasyButton button(BUTTON_PIN);

void onPressed() {
    Serial.println("Short press");
}

void onLongPress() {
    Serial.println("Long press (2+ seconds)");
}

void setup() {
    Serial.begin(115200);
    button.begin();
    button.onPressed(onPressed);
    button.onPressedFor(2000, onLongPress);
}

void loop() {
    button.read();
}
```

### Multiple Sequence Detection

```cpp
#include <EasyButton.h>

#define BUTTON_PIN 2

EasyButton button(BUTTON_PIN);

void onSinglePress() {
    Serial.println("Single press");
}

void onDoubleClick() {
    Serial.println("Double click");
}

void onTripleClick() {
    Serial.println("Triple click");
}

void setup() {
    Serial.begin(115200);
    button.begin();
    
    button.onPressed(onSinglePress);
    button.onSequence(2, 1000, onDoubleClick);   // Double click within 1 second
    button.onSequence(3, 1500, onTripleClick);   // Triple click within 1.5 seconds
}

void loop() {
    button.read();
}
```

### Interrupt-Based Operation

```cpp
#include <EasyButton.h>

#define BUTTON_PIN 2

EasyButton button(BUTTON_PIN);

void onPressed() {
    Serial.println("Button pressed (interrupt mode)");
}

void buttonISR() {
    button.read();
}

void setup() {
    Serial.begin(115200);
    button.begin();
    button.onPressed(onPressed);
    
    if (button.supportsInterrupt()) {
        button.enableInterrupt(buttonISR);
        Serial.println("Using interrupt mode");
    } else {
        Serial.println("Using polling mode");
    }
}

void loop() {
    // In interrupt mode, button.read() is called in ISR
    // Main loop can do other tasks
    delay(100);
}
```

### Multiple Buttons

```cpp
#include <EasyButton.h>

#define BUTTON1_PIN 2
#define BUTTON2_PIN 3

EasyButton button1(BUTTON1_PIN);
EasyButton button2(BUTTON2_PIN);

void button1Pressed() {
    Serial.println("Button 1 pressed");
}

void button2Pressed() {
    Serial.println("Button 2 pressed");
}

void setup() {
    Serial.begin(115200);
    
    button1.begin();
    button1.onPressed(button1Pressed);
    
    button2.begin();
    button2.onPressed(button2Pressed);
}

void loop() {
    button1.read();
    button2.read();
}
```

### Touch Button (ESP32)

```cpp
#include <EasyButtonTouch.h>

#define TOUCH_PIN T0  // GPIO 4

EasyButtonTouch touchButton(TOUCH_PIN);

void onTouchPressed() {
    Serial.println("Touch detected!");
}

void onTouchLongPress() {
    Serial.println("Long touch detected!");
}

void setup() {
    Serial.begin(115200);
    touchButton.begin(30);  // Set sensitivity threshold
    touchButton.onPressed(onTouchPressed);
    touchButton.onPressedFor(1000, onTouchLongPress);
}

void loop() {
    touchButton.read();
}
```

### Virtual Button with I2C Expander

```cpp
#include <EasyButtonVirtual.h>
#include <Wire.h>

bool expanderButton = false;
EasyButtonVirtual vButton(expanderButton);

void onVirtualPressed() {
    Serial.println("Expander button pressed!");
}

void setup() {
    Serial.begin(115200);
    Wire.begin();
    
    vButton.begin();
    vButton.onPressed(onVirtualPressed);
}

void loop() {
    // Read from I2C port expander (example)
    Wire.beginTransmission(0x20);  // PCF8574 address
    Wire.write(0x00);
    Wire.endTransmission();
    
    Wire.requestFrom(0x20, 1);
    if (Wire.available()) {
        uint8_t data = Wire.read();
        expanderButton = !(data & 0x01);  // Assuming button on pin 0
    }
    
    vButton.read();
}
```

## Advanced Usage

### Custom Debounce Timing

```cpp
// Very fast response (10ms debounce)
EasyButton fastButton(2, 10);

// Slow response for noisy environment (100ms debounce)
EasyButton slowButton(3, 100);
```

### Active High Buttons

```cpp
// For buttons that are high when pressed
EasyButton activeHighButton(2, 35, false, false);
//                              |    |      |
//                              |    |      +-- active_high
//                              |    +-- no pullup (external pulldown needed)
//                              +-- debounce time
```

### Memory Optimization

```cpp
// Disable sequences to save memory
#define EASYBUTTON_DO_NOT_USE_SEQUENCES
#include <EasyButton.h>
```

### State Machine Integration

```cpp
enum ButtonState {
    IDLE,
    SINGLE_PRESS,
    DOUBLE_PRESS,
    LONG_PRESS
};

ButtonState currentState = IDLE;

void onSinglePress() {
    currentState = SINGLE_PRESS;
}

void onDoublePress() {
    currentState = DOUBLE_PRESS;
}

void onLongPress() {
    currentState = LONG_PRESS;
}

void processState() {
    switch (currentState) {
        case SINGLE_PRESS:
            // Handle single press
            currentState = IDLE;
            break;
        case DOUBLE_PRESS:
            // Handle double press
            currentState = IDLE;
            break;
        case LONG_PRESS:
            // Handle long press
            currentState = IDLE;
            break;
    }
}
```

## Platform Support

### Arduino Uno/Nano/Pro Mini
- ✅ Basic functionality
- ✅ Interrupt support on pins 2 and 3
- ❌ Touch sensors not supported
- ❌ Functional callbacks not supported (function pointers only)

### ESP8266
- ✅ Full functionality
- ✅ Interrupt support on most pins
- ❌ Touch sensors not supported
- ✅ Functional callbacks supported

### ESP32
- ✅ Full functionality
- ✅ Interrupt support on most pins
- ✅ Touch sensors supported
- ✅ Functional callbacks supported

### Arduino Mega
- ✅ Basic functionality
- ✅ Interrupt support on pins 2, 3, 18, 19, 20, 21
- ❌ Touch sensors not supported
- ❌ Functional callbacks not supported (function pointers only)

## Best Practices

1. **Always call `begin()`** before using the button
2. **Call `read()` regularly** in your main loop or use interrupts
3. **Use interrupts** when available for better performance
4. **Choose appropriate debounce time** based on your button type
5. **Test sequence timing** to ensure good user experience
6. **Handle long press and single press** carefully to avoid conflicts
7. **Use virtual buttons** for complex input sources like I2C expanders

## Troubleshooting

### Common Issues

**Button not responding:**
- Check wiring and pin connections
- Verify `begin()` is called
- Ensure `read()` is called regularly
- Check debounce timing

**False triggers:**
- Increase debounce time
- Check for electrical noise
- Verify proper pull-up/pull-down resistors

**Sequences not working:**
- Check timing parameters
- Ensure sequences don't conflict
- Verify `EASYBUTTON_DO_NOT_USE_SEQUENCES` is not defined

**Touch button issues (ESP32):**
- Adjust threshold value
- Check pin compatibility
- Verify ESP32 touch sensor support

## Contributing

The EasyButton library is open source and welcomes contributions. Visit the [GitHub repository](https://github.com/evert-arias/EasyButton) to:

- Report bugs
- Request features  
- Submit pull requests
- View the latest documentation

## License

MIT License - see LICENSE file for details.

## Authors

- **Evert Arias** - Main developer
- **Felix A. Epp** - Contributor
- **José Gabriel Companioni Benítez** - Virtual button and interrupt features

---

*This documentation covers EasyButton library version 2.0.3. For the latest updates and examples, visit the [official documentation](https://evert-arias.github.io/easybtn-docs/).*