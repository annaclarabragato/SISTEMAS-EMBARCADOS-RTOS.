# SISTEMAS-EMBARCADOS-RTOS.

### a seguir, estarão os prints do esquemático, PCB, e 3D como pedido no roteiro.

> esquemático: ☆

![image](https://github.com/annaclarabragato/SISTEMAS-EMBARCADOS-RTOS./assets/125417531/a05ef127-8977-4b81-a1e3-2fb68f4784f8)

> pcb: ☆

![image](https://github.com/annaclarabragato/SISTEMAS-EMBARCADOS-RTOS./assets/125417531/ca90aab7-526e-405f-b0ad-93bf2230213d)

> 3D: ☆

## parte traseira da placa:
![image](https://github.com/annaclarabragato/SISTEMAS-EMBARCADOS-RTOS./assets/125417531/4e3058f7-13ae-4c6a-a883-38be048ce9bd)

## parte frontal:
![image](https://github.com/annaclarabragato/SISTEMAS-EMBARCADOS-RTOS./assets/125417531/208b6c97-1d15-434c-9f87-c89dbf43b888)


# ATIVIDADE MQTT:

> 1º como o MQTT funciona: ele parte de um broker, no qual é como um tópico em que tudo estará sendo descrito. junto dele, há o pub e o sub. o sub, não envia nada, apenas recebe, o pub é o contrário disto. como será visto na atividade em seguida, demonstrando como o pub envia "ControleRelay: ON", passa pelo broker e retorna então para o sub.

> 2º em seguida, o código utilizado na atividade:

``` sh #include <Arduino.h>
#include <ESP8266WiFi.h>
#include <PubSubClient.h>

const char* ssid = "AndroidAP";
const char* password = "nssa9988";
const char* mqtt_server = "192.168.43.58";

WiFiClient espClient;
PubSubClient client(espClient);
const int Relay = D1;

void setup_wifi() {
  delay(10);
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
  delay(500);
  Serial.print("."); 
  }
  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address");
  Serial.println(WiFi.localIP());
}

void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("]");
  String message;
  for (int i = 0; i < length; i++) {
    message += ((char)payload[i]);

  }
  Serial.println(message);

  if (String(topic) == "ControleRelay"){
    if (message == "ON") {
      digitalWrite(Relay, LOW); 
    }else if (message == "OFF") {
      digitalWrite(Relay, HIGH);
    }
    }
  }

void setup() {
  pinMode(Relay, OUTPUT);
  Serial.begin(9600);
  setup_wifi();
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
}

void reconnect() {
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    if (client.connect("ESP8266Client")) {
      Serial.println("connected");
      client.subscribe("ControleRelay");
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      delay(5000);
    }
  }
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();
} ``` sh

> 3º os prints referentes a: broker:

![image](https://github.com/annaclarabragato/SISTEMAS-EMBARCADOS-RTOS./assets/125417531/f19dc58e-313c-4c66-8402-50a513595980)

> referente ao sub:

![image](https://github.com/annaclarabragato/SISTEMAS-EMBARCADOS-RTOS./assets/125417531/515af302-b3ec-470f-9a9d-9312ca7f7b86)

> referente ao pub:

![image](https://github.com/annaclarabragato/SISTEMAS-EMBARCADOS-RTOS./assets/125417531/77a996a4-e9e2-4899-8722-c7bd38b853ab)

> referente ao led ligando e desligando, na aba terminal do vscode:

![image](https://github.com/annaclarabragato/SISTEMAS-EMBARCADOS-RTOS./assets/125417531/7eb2333c-c26b-4b1e-9fae-acd3de492080)
