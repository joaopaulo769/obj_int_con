obj_int_con

Repositório do código do trabalho de conclusão da disciplina: Objetos Inteligentes conectados.

**
 * @file ESP8266_ESP-01_Entradas_Send_MQTT.ino
 * @author Saulo Aislan
 * @brief Firmware que lê entradas digital e analógica e envia os dados via MQTT.
 * @version 0.1
 */

#include "EspMQTTClient.h"

// Configurações WiFi e MQTT
EspMQTTClient client(
  "Wokwi-GUEST",         // SSID WiFi
  "",                    // Password WiFi
  "test.mosquitto.org",  // MQTT Broker
  "mqtt-wokwi",          // Client
  1883                   // MQTT port
);

// Definição dos pinos
const int pinoDigital = 12; // Pino para a entrada digital
const int pinoAnalogico = A0; // Pino para a entrada analógica

// Variáveis para armazenar os valores dos sensores
int Dsensor; // Valor da entrada digital
int Asensor; // Valor da entrada analógica

void setup() {
  Serial.begin(115200);

  // Configuração do pino digital como entrada
  pinMode(pinoDigital, INPUT);

  // Configuração do cliente MQTT
  client.enableDebuggingMessages(); // Habilita mensagens de debug na saída serial
  client.enableHTTPWebUpdater(); // Habilita o web updater
  client.enableOTA(); // Habilita atualizações OTA
  client.enableLastWillMessage("TestClient/lastwill", "Vou ficar offline");
}

/**
 * @brief Ler os dados das entradas e enviar via MQTT
 */
void lerEnviarDados() {
  // Leitura dos valores dos pinos digital e analógico
  Dsensor = digitalRead(pinoDigital);
  Asensor = analogRead(pinoAnalogico);

  // Exibição dos valores lidos
  Serial.print("Valor Digital (Dsensor): ");
  Serial.print(Dsensor);
  Serial.print(" | Valor Analógico (Asensor): ");
  Serial.println(Asensor);

  // Publicação dos valores dos pinos digital e analógico via MQTT
  client.publish("topicowokwi/Dsensor", String(Dsensor));
  client.publish("topicowokwi/Asensor", String(Asensor));
}

/**
 * @brief Esta função é chamada quando tudo estiver conectado (WiFi e MQTT).
 * Para utilizá-la, deve-se implementar o struct EspMQTTClient
 */
void onConnectionEstablished() {
  // Subscribe no "topicowokwi/msgRecebida/#" e mostra a mensagem recebida na Serial
  client.subscribe("topicowokwi/msgRecebida/#", [](const String & topic, const String & payload) {
    Serial.println("Mensagem recebida no tópico: " + topic + ", payload: " + payload);
  });

  // Chamada da função para ler e enviar dados
  lerEnviarDados();
}

void loop() {
  client.loop(); // Executa em loop
}
