# **ESP8266 RFID🪪 SYSTEM⚙️ WITH GOOGLE🌎 SHEET📑**

![SMART RFID ATTENDANCE SYSTEM_20250327_160748_0000](https://github.com/user-attachments/assets/b385da70-ffcd-4996-a271-0f8e9e001ec4)




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



## **STEPS**
First, we will write data on the RFID card using an ESP8266 and the MFRC522 RFID module. When a card is detected, the system retrieves its Unique Identifier (UID) and displays it on the Serial Monitor. After identifying the card, the program proceeds to write predefined data—"13-JASPREET SINGH"—onto a specific memory block of the RFID card. Once the data is successfully written, the system immediately reads it back from the card to verify that the operation was successful. The retrieved data is then displayed on the Serial Monitor. Finally, the system halts communication with the RFID card and stops encryption, allowing the next card to be scanned. This process is useful for applications such as access control, user authentication, and data storage on RFID cards. 🚀

# ** 📂 Step-by-Step Guide to Upload the Code **

### **Step 1: Install Arduino IDE (if not installed)
Download and install Arduino IDE from:
🔗( https://www.arduino.cc/en/software)

### **Step 2: Install ESP8266 Board in Arduino IDE
1. Open Arduino IDE.
2. Go to File → Preferences.
3. In the Additional Board Manager URLs field, enter:
```
http://arduino.esp8266.com/stable/package_esp8266com_index.json
```
4. Click OK.
5. Go to Tools → Board → Boards Manager.
6. Search for ESP8266 and install the latest version.
   
### **Step 3: Install Required Libraries
Go to Sketch → Include Library → Manage Libraries, then install the following:
1. MFRC522 [by GithubCommunity](https://github.com/miguelbalboa/rfid.git)
2. SPI (built-in with Arduino IDE)
   
### **Step 4: Select the Correct Board and Port
1. Connect your ESP8266 board via USB.
2. Go to Tools → Board → Select your ESP8266 board (NodeMCU 1.0 or Wemos D1 R1).
3. Set Upload Speed to 115200.
4. Select the correct COM Port under Tools → Port.

### **Step 5: Copy and Paste the Code
1. Open Arduino IDE.
2. Create a new sketch (File → New).
3. Copy and paste the RFID code.
```
#include <SPI.h>
#include <MFRC522.h>

constexpr uint8_t RST_PIN = D3;  // 🔄 Reset pin for RFID
constexpr uint8_t SS_PIN = D4;   // 🛠 Slave Select pin for RFID

MFRC522 mfrc522(SS_PIN, RST_PIN);  // 📡 Create MFRC522 instance
MFRC522::MIFARE_Key key;           // 🔐 Key for authentication

int blockNum = 4;  // 💾 Block to store data (16 bytes max)

byte readBlockData[18];  // 📥 Buffer for reading data (18 bytes needed)
byte bufferLen = 18;
MFRC522::StatusCode status;  // 🚦 Status code for RFID operations

// ✍️ Function to write data to RFID card
void WriteDataToBlock(int blockNum, String data) {
byte blockData[16];  
memset(blockData, 0, sizeof(blockData));  // 🧹 Clear the buffer
data.getBytes(blockData, 16);  // 🔣 Convert string to byte array

// 🔐 Authenticate before writing
status = mfrc522.PCD_Authenticate(MFRC522::PICC_CMD_MF_AUTH_KEY_A, blockNum, &key, &(mfrc522.uid));
if (status != MFRC522::STATUS_OK) {
Serial.print("❌ Authentication failed (Write): ");
Serial.println(mfrc522.GetStatusCodeName(status));
return;
}

// 💾 Write data to block
status = mfrc522.MIFARE_Write(blockNum, blockData, 16);
if (status != MFRC522::STATUS_OK) {
Serial.print("❌ Writing failed: ");
Serial.println(mfrc522.GetStatusCodeName(status));
return;
}
    
Serial.println("✅ Data written successfully!");
}

// 📖 Function to read data from RFID card
void ReadDataFromBlock(int blockNum) {
// 🔐 Authenticate before reading
status = mfrc522.PCD_Authenticate(MFRC522::PICC_CMD_MF_AUTH_KEY_A, blockNum, &key, &(mfrc522.uid));
if (status != MFRC522::STATUS_OK) {
Serial.print("❌ Authentication failed (Read) on block ");
Serial.print(blockNum);
Serial.print(": ");
Serial.println(mfrc522.GetStatusCodeName(status));
return;
}

// 📥 Read data from block
status = mfrc522.MIFARE_Read(blockNum, readBlockData, &bufferLen);
if (status != MFRC522::STATUS_OK) {
Serial.print("❌ Reading failed on block ");
Serial.print(blockNum);
Serial.print(": ");
Serial.println(mfrc522.GetStatusCodeName(status));
return;
}

// 🔣 Convert bytes to string and clean up the data
String cardData = String((char*)readBlockData);
cardData.trim();

Serial.print("📄 Card Data: ");
Serial.println(cardData);
}

// 🚀 Setup function
void setup() {
Serial.begin(115200);
SPI.begin();          // 🛠 Initialize SPI communication
mfrc522.PCD_Init();   // 🚀 Initialize RFID reader
Serial.println("🔄 Ready to scan RFID Tag...");

// 🔐 Set default authentication key (0xFF for most cards)
for (byte i = 0; i < 6; i++) {
key.keyByte[i] = 0xFF;
}
}

// 🔄 Main loop function
void loop() {

// 🚫 Return if no card is present or failed to read
if (!mfrc522.PICC_IsNewCardPresent() || !mfrc522.PICC_ReadCardSerial()) {
return;
}

Serial.println("\n🎴 **Card Detected**");

// 🔑 Display Card UID
Serial.print(F("🔹 UID: "));
for (byte i = 0; i < mfrc522.uid.size; i++) {
Serial.print(mfrc522.uid.uidByte[i] < 0x10 ? " 0" : " ");
Serial.print(mfrc522.uid.uidByte[i], HEX);
}
Serial.println();

// 📌 Display Card Type
Serial.print(F("🔸 Type: "));
Serial.println(mfrc522.PICC_GetTypeName(mfrc522.PICC_GetType(mfrc522.uid.sak)));

// ✍️ Write data to card
Serial.println("📝 Writing data to card...");
WriteDataToBlock(blockNum, "13-JASPREET SINGH");  // Example data

// 📖 Read data back from card
Serial.println("📖 Reading data from card...");
ReadDataFromBlock(blockNum);

mfrc522.PICC_HaltA();    // ✋ Halt card
mfrc522.PCD_StopCrypto1();  // 🛑 Stop encryption
delay(2000);  // ⏱ Wait for 2 seconds before next loop
}
```

### **Uploading the Code**
1. Click the Verify (✔) Button to check for errors.
2. Click the Upload (→) Button to upload the code.
3.Wait for "Done Uploading" message.

### **Step 7: Open Serial Monitor
1. Go to Tools → Serial Monitor.
2. Set baud rate to 115200.
3. You should see messages like:
```
🔄 Ready to scan RFID Tag...
🎴 **Card Detected**
🔹 UID: XX XX XX XX
🔸 Type: MIFARE 1K
📝 Writing data to card...
✅ Data written successfully!
📖 Reading data from card...
📄 Card Data: 13-JASPREET SINGH
```

![Capture](https://github.com/user-attachments/assets/e40fda3a-55b3-4cd7-b585-2ca6a501a5bb)


✅ Testing the System
1. Place an RFID tag/card near the RC522 reader.
2. The Serial Monitor should display the UID and Card Type.
3. The predefined data ("13-JASPREET SINGH") will be written to the card.
4. The data will then be read back and displayed on the Serial Monitor.


# **NEXT STEP..
Next, we will integrate Google Sheets to store the scanned RFID data in a cloud-based spreadsheet. This allows real-time logging of card scans, making it useful for applications like attendance systems, access control, and inventory tracking.
To achieve this, we will use Google Apps Script and a web-based interface to send data from the ESP8266 to Google Sheets via an HTTP request. The ESP8266 will send the UID and stored card data to a Google Apps Script endpoint, which will append the received information to a Google Sheet.

### **Steps to Set Up Google Sheets Integration:**
1. Create a New Google Sheet: Open Google Sheets and create a new spreadsheet. Name it RFID Logs.
2. Set Up Columns: In the first row, label the columns as Timestamp, Card UID, and Data to store scan details.
3. Open Google Apps Script: Click Extensions > Apps Script and remove the default code.
4. Insert a Google Apps Script Code: We will write a script to accept incoming HTTP requests from the ESP8266 and append the data to our spreadsheet.Copy this code and paste into your script.

   
```
var ss = SpreadsheetApp.openById('1IudQRvWafJE7iarJkApN6ZI6KuFzmf9Me0cQbuoCDow'); // Google Sheet ID
var sheet = ss.getSheetByName('Sheet1');
var timezone = "Asia/Kolkata"; // Set your timezone

function doGet(e) {
  Logger.log(JSON.stringify(e));

  if (!e.parameter.name || !e.parameter.roll || !e.parameter.status) {
    return ContentService.createTextOutput("Error: Missing 'name', 'roll', or 'status' parameter")
                         .setMimeType(ContentService.MimeType.TEXT);
  }

  var Curr_Date = new Date();
  var Curr_Time = Utilities.formatDate(Curr_Date, "Asia/Kolkata", 'HH:mm:ss');
  var name = stripQuotes(e.parameter.name);
  var roll = stripQuotes(e.parameter.roll);
  var status = stripQuotes(e.parameter.status);  // Now using status from URL

  // Store Data in Google Sheets
  var ss = SpreadsheetApp.openById('1IudQRvWafJE7iarJkApN6ZI6KuFzmf9Me0cQbuoCDow'); // Google Sheet ID
  var sheet = ss.getSheetByName('Sheet1');
  var nextRow = sheet.getLastRow() + 1;
  sheet.getRange("A" + nextRow).setValue(roll);
  sheet.getRange("B" + nextRow).setValue(name);
  sheet.getRange("C" + nextRow).setValue(Utilities.formatDate(Curr_Date, "Asia/Kolkata", 'yyyy-MM-dd'));
  sheet.getRange("D" + nextRow).setValue(Curr_Time);
  sheet.getRange("E" + nextRow).setValue(status); // Now using status from the URL

  // Send Response to ESP8266
  return ContentService.createTextOutput("Roll: " + roll + ", " + name + " marked as " + status)
                       .setMimeType(ContentService.MimeType.TEXT);
}

// ✅ Add the missing stripQuotes function
function stripQuotes(value) {
  return value.toString().replace(/^["']|['"]$/g, ""); // Removes surrounding quotes
}
```


5. Deploy as a Web App: Save and deploy the script to generate a unique URL (webhook) that ESP8266 will use to send data.
6. Modify ESP8266 Code: We will update our ESP8266 code to send card details to the webhook URL whenever an RFID card is scanned.
   
![g](https://github.com/user-attachments/assets/e0622c5e-8a7c-474a-86d4-3c7706d2e7a0)



## ** MAIN CODE**

   
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


   
