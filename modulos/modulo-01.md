# Módulo 1: retomada e ADC com referência

Proposto em 13 out 2026 · Duração estimada: 2 a 3 sessões · Placa: Arduino UNO R3 (ATmega328P)

## Objetivos

Ao final você deve conseguir:

1. Ter o ambiente funcionando (Arduino IDE 2.3.10, UNO R3, monitor serial).
2. Explicar como o ADC do ATmega328P converte tensão em contagens e o que limita a exatidão.
3. Medir o erro de ganho e de offset do ADC contra uma referência, **declarando a incerteza dessa referência**.
4. Medir o tempo de conversão no osciloscópio e o ruído em LSB.
5. Ler uma pilha AA com o ADC (testador v0).

## Material

UNO R3 e cabo USB · Fluke 107 · fonte 0 a 30 V · osciloscópio · mini protoboard · 1 resistor de 1 kΩ · 2 resistores iguais de cerca de 10 kΩ · potenciômetro · capacitor de 100 nF · 1 pilha AA alcalina nova e 1 usada · jumpers · planilha (Google Sheets). Opcional (Experimento F): NI USB-6001 com o driver NI-DAQmx instalado.

## Parte 0: diagnóstico de C/C++

Responda no chat. Serve para eu ajustar o Módulo 2.

1. Quantos bytes tem um `int` no UNO R3 e no ESP32? O que acontece com `int mv = raw * 5000 / 1024;` no UNO?
2. Por que `if (millis() - t0 >= 1000)` é correto e `if (millis() >= t0 + 1000)` falha depois de cerca de 49,7 dias?
3. Quando uma variável precisa ser `volatile`? Dê um exemplo em microcontrolador.
4. O que faz `F("texto")` e por que isso importa com 2 kB de SRAM?
5. Qual a diferença prática entre `#define LIMITE 100` e `const uint8_t LIMITE = 100;`?
6. Por que `raw * (5 / 1024)` dá zero?

## Parte 1: teoria

### 1.1 O que o ADC faz

O ADC do ATmega328P é de **aproximações sucessivas, com 10 bits** e um multiplexador de 6 canais (A0 a A5 no UNO). A relação entre tensão e contagem, segundo o datasheet, é:

`ADC = Vin × 1024 / Vref`

Portanto `Vin = ADC × Vref / 1024`. Divide-se por **1024**, e não por 1023. O maior código (1023) corresponde a `Vref − 1 LSB`.

O tamanho de 1 LSB é `Vref / 1024`:

| Referência | Vref | 1 LSB |
|---|---|---|
| `DEFAULT` (AVcc, ligado ao Vcc da placa) | cerca de 5 V, varia com a USB | cerca de 4,88 mV |
| `INTERNAL` (bandgap) | cerca de 1,1 V nominal | cerca de 1,07 mV |
| `EXTERNAL` (pino AREF) | a tensão que você aplicar | `Vref / 1024` |

O ponto prático: com `DEFAULT`, a referência **é** o Vcc da placa. Se a USB estiver em 4,72 V e você supuser 5,00 V, o erro de leitura é de cerca de +5,9 %, e o ADC não tem como saber disso.

### 1.2 Tempo de conversão

O clock do ADC no Arduino é 16 MHz / 128 = 125 kHz, e uma conversão leva 13 ciclos de ADC: 13 × 8 µs = **104 µs**. Com o código em torno, `analogRead` leva pouco mais, na casa de 110 a 120 µs, e a documentação do Arduino fala em cerca de 10 000 leituras por segundo.

### 1.3 Fontes de erro

- **Referência:** o Vcc varia e o bandgap tem tolerância.
- **Offset e ganho:** desvios lineares, corrigíveis por calibração.
- **Não linearidade (INL):** curva que não é reta. Não corrigível com dois pontos.
- **Ruído:** flutua em torno de 1 LSB ou mais.
- **Impedância da fonte:** o datasheet recomenda fonte de até cerca de 10 kΩ. Acima disso o capacitor de amostragem não carrega.
- **Troca de canal:** a primeira leitura após trocar de canal pode vir contaminada pelo canal anterior. Por isso se descarta a primeira.

Valores de exatidão (por exemplo, "±2 LSB") devem ser conferidos na tabela de características do ADC no datasheet do ATmega328P. Eles não foram verificados nesta sessão.

### 1.4 Incerteza da referência

Calibrar um instrumento contra outro só vale se o de referência for **bem melhor** que o calibrado (regra comum: pelo menos 4 vezes). A exatidão de um multímetro de mão é especificada como `±(x % da leitura + n contagens)`. Procure a tabela de exatidão em tensão CC no manual do seu Fluke 107.

Faça a conta para 5 V. Se a incerteza do Fluke ficar na ordem do LSB do ADC ou acima, a conclusão correta é que você só consegue **separar erros maiores que essa incerteza**: o erro de Vcc, o ganho e o offset grosseiros. O resto vira ruído de medição.

## Parte 2: preparação

1. IDE 2.3.10 → *Tools → Board → Arduino AVR Boards → Arduino Uno*, escolher a porta e abrir o monitor serial em **115200**.
2. Fonte: antes de ligar qualquer coisa, ajuste a tensão em 0 V e o **limite de corrente em cerca de 20 mA** (curto nos terminais com os cabos, gire o limite, retire o curto).
3. Nunca aplique em A0 mais que Vcc + 0,5 V nem menos que −0,5 V. O resistor de 1 kΩ em série com o pino limita a corrente se algo sair do previsto.
4. O terra da fonte (negativo) deve ficar ligado ao GND do UNO.

## Experimento A: Vcc real a partir do bandgap

Mede o bandgap interno (cerca de 1,1 V nominal) usando AVcc como referência. Como a conta é `ADC = Vbg × 1024 / Vcc`, resulta `Vcc = Vbg × 1024 / ADC`.

```cpp
// Experimento A: Vcc estimado pelo bandgap interno
// K = 1,100 V x 1024 x 1000. Ajustar na calibração.
const long BANDGAP_K = 1126400L;

uint16_t readBandgapRaw() {
  ADMUX = _BV(REFS0) | _BV(MUX3) | _BV(MUX2) | _BV(MUX1); // AVcc como ref, canal 1,1 V
  delay(2);                                               // o bandgap precisa estabilizar
  uint32_t acc = 0;
  for (uint8_t i = 0; i < 16; i++) {
    ADCSRA |= _BV(ADSC);
    while (bit_is_set(ADCSRA, ADSC));
    acc += ADC;
  }
  return acc / 16;
}

void setup() { Serial.begin(115200); }

void loop() {
  uint16_t raw = readBandgapRaw();
  long vcc_mV = BANDGAP_K / raw;
  Serial.print(F("raw_bandgap="));
  Serial.print(raw);
  Serial.print(F("  Vcc_estimado(mV)="));
  Serial.println(vcc_mV);
  delay(1000);
}
```

Procedimento:

1. Carregue o sketch e anote `raw_bandgap` (use um valor estável).
2. Com o Fluke, meça a tensão entre o pino 5V e o GND da placa. Anote em mV.
3. Calibre: `K_novo = Vcc_Fluke(mV) × raw_bandgap`. Troque `BANDGAP_K` por esse valor.
4. Conecte o UNO a outra porta USB (ou a um hub) e compare a variação do Vcc.

**Pergunta:** o Vcc estimado antes e depois da calibração concorda com o Fluke dentro da incerteza dele?

## Experimento B: curva de transferência com a fonte

```cpp
// Leitura média de A0 (AVcc como referência), contagens brutas
uint16_t readAvg(uint8_t pin, uint8_t n) {
  analogRead(pin);                       // descarta a primeira leitura
  uint32_t acc = 0;
  for (uint8_t i = 0; i < n; i++) acc += analogRead(pin);
  return (acc + n / 2) / n;
}

void setup() { Serial.begin(115200); }

void loop() {
  Serial.println(readAvg(A0, 64));
  delay(500);
}
```

Ligação: fonte (+) → resistor de 1 kΩ → A0; fonte (−) → GND. O Fluke fica entre A0 e GND, na faixa que der mais dígitos.

Pontos de tensão: 0,00 · 0,25 · 0,50 · 1,00 · 1,50 · 2,00 · 2,50 · 3,00 · 3,50 · 4,00 · 4,50 V. Não suba acima de uns 4,6 V.

Para cada ponto, anote na planilha: V_Fluke (mV) e a contagem média lida no serial.

Na planilha:

1. Faça o gráfico V_Fluke × contagem e ajuste uma reta: `=SLOPE(V_Fluke; contagem)` e `=INTERCEPT(V_Fluke; contagem)`.
2. O ideal é inclinação = Vcc/1024 (cerca de 4,88 mV por contagem) e intercepto próximo de zero.
3. **Vref efetivo** = inclinação × 1024. Compare com o Vcc do Experimento A e com o Fluke no pino 5V.
4. Calcule o erro de cada ponto em mV e em LSB, e verifique se existe curvatura (INL).
5. Compare cada erro com a incerteza do Fluke da seção 1.4.

## Experimento C: ruído e média

Aplique em A0 uma tensão estável (por exemplo, o cursor do potenciômetro ligado a 5 V e GND, ou um divisor de dois resistores iguais, cerca de 2,5 V).

```cpp
const uint16_t N = 1000;

void setup() {
  Serial.begin(115200);
  analogRead(A0);                        // descarta a primeira leitura
  Serial.println(F("n,raw"));
  for (uint16_t i = 0; i < N; i++) {
    Serial.print(i);
    Serial.print(',');
    Serial.println(analogRead(A0));
    delay(10);
  }
  Serial.println(F("fim"));
}

void loop() {}
```

1. Copie as 1000 linhas para a planilha. Calcule média, `STDEV` e a amplitude (máximo menos mínimo), tudo em LSB.
2. Repita com um capacitor de 100 nF entre A0 e GND. Mudou?
3. Repita trocando `analogRead(A0)` por `readAvg(A0, 16)`. Se o ruído for branco, o desvio-padrão deve cair cerca de 4 vezes (raiz de 16). Confira se foi isso.
4. No osciloscópio, com acoplamento CC, veja o nó A0. O ruído do próprio osciloscópio portátil pode ser maior que o do ADC. Anote o que você consegue enxergar.

## Experimento D: tempo de conversão no osciloscópio

```cpp
void setup() { DDRB |= _BV(DDB0); }       // pino digital 8 como saída

void loop() {
  PORTB |= _BV(PORTB0);                   // pino 8 em alto
  (void)analogRead(A0);
  PORTB &= ~_BV(PORTB0);                  // pino 8 em baixo
  delay(1);
}
```

1. Ligue a ponta do osciloscópio no pino 8 e a garra no GND.
2. Meça a largura do pulso em alto. Esperado na faixa de 110 a 120 µs: 104 µs de conversão mais o código da biblioteca.
3. Compare com a teoria da seção 1.2. Se der diferente, descubra por quê.

## Experimento E: testador v0 (pilha AA)

Use este sketch **depois** da calibração do Experimento A. Ele combina as duas funções.

```cpp
const long BANDGAP_K = 1126400L;          // trocar pelo valor calibrado

uint16_t readBandgapRaw() {
  ADMUX = _BV(REFS0) | _BV(MUX3) | _BV(MUX2) | _BV(MUX1);
  delay(2);
  uint32_t acc = 0;
  for (uint8_t i = 0; i < 16; i++) {
    ADCSRA |= _BV(ADSC);
    while (bit_is_set(ADCSRA, ADSC));
    acc += ADC;
  }
  return acc / 16;
}

uint16_t readAvg(uint8_t pin, uint8_t n) {
  analogRead(pin);
  uint32_t acc = 0;
  for (uint8_t i = 0; i < n; i++) acc += analogRead(pin);
  return (acc + n / 2) / n;
}

void setup() { Serial.begin(115200); }

void loop() {
  long vcc_mV = BANDGAP_K / readBandgapRaw();
  uint16_t raw = readAvg(A0, 64);
  long v_mV = (long)raw * vcc_mV / 1024;  // long evita estouro de 16 bits
  Serial.print(F("Vpilha(mV)="));
  Serial.println(v_mV);
  delay(500);
}
```

Cuidados:

1. Antes de conectar a pilha ao UNO, **confira a polaridade com o Fluke**.
2. Pilha AA entre A0 (+) e GND (−). Só alcalina, NiMH ou CR2032. Nada de íon-lítio.
3. Compare a leitura do UNO com a do Fluke para uma pilha nova e uma usada.
4. A tensão em vazio de uma alcalina é um indicador ruim da carga que resta, porque cai pouco até o fim. Por isso o Módulo 2 vai medir sob carga.

## Experimento F (opcional): segunda opinião com o NI USB-6001

O USB-6001 tem 8 entradas analógicas de 14 bits, até 20 kS/s e faixa de ±10 V. Pela página do produto na NI (consultada em 14 out 2026), a **exatidão absoluta da entrada analógica é de 26 mV**. O LSB dele em ±10 V é de cerca de 1,2 mV (20 V / 2^14), muito mais fino que a exatidão. **Resolução não é exatidão.**

1. Instale o driver NI-DAQmx (baixado no site da NI) no computador e conecte o USB-6001. Seu computador é Windows 10, que saiu do suporte da Microsoft em 14 out 2025. A página de download (consultada em 15 out 2026) dá a 2026 Q3 como a versão mais recente, mas não informa quais versões do Windows ela aceita. Abra o *Readme* da versão antes de instalar e confira se o Windows 10 consta. A mesma página traz um aviso para quem usa versões 2026 Q2 ou anteriores (ni.com/ni-pal-update); leia-o.
2. Abra o NI MAX, localize o dispositivo e use o painel de teste da entrada analógica (*Test Panels → Analog Input*) para ler a tensão em `ai0`.
3. Repita os pontos de 0 a 4,5 V do Experimento B, lendo o mesmo ponto com o Fluke, com o ADC do UNO e com o USB-6001. Ligue o GND do USB-6001 ao GND do circuito e use só tensões dentro de ±10 V (neste experimento, no máximo 4,6 V). Confira a impedância de entrada do USB-6001 na folha de dados, para saber se ela altera o ponto medido.
4. Na planilha, coloque as três leituras lado a lado e calcule as diferenças duas a duas.

O UNO e o USB-6001 se alimentam pela USB do mesmo computador, e a página não lista isolação no USB-6001. Trate os terras como ligados.

**Pergunta:** as diferenças ficam dentro das incertezas somadas? Se um aparelho discorda dos outros dois além do esperado, qual deles você suspeita primeiro e por quê?

## Entrega

Uma planilha com:

1. A tabela do Experimento B, o gráfico, a reta ajustada e o Vref efetivo.
2. Os resultados do Experimento C (média, desvio-padrão, amplitude, com e sem capacitor, com e sem média).
3. A largura do pulso do Experimento D.
4. A tabela do Experimento E (pilha nova e usada, UNO e Fluke).
5. Um parágrafo dizendo o quanto você confia no ADC do seu UNO e por quê, incluindo a incerteza do Fluke.
6. (Opcional) A tabela do Experimento F, com as três leituras e as diferenças.

## Avaliação

1. Calcule o LSB para Vref = 5,00 V e para Vref = 1,1 V.
2. Se o Vcc real é 4,72 V e o código supõe 5,00 V, qual o erro percentual na tensão calculada?
3. Por que o Fluke 107 sozinho não valida o ADC a ±1 LSB?
4. Por que descartar a primeira leitura após trocar de canal?
5. Qual o tempo esperado de uma conversão no Arduino e de onde ele vem?
6. Qual a diferença entre resolução e exatidão? Use o USB-6001 (14 bits, exatidão de 26 mV) como exemplo.

## Resumo de continuidade

- **Hardware:** dois UNO R3 (ATmega328P), ESP32, Fluke 107, fonte 0 a 30 V, osciloscópio de 2 canais não isolados, NI USB-6001, shield Ethernet W5500 (RoboCore), 4 sondas DS18B20, 4 resistores de 330 Ω / 50 W. Inventário completo em `inventario.md`.
- **Decisões:** projeto-fio-condutor (testador de pilhas → registrador T e I → registrador da churrasqueira). Íon-lítio só com medida de tensão em vazio.
- **Pendências do inventário:** tipo da churrasqueira, limite do termopar, variante dos módulos de corrente, ZS-042.
- **Próximo:** Módulo 2, com C++ aplicado e o testador sob carga, ajustado pelo diagnóstico da Parte 0.

## Fontes e ressalvas

- Documentação do Arduino (`analogRead`, `analogReference`) em docs.arduino.cc.
- Datasheet do ATmega328P (Microchip), capítulo do conversor analógico-digital.
- Manual do Fluke 107, tabela de exatidão.
- [Página do produto NI USB-6001](https://ni.com/en-us/shop/model/usb-6001.html), consultada em 14 out 2026.
- [NI-DAQmx, página de download](https://www.ni.com/en/support/downloads/drivers/download.ni-daq-mx.html), consultada em 15 out 2026.

Dos documentos acima, só a página do NI USB-6001 foi consultada diretamente (14 out 2026); os demais não. Os valores acima (clock do ADC, 13 ciclos, recomendação de impedância, bandgap nominal, tempo de conversão) vêm do meu conhecimento e devem ser conferidos nas fontes antes de você tratá-los como definitivos. Os sketches não foram compilados nem testados em hardware.
