# iotLM35
Código do projeto de Objetos Inteligentes Conectados com o sensor LM35. 05K

include <ESP8266WiFi.h>
include <PubSubClient.h>

define ID_MQTT "lm35_mqtt"
define TOPICO_PUBLISH_SENSOR_TEMP "topico_sensor_temp"

const char * BROKER = "test.mosquitto.org";
int PORTA = 1883;

int sensorPin = A0;

char * SSID = "NET_2G_217";
char * PASSWORD = "3133032750";

WiFiClient espClient;
PubSubClient MQTT(espClient);

int sensorPin_Value = 0;
int tempC = 0;
char * msg = "";

void ReconectMQTT(void) {
  if (!MQTT.connected()){
    while (!MQTT.connected()) {
      Serial.print("* Conectando-se ao MQTT: ");
      Serial.println(BROKER);
      if (MQTT.connect(ID_MQTT)) {
        Serial.println("Conectado com sucesso");
      } else {
        Serial1.println("Falha ao se conectar.");
        Serial.println("Reiniciando em 2s");
        delay(2000);
      }
    }
  }
}

// Iniciando o MQTT
void StartMQTT(void) {
  MQTT.setServer(BROKER, PORTA);
}

void ReconectWIFI(void) {
  if (WiFi.status() == WL_CONNECTED)
    return;
  WiFi.begin(SSID, PASSWORD);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.print("Conectado na rede ");
}

void ConsultaConexoes(void) {
  ReconectWIFI(); 
  ReconectMQTT(); 
}

float sVoltage;

void setup() 
{
  Serial.begin(9600);
  pinMode(D0, OUTPUT);
  StartMQTT();
}

void loop()      
{
  ConsultaConexoes();
  float xVal = analogRead(sensorPin);

  sVoltage = (xVal*3100.0)/1023;

  tempC = sVoltage;
  Serial.print("A temperatura é de = ");
  Serial.println(tempC);
  delay(3000);

  digitalWrite(D0, HIGH);
  delay(1000);

  digitalWrite(D0, LOW);
  delay(1000);

  sprintf(msg, "%d", tempC);
  
  MQTT.publish(TOPICO_PUBLISH_SENSOR_TEMP, msg);
  MQTT.loop();
  delay(3000);
 
}
