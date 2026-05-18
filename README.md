# Sistema de Recomendação de Filmes

Sistema de recomendação de filmes desenvolvido com técnicas de Aprendizado de Máquina, utilizando filtragem colaborativa baseada em similaridade entre itens sobre o dataset MovieLens.

## Descrição

O sistema recebe como entrada o identificador de um usuário e retorna uma lista personalizada de filmes recomendados com base em seu histórico de avaliações. A abordagem principal é a **filtragem colaborativa baseada em itens** (*item-based collaborative filtering*), que encontra filmes similares àqueles que o usuário já avaliou positivamente.

## Dataset

O projeto utiliza o dataset [Movie Recommendation System](https://www.kaggle.com/datasets/parasharmanas/movie-recommendation-system) disponível no Kaggle, contendo:

| Arquivo | Descrição | Colunas |
|---|---|---|
| `ratings.csv` | Avaliações dos usuários | `userId`, `movieId`, `rating`, `timestamp` |
| `movies.csv` | Informações dos filmes | `movieId`, `title`, `genres` |

> Os arquivos de dados **não estão incluídos** no repositório devido ao tamanho. Faça o download em [kaggle.com/datasets/parasharmanas/movie-recommendation-system](https://www.kaggle.com/datasets/parasharmanas/movie-recommendation-system) e coloque os arquivos na pasta `data/`.

## Estrutura do Projeto

```
├── data/                  # Arquivos CSV do dataset (não versionados)
│   ├── ratings.csv
│   └── movies.csv
├── notebooks/             # Notebooks Jupyter com análises e modelos
├── venv/                  # Ambiente virtual Python (não versionado)
├── requirements.txt       # Dependências do projeto
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
- Filtragem Colaborativa (Item-Based)
- Dataset MovieLens
