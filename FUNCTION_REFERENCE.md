# EasyButton Library - Function Reference

This document provides a quick reference for all public functions, methods, and constants in the EasyButton library.

## Table of Contents

1. [EasyButton Class](#easybutton-class)
2. [EasyButtonBase Class](#easybuttonbase-class)
3. [EasyButtonTouch Class](#easybuttontouch-class)
4. [EasyButtonVirtual Class](#easybuttonvirtual-class)
5. [Constants and Macros](#constants-and-macros)
6. [Callback Types](#callback-types)

---

## EasyButton Class

### Constructor

```cpp
EasyButton(uint8_t pin, uint32_t debounce_time = 35, bool pullup_enable = true, bool active_low = true)
```

**Parameters:**
- `pin` - Arduino pin number (0-255)
- `debounce_time` - Debounce time in milliseconds (default: 35)
- `pullup_enable` - Enable internal pull-up resistor (default: true)
- `active_low` - Button logic, true = active low (default: true)

**Returns:** EasyButton object

---

### Public Methods

#### `void begin()`

**Description:** Initializes the button and configures the pin  
**Parameters:** None  
**Returns:** void  
**Required:** Must be called before using the button  

---

#### `bool read()`

**Description:** Reads and updates the button state  
**Parameters:** None  
**Returns:** Current debounced button state (true = pressed, false = released)  
**Note:** Must be called regularly in main loop or via interrupt  

---

#### `void update()`

**Description:** Updates button pressed time for long press detection  
**Parameters:** None  
**Returns:** void  
**Note:** Only needed when using interrupts  

---

#### `bool supportsInterrupt()`

**Description:** Checks if the button pin supports hardware interrupts  
**Parameters:** None  
**Returns:** true if pin supports interrupts, false otherwise  

---

#### `void enableInterrupt(callback_t callback)`

**Description:** Enables interrupt-based button reading  
**Parameters:**
- `callback` - Function to call on pin state change (ISR)
**Returns:** void  
**Note:** Pin must support interrupts  

---

#### `void disableInterrupt()`

**Description:** Disables interrupt-based reading, returns to polling mode  
**Parameters:** None  
**Returns:** void  

---

## EasyButtonBase Class

### Event Callback Methods

#### `void onPressed(callback_t callback)`

**Description:** Sets callback for single press events (press and release)  
**Parameters:**
- `callback` - Function to call when button is pressed and released
**Returns:** void  

---

#### `void onPressedFor(uint32_t duration, callback_t callback)`

**Description:** Sets callback for long press events  
**Parameters:**
- `duration` - Minimum hold time in milliseconds
- `callback` - Function to call when button is held for duration
**Returns:** void  

---

#### `void onSequence(uint8_t sequences, uint32_t duration, callback_t callback)`

**Description:** Sets callback for button sequence detection  
**Parameters:**
- `sequences` - Number of presses in sequence (1-255)
- `duration` - Maximum time window for sequence in milliseconds
- `callback` - Function to call when sequence is matched
**Returns:** void  
**Note:** Not available if `EASYBUTTON_DO_NOT_USE_SEQUENCES` is defined  

---

### State Query Methods

#### `bool isPressed()`

**Description:** Checks if button is currently pressed  
**Parameters:** None  
**Returns:** true if pressed, false if released  

---

#### `bool isReleased()`

**Description:** Checks if button is currently released  
**Parameters:** None  
**Returns:** true if released, false if pressed  

---

#### `bool wasPressed()`

**Description:** Checks if button was just pressed (state change)  
**Parameters:** None  
**Returns:** true if button changed from released to pressed  

---

#### `bool wasReleased()`

**Description:** Checks if button was just released (state change)  
**Parameters:** None  
**Returns:** true if button changed from pressed to released  

---

#### `bool pressedFor(uint32_t duration)`

**Description:** Checks if button has been pressed for specified time  
**Parameters:**
- `duration` - Time in milliseconds
**Returns:** true if button pressed for at least duration  

---

#### `bool releasedFor(uint32_t duration)`

**Description:** Checks if button has been released for specified time  
**Parameters:**
- `duration` - Time in milliseconds
**Returns:** true if button released for at least duration  

---

## EasyButtonTouch Class

**Availability:** ESP32 only, with touch sensor support

### Constructor

```cpp
EasyButtonTouch(uint8_t pin, uint32_t debounce_time = 35, uint16_t threshold = 50)
```

**Parameters:**
- `pin` - Touch-capable pin number (T0-T9)
- `debounce_time` - Debounce time in milliseconds (default: 35)
- `threshold` - Touch sensitivity threshold (default: 50)

**Returns:** EasyButtonTouch object

---

### Public Methods

#### `void begin()`

**Description:** Initializes touch button with default threshold  
**Parameters:** None  
**Returns:** void  

---

#### `void begin(int threshold)`

**Description:** Initializes touch button with specified threshold  
**Parameters:**
- `threshold` - Touch sensitivity threshold (0-1023)
**Returns:** void  
**Note:** Lower values = more sensitive  

---

#### `void setThreshold(int threshold)`

**Description:** Sets the touch sensitivity threshold  
**Parameters:**
- `threshold` - Touch sensitivity threshold (0-1023)
**Returns:** void  

---

**Inherited Methods:** All methods from EasyButtonBase are available

---

## EasyButtonVirtual Class

### Constructor

```cpp
EasyButtonVirtual(bool &button_abstraction, bool active_low = true)
```

**Parameters:**
- `button_abstraction` - Reference to boolean variable representing button state
- `active_low` - Button logic, true = active low (default: true)

**Returns:** EasyButtonVirtual object

---

### Public Methods

#### `void begin()`

**Description:** Initializes the virtual button  
**Parameters:** None  
**Returns:** void  

---

#### `bool read()`

**Description:** Reads and updates virtual button state from referenced variable  
**Parameters:** None  
**Returns:** Current debounced button state  

---

**Inherited Methods:** All methods from EasyButtonBase are available

---

## Constants and Macros

### Read Type Constants

```cpp
#define EASYBUTTON_READ_TYPE_INTERRUPT 0  // Interrupt-based reading
#define EASYBUTTON_READ_TYPE_POLL 1       // Polling-based reading
```

### Sequence Limits

```cpp
#define MAX_SEQUENCES 5  // Maximum sequences per button
```

### Compilation Options

```cpp
// Disable sequence functionality to save memory
#define EASYBUTTON_DO_NOT_USE_SEQUENCES

// Force disable functional support
#undef EASYBUTTON_FUNCTIONAL_SUPPORT
```

### Touch Pin Constants (ESP32)

```cpp
T0   // GPIO 4
T1   // GPIO 0
T2   // GPIO 2
T3   // GPIO 15
T4   // GPIO 13
T5   // GPIO 12
T6   // GPIO 14
T7   // GPIO 27
T8   // GPIO 33
T9   // GPIO 32
```

---

## Callback Types

### Functional Callback (ESP8266, ESP32)

```cpp
typedef std::function<void()> callback_t;
```

**Usage:**
```cpp
// Function pointer
void myCallback() { /* code */ }
button.onPressed(myCallback);

// Lambda function
button.onPressed([]() { 
    Serial.println("Button pressed"); 
});

// Member function
class MyClass {
public:
    void handleButton() { /* code */ }
};

MyClass obj;
button.onPressed([&obj]() { obj.handleButton(); });
```

### Function Pointer (Arduino Uno, Mega, etc.)

```cpp
typedef void (*callback_t)();
```

**Usage:**
```cpp
// Function pointer only
void myCallback() { /* code */ }
button.onPressed(myCallback);
```

---

## Parameter Ranges and Limits

| Parameter | Type | Range | Default | Notes |
|-----------|------|-------|---------|-------|
| pin | uint8_t | 0-255 | N/A | Must be valid GPIO pin |
| debounce_time | uint32_t | 1-4294967295 | 35 | Milliseconds |
| duration | uint32_t | 1-4294967295 | N/A | Milliseconds |
| sequences | uint8_t | 1-255 | N/A | Number of button presses |
| threshold | uint16_t | 0-1023 | 50 | Touch sensitivity |
| pullup_enable | bool | true/false | true | Internal pull-up |
| active_low | bool | true/false | true | Button logic |

---

## Return Value Meanings

| Method | Return Type | True Means | False Means |
|--------|-------------|------------|-------------|
| read() | bool | Button pressed | Button released |
| isPressed() | bool | Currently pressed | Currently released |
| isReleased() | bool | Currently released | Currently pressed |
| wasPressed() | bool | Just pressed | No state change to pressed |
| wasReleased() | bool | Just released | No state change to released |
| pressedFor() | bool | Held for duration | Not held long enough |
| releasedFor() | bool | Released for duration | Not released long enough |
| supportsInterrupt() | bool | Pin supports interrupts | Pin doesn't support interrupts |

---

## Common Usage Patterns

### Basic Setup Pattern
```cpp
EasyButton button(PIN);
void setup() {
    button.begin();
    button.onPressed(callback);
}
void loop() {
    button.read();
}
```

### Interrupt Pattern
```cpp
void buttonISR() { button.read(); }
void setup() {
    button.begin();
    if (button.supportsInterrupt()) {
        button.enableInterrupt(buttonISR);
    }
}
```

### State Checking Pattern
```cpp
void loop() {
    button.read();
    if (button.wasPressed()) {
        // Handle press
    }
    if (button.wasReleased()) {
        // Handle release
    }
}
```

---

*This function reference covers EasyButton library version 2.0.3*