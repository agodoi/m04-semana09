# Integração Geral entre Entrada, Sensores, Atuadores e a Interface com o Usuário

## Descrição
O objetivo deste encontro é construir o fluxo completo de entrada e processamento de sensores, saídas informacionais e controle e apresentação em interface com o usuário embarcada, de forma a implementar regras de atuação/alertas previstas no escopo do projeto.

## Assuntos relacionados

- Atuadores
- Interfaces de comunicação
- Microcontroladores
- Programação de microcontroladores
- Sensores analógicos e digitais

## Correções a partir da Sprint 4

### 1) Geolocalização por WiFi

O ESP32 permite geolocalização por WiFi mas com baixa precisão e de forma complexa. A geolocalização por WiFi é possível por meio da triangulação de sinais WiFi, onde a posição do dispositivo é estimada com base na intensidade do sinal dBm de diferentes pontos de acesso WiFi nas proximidades.

No entanto, há algumas considerações a serem levadas em conta:

1. **Precisão Limitada:** A precisão da geolocalização por WiFi pode ser limitada em comparação com outras tecnologias, como GPS. A triangulação depende da disponibilidade de pontos de acesso WiFi na área e pode ser afetada por obstáculos físicos, interferências e variações na intensidade do sinal.

2. **Necessidade de Bancos de Dados:** Para realizar a triangulação, muitas vezes é necessário ter acesso a bancos de dados que mapeiam a localização dos pontos de acesso WiFi. Esses bancos de dados podem ser grandes e requerem atualizações regulares para garantir a precisão das informações de localização. Quanto maior o seu banco de dados com no mínimo 3 valores de dBm, maior será a precisão.

3. **Requisitos Adicionais:** Algumas implementações de geolocalização por WiFi podem exigir o uso de serviços externos, como Google Geolocation API ou serviços semelhantes, o que pode envolver custos ou requisitos adicionais.

4. **Configuração Adequada:** Para obter resultados precisos, é necessário configurar corretamente o ESP32 para escanear e interpretar os sinais WiFi ao seu redor. Isso pode exigir a implementação de algoritmos específicos e o ajuste de parâmetros.

### 2) Como fazer Geolocalização?

É bastante complexo, e por isso, não adotamos essa opção como solução no projeto.

**Materiais Necessários:**
1. ESP32 (com WiFi integrado)
2. Sensor de bússola (opcional para melhorar a precisão)
3. Conexão com a internet para acessar serviços de geolocalização

**Passos:**

1. **Configuração Inicial:**
   - Certifique-se de ter o Arduino IDE configurado com o suporte para o ESP32.
   - Instale as bibliotecas necessárias, incluindo a `WiFiScan` para escanear redes WiFi.

2. **Escaneamento de Redes WiFi:**
   - Use a biblioteca `WiFiScan` para escanear redes WiFi ao redor.
   - Armazene informações sobre os pontos de acesso, incluindo o BSSID (identificador único), a intensidade do sinal (RSSI) e possivelmente a localização conhecida dos pontos de acesso.

3. **Triangulação:**
   - Implemente um algoritmo de triangulação com base nas informações coletadas dos pontos de acesso WiFi.
   - Considere fatores como a intensidade do sinal, a distância e a localização conhecida dos pontos de acesso.

4. **Melhorias de Precisão (Opcional):**
   - Adicione um sensor de bússola para ajudar na orientação.
   - Implemente técnicas avançadas, como filtragem de Kalman, para melhorar a precisão.

5. **Acesso a Serviços de Geolocalização Online:**
   - Integre serviços de geolocalização online, como Google Maps Geolocation API ou serviços semelhantes, para obter informações adicionais.
   - Esses serviços podem fornecer uma correção de posição com base nas informações de WiFi e em bancos de dados de localização.

6. **Calibração e Testes:**
   - Realize calibrações regulares para garantir a precisão do sistema.
   - Realize testes em diferentes ambientes para avaliar a confiabilidade do sistema.

7. **Alimentação e Eficiência Energética:**
   - Considere estratégias para otimizar o consumo de energia, especialmente se o projeto for alimentado por bateria.

### 3) Código

Esse código-fonte lê 3 sinais dBm e calcula a distância Eucludiana, que é a distância em linha reta. O cálculo da distância é a alma desse código e por isso, não existe uma única solução.

```
# Algoritmo de Triangulação WiFi para ESP32

# Bibliotecas necessárias
#include <WiFi.h>

# Estrutura para armazenar informações de ponto de acesso
struct AccessPoint {
  String ssid;    # Nome da rede WiFi
  String bssid;   # Endereço MAC único do ponto de acesso
  int rssi;       # Intensidade do sinal recebido
};

# Função para escanear redes WiFi
void escanearRedesWiFi(AccessPoint pontosAcesso[], int& numeroPontosAcesso) {
  numeroPontosAcesso = WiFi.scanNetworks();  # Escanear redes WiFi
  for (int i = 0; i < numeroPontosAcesso; i++) {
    pontosAcesso[i].ssid = WiFi.SSID(i);
    pontosAcesso[i].bssid = WiFi.BSSIDstr(i);
    pontosAcesso[i].rssi = WiFi.RSSI(i);
  }
}

# Função para calcular a localização com base na triangulação
void calcularLocalizacao(AccessPoint pontosAcesso[], int numeroPontosAcesso) {
  # Verificar se há informações suficientes para a triangulação
  if (numeroPontosAcesso < 3) {
    Serial.println("Não há informações suficientes para a triangulação.");
    return;
  }

  # Algoritmo de Triangulação Simples (pode ser aprimorado)
  # Supõe uma área plana sem obstruções significativas

  float x = 0.0, y = 0.0;  # Coordenadas do dispositivo

  # Escolher três pontos de acesso com os melhores sinais
  int indexAP1 = 0;
  int indexAP2 = 1;
  int indexAP3 = 2;

  # Calcular distâncias estimadas usando a intensidade do sinal
  float d1 = calcularDistancia(pontosAcesso[indexAP1].rssi);
  float d2 = calcularDistancia(pontosAcesso[indexAP2].rssi);
  float d3 = calcularDistancia(pontosAcesso[indexAP3].rssi);

  # Calcular a localização usando a triangulação
  x = calcularX(pontosAcesso[indexAP1].x, pontosAcesso[indexAP2].x, pontosAcesso[indexAP3].x, d1, d2, d3);
  y = calcularY(pontosAcesso[indexAP1].y, pontosAcesso[indexAP2].y, pontosAcesso[indexAP3].y, d1, d2, d3);

  Serial.println("Localização estimada: ");
  Serial.print("X: ");
  Serial.println(x);
  Serial.print("Y: ");
  Serial.println(y);
}

# Função auxiliar para calcular distância estimada com base na intensidade do sinal
float calcularDistancia(int rssi) {
  # Implementar uma função que converta o RSSI em distância estimada
  # A relação entre RSSI e distância pode variar dependendo do ambiente
  # Pode ser necessário calibrar essa função experimentalmente
  return 0.0;  # Substituir com a implementação real
}

# Funções auxiliares para calcular a localização usando a triangulação
float calcularX(float x1, float x2, float x3, float d1, float d2, float d3) {
  # Implementar a fórmula de triangulação para calcular a coordenada X
  return 0.0;  # Substituir com a implementação real
}

float calcularY(float y1, float y2, float y3, float d1, float d2, float d3) {
  # Implementar a fórmula de triangulação para calcular a coordenada Y
  return 0.0;  # Substituir com a implementação real
}

# Função principal
void setup() {
  Serial.begin(115200);
  WiFi.begin("SSID", "senha");  # Substituir com suas credenciais WiFi

  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Conectando ao WiFi...");
  }

  Serial.println("Conectado ao WiFi.");

  AccessPoint pontosAcesso[50];  # Array para armazenar informações de até 50 pontos de acesso
  int numeroPontosAcesso = 0;

  escanearRedesWiFi(pontosAcesso, numeroPontosAcesso);
  calcularLocalizacao(pontosAcesso, numeroPontosAcesso);
}

void loop() {
  # O loop pode ser usado para escanear redes WiFi e recalcular a localização periodicamente
  delay(5000);  # Escanear e recalcular a cada 5 segundos (ajustar conforme necessário)
  escanearRedesWiFi(pontosAcesso, numeroPontosAcesso);
  calcularLocalizacao(pontosAcesso, numeroPontosAcesso);
}
```


Esse código abaixo possui um algoritmo de cálculo de distância. Ele precisa de ajustes caso você o utilize:

```
#include <WiFi.h>

// Estrutura para armazenar informações de ponto de acesso
struct AccessPoint {
  String ssid;
  String bssid;
  int rssi;
  float latitude;
  float longitude;
};

// Função para calcular a distância com base no RSSI
float calculateDistance(int rssi) {
  // Modelo de perda de percurso (path loss model)
  // Esta fórmula é um exemplo e pode precisar de ajustes com base no ambiente específico
  // Experimente calibrar a fórmula com medições reais em seu ambiente
  float A = -50; // Parâmetro de atenuação, dependendo do ambiente
  float n = 2.0; // Expoente do caminho, dependendo do ambiente

  return pow(10, (A - rssi) / (10 * n));
}

// Função para calcular a posição com base na triangulação
void triangulate(AccessPoint ap1, AccessPoint ap2, AccessPoint ap3, float &latitude, float &longitude) {
  // Fórmula da triangulação
  // x, y representam as coordenadas do ponto a ser localizado
  // r1, r2, r3 representam as distâncias do ponto aos pontos de acesso
  float x, y;
  float A1 = 2 * (ap2.latitude - ap1.latitude);
  float B1 = 2 * (ap2.longitude - ap1.longitude);
  float C1 = pow(ap2.latitude, 2) - pow(ap1.latitude, 2) + pow(ap2.longitude, 2) - pow(ap1.longitude, 2) + pow(ap1.rssi, 2) - pow(ap2.rssi, 2);

  float A2 = 2 * (ap3.latitude - ap1.latitude);
  float B2 = 2 * (ap3.longitude - ap1.longitude);
  float C2 = pow(ap3.latitude, 2) - pow(ap1.latitude, 2) + pow(ap3.longitude, 2) - pow(ap1.longitude, 2) + pow(ap1.rssi, 2) - pow(ap3.rssi, 2);

  // Solução para x, y
  x = (C1 * B2 - C2 * B1) / (A1 * B2 - A2 * B1);
  y = (C1 * A2 - C2 * A1) / (B1 * A2 - B2 * A1);

  latitude = x;
  longitude = y;
}

void setup() {
  Serial.begin(115200);

  // Conectar ao WiFi
  WiFi.begin("SSID", "PASSWORD");

  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Conectando ao WiFi...");
  }

  Serial.println("Conectado ao WiFi");
}

void loop() {
  // Escanear redes WiFi
  int numNetworks = WiFi.scanNetworks();

  if (numNetworks >= 3) { // Pelo menos três pontos de acesso são necessários para triangulação
    // Coletar informações dos três primeiros pontos de acesso
    AccessPoint ap1 = {"SSID1", WiFi.BSSIDstr(0), WiFi.RSSI(0), LATITUDE1, LONGITUDE1};
    AccessPoint ap2 = {"SSID2", WiFi.BSSIDstr(1), WiFi.RSSI(1), LATITUDE2, LONGITUDE2};
    AccessPoint ap3 = {"SSID3", WiFi.BSSIDstr(2), WiFi.RSSI(2), LATITUDE3, LONGITUDE3};

    // Calcular distâncias
    float distance1 = calculateDistance(ap1.rssi);
    float distance2 = calculateDistance(ap2.rssi);
    float distance3 = calculateDistance(ap3.rssi);

    // Calcular posição
    float latitude, longitude;
    triangulate(ap1, ap2, ap3, latitude, longitude);

    Serial.print("Latitude: ");
    Serial.println(latitude, 6);
    Serial.print("Longitude: ");
    Serial.println(longitude, 6);
  }

  delay(5000); // Aguardar antes de realizar o próximo escaneamento
}

```

### 4) Conclusões

1) Para o M4, não é trivial adotar soluções por medições de dBm para refinar a geolocalização onde a situação envolve limites de zonas;
   
2) No lugar do dBm, tente usar sensores de porta, de presença, de pressão, de movimentos, etc e crie um algoritmo para integrar a identificação de quem entrou ou saiu do ambiente;

3) O acréscimo de um módulo de GPS também não garante precisão **indoor** na sua geolocalização, pois o sinal do satélite não pode ter barreiras como telhados e coberturas. Em projetos do tipo **outdoor**, ok, compensa utilizá-lo.

### Outras Correções

#### 1) Arquitetura da Solução

#### 2) Manual do Projeto

#### 3) Diagrama UML

#### 4) Profundidade de Conceitos

#### 5) Padrão ABNT

## Kahoot Geral


<picture>
   <source media="(prefers-color-scheme: light)" srcset="https://github.com/agodoi/m04-semana09/blob/main/imgs/bombom.jpg">
   <img alt="Bombom" src="(https://github.com/agodoi/m04-semana09/blob/main/imgs/bombom.jpg)" width="700px">
</picture>
