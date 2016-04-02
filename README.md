# timer_led1
/******************************************************************************
 *
 * www.payitforward.edu.vn
 *
 *****************************************************************************/

/******************************************************************************
 *
 * PIF C14 COURSE
 *
 *****************************************************************************/

/******************************************************************************
 *
 *	Module		: timer_led.c
 *	Description	: timer homework c14
 *  Tool		: CCS 6.1.2
 *	Chip		: MSP430G2553
 * 	History		: 4/2/2016
 *  Version     : beta
 *
 *	Author		: Nguyen Duc Huu, chicken at PIF HCMUT
 *	Notes		: Adjust brightness of LED using timer
 *
 *****************************************************************************/

 /****************************************************************************
 * PINS MAPPING
 *
 * ***************************************************************************
 * LED module
 * ***************************************************************************
 * P1.0	====> LED 0
 * P1.1	====> LED 1
  * ***************************************************************************
 * BUTTON MODULE
 * ***************************************************************************
 * Note: Press button to change number on 7SEG MODULE
 * BUTTON 1		====> P2.0 >> Increase 1 level of LED 0 brightness (32 levels)
 * BUTTON 2		====> P2.1 >> Increase 1 level of LED 1 brightness (32 levels)
 *****************************************************************************/
#include <msp430g2553.h>	/*import library of msp430g2253*/

#define deltaT 100

unsigned char i=1,j=1;

void config_IO(void)
{
	P1DIR |= (BIT0+BIT1);	//P1.0, P1.1 output
	P1OUT |= (BIT0+BIT1);
	P2DIR &= ~(BIT0+BIT1);	//P2.0,P2.1 input

	P2REN |= (BIT0+BIT1);	//enable pullup,pulldown res
	P2OUT |= (BIT0+BIT1);	//pullup res
	
	P2IE |= (BIT0+BIT1);		//enable interupt P2.0,P2.1
	P2IES |= (BIT0+BIT1);	//falling edge Int
	P2IFG &= ~(BIT0 + BIT1);//clear Int flag
}

void config_WDT(void)
{
	WDTCTL = WDTPW + WDTHOLD;  //stop watchdog
}

void config_Timera0(void)
{
	TA0CTL=TASSEL_2+ ID_0+ MC_1 + TAIFG;
	TA0CCR0=3200;
	TA0CCR1=100;
	TA0CCR2=100;
	TA0CCTL0=CCIE + CCIFG;
	TA0CCTL1=CCIE + CCIFG;
	TA0CCTL2=CCIE + CCIFG;
}
void main()
{
	config_WDT();
	config_IO();
	config_Timera0();
	_BIS_SR(LPM0_bits+GIE); //ngat toan cuc
}

#pragma vector=TIMER0_A1_VECTOR
__interrupt void TAIV_Int(void)
{
	switch(TA0IV)
	{
		case 0x02:
		{
		P1OUT |= BIT0;
		break;
		}
		case 0x04:
		{
		P1OUT |= BIT1;
		break;
		}
	}
}

#pragma vector=TIMER0_A0_VECTOR
__interrupt void CCR0_Int(void)
{
	P1OUT &= ~(BIT0 + BIT1);
}

#pragma vector = PORT2_VECTOR
__interrupt void P2_Int (void)
{
	switch (P2IFG & 0x03)
	{
		case BIT0:
		{
			i++;
			if (i==32) //tang duty cycle led0
				i = 1;
			TACCR1 = deltaT * i;
			break;
		}
		case BIT1:
		{
			j++;
			if (j==32)	////tang duty cycle led1
				j = 1;
			TACCR2 = deltaT * j;
			break;
		}
	}
	P2IFG &= ~(BIT0 + BIT1);
}
