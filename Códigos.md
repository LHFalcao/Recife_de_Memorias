# SOFTWARE ARDUÍNO
```cpp

const int btnStart    = 22; 
const int btnReset    = 24;

const int btnAzul     = 40;
const int btnRosa     = 48;
const int btnLaranja  = 42;
const int btnAmarelo  = 52;
const int btnVerde    = 36;

const int ledVerde    = 9;
const int ledVermelha = 6;

const char* audios[8] = {
    "parquedasesculturas.wav", 
    "marcozero.wav", 
    "ForteDasCincoPontas.wav", 
    "caisdosertao.wav", 
    "ruadobomjesus.wav",
    "introducaojogo-VEED-aprimorado-v2.wav",
    "sequenciafinal-VEED-aprimorado-v2.wav", 
    "vitoria-VEED-aprimorado-v2.wav" 
};

bool jogoAtivo = false;
int faseAtual = 0;
bool aguardandoResposta = false;

unsigned long tempoUltimoPressionado[10] = {0};
const int debounceDelay = 200; // Aumentado para 200ms para melhorar debounce

// Guardamos o estado anterior de cada botão para debounce
bool estadoAnterior[10] = {HIGH, HIGH, HIGH, HIGH, HIGH, HIGH, HIGH, HIGH, HIGH, HIGH};
// Array para a ordem aleatória das fases
int ordemFases[5] = {0,1,2,3,4};

// Função para embaralhar a ordem das fases
void embaralharOrdem() {
    for (int i = 4; i > 0; i--) {
        int j = random(0, i + 1);
        int temp = ordemFases[i];
        ordemFases[i] = ordemFases[j];
        ordemFases[j] = temp;
    }
}

bool botaoPressionado(int pino, int indice) {
    bool estadoAtual = digitalRead(pino);

    if (estadoAtual == LOW && estadoAnterior[indice] == HIGH && (millis() - tempoUltimoPressionado[indice] > debounceDelay)) {
        tempoUltimoPressionado[indice] = millis();
        estadoAnterior[indice] = LOW;
        return true;
    } else if (estadoAtual == HIGH) {
        estadoAnterior[indice] = HIGH;
    }
    return false;
}

void setup() {
    Serial.begin(9600);

    pinMode(btnStart, INPUT_PULLUP);
    pinMode(btnReset, INPUT_PULLUP);
    pinMode(btnAzul, INPUT_PULLUP);
    pinMode(btnRosa, INPUT_PULLUP);
    pinMode(btnLaranja, INPUT_PULLUP);
    pinMode(btnAmarelo, INPUT_PULLUP);
    pinMode(btnVerde, INPUT_PULLUP);

    pinMode(ledVerde, OUTPUT);
    pinMode(ledVermelha, OUTPUT);
    
    digitalWrite(ledVerde, LOW);
    digitalWrite(ledVermelha, LOW);

    randomSeed(analogRead(0)); // Inicializa o gerador de números aleatórios
}

unsigned long timerInicio = 0;
const unsigned long delayAcerto = 3000;
const unsigned long delayErro = 3000;
const unsigned long delayEntreFases = 1000;

enum EstadoJogo {
    ESPERANDO_APERTO,
    PROCESSANDO_ACERTO,
    PROCESSANDO_ERRO,
    ESPERANDO_ENTRE_FASES,
    REPETIR_SEQUENCIA_FINAL
};

EstadoJogo estado = ESPERANDO_APERTO;

// Array original dos botões, a ordem aleatória será usada para acessar esses
const int botoesCorretos[5] = {btnAzul, btnRosa, btnLaranja, btnAmarelo, btnVerde};
const int botoesIndices[5] = {0, 1, 2, 3, 4};

// Variaveis para repetição final
int repeticaoIndex = 0;

void loop() {
    if (botaoPressionado(btnReset, 5)) {
        jogoAtivo = false;
        faseAtual = 0;
        aguardandoResposta = false;
        estado = ESPERANDO_APERTO;
        repeticaoIndex = 0;
        Serial.println("RESET");
        digitalWrite(ledVerde, LOW);
        digitalWrite(ledVermelha, LOW);
    }

    if (!jogoAtivo && botaoPressionado(btnStart, 0)) {
        jogoAtivo = true;
        faseAtual = 0;
        aguardandoResposta = true;
        estado = ESPERANDO_APERTO;
        repeticaoIndex = 0;
        embaralharOrdem(); // Embaralha a ordem das fases no início
        Serial.println("START");
        delay(1000);
        Serial.println(audios[5]);  // Envia áudio de início de jogo: inicio_jogo.wav
        delay(1000);
        Serial.println(audios[ordemFases[faseAtual]]);
    }

    if (jogoAtivo && aguardandoResposta) {
        if (estado == ESPERANDO_APERTO) {

            if (estado == REPETIR_SEQUENCIA_FINAL) {
                // Isso nunca ocorre aqui (condição de segurança)
            }

            if (estado != REPETIR_SEQUENCIA_FINAL) {
                if (faseAtual < 5) {
                    int indiceFaseAtual = ordemFases[faseAtual];
                    // Verifica botão correto atual pela ordem aleatória
                    if (botaoPressionado(botoesCorretos[indiceFaseAtual], botoesIndices[indiceFaseAtual])) {
                        estado = PROCESSANDO_ACERTO;
                        timerInicio = millis();
                        aguardandoResposta = false;

                        digitalWrite(ledVerde, HIGH);
                        digitalWrite(ledVermelha, LOW);
                        Serial.println("Acerto!");
                    }
                    else if (
                        (botaoPressionado(btnAzul, 0) && botoesCorretos[indiceFaseAtual] != btnAzul) ||
                        (botaoPressionado(btnRosa, 1) && botoesCorretos[indiceFaseAtual] != btnRosa) ||
                        (botaoPressionado(btnLaranja, 2) && botoesCorretos[indiceFaseAtual] != btnLaranja) ||
                        (botaoPressionado(btnAmarelo, 3) && botoesCorretos[indiceFaseAtual] != btnAmarelo) ||
                        (botaoPressionado(btnVerde, 4) && botoesCorretos[indiceFaseAtual] != btnVerde)
                    ) {
                        estado = PROCESSANDO_ERRO;
                        timerInicio = millis();

                        digitalWrite(ledVerde, LOW);
                        digitalWrite(ledVermelha, HIGH);
                        Serial.println("ERRO - Repetindo áudio...");
                        if (faseAtual < 5) {
                            Serial.println(audios[ordemFases[faseAtual]]);
                        }
                        aguardandoResposta = false;
                    }
                }
            }
        }
        else if (estado == REPETIR_SEQUENCIA_FINAL) {
            int indiceRepeticao = ordemFases[repeticaoIndex];
            // Sequencia final, usuário deve repetir toda sequência
            if (botaoPressionado(botoesCorretos[indiceRepeticao], botoesIndices[indiceRepeticao])) {
                repeticaoIndex++;
                digitalWrite(ledVerde, HIGH);
                digitalWrite(ledVermelha, LOW);
                Serial.print("Botao correto na sequencia final: ");
                Serial.println(repeticaoIndex);

                delay(1000); // delay alterado para 1 segundo para acerto

                digitalWrite(ledVerde, LOW);

                if (repeticaoIndex >= 5) {
                    Serial.println("PARABÉNS! Você ganhou o jogo.");
                    Serial.println(audios[7]);  // Envia áudio de vitória
                    
                    jogoAtivo = false;
                    aguardandoResposta = false;
                    estado = ESPERANDO_APERTO;
                    repeticaoIndex = 0;
                }
            }
            else if (
                (botaoPressionado(btnAzul, 0) && botoesCorretos[indiceRepeticao] != btnAzul) ||
                (botaoPressionado(btnRosa, 1) && botoesCorretos[indiceRepeticao] != btnRosa) ||
                (botaoPressionado(btnLaranja, 2) && botoesCorretos[indiceRepeticao] != btnLaranja) ||
                (botaoPressionado(btnAmarelo, 3) && botoesCorretos[indiceRepeticao] != btnAmarelo) ||
                (botaoPressionado(btnVerde, 4) && botoesCorretos[indiceRepeticao] != btnVerde)
            ) {
                digitalWrite(ledVerde, LOW);
                digitalWrite(ledVermelha, HIGH);
                Serial.println("ERRO na repetição da sequência final - Repita novamente!");
                delay(1000);
                digitalWrite(ledVermelha, LOW);
                repeticaoIndex = 0;
            }
        }
    }

    if (estado == PROCESSANDO_ACERTO) {
        if (millis() - timerInicio >= delayAcerto) {
            digitalWrite(ledVerde, LOW);
            faseAtual++;
            if (faseAtual >= 5) {
                Serial.println("Sequência final! Repita os botões na ordem.");
                Serial.println(audios[6]);
                estado = REPETIR_SEQUENCIA_FINAL;
                repeticaoIndex = 0;
                aguardandoResposta = true;
            } else {
                estado = ESPERANDO_ENTRE_FASES;
                timerInicio = millis();
            }
        }
    }

    if (estado == PROCESSANDO_ERRO) {
        if (millis() - timerInicio >= delayErro) {
            digitalWrite(ledVermelha, LOW);
            estado = ESPERANDO_APERTO;
            aguardandoResposta = true;
        }
    }

    if (estado == ESPERANDO_ENTRE_FASES) {
        if (millis() - timerInicio >= delayEntreFases) {
            if (faseAtual < 5) {
                Serial.println(audios[ordemFases[faseAtual]]);
            }
            aguardandoResposta = true;
            estado = ESPERANDO_APERTO;
        }
    }
}

```
# SOFTWARE PYTHON
```python
import serial
import time
import pygame
import os

pygame.init()
arduino = serial.Serial('COM11', 9600, timeout=1)
time.sleep(2)  # Tempo para estabilizar a comunicação

while True:
    comando = arduino.readline().decode().strip()
    
    if comando:
        print("Comando recebido:", comando)

        if comando == "RESET":
            print("Jogo reiniciado! Aguardando START...")
        elif comando == "START":
            print("Jogo iniciado!")
        elif comando == "FIM_DO_JOGO":
            print("Parabéns! Você completou o desafio!")
        elif os.path.exists(comando):  # Verifica se o arquivo existe
            pygame.mixer.music.load(comando)
            pygame.mixer.music.play()
            
            while pygame.mixer.music.get_busy():  # Espera o áudio terminar antes de seguir
               time.sleep(0.1)
```
