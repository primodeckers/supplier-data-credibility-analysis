# Regras para Resolução do Exercício: Análise de Credibilidade dos Dados

Este documento estabelece as diretrizes, regras de negócio e passos técnicos necessários para a realização do exercício de **Análise de Credibilidade dos Dados** cadastrais de fornecedores, avaliando diferentes dimensões de qualidade de dados.

---

## 1. Visão Geral e Objetivos

- **Objetivo Geral:** Avaliar a confiabilidade e integridade cadastral de fornecedores por meio da aplicação prática de métricas e dimensões de **Qualidade de Dados** (*Data Quality*).
- **Dimensões Avaliadas:**
  - **Completude (*Completeness*)**
  - **Unicidade (*Uniqueness*)**
  - **Consistência (*Consistency*)**
  - **Similaridade e Conformidade Textual (*Similarity & Accuracy*)**

---



## 2. Recursos e Fontes de Dados

- **Repositório de Arquivos (Google Drive):** [Acessar Pasta de Dados, Slides e Notebooks](https://drive.google.com/drive/folders/1DogwuPtto-VvPKHPoqKaQ9hc-sAqnGOe?usp=drive_link)
- **Arquivos Utilizados:**
  1. `fornecedores_transf.pkl`: Base cadastral principal contendo os registros dos fornecedores (CNPJ, CPF, UF, CNAE, etc.).
  2. `municipios_transf.pkl`: Base de referência com municípios e respectivas UFs (Unidades Federativas) do Brasil.
  3. `cnaes_transf.pkl`: Base interna de códigos e descrições de CNAEs associados aos cadastros.
  4. `CNAERFB.CSV`: Tabela oficial de referência de CNAEs disponibilizada pela Receita Federal do Brasil (RFB).

---



## 3. Requisitos de Ambiente e Dependências

Para a execução no Jupyter Notebook Python, recomenda-se a instalação dos seguintes pacotes:

```bash
pip install pandas python-Levenshtein
```

> **Alternativa para Levenshtein:** Caso não seja possível instalar `python-Levenshtein`, utilize o módulo nativo `difflib` ou bibliotecas como `thefuzz` / `fuzzywuzzy` / `nltk`.

---



## 4. Passo a Passo e Regras de Negócio



### Etapa 1: Carregamento de Dados

- **Ação:** Importar o arquivo `fornecedores_transf.pkl` e instanciar um `DataFrame` pandas (`df_fornecedores`).
- **Regras / Boas Práticas:**
  - Utilizar `pd.read_pickle("fornecedores_transf.pkl")`.
  - Inspecionar a estrutura inicial: `.info()`, `.head()`, `.shape` e nomes das colunas.

---



### Etapa 2: Dimensão de Completude (*Completeness*)

- **Regra de Negócio:**
  - Todo fornecedor **deve possuir obrigatoriamente pelo menos um documento de identificação fiscal preenchido**: `CNPJ` **OU** `CPF`.
- **Critério de Inconsistência:**
  - Registros onde tanto o `CNPJ` quanto o `CPF` estão nulos (`NaN`, `None` ou cadeia de caracteres vazia `""`).
- **Validação Esperada:**
  1. Identificar a quantidade e a porcentagem de registros sem documento algum.
  2. Exibir a listagem dos fornecedores que violam essa regra.

---



### Etapa 3: Dimensão de Unicidade (*Uniqueness*)

- **Regra de Negócio 3.1 — Duplicidade de CNPJ:**
  - Não deve haver dois ou mais fornecedores com o mesmo número de `CNPJ`.
  - *Validação:* Identificar e listar CNPJs duplicados (desconsiderando valores nulos).
- **Regra de Negócio 3.2 — Exclusividade de Tipo de Pessoa (PF vs. PJ):**
  - Um fornecedor deve ser cadastrado exclusivamente como **Pessoa Jurídica (CNPJ)** ou como **Pessoa Física (CPF)**, **nunca ambos simultaneamente**.
  - *Critério de Inconsistência:* Registros em que as colunas `CNPJ` e `CPF` estão preenchidas ao mesmo tempo no mesmo registro.
- **Validação Esperada:**
  1. Contagem e exibição dos registros com CNPJ duplicado.
  2. Contagem e exibição dos registros com preenchimento simultâneo de CNPJ e CPF.

---



### Etapa 4: Dimensão de Consistência (*Consistency*)

- **Regra de Negócio:**
  - Todas as UFs (Unidades Federativas) informadas na base de fornecedores devem existir e ser válidas no conjunto oficial de UFs brasileiras.
- **Procedimento:**
  1. Importar o arquivo `municipios_transf.pkl` em um DataFrame (`df_municipios`).
  2. Extrair o conjunto/lista de UFs únicas e válidas presentes em `df_municipios`.
  3. Comparar as UFs presentes em `df_fornecedores` contra as UFs válidas de `df_municipios`.
- **Validação Esperada:**
  1. Identificar registros de fornecedores com UFs não encontradas na base de municípios (ex.: siglas incorretas, nulos, erros de digitação ou códigos inválidos).
  2. Exibir o percentual de conformidade da base para a coluna de UF.

---



### Etapa 5: Dimensão de Similaridade e Conformidade de CNAE

- **Regra de Negócio:**
  - O código de Classificação Nacional de Atividades Econômicas (CNAE) deve possuir a mesma descrição padronizada da Receita Federal.
- **Procedimento:**
  1. Importar os arquivos `cnaes_transf.pkl` (`df_cnaes_transf`) e `CNAERFB.CSV` (`df_cnaes_rfb`).
  2. Padronizar a chave de relacionamento (código CNAE) em ambos os DataFrames (tipo de dado string/numérico e remoção de caracteres especiais como pontos, barras ou traços, se houver).
  3. Realizar a junção (*merge/join*) entre as duas bases utilizando o código CNAE como chave.
  4. Comparar as colunas de descrição e filtrar os registros em que o mesmo código CNAE apresenta **descrições diferentes**.
- **Validação Esperada:**
  - Tabela com os códigos CNAE divergentes contendo a descrição original da base de fornecedores e a descrição oficial da Receita Federal.

---



### Etapa 6: Grau de Similaridade Textual (Distância de Levenshtein)

- **Regra de Negócio:**
  - Para cada código CNAE com descrição divergente identificado na **Etapa 5**, quantificar a discrepância textual utilizando o cálculo da **Distância de Levenshtein**.
- **Procedimento:**
  1. Aplicar o algoritmo de Levenshtein entre a string da descrição de `cnaes_transf` e a descrição oficial de `CNAERFB`.
  2. (Opcional/Recomendado) Calcular também o índice de similaridade normalizado (por exemplo, razão percentual de similaridade).
- **Validação Esperada:**
  - Exibir uma tabela com as colunas:
    - `Codigo_CNAE`
    - `Descricao_Fornecedor`
    - `Descricao_RFB`
    - `Distancia_Levenshtein`
    - `Grau_Similaridade_%` (opcional)

---



## 5. Estrutura Sugerida para o Notebook

Para garantir clareza, reprodutibilidade e apresentação executiva, estruture o notebook da seguinte forma:

```text
├── 1. Configuração do Ambiente e Importação de Bibliotecas
├── 2. Carregamento dos Dados
├── 3. Análise de Completude (CNPJ / CPF)
├── 4. Análise de Unicidade (Duplicatas e Conflito PF/PJ)
├── 5. Análise de Consistência Geográfica (UFs)
├── 6. Cruzamento e Análise de Conformidade de CNAEs
├── 7. Cálculo de Similaridade Textual (Distância de Levenshtein)
└── 8. Conclusão e Sumário Executivo de Qualidade dos Dados
```

---



## 6. Checklist de Validação da Entrega

- [x] Todas as 4 bases de dados foram carregadas com sucesso.
- [x] Foram contabilizados os registros sem CPF e sem CNPJ.
- [x] Foram listados CNPJs duplicados e registros com CPF e CNPJ simultâneos.
- [x] Foi verificado se todas as UFs de fornecedores pertencem à lista de UFs de municípios.
- [x] Foi realizado o cruzamento dos CNAEs entre a base do exercício e a tabela da RFB.
- [x] A distância de Levenshtein foi calculada para todos os CNAEs com descrições divergentes.
- [x] O notebook possui comentários explicativos e células Markdown contextualizando cada resultado.