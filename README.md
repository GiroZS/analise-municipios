#  Clustering de Municípios Brasileiros por Perfil Econômico

Análise não-supervisionada dos **5.570 municípios brasileiros** com base na composição setorial do PIB, utilizando K-Means para identificar perfis econômicos distintos.

---

##  Contexto

O Brasil é um país de contrastes econômicos profundos. Municípios industriais convivem com pequenas cidades do interior cuja economia depende quase inteiramente do gasto público. Este projeto aplica técnicas de **Machine Learning não-supervisionado** para agrupar automaticamente os municípios por similaridade econômica, revelando padrões que análises tradicionais dificilmente capturam.
Esse projeto foi feito como estudo das tecnologias e aplicação de análise de dados.

**Fonte dos dados:** [Base dos Dados](https://basedosdados.org) — tabela `br_ibge_pib_municipio`, ano de referência **2021**.

---

##  Objetivo

Identificar grupos de municípios com perfis econômicos semelhantes a partir da participação percentual de cada setor no Valor Adicionado Bruto (VAB):

- Agropecuária
- Indústria
- Serviços
- Administração Pública

---

##  Tecnologias

![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=flat&logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-150458?style=flat&logo=pandas&logoColor=white)
![Scikit-learn](https://img.shields.io/badge/Scikit--learn-F7931E?style=flat&logo=scikit-learn&logoColor=white)
![GeoPandas](https://img.shields.io/badge/GeoPandas-139C5A?style=flat)
![Folium](https://img.shields.io/badge/Folium-77B829?style=flat)
![Jupyter](https://img.shields.io/badge/Jupyter-F37626?style=flat&logo=jupyter&logoColor=white)

---

##  Estrutura do Projeto

```
clustering-municipios-pib/
│
├── analise.ipynb                         # Análise completa
├── br_bd_diretorios_brasil_municipio.csv # Tabela de tradução dos ids
├── br_ibge_pib_municipio.csv.gz          # Dados do IBGE via Base dos Dados
└── README.md
```

---

##  Metodologia

### 1. Pré-processamento
- Carregamento do CSV comprimido diretamente com `pandas`
- Criação de **features percentuais** (participação de cada setor no PIB total) para eliminar o efeito do tamanho do município
- Remoção de valores nulos

### 2. Seleção de features
```python
features = [
    "pct_va_agropecuaria",
    "pct_va_industria",
    "pct_va_servicos",
    "pct_va_adespss"
]
```

### 3. Normalização
Aplicação de `StandardScaler` para garantir que nenhuma variável domine as distâncias do algoritmo.

### 4. Definição do número de clusters
O valor **K=4** foi escolhido combinando dois critérios:
- **Método do Cotovelo** — identificação do ponto de inflexão da inércia
- **Silhouette Score** — maximização da coesão interna dos clusters

### 5. Clustering
Aplicação de **K-Means** com K=4 sobre os dados normalizados.

### 6. Visualização
- **PCA** (2 componentes) para visualização 2D dos clusters
- **Mapa interativo** com Folium e shapefile do IBGE

---

##  Resultados

### Perfil médio por cluster

| Cluster | % Agro | % Indústria | % Serviços | % Adm. Pública | PIB Mediano | Nº Municípios |
|---------|--------|-------------|------------|----------------|-------------|---------------|
| 0  Polos Industriais | 9,3% | 43,0% | 24,3% | 13,3% | R$ 789k | 669 |
| 1  Dependentes do Governo | 15,6% | 5,5% | 25,0% | 48,8% | R$ 108k | 1.783 |
| 2  Rurais / Agrícolas | 46,3% | 6,7% | 21,7% | 20,0% | R$ 221k | 1.587 |
| 3  Urbano-Serviços | 11,6% | 14,2% | 43,1% | 20,2% | R$ 703k | 1.530 |

### Interpretação dos clusters

- ** Cluster 0 — Polos Industriais:** Menor grupo em número de municípios, mas com o maior PIB mediano. Reflete a concentração industrial nas regiões Sul e Sudeste.

- ** Cluster 1 — Dependentes do Governo:** O maior cluster do Brasil, com quase 1.800 municípios cuja economia depende majoritariamente do gasto público. PIB mediano 7x menor que os polos industriais — sinal de alta vulnerabilidade fiscal e baixa diversificação econômica.

- ** Cluster 2 — Rurais/Agrícolas:** Agropecuária responde por quase metade do VAB. Típico de municípios do Centro-Oeste e interior do Sul.

- ** Cluster 3 — Urbano-Serviços:** Economia orientada a serviços privados, comércio e serviços financeiros. Inclui capitais e cidades médias desenvolvidas.

### Análise dos componentes principais (PCA)

| Componente | Interpretação | Variância Explicada |
|---|---|---|
| PC1 | Urbanização: indústria/serviços (+) vs agropecuária (-) | ~60% |
| PC2 | Dependência do setor público (-) vs agropecuária independente (+) | ~25% |

---

##  Mapa Interativo

O arquivo `mapa_clusters.html` contém um mapa interativo de todos os municípios brasileiros coloridos por cluster. Passe o mouse sobre qualquer município para ver seu nome e cluster.

> Para visualizar: abra o arquivo `mapa_clusters.html` diretamente no navegador.

---

##  Limitações

- A análise usa dados de **um único ano (2021)** — não é possível inferir tendências temporais
- **Tamanho populacional** não foi incluído nas features, portanto municípios de portes muito diferentes podem estar no mesmo cluster
- K-Means assume **clusters de formato esférico** — algoritmos como DBSCAN ou Gaussian Mixture poderiam capturar estruturas mais complexas

---

##  Próximos Passos

- [ ] Análise temporal: verificar se municípios migram de cluster entre 2010 e 2021
- [ ] Cruzar com dados de IDH (Atlas Brasil/PNUD) para avaliar se perfil econômico prediz bem-estar social
- [ ] Testar algoritmos alternativos: DBSCAN, Hierarchical Clustering
- [ ] Adicionar dados populacionais para calcular PIB per capita por cluster

---

## Como Reproduzir

### 1. Clone o repositório
```bash
git clone https://github.com/GiroZS/analise-municipios.git
cd analise-municipios
```

### 2. Instale as dependências
```bash
pip install pandas scikit-learn geopandas folium matplotlib seaborn
```

### 3. Baixe os dados
- **PIB Municipal:** [Base dos Dados](https://basedosdados.org) → `br_ibge_pib_municipio`
- **Shapefile:** [IBGE](https://geoftp.ibge.gov.br/organizacao_do_territorio/malhas_territoriais/malhas_municipais/municipio_2021/Brasil/BR/)

### 4. Execute o notebook
```bash
jupyter notebook analise.ipynb
```

---

##  Fonte dos Dados

- **PIB Municipal:** IBGE, via [Base dos Dados](https://basedosdados.org)
- **Malha Municipal:** IBGE — [Malhas Territoriais 2021](https://geoftp.ibge.gov.br/organizacao_do_territorio/malhas_territoriais/)
