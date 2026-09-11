# Workshop: Matriz de Confiança 🎯

Repositório do **Workshop Matriz de Confiança**, focado na avaliação de modelos de Machine Learning em cenários reais de negócio com forte desbalanceamento de classes, análise de custos de erro, calibração de probabilidades e políticas de abstenção operacional.

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
├── 05_meu_modelo.ipynb              # O Desafio do Squad: Benchmark e pipeline ponta a ponta
├── dataset_desafio_credito.csv      # Dataset do desafio de crédito (18.000 amostras)
├── relatorio-de-confianca.md        # Relatório de Confiança executivo preenchido com os 9 entregáveis
├── template_relatorio-de-confianca.md # Template original do entregável
├── matriz-de-confianca-slides.key   # Slides originais do workshop (Keynote)
└── matriz-de-confianca-slides.ppt   # Slides originais do workshop (PowerPoint)
```

---

## 🚀 Principais Descobertas e Resultados

### 1. Benchmark de Seleção de Modelo
Comparamos três famílias nos mesmos splits estratificados (treino, calibração e teste):
- **Regressão Logística (Modelo Campeão):** **AUC-ROC de 0,7972**, **AUPRC de 0,2784** e **ECE de 0,0052**, gerando o menor custo financeiro (**R$ 338.700**).
- **Random Forest:** AUC-ROC de 0,7687 | AUPRC de 0,2347 | Custo R$ 373.100.
- **HistGradientBoosting:** AUC-ROC de 0,7717 | AUPRC de 0,2306 | Custo R$ 369.500.

### 2. A Armadilha do Limiar Default ($t=0,5$) vs Limiar Ótimo ($t^*=0,041$)
- No limiar padrão $0,5$: Acurácia de 92,2%, mas o **Recall era de apenas 0,96%** (deixava passar 413 das 417 fraudes!).
- No limiar operacional derivado pela matriz de custos ($C_{FN}=\text{R\$} 2.000$ e $C_{FP}=\text{R\$} 100$, razão 20:1):
  - **Limiar Ótimo:** $t^* = 0,041$
  - **Recall:** Saltou para **88,73%** (captura 370 fraudes).
  - **Redução de Custo Financeiro:** De R$ 793.900 para R$ 338.700 (**economia de 57,3%** / R$ 455.200 no lote de teste).

### 3. Fatiamento por Subgrupos
- Identificação de disparidade crítica na **Região Norte** (Recall de 69,44% vs 92,86% no Sudeste), motivando a criação de um limiar regional mais protetivo.

### 4. Política de Abstenção e Revisão Humana
- Cobertura de 90%: 10% dos casos mais limítrofes ($|p - t^*|$) são enviados para a **Mesa Humana de Crédito** (~18 casos/dia útil), retendo 26 fraudes limítrofes e elevando o recall automatizado para mais de 90%.

---

## 🛠️ Tecnologias Utilizadas

- **Python 3.12**
- **scikit-learn** (Classificadores, CalibratedClassifierCV, Pipeline, Métricas)
- **pandas** e **numpy**
- **matplotlib**

---

## 📄 Licença

Este material faz parte do workshop **Matriz de Confiança**.
