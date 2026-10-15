# Inventário e plano personalizado

Atualizado em 15 out 2026. Os itens foram identificados **por fotos**; o grau de confiança de cada linha está indicado. Dados marcados como "consultado" vêm das páginas listadas em *Fontes*.

## Placas de desenvolvimento

| Item | Identificação | Confiança | Observações |
|---|---|---|---|
| Arduino UNO R3 nº 1 | ATmega328P-U (DIP), cristal de 16 MHz, USB-B, ATmega16U2 como ponte USB | alta | Lote do chip 2051 |
| Arduino UNO R3 nº 2 | ATmega328P-U (DIP) | alta | Lote do chip 1918 |
| Arduino UNO R3 SMD com CH340 | Listado na nota fiscal ("Uno R3 SMD CH340"). Ainda sem foto | média | **A confirmar** se você tem esta placa ou se é uma das duas acima. A ponte USB-serial CH340 pode exigir driver no Windows. |
| ESP32 | DevKit com módulo ESP-WROOM-32, USB-serial CP2102, regulador AMS1117-3.3, micro-USB, botões EN e BOOT | alta | Lógica de 3,3 V |
| ESP-01 | Módulo ESP8266 (AI-Thinker) com adaptador USB (CH340) | alta | Pads com solda acumulada e sem header visível; conferir curto antes de energizar |
| Shield Ethernet | RoboCore Ethernet W5500 (impresso na placa: "Shield Model Ethernet W5500", feito no Brasil) | alta | Ver *Shield Ethernet W5500* abaixo |

Não há UNO R4 nem placa com CAN.

## Aquisição de dados

| Item | Identificação | Confiança | Observações |
|---|---|---|---|
| NI USB-6001 | Etiqueta: M/N USB-6001, P/N 154424C-03L, "AI/O ±10 V; DI 0–5 V" | alta | Consultado: 8 entradas analógicas (até 4 diferenciais), 14 bits, 20 kS/s, ±10 V; 2 saídas analógicas de ±10 V, 16 bits, 5 kS/s por canal; 13 linhas digitais de 5 V; 1 contador; alimentado pela USB; exatidão absoluta da entrada analógica de **26 mV**; a página não lista isolação. Driver recomendado: NI-DAQmx. |

## Sensores e módulos

| Item | Identificação | Confiança | Observações |
|---|---|---|---|
| Conversor de termopar | MAX6675 (módulo HW-550, dois exemplares), confirmado pelo texto do chip | alta | SPI somente leitura; resolução 0,25 °C; conversão de até ~220 ms (de memória, a conferir) |
| Termopar | Tipo K, ponta roscada | alta | Nota fiscal: "Termopar Tipo K 0 a 800 °C + módulo de leitura MAX6675". É a faixa anunciada pelo vendedor: os terminais de crimpagem e o módulo MAX6675 não suportam essa temperatura. |
| DS18B20 à prova d'água | 4 sondas em tubo de aço inox, saída digital 1-Wire | alta | Consultado: de −55 a +125 °C, exatidão de ±0,5 °C entre −10 e +85 °C, resolução de 9 a 12 bits (12 bits: até 750 ms por conversão), alimentação de 3,0 a 5,5 V, pull-up externo de cerca de 5 kΩ (diagramas com 4,7 kΩ), código ROM de 64 bits único por sensor, vários no mesmo fio |
| Sensor de corrente (alicate) | SCT-013-000, 100 A : 50 mA (confirmado) | alta | Saída em corrente: exige resistor de carga e polarização em Vcc/2 |
| Módulo de corrente ACS712 | 1 módulo de **5 A** (confirmado por você) | alta | Sensibilidade típica de 185 mV/A (de memória, a conferir) |
| Outros dois módulos de corrente | Pela foto, provavelmente ACS712 | média | Variante (5, 20 ou 30 A) **a confirmar** pelo texto do chip |
| Acelerômetro analógico | Provavelmente MMA7361 (pinos X, Y, Z, SL, GS, ST, OG) | média | |
| Acelerômetro digital | ADXL345 | alta | I2C ou SPI |
| RTC | DS3231 em placa ZS-042, com CR2032 | alta | Ver cuidados |
| Temperatura e umidade | DHT22 (AM2302), pelo rótulo na foto | média | A nota fiscal lista um **DHT11**. Pode haver os dois: **a confirmar** |
| Distância | HC-SR04 | alta | |
| Umidade do solo | Sensor analógico Octopus | média | |
| Luz | LDR | média | |
| Módulo KY de 3 pinos com componente TO-92 | Hall ou DS18B20 | baixa | **A confirmar** |

## Interface, armazenamento e potência

- LCD 16×2 (HD44780), interface paralela, sem adaptador I2C visível.
- Módulo para cartão SD de tamanho grande e módulo micro-SD HW-125.
- Módulo de 2 relés 5 V (JQC-3FF, com optoacoplador) e módulo de 4 relés 5 V (SRD-05VDC; na foto aparece "10 A 30 VDC" impresso no relé).
- **4 resistores de potência de 330 Ω e 50 W.** Na potência nominal correspondem a cerca de 128 V e 0,39 A (√(50 × 330)). A potência de 50 W normalmente só vale com dissipador (confirmar na folha de dados do fabricante).
- Mini fonte chaveada AC-DC isolada, provavelmente de 5 V (especificação a conferir).
- Protoboards de 830, 400 e 170 pontos (a de 170 é a vermelha), tira de pinos macho 1×40, trimpot vertical de 5 kΩ (contraste do LCD) e adaptador de bateria de 9 V para Arduino, segundo a nota fiscal.
- Jumpers, 40 unidades de cada tipo, em 20 cm (macho/macho, fêmea/fêmea, macho/fêmea) e em 10 cm (os mesmos três tipos), segundo a nota fiscal.
- Caixa organizadora com resistores, trimpots, capacitores, reguladores, LEDs, buzzers, alto-falante, chaves e potenciômetros.

## Shield Ethernet W5500 (RoboCore)

Consultado na página do fabricante:

- Controlador Wiznet W5500, TCP e UDP, até 8 sockets e 32 KB de buffer de recepção e transmissão. Cada shield tem um MAC único, impresso na etiqueta atrás do conector RJ45.
- **Tem slot para microSD** (até 32 GB, cartão não incluído).
- Pinos no UNO R3: D4, D10, D11, D12 e D13. CS do W5500 no D10, CS do microSD no D4, INT no D3 e detecção de cartão no D2.
- W5500 e microSD **compartilham o barramento SPI**: só um pode estar ativo por vez.
- A RoboCore recomenda a biblioteca `Ethernet.h` do próprio Arduino em vez da biblioteca da RoboCore.
- Relatos na página: o shield não reconecta sozinho após queda de rede com a biblioteca nativa (há contorno em código), e um reset por watchdog do microcontrolador **não** reinicia o shield.

### Orçamento de pinos do UNO com o shield

| Pinos | Uso |
|---|---|
| D0, D1 | UART (USB/serial) |
| D2, D3 | Detecção de cartão e INT do shield |
| D4 | CS do microSD do shield |
| D10 | CS do W5500 |
| D11, D12, D13 | SPI (MOSI, MISO, SCK) |
| D5 a D9 | Livres (5 pinos) |
| A0 a A5 | Livres (A4 e A5 também são I2C) |

O LCD paralelo em 4 bits pede 6 linhas. Com o shield montado, ele disputa pinos com os relés, o CS do MAX6675 e o fio do DS18B20. Esse aperto é um dos motivos para estudar a migração para o ESP32.

## Instrumentos de bancada

- Multímetro Fluke 107.
- Osciloscópio portátil de 2 canais, canais não isolados entre si.
- Fonte regulável de 0 a 30 V, 10 A.
- NI USB-6001 (aquisição de dados, ver acima), para uso em computador com Windows 10.
- Ferro de solda e ferramentas de eletrônica.
- Não há analisador lógico. Um clone de 8 canais com PulseView é opcional a partir do Módulo 5.

## Cuidados registrados

- **RTC ZS-042:** a placa tem circuito de carga (diodo e resistor de cerca de 200 Ω) pensado para LIR2032. Com a CR2032, que não é recarregável, é um risco conhecido da comunidade. Não deixar o módulo ligado a 5 V com a CR2032 instalada até remover o caminho de carga. A confirmação em datasheet e esquemático ficou **pendente**.
- **Rede elétrica:** relés, a fonte AC-DC, o ACS712 e o alicate lidam com tensão de rede. Nos primeiros módulos só se usa baixa tensão. Para corrente alternada, preferir o alicate. Garra de terra do osciloscópio só no terra do circuito, nunca em condutor da rede.
- **Resistores de 330 Ω / 50 W:** em 220 V dissipariam cerca de 147 W, quase o triplo do nominal. Não ligar em tensão acima de ~128 V. Em operação ficam muito quentes: montar em dissipador e longe de material combustível.
- **SCT-013-000:** como qualquer transformador de corrente, o secundário não deve ficar aberto com corrente no primário.
- **NI USB-6001:** respeitar a faixa de entrada de ±10 V da etiqueta. Como a página não lista isolação, tratar o terra dele como ligado ao do computador.
- **Churrasqueira a carvão:** só ao ar livre, por causa do monóxido de carbono. O eletrônico fica longe do calor, da fumaça e da gordura. O termopar não encosta nas brasas (as temperaturas ali podem passar da faixa de 800 °C anunciada), e os terminais de crimpagem e o módulo MAX6675 ficam fora da zona quente.
- **Baterias de íon-lítio:** por ora, apenas medir tensão em vazio. Teste de descarga só em alcalinas, NiMH e CR2032, e sem curto-circuito.

## Objetivos do projeto

- Usar os sensores que já existem.
- Registrador de temperatura e/ou corrente.
- Monitorar o ciclo de aquecimento de uma churrasqueira **a carvão**.
- Testador de pilhas.

## Plano revisado em 15 out 2026

| Módulos | Entrega |
|---|---|
| 1 e 2 | Testador de pilhas v1: tensão em vazio no ADC, saída em serial e LCD. ADC calibrado contra o Fluke 107 e a fonte. Opcional: segunda opinião com o NI USB-6001. |
| 3 | Timer de amostragem e interrupções. Com os dois UNOs, um gera o pulso e o outro responde, e o osciloscópio mede a latência. Comparação com o ESP32. |
| 4 | MAX6675, DS18B20 (4 sondas), ACS712 de 5 A e SCT-013-000. Calibração do DS18B20 no ponto de gelo e em água fervente. O MAX6675 só lê a partir de 0 °C (de memória, a conferir), então o ponto de gelo fica na borda da faixa dele: conferi-lo em pontos mais quentes. Conferência cruzada de tensão com o Fluke e o NI USB-6001. O SCT-013-000 fica para uma carga CA doméstica, só depois das regras de segurança. |
| 5 | SPI, I2C e UART entre os dois UNOs e no registrador. Orçamento de pinos com o shield e SPI compartilhado (W5500, microSD, MAX6675). CAN fica opcional. |
| 7 | Máquina de estados do registrador, leitura não bloqueante do DS18B20 e do MAX6675, e watchdog. Bancada térmica de 30 V com relé e controle liga-desliga. |
| 6 | UNO R3 contra ESP32, CLI e VS Code. |
| 8 | WiFi no ESP32 e Ethernet W5500 no UNO, MQTT e dashboards. Comparação das duas rotas. |
| 9 | Robustez de campo (reconexão de rede, reset do shield), OTA no ESP32. ESP-01 como experimento extra. |
| 10 | Registrador do ciclo de aquecimento da churrasqueira a carvão (temperatura em vários pontos, com relógio, cartão SD e display), validado antes na bancada térmica. Opcional mais tarde: ventoinha controlada pela temperatura. |

### O que mudou com os itens novos

- **Dois UNOs** permitem experimentos de comunicação reais (um é o dispositivo, o outro é o analisador ou gerador), no lugar do CAN.
- **DS18B20** dá vários pontos de temperatura num só fio e um caso concreto de código não bloqueante (conversão de até 750 ms em 12 bits). O limite de +125 °C não cobre a câmara de uma churrasqueira a carvão: ali o termopar tipo K continua sendo o sensor principal.
- **Bancada térmica de 30 V:** quatro resistores de 330 Ω em paralelo (82,5 Ω) com 30 V dissipam cerca de 10,9 W (0,36 A). Cabe nos contatos de 10 A / 30 VDC do relé de 4 canais e serve para desenvolver o registrador sem mexer com a rede. Com esse nível de corrente, o ACS712 de 5 A dá um sinal pequeno (da ordem de 67 mV), o que ensina a escolher a faixa do sensor. O Fluke fica como referência de corrente.
- **NI USB-6001** entra como segunda e terceira opinião de medição. Exatidão de 26 mV (consultada) mostra que resolução e exatidão são coisas diferentes.
- **Churrasqueira a carvão (resposta de 15 out):** não há corrente elétrica a medir no equipamento. O registrador final mede temperatura, e a corrente fica na bancada térmica (ACS712 de 5 A e Fluke) e num experimento com carga CA. O SCT-013 deixa de ser peça central do projeto final.
- **Shield W5500** vira a rota cabeada do Módulo 8, e o aperto de pinos com ele vira exercício de projeto.

## Pendências

- Você tem a placa UNO R3 SMD com CH340 listada na nota fiscal? Foto, se tiver.
- Você tem um DHT11, um DHT22 ou os dois?
- Variante dos outros dois módulos de corrente e do módulo KY de 3 pinos.
- NI-DAQmx no Windows 10: conferir no *Readme* da versão se o Windows 10 consta nos sistemas suportados.
- Cores dos fios das sondas DS18B20 (conferir antes de ligar; costumam ser vermelho, preto e amarelo, mas varia).
- Verificação do circuito de carga do ZS-042 em fontes primárias.
- Respostas do diagnóstico de C/C++ do Módulo 1 (a combinar).

## Fontes

Consultadas em 14 e 15 out 2026:

- [NI USB-6001, página do produto](https://ni.com/en-us/shop/model/usb-6001.html)
- [DS18B20, folha de dados (Analog Devices)](https://www.analog.com/media/en/technical-documentation/data-sheets/DS18B20.pdf)
- [Shield Ethernet W5500, página da RoboCore](https://www.robocore.net/shields-arduino/arduino-shield-ethernet-w5500)
- [NI-DAQmx, página de download](https://www.ni.com/en/support/downloads/drivers/download.ni-daq-mx.html): versão mais recente listada em 15 out 2026 é a 2026 Q3; a página não informa as versões do Windows aceitas.
- Nota fiscal de compra (captura enviada por você em 15 out 2026).

O que não tem fonte listada (por exemplo, o MAX6675, o ACS712 e o DS3231) vem do meu conhecimento e será conferido em datasheets quando o módulo correspondente começar.
