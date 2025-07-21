# EasyButton Library - Comprehensive Examples Guide

This guide provides detailed examples for various use cases and implementation patterns with the EasyButton library.

## Table of Contents

1. [Basic Examples](#basic-examples)
2. [Advanced Button Patterns](#advanced-button-patterns)
3. [Platform-Specific Examples](#platform-specific-examples)
4. [Integration Examples](#integration-examples)
5. [Performance Optimization](#performance-optimization)
6. [Troubleshooting Examples](#troubleshooting-examples)

---

## Basic Examples

### 1. Simple Button Press Detection

```cpp
#include <EasyButton.h>

#define BUTTON_PIN 2

EasyButton button(BUTTON_PIN);

void onButtonPressed() {
    Serial.println("Button pressed!");
    digitalWrite(LED_BUILTIN, !digitalRead(LED_BUILTIN));
}

void setup() {
    Serial.begin(115200);
    pinMode(LED_BUILTIN, OUTPUT);
    
    button.begin();
    button.onPressed(onButtonPressed);
    
    Serial.println("Ready! Press the button.");
}

void loop() {
    button.read();
}
```

### 2. Long Press Detection

```cpp
#include <EasyButton.h>

#define BUTTON_PIN 2

EasyButton button(BUTTON_PIN);

void onShortPress() {
    Serial.println("Short press - LED blink");
    digitalWrite(LED_BUILTIN, HIGH);
    delay(100);
    digitalWrite(LED_BUILTIN, LOW);
}

void onLongPress() {
    Serial.println("Long press - LED stays on");
    digitalWrite(LED_BUILTIN, HIGH);
}

void setup() {
    Serial.begin(115200);
    pinMode(LED_BUILTIN, OUTPUT);
    
    button.begin();
    button.onPressed(onShortPress);
    button.onPressedFor(2000, onLongPress); // 2 seconds
    
    Serial.println("Short press: blink, Long press (2s): LED on");
}

void loop() {
    button.read();
}
```

### 3. Multiple Sequence Detection

```cpp
#include <EasyButton.h>

#define BUTTON_PIN 2

EasyButton button(BUTTON_PIN);

void onSinglePress() {
    Serial.println("Single press");
}

void onDoubleClick() {
    Serial.println("Double click detected!");
}

void onTripleClick() {
    Serial.println("Triple click detected!");
}

void onQuadrupleClick() {
    Serial.println("Quadruple click - Amazing!");
}

void setup() {
    Serial.begin(115200);
    
    button.begin();
    button.onPressed(onSinglePress);
    button.onSequence(2, 1000, onDoubleClick);     // 2 clicks in 1 second
    button.onSequence(3, 1500, onTripleClick);     // 3 clicks in 1.5 seconds
    button.onSequence(4, 2000, onQuadrupleClick);  // 4 clicks in 2 seconds
    
    Serial.println("Try different click patterns!");
}

void loop() {
    button.read();
}
```

---

## Advanced Button Patterns

### 4. State Machine with Button Control

```cpp
#include <EasyButton.h>

#define BUTTON_PIN 2

enum SystemState {
    IDLE,
    WORKING,
    PAUSED,
    ERROR
};

SystemState currentState = IDLE;
EasyButton button(BUTTON_PIN);

void handleSinglePress() {
    switch (currentState) {
        case IDLE:
            currentState = WORKING;
            Serial.println("State: IDLE -> WORKING");
            break;
        case WORKING:
            currentState = PAUSED;
            Serial.println("State: WORKING -> PAUSED");
            break;
        case PAUSED:
            currentState = WORKING;
            Serial.println("State: PAUSED -> WORKING");
            break;
        case ERROR:
            currentState = IDLE;
            Serial.println("State: ERROR -> IDLE (Reset)");
            break;
    }
}

void handleLongPress() {
    if (currentState != IDLE) {
        currentState = IDLE;
        Serial.println("Long press - Reset to IDLE");
    }
}

void handleDoubleClick() {
    currentState = ERROR;
    Serial.println("Double click - ERROR state triggered");
}

void setup() {
    Serial.begin(115200);
    
    button.begin();
    button.onPressed(handleSinglePress);
    button.onPressedFor(3000, handleLongPress);
    button.onSequence(2, 800, handleDoubleClick);
    
    Serial.println("State Machine Demo:");
    Serial.println("Single press: Change state");
    Serial.println("Long press (3s): Reset to IDLE");
    Serial.println("Double click: Trigger ERROR");
}

void loop() {
    button.read();
    
    // Handle state-specific logic
    static unsigned long lastStateUpdate = 0;
    if (millis() - lastStateUpdate > 5000) {
        Serial.print("Current state: ");
        switch (currentState) {
            case IDLE: Serial.println("IDLE"); break;
            case WORKING: Serial.println("WORKING"); break;
            case PAUSED: Serial.println("PAUSED"); break;
            case ERROR: Serial.println("ERROR"); break;
        }
        lastStateUpdate = millis();
    }
}
```

### 5. Multiple Buttons with Different Functions

```cpp
#include <EasyButton.h>

#define BUTTON1_PIN 2
#define BUTTON2_PIN 3
#define BUTTON3_PIN 4

EasyButton button1(BUTTON1_PIN);
EasyButton button2(BUTTON2_PIN);
EasyButton button3(BUTTON3_PIN);

int counter = 0;
bool systemEnabled = false;

void incrementCounter() {
    if (systemEnabled) {
        counter++;
        Serial.print("Counter: ");
        Serial.println(counter);
    } else {
        Serial.println("System disabled - enable first!");
    }
}

void decrementCounter() {
    if (systemEnabled) {
        counter--;
        Serial.print("Counter: ");
        Serial.println(counter);
    } else {
        Serial.println("System disabled - enable first!");
    }
}

void toggleSystem() {
    systemEnabled = !systemEnabled;
    Serial.print("System ");
    Serial.println(systemEnabled ? "ENABLED" : "DISABLED");
    if (!systemEnabled) {
        counter = 0;
        Serial.println("Counter reset to 0");
    }
}

void resetCounter() {
    counter = 0;
    Serial.println("Counter RESET to 0");
}

void setup() {
    Serial.begin(115200);
    
    // Button 1: Increment (short) / Reset (long)
    button1.begin();
    button1.onPressed(incrementCounter);
    button1.onPressedFor(2000, resetCounter);
    
    // Button 2: Decrement
    button2.begin();
    button2.onPressed(decrementCounter);
    
    // Button 3: Toggle system enable/disable
    button3.begin();
    button3.onPressed(toggleSystem);
    
    Serial.println("Multi-button counter system:");
    Serial.println("Button 1: Increment (short) / Reset (long 2s)");
    Serial.println("Button 2: Decrement");
    Serial.println("Button 3: Enable/Disable system");
}

void loop() {
    button1.read();
    button2.read();
    button3.read();
}
```

### 6. Interrupt-Based Multi-Button System

```cpp
#include <EasyButton.h>

#define BUTTON1_PIN 2  // Interrupt capable
#define BUTTON2_PIN 3  // Interrupt capable
#define BUTTON3_PIN 4  // Regular pin

EasyButton button1(BUTTON1_PIN);
EasyButton button2(BUTTON2_PIN);
EasyButton button3(BUTTON3_PIN);

volatile bool button1Triggered = false;
volatile bool button2Triggered = false;

void button1ISR() {
    button1.read();
}

void button2ISR() {
    button2.read();
}

void button1Pressed() {
    button1Triggered = true;
}

void button2Pressed() {
    button2Triggered = true;
}

void button3Pressed() {
    Serial.println("Button 3 pressed (polling)");
}

void setup() {
    Serial.begin(115200);
    
    // Setup interrupt buttons
    button1.begin();
    button1.onPressed(button1Pressed);
    if (button1.supportsInterrupt()) {
        button1.enableInterrupt(button1ISR);
        Serial.println("Button 1: Interrupt mode enabled");
    }
    
    button2.begin();
    button2.onPressed(button2Pressed);
    if (button2.supportsInterrupt()) {
        button2.enableInterrupt(button2ISR);
        Serial.println("Button 2: Interrupt mode enabled");
    }
    
    // Setup polling button
    button3.begin();
    button3.onPressed(button3Pressed);
    Serial.println("Button 3: Polling mode");
    
    Serial.println("Interrupt demo ready!");
}

void loop() {
    // Handle interrupt-triggered events in main loop
    if (button1Triggered) {
        Serial.println("Button 1 pressed (interrupt)");
        button1Triggered = false;
    }
    
    if (button2Triggered) {
        Serial.println("Button 2 pressed (interrupt)");
        button2Triggered = false;
    }
    
    // Poll the non-interrupt button
    button3.read();
    
    // Main application logic can run here
    // without being blocked by button reading
    delay(10);
}
```

---

## Platform-Specific Examples

### 7. ESP32 Touch Button Example

```cpp
#include <EasyButtonTouch.h>

// Use touch-capable pins
#define TOUCH_PIN_1 T0  // GPIO 4
#define TOUCH_PIN_2 T1  // GPIO 0

EasyButtonTouch touchButton1(TOUCH_PIN_1, 35, 40);  // Threshold: 40
EasyButtonTouch touchButton2(TOUCH_PIN_2, 35, 30);  // Threshold: 30 (more sensitive)

void touch1Pressed() {
    Serial.println("Touch sensor 1 activated");
}

void touch1LongPress() {
    Serial.println("Touch sensor 1 - long touch");
}

void touch2Pressed() {
    Serial.println("Touch sensor 2 activated");
}

void touch2DoubleTouch() {
    Serial.println("Touch sensor 2 - double touch!");
}

void setup() {
    Serial.begin(115200);
    
    // Initialize touch buttons
    touchButton1.begin(40);  // Set threshold
    touchButton1.onPressed(touch1Pressed);
    touchButton1.onPressedFor(1500, touch1LongPress);
    
    touchButton2.begin(30);  // More sensitive
    touchButton2.onPressed(touch2Pressed);
    touchButton2.onSequence(2, 1000, touch2DoubleTouch);
    
    Serial.println("ESP32 Touch Button Demo");
    Serial.println("Touch sensor 1: Basic touch + long touch");
    Serial.println("Touch sensor 2: Basic touch + double touch");
}

void loop() {
    touchButton1.read();
    touchButton2.read();
}
```

### 8. ESP8266/ESP32 with Lambda Functions

```cpp
#include <EasyButton.h>

#define BUTTON_PIN 0  // Boot button on many ESP boards

EasyButton button(BUTTON_PIN);

int pressCount = 0;
unsigned long lastPressTime = 0;

void setup() {
    Serial.begin(115200);
    
    button.begin();
    
    // Lambda function for simple press
    button.onPressed([]() {
        pressCount++;
        lastPressTime = millis();
        Serial.printf("Press #%d at %lu ms\n", pressCount, lastPressTime);
    });
    
    // Lambda function for long press with capture
    button.onPressedFor(2000, [&]() {
        Serial.printf("Long press detected! Total presses: %d\n", pressCount);
        pressCount = 0;  // Reset counter
    });
    
    // Lambda for double click
    button.onSequence(2, 800, []() {
        Serial.println("Double click - Quick response!");
    });
    
    Serial.println("ESP Lambda Demo - Boot button ready");
}

void loop() {
    button.read();
    
    // Show periodic status
    static unsigned long lastStatus = 0;
    if (millis() - lastStatus > 10000) {
        Serial.printf("Status: %d presses, last at %lu ms\n", 
                     pressCount, lastPressTime);
        lastStatus = millis();
    }
}
```

### 9. Virtual Button with I2C Port Expander

```cpp
#include <EasyButtonVirtual.h>
#include <Wire.h>

#define PCF8574_ADDRESS 0x20

bool expanderButton1 = false;
bool expanderButton2 = false;

EasyButtonVirtual vButton1(expanderButton1);
EasyButtonVirtual vButton2(expanderButton2);

void virtualButton1Pressed() {
    Serial.println("I2C Expander Button 1 pressed");
}

void virtualButton2Pressed() {
    Serial.println("I2C Expander Button 2 pressed");
}

void readExpanderButtons() {
    Wire.beginTransmission(PCF8574_ADDRESS);
    Wire.write(0xFF);  // Set all pins as input
    Wire.endTransmission();
    
    Wire.requestFrom(PCF8574_ADDRESS, 1);
    if (Wire.available()) {
        uint8_t data = Wire.read();
        
        // Buttons are active low (pulled up by PCF8574)
        expanderButton1 = !(data & 0x01);  // Pin 0
        expanderButton2 = !(data & 0x02);  // Pin 1
    }
}

void setup() {
    Serial.begin(115200);
    Wire.begin();
    
    vButton1.begin();
    vButton1.onPressed(virtualButton1Pressed);
    
    vButton2.begin();
    vButton2.onPressed(virtualButton2Pressed);
    
    Serial.println("I2C Port Expander Button Demo");
    Serial.println("Connect buttons to PCF8574 pins 0 and 1");
}

void loop() {
    readExpanderButtons();
    
    vButton1.read();
    vButton2.read();
    
    delay(10);  // Small delay for I2C stability
}
```

---

## Integration Examples

### 10. Button with LED Strip Control

```cpp
#include <EasyButton.h>

#define BUTTON_PIN 2
#define LED_PIN 6
#define NUM_LEDS 10

EasyButton button(BUTTON_PIN);

enum LedMode {
    OFF,
    SOLID,
    BLINK,
    FADE
};

LedMode currentMode = OFF;
int brightness = 0;
bool fadeDirection = true;

void cycleLedMode() {
    currentMode = (LedMode)((currentMode + 1) % 4);
    
    switch (currentMode) {
        case OFF:
            Serial.println("LED Mode: OFF");
            analogWrite(LED_PIN, 0);
            break;
        case SOLID:
            Serial.println("LED Mode: SOLID");
            analogWrite(LED_PIN, 255);
            break;
        case BLINK:
            Serial.println("LED Mode: BLINK");
            break;
        case FADE:
            Serial.println("LED Mode: FADE");
            break;
    }
}

void turnOffLeds() {
    Serial.println("Long press - All LEDs OFF");
    currentMode = OFF;
    analogWrite(LED_PIN, 0);
}

void setup() {
    Serial.begin(115200);
    pinMode(LED_PIN, OUTPUT);
    
    button.begin();
    button.onPressed(cycleLedMode);
    button.onPressedFor(3000, turnOffLeds);
    
    Serial.println("LED Strip Controller");
    Serial.println("Short press: Cycle modes (OFF->SOLID->BLINK->FADE)");
    Serial.println("Long press (3s): Turn off");
}

void loop() {
    button.read();
    
    // Handle LED animations
    static unsigned long lastUpdate = 0;
    unsigned long now = millis();
    
    switch (currentMode) {
        case BLINK:
            if (now - lastUpdate > 500) {
                static bool blinkState = false;
                analogWrite(LED_PIN, blinkState ? 255 : 0);
                blinkState = !blinkState;
                lastUpdate = now;
            }
            break;
            
        case FADE:
            if (now - lastUpdate > 20) {
                if (fadeDirection) {
                    brightness += 5;
                    if (brightness >= 255) {
                        brightness = 255;
                        fadeDirection = false;
                    }
                } else {
                    brightness -= 5;
                    if (brightness <= 0) {
                        brightness = 0;
                        fadeDirection = true;
                    }
                }
                analogWrite(LED_PIN, brightness);
                lastUpdate = now;
            }
            break;
    }
}
```

### 11. Button with WiFi Control (ESP32/ESP8266)

```cpp
#include <EasyButton.h>
#include <WiFi.h>
#include <WebServer.h>

#define BUTTON_PIN 0

const char* ssid = "YourWiFiSSID";
const char* password = "YourWiFiPassword";

EasyButton button(BUTTON_PIN);
WebServer server(80);

bool ledState = false;
int buttonPressCount = 0;

void handleRoot() {
    String html = "<html><body>";
    html += "<h1>Button Monitor</h1>";
    html += "<p>Button presses: " + String(buttonPressCount) + "</p>";
    html += "<p>LED state: " + String(ledState ? "ON" : "OFF") + "</p>";
    html += "<p><a href='/toggle'>Toggle LED</a></p>";
    html += "<p>Page auto-refreshes every 2 seconds</p>";
    html += "<script>setTimeout(function(){location.reload()}, 2000);</script>";
    html += "</body></html>";
    
    server.send(200, "text/html", html);
}

void handleToggle() {
    ledState = !ledState;
    digitalWrite(LED_BUILTIN, ledState);
    server.sendHeader("Location", "/");
    server.send(302, "text/plain", "");
}

void buttonPressed() {
    buttonPressCount++;
    ledState = !ledState;
    digitalWrite(LED_BUILTIN, ledState);
    
    Serial.printf("Button pressed! Count: %d, LED: %s\n", 
                  buttonPressCount, ledState ? "ON" : "OFF");
}

void buttonLongPress() {
    Serial.println("Long press - Resetting counter");
    buttonPressCount = 0;
}

void setup() {
    Serial.begin(115200);
    pinMode(LED_BUILTIN, OUTPUT);
    
    // Setup button
    button.begin();
    button.onPressed(buttonPressed);
    button.onPressedFor(3000, buttonLongPress);
    
    // Connect to WiFi
    WiFi.begin(ssid, password);
    Serial.print("Connecting to WiFi");
    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }
    Serial.println();
    Serial.print("Connected! IP: ");
    Serial.println(WiFi.localIP());
    
    // Setup web server
    server.on("/", handleRoot);
    server.on("/toggle", handleToggle);
    server.begin();
    
    Serial.println("Web server started");
    Serial.println("Button: Short press = toggle LED, Long press = reset counter");
}

void loop() {
    button.read();
    server.handleClient();
}
```

---

## Performance Optimization

### 12. Memory-Optimized Button (No Sequences)

```cpp
// Disable sequences to save memory
#define EASYBUTTON_DO_NOT_USE_SEQUENCES
#include <EasyButton.h>

#define BUTTON_PIN 2

EasyButton button(BUTTON_PIN, 20);  // Faster debounce for better response

void quickResponse() {
    Serial.println("Quick button response!");
}

void longPressAction() {
    Serial.println("Long press detected");
}

void setup() {
    Serial.begin(115200);
    
    button.begin();
    button.onPressed(quickResponse);
    button.onPressedFor(1000, longPressAction);
    
    Serial.println("Memory optimized button (no sequences)");
    Serial.printf("Free heap: %d bytes\n", ESP.getFreeHeap());
}

void loop() {
    button.read();
    
    // Show memory usage periodically
    static unsigned long lastMemCheck = 0;
    if (millis() - lastMemCheck > 5000) {
        Serial.printf("Free heap: %d bytes\n", ESP.getFreeHeap());
        lastMemCheck = millis();
    }
}
```

### 13. High-Performance Multi-Button with Interrupts

```cpp
#include <EasyButton.h>

#define BUTTON1_PIN 2
#define BUTTON2_PIN 3
#define BUTTON3_PIN 18
#define BUTTON4_PIN 19

EasyButton buttons[] = {
    EasyButton(BUTTON1_PIN),
    EasyButton(BUTTON2_PIN),
    EasyButton(BUTTON3_PIN),
    EasyButton(BUTTON4_PIN)
};

const int numButtons = sizeof(buttons) / sizeof(buttons[0]);

void button1ISR() { buttons[0].read(); }
void button2ISR() { buttons[1].read(); }
void button3ISR() { buttons[2].read(); }
void button4ISR() { buttons[3].read(); }

void (*buttonISRs[])() = {button1ISR, button2ISR, button3ISR, button4ISR};

void buttonPressed(int buttonNum) {
    Serial.printf("Button %d pressed\n", buttonNum + 1);
}

void setup() {
    Serial.begin(115200);
    
    for (int i = 0; i < numButtons; i++) {
        buttons[i].begin();
        
        // Use lambda to capture button number
        buttons[i].onPressed([i]() { buttonPressed(i); });
        
        if (buttons[i].supportsInterrupt()) {
            buttons[i].enableInterrupt(buttonISRs[i]);
            Serial.printf("Button %d: Interrupt enabled\n", i + 1);
        } else {
            Serial.printf("Button %d: Polling mode\n", i + 1);
        }
    }
    
    Serial.println("High-performance multi-button system ready");
}

void loop() {
    // Only poll buttons that don't support interrupts
    for (int i = 0; i < numButtons; i++) {
        if (!buttons[i].supportsInterrupt()) {
            buttons[i].read();
        }
    }
    
    // Main application logic runs here
    // without being blocked by button polling
    delay(1);
}
```

---

## Troubleshooting Examples

### 14. Button Diagnostics and Testing

```cpp
#include <EasyButton.h>

#define BUTTON_PIN 2

EasyButton button(BUTTON_PIN, 35, true, true);  // Standard configuration

unsigned long pressStartTime = 0;
unsigned long releaseTime = 0;
int totalPresses = 0;
bool diagnosticMode = false;

void buttonPressed() {
    totalPresses++;
    releaseTime = millis();
    unsigned long pressDuration = releaseTime - pressStartTime;
    
    Serial.printf("Press #%d - Duration: %lu ms\n", totalPresses, pressDuration);
    
    if (diagnosticMode) {
        Serial.printf("  Press start: %lu ms\n", pressStartTime);
        Serial.printf("  Release: %lu ms\n", releaseTime);
    }
}

void enableDiagnostics() {
    diagnosticMode = !diagnosticMode;
    Serial.printf("Diagnostic mode: %s\n", diagnosticMode ? "ON" : "OFF");
}

void resetCounters() {
    totalPresses = 0;
    Serial.println("Counters reset");
}

void setup() {
    Serial.begin(115200);
    
    button.begin();
    button.onPressed(buttonPressed);
    button.onPressedFor(2000, enableDiagnostics);
    button.onSequence(5, 3000, resetCounters);  // 5 clicks in 3 seconds
    
    Serial.println("Button Diagnostics Tool");
    Serial.println("Short press: Normal operation");
    Serial.println("Long press (2s): Toggle diagnostic mode");
    Serial.println("5 quick clicks: Reset counters");
    
    // Test button configuration
    Serial.printf("Button pin: %d\n", BUTTON_PIN);
    Serial.printf("Supports interrupt: %s\n", button.supportsInterrupt() ? "YES" : "NO");
}

void loop() {
    button.read();
    
    // Track press start time
    if (button.wasPressed()) {
        pressStartTime = millis();
        if (diagnosticMode) {
            Serial.printf("Button pressed at %lu ms\n", pressStartTime);
        }
    }
    
    // Show periodic status
    static unsigned long lastStatus = 0;
    if (millis() - lastStatus > 10000) {
        Serial.printf("Status: %d total presses\n", totalPresses);
        lastStatus = millis();
    }
}
```

### 15. Debounce Testing and Adjustment

```cpp
#include <EasyButton.h>

#define BUTTON_PIN 2

// Test different debounce times
EasyButton testButtons[] = {
    EasyButton(BUTTON_PIN, 10),   // Very fast
    EasyButton(BUTTON_PIN, 35),   // Default
    EasyButton(BUTTON_PIN, 100)   // Slow
};

int currentButtonIndex = 0;
const char* debounceLabels[] = {"10ms", "35ms", "100ms"};

void buttonPressed() {
    Serial.printf("Button pressed (debounce: %s)\n", 
                  debounceLabels[currentButtonIndex]);
}

void switchDebounceTime() {
    currentButtonIndex = (currentButtonIndex + 1) % 3;
    Serial.printf("Switched to debounce time: %s\n", 
                  debounceLabels[currentButtonIndex]);
    
    // Re-initialize the new button
    testButtons[currentButtonIndex].begin();
    testButtons[currentButtonIndex].onPressed(buttonPressed);
}

void setup() {
    Serial.begin(115200);
    
    // Initialize first button
    testButtons[currentButtonIndex].begin();
    testButtons[currentButtonIndex].onPressed(buttonPressed);
    testButtons[currentButtonIndex].onSequence(3, 1500, switchDebounceTime);
    
    Serial.println("Debounce Testing Tool");
    Serial.println("Single press: Test current debounce");
    Serial.println("Triple click: Switch debounce time");
    Serial.printf("Current debounce: %s\n", debounceLabels[currentButtonIndex]);
}

void loop() {
    testButtons[currentButtonIndex].read();
}
```

---

## Tips and Best Practices

### General Guidelines

1. **Always call `begin()`** before using any button
2. **Call `read()` regularly** - at least every 50ms for good responsiveness
3. **Use interrupts** when available for better performance
4. **Test debounce timing** with your specific buttons
5. **Keep callback functions short** to avoid blocking
6. **Use appropriate sequence timing** - not too fast, not too slow

### Memory Considerations

- Use `#define EASYBUTTON_DO_NOT_USE_SEQUENCES` to save memory if sequences aren't needed
- Virtual buttons use less memory than physical buttons
- Consider using arrays for multiple similar buttons

### Platform-Specific Notes

- **Arduino Uno/Nano**: Only pins 2 and 3 support interrupts
- **ESP8266/ESP32**: Most pins support interrupts, lambda functions available
- **ESP32**: Touch sensors available on specific pins only
- **Arduino Mega**: Pins 2, 3, 18, 19, 20, 21 support interrupts

---

*This examples guide covers EasyButton library version 2.0.3. Test examples with your specific hardware configuration.*