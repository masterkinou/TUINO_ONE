# GMX LR
These are the interface libraries to the GMX LoRa module. The **tuino_lora** folder contains the basic example with the best practice to implmenet a LoRa based project.<br/>
Our library is using the Regexp.cpp library from [Nick Gammon](https://github.com/nickgammon/Regexp) which we find really excellent!<br/>
The GMX LR module is a completely independent STM32L0/SX1276 based module where all the LoRa stack is running and it is interfaced to the Tuino via UART. <br/>There is a complete series of commands that give you access to the low level params of the LoRaWAN stack enabling you to control things like RX1 & RX2 window delays, Join window delays, datarate on the RX2 window and enabling/disabling ADR .<br/>
Another important feature that we have implemented is that RX is interrupt based, you simply register the callback function with the module initilazation function and you are set. 

```c
/*
 * Callback function that sets a flag to signal main loop a RX Event
 */
void loraRx(){
  data_received = true;
}

..
..

void setup() {
  // put your setup code here, to run once:
  
  // GMX-LR init pass RX callback function
  gmxLR_init(&loraRx);


```

Here the list of functions that our GMX-LR module supports and a quick description.<br/>


#MODULE INIT AND LOWAWAN JOIN PARAMETERS

Init the board - you will see all the Leds flashing in sequence when the modules boots. As described above you need to specify the RX callback function in the module init function 

```c
byte gmxLR_init(void (*function)());
```


Setting up the LoRaWAN parameters for OTAA/ABP and CLASS.
All keys and EUI ( DevEui, AppEui, AppKey, DevAddr, etc. ) must be specified like a sequence of hex bytes separate by a ':', like this: **00:00:11:99:22:11:22:99**<br/>
The Class are currently 'A' and 'C'.


```c

byte gmxLR_getDevEui(String& devEui);
byte gmxLR_setDevEui(String devEui);

byte gmxLR_getAppEui(String& appEui);
byte gmxLR_setAppEui(String appEui);

byte gmxLR_getAppKey(String& appKey);
byte gmxLR_setAppKey(String appKey);

byte gmxLR_getDevAddr(String& devAddr);
byte gmxLR_setDevAddr(String devAddr);

byte gmxLR_getNetworkID(String& netId);
byte gmxLR_setNetworkID(String netId);

byte gmxLR_getNetworkSessionKey(String& nwsk);
byte gmxLR_setNetworkSessionKey(String nwsk);

byte gmxLR_setApplicationSessionKey(String appsk);
byte gmxLR_getApplicationSessionKey(String& appsk);

byte gmxLR_getClass(String& lrclass);
byte gmxLR_setClass(String lrclass);


```



#NETWORK JOIN
The **gmxLR_Join()** call is non blocking, it will start the join process and return immediately. Your code has to poll the module with the **gmxLR_isNetworkJoined()** to verify if it has actually joined the network. The join procedure will go on forever until a join is achieved.

```c
byte gmxLR_Join(void);
byte gmxLR_isNetworkJoined(void);

byte gmxLR_setJoinMode(byte mode);
byte gmxLR_getJoinMode(String& mode);
```

It's up to your application what to do if if can't join the network for a long time, here is our join code in the example project you find in this repo. It resets the module after n tries, but it's up you what you want to do....

```c
  
  Serial.println("Joining...");
  join_wait = 0; 
  while((join_status = gmxLR_isNetworkJoined()) != LORA_NETWORK_JOINED) {

  Serial.print("Join:");
  Serial.println(join_wait);
  
    if ( join_wait == 0 )
      gmxLR_Join();
    
    join_wait++;

    if (!( join_wait % 100 )) {
      gmxLR_Reset();
      join_wait = 0;
    }

    delay(5000);

  };

```


#LOWER LEVEL LORAWAN FUNCTIONS
Here is the list of functions with which you can tune the LoRaWAN stack, actually you shouldn't because it is already on the standard values, but if you really want...<br/>


```c
byte gmxLR_getADR();
byte gmxLR_setADR(String on_off);

byte gmxLR_getDutyCycle();
byte gmxLR_setDutyCycle(String dutycyle);

byte gmxLR_getRSSI(String& rssi);
byte gmxLR_getSNR(String& snr);

byte gmxLR_getTXPower(String& power);
byte gmxLR_setTXPower(String power);

byte gmxLR_getJoinRX1Delay(String& joinrx1);
byte gmxLR_setJoinRX1Delay(String joinrx1);

byte gmxLR_getJoinRX2Delay(String& joinrx2);
byte gmxLR_setJoinRX1Delay(String joinrx2);

byte gmxLR_getRX1Delay(String& rx1);
byte gmxLR_setRX1Delay(String rx1);

byte gmxLR_getRX2Delay(String& rx2);
byte gmxLR_setRX2Delay(String rx2);

byte gmxLR_getRX2DataRate(String& rx2dr);
byte gmxLR_setRX2DataRate(String rx2dr);
```


#SENDING AND RECEIVING DATA

You have two functions to transmit data, the first without the port number, which defaults to port 1, and the second if you want to specify the port. The payload **data** must be an hexadecimal string, in the form 00a133b577... If the format is incorrect the function will return an error. This **gmxLR_TXData()** is non blocking and will return immediately, the packet might or might not be sent immediately depending on the available duty cycle slots.<br/>
While the module will generate an interrupt on every RX packet, you need to call the **gmxLR_RXData()** function to retrieve the paylaod and the port on which it has been received.

```c
byte gmxLR_TXData(String data);
byte gmxLR_TXData(String data, int port);
byte gmxLR_RXData(String& data, int *port);
```

By default confirmation(acknoledge) is disabled on all transmissions, but you can enable it with the specific functions. If the confirmation is enable you need to call the **gmxLR_getMessageConfirmation()** after a TX to verify if the message has been confirmed.


```c
byte gmxLR_getConfirmationMode(void);
byte gmxLR_setConfirmationMode(String cfm);

byte gmxLR_getMessageConfirmation(void);
```


#UTILITIES
There is the possibility of driving the fourth LED on the GMX-LR1 board. 1 turns if on, 0 turns it off.<br/>
And as always there is reset function that reboots the module.

```c
byte gmxLR_Led(byte led_state);

void gmxLR_Reset(void);
```
