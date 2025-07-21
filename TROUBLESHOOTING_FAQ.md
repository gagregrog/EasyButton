# EasyButton Library - Troubleshooting & FAQ

This guide addresses common issues, solutions, and frequently asked questions about the EasyButton library.

## Table of Contents

1. [Common Issues](#common-issues)
2. [Platform-Specific Problems](#platform-specific-problems)
3. [Performance Issues](#performance-issues)
4. [Frequently Asked Questions](#frequently-asked-questions)
5. [Best Practices](#best-practices)
6. [Debugging Tools](#debugging-tools)

---

## Common Issues

### Issue: Button Not Responding

**Symptoms:**
- Button callbacks are not triggered
- No response when pressing the button
- Silent failure

**Possible Causes & Solutions:**

1. **`begin()` not called**
   ```cpp
   // Wrong
   EasyButton button(2);
   void setup() {
       button.onPressed(callback);  // begin() not called!
   }
   
   // Correct
   EasyButton button(2);
   void setup() {
       button.begin();              // Always call begin() first
       button.onPressed(callback);
   }
   ```

2. **`read()` not called regularly**
   ```cpp
   // Wrong
   void loop() {
       // read() never called - button won't work
       delay(1000);
   }
   
   // Correct
   void loop() {
       button.read();  // Must call regularly
       delay(10);
   }
   ```

3. **Incorrect pin number**
   ```cpp
   // Check your wiring and pin definitions
   #define BUTTON_PIN 2  // Verify this matches your hardware
   EasyButton button(BUTTON_PIN);
   ```

4. **Wrong pull-up/pull-down configuration**
   ```cpp
   // For buttons that need external pull-down
   EasyButton button(2, 35, false, false);  // No internal pullup, active high
   
   // For standard buttons with internal pull-up (most common)
   EasyButton button(2, 35, true, true);    // Internal pullup, active low
   ```

### Issue: False Triggers / Button Bouncing

**Symptoms:**
- Multiple callbacks triggered for single press
- Erratic behavior
- Button seems "too sensitive"

**Solutions:**

1. **Increase debounce time**
   ```cpp
   // Default is 35ms, try increasing
   EasyButton button(2, 100);  // 100ms debounce
   ```

2. **Check for electrical noise**
   ```cpp
   // Add hardware debouncing (capacitor across button)
   // Use shielded cables for long wire runs
   // Keep wires away from power lines and motors
   ```

3. **Verify proper pull-up/pull-down resistors**
   ```cpp
   // Ensure button has proper pull-up or pull-down
   // Internal pull-up (most common)
   EasyButton button(2, 35, true, true);
   ```

### Issue: Long Press Not Working

**Symptoms:**
- `onPressedFor()` callback never triggers
- Only short press callbacks work

**Solutions:**

1. **Check timing conflicts**
   ```cpp
   // Wrong - sequence might interfere with long press
   button.onPressed(shortPress);
   button.onPressedFor(1000, longPress);
   button.onSequence(2, 800, doubleClick);  // Too close timing
   
   // Better - give more time
   button.onPressed(shortPress);
   button.onPressedFor(2000, longPress);    // Longer duration
   button.onSequence(2, 1500, doubleClick); // More time for sequence
   ```

2. **Verify button stays pressed**
   ```cpp
   // Debug: Check if button is actually held down
   void loop() {
       button.read();
       if (button.isPressed()) {
           Serial.println("Button is currently pressed");
       }
   }
   ```

3. **Use `update()` with interrupts**
   ```cpp
   void buttonISR() {
       button.read();
   }
   
   void setup() {
       button.begin();
       button.onPressedFor(2000, longPress);
       button.enableInterrupt(buttonISR);
   }
   
   void loop() {
       button.update();  // Needed for long press with interrupts
   }
   ```

### Issue: Sequences Not Detected

**Symptoms:**
- `onSequence()` callback never triggers
- Double/triple clicks not working

**Solutions:**

1. **Check if sequences are disabled**
   ```cpp
   // If this is defined somewhere, sequences won't work
   // #define EASYBUTTON_DO_NOT_USE_SEQUENCES
   
   // Make sure it's not defined, or remove it
   ```

2. **Adjust timing parameters**
   ```cpp
   // Too fast - hard for users
   button.onSequence(2, 300, doubleClick);
   
   // Better timing
   button.onSequence(2, 1000, doubleClick);  // 1 second window
   ```

3. **Check sequence limit**
   ```cpp
   // Maximum 5 sequences per button
   button.onSequence(2, 1000, callback1);  // OK
   button.onSequence(3, 1000, callback2);  // OK
   // ... up to 5 total sequences
   ```

### Issue: Compilation Errors

**Common Compilation Issues:**

1. **Library not found**
   ```
   Error: EasyButton.h: No such file or directory
   ```
   **Solution:** Install the library through Arduino IDE Library Manager

2. **Functional support errors on ESP32/ESP8266**
   ```cpp
   // If you get std::function errors, try:
   #undef EASYBUTTON_FUNCTIONAL_SUPPORT
   #include <EasyButton.h>
   ```

3. **Touch button compilation errors**
   ```cpp
   // EasyButtonTouch only works on ESP32 with touch support
   #if defined(ESP32)
   #include <EasyButtonTouch.h>
   EasyButtonTouch touchButton(T0);
   #endif
   ```

---

## Platform-Specific Problems

### Arduino Uno/Nano Issues

**Issue: Interrupts not working**
```cpp
// Only pins 2 and 3 support interrupts on Uno/Nano
EasyButton button1(2);  // OK for interrupts
EasyButton button2(3);  // OK for interrupts
EasyButton button3(4);  // NO interrupts - polling only

void setup() {
    button1.begin();
    if (button1.supportsInterrupt()) {
        button1.enableInterrupt(buttonISR);  // Will work
    }
    
    button3.begin();
    if (button3.supportsInterrupt()) {      // Will be false
        button3.enableInterrupt(buttonISR); // Won't be called
    }
}
```

**Issue: Memory limitations**
```cpp
// Uno has limited memory - disable sequences if needed
#define EASYBUTTON_DO_NOT_USE_SEQUENCES
#include <EasyButton.h>
```

### ESP32 Issues

**Issue: Touch buttons not working**
```cpp
// Check if your ESP32 supports touch sensors
#if defined(SOC_TOUCH_SENSOR_SUPPORTED)
    EasyButtonTouch touchButton(T0);
#else
    // Use regular button instead
    EasyButton touchButton(4);  // T0 = GPIO 4
#endif
```

**Issue: Touch sensitivity problems**
```cpp
// Adjust threshold - lower = more sensitive
EasyButtonTouch touchButton(T0, 35, 30);  // Very sensitive
EasyButtonTouch touchButton(T0, 35, 60);  // Less sensitive

// Test to find right threshold
void setup() {
    Serial.begin(115200);
    while (true) {
        int touchValue = touchRead(T0);
        Serial.printf("Touch value: %d\n", touchValue);
        delay(100);
    }
}
```

### ESP8266 Issues

**Issue: Boot button behavior**
```cpp
// GPIO 0 (boot button) has special behavior
EasyButton button(0);  // Boot button

void setup() {
    button.begin();
    // Don't hold button during boot/programming
}
```

**Issue: Pin limitations**
```cpp
// Some pins have restrictions on ESP8266
// Avoid: GPIO 6-11 (connected to flash)
// Safe pins: 0, 2, 4, 5, 12, 13, 14, 15, 16
```

---

## Performance Issues

### Issue: Slow Response Time

**Solutions:**

1. **Reduce debounce time**
   ```cpp
   EasyButton button(2, 15);  // Faster response (default is 35ms)
   ```

2. **Use interrupts**
   ```cpp
   void buttonISR() { button.read(); }
   
   void setup() {
       button.begin();
       if (button.supportsInterrupt()) {
           button.enableInterrupt(buttonISR);
       }
   }
   ```

3. **Call `read()` more frequently**
   ```cpp
   void loop() {
       button.read();
       // Avoid long delays
       delay(1);  // Instead of delay(100)
   }
   ```

### Issue: High Memory Usage

**Solutions:**

1. **Disable sequences**
   ```cpp
   #define EASYBUTTON_DO_NOT_USE_SEQUENCES
   #include <EasyButton.h>
   ```

2. **Use virtual buttons for I2C expanders**
   ```cpp
   // Instead of multiple physical buttons
   bool virtualButtons[8];
   EasyButtonVirtual vButtons[] = {
       EasyButtonVirtual(virtualButtons[0]),
       EasyButtonVirtual(virtualButtons[1]),
       // ...
   };
   ```

### Issue: CPU Usage Too High

**Solutions:**

1. **Use interrupts instead of polling**
   ```cpp
   // High CPU usage
   void loop() {
       for (int i = 0; i < 10; i++) {
           buttons[i].read();  // Polling 10 buttons
       }
   }
   
   // Lower CPU usage
   void setup() {
       for (int i = 0; i < 10; i++) {
           if (buttons[i].supportsInterrupt()) {
               buttons[i].enableInterrupt(buttonISRs[i]);
           }
       }
   }
   ```

---

## Frequently Asked Questions

### Q: Can I use multiple buttons on the same pin?
**A:** No, each EasyButton instance needs its own unique pin. For multiple buttons on one pin, use a voltage divider or analog reading approach (not supported by this library).

### Q: How many buttons can I use?
**A:** Limited only by available pins and memory. Each button uses some RAM for state tracking and callbacks.

### Q: Can I change button configuration after `begin()`?
**A:** Some settings can't be changed after initialization. Create a new button instance if you need different parameters.

### Q: Why doesn't my callback function get called?
**A:** Common causes:
- `begin()` not called
- `read()` not called regularly
- Incorrect wiring
- Wrong pin configuration

### Q: Can I use lambda functions as callbacks?
**A:** Yes, on ESP8266 and ESP32. Arduino Uno/Mega only support function pointers.

```cpp
// ESP8266/ESP32
button.onPressed([]() { Serial.println("Pressed"); });

// Arduino Uno/Mega
void buttonPressed() { Serial.println("Pressed"); }
button.onPressed(buttonPressed);
```

### Q: How do I handle multiple button combinations?
**A:** The library doesn't directly support combinations. Implement this logic in your callbacks:

```cpp
bool button1Pressed = false;
bool button2Pressed = false;

void button1Callback() { button1Pressed = true; }
void button2Callback() { button2Pressed = true; }

void loop() {
    button1.read();
    button2.read();
    
    if (button1Pressed && button2Pressed) {
        // Both buttons pressed
        Serial.println("Combination detected");
        button1Pressed = false;
        button2Pressed = false;
    }
}
```

### Q: Can I use EasyButton with analog buttons?
**A:** No, EasyButton is designed for digital inputs. For analog button arrays, you'd need a different approach.

### Q: How do I implement a momentary vs toggle button?
**A:** Use button states in your callback:

```cpp
bool ledState = false;

// Momentary - LED on while pressed
void momentaryCallback() {
    if (button.isPressed()) {
        digitalWrite(LED_PIN, HIGH);
    } else {
        digitalWrite(LED_PIN, LOW);
    }
}

// Toggle - LED changes state on each press
void toggleCallback() {
    ledState = !ledState;
    digitalWrite(LED_PIN, ledState);
}
```

### Q: Can I use EasyButton in an interrupt service routine?
**A:** The `read()` method can be called from an ISR, but keep callbacks short and avoid Serial.print() in ISRs.

### Q: How do I detect button release?
**A:** Use the state query methods:

```cpp
void loop() {
    button.read();
    
    if (button.wasReleased()) {
        Serial.println("Button was just released");
    }
}
```

---

## Best Practices

### Hardware Setup

1. **Use proper pull-up/pull-down resistors**
   - Internal pull-ups are usually sufficient
   - Use external resistors for long wire runs

2. **Add hardware debouncing for noisy environments**
   - Small capacitor (0.1µF) across button terminals
   - RC filter for severe noise

3. **Keep wires short**
   - Long wires can pick up noise
   - Use shielded cables if necessary

### Software Implementation

1. **Always call `begin()` before use**
2. **Call `read()` regularly (at least every 50ms)**
3. **Keep callback functions short**
4. **Use interrupts when available**
5. **Test timing parameters with your specific hardware**

### Debugging Approach

1. **Start simple** - test basic button press first
2. **Add features gradually** - sequences, long press, etc.
3. **Use Serial output** for debugging
4. **Check hardware with multimeter** if software seems correct

---

## Debugging Tools

### Basic Button Test

```cpp
#include <EasyButton.h>

#define BUTTON_PIN 2

EasyButton button(BUTTON_PIN);

void setup() {
    Serial.begin(115200);
    button.begin();
    
    Serial.println("Button Test Started");
    Serial.printf("Pin: %d\n", BUTTON_PIN);
    Serial.printf("Supports interrupt: %s\n", 
                  button.supportsInterrupt() ? "YES" : "NO");
}

void loop() {
    button.read();
    
    static bool lastState = false;
    bool currentState = button.isPressed();
    
    if (currentState != lastState) {
        Serial.printf("Button state changed: %s\n", 
                      currentState ? "PRESSED" : "RELEASED");
        lastState = currentState;
    }
    
    delay(10);
}
```

### Advanced Diagnostics

```cpp
#include <EasyButton.h>

#define BUTTON_PIN 2

EasyButton button(BUTTON_PIN);

unsigned long pressCount = 0;
unsigned long lastPressTime = 0;
unsigned long longestPress = 0;

void buttonPressed() {
    pressCount++;
    lastPressTime = millis();
}

void buttonLongPress() {
    unsigned long pressDuration = millis() - lastPressTime;
    if (pressDuration > longestPress) {
        longestPress = pressDuration;
    }
    Serial.printf("Long press: %lu ms\n", pressDuration);
}

void setup() {
    Serial.begin(115200);
    button.begin();
    button.onPressed(buttonPressed);
    button.onPressedFor(1000, buttonLongPress);
    
    Serial.println("Advanced Button Diagnostics");
}

void loop() {
    button.read();
    
    // Periodic status report
    static unsigned long lastReport = 0;
    if (millis() - lastReport > 5000) {
        Serial.printf("Status: %lu presses, longest: %lu ms\n", 
                      pressCount, longestPress);
        lastReport = millis();
    }
}
```

---

## Getting Help

If you're still experiencing issues:

1. **Check the GitHub repository**: [EasyButton Issues](https://github.com/evert-arias/EasyButton/issues)
2. **Review the official documentation**: [EasyButton Docs](https://evert-arias.github.io/easybtn-docs/)
3. **Post detailed questions** with:
   - Hardware setup description
   - Complete minimal code example
   - Expected vs actual behavior
   - Platform and library version

---

*This troubleshooting guide covers EasyButton library version 2.0.3. Always check for the latest version and documentation.*