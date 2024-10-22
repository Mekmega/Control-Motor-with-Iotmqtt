# Control-Motor-with-Iotmqtt
Youtube
https://youtu.be/UZxQt5q-JKY?si=wfgeJ6Zaq8gT5ceM

!!Arduino Code!!
#include <WiFi.h>
#include <PubSubClient.h>

const char* ssid = "your-ssid";  
const char* password = "your-password";  

const char* mqttServer = "your-IP"; 
const int mqttPort = 1883; 
const char* mqttClientID = "ESP32Client";  
const char* topic = "your-topic";  

WiFiClient espClient;
PubSubClient client(espClient);

const int motorPin1 = 2;  
const int motorPin2 = 4;

void setup() {
  Serial.begin(115200);

  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.println("Connecting to WiFi...");
  }
  Serial.println("Connected to WiFi");

  client.setServer(mqttServer, mqttPort);
  client.setCallback(callback);

  while (!client.connected()) {
    Serial.println("Connecting to MQTT...");
    if (client.connect(mqttClientID)) {
      Serial.println("Connected to MQTT");
      client.subscribe(topic);  
    } else {
      delay(5000);
    }
  }

  pinMode(motorPin1, OUTPUT);
  pinMode(motorPin2, OUTPUT);
}

void callback(char* topic, byte* message, unsigned int length) {
  String msg = "";
  for (int i = 0; i < length; i++) {
    msg += (char)message[i];
  }
  Serial.print("Message arrived: ");
  Serial.println(msg);

  if (msg == "forward") {
    digitalWrite(motorPin1, HIGH);  
    digitalWrite(motorPin2, LOW);
  } else if (msg == "backward") {
    digitalWrite(motorPin1, LOW);  
    digitalWrite(motorPin2, HIGH);
  } else if (msg == "stop") {
    digitalWrite(motorPin1, LOW);  
    digitalWrite(motorPin2, LOW);
  }
}

void loop() {
  client.loop(); 
}


***********************************
***********************************

!!Python Code!!
import paho.mqtt.client as mqtt
import pygame
import sys

MQTT_BROKER = "your-IP"  
MQTT_PORT = 1883
MQTT_TOPIC = "your-topic"

def on_connect(client, userdata, flags, rc):
    print(f"Connected with result code {rc}")

client = mqtt.Client()
client.on_connect = on_connect
client.connect(MQTT_BROKER, MQTT_PORT, 60)
client.loop_start()

pygame.init()
screen = pygame.display.set_mode((400, 300))
pygame.display.set_caption("Motor Control")

font = pygame.font.Font(None, 36)  
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)

status_text = "Press W to Move Forward, S to Move Backward, Q to Stop"

running = True
while running:
    screen.fill(BLACK)  
    text_surface = font.render(status_text, True, WHITE)  
    screen.blit(text_surface, (20, 150)) 
    pygame.display.flip() 

    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        elif event.type == pygame.KEYDOWN:
            if event.key == pygame.K_w:
                client.publish(MQTT_TOPIC, "forward")
                print("Moving Forward")
                status_text = "Moving Forward"
            elif event.key == pygame.K_s: 
                client.publish(MQTT_TOPIC, "backward")
                print("Moving Backward")
                status_text = "Moving Backward"
            elif event.key == pygame.K_q: 
                client.publish(MQTT_TOPIC, "stop")
                print("Stop")
                status_text = "Stop"

client.loop_stop()
client.disconnect()
pygame.quit()
sys.exit()
