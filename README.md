# Introdução

Vamos criar uma aplicação em tempo real com Node. A natureza non-blocking do Node  faz dele uma boa alternativa para este tipo de aplicação (Chat App). 


# Criando o projeto

Nesta seção vamos dar kick-off no projeto: criando um novo diretório para o Chat App e configurando um webserver básico com express que utilizaremos durante o desenvolvimento do Chat App.

### Criando a estrutura base

**Passo 1** - Crie a estrutura de arquivos conforme apresentado na figura abaixo. Estamos criando todos os arquivos do nosso projeto de forma antecipada, não se preocupe que iremos escrever o conteúdo destes arquivos em breve. 

![](https://lh3.googleusercontent.com/QAMRiZMyC7mrOjReMaZbcHYC53VQI9PNJZsLBrkvMFV86aivI8HtuZ5ycKekdX3LjcaydHAIf5U "Estrutura")

**Passo 2** - Inicialize o npm e instale o Express.

Execute o seguinte comando no terminal:
```
npm init -y
```
A flag -y vai configurar o valor padrão para as configurações básicas do projeto. O resultado deve ser um arquivo **package.json** no seguinte formato:

```json
{
  "name": "chat-app",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

Para instalar o Express basta executar o seguinte comando no terminal:

```
npm install express@4.17.1
```

**Passo 3** - Criar um servidor com Express

```javascript
const express = require('express')
const app = express()
const port = process.env.PORT || 3000

app.listen(port, () => {
	console.log(`Server is up on port ${port}`)
})
```

O código acima importa o express, que é uma função. Em seguinte atribuímos esta função a uma variável de nome **app**. Na terceira linha do código recuperamos a porta das variável de ambiente **PORT** caso exista. Do contrário utilizamos a porta **3000**.

Na linha 5 inicializamos o servidor passando a porta e uma função de callback. A função de callback não é obrigatória. No exemplo do código acima, quando o servidor for iniciado a mensagem *Server is up on port 3000* será mostrada no terminal. Estamos utilizando ES6 template string para imprimir a mensagem no terminal.

**Passo 4** - Configurar o diretório público que usaremos para servir os arquivos estáticos de nossa aplicação.

```javascript
const path = require('path')
const express = require('express')

const app = express()

const port = process.env.PORT || 3000
const publicDirectoryPath = path.join(__dirname, '../public')

app.use(express.static(publicDirectoryPath))

app.listen(port, () => {
	console.log(`Server is up on port ${port}`)
})
```
No código acima primeiro importamos o módulo **path** que nos permite trabalhar caminhos de arquivos e diretórios. Em seguida criamos uma constante para armazenar o caminho para o diretório público da nossa aplicação. Para isso utilizamos o método ```path.join()``` que faz o join dos caminhos fornecidos para um caminho apenas.   A variável **__dirname** representa o diretório do arquivo corrente (no caso, do arquivo index.js).
