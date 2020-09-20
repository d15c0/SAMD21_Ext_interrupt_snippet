# SAMD21_Ext_interrupt_snippet

// SETUP external interrupt
#define interruptPin 8
volatile bool SLEEP_FLAG;

void EIC_ISR(void) {
  SLEEP_FLAG ^= true;  // toggle SLEEP_FLAG by XORing it against true
  //Serial.print("EIC_ISR SLEEP_FLAG = ");
  //Serial.println(SLEEP_FLAG);
}


void setup()
{

attachInterrupt(digitalPinToInterrupt(interruptPin), EIC_ISR, CHANGE);  // Attach interrupt to pin 6 with an ISR and when the pin state CHANGEs

  SYSCTRL->XOSC32K.reg |=  (SYSCTRL_XOSC32K_RUNSTDBY | SYSCTRL_XOSC32K_ONDEMAND); // set external 32k oscillator to run when idle or sleep mode is chosen
  REG_GCLK_CLKCTRL  |= GCLK_CLKCTRL_ID(GCM_EIC) |  // generic clock multiplexer id for the external interrupt controller
                       GCLK_CLKCTRL_GEN_GCLK1 |  // generic clock 1 which is xosc32k
                       GCLK_CLKCTRL_CLKEN;       // enable it
  while (GCLK->STATUS.bit.SYNCBUSY);              // write protected, wait for sync

  EIC->WAKEUP.reg |= EIC_WAKEUP_WAKEUPEN4;        // Set External Interrupt Controller to use channel 4 (pin 6)

  
  PM->SLEEP.reg |= PM_SLEEP_IDLE_CPU;  // Enable Idle0 mode - sleep CPU clock only
  //PM->SLEEP.reg |= PM_SLEEP_IDLE_AHB; // Idle1 - sleep CPU and AHB clocks
  //PM->SLEEP.reg |= PM_SLEEP_IDLE_APB; // Idle2 - sleep CPU, AHB, and APB clocks

  // It is either Idle mode or Standby mode, not both. 
  SCB->SCR |= SCB_SCR_SLEEPDEEP_Msk;   // Enable Standby or "deep sleep" mode

  SLEEP_FLAG = false; // begin awake
  
  }
