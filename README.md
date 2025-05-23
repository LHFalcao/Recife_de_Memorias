# Recife de Memórias

Recife de Memórias é um jogo sensorial em que o jogador vive uma “viagem” interativa pelos cinco principais pontos turísticos do bairro do Recife Antigo. A cada rodada, o sistema reproduz um trecho de áudio que evoca um local histórico (Cais do Sertão, Marco Zero, Paço do Frevo, Parque das Esculturas e Rua do Bom Jesus). Após ouvir o som, o participante deve pressionar o botão cuja cor corresponde ao ponto turístico representado. Se acertar, um LED verde acende; se errar, acende o vermelho. O jogo termina quando todos os cinco locais são identificados corretamente e o jogador repete a sequência dos botões pressionados, completando esse percurso cultural que estimula a memória afetiva e o conhecimento sobre a paisagem recifense.

---
## Requisitos

### Hardware
- Arduino Mega
- 7 botões físicos
  - 5 para respostas (cores)
  - 1 botão de início (Start)
  - 1 botão de reinício (Reset)
- 2 LEDs (verde e vermelho)
- Resistores (~300Ω)

### Software
- Python 3.x
- Bibliotecas:
  - `pyserial`
  - `pygame`
- 5 arquivos `.wav` (áudios dos pontos turísticos)

---

## Instalação

1. Clone o repositório:
```
   git clone https://github.com/seu-usuario/projeto-recife.git
   cd projeto-recife
````

2. Instale as dependências:
```
   pip install pyserial pygame
```

3. Envie o código `arduino.ino` para o seu Arduino Mega via Arduino IDE.

4. Coloque os arquivos `.wav` na pasta `/audios/` com os seguintes nomes:

   * `caisdosertao.wav`
   * `marcozero.wav`
   * `pacodofrevo.wav`
   * `parquedasesculturas.wav`
   * `ruadobomjesus.wav`

---

## Como Jogar

1. Conecte o Arduino via USB.
2. Execute o jogo:

   ```
   python jogo.py
   ```
3. Pressione o botão **Start** para iniciar.
4. Escute o áudio.
5. Pressione o botão correspondente ao ponto turístico.
6. O LED indicará se você acertou ou errou.
7. Após 5 rodadas, o jogo reinicia. Você também pode apertar o botão **Reset** a qualquer momento.

---

## Estrutura de Arquivos

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

---

## Créditos

Desenvolvido por Ana Lahalle, Davi Marcelo Pedrosa, Gabrielle Vital, Gustavo Torres, Matheus Cavalcanti, Marina Cabral, Luiz Henrique Falcão, João Guilherme Barros, João Pedro Pessoa e Yasmin Correia.

Projeto acadêmico interdisciplinar — tecnologia, memória e cultura.

---

## Licença
Este projeto está sob a licença MIT. Veja o arquivo LICENSE para mais detalhes.
