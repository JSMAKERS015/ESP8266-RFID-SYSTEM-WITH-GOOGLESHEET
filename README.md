## **ESP8266 RFID SYSTEM WITH GOOGLESHEET**



### **CONNECTIONS:**
The connection of the ESP8266 to the display is relatively straightforward, and it is crucial to emphasize that the display operates on a 5V power supply. Make sure RFID cnnections shouls be correct...

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

To program the Esp8266, upload the "RAW_CODE.ino" file, which will handle drawing and communication with the computer. Use the Arduino IDE to compile and upload the code. Make sure the esp8266 is connected to the computer during this process.
![Screenshot 2025-03-28 143240](https://github.com/user-attachments/assets/072874c1-46fe-451d-809a-9b69ecf15e89)


### **Uploading the Code**

1.  Go to **Tools > Board > esp8266 > NodeMCU 1.0(ESP-12E Module)**. 
2.  "Copy this code and paste it into your IDE."
3.  Please compile the code, upload it, and take pleasure in the successful execution of your work.
```
/* 🎓 JS MAKERS
*  - Github- https://github.com/JSMAKERS015/ESP8266-RFID-SYSTEM-WITH-GOOGLESHEET.git
* 📷 Instagram- https://www.instagram.com/js_makers015?utm_source=qr&igsh=MWM2Zmp6YmRoem0wdQ==
* 📌 Smart Attendance System using ESP8266, RFID, Google Sheets & LCD Display
* 📡 Features:
*  - RFID-based attendance tracking 🏷️
*  - Sends data to Google Sheets 📊
*  - Real-time clock using NTP 🕒
*  - LCD Display for live updates 📟
*/

#include <Wire.h>
#include <SPI.h>
#include <MFRC522.h>
#include <Arduino.h>
#include <ESP8266WiFi.h>
#include <ESP8266HTTPClient.h>
#include <WiFiClient.h>
#include <WiFiClientSecureBearSSL.h>
#include <LiquidCrystal_I2C.h>
#include <NTPClient.h>
#include <WiFiUdp.h>
#include <set>  // Include set for tracking scanned cards

std::set<String> scannedCards;  // Stores UIDs of scanned cards before 5 PM

/* 🎯 Pin Configuration */
#define RST_PIN  D3
#define SS_PIN   D4
#define BUZZER   D8

MFRC522 mfrc522(SS_PIN, RST_PIN);  // RFID Scanner Instance
MFRC522::MIFARE_Key key;
MFRC522::StatusCode status;

/* 📌 RFID Storage Block */
int blockNum1 = 4;
byte bufferLen = 18;
byte readBlockData[18];
int getHour();
int getMinute();
void ReadDataFromBlock(int blockNum, byte readBlockData[]);



/* 🌍 Google Sheets URL */
const String sheet_url = "GOOGLE SHEET URL";


/* 📶 Wi-Fi Credentials */
#define WIFI_SSID "USER ID"         //YOUR WIFI CREDENTIALS
#define WIFI_PASSWORD "PASSWORD"    // your wiifi password

LiquidCrystal_I2C lcd(0x27, 20, 4);  // LCD Display Instance
WiFiUDP ntpUDP;
NTPClient timeClient(ntpUDP, "pool.ntp.org", 19800, 60000); // GMT+5:30 (India)

/* ⏳ Time Management Variables */
String lastCardUID = "";  // Last scanned UID
unsigned long lastScanTime = 0;  // Last scan time
const unsigned long scanDelay = 30000;  // 30 seconds delay

/* 📡 Wi-Fi Icon */
byte wifiIcon[8] = {
  0b00000,
  0b00100,
  0b01010,
  0b10001,
  0b00000,
  0b00100,
  0b00000,
  0b00000
};


/* 🚀 Setup Function */
void setup() {
Serial.begin(9600);
lcd.init();
lcd.backlight();
lcd.createChar(0, wifiIcon);  // Create Wi-Fi icon

lcd.setCursor(1, 1);
lcd.print("Initializing");
for (int repeat = 0; repeat < 3; repeat++) {
for (int dots = 0; dots <= 5; dots++) {
lcd.setCursor(13, 1);
lcd.print(".....");
delay(500);
lcd.setCursor(13, 1);
lcd.print(".....");
}
}

/* 📡 Connect to Wi-Fi */
Serial.print("Connecting to WiFi");
WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
while (WiFi.status() != WL_CONNECTED) {
Serial.print(".");
delay(500);
}
lcd.clear();
Serial.println("\n✅ WiFi Connected!");
lcd.setCursor(3, 1);
lcd.print("WIFI CONNECTED");
lcd.setCursor(19, 0);
lcd.write(byte(0));  // Wi-Fi icon after connection
delay(2000);
lcd.clear();

/* 🏷️ Initialize RFID & NTP */
pinMode(BUZZER, OUTPUT);
SPI.begin();
mfrc522.PCD_Init();
timeClient.begin();
}

/* 🌐 URL Encoding for HTTP Requests */
String urlencode(String str) {
String encoded = "";
char hexBuffer[4];

for (int i = 0; i < str.length(); i++) {
char c = str.charAt(i);
if (isalnum(c)) {
encoded += c;
} else if (c == ' ') {
encoded += "%20";
} else {
sprintf(hexBuffer, "%%%02X", c);
encoded += hexBuffer;
}
}
return encoded;
}


/* 📶 Check WiFi Connection */
void checkWiFiConnection() {
if (WiFi.status() != WL_CONNECTED) {
Serial.println("❌ WiFi Disconnected! Reconnecting...");
WiFi.disconnect();
WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
while (WiFi.status() != WL_CONNECTED) {
Serial.print(".");
lcd.setCursor(0, 3);
lcd.print("Reconnecting WiFi...");
delay(500);
}
Serial.println("\n✅ WiFi Reconnected!");
lcd.setCursor(0, 3);
lcd.print("WiFi Connected!    ");
delay(2000);
lcd.clear();
}
}



/* 🔄 Main Loop */
void loop() {
timeClient.update(); // Sync time from NTP

lcd.setCursor(3, 1);
lcd.print("SCAN YOUR CARD");
lcd.setCursor(19, 0);
lcd.write(WiFi.status() == WL_CONNECTED ? byte(0) : 'X');  // Wi-Fi icon or 'X' for disconnected


if (!mfrc522.PICC_IsNewCardPresent()) return;
if (!mfrc522.PICC_ReadCardSerial()) return;

// **Get Current Time**
int currentHour = getHour();     // Declare currentHour here
int currentMinute = getMinute(); // Declare currentMinute here


/* **Duplicate Scan Prevention** */
String currentCardUID = "";
for (byte i = 0; i < mfrc522.uid.size; i++) {
  currentCardUID += String(mfrc522.uid.uidByte[i], HEX);
}



// Duplicate Scan Prevention: Check if same card and within scan delay
if (currentCardUID == lastCardUID && (millis() - lastScanTime) < scanDelay && currentHour < 17) {
  lcd.setCursor(3, 3);
  lcd.print("DUPLICATE SCAN");
  delay(3000);
  lcd.clear();
  return;
}

// Update last scanned card and time
lastCardUID = currentCardUID;
lastScanTime = millis();




Serial.println("🔍 Reading data from RFID...");
ReadDataFromBlock(blockNum1, readBlockData);
String cardData = String((char*)readBlockData);
cardData.trim();

/* 🏷️ Extract Roll Number & Name */
int separatorIndex = cardData.indexOf('-');
if (separatorIndex == -1) {
Serial.println("❌ Invalid card data!");
return;
}
    
String rollNum = cardData.substring(0, separatorIndex);
String studentName = cardData.substring(separatorIndex + 1);

/* ✅ Check Attendance Status */
Serial.print("🕒 Current Time: ");
Serial.print(currentHour);
Serial.print(":");
Serial.println(currentMinute);

String attendanceStatus;

if (currentHour >= 17) { // After 5:00 PM
attendanceStatus = "Left";
} else if (currentHour > 9 || (currentHour == 9 && currentMinute > 30)) {

// After 9:30 AM
attendanceStatus = "Late";
} else {
attendanceStatus = "Present";
}


/* 📟 Display on LCD */
lcd.clear();
lcd.setCursor(0, 0);
lcd.print("Name: " + studentName);
lcd.setCursor(0, 1);
lcd.print("Roll No: " + rollNum);
lcd.setCursor(0, 2);
lcd.print("Status: " + attendanceStatus);
lcd.setCursor(19, 0);
lcd.write(WiFi.status() == WL_CONNECTED ? byte(0) : 'X');

/* 🔊 Buzzer Feedback */
digitalWrite(BUZZER, HIGH);
delay(200);
digitalWrite(BUZZER, LOW);
    
delay(1000);

/* 🌍 Send Data to Google Sheets */
if (WiFi.status() == WL_CONNECTED) {
std::unique_ptr<BearSSL::WiFiClientSecure> client(new BearSSL::WiFiClientSecure);
client->setInsecure();
String requestURL = sheet_url + urlencode(rollNum) + "&name=" + urlencode(studentName) + "&status=" + attendanceStatus;

HTTPClient https;
if (https.begin(*client, requestURL)) {
int httpCode = https.GET();
if (httpCode > 0) {
        Serial.println("✅ Data Sent to Google Sheets");
        lcd.setCursor(3, 3);
        lcd.print("DATA RECORDED!");
    } else {
        Serial.printf("❌ HTTP Error: %s\n", https.errorToString(httpCode).c_str());
        lcd.setCursor(3, 3);
        lcd.print("DATA SEND FAIL!");
    }
    https.end();
} else {
    Serial.println("❌ Unable to connect to Google Sheets.");
}
}
    
delay(5000);
mfrc522.PICC_HaltA();      // Halt the current card
mfrc522.PCD_StopCrypto1(); // Stop encryption on the PCD

lcd.clear();
}

/* ⏳ Get Current Hour */
int getHour() {
return timeClient.getHours();
}

/* ⏳ Get Current Minute */
int getMinute() {
return timeClient.getMinutes();
}

/* 🏷️ Read Data from RFID Card */
void ReadDataFromBlock(int blockNum, byte readBlockData[]) {
for (byte i = 0; i < 6; i++) {
key.keyByte[i] = 0xFF;
}

status = mfrc522.PCD_Authenticate(MFRC522::PICC_CMD_MF_AUTH_KEY_A, blockNum, &key, &(mfrc522.uid));
if (status != MFRC522::STATUS_OK) {
Serial.print("❌ Authentication failed for Read: ");
Serial.println(mfrc522.GetStatusCodeName(status));
return;
}
    
status = mfrc522.MIFARE_Read(blockNum, readBlockData, &bufferLen);
if (status != MFRC522::STATUS_OK) {
Serial.print("❌ Reading failed: ");
Serial.println(mfrc522.GetStatusCodeName(status));
return;
}
}

```


   
