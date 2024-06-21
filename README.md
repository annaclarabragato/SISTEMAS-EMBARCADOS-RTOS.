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

> 1. Broker
O broker é o servidor central que gerencia a troca de mensagens entre os clientes. Ele recebe mensagens publicadas por clientes e as encaminha para outros clientes que estão inscritos nos tópicos correspondentes.

Funções do broker:
Receber mensagens dos publishers.
Filtrar mensagens por tópico.
Enviar mensagens para os subscribers apropriados.
Gerenciar conexões de clientes.
Implementar políticas de segurança e autenticação.

> 2. Clients (Clientes)
Os clientes são dispositivos ou aplicativos que se comunicam entre si através do broker. Cada cliente pode ser um publisher, um subscriber ou ambos.

Publisher:
Envia mensagens para um ou mais tópicos no broker.
Subscriber:
Recebe mensagens de um ou mais tópicos aos quais está inscrito.

> 3. Messages (Mensagens)
As mensagens são os dados trocados entre os clientes. Cada mensagem tem um tópico associado, que é usado para determinar quais subscribers devem receber a mensagem.

Componentes de uma mensagem:
Tópico: Identifica a categoria da mensagem.
Payload (Carga útil): O conteúdo da mensagem.
Quality of Service (QoS): Nível de garantia de entrega da mensagem.
Retain Flag: Indica se a mensagem deve ser retida pelo broker.

> 4. Topics (Tópicos)
Tópicos são strings hierárquicas que servem como endereços para mensagens. Os publishers enviam mensagens para tópicos específicos, e os subscribers se inscrevem nesses tópicos para receber mensagens.

Hierarquia de tópicos: Por exemplo, casa/sala/temperatura.
Wildcards: + (corresponde a um único nível) e # (corresponde a múltiplos níveis).

> 5. Sessions (Sessões)
As sessões são usadas para manter o estado de conexão entre um cliente e o broker, permitindo reconexões e a entrega de mensagens pendentes.

Sessões persistentes: Mantêm o estado e mensagens não entregues mesmo após a desconexão.
Sessões não persistentes: Não mantêm o estado após a desconexão.

> 6. Quality of Service (QoS)
O QoS define o nível de garantia de entrega das mensagens.

QoS 0 - At most once: Mensagens são entregues no máximo uma vez, sem confirmação.
QoS 1 - At least once: Mensagens são entregues pelo menos uma vez, com confirmação.
QoS 2 - Exactly once: Mensagens são entregues exatamente uma vez, com um protocolo de handshake mais complexo.

> 7. Last Will and Testament (LWT)
O LWT é uma mensagem que um cliente pode definir para ser enviada pelo broker em caso de desconexão inesperada do cliente. Isso é útil para notificar outros clientes sobre a falha de um dispositivo. 

> 2º em seguida, o código utilizado na atividade:

```sh
#include <Arduino.h>
#include <ESP8266WiFi.h>
#include <PubSubClient.h>
> inclui as bibliotecas necessárias

const char* ssid = "AndroidAP";
const char* password = "nssa9988";
const char* mqtt_server = "192.168.43.58";
> define as credenciais da rede Wi-Fi (ssid e password) à qual o ESP8266 se conectará, e o endereço do servidor MQTT local (mqtt_server)

WiFiClient espClient;
PubSubClient client(espClient);
const int Relay = D1;
> cria um cliente WiFi (espClient) e um cliente MQTT (client) usando o objeto espClient. Define o pino D1 como saída para controlar o relé

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
> função para conectar o ESP8266 à rede Wi-Fi. Ele tenta conectar e espera até a conexão ser estabelecida, exibindo o progresso no monitor serial

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
> callback que é chamada quando uma mensagem MQTT é recebida. Ele interpreta o tópico e a mensagem recebida, e controla o relé (Relay) com base no conteúdo da mensagem

void setup() {
  pinMode(Relay, OUTPUT);
  Serial.begin(9600);
  setup_wifi();
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
}
> configurações iniciais no setup(): define o pino do relé como saída, inicia a comunicação serial e a conexão Wi-Fi, configura o servidor MQTT e define a função de callback para mensagens recebidas

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
> função para reconectar ao servidor MQTT caso a conexão seja perdida. Tenta conectar-se e assina o tópico "ControleRelay" após a conexão ser bem-sucedida

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();
}
> função loop() principal que verifica se o cliente MQTT está conectado. Se não estiver, chama reconnect() para tentar se reconectar. o método client.loop() é usado para manter a comunicação com o servidor MQTT
```

> 3º os prints referentes a: broker:

![image](https://github.com/annaclarabragato/SISTEMAS-EMBARCADOS-RTOS./assets/125417531/f19dc58e-313c-4c66-8402-50a513595980)

> referente ao sub:

![image](https://github.com/annaclarabragato/SISTEMAS-EMBARCADOS-RTOS./assets/125417531/515af302-b3ec-470f-9a9d-9312ca7f7b86)

> referente ao pub:

![image](https://github.com/annaclarabragato/SISTEMAS-EMBARCADOS-RTOS./assets/125417531/77a996a4-e9e2-4899-8722-c7bd38b853ab)

> referente ao led ligando e desligando, na aba terminal do vscode:

![image](https://github.com/annaclarabragato/SISTEMAS-EMBARCADOS-RTOS./assets/125417531/7eb2333c-c26b-4b1e-9fae-acd3de492080)
