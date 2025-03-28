## **ESP8266 RFID SYSTEM WITH GOOGLESHEET**



### **CONNECTIONS:**
The connection of the Arduino to the display is relatively straightforward, and it is crucial to emphasize that the display operates on a 3V power supply; thus, it should not be connected to the 5V pin. We adhered to the pinout recommended by the Adafruit_SSD1306 library for the I2C connection, as detailed below:

|ESP8266|LCD|RFID|BUZZER|
|---|---|---|---|
|3V3|VCC|3.3V|
|GND|GND|GND |GND|
|D1|SCL|---|
|D2|SDA|---|
|D3|---|RST|
|D4|---|SDA|
|D5|---|SCK|
|D6|---|MISO|
|D7|---|MOSI|
|D8|---|----|POSITIVE|

This configuration creates the wiring shown in the following figure.
![CIRCUIT DIAGRAM](https://github.com/user-attachments/assets/7592d805-8a67-4d91-80bd-768e5a552107)

