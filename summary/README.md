# Primeiros passos usando o Flask

Primeiramente, fazemos a importação do flask para o nosso ambiente `from flask import Flask` e inicializamos uma varíavel `app`

```python
from flask import Flask

app = Flask(__name__)

```
De forma rebuscada esse comando diz que estamos criando um aplicativo flask com o nome que nosso arquivo tem (main.py).

Depois iremos adicionar o comando `@app.route()`. Esse comando define uma rota (uma URL) e liga essa rota a uma função.

Por fim, utilizaremos o `app.run()` que serve para colocarmos o site no ar 

```python
#Esse '/' serve somente para dizer que essa será a homepage do site
@app.route("/") 
           
def homepage():
    return "FakePinterest -  Meu primeiro site no ar"

#Usamos uma verificação para se o arquivo 'main.py' for executado, execute o app.run()
if __name__ == "__main__":
    app.run(debug=True) #debug=True diz que as alterações que fizermos no código irão automaticamente para o site enquanto ele estiver rodando.
```

# Rotas e páginas dinâmicas

Com um volume maior de usuários é importante que o site responda de maneira específica para cada pessoa que possuir um perfil e não de maneira padronizada como ver um "bem vindo ao site tal" na homepage. Com páginas dinâmicas iremos montar uma estrutura em que o conteúdo do site apresentará de forma diferente para cada usuário. 

## Variáveis

Para começarmos precisamos fazer pequenas alterações na nossa rota de perfil. Agora ela será acompanhada por uma variável e assim a rota vai mudar dependendo do usuário que acessou 

```python
@app.route("/perfil/<usuario>") # Variável representada por "<>"
def perfil(usuario): # Se criarmos uma variável na rota precisamos que a mesma variável seja usada como parâmetro
    return render_template("perfil.html", usuario=james) # Isso serve para que possamos passar o valor da variavel para nosso código html
```
Quando usamos o `render_template()` ela permite que passemos parâmetros para o html, e para acessarmos, usamos `{{variavel}}` 

```html
<!doctype html>
<html lang="pt-br">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>FakePinterest</title>
  </head>
  <body>
    <p>FakePinterest -  Meu primeiro site no ar</p>
    {{usuario}}
  </body>
</html>
```
### Url_for
Por questões de praticidade queremos que quando o usuário entrar na página de perfil ele possa voltar para a homepage sem ter que escrever na url. Iremos colocar um link para quando clicado automaticamente a página muda para a homepage.

```html
<h1>Pagina perfil do: {{usuario}}</h1>
<a href="/">Sair</a>
```
O link está referenciando a url que definirmos no `app.route()` e desse jeito funcionaria normalmente. Contudo, se por algum motivo quisermos mudar a url, por "homepage", "pagina principal" ou algo assim este link não funcionaria mais. 


Um jeito mais inteligente de resolvermos esse futuro problema é utilizando o `url_for()` ele é uma função do flask que permite redirecionar para um link específico, só que ao invés de usar o texto que está no app.route, ele utiliza o nome da função, que é algo improvável de mudarmos em um futuro distante. 

Apenas trocando a referencia do link:

```html
<h1>Pagina perfil do: {{usuario}}</h1>
<a href="{{ url_for('homepage') }}">Sair</a>
```
Agora se mudarmos o nome da rota o código não irá quebrar, pois o link está utilizando o nome da função como referência. 

# Conectando as páginas html

No site que estamos criando podemos ver que temos por enquanto duas páginas, a principal e a de usuários, e para cada página precisamos montar a estrutura html (head,body,nav). Conforme progredimos na criação do site vamos precisar criar mais páginas, porém muitas delas vão seguir a mesma estrutura da página principal, talvez a mesma cor, mesma barra de navegação, assim ter que montar uma estrutura html para cada uma é um trabalho desnecessário.

Para resolvermos esse conflito o flask nos da uma ferramenta importante, com ela poderemos criar uma base única html, ou seja todas as páginas html que formos criando vão seguir a estrutura principal e mudamos o que acharmos necessária ao invés de ter que criar um código html inteiro do zero repetindo os mesmos comandos. 

1. Definimos a nossa base única, será a homepage do nosso site
2. Selecionar o que queremos editar utilizando blocos: 

Sintaxe:
```
{% block nome_do_bloco %}
//Espaço onde ficará o conteúdo//
{% enblock %}
```
Exemplo:
```html 
<title>
    {% block titulo %}
    FakePinterest
    {% endblock %}
</title>
```
3. Em outra página vamos puxar o conteúdo da estrutura principal: `{%extends "nome_do arquivo" %}`
4. Agora na página de usuários vamos editar o título e o body que selecionamos 

```html
{% extends "index.html" %}

{% block titulo %}
Perfil - {{ usuario }}
{% endblock %}

{% block body %}
<body>
    <h1>Pagina perfil do: {{usuario}}</h1>
    <a href="{{ url_for('homepage') }}">Sair</a>
</body>
{% endblock %}
```

# Montagem do banco de dados 

Iremos começar criando nosso banco de dados no arquivo `__init__` 
```python
from flask_sqlalchemy import SQLAlchemy

app = __name__
app.config["SQLALCHEMY_DATABASE_URI"] = "sqlite:///comunidade.db"

database = SQLAlchemy(app)


```
Vamos agora criar um arquivo python que servirá para criar o banco de dados. Esse arquivo não vai precisar existir no final do projeto, mas vamos criá-lo aqui para o banco já existir e podermos visualizar-lo melhor.

```python
from fakepinterest import database, app


with app.app_context():
    database.create_all()
```

## Criar as tabelas
O banco de dados que criamos ele vem completamente vazio, para preencher-lo precisamos inicialmente importar as tabelas que ele terá e definiremos isso no arquivo `models.py`. 

Então agora vamos precisar criar essas tabelas:

- Criamos as classes que servirão como tabelas para o banco de dados.
- As classes que definimos serão subclasses do `database.Model`.
- database.Model serve para criar a classe no formato que o nosso `database` entende como uma tabela.
```python
from fakepinterest import database

class Usuario(database.Model):
    pass

class Foto(database.Model):
    pass
```
- Agora definimos os objetos das classes como colunas do banco de dados
- Definimos nos parênteses as regras que as colunas terão
- Dizemos as regras que cada coluna irá ter
```python
class Usuario(database.Model):
    id = database.Column(database.Integer, primary_key=True) # Ele vai ser um número inteiro e possui um identificador unico. Não aceita valores nulos ou duplicados.
    username = database.Column(database.String, nullable=False) # Ele será uma string e não pode ser nulo
    email = database.Column(database.String, nullable=False, unique=True) # Ele será uma string e não pode ser nulo e duplicado
    senha = database.Column(database.String, nullable=False) # Ele será uma string e não pode ser nulo
    fotos = database.relationship() # Esse objeto será uma instância da classe Foto

class Foto(database.Model):
    id = database.Column(database.Integer, primary_key=True)
    imagem = database.Column(database.String, default="default.png")
    data_criacao = database.Column(database.DataTime, nullable=false, default=datatime.utcnow()) # Informa o exato horario que a imagem foi criada.
    id_usuario = database.Column()  # Para dizer qual usuário postou a foto
```

>[!NOTE]
> O banco de dados será uma string pois a informação que iremos armazenar será o local de onde a imagem está dentro do sistema. Imagens que iremos armazenar na pasta `static` 

Vamos trabalhar melhor agora o objeto fotos que definimos ser uma instância. Para caso queiramos procurar no nosso banco de dados por uma foto específica de um determinado usuário, não precisaremos buscar por todas as fotos até encontrar a foto com o id do usuário escolhido. É muito mais fácil fazer com que o banco de dados busque um usuário e verifique todas as fotos que ele possui, é para isso que serve o `relationship()`

- Passamos o nome da classe com qual ele irá se relacionar
```python
fotos = database.relationship("Foto", backref="usuario", lazy=True) #backref serve para pegarmos uma foto e encontrar um usuário. lazy=True otimizar busca de informação no banco de dados
```

Agora por último temos acrescentar mais informações ao id_usuario. Ele será uma `Foreign key`, uma coluna de uma tabela que faz referência à uma chave primária, neste caso à coluna id de Usuários.
```python
id_usuario = database.Column(database.Integer, database.ForeignKey('usuario.id'),  nullable=False)
```



# Criando o sistema de Login

Utilizaremos a ferramenta do flask, o *flask login* que fará esse gerenciamento para nós e também para outros quesitos de segurança iremos baixar as ferramentas abaixo: 

> pip install flask-login flask-bcrypt

Flask-login: Gerencia as senhas e logins
Flask-bcrypt: Ele fará a criptografia das senhas e logins

Importaremos as ferramentas que instalamos e depois inicalizaremos 
```python 
from flask_login import LoginManager
from flask_bcrypt import Bcrypt

app.config["SECRET_KEY"] = "" # Aqui definimos uma chave de segurança

bcrypt = Bcrypt(app)
login_manager = LoginManager(app)
login_manager.login_view = "homepage" # Colocamos a route que o usuario será mandado quando não estiver logado
```

Agora no `models.py` vamos gerenciar a estrutura de login

- Importamos a biblioteca UserMixin
- A classe Usuario receberá o UserMixin como um novo parâmetro
- Importamos o login manager que definimos e ela pedirá por uma função obrigatória
```python 
from fakepinterest import database, login_manager
from flask_login import UserMixin

@load_manager.user_loader
def load_usuario(id_usuario):
    return Usuario.query.get(int(id_usuario))

class Usuario(database.Model, UserMixin): # UserMixin é quem diz qual a classe que vai gerenciar a estrutura de logins
```
Uma pequena alteração antes de prosseguirmos: 

No arquivo routes iremos colocar o decorator `@login_required` que serve basicamente para restringir o acesso àquela determinada página de usuários que ainda não estão logados.  
```python
from flask_login import login_required

@app.route("/perfil/<usuario>")
@login_required
def perfil(usuario):
    return render_template("perfil.html", usuario=usuario)
```

# Criando formulários de login

Agora que criamos o gerenciamento de logins precisaremos de fato criar os formulários para que os usuários possam se registrar no nosso site. 