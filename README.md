# Workshop: Matriz de Confiança 🎯

Repositório do **Workshop Matriz de Confiança**, focado na avaliação de modelos de Machine Learning em cenários reais de negócio com forte desbalanceamento de classes, análise de custos de erro, calibração de probabilidades, políticas de abstenção e engenharia de features de crédito e antifraude.

---

## 📌 Visão Geral do Projeto

Muitos modelos de Machine Learning apresentam acurácias aparentemente elevadas (ex: 92%), mas falham catastroficamente ao serem avaliados sob as métricas de negócio e assimetria de custos do mundo real.

Neste projeto, resolvemos o desafio prático de **Detecção de Fraude e Risco de Crédito** (`dataset_desafio_credito.csv` com 18.000 registros), implementando o pipeline completo no notebook [`05_meu_modelo.ipynb`](05_meu_modelo.ipynb) e consolidando as decisões no [`relatorio-de-confianca.md`](relatorio-de-confianca.md).

---

## 📂 Estrutura do Repositório

```bash
├── 00_setup.ipynb                  # Verificação de ambiente e dependências
├── 01_confusao.ipynb                # Bloco 1: Matriz de confusão e armadilha da acurácia
├── 02_custo_limiar.ipynb            # Bloco 2: Matriz de custo e otimização de limiar operacional
├── 03_calibracao.ipynb              # Bloco 3: Confiabilidade, ECE, Brier e calibração de probabilidades
├── 04_incerteza.ipynb               # Bloco 4: Incerteza epistêmica e política de abstenção
├── 05_meu_modelo.ipynb              # O Desafio do Squad: Benchmark, Feature Engineering e Pipeline Completo
├── dataset_desafio_credito.csv      # Dataset do desafio de crédito (18.000 amostras)
├── relatorio-de-confianca.md        # Relatório de Confiança com os 9 entregáveis + Anexos Técnicos
├── template_relatorio-de-confianca.md # Template original preenchido
├── matriz-de-confianca-slides.key   # Slides originais do workshop (Keynote)
└── matriz-de-confianca-slides.ppt   # Slides originais do workshop (PowerPoint)
```

---

## 🚀 Principais Descobertas e Resultados

### 1. Benchmark Comparativo Completo (Modelos 100% Treinados)

Avaliamos nos mesmos splits estratificados os dois modelos principais com e sem Engenharia de Features:

| Modelo | Limiar Ótimo ($t^*$) | Custo Total em $t^*$ | VP (Fraudes Pegas) | FN (Perdidas) | FP (Atrito) | Recall | Precisão | Balanced Acc | MCC | AUC-ROC | ECE |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **Regressão Logística (Base)** | 0,0410 | R$ 338.700 | 370 | 47 | 2.447 | 88,7% | 13,1% | 69,8% | 0,2117 | **0,7972** | **0,0052** |
| **Regressão Logística (+ Features)** | **0,0560** | **R$ 336.600** | 344 | 73 | **1.906** | 82,5% | **15,3%** | **72,1%** | **0,2396** | 0,7952 | 0,0066 |
| **Random Forest (+ Feat Bruta)** | 0,0360 | R$ 391.000 | 363 | 54 | 2.830 | 87,1% | 11,4% | 65,1% | 0,1643 | 0,7619 | 0,0132 |
| **Random Forest (+ Feat Calibrada)** | 0,0310 | R$ 394.200 | 334 | 83 | 2.282 | 80,1% | 12,8% | 67,2% | 0,1832 | 0,7597 | 0,0080 |

### 2. O Impacto da Engenharia de Features
Desenvolvemos 11 variáveis de domínio bancário (*DTI da nova operação*, *alavancagem de salários*, *estabilidade profissional relativa*, *índice de estresse de consultas e atrasos*):
- Na **Regressão Logística**, as variáveis de razão permitiram capturar relações não-lineares de solvência, **eliminando 541 falsos positivos** (de 2.447 para 1.906), elevando a precisão para **15,29%** e a Balanced Accuracy para **72,12%**, com o menor custo financeiro do projeto (**R$ 336.600**).
- Na **Random Forest**, o aumento de dimensionalidade com variáveis colineares aumentou a variância das árvores, elevando o custo para R$ 391.000.

### 3. A Solução Arquitetural: Esteira de 3 Zonas
Para não submeter clientes a uma decisão binária cega em um cenário de classe desbalanceada, adotamos:
1. **Zona Verde ($p < 0,05$):** Aprovação direta no app (60% do volume).
2. **Zona Amarela ($0,05 \le p \le 0,25$):** Fricção leve com biometria facial e análise pela Mesa Humana de Risco (SLA de 4 horas).
3. **Zona Vermelha ($p > 0,25$):** Recusa preventiva e direcionamento para agência.

---

## 🛠️ Tecnologias Utilizadas

- **Python 3.12**
- **scikit-learn** (Pipelines, LogisticRegression, RandomForestClassifier, CalibratedClassifierCV)
- **pandas** e **numpy**
- **matplotlib**

---

## 📄 Licença

Este material faz parte do workshop **Matriz de Confiança**.
