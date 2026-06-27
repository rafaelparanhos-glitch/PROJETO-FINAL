# AgroIoT — Firmware ESP32 DevKit v1 (5 Cloud Variables)

Versão com apenas **5 Cloud Variables** sincronizadas com o Arduino IoT Cloud — as demais (vazão, total de litros, alerta de desperdício, limites de umidade) continuam funcionando no firmware, mas só localmente (não aparecem no dashboard).

## 1. As 5 Cloud Variables

| Nome               | Tipo  | Permissão     | O que é |
|--------------------|-------|---------------|---------|
| `umidadeSolo`      | float | Read Only     | % de umidade do solo (sensor capacitivo/resistivo) |
| `temperatura`      | float | Read Only     | Temperatura do ar (DHT22) |
| `umidadeAr`        | float | Read Only     | Umidade relativa do ar (DHT22) |
| `bombaLigada`      | bool  | Read & Write  | Estado da bomba — você pode ligar/desligar manualmente pelo dashboard (se `MODO_AUTOMATICO = false` no código) |
| `alertaDesperdicio`| bool  | Read Only     | `true` quando a bomba está ligada mas o sensor de fluxo não detecta vazão real |

## 2. O que NÃO vai mais para a nuvem

Essas variáveis continuam existindo e funcionando no código, mas agora são **locais** (só aparecem no Serial Monitor, não no dashboard):

- Vazão de água (`vazaoLPM_local`)
- Total de litros acumulado (`totalLitros_local`)
- Modo automático/manual — agora é uma **constante fixa** no código (`MODO_AUTOMATICO = true`). Se quiser operar em modo manual (só via `bombaLigada` pelo dashboard), mude para `false` e grave de novo.
- Limites de umidade — também **constantes fixas** (`LIMITE_UMIDADE_MIN = 30.0`, `LIMITE_UMIDADE_MAX = 70.0`), não dá mais pra ajustar pelo dashboard. Se quiser mudar esses valores, edite direto essas linhas no `.ino` e grave de novo.

## 3. Criando o Thing no Arduino IoT Cloud

1. Acesse [cloud.arduino.cc](https://cloud.arduino.cc) → **Things** → **Create Thing**.
2. Associe seu ESP32 DevKit v1 (gera `Device ID` e `Secret Key`).
3. Em **Cloud Variables**, crie exatamente as 5 variáveis da tabela acima (nome e tipo precisam bater com o código).
4. Copie o `Device ID` e a `Secret Key` para as constantes `DEVICE_LOGIN_NAME` e `DEVICE_KEY` no topo do `.ino`.

## 4. Bibliotecas necessárias (Library Manager)

- **DHT sensor library** (Adafruit)
- **Adafruit Unified Sensor**
- **ArduinoIoTCloud**
- **Arduino_ConnectionHandler**

Placa: `ESP32 Dev Module`.

## 5. Ligações (sem mudanças)

| Componente              | Pino ESP32 | Observação |
|---------------------------|-----------|------------|
| Sensor umidade do solo     | GPIO 34   | Entrada analógica (ADC1, *input only*) |
| DHT22 (dado)               | GPIO 4    | Digital |
| YF-S201 (sinal)            | GPIO 27   | Interrupção, `INPUT_PULLUP` (uso local) |
| Relé — Mini bomba 12V      | GPIO 26   | Saída digital |

## 6. Ajustes antes de gravar

No topo do `.ino`:
```cpp
const char SSID[] = "NOME_DA_SUA_REDE";
const char PASS[] = "SENHA_DA_SUA_REDE";
const char DEVICE_LOGIN_NAME[] = "SEU_DEVICE_ID_AQUI";
const char DEVICE_KEY[]        = "SUA_DEVICE_KEY_AQUI";
```

Se quiser mudar o modo de operação ou os limites de irrigação automática:
```cpp
const bool  MODO_AUTOMATICO    = true;  // false = só controle manual via bombaLigada
const float LIMITE_UMIDADE_MIN = 30.0;  // % - liga a bomba abaixo disso
const float LIMITE_UMIDADE_MAX = 70.0;  // % - desliga a bomba acima disso
```
