# Documentação Técnica – Recife de Memórias

## 1. Visão Geral

O **Recife de Memórias** é um jogo sensorial que proporciona ao jogador uma jornada interativa pelos cinco principais pontos turísticos do bairro do Recife. A cada rodada:

1. O Arduino Mega, com sete botões físicos (cinco de escolha, um de Start e um de Reset) e dois LEDs (verde/vermelho), aguarda o comando de início.
2. Ao pressionar **Start**, o Arduino envia `INICIAR` à aplicação Python.
3. O Python seleciona aleatoriamente um dos pontos turísticos — **Cais do Sertão**, **Marco Zero**, **Paço do Frevo**, **Parque das Esculturas** ou **Rua do Bom Jesus** — e reproduz o áudio correspondente.
4. Após o áudio, o Python envia um número (1–5) ao Arduino, indicando qual botão o jogador deve pressionar.
5. O Arduino aguarda até 15 segundos por uma resposta:
   - **Acerto**: acende o LED verde e envia `CORRETO`.
   - **Erro**: acende o LED vermelho e envia `ERRADO`.
   - **Sem resposta**: pisca os LEDs e envia `TEMPO_ESGOTADO`.
6. Isso se repete por cinco rodadas. O botão **Reset** pode ser pressionado a qualquer momento para reiniciar o jogo.

---

## 2. Componentes e Requisitos

### 2.1 Hardware

- **Placa**: Arduino Mega  
- **Botões (com INPUT_PULLUP)**:
  - Amarelo → pino 52
  - Laranja → pino 48
  - Rosa    → pino 44
  - Verde   → pino 40
  - Azul    → pino 36
  - Start   → pino 12
  - Reset   → pino 11
- **LEDs**:
  - Verde    → pino 9
  - Vermelho → pino 6

> Os botões devem estar conectados entre o pino digital e o GND. Os LEDs devem ser ligados com resistores (~220Ω) entre o pino e o GND.

### 2.2 Software

- **Python 3.x**
- Bibliotecas Python:
  - `pyserial`
  - `pygame`
- **Áudios**: 5 arquivos `.wav`, nomeados como:
  - `caisdosertao.wav`
  - `marcozero.wav`
  - `pacodofrevo.wav`
  - `parquedasesculturas.wav`
  - `ruadobomjesus.wav`

---

## 3. Firmware Arduino

### 3.1 Setup

```cpp
void setup() {
  Serial.begin(9600);
  while (!Serial);
  
  pinMode(btnAmarelo, INPUT_PULLUP);
  pinMode(btnLaranja, INPUT_PULLUP);
  pinMode(btnRosa, INPUT_PULLUP);
  pinMode(btnVerde, INPUT_PULLUP);
  pinMode(btnAzul, INPUT_PULLUP);
  pinMode(btnStart, INPUT_PULLUP);
  pinMode(btnReset, INPUT_PULLUP);

  pinMode(ledVerde, OUTPUT);
  pinMode(ledVermelho, OUTPUT);
  digitalWrite(ledVerde, LOW);
  digitalWrite(ledVermelho, LOW);

  Serial.println("ARDUINO_INICIADO");
}
````

---

### 3.2 Funções Críticas

#### `getPressedButton()`

```cpp
int getPressedButton() {
  if (digitalRead(btnAmarelo) == LOW) { delay(50); if (digitalRead(btnAmarelo) == LOW) return 1; }
  if (digitalRead(btnLaranja) == LOW) { delay(50); if (digitalRead(btnLaranja) == LOW) return 2; }
  if (digitalRead(btnRosa) == LOW) { delay(50); if (digitalRead(btnRosa) == LOW) return 3; }
  if (digitalRead(btnVerde) == LOW) { delay(50); if (digitalRead(btnVerde) == LOW) return 4; }
  if (digitalRead(btnAzul) == LOW) { delay(50); if (digitalRead(btnAzul) == LOW) return 5; }
  return -1;
}
```

#### `checkStartButton()`

```cpp
void checkStartButton() {
  if (currentState == WAITING && digitalRead(btnStart) == LOW) {
    delay(50);
    if (digitalRead(btnStart) == LOW) {
      Serial.println("INICIAR");
      currentState = PLAYING;
    }
  }
}
```

#### `checkResetButton()`

```cpp
void checkResetButton() {
  if (digitalRead(btnReset) == LOW) {
    delay(50);
    if (digitalRead(btnReset) == LOW) {
      Serial.println("RESET");
      currentState = WAITING;
    }
  }
}
```

#### `processGame()`

```cpp
void processGame() {
  if (Serial.available()) {
    String message = Serial.readStringUntil('\n');
    int expectedButton = message.toInt();

    Serial.print("Recebido: ");
    Serial.println(expectedButton);
    Serial.print("Esperando botão (esperado ");
    Serial.print(expectedButton);
    Serial.println(")...");

    unsigned long startTime = millis();
    int pressedButton = -1;

    while ((millis() - startTime < 15000) && pressedButton == -1) {
      pressedButton = getPressedButton();

      if (pressedButton != -1) {
        Serial.print("Botao detectado: ");
        Serial.println(pressedButton);
        bool acerto = (pressedButton == expectedButton);
        digitalWrite(acerto ? ledVerde : ledVermelho, HIGH);
        Serial.println(acerto ? "CORRETO" : "ERRADO");
        delay(2000);
        digitalWrite(ledVerde, LOW);
        digitalWrite(ledVermelho, LOW);
        return;
      }

      if (digitalRead(btnReset) == LOW) {
        Serial.println("RESET");
        currentState = WAITING;
        return;
      }

      blinkLeds();
    }

    Serial.println("TEMPO_ESGOTADO");
  }
}
```

---

### 3.3 Loop Principal

```cpp
void loop() {
  checkStartButton();
  checkResetButton();
  if (currentState == PLAYING) {
    processGame();
  }
  delay(10);
}
```

---

## 4. Software Python

### 4.1 Estrutura da Classe `AudioGame`

* `__init__()` – define os arquivos de áudio e chama `setup_audio_files()`.
* `setup_audio_files()` – verifica se os arquivos `.wav` existem.
* `connect_arduino()` – busca portas seriais, conecta a 9600 baud.
* `play_audio()` – toca o áudio com `pygame`, e monitora o canal para `RESET`.
* `game_loop()` – executa as 5 rodadas do jogo.
* `run()` – fluxo principal: conecta, espera `"INICIAR"`, executa o jogo, reinicia se necessário.

### 4.2 Fluxo de Execução
```
Python inicia → connect_arduino() → aguarda "ARDUINO_INICIADO"
Botão Start → Arduino → "INICIAR" → Python inicia game_loop()
Para cada rodada:
  Python limpa buffer, envia número, toca áudio
  Arduino processa e envia "CORRETO", "ERRADO" ou "TEMPO_ESGOTADO"
  Python registra resposta e avança
Após 5 rodadas → Python envia "RESET"
Arduino retorna a WAITING
```

---

## 5. Notas de Depuração e Boas Práticas

* **Debounce**: `delay(50)` suficiente para eliminar múltiplas leituras.
* **Buffer serial**: sempre limpe com `reset_input_buffer()` antes de enviar comandos.
* **Mensagens seriais**: use `Serial.println()` no Arduino e `readline().decode().strip()` no Python para consistência.
* **Permissões**: no Linux, adicione o usuário ao grupo `dialout`.
* **Logs**: verifique tanto o console do Python quanto o Monitor Serial do Arduino para depuração.

---

## 6. Estrutura Recomendada de Diretórios
```
projeto-recife/
├── docs/
│   └── DOCUMENTATION.md
├── jogo.py
├── arduino.ino
├── audios/
│   ├── caisdosertao.wav
│   ├── marcozero.wav
│   ├── pacodofrevo.wav
│   ├── parquedasesculturas.wav
│   └── ruadobomjesus.wav
└── README.md
```
