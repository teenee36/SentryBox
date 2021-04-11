1.-----------------------------------------------------------ESP8622-----------------------------------------------------------
#include <ESP8266WiFi.h>
#include <PubSubClient.h>
#include <ESP8266HTTPClient.h>
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
LiquidCrystal_I2C  lcd(0x27,2,1,0,4,5,6,7);
#include <DHT.h>
#define DHTPIN D3
#define DHTTYPE DHT22
DHT dht(DHTPIN, DHTTYPE,20);   


unsigned long previousMillis = 0;
const long interval = 1000;  
int buttonState[5];
int lastButtonState[5];

float h;
float t;
int light;
int co;
String cov[2]={"NO ","YES"};

const char* ssid = "eieigum"; //ssid wifi
const char* password = "eieilovelove"; //password wifi
#define LINE_TOKEN "linetoken"
const char* mqtt_server = "192.168.0.88";
const String server="192.168.0.88";
const String folder="1484";
int cnt;

HTTPClient http;
WiFiClient espClient;
PubSubClient client(espClient);

void Line_Notify(String token,String message) {
  BearSSL::WiFiClientSecure client;
  client.setInsecure();
  if (!client.connect("notify-api.line.me", 443)) {
    Serial.println("connection failed");
    return;   
  }
  String req = "";
  req += "POST /api/notify HTTP/1.1\r\n";
  req += "Host: notify-api.line.me\r\n";
  req += "Authorization: Bearer " + String(token) + "\r\n";
  req += "Cache-Control: no-cache\r\n";
  req += "User-Agent: ESP8266\r\n";
  req += "Content-Type: application/x-www-form-urlencoded\r\n";
  req += "Content-Length: " + String(String("message=" + message).length()) + "\r\n";
  req += "\r\n";
  req += "message=" + message;
  // Serial.println(req);
  client.print(req);
  delay(20);
  while(client.connected()) {
    String line = client.readStringUntil('\n');
    if (line == "\r") {
      break;
    }
    //Serial.println(line);
  }
}


void connect_serv(String val){
    if((WiFi.status() == WL_CONNECTED)) {
    http.begin(server, 80, "/"+folder+"/cmd.php?"+val+"");
        int httpCode = http.GET();
        if(httpCode) {
            Serial.printf("[HTTP] GET... code: %d\n", httpCode);

            // file found at server
            if(httpCode == 200) {
              String payload = http.getString();
               // http.writeToStream(&USE_SERIAL);
                Serial1.println("1");
            }else{
                Serial1.println("0");
            }
        } else {
            Serial.print("[HTTP] GET... failed, no connection or no HTTP server\n");
        }
    } 
}

void callback(String topic, byte* message, unsigned int length) {
  Serial.print("Message arrived on topic: ");
  Serial.print(topic);
  Serial.print(". Message: ");
  String messageTemp;
  
  for (int i = 0; i < length; i++) {
    Serial.print((char)message[i]);
    messageTemp += (char)message[i];
  }
  Serial.println();

  // Feel free to add more if statements to control more GPIOs with MQTT
  // If a message is received on the topic esp8266/output, you check if the message is either on or off. 
  // Turns the output according to the message received
  if(topic=="esp8266/output"){
    Serial.print("Changing output to ");
    if(messageTemp == "on"){
      digitalWrite(D7, HIGH);
      delay(50);
      digitalWrite(D7, LOW);
      Serial.print("on");
    }
    else if(messageTemp == "off"){
      digitalWrite(D5, LOW);
      Serial.print("off");
    }
  }
  Serial.println();
}

void setup() {
  Serial.begin(9600);
  Serial1.begin(9600);
  lcd.begin (16,2); 
  lcd.setBacklightPin(3,POSITIVE);
  lcd.setBacklight(HIGH);
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.print("WiFi connected - ESP IP address: ");
  Serial.println(WiFi.localIP());
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
  dht.begin();
  pinMode(D0,INPUT);
  pinMode(D5,INPUT_PULLUP);
  pinMode(D6,INPUT_PULLUP);
  pinMode(A0,INPUT);
  pinMode(D7,OUTPUT);
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  if (buttonState[1] != lastButtonState[1]) {
    if (buttonState[1] == 1) {
    Line_Notify(LINE_TOKEN,"Warning%20high%20CO");//%20 = Space bar ห้ามเว้นวรรคเด็ดขาด ต้องใช้ %20 เท่านั้นถ้าจะเว้น
    Serial.println("Line send");
    }
    lastButtonState[1] = buttonState[1]; 
  }

  unsigned long currentMillis = millis();
if (currentMillis - previousMillis >= interval) {
  previousMillis = currentMillis;
   h = dht.readHumidity();
   t = dht.readTemperature();
   light=analogRead(A0);
   co=digitalRead(D0);
  
  client.publish("esp8266/temp", String(t,2).c_str());
  client.publish("esp8266/humi", String(h,2).c_str());
  client.publish("esp8266/light", String(light).c_str());
  //client.publish("esp8266/state", String("PH HIGH").c_str());

  if(digitalRead(D5)==LOW){
  client.publish("esp8266/door", "Door Open");
  }else{
  client.publish("esp8266/door", "Door Close");
  }

  if(digitalRead(D6)==LOW){
  client.publish("esp8266/pir", "Detect!");
  client.publish("esp8266/state", String("There are people").c_str());
  }else{
  client.publish("esp8266/pir", "Not Detect!");
  client.publish("esp8266/state", String("No people present").c_str());
  }

  if(digitalRead(D0)==HIGH){
  client.publish("esp8266/co", "GAS Leak");
  buttonState[1]=1;
  }else{
  client.publish("esp8266/co", "GAS Normal");
  buttonState[1]=0;
  }
  

  cnt+=1;
  if(cnt>=10){
  connect_serv("t="+String(t,2)+"&h="+String(h,2)+"");
  cnt=0;

    }

lcd.setCursor(0,0);
lcd.print("T:");
lcd.print(t,1);   
lcd.print(" ");

lcd.setCursor(7,0);
lcd.print("H:");
lcd.print(h,1);
lcd.print(" ");

lcd.setCursor(0,1);
lcd.print("L:");
lcd.print(light);   
lcd.print(" ");

lcd.setCursor(7,1);
lcd.print("MQ:");
lcd.print(cov[co]);
lcd.print(" ");

}

}


void reconnect() {
  // Loop until we're reconnected
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    // Create a random client ID
    String clientId = "ESP8266Client-";
    clientId += String(random(0xffff), HEX);
    // Attempt to connect
    if (client.connect(clientId.c_str())) {
      Serial.println("connected");  
      // Subscribe or resubscribe to a topic
      // You can subscribe to more topics (to control more outputs)
      client.subscribe("esp8266/output");  
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      // Wait 5 seconds before retrying
      delay(5000);
    }
  }
}










-----------------------------------------------------------2.ESP32-----------------------------------------------------------
#include <WiFi.h>
#include "esp_camera.h"
#include "esp_system.h"
void Camera_capture();

#include <TridentTD_LineNotify.h>
#define SSID        "eieigum"   //WiFi name
#define PASSWORD    "eieilovelove"   //PASSWORD
#define LINE_TOKEN  "linetoken"   

// Pin definition for CAMERA_MODEL_AI_THINKER
#define PWDN_GPIO_NUM     32
#define RESET_GPIO_NUM    -1
#define XCLK_GPIO_NUM      0
#define SIOD_GPIO_NUM     26
#define SIOC_GPIO_NUM     27

#define Y9_GPIO_NUM       35
#define Y8_GPIO_NUM       34
#define Y7_GPIO_NUM       39
#define Y6_GPIO_NUM       36
#define Y5_GPIO_NUM       21
#define Y4_GPIO_NUM       19
#define Y3_GPIO_NUM       18
#define Y2_GPIO_NUM        5
#define VSYNC_GPIO_NUM    25
#define HREF_GPIO_NUM     23
#define PCLK_GPIO_NUM     22

const int Led_Flash = 4;
boolean startTimer = false;
unsigned long time_now=0;
int time_capture=0;

const int buttonPin=12;
  int buttonState;
  int lastButtonState;

void setup() {

  Serial.begin(115200);
   while (!Serial) {  ;  }
  pinMode(Led_Flash, OUTPUT);
  WiFi.begin(SSID, PASSWORD);
  Serial.printf("WiFi connecting to %s\n",  SSID);
  while(WiFi.status() != WL_CONNECTED) { Serial.print("."); delay(400); }
  Serial.printf("\nWiFi connected\nIP : ");
  Serial.println(WiFi.localIP());  
  LINE.setToken(LINE_TOKEN);
  pinMode(13,OUTPUT);
  digitalWrite(13,HIGH);

  camera_config_t config;
  config.ledc_channel = LEDC_CHANNEL_0;
  config.ledc_timer = LEDC_TIMER_0;
  config.pin_d0 = Y2_GPIO_NUM;
  config.pin_d1 = Y3_GPIO_NUM;
  config.pin_d2 = Y4_GPIO_NUM;
  config.pin_d3 = Y5_GPIO_NUM;
  config.pin_d4 = Y6_GPIO_NUM;
  config.pin_d5 = Y7_GPIO_NUM;
  config.pin_d6 = Y8_GPIO_NUM;
  config.pin_d7 = Y9_GPIO_NUM;
  config.pin_xclk = XCLK_GPIO_NUM;
  config.pin_pclk = PCLK_GPIO_NUM;
  config.pin_vsync = VSYNC_GPIO_NUM;
  config.pin_href = HREF_GPIO_NUM;
  config.pin_sscb_sda = SIOD_GPIO_NUM;
  config.pin_sscb_scl = SIOC_GPIO_NUM;
  config.pin_pwdn = PWDN_GPIO_NUM;
  config.pin_reset = RESET_GPIO_NUM;
  config.xclk_freq_hz = 20000000;
  config.pixel_format = PIXFORMAT_JPEG; 
  
  if(psramFound()){
// FRAMESIZE_ +
//QQVGA/160x120//QQVGA2/128x160//QCIF/176x144//HQVGA/240x176
//QVGA/320x240//CIF/400x296//VGA/640x480//SVGA/800x600//XGA/1024x768
//SXGA/1280x1024//UXGA/1600x1200//QXGA/2048*1536
    config.frame_size = FRAMESIZE_SXGA; 
    config.jpeg_quality = 10;
    config.fb_count = 2;
  } else {
    config.frame_size = FRAMESIZE_QQVGA;
    config.jpeg_quality = 12;
    config.fb_count = 1;
  }
  pinMode(12,INPUT_PULLUP);
  // Init Camera
  esp_err_t err = esp_camera_init(&config);
  if (err != ESP_OK) {
    Serial.printf("Camera init failed with error 0x%x", err);
    return;
  }
}
void loop() {
buttonState = digitalRead(buttonPin);

if (buttonState != lastButtonState) {
    if (buttonState == LOW) {
      Camera_capture();
      startTimer = true; 
    }
    lastButtonState = buttonState; 
  }
}
void Camera_capture() {
  digitalWrite(Led_Flash, HIGH);
  delay(100); 
  digitalWrite(Led_Flash, LOW);
  delay(100);
  digitalWrite(Led_Flash, HIGH);
  camera_fb_t * fb = NULL;
  delay(200); 
  // Take Picture with Camera
  fb = esp_camera_fb_get(); 
  if(!fb) {
    Serial.println("Camera capture failed");
    return;
  }
   digitalWrite(Led_Flash, LOW);
   Send_line(fb->buf,fb->len);
   esp_camera_fb_return(fb); 
  // Serial.println("Going to sleep now");
  // esp_deep_sleep_start();
  // Serial.println("This will never be printed");

}

void Send_line(uint8_t *image_data,size_t   image_size){
   LINE.notifyPicture("DETECT!!",image_data, image_size);
   digitalWrite(13,LOW);
   delay(5000);
   digitalWrite(13,HIGH);
  }
