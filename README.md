#include <Wire.h>
#include <Adafruit_MFRC522.h>
#include <esp_camera.h>

#define SDA_PIN 2
#define SCL_PIN 3
#define RST_PIN 9 // Reset pin for MFRC522

#define CAMERA_MODEL_AI_THINKER
#include "camera_pins.h"

Adafruit_MFRC522 mfrc522(SDA_PIN, RST_PIN);

const int trigPin = 9;  // Connect HCS04 trig pin to Arduino pin 9
const int echoPin = 10; // Connect HCS04 echo pin to Arduino pin 10

void setup() {
  Serial.begin(9600);
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);
  
  Wire.begin(SDA_PIN, SCL_PIN);
  mfrc522.PCD_Init();

  camera_init();
  esp_camera_fb_config_t fb_config;
  fb_config.pixel_format = PIXFORMAT_JPEG;
  fb_config.frame_size = FRAMESIZE_CIF;
  fb_config.jpeg_quality = 10;
  fb_config.fb_count = 1;
  camera_fb = esp_camera_fb_get(&fb_config);
}

void loop() {
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);

  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);

  long duration = pulseIn(echoPin, HIGH);
  int distance = duration * 0.034 / 2;

  Serial.print("Distance: ");
  Serial.print(distance);
  Serial.println(" cm");

  if (distance < 10) {
    if (mfrc522.PICC_IsNewCardPresent() && mfrc522.PICC_ReadCardSerial()) {
      Serial.println("Card detected:");
      printCardSerial();
    } else {
      captureAndSendPhoto();
    }
  }

  delay(1000); // Delay before next measurement
}

void printCardSerial() {
  for (byte i = 0; i < mfrc522.uid.size; i++) {
    Serial.print(mfrc522.uid.uidByte[i] < 0x10 ? " 0" : " ");
    Serial.print(mfrc522.uid.uidByte[i], HEX);
  }
  Serial.println();
  mfrc522.PICC_HaltA();
  mfrc522.PCD_StopCrypto1();
}

void captureAndSendPhoto() {
  camera_fb = esp_camera_fb_get();
  if (!camera_fb) {
    Serial.println("Camera capture failed");
    return;
  }
  
  // Send the captured photo to a server or store it as needed
  // Implement your logic here
  
  esp_camera_fb_return(camera_fb);
}

