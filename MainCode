#include <stdio.h>
#include "L138_LCDK_aic3106_init.h"
#include "L138_LCDK_switch_led.h"
#include <math.h>
#include "evmomapl138_gpio.h"
#include <stdint.h>
#include <ti/dsplib/dsplib.h>

// Global Definitions and Variables

#define PI 3.14159265358979323
#define N 1024

int data;
int counter;
int count_switch;
int switch5;
int switch6;
int switch7;
int switch8;
int LED_succesion_flag = 0;
int m=0;
int M = 26;
int n=0;
int j=0;
int k=0;
float indexer = 0.0;
int K = (1024/2)+1;
int repeat=0;
int output;

float alpha = .54;
float beta = .46;
int flag = 0;
float y[13];

int blinker_counter=0;
int blinker_numberfive=0;
int twice=0;

//Data Arrays
float microphone[1024];
int hundred = 0;
float hanning[1024];
float product[1024];
float magnitude[1024];
float fmel[28];
float lambda[28];
float freqResp[26][513];
float multAccum[26];
float xi[26];
float mfcc[13];
float phi[28];
float h[13];
float temp;
float sx[5];
float ox[4];
float output1[4];
float summationOfx = 0.0;

//Neural Net Values
float x1step1xoffset[13] = {189.9375,13.06396,-8.696548,14.23487,-1.749886,8.781987,-6.512872,11.31746,0.05867729,8.824528,0.004018578,6.907649,-0.1373394};
float x1step1gain[13] = {0.0304246366537768,0.0802794044392102,0.165614764490008,0.205358628040078,0.250459499258828,0.156567134069141,0.192835245037674,0.269633716078393,0.609282657072894,0.243773358058452,0.439514515413199,0.273299798537054,0.389937628306539};
float x1step1ymin = -1.0;

// Layer 1
float b1[5] = {-1.6829024581844724828,0.9340810088600818295,0.56730273089740546055,-1.0534476335955005322,-1.187588451404443024};
float weight1[5][13] = {{0.57417038831455524672 ,0.25125237917764309259 ,0.25527904414820407064 ,-0.38944481365597011235 ,0.059850938748013893242 ,-0.73535534026156268439 ,0.75072335340338658405 ,-0.53325903230440996339 ,0.1878961857305334604 ,0.50161102249467004466 ,0.034674016994926747104 ,-0.019646113831179163461 ,-0.4280490924385793039},{0.26214937783820685402 ,0.37870119778970806612 ,-0.38882813351675782343 ,0.86280621364608467427 ,0.53050945615217215412 ,-0.41056411681876436859 ,-1.4410196493678766316 ,0.32166431542183460124 ,0.75105201523983589063 ,0.16987190452729417078 ,0.88713312649816022937 ,-0.20290642624978919017 ,-0.036774720983833011911},{-1.3091557623997716053, 0.19288471208025506631 ,-0.41307722443051347883 ,-1.026917411284244297 ,0.43484269705840722375 ,-0.64068413928946488234, 0.32095965789940428259, -0.62950326547043633507, -0.11083362444386311851, -0.39687514913846977382, 0.20653738889701733772, -0.98095573968889404615, -0.96463138382836355333},{-0.027199419143750468847, -1.1834514349271081368, -0.89494758524429440794, 0.59686954283713322678, 1.5419892494877294187, 0.75984242132690749116, -0.65129742854917782591, 0.027331167363196184894, 0.42087521384846288131, -0.76839000277045321763, -0.084164177385889105132, 0.21208383248783005404, -0.28677909179845767484},{-0.13189337129164227513, 0.76096386719304953683, 1.402851661945180517, 0.17282684527313404987, -0.12392785499212929934, -1.4631209338577269019, -1.9727565176850212492, -0.019090880858788513569, 0.15114516090109778035, -0.16052506982288045068, 0.2361306758809255113, -0.48123749064511417428, -0.38380130817891278117}};

// Layer 2
float b2[4] = {0.59063349533615983056,-1.2801667642768714828,0.4228528334440181391,0.048951154770967925112};
float weight2[4][5] = {{0.060211327344333487321 ,1.3231053728994146557 ,1.6765960344704182727, -1.8746711858079188051 ,5.8023860705200913657},{0.62474876094257236758 ,-4.8844108415113458577 ,3.8836946180880196344 ,-2.0996531211858933119 ,-2.1185774586097303462},{0.45759531514542478581 ,2.5045646479824443809 ,1.6208960977140163262 ,5.0866844183488382214 ,-1.0811801834664398481},{-0.058581817545676213854, -0.18623172500112583627, -6.3042474340154122103, -2.0806334214673070093, -1.5119399559633039942}};

/* Align the tables that we have to use */

// The DATA_ALIGN pragma aligns the symbol in C, or the next symbol declared in C++, to an alignment boundary.
// The alignment boundary is the maximum of the symbol's default alignment value or the value of the constant in bytes.
// The constant must be a power of 2. The maximum alignment is 32768.
// The DATA_ALIGN pragma cannot be used to reduce an object's natural alignment.

//The following code will locate mybyte at an even address.
//#pragma DATA_ALIGN(mybyte, 2)
//char mybyte;

//The following code will locate mybuffer at an address that is evenly divisible by 1024.
//#pragma DATA_ALIGN(mybuffer, 1024)
//char mybuffer[256];
#pragma DATA_ALIGN(x_in,8);
int16_t x_in[2048];

#pragma DATA_ALIGN(x_sp,8);
float   x_sp [2048];
#pragma DATA_ALIGN(y_sp,8);
float   y_sp [2048];
#pragma DATA_ALIGN(w_sp,8);
float   w_sp [2048];

// brev routine called by FFT routine
unsigned char brev[64] = {
		0x0, 0x20, 0x10, 0x30, 0x8, 0x28, 0x18, 0x38,
		0x4, 0x24, 0x14, 0x34, 0xc, 0x2c, 0x1c, 0x3c,
		0x2, 0x22, 0x12, 0x32, 0xa, 0x2a, 0x1a, 0x3a,
		0x6, 0x26, 0x16, 0x36, 0xe, 0x2e, 0x1e, 0x3e,
		0x1, 0x21, 0x11, 0x31, 0x9, 0x29, 0x19, 0x39,
		0x5, 0x25, 0x15, 0x35, 0xd, 0x2d, 0x1d, 0x3d,
		0x3, 0x23, 0x13, 0x33, 0xb, 0x2b, 0x1b, 0x3b,
		0x7, 0x27, 0x17, 0x37, 0xf, 0x2f, 0x1f, 0x3f
};

// The seperateRealImg function separates the real and imaginary data
// of the FFT output. This is needed so that the data can be plotted
// using the CCS graph feature
float y_real_sp [1024];
float y_imag_sp [1024];

separateRealImg () {
	int i, j;

	for (i = 0, j = 0; j < N; i+=2, j++) {
		y_real_sp[j] = y_sp[i];
		y_imag_sp[j] = y_sp[i + 1];
	}
}

// Function for generating sequence of twiddle factors
void gen_twiddle_fft_sp (float *w, int n)
{
	int i, j, k;
	double x_t, y_t, theta1, theta2, theta3;

	for (j = 1, k = 0; j <= n >> 2; j = j << 2)
	{
		for (i = 0; i < n >> 2; i += j)
		{
			theta1 = 2 * PI * i / n;
			x_t = cos (theta1);
			y_t = sin (theta1);
			w[k] = (float) x_t;
			w[k + 1] = (float) y_t;

			theta2 = 4 * PI * i / n;
			x_t = cos (theta2);
			y_t = sin (theta2);
			w[k + 2] = (float) x_t;
			w[k + 3] = (float) y_t;

			theta3 = 6 * PI * i / n;
			x_t = cos (theta3);
			y_t = sin (theta3);
			w[k + 4] = (float) x_t;
			w[k + 5] = (float) y_t;
			k += 6;
		}
	}
}

//Blink Operations
void led_blinker()
{
	if (blinker_counter < 4000)
	{
		LCDK_LED_on(4);
		LCDK_LED_on(5);
		LCDK_LED_on(6);
		LCDK_LED_on(7);
		blinker_counter++;
	}
	else if (blinker_counter < 8000)
	{
		LCDK_LED_off(4);
		LCDK_LED_off(5);
		LCDK_LED_off(6);
		LCDK_LED_off(7);
		blinker_counter++;
	}
	else if (blinker_counter == 8000){
		blinker_counter=0;
	}
	return;
}

//Record and Switch Operations
void led_numberfive()
{
	if (blinker_numberfive < 8000)
	{
		LCDK_LED_on(4);
		LCDK_LED_off(5);
		LCDK_LED_off(6);
		LCDK_LED_off(7);
		blinker_numberfive++;
	}
	else if (blinker_numberfive < 16000)
	{
		LCDK_LED_off(4);
		LCDK_LED_on(5);
		LCDK_LED_off(6);
		LCDK_LED_off(7);
		blinker_numberfive++;
	}
	else if (blinker_numberfive < 24000){
		LCDK_LED_off(4);
		LCDK_LED_off(5);
		LCDK_LED_on(6);
		LCDK_LED_off(7);
		blinker_numberfive++;
	}
	else if (blinker_numberfive < 32000){
		LCDK_LED_off(4);
		LCDK_LED_off(5);
		LCDK_LED_off(6);
		LCDK_LED_on(7);
		blinker_numberfive++;
	}
	if (blinker_numberfive == 32000)
	{
		LED_succesion_flag = 1;
	}
	return;
}

//Check Switch State
void switch_state()
{
	switch5 = LCDK_SWITCH_state(5);
	switch6 = LCDK_SWITCH_state(6);
	switch7 = LCDK_SWITCH_state(7);
	switch8 = LCDK_SWITCH_state(8);
	count_switch = switch5+switch6+switch7+switch8;
}

//FFT Magnitude
magnitudeFFT()
{
	int r;
	for(r=0; r<N; r++)
	{
		magnitude[r] = ((y_real_sp[r]*y_real_sp[r])+(y_imag_sp[r]*y_imag_sp[r]));
	}

}

//Mel Frequencies
fMelCalc()
{
	int r;
	for(r=0; r<28; r++)
	{
		fmel[r] = 2595*log10(1+(250.0+r*287.0)/700.0);

	}

}

//Phi Values
phiCalc()
{
	int r;
	for(r=0; r<28; r++)
	{
		phi[r] = 250.0+r*287.0;

	}
}

interrupt void interrupt4(void)  // interrupt service routine
{
	switch_state();
	int left_sample = input_sample();
	if(count_switch>1) //more than one switch
	{
		led_blinker();
		output_sample(0);
	}
	else if(switch5 && !switch6 && !switch7 && !switch8) //switch 5 record
	{
		led_numberfive();
		if(LED_succesion_flag == 1 && m < 1024 && hundred > 99){
			microphone[m]= left_sample;
			m++;
		}
		hundred++;
		if(m==1024)
		{
			flag = 1;
		}
		output_sample(0);
	}
	else if(!switch5 && switch6 && !switch7 && !switch8){ //switch 6 output

		output=(microphone[n]);
		n++;
		n = n%8000;
		output_sample(output);
	}
	else{
		output_sample(0);}

	return;
}
int flag1=0;
int flag2=0;
int main(void)
{
	int i; //record voice
	for (i=0;i<1024;i++){
		microphone[i]=0;
	}
	for (i=0;i<1024;i++) //hanning calculation
	{
		hanning[i]= alpha - beta*cos(2*PI*i/(N-1));
	}
	counter = 0;

	L138_initialise_intr(FS_16000_HZ,ADC_GAIN_12DB,DAC_ATTEN_0DB,LCDK_MIC_INPUT);
	LCDK_GPIO_init();
	LCDK_LED_init();
	LCDK_SWITCH_init();

	while(flag1==0){if(flag==1){flag1=1;}}
	flag1=1;

	if (m >= 1024) //hanning product
	{
		for (i=0;i<1024;i++)
		{
			product[i] = hanning[i] * microphone[i];
		}
	}

	// Copy input data to the array used by DSPLib functions
	int o;
	for (o=0; o<N; o++)
	{
		x_sp[2*o]   = product[o];
		x_sp[2*o+1] = 0.0;
	}

	// Call twiddle function to generate twiddle factors needed for FFT and IFFT functions
	gen_twiddle_fft_sp(w_sp,N);

	// Call FFT routine
	DSPF_sp_fftSPxSP(N,x_sp,w_sp,y_sp,brev,4,0,N);

	// Call routine to separate the real and imaginary parts of data
	// Results saved to floats y_real_sp and y_imag_sp
	separateRealImg (); //seprate real and imag fft values
	magnitudeFFT(); //calculate fft magnitude
	fMelCalc(); //mel frequency function call
	phiCalc(); //phi function call
	for(i=0; i<28; i++) //calculate lambdas
	{
		lambda[i] = (int)(513.0*phi[i]/8000.0);
	}

	//filter
	for (n = 0; n < 26; n++){
		for (k = 0; k < 513; k++){
			if (k+1 < (int16_t)lambda[n])
				freqResp[n][k] = 0;
			else if (k+1 <= (int16_t)lambda[n+1]) {
				freqResp[n][k] = ((k+1) - lambda[n])/(lambda[n+1] - lambda[n]);
				if (freqResp[n][k] < 0.0)
					freqResp[n][k] = freqResp[n][k] * -1.0;
			}
			else if (k+1 <= (int16_t)lambda[n+2]){
				freqResp[n][k] = (lambda[n+2] - (k+1))/(lambda[n+2] - lambda[n+1]);
				if (freqResp[n][k] < 0.0)
					freqResp[n][k] = freqResp[n][k] * -1.0;
			}
			else
				freqResp[n][k] = 0;
		}
	}

	//multiply and accumulate
	for(i=0; i<26; i++)
	{
		multAccum[i]=0;
	}

	for(i=0; i<26; i++)
	{
		for(k=0; k<513;k++)
		{
			multAccum[i]+=magnitude[k]*freqResp[i][k];
		}
	}

	//calculate Xi
	for(i=0; i<26; i++)
	{
		xi[i]=log10(multAccum[i]);
	}
	//mfcc coefficients
	for (i=0;i<13;i++)
	{
		mfcc[i]=0;
	}
	for(i=0;i<13; i++)
	{
		for(k=0;k<26;k++)
		{
			mfcc[i] += xi[k]*cos((float)(i+1)*((float)(k+1)-0.5)*(PI/26.0));
		}
	}

	//preprocessing
	for (i = 0; i < 13; i++) {
		y[i] = mfcc[i] - x1step1xoffset[i];
		y[i] *= x1step1gain[i];
		y[i] += x1step1ymin;
	};

	// Compute weights for hidden layer neurons
	for (i = 0; i < 5; i++) {
		for(k = 0; k < 13; k++)
			temp += weight1[i][k] * y[k];
		temp += b1[i];
		h[i] = (2.0 / (1.0 + exp(-2.0 * temp))) - 1.0;
		temp = 0.0;
	}

	// Compute weights for output layer neurons
	for (i = 0; i < 4; i++) {
		for(k = 0; k < 5; k++)
			temp += weight2[i][k] * h[k];
		ox[i] = temp + b2[i];

		temp = 0.0;
	}

	//vowel recognition
	for(i = 0; i < 4; i++)
		temp += exp(ox[i]);

	for(i = 0; i < 4; i++)
		output1[i] = exp(ox[i]) / temp;

	while(1);

}
