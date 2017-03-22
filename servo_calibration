#include <Servo.h> //import servo library
#include <EEPROM.h> //import EEPROM library
#include <PID_v1.h>

const int numServos = 3; //how many servos do you have?
const int sPin[numServos] = {9, 10, 11}; //what pins do they correlate to?
Servo servo[numServos]; //declare the servo array
int highPulse[numServos]; //high pulse width
int lowPulse[numServos]; //low pulse width
int A[numServos]; //lowPulse feedback reading
int B[numServos]; //highPulse feedback reading
int here[numServos]; //angle moving from
int there[numServos]; //angle moving to

float x[numServos]; //angle converted to radians to derive cosine wave

int h;
int t;
int feedBack; //used to hold servo feedback value
int Button = 2;

int e = 1; //EEPROM address to start storing/retrieving servo data
int btwReadings = 20; //delay time between
int whlReading = 3; //delay time between analog readings of internal pot
boolean rangeTest = false;
boolean doneMove[numServos];

void setup()
{

	Serial.begin(19200); // initialize serial output
	Serial.println("it's on!");
	pinMode (Button, INPUT);

	analogReference(EXTERNAL); //set to ARef

	for(int i = 0; i < numServos ; i++)  //attach servos
	{
		servo[i].attach(sPin[i]);
	}

	int n = EEPROM.read(0); //check EEPROM for recorded Servo Range
	if (n == 20)
	{
		callRange(); //If range is recorded, retrieve range
	}
	if (rangeTest == false)
	{
		for (int i = 0; i < numServos; i++)
		{
			setRange(i); // if no range is recorded, go test the range and set the values
			doneMove[i] = true;
		}
		rangeTest = true;
		EEPROM.write(0, 20); //indicate future startups that we've done this!
	}
	delay(1000);
}


void loop() {}


/* This Function callibrates and sets the range
for each servo defined and recoreds them to EEPROM */


void setRange(int x) //parameter passed is array position of servo
{
	int pb = 0; //used to hold push button reading
	int h;
	int t;
	int pulse = 1500; //first uS pulse used in range test
	Serial.print("Press button to set range of Servo[");
	Serial.print(x);
	Serial.println("].");

	while(!pb)
	{
		pb = digitalRead(Button);
	}

	pb = 0;
	Serial.print("Setting range limits in ..3");

	for (int i = 2; i >= 0; i--) //count down three seconds
	{
		delay(1000);
		Serial.print("..");
		Serial.print(i);
	}

	Serial.println();
	servo[x].attach(sPin[x]);
	delay(20);
	servo[x].writeMicroseconds(pulse); //send servo to middle of range
	delay(2000); //wait for it to get there
	h = getFeedback(x);
	t = h - 10;
	Serial.print("t,");
	Serial.println(t);
	Serial.print("h,");
	Serial.println(h);

	do
	{
		t = getFeedback(x);
		pulse += 30; //incriment uS pulse width
		readMove(x, pulse);
		delay(200);
		h = getFeedback(x);
		Serial.print("t,");
		Serial.println(t);
		Serial.print("h,");
		Serial.println(h);
		
	}
	while(h > t);  //condition to keep testing range

	highPulse[x] = pulse - 20; //stay away from range extreme
	B[x] = h - 10; //adjust feedback away from extreme
	Serial.println();
	servo[x].writeMicroseconds(highPulse[x]);
	pulse = highPulse[x];
	delay(500);

	do
	{
		pulse -= 10;
		readMove(x, pulse);
	}
	while(h < t);

	lowPulse[x] = pulse + 20;
	A[x] = t + 10;
	servo[x].writeMicroseconds(lowPulse[x]);

	feedBack = getFeedback(x); //take current reading from pot
	there[x] = map(feedBack, A[x], B[x], 0, 180); //adjust feedback to degree output
	servo[x].attach(sPin[x], lowPulse[x], highPulse[x]); //attach this servo
	servo[x].write(there[x]); //send out pulse for where we are

	doneMove[x] = true;


	//writeWordE(A[x]); //store low feedback reading
	//writeWordE(B[x]); // store high feedback reading

	//writeWordE(lowPulse[x]);  //store low control pulse
	//writeWordE(highPulse[x]); //store high control pulse

	Serial.println("Feedback Range:");
	Serial.print(A[x]);
	Serial.print(" <> ");
	Serial.println(B[x]);
	Serial.println("uS Pulse Range:");
	Serial.print(lowPulse[x]);
	Serial.print(" <> ");
	Serial.println(highPulse[x]);
	Serial.print("Servo[");
	Serial.print(x);
	Serial.println("] attached, data saved in EEPROM");

}//end setRange()



/* This Function retrieves previously recorded servo callibrations
**The same servos must be on the same pins of the same board as when callibrated*/
void callRange()
{
	Serial.print("To reset saved range press button on pin 2 ");
	for (int i = 5; i >= 0; i--)
	{
		Serial.print("..");
		Serial.print(i);
		for (int j = 0; j < 100; j++)
		{
			if (digitalRead(Button) == 1)
			{
				Serial.println();
				delay(1000);
				return;
			}
			delay(10);
		}
	}
	Serial.println();
	Serial.println("Retreiving servo data");
	for (int i = 0; i < numServos; i++)
	{

		A[i] = readWordE(); //get stored low feedback reading
		B[i] = readWordE(); //get stored high feedback reading
		lowPulse[i] = readWordE(); //get storeed low control pulse
		highPulse[i] = readWordE(); //get stored high control pulse
		feedBack = getFeedback(i); //take current reading from pot
		there[i] = map(feedBack, A[i], B[i], 0, 180); //adjust feedback to degree output
		servo[i].attach(sPin[i], lowPulse[i], highPulse[i]); //attach this servo
		servo[i].write(there[i]); //send out pulse for where we are
		doneMove[i] = true; //set up to make first move
//    Serial.println("Feedback Range:");
//    Serial.print(A[i]);
//    Serial.print(" <> ");
//    Serial.println(B[i]);
//    Serial.println("uS Pulse Range:");
//    Serial.print(lowPulse[i]);
//    Serial.print(" <> ");
//    Serial.println(highPulse[i]);
//    Serial.print("Servo[");
//    Serial.print(i);
//    Serial.println("] attached, data retrieved from EEPROM");
//    Serial.print("servo ");
//    Serial.print(i);
//    Serial.print(" current position = ");
//    Serial.println(there[i]);
//    Serial.println();
//
	}

	rangeTest = true; //set the rangeTest flag
} //End callRange()


/* this function moves the servo and updates the "here" variable
 *  it is only used for callibration
 */ 
void readMove(int n, int p)
{
	//t = getFeedback(n);
	//Serial.print("T,");
	//Serial.println(t);
	servo[n].writeMicroseconds(p);
	delay(200);
	//h = getFeedback(n);
	//Serial.print("H,");
	//Serial.println(h);
} //end readMove()


/* this function takes a number of measurements from
 *  the position feedback signal, discards the high and
 *  low values, and averages the rest
 */

int getFeedback(int a)
{

	int j;
	int mean;
	int result;
	int test;
	int reading[20];
	boolean done;

	for (j = 0; j < 20; j++)
	{
		reading[j] = analogRead(a); //get raw data from servo potentiometer
		delay(whlReading);
	} // sort the readings low to high in array

	done = false; // clear sorting flag
	while(done != true)  // simple swap sort, sorts numbers from lowest to highest
	{
		done = true;
		for (j = 0; j < 20; j++)
		{
			if (reading[j] > reading[j + 1])  // sorting numbers here
			{
				test = reading[j + 1];
				reading [j + 1] = reading[j] ;
				reading[j] = test;
				done = false;
			}
		}
	}
	mean = 0;
	for (int k = 6; k < 14; k++) //discard the 6 highest and 6 lowest readings
	{
		mean += reading[k];
	}
	result = mean / 8; //average useful readings
	return(result);
} //end getFeedback()



/*this function reads two consecutive bytes from the EEPROM,
concatenates them into a Word, and returns that value*/
int readWordE()
{
	int A;
	int E;
	A = EEPROM.read(e);
	E = A << 8;
	e += 1;
	E = E + EEPROM.read(e);
	e += 1;
	return E;
}

/* this function takes a 16 bit word, parses it
 *and saves it in consecutive EEPROM addresses
 */
void writeWordE(int b)
{
	int A;
	A = b >> 8;
	EEPROM.write(e, A);
	e += 1 ;
	EEPROM.write(e, b);
	e += 1 ;
}

