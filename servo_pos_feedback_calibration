/* >
This code finds and records the limit for each servo being tested, using
a position feedback signal
it increments up, then down, until DeltaPos (h-t) ~= 0 at each limit
for accuract, it waits 200ms between each reading, and smooths the
position input from a poll of 100 readings, discarding the oulying 15%
readings at each extreme
The outer limit pulse and position for each servo are then recorded in
an array
Dillon MacEwan
*/


#include <Servo.h> //import servo library


const int numServos = 3; //how many servos do you have?
const int sPin[numServos] = {3, 5, 6}; //what pins do they correlate to?
Servo servo[numServos]; //declare the servo array

#define Button 2

int highPulse[numServos]; //high pulse width
int lowPulse[numServos]; //low pulse width
int A[numServos]; //lowPulse feedback reading
int B[numServos]; //highPulse feedback reading
int here[numServos]; //angle moving from
int there[numServos]; //angle moving to

String joints[numServos] = {"Coxia", "Femur", "Tibia"}; //name of servos

float x[numServos]; //angle converted to radians to derive cosine wave

int h; //current pot position
int t; //previous pot position
int feedBack; //used to hold servo feedback value


int angle; //angle derived from cosine function. sent to servo in loop

int whlReading = 2; //delay time between analog readings of internal pot

boolean rangeTest = false;
boolean doneMove[numServos];

void setup()
{

	Serial.begin(19200); // initialize serial output
	Serial.println("it's on!");
	analogReference(EXTERNAL);
	pinMode (Button, INPUT);
	for (int i = 0; i < numServos; i++)
	{
		servo[i].attach(sPin[i]);
	}


	for (int i = 0; i < numServos; i++)
	{
		setRange(i); //go test the range and set the values
		doneMove[i] = true;
	}

	delay(1000);
}

void loop()
{

	for (int i = 0; i < numServos; i++)
	{
		if(doneMove[i] == true)
		{
			doneMove[i] = false;
			here[i] = there[i];
			there[i] = random(180.1) + 0.5;
			if(there[i] == here[i])
			{
				there[i] = random(180.1) + 0.5;
			}
			if(here[i] < there[i])
			{
				x[i] = 0;
			}
			else
			{
				x[i] = 180;
			}
			Serial.print("Move ");
			Serial.print(joints[i]);
			Serial.print(" from ");
			Serial.print(here[i]);
			Serial.print(" to ");
			Serial.println(there[i]);
		}
	}

//calcCos(current position, desired position, step, servo array position)
	for (int i = 0; i < numServos; i++)
	{
		angle = calcCos(here[i], there[i], 1.5, i);
		if (doneMove[i] == false)
		{
			servo[i].write(angle);
			delay(5);
		}
	}
}//   END VOID LOOP






/*
THIS FUNCTION AUTO-SETS THE SERVO RANGE
ASSUMES PUSHBUTTON ON PIN 2
*/



void setRange(int x)  //parameter passed is array position of servo
{
	int pb = 0; //used to hold push button reading
	int test; //general use variable
	int h;
	int t;
	int pulse = 1500; //first uS pulse used in range test
	Serial.print("Press button to set range of ");
	Serial.print(joints[x]);
	Serial.println();
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

	delay(20);

	Serial.println();
	pulse = 1500;
	servo[x].writeMicroseconds(pulse); //send servo to middle of range
	delay(2000); //wait for it to get there

	h = getFeedback(x);
	t = h - 20;	//set initial t away from h
	Serial.println(t);
	Serial.println(h);
	Serial.println();

	do
	{
		t = getFeedback(x);
		pulse += 10; //incriment uS pulse width
		servo[x].writeMicroseconds(pulse);
		delay(200);		//allow position to settle
		h = getFeedback(x);
		Serial.print("t,");
		Serial.print(t);
		Serial.print(",h,");
		Serial.print(h);
		Serial.println();
	}
	while(h > t);  //condition to keep testing range

	highPulse[x] = pulse - 20; //stay away from range extreme
	B[x] = h - 10; //adjust feedback away from extreme
	Serial.println();
	servo[x].writeMicroseconds(highPulse[x]);
	pulse = highPulse[x];
	delay(500);
	Serial.println();
	pulse = 1500;
	servo[x].writeMicroseconds(pulse); //send servo to middle of range
	delay(2000); //wait for it to get there

	h = getFeedback(x);
	t = h + 20; //start t out of range of h

	Serial.println(t);
	Serial.println(h);
	Serial.println();

	do
	{
		t = getFeedback(x);
		pulse -= 10; //incriment uS pulse width
		servo[x].writeMicroseconds(pulse);
		delay(200);		//allow position to settle
		h = getFeedback(x);
		Serial.print("t,");
		Serial.print(t);
		Serial.print(",h,");
		Serial.print(h);
		Serial.println();
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



	Serial.println("Feedback Range:");
	Serial.print(A[x]);
	Serial.print(" <> ");
	Serial.println(B[x]);
	Serial.println("uS Pulse Range:");
	Serial.print(lowPulse[x]);
	Serial.print(" <> ");
	Serial.println(highPulse[x]);
	Serial.print(joints[x]);



}//end setRange()


/*
THIS FUNCTION READS THE INTERNAL SERVO POTENTIOMETER
*/
int getFeedback(int a)
{
	int j;
	int mean;
	int result;
	int test;
	int reading[100];
	boolean done;

	for (j = 0; j < 100; j++)
	{
		reading[j] = analogRead(a); //get raw data from servo potentiometer
		delay(whlReading);
		
	} // sort the readings low to high in array
	done = false; // clear sorting flag
	while(done != true)  // simple swap sort, sorts numbers from lowest to highest
	{
		done = true;
		for (j = 0; j < 100; j++)
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
	for (int k = 15; k < 85; k++) //discard the 6 highest and 6 lowest readings
	{
		mean += reading[k];
	}
	result = mean / 8; //average useful readings
	return(result);
}    // END GET FEEDBACK






/*
THIS FUNCTION CREATES SMOOTH (COSINE) MOVEMENT FROM HERE TO THERE
*/

int calcCos(int h, int th, float s, int n)
{
	int r;
	int a;
	if(h < th)
	{
		x[n] += s;
		if (x[n] >= 181)
		{
			doneMove[n] = true;
		}
		r = (cos(radians(x[n])) * 100);
		a = map(r, 100, -100, h, t);
	}
	if(h > th)
	{
		x[n] -= s;
		if (x[n] <= -1)
		{
			doneMove[n] = true;
		}
		r = (cos(radians(x[n])) * 100);
		a = map(r, -100, 100, h, t);
	}
	return a;
}     //END CALC COS

