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

# Mensagens

Antes de começarmos a usar events na nossa aplicação vamos criar o utilitário que gera as mensagens do chat e de localização.  

**Passo 7** - Criando o utilitário de mensagens.

A mensagem de chat gerada possui o nome de usuário, o texto da mensagem e a data de criação. A mensagem de localização do cliente possui o nome de usuário, uma url do google maps com a latitude e longitude do usuário (capturada do browser) e a data de criação da mensagem.

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

# Usuários
O último passo antes de começarmos a trabalhar com WebSockets será criar as regras de negócio da nossa aplicação.

**Passo 8** - Criando as regras de negócio de nossa aplicação.

Já adicionamos um arquivo **users.js** no diretório **utils**. Neste arquivo colocaremos a estrutura de dados que armazenará os usuários, um array simples, e quatro funções: **addUser**, que adiciona um usuário na array de usuários; **removeUser**, que remove um usuário da array de usuários;  **getUser**, que recupera um usuário específico; e **getUsersInRoom**, que recupera todos os usuários que estão em determinada sala de chat.

Copie o código abaixo e cole no arquivo **src/utils/users.js**

 ```javascript
const  users  = []

/**
* Adiciona um cliente no chat.
* @param  {Object}  objeto Representação de um cliente do chat.
*/
const  addUser  = ({ id, username, room }) => {
  // Trata os dados removendo espaços em branco e transformando em caixa baixa
  username  =  username.trim().toLowerCase()
  room  =  room.trim().toLowerCase()

  // Valida os dados
  if (!username  ||  !room) {
    return {
      error:  'Username and room are required!'
    }
  }

  // Verifica a existência de um usuário
  const  existingUser  =  users.find((user) => {
    return  user.room  ===  room  &&  user.username  ===  username
  })

  // Valida o nome de usuário
  if (existingUser) {
    return {
      error:  'Username is in use!'
    }
  }  

  // Registra o usuário
  const  user  = { id, username, room }
  users.push(user)

  return { user }
}

/**
* Remove um cliente do chat.
* @param  {number}  id Identificador único do usuário.
*/
const  removeUser  = (id) => {
  // Encontra o índice do usuário com a id informada
  // Embora filter pudesse ser utilizado, essa abordagem é mais performática
  const  index  =  users.findIndex((user) =>  user.id  ===  id)

  // Remove o usuário pelo índice da array
  if (index  !==  -1) {
    return  users.splice(index, 1)[0]
  }
  
}

/**
* Recupera um usuário por meio da id.
* @param  {number}  id Identificador único do usuário.
*/
const  getUser  = (id) => {
  return  users.find((user) =>  user.id  ===  id)
}

/**
* Recupera os usuários de uma sala de chat.
* @param  {Object}  room Sala de chat
*/
const  getUsersInRoom  = (room) => {
  room  =  room.trim().toLowerCase()
  return  users.filter((user) =>  user.room  ===  room)
}

module.exports  = {
  addUser,
  removeUser,
  getUser,
  getUsersInRoom
}
```

# WebSockets e Socket.IO

O protocolo WebSocket suporta comunicação bi-direcional (full-duplex) em tempo real, o que faz dele uma boa opção para criar o nosso aplicativo de chat. Para mais informações acerca do protocolo WebSocket veja [este artigo](https://medium.com/reactbrasil/como-o-javascript-funciona-aprofundando-em-websockets-e-http-2-com-sse-como-escolher-o-caminho-d4639995ef85). 

Nesta aplicação utilizaremos Socket.IO. Socket.IO é uma biblioteca que permite a comunicação em tempo real, bidirecional e baseada em eventos entre o navegador, que consiste de um servidor Node.js e uma biblioteca cliend-side para o browser.

- Para mais informações:  
	- [O que é Socket.io?](https://socket.io/docs/#What-Socket-IO-is)
	- [O que Socket.io não é?](https://socket.io/docs/#What-Socket-IO-is-not)

**Passo 9** - Instalando e configurando Socket.io.

Vamos instalar e configurar Socket.io, o qual possui tudo que precisamos para configuramos um servidor WebSocket com Node.

Para instalar socket.io execute o comando abaixo:

```
npm install socket.io@2.3.0
```

Socket.io pode ser utilizado de forma standalone ou com o Express. Haja vista que a nossa aplicação servirá assets do lado do cliente, ambos serão configurados. O arquivo do servidor abaixo mostra como isto pode ser feito.

```javascript
const path = require('path')
const http = require('http')
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
	console.log('Nova conexão com WebSocket')
})

app.listen(port, () => {
	console.log(`Servidor está on na porta ${port}`)
})
```

O servidor logo acima usa **io.on** que é fornecido por Socket.io. **on** faz com que o servidor escute/monitore novas conexões, o que permite a execução de algum código sempre que um novo cliente conectar-se ao servidor WebSocket.

Socket.io será utilizado no cliente para conectar-se com o servidor. Socket.io automaticamente serve o arquivo **/socket.io/socket.io.js** que contém o código do lado do cliente. Para fazer o cliente se conectar ao nosso servidor devemos adicionar algumas tags de script no arquivo **public/index.html**, carregando assim a biblioteca do lado do cliente. Vamos também aproveitar e modificar o arquivo **/js/chat.js**.

No arquivo **public/index.html** adicione os seguintes scripts:

```html
<script  src="/socket.io/socket.io.js"></script>
<script  src="/js/chat.js"></script>
```
O arquivo **public/index.html** deve estar da seguinte forma: 
```html javascript
<!DOCTYPE  html>
<html>
  <head></head>
  <body>
    <script  src="/socket.io/socket.io.js"></script>
    <script  src="/js/chat.js"></script>
  </body>
</html>
```

A partir deste momento o JavaScript do lado do cliente pode conectar-se com o servidor Socket.io chamando ``io`` . ``io`` é fornecido pela biblioteca Socket.io do lado do cliente. Ao chamar esta função a configuração será realizada, fazendo com que o código do handler do evento ``connection`` seja executado.

No arquivo **public/js/chat.js**
```javascript
const  socket  =  io()
``` 
- Para mais informações consulte a documentação oficial:
	- [Socket.io](https://socket.io/)

# Eventos 

Eventos em Socket.io permitem a transferência de dados do cliente para o servidor ou do servidor para o cliente.

Antes de por a mão na massa, vamos estabelecer alguns conceitos básicos. Há dois lados para cada evento, o remetente (sender) e o receptor (receiver). Se o servidor é o remetente, o cliente é o receptor. Se o cliente é o remetente, o servidor é o receptor.

Eventos são enviados do remetente usando o método ``emit``. Eventos são recebidos pelo receptor usando o método ``on``, que funciona como um *listener*. Na nossa aplicação iremos utilizar eventos padrões do Socket.IO como ``connection``, ``message`` e ``disconnection``, mas também criaremos eventos personalizados. Aliás, já utilizamos o evento ``connection`` no **index.js**, de forma que emite uma mensagem no console sempre que houver uma nova conexão. 

Na nossa aplicação de bate-papo, seria interessante que os usuários em uma sala fossem notificados do ingresso de um novo usuário na sala. Para isso, vamos usar eventos de **broadcasting**.

**Passo 10** - Notificando a conexão de um novo usuário e saudá-o.


**Broadcasting**

Eventos podem ser transmitidos a partir do servidor usando ``socket.broadcast.emit``. Este evento será enviado para todos os sockets, exceto o que transmitiu o evento. O alteração de código abaixo mostra isso. Quando um novo usuário ingressa no aplicativo de bate-papo, ``socket.broadcast.emit`` é usado para enviar uma mensagem a todos os outros usuários, informando notificando-os do ingresso de um novo usuário na sala de bate-papo.

Vamos modificar um pouco o arquivo **src/index.js** para usar `` broadcast.emit`` e receber um objeto ``socket`` como parâmetro.
 
```javascript
// Monitora novas conexões para Socket.io
io.on('connection', (socket) => {
	// console.log('Nova conexão com WebSocket') <- Remova essa linha
	
	// Notifica o ingresso de um novo usuário no bate-papo
	socket.broadcast.emit('message', 'Um novo usuário se conectou!') 
})
```

``socket`` contém informações sobre cada nova conexão. Dessa forma, podemos utilizar métodos do objeto ``socket`` para se comunicar com aquele específico cliente que realizou a conexão. No código acima estamos emitindo um evento do tipo ``broadcast`` que é recebido por todos os clientes/conexões ativas com exceção do remetente (o cliente que acaba de se conectar, representado por ``socket``).

Se iniciarmos o servidor não veremos nenhuma alteração. Para vermos algo realmente acontecer vamos alterar o código do lado do cliente para ficar escutando o evento ``message``.

Em **public/js/chat.js** vamos adicionar o seguinte código:

```javascript
const  socket  =  io()

socket.on('message', (msg) => {
  console.log(msg)
})
``` 

Para testar, podemos abrir quatro instâncias do browser, exibir o console (ativando o modo desenvolvedor) e acessar o endereço **localhost:3000** em cada uma das instâncias.

![enter image description here](https://i.imgur.com/wMRXEvz.gif)

Vamos adicionar mais um evento para monitorar as desconexões dos clientes. No nosso aplicativo de bate-papo servirá para notificar à sala da saída de um membro do bate-papo.

Vamos modificar mais um pouco o arquivo **src/index.js** adicionando um *listener* para monitorar eventos do tipo ``disconnect`` (i.e., desconexões dos clientes). No nosso aplicativo de bate-papo, esse trecho de código servirá para notificar todos os membros da sala de bate-papo da saída de alguém. 
 
```javascript
// Monitora novas conexões para Socket.io
io.on('connection', (socket) => {
	// Notifica o ingresso de um novo usuário no bate-papo
	socket.broadcast.emit('message', 'Um novo usuário se conectou!') 

	// Monitora desconexões dos clientes
	socket.on('disconnect', (socket) => {
		// Notifica a saída de um usuário da sala de bate-papo	
		io.emit('message', 'Um usuário saiu da sala!')
	})
})
```

Nossa aplicação de bate-papo deve estar da seguinte forma:

![enter image description here](https://i.imgur.com/jQVypQo.gif)

Reparem que não estamos usando ``broadcast`` no *handler* de eventos de desconexões, e sim ``io.emit``, que emite o evento do tipo `message` para todos os clientes conectados, incluindo o emissor. Como este evento será emitido quando o emissor sair da sala, não há problema em usar ``io.emit``.

Agora vamos adicionar uma mensagem de boas vindas para cada novo usuário que se logar. Essa mensagem será enviada somente para o usuário que estiver se conectando no momento e não será vista pelos demais presentes na sala de bate-papo.

Em **src/index.js**:
```javascript
// Monitora novas conexões para Socket.io
io.on('connection', (socket) => {
	// Envia mensagem de boas vindas a um novo usuário que se conectou
	// A mensagem é enviada ao novo usuário (nova conexão), somente
	socket.emit('message', "Welcome!")
	
	// Notifica o ingresso de um novo usuário no bate-papo
	// A mensagem é enviada para todos, exceto para o cliente que socket faz referencia
	socket.broadcast.emit('message', 'A new user has joined!') 

	// Monitora desconexões dos clientes
	socket.on('disconnect', (socket) => {
		// Notifica a saída de um usuário da sala de bate-papo	
		io.emit('message', 'User has left!')
	})
})
```

Nossa aplicação de bate-papo deve estar da seguinte forma:

![enter image description here](https://i.imgur.com/PXuHBcq.gif)

# Interface do usuário
Como o intuito aqui é entendermos os conceitos mais básicos do Socket.io, vamos dar o "pulo do gato" e copiar alguns arquivos de estilo e html para criar a interface de usuário inicial de nossa aplicação de bate-papo.

Sendo assim, copie o seguinte código css para o arquivo **public/css/styles.css**:

```css
/* Estilo Geral */
* {
	margin: 0;
	padding: 0;
	box-sizing: border-box;
}

html {
	font-size: 16px;
}

input {
	font-size: 14px;
}

body {
	line-height: 1.4;
	color: #333333;
	font-family: Helvetica, Arial, sans-serif;
}

h1 {
	margin-bottom: 16px;
}

label {
	display: block;
	font-size: 14px;
	margin-bottom: 8px;
	color: #777;
}

input {
	border: 1px  solid  #eeeeee;
	padding: 12px;
	outline: none;
}

button {
	cursor: pointer;
	padding: 12px;
	background: #7C5CBF;
	border: none;
	color: white;
	font-size: 16px;
	transition: background  .3s  ease;
}

button:hover {
	background: #6b47b8;
}

button:disabled {
	cursor: default;
	background: #7c5cbf94;
}

/* Estilos da página de Join */
.centered-form {
	background: #333744;
	width: 100vw;
	height: 100vh;
	display: flex;
	justify-content: center;
	align-items: center;
}

.centered-form__box {
	box-shadow: 0px  0px  17px  1px  #1D1F26;
	background: #F7F7FA;
	padding: 24px;
	width: 250px;
}

.centered-form  button {
	width: 100%;
}

.centered-form  input {
	margin-bottom: 16px;
	width: 100%;
}

/* Layout da página de chat */

.chat {
	display: flex;
}

.chat__sidebar {
	height: 100vh;
	color: white;
	background: #333744;
	width: 225px;
	overflow-y: scroll
}

/* Estilos do Chat */
.chat__main {
	flex-grow: 1;
	display: flex;
	flex-direction: column;
	max-height: 100vh;
}

.chat__messages {
	flex-grow: 1;
	padding: 24px  24px  0  24px;
	overflow-y: scroll;
}

/* Estilo das Mensagens */
.message {
	margin-bottom: 16px;
}

.message__name {
	font-weight: 600;
	font-size: 14px;
	margin-right: 8px;
}

.message__meta {
	color: #777;
	font-size: 14px;
}

.message  a {
	color: #0070CC;
}

/* Estilos da Composição de Mensagens */
.compose {
	display: flex;
	flex-shrink: 0;
	margin-top: 16px;
	padding: 24px;
}

.compose  form {
	display: flex;
	flex-grow: 1;
	margin-right: 16px;
}

.compose  input {
	border: 1px  solid  #eeeeee;
	width: 100%;
	padding: 12px;
	margin: 0  16px  0  0;
	flex-grow: 1;
}

.compose  button {
	font-size: 14px;
}

/* Estilo da Sidebar */
.room-title {
	font-weight: 400;
	font-size: 22px;
	background: #2c2f3a;
	padding: 24px;
}

.list-title {
	font-weight: 500;
	font-size: 18px;
	margin-bottom: 4px;
	padding: 12px  24px  0  24px;
}

.users {
	list-style-type: none;
	font-weight: 300;
	padding: 12px  24px  0  24px;
}
```

Agora, copie o seguinte código html para o arquivo **src/index.html**:

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Chat App</title>
    <link rel="icon" href="/img/favicon.png" />
    <link rel="stylesheet" href="/css/styles.css" />
  </head>
  <body>
    <div class="chat">
      <div id="sidebar" class="chat__sidebar"></div>
      <div class="chat__main">
        <div id="messages" class="chat__messages"></div>

        <div class="compose">
          <form id="message-form">
            <input
              placeholder="Message"
              name="message"
              required
              autocomplet="off"
            />
            <button>Send</button>
          </form>
          <button id="send-location">Send Location</button>
        </div>
      </div>
    </div>

    <script id="message-template" type="text/html">
      <div class="message">
          <p>
              <span class="message__name"> {{ username }}</span>
              <span class="message__meta"> {{ createdAt }} </span>
          </p>
          <p>{{ message }}</p>
      </div>
    </script>

    <script id="location-message-template" type="text/html">
      <div class="message">
            <p>
                <span class="message__name"> {{ username }}</span>
                <span class="message__meta"> {{ createdAt }} </span>
            </p>

            <p> <a href="{{ url }}" target="_blank">My current location</a> </p>
      </div>
    </script>

    <script id="sidebar-template" type="text/html">
      <h2 class="room-title">{{ room }}</h2>
      <h3 class="list-title">Users</h3>
      <ul class="users">
          {{ #users }}
              <li>{{ username }}</li>
          {{ /users }}
      </ul>
    </script>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/mustache.js/3.0.1/mustache.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/moment.js/2.22.2/moment.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/qs/6.6.0/qs.min.js"></script>
    <script src="/socket.io/socket.io.js"></script>
    <script src="/js/chat.js"></script>
  </body>
</html>
```

Ao salvar as alterações o nodemon deve reiniciar o servidor e nossa aplicação deve estar da seguinte forma:

![enter image description here](https://lh3.googleusercontent.com/gd4IKeOtUs2dM1m0QV0K3_DlCQIoARIBE4CLZpp08ZrMik7GeAuANkBSSxyFYm5V4O4Q4FzIbvc "UI") 
