# Dashboard de Controle de Prazos e Movimentações Processuais

**Projeto de análise de dados e Business Intelligence | Python, Pandas, Matplotlib, HTML, CSS, JavaScript e Plotly.js**

Este projeto transforma uma base **sintética** de procedimentos em informações para acompanhamento de prazos, movimentações, distribuição de demandas e qualidade dos dados. O trabalho reúne o processo analítico documentado em um notebook Jupyter e um dashboard interativo publicado como página web estática.

### 🌐 Acesse o dashboard interativo

**[▶ ABRIR O DASHBOARD NO NAVEGADOR](https://ayrton-marquezin-venancio.github.io/controle-prazos-movimentacoes-processuais/)**

> **Publicação:** o endereço acima foi preparado para o repositório com nome `controle-prazos-movimentacoes-processuais`. Ele só funcionará depois que os arquivos forem enviados para esse repositório e o **GitHub Pages** for ativado. Caso escolha outro nome, atualize o link. Para a experiência correta, use o endereço do **GitHub Pages**, não o link de visualização do arquivo `.html` no GitHub.

> **Aviso:** ambiente demonstrativo com dados fictícios/didáticos. Não representa um sistema oficial de um Ministério Público, nem utiliza regras jurídicas oficiais de contagem de prazos.

---

## 1. O problema e o objetivo

Uma base administrativa pode conter centenas de registros, mas a quantidade de linhas, isoladamente, não mostra quais demandas exigem atenção. Além de saber **quantos** procedimentos existem, interessa entender:

- Quais registros continuam ativos e possuem prazo vencido?
- Quais estão próximos do prazo?
- Onde há maior concentração de demandas e possíveis gargalos operacionais?
- Há registros sem movimentação recente?
- Quais informações estão incompletas, inconsistentes ou precisam de revisão?
- Como priorizar uma fila de atenção sem confundir volume de trabalho com desempenho de um setor?

**Objetivo:** construir um fluxo de ponta a ponta — **base bruta → tratamento → auditoria → indicadores → análise exploratória → dashboard interativo** — para transformar registros em informações organizadas, rastreáveis e úteis à análise operacional.

## 2. Visão geral dos dados e resultados

Os números abaixo correspondem ao **recorte utilizado na versão entregue**, com **data de referência de 07/10/2026**. Não são indicadores atualizados em tempo real.

| Medida | Resultado | Interpretação |
|:--|--:|:--|
| Registros recebidos | **220** | Linhas do CSV bruto |
| Duplicatas exatas removidas | **9** | Linhas totalmente repetidas após a padronização |
| Registros na base tratada | **211** | Linhas mantidas para análise e auditoria |
| IDs válidos distintos | **209** | Contagem de procedimentos identificados por IDs válidos |
| Registros ativos | **137** | Situações classificadas como ativas |
| Registros ativos com prazo vencido | **120** | Prazo ultrapassado e situação ativa |
| Prazos próximos | **8** | Vencimento de 0 a 30 dias; considera todas as situações |
| Registros classificados como críticos | **26** | Score de atenção igual ou superior a 6 |
| Registros com possíveis inconsistências | **104** | Apresentam pelo menos uma flag de auditoria |

**Atenção à unidade de análise:** *registro* (linha) e *procedimento com ID válido distinto* são contagens diferentes. Nem toda ocorrência é um procedimento único, e algumas situações exigem validação antes de qualquer interpretação operacional.

## 3. Tecnologias utilizadas

| Tecnologia | Papel no projeto |
|:--|:--|
| **Python** | Estruturação da análise e das regras de tratamento |
| **Pandas** | Leitura do CSV, padronizações, flags, agregações e exportação |
| **NumPy** | Suporte às operações analíticas no notebook |
| **Jupyter Notebook** | Registro sequencial, exploratório e reproduzível das etapas |
| **Matplotlib** | Gráficos da análise exploratória dentro do notebook |
| **HTML e CSS** | Estrutura, navegação lateral, responsividade e identidade visual |
| **JavaScript** | Filtros combinados, atualização dinâmica, tabelas e insights |
| **Plotly.js** | Gráficos interativos, tooltips e diferentes formas de visualização |
| **GitHub Pages** | Publicação do dashboard para acesso direto pelo navegador |
| **Inteligência artificial generativa** | Apoio orientado por prompts à revisão de código, à construção de células específicas e à interface HTML |

O dashboard foi desenvolvido como uma página estática: **não exige backend nem servidor Python para sua visualização**. Os dados do recorte tratado estão incorporados ao HTML em formato JSON para alimentar os componentes JavaScript. Ao acessar o **site publicado**, o visitante vê a interface renderizada, e não o JSON como página principal.

## 4. Etapas do projeto: o que foi feito e por quê

### Etapa 1 — Importação e entendimento da base

O notebook inicia a leitura do arquivo `mp_controle_prazos_bruto.csv` com **Pandas** e separador `;`. Na exploração inicial, são inspecionados as colunas, dimensões, tipos de dados, valores ausentes e estatísticas descritivas.

**Por quê?** Antes de calcular indicadores, é necessário conhecer o formato dos campos e identificar problemas que poderiam distorcer os resultados.

### Etapa 2 — Normalização e correção de formatos

Foram aplicadas regras de padronização de:

- **IDs de procedimento:** normalização para o padrão `PROC-AAAA-NNNN`, com sinalização de identificadores que não atendem ao formato esperado;
- **Campos categóricos:** unificação de grafias, abreviações, capitalização e variações de nomes em tipo, promotoria, área, assunto, município, situação, setor e prioridade;
- **Datas:** conversão para o tipo de data, com valores impossíveis ou não reconhecidos tratados como ausentes;
- **Números:** recuperação da parte numérica de campos como prazo em dias e quantidade de movimentações;
- **Campos ausentes:** preservação de um marcador explícito, `Verificar Inconsistência da Base`, quando necessário nas categorias.

**Por quê?** Sem padronização, duas grafias para a mesma categoria podem gerar dois grupos falsamente diferentes em gráficos e tabelas.

### Etapa 3 — Duplicatas e preservação de registros auditáveis

As **9 duplicatas exatas** foram removidas. Em contrapartida, registros com **ID repetido, mas informações potencialmente diferentes**, não foram excluídos automaticamente: foram sinalizados para revisão por meio da coluna `id_repetido`.

**Por quê?** Uma linha integralmente repetida tende a inflar contagens. Já dois registros de mesmo ID com informações diferentes podem indicar uma inconsistência que precisa ser investigada — apagá-los ocultaria um sinal importante.

### Etapa 4 — Auditoria da qualidade dos dados

O projeto cria verificações específicas (*flags*) para situações como:

| Grupo | Exemplos de verificações |
|:--|:--|
| **Identificação** | ID inválido ou repetido |
| **Cronologia** | Última movimentação anterior à abertura; data limite anterior à abertura; datas futuras de abertura ou movimentação |
| **Prazos** | Diferença entre prazo informado e intervalo calculado; prazo sem data limite; data limite sem prazo |
| **Movimentações** | Quantidade negativa; zero movimentações com data de movimentação registrada |
| **Cadastro** | Categorias ausentes ou marcadas para verificação |

A coluna `possui_inconsistencia` identifica registros com **ao menos uma** dessas ocorrências. As flags são independentes: um mesmo registro pode apresentar mais de um motivo de revisão.

**Por quê?** Em vez de esconder ou corrigir silenciosamente situações ambíguas, o notebook mantém os dados auditáveis e torna o problema identificável para avaliação humana. Uma flag **não comprova irregularidade**.

### Etapa 5 — Indicadores de prazo e movimentação

Com uma **data de referência explícita**, foram criadas métricas como:

- `dias_em_aberto`: intervalo entre abertura e data de referência;
- `dias_sem_movimentacao`: intervalo desde a última movimentação;
- `dias_para_prazo`: diferença entre data limite e referência;
- `prazo_vencido`: prazo cujo número de dias restantes é negativo;
- `prazo_vencido_ativo`: prazo vencido e situação ainda ativa;
- `prazo_proximo`: prazo de **0 a 30 dias**, inclusive;
- `faixa_prazo` e `faixa_tempo_parado`: faixas ordenadas para facilitar comparações.

As situações consideradas **ativas** no notebook são: **Em Andamento, Em Análise, Aguardando Manifestação e Suspenso**.

**Por quê?** O mesmo registro pode estar vencido e, ainda assim, encerrado. Separar *prazo vencido* de *prazo vencido em registro ativo* permite uma leitura operacional mais útil. A contagem está em **dias corridos**, no contexto didático do projeto — não representa apuração de prazo jurídico em dias úteis.

### Etapa 6 — Indicadores e análise exploratória

O notebook apresenta contagens de registros e médias para situação, prioridade, setor, área, promotoria e município, além de análise temporal e de inatividade. Para métricas gerenciais, marcadores de inconsistência não são tratados como categorias legítimas e números negativos incompatíveis são desconsiderados das médias pertinentes, sem apagar a linha da base tratada.

Foram usados gráficos do **Matplotlib**, com atenção a cores, ordem lógica, legibilidade, rótulos e adequação do tipo de gráfico à variável.

**Por quê?** A análise exploratória serve para compreender distribuições e possíveis concentrações **antes** de propor uma visualização executiva.

### Etapa 7 — Análise por setor e priorização

Foi construída uma visão por setor com informações como volume de registros, média de dias sem movimentação, prazos vencidos ativos e média de movimentações.

No dashboard, o acompanhamento também diferencia **quantidade absoluta** de vencimentos da **taxa de vencimentos entre registros ativos**:

`taxa de vencimento = registros ativos vencidos do setor / registros ativos do setor`

**Por quê?** Um setor com muitos vencimentos em números absolutos pode ter simplesmente uma carteira maior. Comparar taxa e volume evita concluir, sem evidências suficientes, que ele apresenta necessariamente o pior desempenho.

### Etapa 8 — Fila de atenção: um score analítico e didático

A fila organiza a revisão dos registros com base nas regras implementadas no notebook:

| Condição | Pontos |
|:--|--:|
| Prazo vencido e registro ativo | +3 |
| Prioridade urgente | +3 |
| Prioridade alta | +2 |
| Mais de 60 dias sem movimentação | +2 |
| Prazo próximo (0 a 30 dias) | +1 |

Classificação:

- **Normal:** score de 0 a 2;
- **Atenção:** score de 3 a 5;
- **Crítico:** score a partir de 6.

**Por quê?** Uma fila orientada por critérios explícitos ajuda a ordenar a leitura dos registros. **Não é um modelo preditivo**, não substitui avaliação humana e **não corresponde a uma norma oficial**.

### Etapa 9 — Exportação da base tratada

O notebook exporta `mp_controle_prazos_tratado.csv`, preservando as colunas originais, as métricas derivadas e as flags de auditoria. A versão disponibilizada contém **211 registros e 40 colunas**.

**Por quê?** Isso separa a etapa de tratamento da apresentação dos dados e deixa os indicadores mais fáceis de inspecionar e reproduzir.

### Etapa 10 — Desenvolvimento do dashboard interativo

Com o conjunto tratado e as regras analíticas do notebook como referência, foi produzido um dashboard **HTML/CSS/JavaScript + Plotly.js**. A interface tem menu lateral e oito áreas de navegação:

| Página | O que permite analisar |
|:--|:--|
| **Visão Geral** | KPIs, retrato executivo e insights calculados |
| **Procedimentos** | Situações, prioridades, vencimentos e tempo sem movimentação |
| **Setores** | Rankings, quantidades, taxas e matriz de criticidade |
| **Demandas** | Concentração por área, promotoria, município e evolução de aberturas |
| **Atenção** | Registros que merecem priorização e seus scores |
| **Qualidade dos Dados** | Distribuição de inconsistências e protocolos para revisão |
| **Consulta** | Pesquisa e exploração detalhada dos registros |
| **Metodologia** | Definições e limites dos indicadores |

Os **filtros globais** permitem combinar recortes e atualizar cartões, gráficos, tabelas e insights. A visualização utiliza barras para comparação de categorias, roscas para composições, linha para a série mensal de aberturas e dispersão para relações entre indicadores de setores.

**Por quê?** O dashboard aproxima a análise exploratória de uma experiência de consulta acessível para alguém que não utilizou o notebook.

---

## 5. Como a inteligência artificial foi utilizada

A utilização de IA foi parte **declarada e orientada por prompts**, tanto na evolução do notebook quanto na construção do dashboard. A IA apoiou tarefas de programação, revisão técnica, organização visual e testes. Os dados utilizados não foram obtidos de um modelo generativo: vieram do CSV do projeto.

### No notebook Jupyter

Foram elaborados prompts com **contexto, objetivo, restrições e critérios de aceitação** para apoiar etapas específicas, especialmente:

- refinamento de células de limpeza, padronização e consistência;
- revisão de tratamento de duplicatas e preservação de registros suspeitos;
- orientação para evitar `SettingWithCopyWarning` com cópias explícitas, em vez de ocultar avisos;
- revisão de indicadores e consistência das agregações;
- melhoria dos gráficos Matplotlib, incluindo paletas, arredondamento das barras, rótulos e legibilidade;
- preservação da estrutura e do estilo já desenvolvidos no notebook, sem reescrever o projeto inteiro.

### No dashboard HTML

Foram elaborados prompts mais amplos para orientar o desenvolvimento e sucessivas revisões do painel. As especificações abordaram:

- leitura integral do notebook e uso da base tratada;
- preservação dos cálculos e das flags;
- escolha do tipo de gráfico conforme o objetivo analítico;
- KPIs, rankings, tabelas de revisão e filtros combinados;
- navegação por abas laterais e consistência dos filtros entre páginas;
- melhoria de cores, tipografia, cartões, interações e responsividade;
- correção de datas quebradas em tabelas e refinamento de visualizações Plotly;
- criação de backups e testes de funcionamento sempre que o ambiente permitisse.

### Como os prompts foram estruturados

A lógica usada na elaboração dos prompts foi definir, para cada tarefa:

1. **Papel e contexto:** especialista simulado (analista sênior, desenvolvedor front-end, UI/UX) e objetivo do projeto;
2. **Entrada:** notebook, CSV e/ou HTML existente;
3. **Escopo:** componente ou etapa que deveria ser alterado;
4. **Restrições:** preservar dados, cálculos, regras e arquivos originais;
5. **Critérios analíticos:** legibilidade, unidade de análise, denominadores, ordenação, semântica das cores;
6. **Critérios de validação:** comparar indicadores, verificar filtros, conferir tabelas e relatar limitações.

**Exemplo resumido de orientação de prompt usada no fluxo:**

> Analise o código existente e melhore apenas os gráficos Matplotlib. Preserve as colunas, os cálculos e a sequência do notebook. Corrija problemas de visualização sem esconder erros, mantenha cores coerentes com a interpretação e valide as contagens utilizadas.

Outro exemplo, relacionado ao dashboard:

> Use as regras documentadas no notebook e a base tratada para construir um HTML interativo. Mantenha os cálculos de prazos, inconsistências e score, com filtros globais que afetem toda a interface. Crie ranking de setores sem confundir volume absoluto com taxa, escolha gráficos conforme a pergunta e preserve uma cópia antes de alterações.

**Transparência:** esses exemplos são sínteses das orientações adotadas ao longo do desenvolvimento, não transcrições literais de uma conversa única. A IA foi usada como ferramenta de apoio e implementação, e não como justificativa para aceitar resultados sem revisão. A leitura das métricas, as definições do projeto e a responsabilidade pela apresentação permanecem com quem desenvolve e publica o trabalho.

---

## 6. Estrutura do repositório

```text
controle-prazos-movimentacoes-processuais/
├── README.md
├── index.html
├── dashboard_controle_prazos_movimentacoes_processuais.html
├── controle_e_movimentações.ipynb
├── mp_controle_prazos_bruto.csv
└── mp_controle_prazos_tratado.csv
```

- **`README.md`:** apresentação, decisões técnicas e instruções de execução/publicação;
- **`index.html`:** ponto de entrada que redireciona para o dashboard renderizado no GitHub Pages;
- **`dashboard_...html`:** aplicação interativa de fato, com dados incorporados;
- **`controle_e_movimentações.ipynb`:** roteiro do tratamento, auditoria e análise;
- **`mp_controle_prazos_bruto.csv`:** entrada original;
- **`mp_controle_prazos_tratado.csv`:** saída tratada utilizada pela interface.

Os nomes acima são os nomes padronizados de publicação. A cópia de trabalho pode ter sufixos como `(3)` ou `(1)` decorrentes de downloads e anexos.

## 7. Como executar o projeto

### Abrir apenas o dashboard

Abra `index.html` ou `dashboard_controle_prazos_movimentacoes_processuais.html` em um navegador atualizado. Não é necessário instalar Python para visualizar o dashboard.

### Reproduzir a análise no notebook

1. Coloque o notebook e `mp_controle_prazos_bruto.csv` na mesma pasta.
2. Instale Python e as bibliotecas básicas:

   ```bash
   pip install pandas numpy matplotlib jupyter
   ```

3. Inicie o Jupyter:

   ```bash
   jupyter notebook
   ```

4. Abra `controle_e_movimentações.ipynb` e execute as células na ordem.
5. Ao final, o notebook exportará `mp_controle_prazos_tratado.csv`.

**Reprodutibilidade:** no notebook, `USAR_DATA_ATUAL = True` recalcula os indicadores temporais usando a data do ambiente de execução (fuso `America/Araguaina`). Para reproduzir **exatamente o recorte de 07/10/2026**, altere para `USAR_DATA_ATUAL = False`, pois o notebook contém essa data fixa no ramo alternativo. Reexecutar o notebook não atualiza automaticamente os registros já embutidos no HTML; para isso, seria necessário regenerar o dashboard.

## 8. Publicar o dashboard em um link, sem exibir JSON

O GitHub apresenta arquivos HTML na interface do repositório como **arquivos de código**. Para que um recrutador veja o **dashboard funcionando**, use o **GitHub Pages**.

1. Crie um repositório público chamado **`controle-prazos-movimentacoes-processuais`** no perfil `ayrton-marquezin-venancio` (ou ajuste os endereços abaixo ao nome que escolher).
2. Envie os **seis arquivos** da estrutura anterior para a **raiz** do repositório. Não altere o nome `index.html`.
3. Acesse **Settings → Pages**.
4. Em **Build and deployment**, selecione **Deploy from a branch**.
5. Escolha **`main`** e **`/(root)`**, depois clique em **Save**.
6. Quando a publicação estiver disponível, abra:

   **`https://ayrton-marquezin-venancio.github.io/controle-prazos-movimentacoes-processuais/`**

7. Teste filtros, navegação e gráficos no endereço publicado. Só então mantenha o link em destaque no topo do README.

**Como funciona o redirecionamento:** o `index.html` encaminha automaticamente o visitante ao arquivo HTML do dashboard, no mesmo site. O visitante vê o dashboard no **Google Chrome ou em qualquer navegador compatível**, e não a visualização de código/JSON do GitHub. **O site é hospedado pelo GitHub Pages, não pelo Google.** Sua aparição na pesquisa do Google depende de indexação posterior e não é automática.

> Importante: embora a interface não mostre o JSON, os dados incorporados ao HTML são públicos e podem ser inspecionados no código-fonte. Por isso, a hospedagem deve ser utilizada apenas com bases apropriadas para divulgação — aqui, dados sintéticos.

## 9. Limitações e cuidados de interpretação

- **Dados fictícios:** não é possível tirar conclusões sobre um Ministério Público real a partir desta base.
- **Recorte temporal:** a versão do dashboard representa os dados e a referência de **07/10/2026**; não recebe atualizações de sistemas externos.
- **Prazos didáticos:** os cálculos consideram diferenças entre datas, sem implementar calendários forenses, feriados ou regras legais de contagem de prazos.
- **Inconsistências:** flags indicam hipóteses para revisão; não comprovam erro material, negligência ou irregularidade.
- **Setores:** volume e taxa de vencimentos ajudam a identificar pontos para investigação, mas não são, isoladamente, uma avaliação justa de desempenho.
- **Score:** a fila de atenção usa pesos explícitos e didáticos, não previsão estatística ou classificação jurídica oficial.
- **Histórico:** evolução de aberturas mensais não equivale à evolução histórica do estoque de procedimentos ativos ou vencidos.

## 10. O que este projeto demonstra

A principal entrega não foi apenas um gráfico ou uma interface. O trabalho mostra como estruturar um problema de negócio a partir de uma base imperfeita, definir indicadores interpretáveis, preservar rastreabilidade, escolher visualizações apropriadas e transformar resultados em uma ferramenta de consulta.

O processo também demonstra o uso criterioso de **engenharia de prompts**, com IA generativa aplicada à construção e revisão de componentes, sempre orientada por objetivos, restrições e checagens de consistência.

**Projeto demonstrativo para fins acadêmicos e de portfólio.**
