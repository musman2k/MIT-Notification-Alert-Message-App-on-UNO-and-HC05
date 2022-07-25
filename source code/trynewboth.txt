#include <Filters.h>
#include <Filters/Butterworth.hpp>

unsigned long pre = 0;
long pre2 = 0;
long pre3 = 0;
int count = 0, store = 0, pot_val = 0;
int count2 = 0;
int count3 = 0;
long val = 0;
bool b = false;
const int GSR = A1;
int sensorValue;
double HResist;
float BASELINE;
float PHASIC;
float TOTAL;
 

void setup() {
 Serial.begin(9600);
  store = sta(A0);
}
    // Sampling frequency
const double f_s = 10; // Hz
// Cut-off frequency (-3 dB)
const double f_c = 0.05; // Hz
 
// Butterworth filter
auto filter = butter<2>(f_c); //
  


void loop() {
  pot_val = sta(A0);

  unsigned long s_t = millis();

  //Serial.println(count);
  if (pot_val == store) {

    if (count > 10 && b == false) {
      //Serial.println("Stopped breathing ALERT");
      b = true;
    }
    count2 = 0;
    if (s_t - pre >= 1000) {
      count++;
      if (count > 12) count = 12;
      pre = s_t;
    }

  }
  else {

    if (store < pot_val) {
      pre2 = millis();
      //Serial.print("UP  ");
      count2++;

    }
    if (store > pot_val) {
      pre3 = millis();
      //Serial.print("Down  ");
      count2--;
    }

    if (count2 > 4 || count2 < -4 ) {
      //Serial.println("Alart msg abnormal condition");
    }
    if (count2 == 1 || count2 == -1) {
      //Serial.println("Normal condition");
    }

    //Serial.println( count2);
    count = 0;
    b = false;
  }
  delay(20);
  
  store = pot_val;

 sensorValue = analogRead(GSR);
  HResist = 10000 * ((1024.0 + 2.0 * sensorValue) ) / (512 - sensorValue);
  HResist = HResist / 1E6;// to convert to MegOhm
  //Serial.print("    EDA = ");
  //Serial.print(1.0 / HResist, 4);
  //Serial.print(" microSiemens  ");
 
  float BASELINE = filter(1.0 / HResist);
  //Serial.print("    BASELINE = ");
  //Serial.print(BASELINE, 4);
  PHASIC = (1.0 / HResist) - BASELINE;
  //Serial.print("    PHASIC = ");
 // Serial.print(PHASIC, 4);
 // Serial.println();
  

//both condition // THIS IS SENT TO THE APPLICATION
if (PHASIC > 20 && (((count2 > 4 || count2 < -4 ))  || (count > 10 && b == false))) { 
    Serial.print("seizure!!!");
    Serial.println();

   if (PHASIC > 15) {
      Serial.print("Pre detection!!!");
    Serial.println();
    }
}



}

int sta(const byte pin)
{
  static int pre = 0; // static is better than a global

  int new_val = analogRead(pin);
  if ((new_val - 8 > pre) || (new_val + 8 < pre))
  {
    pre = new_val;
  }
  return pre;
}
