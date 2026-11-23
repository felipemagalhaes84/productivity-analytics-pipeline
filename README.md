# 📊 Productivity & Behavioral Analytics — To-Do List Data Pipeline

![Status](https://img.shields.io/badge/Status-Em%20Desenvolvimento-yellow)
![Python](https://img.shields.io/badge/Python-3.11+-blue?logo=python&logoColor=white)
![SQLite](https://img.shields.io/badge/SQLite-Relational%20DB-003B57?logo=sqlite&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-Data%20Analysis-150458?logo=pandas&logoColor=white)
![Streamlit](https://img.shields.io/badge/Streamlit-Web%20App-FF4B4B?logo=streamlit&logoColor=white)
![Power BI](https://img.shields.io/badge/Power%20BI-Dashboard-F2C811?logo=powerbi&logoColor=black)

Sistema local de gestão de tarefas projetado não apenas para organizar rotinas diárias, mas como um **pipeline de captura de dados comportamentais** para análise de produtividade e tomada de decisão no **Power BI**.

---

## 🎯 Problema de Negócio e Objetivo

Ferramentas tradicionais de tarefas mostram apenas o que foi marcado como concluído, mas falham em responder perguntas analíticas essenciais sobre o desempenho do usuário:
* *Onde estou investindo mais tempo versus o que realmente importa?*
* *Qual é a minha taxa real de execução em tarefas de **Alta Prioridade**?*
* *Em quais categorias (Estudos, Trabalho, Saúde, Projetos Pessoais, Tarefas Domésticas) ocorrem os maiores gargalos ou interrupções?*
* *A minha baixa conclusão é falha de execução ou excesso de planejamento?*

Este projeto resolve esse problema criando uma **fonte de dados estruturada, auditável e imutável** a partir da rotina diária, permitindo diagnosticar falhas de planejamento e hábitos de execução por meio de indicadores quantitativos e dashboards interativos.

---

## 🏗️ Arquitetura da Solução

```text
┌───────────────────┐     ┌──────────────────────────┐     ┌─────────────────────────────┐
│  Interface Web    │────▶│  Regras de Negócio       │────▶│  Banco Relacional (SQLite)  │
│  (Streamlit)      │     │  (Python + ZoneInfo)     │     │  Triggers, Checks & Índices │
└───────────────────┘     └──────────────────────────┘     └──────────────┬──────────────┘
                                                                          │
┌───────────────────┐     ┌──────────────────────────┐                    │
│  Dashboards       │◀────│  ETL & Indicadores       │◀───────────────────┘
│  (Power BI/Plotly)│     │  (Pandas + NumPy + CSV)  │
└───────────────────┘     └──────────────────────────┘
```

### Stack Tecnológica
* **Linguagem:** Python
* **Banco de Dados:** SQLite (`sqlite3`) com integridade referencial, restrições `CHECK`/`UNIQUE`, `TRIGGER` e índice único parcial
* **Manipulação e Métricas:** Pandas, NumPy e OpenPyXL
* **Controle Temporal:** `zoneinfo` / `tzdata` (sincronização entre `UTC` no banco e horário local `America/Sao_Paulo`)
* **Interface e Visualização:** Streamlit, Plotly e Microsoft Power BI

---

## 🗄️ Modelagem Relacional e Governança de Dados

O banco de dados (`database/todolist.db`) foi estruturado em **três entidades principais** (`1:N:N`) para evitar redundância e permitir múltiplas sessões de trabalho por atividade:

1. **`listas` (Ciclo Diário):** Representa um único dia (`data` com restrição `UNIQUE`), operando das `00:00` às `23:59:59` (horário de Brasília). Armazena timestamps de criação e encerramento.
2. **`tarefas` (Planejamento):** Vinculada a uma lista diária (`lista_id`), classifica cada atividade por **categoria** (*Estudos*, *Trabalho*, *Saúde e exercícios*, *Tarefas domésticas*, *Projetos pessoais*), **prioridade** (*alta*, *média*, *baixa*) e **status** (*pendente*, *realizada*).
3. **`registros_tempo` (Execução Real):** Registra cada bloco de tempo dedicado a uma tarefa (`inicio`, `termino`, `duracao_segundos` e `origem`: `'cronometro'` ou `'manual'`). Permite interromper e retomar uma tarefa várias vezes ao dia sem perder a granularidade das sessões.

### Diferenciais de Engenharia e Integridade
* **Imutabilidade Histórica (Proteção em Duas Camadas):** À meia-noite — ou na primeira abertura do sistema após a virada do dia — listas anteriores são encerradas automaticamente. **Triggers no SQLite** bloqueiam qualquer `INSERT`, `UPDATE` ou `DELETE` em listas encerradas ou em suas tarefas, garantindo que os dados históricos reflitam a realidade daquele dia.
* **Cronômetro Resiliente a Falhas:** Um índice único parcial (`idx_cronometro_ativo_tarefa`) impede que uma mesma tarefa tenha dois cronômetros ativos simultaneamente (`termino IS NULL`), permitindo recuperar sessões em andamento caso o navegador seja fechado inesperadamente.
* **Replicação Inteligente:** Ao replicar a lista de um dia anterior para reaproveitar rotinas, o sistema preserva descrições, categorias e prioridades, mas reinicia os status para `pendente` e mantém o histórico original intacto.

---

## 📈 Indicadores Analíticos (KPIs)

A estrutura de dados alimenta a camada de análise (Python/Pandas e Power BI) com métricas desenhadas para separar **cumprimento de planejamento** de **esforço efetivo**:

| Indicador | Fórmula / Lógica | Objetivo Analítico |
| :--- | :--- | :--- |
| **Taxa de Conclusão** | `(Tarefas Realizadas ÷ Total de Tarefas) × 100` | Medir a aderência ao planejamento diário |
| **Taxa de Pendências** | `(Tarefas Pendentes ÷ Total de Tarefas) × 100` | Identificar sobrecarga ou adiamento recorrente |
| **Tempo Diário Investido** | `Σ duracao_segundos` por dia | Quantificar o esforço líquido real |
| **Distribuição por Categoria** | `Σ duracao_segundos` agrupado por categoria | Avaliar o equilíbrio entre áreas da vida |
| **Execução por Prioridade** | Taxa de conclusão segmentada (`Alta`, `Média`, `Baixa`) | Detectar procrastinação em tarefas críticas |
| **Consistência** | Contagem de dias com listas registradas | Medir disciplina de acompanhamento |

---

## 📂 Estrutura do Projeto

```text
todo_list/
├── app.py                     # Interface web interativa (Streamlit)
├── database/
│   └── todolist.db            # Banco de dados relacional SQLite
├── modules/
│   ├── database.py            # Conexão, DDL, triggers, índices e regras de integridade
│   ├── listas.py              # Regras de ciclo diário, encerramento automático e replicação
│   ├── tarefas.py             # Operações CRUD de tarefas
│   ├── registros_tempo.py     # Controle de cronômetro e lançamentos manuais
│   └── indicadores.py         # Cálculo de KPIs e exportação para CSV/Excel
├── exports/
│   ├── tarefas.csv            # Base estruturada para ingestão no Power BI
│   └── tarefas.xlsx           # Relatório tabular exportado
├── dashboards/
│   └── dashboard.py           # Visualizações exploratórias com Plotly
├── tests/                     # Scripts de validação de banco, CRUD, triggers e fuso horário
└── requirements.txt           # Dependências do projeto
```

---

## 🚧 Status do Projeto e Roadmap

O projeto está sendo desenvolvido de forma modular, com validação automatizada em cada camada antes da construção da interface final:

- [x] **Modelagem Relacional & Banco SQLite (`modules/database.py`):** Criação das tabelas, chaves estrangeiras, `CHECK` constraints, `TRIGGER` de imutabilidade e índice único parcial.
- [x] **Gestão de Listas & Ciclo Diário (`modules/listas.py`):** Criação, consulta, encerramento manual e encerramento automático sensível ao fuso horário (`America/Sao_Paulo`).
- [x] **CRUD de Tarefas (`modules/tarefas.py`):** Cadastro, leitura, edição, atualização de status e exclusão com validação de integridade.
- [x] **Testes de Proteção e Integridade:** Bateria de testes validando bloqueio de escrita em listas encerradas e unicidade de cronômetros ativos.
- [ ] **Módulo de Controle de Tempo (`modules/registros_tempo.py`):** Finalização das funções de iniciar/encerrar cronômetro, recuperação de sessão aberta e lançamento manual.
- [ ] **Camada Analítica & Exportação (`modules/indicadores.py`):** Cálculo consolidado de KPIs e exportação automatizada para `.csv` e `.xlsx`.
- [ ] **Interface Web (`app.py`):** Desenvolvimento das telas interativas em Streamlit.
- [ ] **Dashboard Executivo (Power BI):** Modelagem estrela/tabular e painéis visuais de diagnóstico de produtividade.

---

## ⚙️ Como Executar Localmente

1. **Clone o repositório:**
   ```bash
   git clone [https://github.com/felipemagalhaes84/seu-repositorio.git](https://github.com/felipemagalhaes84/seu-repositorio.git)
   cd todo_list
   ```

2. **Crie e ative o ambiente virtual:**
   ```bash
   python -m venv venv
   # Windows (PowerShell):
   .\venv\Scripts\Activate.ps1
   # Linux/macOS:
   source venv/bin/activate
   ```

3. **Instale as dependências:**
   ```bash
   pip install -r requirements.txt
   ```

4. **Inicialize o banco de dados:**
   ```bash
   python modules/database.py
   ```

---

## 👤 Autor

**Luis Felipe Magalhães Pereira**  
Estudante de Ciência de Dados e Inteligência Artificial | Python • SQL • Pandas • Power BI  
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Conectar-0A66C2?logo=linkedin&logoColor=white)](https://www.linkedin.com/in/luis-felipe-magalhaes-pereira-aa7aa3ba/)
