# Sistema de Recomendação de Filmes

Sistema de recomendação de filmes desenvolvido com técnicas de Aprendizado de Máquina, utilizando filtragem colaborativa baseada em similaridade entre itens sobre o dataset MovieLens.

## Descrição

O sistema recebe como entrada o identificador de um usuário e retorna uma lista personalizada de filmes recomendados com base em seu histórico de avaliações. A abordagem é a **filtragem colaborativa baseada em itens com fatoração de matrizes (SVD)**.

### Como funciona

O dataset tem 99,74% de esparsidade: cada usuário avaliou menos de 0,3% dos filmes. Isso torna a similaridade direta entre filmes instável (dois filmes raramente compartilham avaliadores em comum). A solução é usar **SVD (Decomposição em Valores Singulares)** para comprimir a matriz usuário × filme em 50 "dimensões ocultas" (fatores latentes) que representam conceitos como "gosta de animação", "prefere drama", etc.

**Fluxo do modelo:**

```
ratings do usuário
  → Matriz usuário × filme (scipy.sparse)
  → TruncatedSVD (50 fatores latentes)
  → Vetor latente denso para cada filme
  → Similaridade de cosseno entre filmes
  → Top-N recomendações
```

**Exemplo:** suponha que o Usuário 42 avaliou bem Toy Story (5.0) e Shrek (4.5).

1. O modelo encontra os filmes mais similares a Toy Story no espaço latente: Finding Nemo, Monsters Inc., A Bug's Life...
2. Faz o mesmo para Shrek: Kung Fu Panda, Madagascar...
3. Pondera a similaridade pela nota do usuário (filmes bem avaliados influenciam mais)
4. Remove filmes já assistidos e retorna o top-10:

| Recomendação | Score |
|---|---|
| Finding Nemo | 0.91 |
| Monsters Inc. | 0.88 |
| Kung Fu Panda | 0.82 |
| ... | ... |

## Dataset

O projeto utiliza o dataset [Movie Recommendation System](https://www.kaggle.com/datasets/parasharmanas/movie-recommendation-system) disponível no Kaggle, contendo:

| Arquivo | Descrição | Colunas |
|---|---|---|
| `ratings.csv` | Avaliações dos usuários | `userId`, `movieId`, `rating`, `timestamp` |
| `movies.csv` | Informações dos filmes | `movieId`, `title`, `genres` |

> Os arquivos de dados **não estão incluídos** no repositório devido ao tamanho. Faça o download em [kaggle.com/datasets/parasharmanas/movie-recommendation-system](https://www.kaggle.com/datasets/parasharmanas/movie-recommendation-system) e coloque os arquivos na pasta `data/`.

## Estrutura do Projeto

```
├── data/
│   ├── ratings.csv              # Dados brutos (não versionados)
│   ├── movies.csv               # Dados brutos (não versionados)
│   └── processed/               # Dados tratados (não versionados)
│       ├── ratings_clean.csv    # Ratings sem duplicatas + coluna date
│       ├── movies_clean.csv     # Filmes com pelo menos 1 avaliação
│       └── ratings_sample.csv  # 10% dos ratings (para prototipagem)
├── notebooks/
│   ├── 01_exploracao.ipynb      # Análise exploratória
│   ├── 02_preprocessamento.ipynb # Limpeza e geração dos dados processados
│   └── 03_modelo.ipynb          # Modelo SVD + recomendação
├── venv/                        # Ambiente virtual Python (não versionado)
├── requirements.txt
├── .gitignore
└── README.md
```

## Instalação

**1. Clone o repositório:**
```bash
git clone <url-do-repositorio>
cd aprendizado_de_maquina
```

**2. Crie e ative o ambiente virtual:**
```bash
python3 -m venv venv
source venv/bin/activate      # Linux/Mac
venv\Scripts\activate         # Windows
```

**3. Instale as dependências:**
```bash
pip install -r requirements.txt
```

**4. Registre o kernel no Jupyter:**
```bash
python -m ipykernel install --user --name=recomendacao_filmes --display-name "Python (recomendacao_filmes)"
```

**5. Adicione os dados na pasta `data/` e abra os notebooks:**
```bash
jupyter notebook notebooks/
```

## Dependências

| Pacote | Uso |
|---|---|
| `pandas` | Manipulação do dataset |
| `numpy` | Operações matriciais |
| `scikit-learn` | Métricas e modelos |
| `scipy` | Matrizes esparsas e similaridade |
| `matplotlib` / `seaborn` | Visualizações |
| `jupyter` | Ambiente de desenvolvimento |
| `tqdm` | Barras de progresso |

## Tecnologias

- Python 3
- Jupyter Notebook
- SVD via `TruncatedSVD` (scikit-learn) + similaridade de cosseno (scipy)
- Filtragem Colaborativa Baseada em Itens
- Dataset MovieLens (25M avaliações)
