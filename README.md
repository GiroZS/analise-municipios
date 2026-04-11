# Clustering de Municípios Brasileiros por Perfil Econômico

Análise não-supervisionada dos **5.570 municípios brasileiros** com base na composição setorial do PIB, utilizando K-Means para identificar perfis econômicos distintos e rastrear sua evolução entre 2003 e 2021.

---

## Contexto

O Brasil é um país de contrastes econômicos profundos. Municípios industriais convivem com pequenas cidades do interior cuja economia depende quase inteiramente do gasto público. Este projeto aplica técnicas de **Machine Learning não-supervisionado** para agrupar automaticamente os municípios por similaridade econômica, revelando padrões que análises tradicionais dificilmente capturam. Foi desenvolvido como estudo de análise de dados e clustering aplicado a dados reais de escala nacional.

**Fonte dos dados:** [Base dos Dados](https://basedosdados.org/dataset/fcf025ca-8b19-4131-8e2d-5ddb12492347?table=fbbbe77e-d234-4113-8af5-98724a956943) — tabela `br_ibge_pib_municipio`, anos de referência **2003 a 2021**.

---

## Objetivo

Identificar grupos de municípios com perfis econômicos semelhantes a partir da participação percentual de cada setor no Valor Adicionado Bruto (VAB), e analisar como esses grupos evoluem ao longo do tempo.

Os quatro setores utilizados como features:
- Agropecuária
- Indústria
- Serviços
- Administração Pública

---

## Tecnologias

![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=flat&logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-150458?style=flat&logo=pandas&logoColor=white)
![Scikit-learn](https://img.shields.io/badge/Scikit--learn-F7931E?style=flat&logo=scikit-learn&logoColor=white)
![GeoPandas](https://img.shields.io/badge/GeoPandas-139C5A?style=flat)
![Folium](https://img.shields.io/badge/Folium-77B829?style=flat)
![Jupyter](https://img.shields.io/badge/Jupyter-F37626?style=flat&logo=jupyter&logoColor=white)

---

## Estrutura do Projeto

```
analise-municipios/
│
├── analise.ipynb                         # Análise completa
├── br_bd_diretorios_brasil_municipio.csv # Tabela de tradução dos ids
├── br_ibge_pib_municipio.csv.gz          # Dados do IBGE via Base dos Dados
├── ipea_idh.csv                          # IDH de cada município (2010)
└── README.md
```

---

## Metodologia

### 1. Pré-processamento

- Carregamento do CSV comprimido diretamente com `pandas`
- Criação de **features percentuais**: cada setor foi calculado como `VAB_setor / (VAB_agro + VAB_industria + VAB_servicos + VAB_adespss)`, eliminando o efeito do tamanho do município e evitando distorções causadas pelos impostos líquidos sobre produtos, que fazem o PIB oficial divergir da soma dos VABs setoriais
- Remoção de valores nulos e exclusão de municípios com dados estruturalmente inconsistentes identificados via inspeção manual

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

### 5. Clustering estático (2021)

Aplicação de **K-Means** com K=4 sobre os dados normalizados do ano de referência.

### 6. Clustering temporal (2003–2021)

O K-Means foi aplicado independentemente para cada ano. Como os labels dos clusters são arbitrários a cada execução, foi aplicado um **alinhamento de centroides** entre anos usando o algoritmo húngaro (`scipy.optimize.linear_sum_assignment`), garantindo que o mesmo label represente o mesmo perfil econômico ao longo de toda a série.


### 7. Visualização

- **PCA** (2 componentes) para visualização 2D dos clusters
- **Mapa interativo** com Folium e shapefile do IBGE
- **Heatmaps** de distribuição por UF ao longo dos anos

---

## Resultados

### Perfil dos clusters (ano base: 2003)

| Cluster | Perfil dominante | Característica principal |
|---|---|---|
| Agropecuário | Alto VAB agrícola | Típico do Centro-Oeste e interior do Sul |
| Industrial | Alto VAB industrial | Concentrado no Sul e Sudeste |
| Urbano-Serviços | Alto VAB de serviços privados | Capitais e cidades médias desenvolvidas |
| Dependente do Governo | Alto VAB de administração pública | Interior do Norte e Nordeste |

### Cruzamento com IDH (IPEA, 2010)

| Cluster | IDHM médio | IDHM mediano |
|---|---|---|
| Industrial | 0,705 | 0,713 |
| Urbano-Serviços | 0,712 | 0,719 |
| Agropecuário | 0,671 | 0,682 |
| Dependente do Governo | 0,595 | 0,592 |

O cluster Dependente do Governo apresenta IDHM médio significativamente inferior aos demais, reforçando a relação entre dependência do gasto público e baixo desenvolvimento humano.

### Tendências regionais (2003–2021)

- **Rondônia:** migração quase completa do perfil de serviços para agropecuário ao longo das duas décadas, refletindo a expansão do agronegócio no estado
- **Tocantins:** virada acentuada entre 2019 e 2021, com crescimento expressivo do cluster agropecuário, possivelmente relacionado à expansão do MATOPIBA
- **Espírito Santo:** crescimento do perfil industrial entre 2003 e 2017, com provável influência do pré-sal e do setor industrial do estado
- **Distrito Federal:** 100% no cluster Urbano-Serviços em todos os anos, resultado esperado dado que o DF não possui municípios rurais nem base industrial relevante

### Análise dos componentes principais (PCA)

| Componente | Interpretação | Variância Explicada |
|---|---|---|
| PC1 | Urbanização: indústria/serviços (+) vs agropecuária (-) | ~60% |
| PC2 | Dependência do setor público (-) vs agropecuária independente (+) | ~25% |

---

## Mapa Interativo

O arquivo `mapa_clusters.html` contém um mapa interativo de todos os municípios brasileiros coloridos por cluster. Passe o mouse sobre qualquer município para ver seu nome e cluster.

> Para visualizar: abra o arquivo `mapa_clusters.html` diretamente no navegador.

---

## Limitações

- A feature de tamanho populacional não foi incluída no clustering, portanto municípios de portes muito diferentes podem pertencer ao mesmo grupo
- O cruzamento com IDH está limitado ao ano de 2010, único disponível com cobertura municipal pelo IPEA
- K-Means assume clusters de formato esférico e impõe uma partição sobre dados que não possuem separação natural bem definida
- Dois municípios foram excluídos nos anos de 2013 e 2014 por apresentarem dados inconsistentes na base do IBGE

---

## Como Reproduzir

```bash
git clone https://github.com/GiroZS/analise-municipios.git
cd analise-municipios
pip install pandas scikit-learn geopandas folium matplotlib seaborn scipy
jupyter notebook analise.ipynb
```

**Dados necessários:**
- PIB Municipal: [Base dos Dados](https://basedosdados.org/dataset/fcf025ca-8b19-4131-8e2d-5ddb12492347?table=fbbbe77e-d234-4113-8af5-98724a956943) → `br_ibge_pib_municipio`
- Shapefile: [IBGE — Malhas Territoriais 2021](https://geoftp.ibge.gov.br/organizacao_do_territorio/malhas_territoriais/malhas_municipais/municipio_2021/Brasil/BR/)

---

## Próximos passos

- [] Aumentar o valor de k para descobrir subgrupos econômicos dentro dos clusters

## Fonte dos Dados

- **PIB Municipal:** IBGE, via [Base dos Dados](https://basedosdados.org/dataset/fcf025ca-8b19-4131-8e2d-5ddb12492347?table=fbbbe77e-d234-4113-8af5-98724a956943)
- **Malha Municipal:** IBGE — [Malhas Territoriais 2021](https://geoftp.ibge.gov.br/organizacao_do_territorio/malhas_territoriais/)
- **IDH Municipal:** [IPEA](https://ipeadata.gov.br/Default.aspx) Instituto de Pesquisa Econômica Aplicada