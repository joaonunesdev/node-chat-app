# Introdução

Vamos criar uma aplicação em tempo real com Node. A natureza non-blocking do Node  faz dele uma boa alternativa para este tipo de aplicação (Chat App). 


# Criando o projeto

Nesta seção vamos dar kick-off no projeto: criando um novo diretório para o Chat App e configurando um webserver básico com express que utilizaremos durante o desenvolvimento do Chat App.

### Criando a estrutura base

**Passo 1** - Crie a estrutura de arquivos conforme apresentado na figura abaixo. Estamos criando todos os arquivos do nosso projeto de forma antecipada, não se preocupe que iremos escrever o conteúdo destes arquivos em breve. 

![](https://lh3.googleusercontent.com/QAMRiZMyC7mrOjReMaZbcHYC53VQI9PNJZsLBrkvMFV86aivI8HtuZ5ycKekdX3LjcaydHAIf5U "Estrutura")

**Passo 2** - Inicialize o npm e instale o Express.

Execute o seguinte comando no terminal (a partir do diretório chat-app):
```
> npm init -y
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

Para instalar o Express basta executar o seguinte comando no terminal (a partir do diretório chat-app):

```
> npm install express@4.17.1
```

**Passo 3** - Criar um servidor com Express.

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
No código acima primeiro importamos o módulo **path** que nos permite trabalhar caminhos de arquivos e diretórios. Em seguida criamos uma constante para armazenar o caminho para o diretório público da nossa aplicação. Para isso utilizamos o método ```path.join()``` que faz o join dos caminhos fornecidos para um caminho apenas.   A variável **__dirname** representa o caminho do arquivo corrente (no caso, do arquivo index.js).

**Passo 5** - Preenchemos o aquivo index.html com o código abaixo para que seja renderizado o texto "Chat App" na tela.

```html
<!DOCTYPE html>
<html>
	<head></head>
	<body>
		Chat App
	</body>
</html>
```
Depois executamos o comando abaixo 
```
> node src/index.js
> Server is up on port 3000
```

Ao acessar o endereço **localhost:/3000** a página index.html deve ser renderizada com o texto "Chat App".

**Passo 6** - Configurando scripts no **package.json**.

Primeiro vamos criar o script "start" para iniciar o app usando node. Para isso vamos editar o arquivo **package.json**:
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
	"license": "ISC",
	"dependencies": {
	"express": "^4.17.1"
	}
}
 ```

Em "scripts" removemos a chave "test" (não iremos escrever testes neste projeto),  e adicionamos "start" e "dev" conforme o json abaixo:

 ```json
 {
	"name": "chat-app",
	"version": "1.0.0",
	"description": "",
	"main": "index.js",
	"scripts": {
		"start": "node src/index.js",
		"dev": "nodemon src/index.js"
	},
	"keywords": [],
	"author": "",
	"license": "ISC",
	"dependencies": {
	"express": "^4.17.1"
	}
}
 ```

Em seguida instalamos o nodemon como uma dependência de desenvolvimento (*--save-dev*) com o comando descrito abaixo:

```
> npm install nodemon@1.19.3
```
Se conferirmos o **package.json** vamos ver que **devDependecies** foi adicionado com **nodemon** como primeira dependência. A partir de agora, para rodar o servidor basta executar o comando ``npm run <nome-do-script>``, conforme exemplo abaixo:

```
> npm run dev
```

# WebSockets

O protocolo WebSocket suporta comunicação bi-direcional (full-duplex) em tempo real, o que faz dele uma boa opção para criar o nosso aplicativo de chat. Para mais informações acerca do protocolo WebSocket veja [este artigo](https://medium.com/reactbrasil/como-o-javascript-funciona-aprofundando-em-websockets-e-http-2-com-sse-como-escolher-o-caminho-d4639995ef85). 

**Passo 7** - Instalando e configurando Socket.io.

Vamos instalar e configurar Socket.io, o qual possui tudo que precisamos para configuramos um servidor WebScoket com Node.

Para instalar socket.io execute o comando abaixo:

```
npm install socket.io@2.3.0
```

Socket.io pode ser utilizado de forma standalone ou com o Express. Haja vista que a nossa aplicação servirá assets do lado do cliente, ambos serão configurados. O arquivo do servidor abaixo mostra como isto pode ser feito.

```javascript
const path = require('path')
const http = require('http'
const express = require('express')
const socketio = require('socket.io')

// Cria a aplicação Express
const app = express()

// Cria um servidor HTTP usando a aplicação Express
const server= http.createServer(app)

// Conecta socket.io com o servidor HTTP
const io = socketio(server)

const port = process.env.PORT || 3000
const publicDirectoryPath = path.join(__dirname, '../public')

app.use(express.static(publicDirectoryPath))

// Monitora novas conexões para Socket.io
io.on('connection', () => {
	cosole.log('Nova conexão com WebSocket')
})

app.listen(port, () => {
	console.log(`Servidor está on na porta ${port}`)
})
```

O servidor logo acima usa **io.on** que é fornecido por Socket.io. **on** faz com que o servidor escute/monitore novas conexões, o que permite a execução de algum código sempre que um novo cliente conectar-se ao servidor WebSocket.

Socket.io será utilizado no cliente para conectar-se com o servidor. Socket.io automaticamente serve o arquivo **/socket.io/socket.io.js** que contém o código do lado do cliente. Para fazer o cliente se conectar ao nosso servidor devemos adicionar as tags de script abaixo para carregar a biblioteca do lado do cliente. Vamos aproveitar e adicionar um arquivo customizado que já criamos **/js/chat.js**.

No arquivo **public/chat.html** adicione os seguintes scripts:

```html
<script  src="/socket.io/socket.io.js"></script>
<script  src="/js/chat.js"></script>
```
A partir deste momento o JavaScript do lado do cliente pode conectar com o servidor Socket.io chamando **io**. **io** é fornecido pela biblioteca Socket.io do lado do cliente. Ao chamar esta função a configuração será realizada, fazendo com que o código do handler do evento **connection** seja executado.

```javascript
io()
``` 

Documentação:
- [Socket.io](https://socket.io/)

# Messages

Antes de começarmos a usar events na nossa aplicação vamos criar o utilitário que gera as mensagens do chat e de localização.  A mensagem de chat gerada possui o nome de usuário, o texto da mensagem e a data de criação. A mensagem de localização do cliente possui o nome de usuário, uma url do google maps com a latitude e longitude do usuário (capturada do browser) e a data de criação da mensagem.

Copie o código abaixo e cole no arquivo **src/utils/message.js**

```javascript
/**
* Gera uma mensagem do chat.
*
* A mensagem contém o nome do usuário, o texto da mensagem
* e a data da criação da mensagem.
*
* @param  {string}  username
* @param  {string}  text
*/
const  generateMessage  = (username, text) => {
  return {
    username,
    text,
    createdAt:  new  Date().getTime()
  }
}

/**
* Gera a mensagem de localização do cliente.
*
* A mensagem contém o nome de usuário, a url do google maps
* com as coordenadas e a data de criação da mensagem.
*
* @param  {string}  username
* @param  {string}  url
*/
const  generateLocationMessage  = (username, url) => {
  return {
    username,
    url,
    createdAt:  new  Date().getTime()
  }
 }

module.exports  = {
  generateMessage,
  generateLocationMessage
}
```

