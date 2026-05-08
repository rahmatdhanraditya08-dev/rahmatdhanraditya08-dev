/*
   MODBUS RTU FLOAT 32 BIT
   ESP32 MASTER -> PLC
*/

#include "DFRobot_RTU.h"

#define RXD2 16
#define TXD2 17

// Modbus via Serial2
DFRobot_RTU modbus_Data(&Serial2);

// Variabel float
float TempNo_1;
float TempNo_2;

void setup() {

  Serial.begin(115200);

  // Baudrate 9600, SERIAL_8N1
  Serial2.begin(9600, SERIAL_8N1, RXD2, TXD2);

  Serial.println("MODBUS FLOAT TEST");
}

void loop() {

  TempNo1();
  TempNo2();

  delay(3000);
}


// ======================================================
// FUNCTION TEMP 1
// ======================================================

void TempNo1() {

  // Generate data random float
  TempNo_1 = random(1, 70) / 2.30f;

  Serial.println("==========================");
  Serial.print("TempNo_1 : ");
  Serial.println(TempNo_1);

  // Buffer register 16 bit x2
  uint16_t WriteRegister[2] = {0, 0};

  /*
     Copy data float ke buffer register
     sizeof(float) = 4 byte = 32 bit
  */
  memcpy(WriteRegister, &TempNo_1, sizeof(float));

  /*
     Kirim ke PLC
     Register:
     D15 = low word
     D16 = high word
  */

  modbus_Data.writeHoldingRegister(1, 15, WriteRegister[0]);
  modbus_Data.writeHoldingRegister(1, 16, WriteRegister[1]);

  delay(500);

  // ===============================
  // READ BACK FLOAT
  // ===============================

  uint16_t ReadRegister[2] = {0, 0};

  ReadRegister[0] =
    modbus_Data.readHoldingRegister(1, 15);

  ReadRegister[1] =
    modbus_Data.readHoldingRegister(1, 16);

  float ReadFloat;

  memcpy(&ReadFloat, ReadRegister, sizeof(float));

  Serial.print("Read Float PLC : ");
  Serial.println(ReadFloat);

  Serial.println("==========================");
}



// ======================================================
// FUNCTION TEMP 2
// ======================================================

void TempNo2() {

  // Generate data random float
  TempNo_2 = random(10, 90) / 1.75f;

  Serial.println("==========================");
  Serial.print("TempNo_2 : ");
  Serial.println(TempNo_2);

  uint16_t WriteRegister2[2] = {0, 0};

  memcpy(WriteRegister2, &TempNo_2, sizeof(float));

  /*
     Kirim ke PLC
     D17 = low word
     D18 = high word
  */

  modbus_Data.writeHoldingRegister(1, 17, WriteRegister2[0]);
  modbus_Data.writeHoldingRegister(1, 18, WriteRegister2[1]);

  delay(500);

  // ===============================
  // READ BACK FLOAT
  // ===============================

  uint16_t ReadRegister2[2] = {0, 0};

  ReadRegister2[0] =
    modbus_Data.readHoldingRegister(1, 17);

  ReadRegister2[1] =
    modbus_Data.readHoldingRegister(1, 18);

  float ReadFloat2;

  memcpy(&ReadFloat2, ReadRegister2, sizeof(float));

  Serial.print("Read Float PLC : ");
  Serial.println(ReadFloat2);

  Serial.println("==========================");
}
