# RFID-BASED-ATTENTECE-SYSTEM
#include <WiFiClientSecure.h>
#include <LiquidCrystal_I2C.h>
#include "Wire.h"
LiquidCrystal_I2C lcd (0x27, 16, 2);

const char* ssid = "ABINESH";
const char* password =  "12345678";
int keyIndex = 0;            // your network key Index number (needed only for WEP)


//----------------------------------------Host & httpsPort
const char* host = "script.google.com";
const int httpsPort = 443;
//----------------------------------------

WiFiClientSecure client; //--> Create a WiFiClientSecure object.

String GAS_ID = "AKfycbwmMbGR_Gr2mrfEza5s-g1otZUvBQosvwVdS6l0_Yl7pdbxudNe0Z3pSq_8QPmzS9MP"; //--> spreadsheet script ID
String NAME;
int quantity, ratepi, overall_total;



//timing code
#include "Wire.h"
#define DS3231_I2C_ADDRESS 0x68
// Convert normal decimal numbers to binary coded decimal
byte decToBcd(byte val) {
  return ( (val / 10 * 16) + (val % 10) );
}
// Convert binary coded decimal to normal decimal numbers
byte bcdToDec(byte val) {
  return ( (val / 16 * 10) + (val % 16) );
}
//reader code
int count = 0;
char card_no[12];
int sw = 25;
int Exit = 0;
char* rfid_id[] = {"5300A556CD6D", "5300A96D6DFA", "5300A566E373", "5300A5579B3A", "5300A567B425"};
char* names[] = {"Richard", "Robin", "Marco", "Abinesh", "Alice"};
int presence[10];


void setup()
{
  Wire.begin();
  Serial.begin(9600);
  // set the initial time here:
  // DS3231 seconds, minutes, hours, day, date, month, year
  setDS3231time(40, 40, 8,1, 2, 10, 22);
}
void setDS3231time(byte second, byte minute, byte hour, byte dayOfWeek, byte
                   dayOfMonth, byte month, byte year) {
  // sets time and date data to DS3231
  Wire.beginTransmission(DS3231_I2C_ADDRESS);
  Wire.write(0); // set next input to start at the seconds register
  Wire.write(decToBcd(second)); // set seconds
  Wire.write(decToBcd(minute)); // set minutes
  Wire.write(decToBcd(hour)); // set hours
  Wire.write(decToBcd(dayOfWeek)); // set day of week (1=Sunday, 7=Saturday)
  Wire.write(decToBcd(dayOfMonth)); // set date (1 to 31)
  Wire.write(decToBcd(month)); // set month
  Wire.write(decToBcd(year)); // set year (0 to 99)
  Wire.endTransmission();
}
void readDS3231time(byte *second,
                    byte *minute,
                    byte *hour,
                    byte *dayOfWeek,
                    byte *dayOfMonth,
                    byte *month,
                    byte *year) {
  Wire.beginTransmission(DS3231_I2C_ADDRESS);
  Wire.write(0); // set DS3231 register pointer to 00h
  Wire.endTransmission();
  Wire.requestFrom(DS3231_I2C_ADDRESS, 7);
  // request seven bytes of data from DS3231 starting from register 00h
  *second = bcdToDec(Wire.read() & 0x7f);
  *minute = bcdToDec(Wire.read());
  *hour = bcdToDec(Wire.read() & 0x3f);
  *dayOfWeek = bcdToDec(Wire.read());
  *dayOfMonth = bcdToDec(Wire.read());
  *month = bcdToDec(Wire.read());
  *year = bcdToDec(Wire.read());
}
void displayTime() {
  byte second, minute, hour, dayOfWeek, dayOfMonth, month, year;
  // retrieve data from DS3231
  readDS3231time(&second, &minute, &hour, &dayOfWeek, &dayOfMonth, &month,
                 &year);
  // send it to the serial monitor
  Serial.print(hour, DEC);
  // convert the byte variable to a decimal number when displayed
  Serial.print(":");
  if (minute < 10) {
    Serial.print("0");
  }
  Serial.print(minute, DEC);
  Serial.print(":");
  if (second < 10) {
    Serial.print("0");
  }
  Serial.print(second, DEC);
  Serial.print(" ");
  Serial.print(dayOfMonth, DEC);
  Serial.print("/");
  Serial.print(month, DEC);
  Serial.print("/");
  Serial.print(year, DEC);
  Serial.print(": ");
  switch (dayOfWeek) {
    case 1:
      Serial.println("Sunday");
      break;
    case 2:
      Serial.println("Monday");
      break;
    case 3:
      Serial.println("Tuesday");
      break;
    case 4:
      Serial.println("Wednesday");
      break;
    case 5:
      Serial.println("Thursday");
      break;
    case 6:
      Serial.println("Friday");
      break;
    case 7:
      Serial.println("Saturday");
      break;
  }
  // Initialize the LCD connected
  lcd. init ();

  // Turn on the backlight on LCD.
  lcd. backlight ();

  Serial.begin(9600);
  pinMode(23, OUTPUT);
  pinMode(15, OUTPUT);
  pinMode(33, OUTPUT);
  pinMode(sw, INPUT);
  lcd.clear();
  Serial.begin(9600);  //Initialize serial
  while (!Serial)
{
    ; // wait for serial port to connect. Needed for Leonardo native USB port only
}
  
  WiFi.mode(WIFI_STA);   
}

void loop()
{
   // Connect or reconnect to WiFi
     if(WiFi.status() != WL_CONNECTED)
     {
     Serial.print("Attempting to connect to SSID: ");
     }
     while(WiFi.status() != WL_CONNECTED)
     {
      WiFi.begin(ssid, password);  // Connect to WPA/WPA2 network. Change this line if using open or WEP network
      Serial.print(".");
      delay(15000);      
      Serial.println("\nConnected.");
     } 
  char response[12];
  if (Serial.available())
  {
    count = 0;
    while (Serial.available() && count < 12)
    {
      card_no[count] = Serial.read();
      count++;
      delay(5);
      if (strcmp(card_no, rfid_id[0]) == 0 && Exit != 0)
      {

        lcd.setCursor(0, 0);
        lcd.print("Welcome ! ");
        delay(10);
        lcd.setCursor(0, 1);
        lcd.println("Richard          ");
        Serial.println("Welocome ! ");
        Serial.println(  names[0]);
        displayTime(); // display the real-time clock data on the Serial Monitor,
        delay(10);
        digitalWrite(23, HIGH);
        digitalWrite(15, HIGH);
        delay(600);
        digitalWrite(23, LOW);
        digitalWrite(15, LOW);
        NAME="Richard";
      }
      else if (strcmp(card_no, rfid_id[1]) == 0 && Exit != 0)
      {
        lcd.setCursor(0, 0);
        lcd.print("Welcome ! ");
        delay(10);
        lcd.setCursor(0, 1);
        lcd.print(  names[1]);
        Serial.println("Welocome ! ");
        Serial.println(names[1]);
        displayTime(); // display the real-time clock data on the Serial Monitor,
        delay(10);
        digitalWrite(23, HIGH);
        digitalWrite(15, HIGH);
        delay(1000);
        digitalWrite(23, LOW);
        digitalWrite(15, LOW);
         NAME="ROBIN";
      }
      else if (strcmp(card_no, rfid_id[2]) == 0 && Exit != 0)
      {
        lcd.setCursor(0, 0);
        lcd.print("Welcome ! ");
        delay(10);
        lcd.setCursor(0, 1);
        lcd.print(  names[2]);
        Serial.println("Welocome ! ");
        Serial.println(names[2]);
        displayTime(); // display the real-time clock data on the Serial Monitor,
        delay(10);
        digitalWrite(23, HIGH);
        digitalWrite(15, HIGH);
        delay(1000);
        digitalWrite(23, LOW);
        digitalWrite(15, LOW);
         NAME="Marco";
      }
      else if (strcmp(card_no, rfid_id[3]) == 0 && Exit != 0)
      {
        lcd.setCursor(0, 0);
        lcd.print("Welcome ! ");
        delay(10);
        lcd.setCursor(0, 1);
        lcd.print(  names[3]);
        Serial.println("Welocome ! ");
        Serial.println(names[3]);
        displayTime(); // display the real-time clock data on the Serial Monitor,
        delay(10);
        digitalWrite(23, HIGH);
        digitalWrite(15, HIGH);
        delay(1000);
        digitalWrite(23, LOW);
        digitalWrite(15, LOW);
         NAME="ABINESH";
      }
      else if (strcmp(card_no, rfid_id[4]) == 0 && Exit != 0)
      {
        lcd.setCursor(0, 0);
        lcd.print("Welcome ! ");
        delay(10);
        lcd.setCursor(0, 1);
        lcd.print(  names[4]);
        Serial.println("Welocome ! ");
        Serial.println(names[4]);
        displayTime(); // display the real-time clock data on the Serial Monitor,
        delay(10);
        digitalWrite(23, HIGH);
        digitalWrite(15, HIGH);
        delay(1000);
        digitalWrite(23, LOW);
        digitalWrite(15, LOW);
         NAME="ALICE";
      }
    }
     



 sendData(NAME); //--> Calls the sendData Subroutine
  }







}
// Subroutine for sending data to Google Sheets
void sendData(String NAME) {
  Serial.println("==========");
  Serial.print("connecting to ");
  Serial.println(host);

  //----------------------------------------Connect to Google host
  if (!client.connect(host, httpsPort)) {
    Serial.println("connection failed");
    return;
  }
  //----------------------------------------

  //----------------------------------------Processing data and sending data
  String url = "/macros/bitsathy.ac.in/s/" + GAS_ID + "/exec?item=" + NAME ;
  Serial.print("requesting URL: ");
  Serial.println(url);

  client.print(String("GET ") + url + " HTTP/1.1\r\n" +
               "Host: " + host + "\r\n" +
               "User-Agent: BuildFailureDetectorESP32\r\n" +
               "Connection: close\r\n\r\n");

  Serial.println("request sent");
  //----------------------------------------

  //----------------------------------------Checking whether the data was sent successfully or not
  while (client.connected()) {
    String line = client.readStringUntil('\n');
    if (line == "\r") {
      Serial.println("headers received");
      break;
    }
  }
  String line = client.readStringUntil('\n');
  if (line.startsWith("{\"state\":\"success\"")) {
    Serial.println("esp32/Arduino CI successfull!");
  } else {
    Serial.println("esp32/Arduino CI has failed");
  }
  Serial.print("reply was : ");
  Serial.println(line);
  Serial.println("closing connection");
  Serial.println("==========");
  Serial.println();
  //----------------------------------------

}
