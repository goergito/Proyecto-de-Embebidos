# Proyecto-de-Embebidos
# Codigo del proyecto: George Maquilon - Ricardo Romero
#include <Arduino.h>
#include <Wire.h>
#include <ESP32Time.h>
#include <Esp32Servo.h>
#include <LiquidCrystal_I2C.h>
#include <SPI.h>
#include <MFRC522.h>
#include <UniversalTelegramBot.h>
#include <WiFiClientSecure.h>
#include <vector>
#include "FS.h"
#define COLUMS           20   //LCD columns
#define ROWS             4    //LCD rows
#define LCD_SPACE_SYMBOL 0x20 //space symbol from LCD ROM, see p.9 of GDM2004D datasheet
#define LED_PIN_1 25
#define LED_PIN_2 26
#define LED_PIN_3 27
#define RST_PIN         2    // Pin de reinicio (RST) del lector RC522
#define SS_PIN          15   // Pin SS (Slave Select) del lector RC522
const char* ssid = "Nombre de Red";       // Nombre de la red WiFi
const char* password = "Contraseña";   // Contraseña de la red WiFi
LiquidCrystal_I2C lcd(PCF8574_ADDR_A21_A11_A01, 4, 5, 6, 16, 11, 12, 13, 14, POSITIVE);
ESP32Time rtc;
const char* ntpServer="pool.ntp.org";
const long gmtOffset_sec=-5*3600;
const int daylightOffset_sec=0;
int servoPin1 = 32;
int speed = 10;
Servo servo1;
std::vector<String> registeredCards;
MFRC522 mfrc522(SS_PIN, RST_PIN); 
String cardUID = "";
int cont=0;
int cont2=0;
int cont3=0;
int horap=0;
int minp=0;
int horaib=0;
int minib=0;
int horalem=0;
int minlem=0;
int horalor=0;
int minlor=0;

void printTime(LiquidCrystal_I2C& lcd) {
  lcd.print(rtc.getHour(true));
  lcd.print(":");
  lcd.print(rtc.getMinute() < 10 ? "0" : "");
  lcd.print(rtc.getMinute());
}

bool isCardRegistered(const String& cardUID) {
  for (const String& registeredUID : registeredCards) {
    if (registeredUID == cardUID) {
      return true;
    }
  }
  return false;
}

//Credenciales Telegram Bot
const String botToken = "Token del Bot de Telegram";
//Seteo de TelegramBot
WiFiClientSecure secured_client;
UniversalTelegramBot bot(botToken, secured_client);
// Restablecer el estado de los mensajes
bool mensajeEnviado = false;
bool mensajeEnviadoOptimo = false;

void setup() {
Serial.begin(115200);
Serial.printf("Intentando Conectarnos a %s% : ",ssid);
WiFi.begin(ssid, password);   // Conexión a la red WiFi
while (WiFi.status() != WL_CONNECTED) {
  delay(1000);
  Serial.println("Conectando a WiFi...");
}
lcd.setCursor(0,0);
lcd.print("Conexión Wifi");
lcd.setCursor(0,1);
lcd.print("Establecida");
Serial.println("Conexión WiFi establecida!");
Serial.print("Dirección IP asignada: ");
Serial.println(WiFi.localIP());
configTime(gmtOffset_sec,daylightOffset_sec,ntpServer);
struct tm timeinfo;
if (getLocalTime(&timeinfo)){
  rtc.setTimeStruct(timeinfo);
}
pinMode(LED_PIN_1, OUTPUT);
pinMode(LED_PIN_2, OUTPUT);
pinMode(LED_PIN_3, OUTPUT);
digitalWrite(LED_PIN_1, HIGH); 
digitalWrite(LED_PIN_2, HIGH); 
digitalWrite(LED_PIN_3, HIGH); 
while (lcd.begin(COLUMS, ROWS, LCD_5x8DOTS) != 1) //colums, rows, characters size
{
    Serial.println(F("PCF8574 is not connected or lcd pins declaration is wrong. Only pins numbers: 4,5,6,16,11,12,13,14 are legal."));
    delay(5000);   
}
  lcd.clear();
  lcd.print(F("Pantalla Correcta"));    //(F()) saves string to flash & keeps dynamic memory free
  delay(2000);
  lcd.clear();
  SPI.begin(14,12,13,SS_PIN);           // Iniciar comunicación SPI
  mfrc522.PCD_Init();    // Iniciar el lector RC522
  mfrc522.PCD_DumpVersionToSerial();  // Mostrar versión del lector en el monitor serial
  lcd.setCursor(0,0);
  lcd.print("Lector Trabajando");
  lcd.setCursor(0,1);
  lcd.print("Correctamente");
  delay(2000);
  lcd.clear();
  secured_client.setCACert(TELEGRAM_CERTIFICATE_ROOT); //Agregar Certificado raíz para api.telegram 
  // Envia un mensaje al bot de Telegram indicando que el sistema inció
  String chatId = "Chat Id ";
  String message = "¡Bienvenido! Tu Bot de Dispensador de Pastillas te saluda 👨‍⚕️\n🔒 Verifique su Identificación...."; 
   if (bot.sendMessage(chatId, message, "")) {
    Serial.println("Mensaje enviado con éxito.");
   } else {
    Serial.println("Error al enviar el mensaje.");
  }
}

void loop() {
  int numNewMessages = bot.getUpdates(bot.last_message_received + 1); 
  String chatId = "1296289484";
  // Verificar si se detecta una tarjeta RFID
  while(cont==0){
    if (mfrc522.PICC_IsNewCardPresent() && mfrc522.PICC_ReadCardSerial()) {
    // Obtener UID de la tarjeta
    for (byte i = 0; i < mfrc522.uid.size; i++) {
      cardUID += String(mfrc522.uid.uidByte[i], HEX);
    } 
    Serial.print("Tarjeta detectada. UID: ");
    Serial.println(cardUID);
    registeredCards.push_back(cardUID);
    mfrc522.PICC_HaltA();   // Detener comunicación con la tarjeta
    lcd.clear();
    lcd.setCursor(0,0);
    lcd.print("REGISTRADO");
    lcd.setCursor(0,1);
    lcd.print("ACCESO PERMITIDO");
    delay(5000);
    lcd.clear();
    cont+=1;
    }
  }
  delay(2000);
  if (!mensajeEnviado) {
        mensajeEnviadoOptimo = false;
        String message = "¡Bienvenido Enfermero/a!👨‍⚕️\n🔓 Acceso Permitido \nUtilice los siguientes comandos para trabajar:\n/rellenar: Para rellenar el dispensador.\n/horario: Para realizar la programación de horarios.";
        if (bot.sendMessage(chatId, message, "")) {
          Serial.println("Mensaje enviado con éxito.");
        } else {
          Serial.println("Error al enviar el mensaje.");
        }
        mensajeEnviado = true; // Marcar el mensaje como enviado
      }
    for (int i = 0; i < numNewMessages; i++) {
    String chatId = bot.messages[i].chat_id;
    String text = bot.messages[i].text;
    // Procesa el mensaje
    if (text == "/rellenar") {
      String message = "⚠️ Precaución tiene 25 segundos para rellenar cada sección ";
      bot.sendMessage(chatId, message, "");
      String message1 = "Rellenando...";
      bot.sendMessage(chatId, message1, "");
      Serial.begin(115200);
      servo1.attach(servoPin1);
      Serial.println("Rellenando...");
      lcd.clear();
      lcd.setCursor(0, 0);
      lcd.print("Rellenando...");
      lcd.setCursor(0,1);
      lcd.print("Paracetamol");
      String message2 = "💊 Rellenando Paracetamol";
      bot.sendMessage(chatId, message2, "");
      digitalWrite(LED_PIN_1, LOW);  
      digitalWrite(LED_PIN_2, HIGH);
      digitalWrite(LED_PIN_3, HIGH);
      servo1.write(1500 + speed * 5);
      delay(2500);
      servo1.write(90);
      delay(25000);
      //delay(45000);
      lcd.clear();
      lcd.setCursor(0, 0);
      lcd.print("Rellenando...");
      lcd.setCursor(0,1);
      lcd.print("Ibuprofeno");
      String message3 = "💊 Rellenando Ibuprofeno";
      bot.sendMessage(chatId, message3, "");
      digitalWrite(LED_PIN_1, HIGH); 
      digitalWrite(LED_PIN_2, LOW); 
      digitalWrite(LED_PIN_3, HIGH); 
      servo1.write(1500 + speed * 5);
      delay(2550);
      servo1.write(90);
      delay(25000);
      //delay(45000);
      lcd.clear();
      lcd.setCursor(0, 0);
      lcd.print("Rellenando...");
      lcd.setCursor(0,1);
      lcd.print("Lemonflu");
      String message4 = "💊 Rellenando LemonFlu";
      bot.sendMessage(chatId, message4, "");
      digitalWrite(LED_PIN_1, HIGH); 
      digitalWrite(LED_PIN_2, HIGH); 
      digitalWrite(LED_PIN_3, LOW); 
      servo1.write(1500 + speed * 5);
      delay(2475);
      servo1.write(90);
      delay(25000);
      //delay(45000);
      lcd.clear();
      lcd.setCursor(0, 0);
      lcd.print("Rellenando...");
      lcd.setCursor(0,1);
      lcd.print("Loratidina");
      String message5 = "💊 Rellenando Loratidina";
      bot.sendMessage(chatId, message5, "");
      digitalWrite(LED_PIN_1, HIGH); 
      digitalWrite(LED_PIN_2, LOW); 
      digitalWrite(LED_PIN_3, LOW); 
      servo1.write(1500 + speed * 5);
      delay(2370);
      servo1.write(90);
      delay(25000);
      //delay(45000);
      lcd.clear();
      lcd.setCursor(0, 0);
      lcd.print("Relleno");
      lcd.setCursor(0,1);
      lcd.print("Exitoso");
      String message6 = "🟩 ¡Relleno Exitoso!";
      bot.sendMessage(chatId, message6, "");
      digitalWrite(LED_PIN_1, HIGH); 
      digitalWrite(LED_PIN_2, HIGH); 
      digitalWrite(LED_PIN_3, HIGH); 
      servo1.write(1500 + speed * 5);
      delay(2500);
      //delay(10000);
      lcd.clear();
    }
    if (text=="/horario"){
      String message = "⏰Inserte el horario para las pastillas con:\n/sethora1 H\n/setminutos1 M";
      bot.sendMessage(chatId, message, "");
      while(cont2==0){
        if (text.startsWith("/sethora1 ")) {
            horap = text.substring(10).toInt(); // Extrae la hora del mensaje
            bot.sendMessage(chatId, "✅Hora de Paracetamol configurada: " + String(horap));
          } else if (text.startsWith("/setminutos1 ")) {
            minp = text.substring(13).toInt(); // Extrae los minutos del mensaje
            bot.sendMessage(chatId, "✅Minutos de Paracetamol configurados: " + String(minp));
          } else if (text.startsWith("/sethora2 ")) {
            horaib = text.substring(13).toInt(); // Extrae los minutos del mensaje
            bot.sendMessage(chatId, "✅Hora de Ibuprofeno configurada: " + String(horaib));
          } else if (text.startsWith("/setminutos2 ")) {
            minib = text.substring(13).toInt(); // Extrae los minutos del mensaje
            bot.sendMessage(chatId, "✅Minutos de Ibuprofeno configurados: " + String(minp));
          } else if (text.startsWith("/sethora3 ")) {
            horalem = text.substring(13).toInt(); // Extrae los minutos del mensaje
            bot.sendMessage(chatId, "✅Hora de Lemonflu configurada: " + String(horalem));
          } else if (text.startsWith("/setminutos3 ")) {
            minlem = text.substring(13).toInt(); // Extrae los minutos del mensaje
            bot.sendMessage(chatId, "✅Minutos de Lemonflu configurados: " + String(minlem));
          } else if (text.startsWith("/sethora4 ")) {
            horalor = text.substring(13).toInt(); // Extrae los minutos del mensaje
            bot.sendMessage(chatId, "✅Hora de Loratidina configurada: " + String(horalor));
          } else if (text.startsWith("/setminutos4 ")) {
            minlor = text.substring(13).toInt(); // Extrae los minutos del mensaje
            bot.sendMessage(chatId, "✅Minutos de Loratidina configurados: " + String(minlor));
            cont2+=1;
            cont3+=1;
          }
      }
    }
    if(cont3==1){
      Serial.print(horap);
      Serial.print(minp);
      Serial.print(horaib);
      Serial.print(minib);
      Serial.print(horalem);
      Serial.print(minlem);
      Serial.print(horalor);
      Serial.print(minlor);
      String message = "⏰Esperando Horario....";
      bot.sendMessage(chatId, message, "");
      while(cont2==1){
      if (rtc.getHour(true) == horap && rtc.getMinute() ==minp ) {
          String message = "💊 Dispensando Paracetamol";
          bot.sendMessage(chatId, message, "");
          lcd.clear();
          lcd.setCursor(0, 0);
          lcd.print("Horario: ");
          printTime(lcd);
          lcd.setCursor(0, 1);
          lcd.print("");
          lcd.print("Paracetamol");
      } 
      else if (rtc.getHour(true) == horap && rtc.getMinute() ==minp ) {
          String message = "💊 Dispensando Ibuprofeno";
          bot.sendMessage(chatId, message, "");          
          lcd.clear();
          lcd.setCursor(0, 0);
          lcd.print("Horario: ");
          printTime(lcd);
          lcd.setCursor(0, 1);
          lcd.print("");
          lcd.print("Ibuprofeno");
      } 
      else if (rtc.getHour(true) == horap && rtc.getMinute() ==minp ) {
          String message = "💊 Dispensando Lemonflu";
          bot.sendMessage(chatId, message, "");
          lcd.clear();
          lcd.setCursor(0, 0);
          lcd.print("Horario: ");
          printTime(lcd);
          lcd.setCursor(0, 1);
          lcd.print("");
          lcd.print("LemonFlu");
      } 
      else if (rtc.getHour(true) == horap && rtc.getMinute() ==minp ) {
          String message = "💊 Dispensando Loratidina";
          bot.sendMessage(chatId, message, "");
          lcd.clear();
          lcd.setCursor(0, 0);
          lcd.print("Horario: ");
          printTime(lcd);
          lcd.setCursor(0, 1);
          lcd.print("");
          lcd.print("Loratidina");
          cont2+=1;
      } 
    }
  }
    delay(1000);
  }
}
