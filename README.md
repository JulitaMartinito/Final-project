//Transmisor

//Include Libraries
#include <SPI.h>
#include <nRF24L01.h>
#include <RF24.h>

#define BUF 20

//variables
int average[20];
int pos = 0;

int average2[100];
int pos2 = 0;

int soundPin = 6;
int sensorValue = 0;

//create an RF24 object
RF24 radio(9, 8);  // CE, CSN

//address through which two modules communicate.
const byte address[6] = "Marti";

void setup()
{
  radio.begin();

  //set the address
  radio.openWritingPipe(address);

  //Set module as transmitter
  radio.stopListening();

  Serial.begin (9600);
}

void loop()
{

  int sensorValue = analogRead(A0);
  // print out the value you read
  addValue(sensorValue);
  sensorValue = abs(sensorValue - getMean());

  //define the value in to 100
  average2[pos2] = sensorValue;
  pos2++;

  //Define the only values we want
  if (pos2 >= 100) {
    long mean = 0;
    for (int i = 0; i < 100; i++) {
      mean = mean + average2[i];
    }

    //Takes all the new values and tranmits the information
    sensorValue = mean / 100;
    radio.write(&sensorValue, sizeof(sensorValue));
    pos2 = 0;
  }

  delay(1);
}

void addValue(int val) {
  average[pos] = val;
  pos++;
  if (pos >= BUF) {
    pos = 0;
  }
}

//Our body only detects certan variations of value, heare we divide al the values that a microphone in to values that the people dectect
int getMean() {
  long mean = 0;
  for (int i = 0; i < BUF; i++) {
    mean = mean + average[i];
  }
  return mean / BUF;
}


//Reciver

int vibrador = 3;

//Include Libraries
#include <SPI.h>
#include <nRF24L01.h>
#include <RF24.h>

//create an RF24 object
RF24 radio(9, 8);  // CE, CSN

//address through which two modules communicate.
const byte address[6] = "Marti";


void setup()
{
  while (!Serial);
  Serial.begin(9600);

  radio.begin();

  //set the address
  radio.openReadingPipe(0, address);

  //Set module as receiver
  radio.startListening();

  pinMode(vibrador, OUTPUT);
}

void loop()
{
  //Read the data if available in buffer
  if (radio.available())
  {
    //Recive the values of the transmisor
    int sensorValue = 0;
    radio.read(&sensorValue, sizeof(sensorValue));

    //Transforms the value of the microphone to a value between the rank of the vibration
    sensorValue = map(sensorValue, 0, 10, 0, 255);
    sensorValue = constrain(sensorValue, 0, 255);

    Serial.println(sensorValue);

    //Makes the vibratior shakes
    analogWrite (vibrador, sensorValue);
  }
}
