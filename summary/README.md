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