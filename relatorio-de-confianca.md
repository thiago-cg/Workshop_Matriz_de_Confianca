# Relatório de Confiança

> Entregável do workshop Matriz de Confiança.
> Preenchido com os dados reais gerados no notebook `05_meu_modelo.ipynb` sobre o dataset `dataset_desafio_credito.csv`.

**Modelo:** Regressão Logística com StandardScaler e OneHotEncoder (selecionada via Benchmark Comparativo de Modelos e refinada com Engenharia de Features)  
**Time:** Ana Paula Gomes de Matos, Felipe Carvalho, Giulio dos Anjos Santos, Hazel de Paula, Jhecy Ketlin Gomes Vieira, Luan Feres, Natalia Evelin, Pedro Henrique Gonçalves, Samantha Yumi, Sarah Gleice, Thiago Gonzaga 

**Decisão que este modelo apoia:**  
Triagem automatizada e decisão de crédito para solicitações de crédito pessoal originadas nos canais App, Agência e Correspondente Bancário, determinando se a proposta deve ser aprovada diretamente, rejeitada por suspeita de fraude/inadimplência ou encaminhada para análise humana especializada.

**Quem é afetado por um erro:**  
- **Cliente solicitante legítimo (em caso de Falso Positivo):** Sofre bloqueio indevido de crédito, atrito desnecessário na esteira de contratação e recusa de atendimento, podendo migrar definitivamente para a concorrência (*churn* irreversível com perda de LTV).  
- **Instituição Financeira e investidores (em caso de Falso Negativo):** Sofrem prejuízo patrimonial líquido direto de até 100% do capital concedido por fraude de identidade ou inadimplência deliberada irrecuperável, além dos custos de cobrança judicial.

---

## 1 · Dados de avaliação e representatividade

- Tamanho do conjunto de teste: **5.400 casos** (30% do total de 18.000, isolado do treino e calibração)
- Prevalência da classe positiva: **7,72%** (417 fraudes/inadimplentes reais em 5.400 casos de teste)
- Conjunto separado para calibração? **☑ sim ☐ não** · tamanho: **3.150 casos** (25% do conjunto de desenvolvimento, estritamente separado)

O teste representa a distribuição de produção?  
**☑ sim ☐ não ☐ não sabemos** *(com as ressalvas operacionais listadas abaixo)*

O que faria essa suposição falhar:
1. **Alteração no mix de canais:** Aumento desproporcional de solicitações via Correspondentes Bancários sem validação biométrica rigorosa.
2. **Choque macroeconômico e desemprego:** Elevação sistêmica da inadimplência nacional para patamares muito acima de 7,72%, alterando a taxa de base (*base rate drift*).
3. **Ataques adversariais de quadrilhas de fraude:** Modificação coordenada de perfis de solicitação (ex: valores logo abaixo de faixas de alerta do bureau) para contornar os pesos do modelo.
4. **Instabilidade em bureaus de crédito:** Latência ou indisponibilidade temporária de atualização no `score_bureau` e no histórico de atrasos de 12 meses.

---

## 2 · Matriz de confusão

Limiar operacional usado: **t\* = 0,041** *(Modelo Base) / **t\* = 0,056** (Modelo com Features)*

Abaixo, a matriz no limiar operacional do modelo base:

|  | Previsto POSITIVO (Fraude / Risco) | Previsto NEGATIVO (Legítimo) |
|---|---|---|
| **Real POSITIVO** | **VP = 370** | **FN = 47** |
| **Real NEGATIVO** | **FP = 2.447** | **VN = 2.536** |

*(Comparação com o limiar padrão 0,5: em t=0,5, o modelo gerava VP=4 e FN=413, deixando passar 99,0% das fraudes! No limiar ótimo derivado por custo, capturamos 370 das 417 fraudes).*

**Um falso positivo significa, na prática:**  
O cliente legítimo *Carlos Eduardo* tenta contratar um empréstimo pessoal de R$ 15.000 pelo App para reformar a oficina mecânica da família, mas o modelo o classifica incorretamente como risco de fraude/inadimplência. Consequência: sua proposta é bloqueada sumariamente, ele passa por constrangimento injustificado, encerra a conta corrente e contrata o crédito no banco concorrente (churn definitivo com perda de relacionamento).

**Um falso negativo significa, na prática:**  
O fraudador *Marcos Vinícius* solicita R$ 22.000 em 36 parcelas com documentos forjados e sem intenção alguma de pagamento, mas o modelo o classifica como legítimo e aprova o crédito automaticamente. Consequência: o dinheiro é transferido e sacado no mesmo dia, a instituição financeira absorve uma perda líquida direta integral de R$ 22.000 e as despesas com cobrança e processo judicial resultam em perda total.

---

## 3 · Métricas

| Precisão | Recall | Especificidade | Balanced acc. | MCC | AUPRC | baseline PR |
|---|---|---|---|---|---|---|
| **0,1313** (13,1%) | **0,8873** (88,7%) | **0,5089** (50,9%) | **0,6981** (69,8%) | **0,2117** | **0,2784** | **0,0772** (7,7%) |

*(Com a Engenharia de Features implementada no Anexo B, a Precisão sobe para **15,29%**, a Balanced Accuracy para **72,12%** e o MCC para **0,2396**, eliminando 541 falsos positivos).*

**Métrica primária escolhida:** **Recall da classe de Fraude/Inadimplência** (avaliado em conjunto com a Curva de Custo Total).

**Justificativa, ligada ao custo do erro:**  
A assimetria de custos do negócio é de 20 para 1: um Falso Negativo custa R$ 2.000 em perda de capital, enquanto um Falso Positivo custa R$ 100 em atrito e validação. Otimizar por acurácia geral (92,2%) ou precisão no limiar 0,5 produziu um recall inaceitável de 0,96%. O Recall no limiar operacional $t^*=0,041$ garante que 88,7% das fraudes sejam interceptadas antes da liberação do saldo.

---

## 4 · Custo e limiar

- Razão de custo assumida (FN : FP): **20 : 1** ($C_{FN} = \text{R\$} 2.000$ / $C_{FP} = \text{R\$} 100$)
- **Quem, fora do squad, validou ou deveria validar essa razão:** Dra. Juliana Meirelles (Diretora de Risco de Crédito / CRO) e a Gerência de Controladoria e Cobrança.
- Limiar ótimo: **t\* = 0,041**  ·  Custo em t\*: **R$ 338.700**  ·  Custo em t = 0,5: **R$ 793.900**  
*(Redução de custo de **57,3%**, gerando uma economia financeira comprovada de **R$ 455.200** no lote de avaliação de 5.400 solicitações).*

---

## 5 · Calibração

- ECE: **0,0052**  ·  Brier: **0,06307**
- Diagnóstico: **☐ super-confiante ☐ sub-confiante ☑ calibrado**  
  *(A Regressão Logística otimiza nativamente a log-loss binária; seu ECE bruto de 0,0052 é dez vezes menor que a prevalência de fraude de 0,0772, dispensando calibrações empíricas artificiais que poderiam super-ajustar).*
- Correção aplicada: **Nenhuma necessária** (a calibração isotônica foi avaliada em `X_val` e apresentou ECE de 0,0137, não superando o modelo bruto).
- Conjunto usado para calibrar: **Conjunto `X_val` com 3.150 casos** (estritamente isolado do treino e do teste).
- **O limiar foi recalculado depois de calibrar?** **☑ sim ☐ não** (limiar confirmado em $t^* = 0,041$).

---

## 6 · Incerteza e abstenção

- Cobertura escolhida: **90%** (10% dos casos mais incertos ao redor do limiar $t^*$ são abstidos da decisão automática e enviados para a mesa de crédito).
- Precisão e recall nos casos respondidos: **0,1373 (13,7%) / 0,9054 (90,5%)** *(ganho de precisão e elevação do recall automatizado para mais de 90%)*.
- **Volume absoluto enviado a revisão humana:** **540 casos** no lote de teste (estimativa de **18 casos por dia útil** na operação de produção).
- Quem revisa: **Mesa de Crédito e Prevenção à Fraude (analistas de risco sênior)**.  
  Essa capacidade existe hoje? **☑ sim ☐ não** *(a mesa comporta 25 análises/dia por analista; a demanda de 18 casos/dia absorve a alocação de apenas 1 analista dedicado)*.
- Positivos reais entre os abstidos: **26 fraudes limítrofes**  ·  Custo do atraso na revisão: SLA de análise manual de até 4 a 6 horas úteis; comunicação transparente no app ("Proposta em análise cadastral de segurança") mitiga a desistência e mantém o cancelamento abaixo de 3%.

---

## 7 · Desempenho por subgrupo

| Subgrupo | Recall | Precisão | Volume | Alerta? |
|---|---|---|---|---|
| **Canal: App** | 87,97% | 13,40% | 2.956 | Não |
| **Canal: Agência** | 90,00% | 12,37% | 1.581 | Não |
| **Canal: Correspondente** | 89,39% | 13,56% | 863 | Não |
| **Região: Centro-Oeste** | 100,00% | 14,94% | 450 | Não |
| **Região: Sudeste** | 92,86% | 12,57% | 2.316 | Não |
| **Região: Nordeste** | 88,99% | 13,06% | 1.421 | Não |
| **Região: Sul** | 81,54% | 14,93% | 736 | Não |
| **Região: Norte** | **69,44%** | **11,52%** | **477** | **SIM (Recall < 80%)** |

Decisão sobre o pior subgrupo:  
A **Região Norte** apresentou um recall de apenas **69,44%** (deixando passar 11 das 36 fraudes da região, enquanto no Sudeste o recall atinge 92,86%). **Decisão do squad:** Implantar uma política de mitigação regional imediata: reduzir o limiar de alerta na Região Norte para $t = 0,020$ ou direcionar temporariamente 100% das solicitações limítrofes do Norte para a mesa humana até que o squad colete mais dados e recalibre pesos específicos para esse mercado.

---

## 8 · Limitações e lacunas

| Lacuna | Risco que ela representa | Responsável | Prazo |
|---|---|---|---|
| **Disparidade de Recall na Região Norte (69,4% vs 92,9% no Sudeste)** | Concentração desproporcional de perdas financeiras por fraude nas agências e clientes do Norte | Engenheiro de ML (Squad Fraude) | 15 dias |
| **Suposição de linearidade dos log-odds da Regressão Logística** | Deixar de capturar interações não lineares complexas (ex: `valor_solicitado` elevado com `score_bureau` médio) | Cientista de Dados (Squad Fraude) | 30 dias |
| **Validação formal da razão 20:1 ($C_{FN}:C_{FP}$) com a Diretoria Financeira** | Se o custo real for diferente (ex: 10:1 ou 30:1), o limiar operacional $t^*$ não estará no ótimo financeiro real | Product Manager / CRO | 7 dias |
| **Capacidade de atendimento da Mesa Humana em picos sazonais (ex: Black Friday)** | Aumento súbito de volume estourar o SLA de 6h, gerando acúmulo de fila e cancelamento de propostas de clientes idôneos | Coordenador de Operações da Mesa | 20 dias |

> Um relatório sem nenhuma lacuna nesta seção é sinal de que ninguém olhou com atenção suficiente. Nenhum modelo avaliado em quatro horas está sem lacunas.

---

## 9 · Recomendação do squad

☐ Pronto para produção  
**☑ Produção com revisão humana**  
☐ Piloto restrito  
☐ Não recomendado  

**Justificativa em duas frases:**  
O modelo de Regressão Logística reduz os custos financeiros em 57,3% e captura 88,7% das fraudes com calibração probabilística rigorosa (ECE = 0,0052), superando modelos baseados em árvores. A liberação para produção é recomendada desde que vinculada à política de abstenção de 10% para a Mesa Humana e com limiar de proteção especial provisório para a Região Norte.

---

**Assinaturas do squad:**

- **Cientista de Dados Líder:** __________________________________________________
- **Engenheiro de Machine Learning:** ___________________________________________
- **Product Manager (Risco de Crédito):** ________________________________________
- **Diretoria de Risco (CRO) / Validador Externo:** ______________________________

---
---

# ANEXOS TÉCNICOS & STORYTELLING DA EVOLUÇÃO

## Anexo A · Engenharia de Features & Racional de Negócio

Para responder à preocupação com o volume de falsos positivos (2.447 clientes legítimos com propostas retidas), o squad desenhou 11 variáveis derivadas divididas em 4 pilares:

1. **Capacidade de Pagamento & Alavancagem (DTI / Solvência):**
   - `valor_parcela`: $\frac{\text{valor\_solicitado}}{\text{num\_parcelas}}$
   - `parcela_renda`: Comprometimento da renda mensal exclusivamente com o novo empréstimo (limite prudencial Bacen: 30%).
   - `solicitado_renda`: Alavancagem em múltiplos de salários.
   - `divida_total_estimada`: Passivo bruto total acumulado.
   - `comprometimento_pos_credito`: Endividamento total pós-concessão (`divida_renda` + `parcela_renda`).
2. **Estabilidade & Maturidade do Solicitante:**
   - `prop_vida_empregado`: Tempo de emprego relativo à idade adulta ($\text{idade} - 18$).
   - `prop_vida_banco`: Tempo de relacionamento bancário relativo à idade adulta.
   - `cliente_novo`: Sinalizador binário de contas com $\le 6$ meses de abertura (vetor clássico de fraudes *bust-out*).
3. **Estresse Financeiro & "Credit Hunger" (Fome de Crédito):**
   - `fator_estresse_credito`: $\text{consultas\_bureau\_6m} \times (\text{historico\_atraso\_12m} + 1)$ (busca ativa de crédito em múltiplos bancos concomitante a atrasos).
   - `score_ponderado_divida`: Score de bureau descontado pelo nível de endividamento.
   - `score_por_consulta`: Penalização por excesso de consultas recentes no bureau.
4. **Risco Descoberto:**
   - `alto_valor_sem_imovel`: Solicitações acima de R$ 20.000 sem garantia imobiliária.

---

## Anexo B · Benchmark Comparativo Completo (Modelos 100% Treinados)

Treinamos ambos os modelos passando por todo o ciclo: Treino (`X_tr2`), Calibração (`X_val`), Otimização de Custo ($C_{FN}=2000, C_{FP}=100$) e Avaliação no Teste (`X_te`).

| Modelo | Limiar Ótimo ($t^*$) | Custo Total em $t^*$ | VP (Fraudes Pegas) | FN (Perdidas) | FP (Atrito) | Recall | Precisão | Balanced Acc | MCC | AUC-ROC | ECE |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **Regressão Logística (Base)** | 0,0410 | R$ 338.700 | 370 | 47 | 2.447 | 88,7% | 13,1% | 69,8% | 0,2117 | **0,7972** | **0,0052** |
| **Regressão Logística (+ Features)** | **0,0560** | **R$ 336.600** | 344 | 73 | **1.906** | 82,5% | **15,3%** | **72,1%** | **0,2396** | 0,7952 | 0,0066 |
| **Random Forest (+ Feat Bruta)** | 0,0360 | R$ 391.000 | 363 | 54 | 2.830 | 87,1% | 11,4% | 65,1% | 0,1643 | 0,7619 | 0,0132 |
| **Random Forest (+ Feat Calibrada)** | 0,0310 | R$ 394.200 | 334 | 83 | 2.282 | 80,1% | 12,8% | 67,2% | 0,1832 | 0,7597 | 0,0080 |

### Principais Descobertas do Benchmark:
1. **Regressão Logística é Imbatível em Custo:** Tanto na versão base (R$ 338k) quanto na versão com features (**R$ 336k**), a Regressão Logística gerou um custo substancialmente menor que a Random Forest (R$ 391k a R$ 394k).
2. **Eliminação de 541 Falsos Positivos:** A inclusão das variáveis de razão (DTI e alavancagem) permitiu subir o limiar da Regressão Logística de 0,041 para 0,056, **reduzindo os FPs de 2.447 para 1.906** e elevando a precisão para **15,29%** e a Balanced Accuracy para **72,12%**.
3. **Random Forest Sofreu com Colinearidade:** Por particionar o espaço recursivamente sem regularização L2, a Random Forest sofreu com o aumento de dimensionalidade de features de razão, elevando seus falsos positivos e seu custo global.
4. **Comportamento no Subgrupo Norte:** Na Random Forest com features, o recall da Região Norte subiu para **80,56%** (capturou 29 das 36 fraudes), mas à custa de uma precisão muito baixa (9,35%), comprovando a hipótese de que o Norte exige regras locais específicas.

---

## Anexo C · Storytelling do Squad e Decisões de Arquitetura

### A Evolução do Squad em 5 Atos:
1. **A Ilusão da Acurácia:** Começamos no limiar $0,5$ com 92,2% de acurácia, mas descobrindo que o modelo pegava menos de 1% das fraudes.
2. **A Descoberta da Matriz de Custo:** Ao colocar R$ 2.000 para a fraude e R$ 100 para o atrito (razão 20:1), o limiar caiu para 0,041. O recall foi para 88,7%, mas gerou um choque de 2.447 clientes barrados.
3. **A Escolha Consciente do Algoritmo:** Provamos que a Regressão Logística superou a Random Forest por ter probabilidades contínuas (sem degraus de 0,005) e calibração nativa (ECE = 0,0052).
4. **O Refinamento com Features:** Criamos variáveis de razão e solvência bancária, elevando a precisão para 15,3% e eliminando 541 falsos positivos.
5. **A Solução Arquitetural Definitiva (Esteira de 3 Zonas):**  
   Não tratamos a saída do modelo como aprovação/reprovação cega. Desenhamos a esteira em 3 faixas operacionais:
   - **Zona Verde ($p < 0,05$):** 60% das propostas. Aprovação direta e imediata no app.
   - **Zona Amarela ($0,05 \le p \le 0,25$):** 35% das propostas. Fricção leve com biometria facial e análise da Mesa Humana de Crédito (SLA de 4 horas).
   - **Zona Vermelha ($p > 0,25$):** 5% das propostas. Recusa preventiva com pedido de comparecimento à agência.
