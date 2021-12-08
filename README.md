#include <WiFi.h>
//#endif
#include <WiFiUdp.h>
#include <OSCBundle.h>  
#include <OSCMessage.h> // OSC Send
#include <OSCData.h>


#include <FastLED.h>

#define NUM_LEDS  300
#define LED_PIN   3

CRGB leds[NUM_LEDS];



//網路名稱與密碼
const char* ssid = "ApPerform";
const char* pass = "qqqqqqq1";

//IP Port 設定
WiFiUDP Udp;                           // A UDP instance to let us send and receive packets over UDP
const IPAddress outIp(192,168,1,100);   // remote IP of your computer
const unsigned int localPort = 8002;   // local port to listen for UDP packets at the NodeMCU (another device must send OSC messages to this port)
const unsigned int outPort = 4560;    // remote port of the target device where the NodeMCU sends OSC to




void setup() {
  Serial.begin(115200);

    
    //wifi設定程序
  WiFi.config(IPAddress(192, 168, 1, 25), IPAddress(192, 168, 1, 1), IPAddress(255, 255, 255, 0));
  //Connect to WiFi network
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);
  WiFi.begin(ssid,pass);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
    digitalWrite(LED_BUILTIN,1);
  }
  
  digitalWrite(LED_BUILTIN,0);//if led State is 0,led will be light(nodemcu's LED_BUILTIN is reversed logic) 
  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
  Serial.println("Starting UDP");
  
  Udp.begin(localPort);
  Serial.print("Local port: ");
  Serial.println(localPort);  
  


  //燈帶設定
  FastLED.addLeds<WS2812B, LED_PIN, GRB>(leds, NUM_LEDS);
  FastLED.setBrightness(255);
}

void loop() {


  //Wifi自動連接
      if(WiFi.status() == WL_CONNECTED){
    digitalWrite(LED_BUILTIN,0);;
  }
  else{
   Serial.println("failed!!!!");
   digitalWrite(LED_BUILTIN,1);
   WiFi.reconnect();
  }  

  // Waves for LED position
  uint8_t posBeat  = beatsin8(1, 0, NUM_LEDS - 1, 0, 0);
  uint8_t posBeat2 = beatsin8(2, 0, NUM_LEDS - 1, 0, 0);
  uint8_t posBeat3 = beatsin16(1, 0, NUM_LEDS - 1, 0, 127);
  uint8_t posBeat4 = beatsin16(random8(5), 0, NUM_LEDS - 1, 0, 127);


  // In the video I use beatsin8 for the positions. For longer strips,
  // the resolution isn't high enough for position and can lead to some
  // LEDs not lighting. If this is the case, use the 16 bit versions below
  // uint16_t posBeat  = beatsin16(30, 0, NUM_LEDS - 1, 0, 0);
  // uint16_t posBeat2 = beatsin16(60, 0, NUM_LEDS - 1, 0, 0);
  // uint16_t posBeat3 = beatsin16(30, 0, NUM_LEDS - 1, 0, 32767);
  // uint16_t posBeat4 = beatsin16(60, 0, NUM_LEDS - 1, 0, 32767);

  // Wave for LED color
  uint8_t colBeat  = beatsin8(5, 213, 0, 0, 0);
  uint8_t colBeat2  = beatsin8(5, 250, 0, 0, 0);  
  leds[(posBeat + posBeat2) / 2]  = CHSV(colBeat, 255, 255);
  leds[(posBeat3 + posBeat4) / 2]  = CHSV(colBeat2, 255, 255);

  fadeToBlackBy(leds, NUM_LEDS, random8(50));

  FastLED.show();


  int num = (posBeat + posBeat2) / 2;
  
  Serial.println(num);
  
       // osc send 
    OSCMessage msg("num");
    msg.add(num);
    Udp.beginPacket(outIp, outPort);
    msg.send(Udp);
    Udp.endPacket();
    msg.empty();     // osc send 

}
