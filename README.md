# API de Recomendações

## Visao Geral
Esta API, desenvolvida com FastAPI, fornece funcionalidades para buscar recomendações personalizadas para alunos com base em seu perfil e interesses. A API utiliza um banco de dados PostgreSQL para armazenar as recomendações e a biblioteca FAISS para realizar buscas eficientes em embeddings gerados a partir de descrições de oportunidades.

## Configuracao do Banco de Dados
A API usa PostgreSQL como banco de dados. A conexão é definida pelas variáveis de ambiente extraídas de um arquivo `.env`.

```python
postgres_url = f"postgresql+psycopg2://{DB_USERNAME}:{DB_PASSWORD}@{DB_HOST}:{DB_PORT}/{DB_DATABASE}"
engine = create_engine(postgres_url, echo=True)
```

A criação das tabelas ocorre na inicialização da aplicação:

```python
def create_db_and_tables():
    SQLModel.metadata.create_all(engine)
```

## Schemas

### Recomendação
```python
class Recomendacao(SQLModel, table=True):
    id: Optional[int] = Field(default=None, primary_key=True)
    id_usuario: str
    id_oportunidade: int

    metadata = MetaData(schema="recomendacao")
```

### AlunoId
```python
class AlunoId(BaseModel):
    id_aluno: str
```

## Endpoints

### `GET /`
**Descricao:** Endpoint raiz para verificar o status da API.
**Resposta:**
```json
{
  "hello": "world"
}
```

### `POST /search`
**Descricao:** Busca recomendações para um aluno com base em seu perfil.

**Input:**
```json
{
  "id_aluno": "12345"
}
```

**Fluxo:**
1. Recupera informacoes do aluno via `HOST_GATEWAY`.
2. Gera uma consulta baseada nos interesses e disponibilidade do aluno.
3. Realiza a busca usando FAISS.
4. Filtra oportunidades com base na localização.
5. Salva as recomendações no banco de dados.

**Resposta:**
```json
{
  "text": [
    {
      "nome": "Nome da Oportunidade",
      "cidade": "Cidade",
      "uf": "Estado",
      "descricao": "Descricao detalhada"
    }
  ]
}
```

### `GET /recomendacoes/`
**Descricao:** Retorna todas as recomendações armazenadas.
**Resposta:**
```json
[
  {
    "id": 1,
    "id_usuario": "12345",
    "id_oportunidade": 987
  }
]
```

### `GET /recomendacoes/{recomendacao_id}`
**Descricao:** Retorna uma recomendação especifica.

**Input:**
- `recomendacao_id` (int) - ID da recomendacao.

**Resposta:**
```json
{
  "id": 1,
  "id_usuario": "12345",
  "id_oportunidade": 987
}
```

### `GET /recomendacoes/aluno/{aluno_id}`
**Descricao:** Retorna todas as recomendações de um aluno especifico.

**Input:**
- `aluno_id` (str) - ID do aluno.

**Resposta:**
```json
[
  {
    "id": 1,
    "id_usuario": "12345",
    "id_oportunidade": 987
  }
]
```

## Funcionalidades Adicionais

### `save_recomendacoes`
Salva recomendacoes no banco de dados com base nos resultados da busca.

### `filter_oportunidades`
Filtra oportunidades com base na localizacao do aluno.

### `create_recomendacao`
Cria uma nova recomendacao no banco de dados.

## Configuracao do FAISS
A API utiliza FAISS para buscar oportunidades similares ao perfil do aluno:

```python
faiss_db = FAISS.load_local("oportunidades_embeddings", embeddings, allow_dangerous_deserialization=True)
```

## Configuracao CORS
A API permite requisicoes de qualquer origem:
```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=['*'],
    allow_credentials=False,
    allow_methods=['*'],
    allow_headers=['*'],
)
```
