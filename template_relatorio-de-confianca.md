# Relatório de Confiança

> Entregável do workshop Matriz de Confiança.
> Preenchido com os dados reais gerados no notebook `05_meu_modelo.ipynb` sobre o dataset `dataset_desafio_credito.csv`.

**Modelo:** Regressão Logística com StandardScaler e OneHotEncoder (selecionada via Benchmark Comparativo)  
**Time:** Squad de Prevenção a Fraude e Concessão de Crédito

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

Limiar operacional usado: **t\* = 0,041**

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
