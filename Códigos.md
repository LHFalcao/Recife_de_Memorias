# SOFTWARE ARDUÍNO
```cpp

const int btnStart    = 22; 
const int btnReset    = 24;

const int btnAzul     = 40;
const int btnRosa     = 48;
const int btnLaranja  = 44;
const int btnAmarelo  = 52;
const int btnVerde    = 32;

const int ledVerde    = 9;
const int ledVermelha = 6;

const char* audios[5] = {
    "parquedasesculturas.wav", 
    "marcozero.wav", 
    "pacodofrevo.wav", 
    "caisdosertao.wav", 
    "ruadobomjesus.wav"
};

bool jogoAtivo = false;
int faseAtual = 0;
bool aguardandoResposta = false;

unsigned long tempoUltimoPressionado[10] = {0};
const int debounceDelay = 200; // Aumentado para 200ms para melhorar debounce

// Guardamos o estado anterior de cada botão para debounce
bool estadoAnterior[10] = {HIGH, HIGH, HIGH, HIGH, HIGH, HIGH, HIGH, HIGH, HIGH, HIGH};

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
        Serial.println("START");
        delay(1000);
        Serial.println(audios[faseAtual]);
    }

    if (jogoAtivo && aguardandoResposta) {
        if (estado == ESPERANDO_APERTO) {

            if (estado == REPETIR_SEQUENCIA_FINAL) {
                // Isso nunca ocorre aqui (condição de segurança)
            }

            if (estado != REPETIR_SEQUENCIA_FINAL) {
                // Jogo normal, avançando fases
                if (faseAtual < 5) {
                    // Verifica botão correto atual
                    if (botaoPressionado(botoesCorretos[faseAtual], botoesIndices[faseAtual])) {
                        estado = PROCESSANDO_ACERTO;
                        timerInicio = millis();
                        aguardandoResposta = false;

                        digitalWrite(ledVerde, HIGH);
                        digitalWrite(ledVermelha, LOW);
                        Serial.println("Acerto!");
                    }
                    // Botão errado
                    else if (
                        (botaoPressionado(btnAzul, 0) && botoesCorretos[faseAtual] != btnAzul) ||
                        (botaoPressionado(btnRosa, 1) && botoesCorretos[faseAtual] != btnRosa) ||
                        (botaoPressionado(btnLaranja, 2) && botoesCorretos[faseAtual] != btnLaranja) ||
                        (botaoPressionado(btnAmarelo, 3) && botoesCorretos[faseAtual] != btnAmarelo) ||
                        (botaoPressionado(btnVerde, 4) && botoesCorretos[faseAtual] != btnVerde)
                    ) {
                        estado = PROCESSANDO_ERRO;
                        timerInicio = millis();

                        digitalWrite(ledVerde, LOW);
                        digitalWrite(ledVermelha, HIGH);
                        Serial.println("ERRO - Repetindo áudio...");
                        if (faseAtual < 5) {
                            Serial.println(audios[faseAtual]);
                        }
                        aguardandoResposta = false;
                    }
                }
            }
        }
        else if (estado == REPETIR_SEQUENCIA_FINAL) {
            // Sequencia final, usuário deve repetir toda sequência

            // Captura qual botão foi pressionado, considerando a ordem esperada (botoesCorretos[repeticaoIndex])

            // Verifica se botão correto na repetição
            if (botaoPressionado(botoesCorretos[repeticaoIndex], botoesIndices[repeticaoIndex])) {
                repeticaoIndex++;
                digitalWrite(ledVerde, HIGH);
                digitalWrite(ledVermelha, LOW);
                Serial.print("Botao correto na sequencia final: ");
                Serial.println(repeticaoIndex);

                delay(200); // pequeno delay para o usuário notar

                digitalWrite(ledVerde, LOW);

                // Se terminou toda a sequência correta
                if (repeticaoIndex >= 5) {
                    Serial.println("PARABÉNS! Você ganhou o jogo.");
                    jogoAtivo = false;
                    aguardandoResposta = false;
                    estado = ESPERANDO_APERTO;
                    repeticaoIndex = 0;
                }
            }
            // Se outro botão foi pressionado (que não é o correto da sequência no índice)
            else if (
                (botaoPressionado(btnAzul, 0) && botoesCorretos[repeticaoIndex] != btnAzul) ||
                (botaoPressionado(btnRosa, 1) && botoesCorretos[repeticaoIndex] != btnRosa) ||
                (botaoPressionado(btnLaranja, 2) && botoesCorretos[repeticaoIndex] != btnLaranja) ||
                (botaoPressionado(btnAmarelo, 3) && botoesCorretos[repeticaoIndex] != btnAmarelo) ||
                (botaoPressionado(btnVerde, 4) && botoesCorretos[repeticaoIndex] != btnVerde)
            ) {
                digitalWrite(ledVerde, LOW);
                digitalWrite(ledVermelha, HIGH);
                Serial.println("ERRO na repetição da sequência final - Repita novamente!");
                delay(3000);
                digitalWrite(ledVermelha, LOW);
                repeticaoIndex = 0;
            }
        }
    }

    // Estado para aguardar o delay após acerto
    if (estado == PROCESSANDO_ACERTO) {
        if (millis() - timerInicio >= delayAcerto) {
            digitalWrite(ledVerde, LOW);
            faseAtual++;
            if (faseAtual >= 5) {
                // Chegou ao fim das fases, começa a repetição final
                Serial.println("Sequência final! Repita os botões na ordem.");

                estado = REPETIR_SEQUENCIA_FINAL;
                repeticaoIndex = 0;
                aguardandoResposta = true;
            } else {
                estado = ESPERANDO_ENTRE_FASES;
                timerInicio = millis();
            }
        }
    }

    // Estado para aguardar o delay após erro
    if (estado == PROCESSANDO_ERRO) {
        if (millis() - timerInicio >= delayErro) {
            digitalWrite(ledVermelha, LOW);
            estado = ESPERANDO_APERTO;
            aguardandoResposta = true;
        }
    }

    // Estado para aguardar um pequeno delay antes de mostrar o próximo áudio
    if (estado == ESPERANDO_ENTRE_FASES) {
        if (millis() - timerInicio >= delayEntreFases) {
            if (faseAtual < 5) {
                Serial.println(audios[faseAtual]);
            }
            aguardandoResposta = true;
            estado = ESPERANDO_APERTO;
        }
    }
}
```
# SOFTWARE PYTHON
```
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
