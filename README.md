Configurar o Banco de Dados

No Azure, crie um Banco de Dados SQL ou use o Cosmos DB (dependendo da sua preferência por SQL ou NoSQL).
Crie uma tabela/coleção para armazenar as informações do catálogo, incluindo campos como Id, Título, Descrição, Gênero, Ano, Avaliação, etc.
Parte 2: Criar a Função do Azure
Iniciar um Novo Projeto de Funções

Abra o Visual Studio Code e crie uma nova pasta para o projeto.
No terminal integrado, execute:
func init GerenciadorCatalogo --python
cd GerenciadorCatalogo
func new
Escolha HTTP trigger como tipo de função e dê um nome (por exemplo, GerenciarCatalogo).
Implementar a Lógica da Função

Abra o arquivo da função criada (por exemplo, GerenciarCatalogo/__init__.py) e implemente a lógica para interagir com o banco de dados.
Aqui está um exemplo básico de como poderia ser a função:

import logging
import azure.functions as func
import pyodbc

def main(req: func.HttpRequest) -> func.HttpResponse:
    logging.info('Python HTTP trigger function processed a request.')

    # Conexão com o banco de dados
    conn = pyodbc.connect('Driver={ODBC Driver 17 for SQL Server};'
                          'Server=<SEU_SERVER>;'
                          'Database=<SEU_DATABASE>;'
                          'UID=<SEU_USER>;'
                          'PWD=<SUA_PASSWORD>;')
    cursor = conn.cursor()

    # Lógica para adicionar, listar ou deletar itens do catálogo
    if req.method == 'POST':
        # Adicionar um novo item ao catálogo
        data = req.get_json()
        cursor.execute("INSERT INTO Catalogo (Titulo, Descricao, Genero, Ano, Avaliacao) VALUES (?, ?, ?, ?, ?)",
                       data['Titulo'], data['Descricao'], data['Genero'], data['Ano'], data['Avaliacao'])
        conn.commit()
        return func.HttpResponse("Item adicionado com sucesso", status_code=201)

    elif req.method == 'GET':
        # Listar todos os itens do catálogo
        cursor.execute("SELECT * FROM Catalogo")
        rows = cursor.fetchall()
        items = [{"Id": row[0], "Titulo": row[1], "Descricao": row[2], "Genero": row[3], "Ano": row[4], "Avaliacao": row[5]} for row in rows]
        return func.HttpResponse(json.dumps(items), mimetype="application/json")

    return func.HttpResponse("Método não suportado", status_code=405)
Parte 3: Configurar o Banco de Dados
Criar Tabela/Modelo

Execute um script SQL para criar uma tabela no seu banco de dados:
CREATE TABLE Catalogo (
    Id INT PRIMARY KEY IDENTITY(1,1),
    Titulo NVARCHAR(100),
    Descricao NVARCHAR(255),
    Genero NVARCHAR(50),
    Ano INT,
    Avaliacao FLOAT
);
Configurar Conexões

Certifique-se de que as configurações de conexão estão corretas em sua função (substituindo <SEU_SERVER>, <SEU_DATABASE>, <SEU_USER>, <SUA_PASSWORD>).
Parte 4: Testar Localmente
Executar a Função Localmente
No terminal, execute:
func start
Use o Postman ou um navegador para testar os endpoints definidos (ex: http://localhost:7071/api/GerenciarCatalogo).
Parte 5: Publicar no Azure
Criar um App Service para Funcões

No portal Azure, crie um novo Azure Function App.
Publicar a Função

No Visual Studio Code, abra o comando de palete (Ctrl + Shift + P) e escolha Azure Functions: Deploy to Function App.
Siga as instruções para escolher a assinatura do Azure e o Function App que você criou.
Testar a Função no Azure

Após a publicação, você receberá uma URL para acessar sua função. Teste os endpoints usando o Postman ou navegador.
Parte 6: Monitoramento e Melhoria
Habilitar o Application Insights

No portal do Azure, habilite o Application Insights para monitorar logs, desempenho e erros da sua função.
Melhorias Futuras

Considere implementar autenticação, paginação para listagem de resultados, ou funcionalidades adicionais como atualização e deleção de itens no catálogo.
