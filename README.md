# Super Base Tensor 1D: Multimodal Optical Fiber Aging Dataset (ITU-T G.652)

![Status: Ativo](https://img.shields.io/badge/Status-Complete-success)
![Norma: ITU--T G.652](https://img.shields.io/badge/Norma-ITU--T_G.652-blue)
![Aplicação: PINNs / Time Series](https://img.shields.io/badge/Application-PINNs_%7C_Autoformer_%7C_AD-purple)
![Licença: MIT](https://img.shields.io/badge/License-MIT-green)

## 📌 Visão Geral
Este repositório disponibiliza a **Super Base Tensor 1D**, um conjunto de dados multimodal e padronizado construído para o desenvolvimento de modelos de Aprendizado de Máquina (ML) e Aprendizado Profundo Informado pela Física (*Physics-Informed Neural Networks* - PINNs) voltados ao estudo de envelhecimento, estimativa de Vida Útil Restante (RUL) e Detecção de Anomalias (AD) em redes ópticas monomodo sob a norma **ITU-T G.652** (125 µm de diâmetro de casca de sílica).

O avanço na manutenção preditiva de infraestruturas de telecomunicações enfrenta um gargalo crítico na literatura: a inexistência de bases abertas que unifiquem simulações físicas de laboratório, condições climáticas externas e *logs* operacionais dinâmicos de rede. Este dataset resolve esta fragmentação integrando quatro vertentes analíticas em uma única régua de tempo contínua ($X \in \mathbb{R}^{T \times F}$):
1. **Telemetria Operacional e Lógica de Rede** (*Soft Failures*).
2. **Refletometria Óptica no Domínio do Tempo** (*OTDR Traces*).
3. **Estressores Físico-Ambientais** (Séries Termohigrométricas).
4. **Mecânica da Fratura e Fadiga de Sílica** (Ensaios em transdutores FBG).

---

## 🛠️ Engenharia de Dados e Metodologia de Fusão
O dataset consolidado é composto por **2.970 registros temporais contínuos e 52 atributos estruturados**, gerados através de um pipeline rigoroso de Extração, Transformação e Carga (ETL) e Acoplamento de Domínios (*Domain Coupling*):

* **Ancoragem Temporal (ISO 8601):** Superação das lacunas de indexação relacional mediante a extração de *timestamps* limpos e normalizados a partir das séries de rastreio óptico, estabelecendo um vetor cronológico universal.
* **Acoplamento Analítico Aeroelástico:** Para integrar ensaios de fadiga mecânica estáticos de bancada às medições dinâmicas do enlace, adotou-se a velocidade do vento (`Wind_Speed_kmh`) como força motriz *proxy* para estimar a deformação mecânica transitória (`Strain_Provisorio`). A junção foi executada via **Junção Relacional por Proximidade (`pd.merge_asof` com direção `nearest`)**, mapeando exatamente o estresse em Pascal (`Stress_Pa`) e as taxas de degradação estrutural correspondentes ao estresse ambiental exercido.
* **Preservação do Gradiente Físico:** A premissa de preenchimento por zeros ou médias simples foi rejeitada para evitar saltos artificiais nas derivadas parciais durante o treinamento de PINNs. Aplicaram-se técnicas de **Interpolação Linear Assimétrica** (para os passos de tensão de bancada) e **Propagação de Estado Estável (*Forward/Backward Fill*)**, assumindo que o filamento óptico mantém seu último estado termodinâmico e estrutural até a detecção de uma nova variação pelos sensores.

---

## 📊 Estrutura e Dicionário de Dados
As colunas do dataset compõem quatro blocos dimensionais de *features*:

### 1. Bloco Temporal e Específico
| Atributo | Tipo | Descrição |
| :--- | :--- | :--- |
| `Timestamp` | `DATETIME` | Carimbo de tempo padronizado (ISO 8601) - Âncora temporal do tensor. |
| `Measurement_ID` | `INT` | Identificador único da leitura de rastreio óptico do OTDR. |
| `LP length (km)` | `FLOAT` | Extensão linear total do enlace de fibra analisado (514 km). |

### 2. Bloco de Estressores Físico-Ambientais (Causas do Desgaste)
| Atributo | Tipo | Descrição |
| :--- | :--- | :--- |
| `Temperature_C` | `FLOAT` | Temperatura ambiente externa (°C) - Parâmetro para equações de Arrhenius. |
| `Relative_Humidity_pct`| `FLOAT` | Umidade relativa do ar (%), influenciando na difusão de hidrogênio e corrosão sob tensão. |
| `Wind_Speed_kmh` | `FLOAT` | Velocidade constante do vento (km/h) na rota do cabo de fibra óptica. |
| `Wind_Gusts_kmh` | `FLOAT` | Rajadas de vento de pico (km/h) geradoras de vibração aeroelástica. |
| `Strain_Micro` | `FLOAT` | Microdeformação longitudinal da fibra ($\mu\epsilon$) mapeada em bancada. |
| `Stress_Pa` | `FLOAT` | Tensão mecânica aplicada em Pascals (Pa), parametrizada para distribuições de Weibull. |
| `Rate_FullCoverage` | `FLOAT` | Taxa normalizada de integridade do encapsulamento protetor do filamento. |

### 3. Bloco de Telemetria Lógica e Refletometria Óptica (Sintomas)
| Atributo | Tipo | Descrição |
| :--- | :--- | :--- |
| `OSNR (dB)` | `FLOAT` | Relação Sinal-Ruído Óptica (*Optical Signal-to-Noise Ratio*) no receptor. |
| `BER (dB)` | `FLOAT` | Taxa de Erro de Bit pré-correção de erro (*Pre-FEC Bit Error Rate*). |
| `Laser current (mA)` | `FLOAT` | Corrente operacional do diodo emissor laser. |
| `Loss_dB` | `FLOAT` | Atenuação e perda de potência registrada em todo o tramo medido. |
| `P1` ... `P30` | `FLOAT` | Vetor de 30 pontos espaciais de potência/refletância capturados ao longo da fibra pelo OTDR. |
| `Position_m` | `FLOAT` | Posição longitudinal exacta do evento ou anomalia no cabo de fibra (em metros). |

### 4. Bloco Alvo (Targets para Machine Learning Supervisionado)
| Atributo | Tipo | Descrição |
| :--- | :--- | :--- |
| `Class` / `Failure type` | `FLOAT` / `INT` | Identificador categórico da severidade e tipo de degradação (*Soft Failure*). |

---

## 🔬 Aplicações Em Inteligência Computacional
A uniformidade e completude desta base tornam-na ideal para:
* **Física Informada por Dados (PINNs):** Acoplamento direto das variáveis térmicas (`Temperature_C`) à **Equação de Arrhenius** e da fadiga mecânica (`Stress_Pa`) à **Distribuição de Weibull**, restringindo a função de perda (*loss function*) dos algoritmos às leis fundamentais da conservação física.
* **Séries Temporais Profundas (*Deep Time Series*):** Alimentação de arquiteturas **Autoformer (Transformers com autocorrelação)** e **Bi-LSTM** para extração de dependências bidirecionais de longo alcance e predição de horizonte estendido do RUL.
* **Detecção de Anomalias (AD):** Treinamento de **Autoencoders** e **Isolation Forests** sob as leituras multivariadas de telemetria e OTDR para identificação proativa de falhas incipientes (*soft failures*) antes do rompimento da fibra.

---

## 📥 Como Usar (Python / Pandas)
Para carregar o dataset em seu ambiente de pesquisa, utilize o trecho de código abaixo garantindo o reconhecimento correto da indexação cronológica:

```python
import pandas as pd

# Carregar a Super Base Tensor 1D completa
url = "[https://raw.githubusercontent.com/SEU-USUARIO/SEU-REPOSITORIO/main/Super_Base_Tensor_1D_COMPLETA.csv](https://raw.githubusercontent.com/SEU-USUARIO/SEU-REPOSITORIO/main/Super_Base_Tensor_1D_COMPLETA.csv)"
df_tensor = pd.read_csv(url)

# Converter a coluna Timestamp para o formato de data/hora nativo do Python
df_tensor['Timestamp'] = pd.to_datetime(df_tensor['Timestamp'])
df_tensor.set_index('Timestamp', inplace=True)

# Exibir os atributos físicos, mecânicos e ópticos perfeitamente alinhados
print("Dimensões do Tensor 1D:", df_tensor.shape)
print(df_tensor[['OSNR (dB)', 'Loss_dB', 'Temperature_C', 'Stress_Pa']].head())
