##📌 Visão Geral
Este repositório contém a Super Base (Tensor 1D), um conjunto de dados multimodal estruturado para o treinamento de modelos de Aprendizado de Máquina (como Autoformers) e Redes Neurais Informadas pela Física (PINNs) aplicados ao envelhecimento e detecção de anomalias em fibras ópticas. O dataset representa um cenário operacional compatível com a recomendação ITU-T G.652.

🔬 Metodologia de HarmonizaçãoO dataset consolidado possui 2970 registros contínuos. A integração foi realizada em múltiplas etapas, seguindo um rigoroso pipeline de acoplamento de domínios:
Ancoragem Temporal: A cronologia obtida dos registros físicos de OTDR foi adotada como eixo temporal universal para a harmonização das bases ópticas e de telemetria.
Acoplamento Climático: As variáveis ambientais (Open-Meteo) foram incorporadas por proximidade temporal utilizando o método merge_asof.
Acoplamento Mecânico: Foi estabelecida uma regra experimental de acoplamento entre a velocidade do vento e a deformação estrutural, definida formalmente por $\varepsilon^{*}(t) = k_v v(t)$, com $k_v = 5$. O estado mecânico final foi obtido por associação ao valor mais próximo na base experimental de laboratório (IEEE DataPort).
Propagação de Estado: Lacunas amostrais foram tratadas pelo método de propagação do último estado válido (Forward Fill), evitando a introdução arbitrária de zeros e preservando o gradiente físico para o treinamento da rede neural.

📊 Estrutura do Tensor 1D
O vetor multimodal harmonizado integra quatro dimensões de atributos:
Metadados e Tempo: Carimbo ISO 8601 e especificações do enlace.
Sintomas Lógicos (Telemetria): OSNR (dB), BER, potência do laser.
Sintomas Físicos (OTDR): Atenuação e mapeamento espacial de refletância ($P1$ a $P30$).
Estressores (Clima e Mecânica): Temperatura, vento, tensão aplicada em Pascals e taxa de fadiga.

⚖️ Declaração Científica e de UsoNota Metodológica: 
Os registros são derivados de fontes originalmente independentes, posteriormente estruturadas no modelo proposto. Ressalta-se que o processo de harmonização não implica aquisição simultânea das diferentes modalidades sobre a mesma fibra ou enlace físico no mundo real, constituindo um cenário representativo e validado para experimentação in silico.
