# 16-intelligent-agriculture
第16小组，我们是刘水晶石，宋洋。

项目名称：16组-智慧农业


问题定义：农业立国之本也，民以食为天，必须保证粮食供应，才能保证人类的荣光，现代发展催生了对智慧农业的需求。是农业中的智慧经济，或智慧经济形态在农业中的具体表现。智慧农业是智慧经济重要的组成部分；对于发展中国家而言，智慧农业是智慧经济主要的组成部分，是发展中国家消除贫困、实现后发优势、经济发展后来居上、实现赶超战略的主要途径。
    具体到技术层面，智慧农业就是将物联网技术运用到传统农业中去，运用传感器和软件通过移动平台或者电脑平台对农业生产进行控制，使传统农业更具有“智慧”。除了精准感知、控制与决策管理外，从广泛意义上讲，智慧农业还包括农业电子商务、食品溯源防伪、农业休闲旅游、农业信息服务等方面的内容。
所谓“智慧农业”就是充分应用现代信息技术成果，集成应用计算机与网络技术、物联网技术、音视频技术、3S技术、无线通信技术及专家智慧与知识，实现农业可视化远程诊断、远程控制、灾变预警等智能管理。
智慧农业是农业生产的高级阶段，是集新兴的互联网、移动互联网、云计算和物联网技术为一体，依托部署在农业生产现场的各种传感节点（环境温湿度、土壤水分、二氧化碳、图像等）和无线通信网络实现农业生产环境的智能感知、智能预警、智能决策、智能分析、专家在线指导，为农业生产提供精准化种植、可视化管理、智能化决策。
“智慧农业”是云计算、传感网、3S等多种信息技术在农业中综合、全面的应用，实现更完备的信息化基础支撑、更透彻的农业信息感知、更集中的数据资源、更广泛的互联互通、更深入的智能控制、更贴心的公众服务。“智慧农业”与现代生物技术、种植技术等高新技术融合于一体，对建设世界水平农业具有重要意义。
    衍生应用：智能农业大棚、农机定位、仓储管理、食品溯源等。
    
    总体情况：大部分是以通过物联网连接传感器实现监测。升级生产领域，由人工走向智能。智慧农业实现农业精细化、高效化、绿色化发展，促进智慧农业大发展的思路。
    
    概念设计：我们要做的就是根据传感器检测土壤温湿度，ph值等，同时对这些值进行修正，这就是我们的产品。
    产品介绍：通过基于lora模块的网关、节点，再搭配上温湿度传感器、液晶显示屏、蜂鸣器，实现温湿度的测量并反馈。
    
    性能指标：用普通充电宝等可以当作电源，功耗较低，对环境的影响可以忽略，工作频段就是lora模块的频段，可以实时显示所测得得环境温湿度
    
源代码如下：
    
网关：
#include <LoRaNow.h>
#include <WiFi.h>
#include <WebServer.h>
#include <StreamString.h>
#include <PubSubClient.h>

//vspi
//#define MISO 19
//#define MOSI 23
//#define SCK 18
//#define SS 5

//hspi,only for ESP-32-Lora-Shield-Ra02
#define SCK 14
#define MISO 12
#define MOSI 13
#define SS 15

#define DIO0 4

const char *ssid = "codes2things";
const char *password = "swjtumaker";
const char *mqtt_server = "broker.hivemq.com";

WebServer server(80);
WiFiClient espClient;
PubSubClient mqttclient(espClient);

const char *script = "<script>function loop() {var resp = GET_NOW('loranow'); var area = document.getElementById('area').value; document.getElementById('area').value = area + resp; setTimeout('loop()', 1000);} function GET_NOW(get) { var xmlhttp; if (window.XMLHttpRequest) xmlhttp = new XMLHttpRequest(); else xmlhttp = new ActiveXObject('Microsoft.XMLHTTP'); xmlhttp.open('GET', get, false); xmlhttp.send(); return xmlhttp.responseText; }</script>";

unsigned long lastMsg = 0;
#define MSG_BUFFER_SIZE  (60)
char msg[MSG_BUFFER_SIZE];

int value = 0;

void handleRoot()
{
  String str = "";
  str += "<html>";
  str += "<head>";
  str += "<title>ESP32 - LoRaNow</title>";
  str += "<meta name='viewport' content='width=device-width, initial-scale=1'>";
  str += script;
  str += "</head>";
  str += "<body onload='loop()'>";
  str += "<center>";
  str += "<textarea id='area' style='width:800px; height:400px;'></textarea>";
  str += "</center>";
  str += "</body>";
  str += "</html>";
  server.send(200, "text/html", str);
}

static StreamString string;

void handleLoRaNow()
{
  server.send(200, "text/plain", string);
  while (string.available()) // clear
  {
    string.read();
  }
}

void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();

  // Switch on the LED if an 1 was received as first character
  if ((char)payload[0] == '1') {
    //digitalWrite(BUILTIN_LED, LOW);   // Turn the LED on (Note that LOW is the voltage level
    // but actually the LED is on; this is because
    // it is active low on the ESP-01)
  } else {
    //digitalWrite(BUILTIN_LED, HIGH);  // Turn the LED off by making the voltage HIGH
  }

}

void setup(void)
{

  Serial.begin(115200);

  WiFi.mode(WIFI_STA);
  //if (ssid != "")
    WiFi.begin(ssid, password);
  WiFi.begin();
  Serial.println("");

  // Wait for connection
  while (WiFi.status() != WL_CONNECTED)
  {
    delay(500);
    Serial.print(".");
  }

  Serial.println("");
  Serial.print("Connected to ");
  Serial.println(ssid);
  Serial.print("IP address: ");
  Serial.println(WiFi.localIP());

  server.on("/", handleRoot);
  server.on("/loranow", handleLoRaNow);
  server.begin();
  Serial.println("HTTP server started");

  LoRaNow.setFrequencyCN(); // Select the frequency 486.5 MHz - Used in China
  // LoRaNow.setFrequencyEU(); // Select the frequency 868.3 MHz - Used in Europe
  // LoRaNow.setFrequencyUS(); // Select the frequency 904.1 MHz - Used in USA, Canada and South America
  // LoRaNow.setFrequencyAU(); // Select the frequency 917.0 MHz - Used in Australia, Brazil and Chile

  // LoRaNow.setFrequency(frequency);
  // LoRaNow.setSpreadingFactor(sf);
  // LoRaNow.setPins(ss, dio0);

  LoRaNow.setPinsSPI(SCK, MISO, MOSI, SS, DIO0); // Only works with ESP32

  if (!LoRaNow.begin())
  {
    Serial.println("LoRa init failed. Check your connections.");
    while (true)
      ;
  }

  LoRaNow.onMessage(onMessage);
  LoRaNow.gateway();
  //mqtt
  mqttclient.setServer(mqtt_server, 1883);
  mqttclient.setCallback(callback);

}

void loop(void)
{
  LoRaNow.loop();
  server.handleClient();
  mqttloop();
}

void onMessage(uint8_t *buffer, size_t size)
{
  unsigned long id = LoRaNow.id();
  byte count = LoRaNow.count();

  Serial.print("Node Id: ");
  Serial.println(id, HEX);
  Serial.print("Count: ");
  Serial.println(count);
  Serial.print("Message: ");
  Serial.write(buffer, size);
  Serial.println();
  Serial.println();
  
   //此处通过mqtt发送信息。
   snprintf (msg, MSG_BUFFER_SIZE, "#%d#%s", count,buffer);
   Serial.print("Publish message: ");
   Serial.println(msg);
   mqttclient.publish("C2TLOutTopic", msg);

  if (string.available() > 512)
  {
    while (string.available())
    {
      string.read();
    }
  }

  string.print("Node Id: ");
  string.println(id, HEX);
  string.print("Count: ");
  string.println(count);
  string.print("Message: ");
  string.write(buffer, size);
  string.println();
  string.println();

  // Send data to the node
  LoRaNow.clear();
  LoRaNow.print("LoRaNow Gateway Message ");
  LoRaNow.print(millis());
  LoRaNow.send();
}

void reconnect() {
  // Loop until we're reconnected
  while (!mqttclient.connected()) {
    Serial.print("Attempting MQTT connection...");
    // Create a random mqttclient ID
    String clientId = "ESP8266Client-";
    clientId += String(random(0xffff), HEX);
    // Attempt to connect
    if (mqttclient.connect(clientId.c_str())) {
      Serial.println("connected");
      // Once connected, publish an announcement...
      mqttclient.publish("C2TLOutTopic", "hello world");
      // ... and resubscribe
      mqttclient.subscribe("C2TLInTopic");
    } else {
      Serial.print("failed, rc=");
      Serial.print(mqttclient.state());
      Serial.println(" try again in 5 seconds");
      // Wait 5 seconds before retrying
      delay(5000);
    }
  }
}

void mqttloop() {

  if (!mqttclient.connected()) {
    reconnect();
  }
  mqttclient.loop();

//  unsigned long now = millis();
//  if (now - lastMsg > 2000) {
//    lastMsg = now;
//    ++value;
//    snprintf (msg, MSG_BUFFER_SIZE, "hello world #%ld", value);
//    Serial.print("Publish message: ");
//    Serial.println(msg);
//    mqttclient.publish("C2TLOutTopic", msg);
//  }
}


节点：
#include <HardwareSerial.h>
HardwareSerial SerialBD(1);
#include <NMEAGPS.h>
NMEAGPS gps;
gps_fix fix;

//for 18b20
#include <OneWire.h>
#include <DallasTemperature.h>

// GPIO where the DS18B20 is connected to
const int oneWireBus = 25;     

// Setup a oneWire instance to communicate with any OneWire devices
OneWire oneWire(oneWireBus);

// Pass our oneWire reference to Dallas Temperature sensor 
DallasTemperature sensors(&oneWire);
float tmp;
// for Lora
#include <LoRaNow.h>

//vspi for lora radio module
#define MISO 19
#define MOSI 23
#define SCK 18
#define SS 5

//hspi for lora radio module
//#define MISO 12
//#define MOSI 13
//#define SCK 14
//#define SS 15

#define DIO0 4

#define MSG_DELAY 2000 //should sleep,use delay for tem use

void setup() {
  Serial.begin(115200);
  Serial.println("LoRa Node With Beidou and Temperature");
  SerialBD.begin(9600,SERIAL_8N1,16,17);//16 rx,17 tx

  // Start the DS18B20 sensor
  sensors.begin();
  
   LoRaNow.setFrequencyCN(); // Select the frequency 486.5 MHz - Used in China
  // LoRaNow.setFrequencyEU(); // Select the frequency 868.3 MHz - Used in Europe
  // LoRaNow.setFrequencyUS(); // Select the frequency 904.1 MHz - Used in USA, Canada and South America
  // LoRaNow.setFrequencyAU(); // Select the frequency 917.0 MHz - Used in Australia, Brazil and Chile

  // LoRaNow.setFrequency(frequency);
  // LoRaNow.setSpreadingFactor(sf);
  // LoRaNow.setPins(ss, dio0);

   LoRaNow.setPinsSPI(SCK, MISO, MOSI, SS, DIO0); // Only works with ESP32

  if (!LoRaNow.begin()) {
    Serial.println("LoRa init failed. Check your connections.");
    while (true);
  }

  LoRaNow.onMessage(onMessage);
  LoRaNow.onSleep(onSleep);
  LoRaNow.showStatus(Serial);
}

void loop() {
  sensors.requestTemperatures(); 
  tmp = sensors.getTempCByIndex(0);
  readBD();
  LoRaNow.loop();
}

void onMessage(uint8_t *buffer, size_t size)
{
  Serial.print("Receive Message: ");
  Serial.write(buffer, size);
  Serial.println();
}

void onSleep()
{
  Serial.println("Sleep");
  //readBD(); 
  
  Serial.println("Lora Send Message");
  delay(4000); // "kind of a sleep"

  //LoRaNow.print(message);
  //LoRaNow.print("message");

    LoRaNow.print(fix.status);  
    LoRaNow.print(",");
    LoRaNow.print(fix.longitude());
    LoRaNow.print(",");
    LoRaNow.print(fix.latitude());    
    LoRaNow.print(",");
    LoRaNow.print(fix.altitude());
    LoRaNow.print(",");
    LoRaNow.print(fix.speed_kph());
    LoRaNow.print(",");
    LoRaNow.print(fix.satellites);
    LoRaNow.print(","); 
    LoRaNow.print(fix.dateTime.year);
    LoRaNow.print("-"); 
    LoRaNow.print(fix.dateTime.month);
    LoRaNow.print("-"); 
    LoRaNow.print(fix.dateTime.date);
    LoRaNow.print("~"); 
    LoRaNow.print(fix.dateTime.hours);
    LoRaNow.print(":"); 
    LoRaNow.print(fix.dateTime.minutes);
    LoRaNow.print(":"); 
    LoRaNow.print(fix.dateTime.seconds);
    LoRaNow.print(",");
    LoRaNow.print(tmp);
    LoRaNow.send();
}



void readBD(){
  if (gps.available( SerialBD )) {
    fix = gps.read();
    Serial.println("--GPS info--");
//    //0 STATUS_NONE, 1 STATUS_EST, 2 STATUS_TIME_ONLY, 3 STATUS_STD or 4 STATUS_DGPS  
    Serial.print("status:");
    Serial.println(fix.status);  
    Serial.print("longitude:");
    Serial.println(fix.longitude());
    Serial.print("latitude:");
    Serial.println(fix.latitude());    
    Serial.print("altitude:");
    Serial.println(fix.altitude());
    Serial.print("speed:");
    Serial.println(fix.speed_kph());
    Serial.print("satellites count:");
    Serial.println(fix.satellites);
    Serial.print("UTC "); 
    Serial.print(fix.dateTime.year);
    Serial.print("-"); 
    Serial.print(fix.dateTime.month);
    Serial.print("-"); 
    Serial.print(fix.dateTime.date);
    Serial.print(" "); 
    Serial.print(fix.dateTime.hours);
    Serial.print(":"); 
    Serial.print(fix.dateTime.minutes);
    Serial.print(":"); 
    Serial.println(fix.dateTime.seconds);
    
  }
 }
 
 
 代码如何运行：模块运行在esp32上，所以在下载上开发板之前必须配置esp32的开发环境先，这个可以直接在arduino管理库中下载，也可以直接在github上找到esp32,下载后arduino离线安装，搭建好之后直接使用arduino编译器将程序内容下载到开发板上，可以实现所需要的功能。节点和网关分别通上电后便可使用。在网关处的液晶显示屏和蜂鸣器对温湿度进行实时反馈。
 
 
 
 
 
    
    
    
