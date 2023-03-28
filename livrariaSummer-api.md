
---
# Introdução

## **Testando nossa Api Rest com Express e MySQL**


## O que vamos aprender?
#### no último bloco você aprendeu sobre Express e MySQL,hoje vamos nosaprofundar em como testar nossa Api Rest com testes de integração.

### Para isso vamos utilizar três ferramentas para escrevermos nossos tests com Node.js
- Mocha
- Chai
- Sinon
## Você será capaz de :
1. Criar testes de integração para Apis Rest;
1. Estruturar cenários de testes de acordo com os requisitos;
1. Aprender e utilizar a prática TDD(desenvolvimento orientado a testes);

## Por que isso é importante?
#### Testes fazem parte do dia a dia de pessoas desenvolvedoras.Testes permitem identificar erros durante as etapas de desenvolvimento, assegurar a qualidade do produto e seu funcionamento correto,garantindo que alterações futuras no código não gerem novos bugs.
---

# Teste de Integração
### Os testes de integração ou integration tests,servem para verificar se a comunicação entre os componentes de um sistema está ocorrendo da forma esperada.

### Para escrevê-los vamos utilizar a prática do TDD (desenvolvimento orientado a testes),cuja a idéia principal é começar a escrever os testes para códigos que ainda não existem.É completamente normal que um detalhe ou outro possamos esquecer ,existindo a necessidade de fazer algum ajuste nos testes.

### Para melhor exemplificar a prática do TDD com testes de integração, iremos construir uma Api empregando esses dois conceitos,começaremos defindo os cenários de testes e a medida que os testes falharem vamos implementar o código,até eles serem executados com sucesso.

### Por meio dos contratos de uma Api, definimos o que devem ser testados e desenvolvidos, e é sobre isso que vamos estudar a seguir.

---
# Contratos de uma Api

### Sempre que consumimos ou fornecemos um serviço,como por exemplo uma Api Rest precisamos ter os comportamentos predefinidos.Esses comportamentos são definidos de acordo com as regras de entrada e saída de dados de uma Api,essas regras são chamadas de contrato,o contrato é tudo aquilo que foi previamente acordado , ou seja, como uma Api deve se comportar.

### Por exemplo: Ao chamar um endPoint GET/books, podemos dizer que o contrato dele é retornar todos os livros, caso exista algum retornará um status HTTP:200-ok , caso não exista ele deve retornar um código HTTP :404-NOT FOUND.

---
# Definindo cenário de teste
### Para exemplificar os testes de integração , vamos construir uma Api chamada livraria Summer , uma api que fornece endPoints para gerenciar os livros existentes.Agora que já sabemos o que vamos fazer , podemos iniciar o nosso projeto.

---
# Iniciando um novo Projeto

### Utilizaremos nesse projeto algumas outras coisas que você aprendeu antes:
-MySQL

-Docker

### Segue abaixo comandos para criar a pasta livrariasummer-api,iniciar o projeto com Node.js e criar os subdiretórios:

```javascript
mkdir livrariasummer-api
cd livrariasummer-api
npm init -y 
npm i nodemon@2.0.15 --save-dev --save-exact
mkdir src
mkdir test
```

###Vamos criar o arquivo server.js dentro da pasta src (sem nenhum conteúdo ainda), que será o ponto de entrada da sua aplicação,ou seja, quando vocẽ iniciar a sua aplicação node.js seu código vai ser executado apartir deste arquivo.

Agora vamos configurar o nodemon no package.json para restartar a sua aplicação a cada modificação.

```
{ //...

  "main": "src/server.js",
  "scripts": {
    "start": "node src/server.js",
    "dev": "nodemon src/server.js",
    "test":"mocha test/**/livrariasummer.test.js --exit"
  }
  //...
  }
```
### No package.json acima a chave main foi modificada para src/server.js,também foram criadas as chaves start e dev ambas para iniciar a aplicação, porém o nodemon como dito antes reinicia a aplicação a cada modificação.
Podendo assim iniciar a aplicação com o comando :
`npm run dev`

### Agora vamos criar um arquivo na raiz do projeto com o nome e livrariasummer_script.sql , esse arquivo cria as tabelas , coloque nele o código abaixo:


```
DROP DATABASE IF EXISTS livrariasummerdb;

CREATE DATABASE livrariasummerdb;
USE livrariasummerdb;

CREATE TABLE books (
    id INT NOT NULL AUTO_INCREMENT,
    name_book VARCHAR(45) NOT NULL,
    author_name VARCHAR(45) NOT NULL,
    genre_name VARCHAR(45) NOT NULL,
    pages VARCHAR(60) NOT NULL,
    release_year VARCHAR(20),
    PRIMARY KEY(id)
);

  INSERT INTO livrariasummerdb.books (name_book, author_name, genre_name, pages, release_year)
  VALUES
    ('Brigerton O duque e eu','Julia Quinn','Romance',496,2000),
    ('O menino do pijama listrado','John Boyne','Drama',192,2007),
    ('O conto da aia','Margaret Atwood','Drama',366,1985),
    ('Anne de Avonlea ','Lucy Maud Montgomery','Romance',240,1909)
  ;
  ```
  
  ---
  # MySQL no Docker

  ### Na raiz  do projeto vamos criar o arquivo  docker-compose.yaml com o seguinte conteúdo: 
 
 ##  docker-compose.yaml:

 ```
services:
  # Serviço que irá rodar o node
  node:
    # Imagem base do container
    image: node:16.14
    # Nome do container para facilitar execução
    container_name: livrariasummer_api
    # Mesmo que `docker run -t`
    tty: true
    # Mesmo que `docker run -i`
    stdin_open: true
    # Substitui o comando padrão da imagem do node
    command: bash # você pode subistituir `bash` por `npm run debug` para iniciar a API quando container for iniciado  
    # Restarta a imagem caso algo a faça parar
    restart: always
    # Diretório padrão de execução
    working_dir: /home/node/app
    # Lista de volumes (diretórios) mapeados de fora para dentro do container
    volumes:
      # Monta o diretório atual, com todos os dados do projeto, 
      # dentro do diretório /app
      - ./:/home/node/app   #volume ele fica"escutando as alterações feitas no projeto e acontece a alteraçao no docker 
    # Lista de serviços do qual este serviço depende
    depends_on:
      # Precisa do banco funcionando antes de subir o node
      - db
    # Lista de portas mapeadas de dentro para fora do container
    # na notação porta_de_fora:porta_de_dentro
    ports:
      # Expõe a porta padrão da aplicação: altere aqui caso use outra porta
      - 3000:3000
      - 9229:9229
    # Configura as variáveis de ambiente dentro do container
    environment:
      MYSQL_USER: root
      MYSQL_PASSWORD: password
      MYSQL_HOST: db # Nome do service logo abaixo
      PORT: '3000'
      HOST: livraria_summer
    networks:
      # Conecta esse serviço com a rede que criada
      - livraria_summer_net

  # Serviço que irá rodar o banco de dados
  db:
    container_name: livrariasummerdb
    image: mysql:8.0.23
    environment:
      MYSQL_ROOT_PASSWORD: 'root'
    ports:
      # Garanta que não haverá conflitos de porta com um banco que esteja
      # rodando localmente
      - 3306:3306
    volumes:
      - ./livrariasummer_script.sql:/docker-entrypoint-initdb.d/livrariasummer_script.sql
    networks:
      # Conecta esse serviço com a rede que criada 
      - livraria_summer_net

# Lista de redes que os containeres irão utilizar
networks:
  # Cria a rede que conecta os serviços `db` e `node` 
  livraria_summer_net:
    # Específica o drive da rede
    driver: bridge
    # o docker compose faz uma ponte entre a rede localhost e a rede criada pelo docker-compose.

# As chaves `tty`, `stdin_open` e `command` fazem com que o container fique
# rodando em segundo plano, bem como tornam possível o uso do comando
# `docker attach`, já deixando o terminal atual acoplado ao container, direto
# no bash. Apesar disso, utilizar o attach mais de uma vez irá replicar o
# terminal, portanto é melhor utilizar o comando `docker exec`.

# A renomeação da rede padrão é feita pois caso contrário o nome da rede será
# o nome do diretório onde o arquivo atual se encontra, o que pode dificultar
# a listagem individual.
      
 ```
 ### A opção volumes do database,faz  o espelhamento do nosso script .sql na pasta docker-entrypoint-initdb.d. Essa alteração fará com que as tabelas já sejam criadas no momento em que subirmos o container.

## Aviso antes de iniciar o container para não ter problemas execute o comando: `docker system prune -af`

### Após a criação dos arquivos ,iniciaremos o container om o comando: `docker-compose up -d`

### Dentro do container que iniciaremos nossa aplicação no futuro. Você lembra o comando para entrar no container?
### Não? sem problemas está na mão:`Docker exec -it 'meu container' sh` (trocando o nome "meu container" pelo nome do container)

### outra opção seria clicar na extensão do docker ir até o container node clicar com o botão direito do mouse nele e escolher "Attach Shell" 
## pronto vocẽ está dentro do container!

---
# Configurando o MySQL no Express

### Primeiro vamos instalar as dependências necessárias para configurarmos um projeto Express capaz de conectar o MySQL.

`npm i express@4.17.1 mysql2@2.3.3 --save-exact`

### O mysql2 é o responsável por premitir que uma aplicação node.js consiga comunicar-se com o MySQL , chamamos esse tipo de biblioteca de client o qual possui todo o código necessário para enviarmos comandos SQL para nosso banco de dados, no caso o MySQL.

### O próximo passo é criar a pasta  src/db com o arquivo chamado connection.js, que será responsável por realizar a conexão com o servidor MySQL utilizando a biblioteca mysql2.

```
// src/db/connection.js


const mysql = require('mysql2/promise');

const connection = mysql.createPool({
//o docker compose cria uma rede cujo o endereço IP é passado na url do serviço db, por isso colocamos no host.
  host: process.env.MYSQL_HOST, 
  //O número da porta que nossa aplicação utilizará para acessar o MySQL
  port: 3306,
//O nome de usuário que nossa aplicação utilizará para acessar o MySQL
  user: 'root', 
 //A senha do usuário que nossa aplicação utilizará para acessar o MySQL
  password: 'root',
//O nome do banco de dados MySQL, o qual queremos que nossa aplicação realize uma conexão
  database: 'livrariasummerdb', 
 //Determina qual será a ação da pool de conexões quando nenhuma conexão estiver disponível na pool e quando o limite de criação de novas conexões tiver sido alcançado
  waitForConnections: true,
//O número máximo de requisições de conexão que a pool criará de uma vez
  connectionLimit: 10, 
//O número máximo de requisições de conexão que a pool irá enfileirar antes de retornar um erro
 //Se o valor deste parâmetro for igual a 0 significa que não existe limite. 
 queueLimit: 0, 

});
 

module.exports = connection;


```

### No código acima estamos importando o mysql2 com o recurso promises,para poder utilizar o MySQL de forma assíncrona.Logo após criamos uma constante connection que recebe um pool de conexões,criado com a função createPool.

### pool de conexões é um cache de conexões de banco de dados mantido de forma que as conexões possam ser reutilizadas quando requisições futuras ao banco de dados forem requeridas. Pools de conexões são usados para garantir o desempenho da execução de comandos sobre um banco de dados.


## Agora está na hora de configurar o express no projeto:

### Vamos criar o nosso arquivo src/app.js e configurar o src/server.js.

### src/app.js
```
const express = require('express');
const app = express();
app.use(express.json());
module.exports = app;
```
### Agora adicione o código abaixo no arquivo server.js:
```
const app = require('./app');

const PORT = 3001;

app.listen(PORT, () => {
  console.log(`API livrariaSummer está sendo executada na porta ${PORT}`);
});
```
### Agora que tudo está devidamente configurado , vamos começar a escrever os testes , utilizando a prática do TDD.

## Então vamos começar!!!

### 1° passo vamos instalar as dependências necessárias para escrevermos nossos testes de integração.


`npm i mocha@10.0.0 chai@4.3.6 sinon@14.0.0 chai-http@4.3.0 -D`

### 2° passo Agora crie uma pasta dentro da pasta test chamada test.integration e logo após crie um arquivo chamado livrariasummer.test.js

## Aviso: é extremamente importante você colocar os nomes que foram indicados , qualquer erro seu teste não vai rodar.

### 3° passo Vamos definir o cenário de test: 
### - O cenário de test é recuperar todos os livros da livrariasummer com o endPoint GET/books.


### 4° passo cole o código abaixo no arquivo livrariasummer.test.js

```


const chai = require('chai');
const chaiHttp = require('chai-http');
const sinon = require('sinon');
const app = require('../../src/app');
const connection = require('../../src/db/connection');

const { expect,use } = chai;

use(chaiHttp);

const bookList = [ //nosso mock queserá utilizado no stub
  {
    id: 1,
    name_book: 'Brigerton O duque e eu',
    author_name: 'Julia Quinn',
    genre_name: 'Romance',
    pages: '496',
    release_year:'2000',
  },
  {
    id: 2,
    name_book: 'O menino do pijama listrado',
    author_name: 'John Boyne',
    genre_name: 'Drama',
    pages: '192',
    release_year:'2007',
  },
  {
    id: 3,
    name_book: 'O conto da aia',
    author_name: 'Margaret Atwood',
    genre_name: 'Drama',
    pages: '366',
    release_year:'1985',
  },
  {
    id: 4,
    name_book: 'Anne de Avonlea',
    author_name: 'Lucy Maud Montgomery',
    genre_name: 'Romance',
    pages: '240',
    release_year:'1909',
  },
];
describe('CRUD na rota /books', function () {
    
    it('Retorna a lista completa de livros!', async function () {
      sinon.stub(connection, 'execute').resolves([bookList]);
        const response = await chai
            .request(app)
            .get('/books');

            expect(response.status).to.equal(200);
            expect(response.body).to.
              deep.equal(bookList);
          });  
           afterEach(sinon.restore);
});
```
---
### - Adicionamos o plugin chai http ao chai , assim podemos consumir o server express por meio dele,sem que haja a necesidade de subir a Api manualmente, em outras palavras o chai cria seu próprio listen().
---

### - Utilizamos também o método request , que foi adicionado ao chai pelo plugin,ele nos permite chamar diretamente nossos endPoints simulando chamadas http.
---
### - Usamos o stub com o sinon na função execute de connection de maneira que quando essa função for chamada no teste , ela retornará um array de objetos com todos os livros.
---
### - Após definimos nosso expect , você pode observar que utilizamos o `to.deep.equal`, isso quer dizer que ela faz uma verificação das informações de uma forma mais profunda.
---
## Agora você pode executar os test com o comando :
`npm test`

## E então.... Buhhh!!!


### Os tests não passaram ,isso aconteceu porque não fizemos a nossa Api, então agora vamos começar a construção!

## Let's go to the code!

### Crie dentro da pasta src/db o arquivo livrariadb.js e nele vamos criar a função getAll que vai se comunicar com o banco de dados utilizando o connection e utilizando as querys SQL para manipular o banco de dados.

cole no arquivo livrariadb.js o código abaixo:
```
const conn = require('../db/connection');
const getAll = () => conn.execute('SELECT * FROM books');
module.exports={getAll};
```
### Agora crie a pasta src/routes e um arquivo dentro dela chamado livrariaRoutes.js  aonde será feito nossos endPoints que fará a chamada a função getAll.

```


const express = require('express');

const router = express.Router();
const livrariadb=require('../db/livrariadb');

router.get('/', async(_req, res) => {
  try{
  const[result]=await livrariadb.getAll();
  res.status(200).json(result)
  }catch(err){
    return res.status(500).json({ message: err.sqlMessage });
  };
});
```
### O próximo passo é fazer a chamada de routers no app.js

```


//const express = require('express');
const livrariaRoutes = require('./routes/livrariaRoutes');

// const app = express();

//app.use(express.json());

app.use('/books', livrariaRoutes);
// module.exports = app;
```

## Agora tudo está pronto, vamos testar novamente?

### utilize o comando :`npm test`

## E.... passou!!!

## Hoje você testou uma Api com test de integração utilizando a prática do TDD! PARABÉNS

---
# Recursos Adicionais

[TDD: fundamentos do desenvolvimento orientado a testes](https://www.devmedia.com.br/tdd-fundamentos-do-desenvolvimento-orientado-a-testes/28151#:~:text=O%20TDD%20transforma%20o%20desenvolvimento,deseja%20em%20rela%C3%A7%C3%A3o%20ao%20c%C3%B3digo.)

[TDD (Test Driven Development) // Dicionário do Programador](https://www.youtube.com/watch?v=bLdEypr2e-8)

---
# Exercícios do dia
---
## Teste o endPoint GET/books/1,utilizano o mesmo banco de dados,acrescente os códigos abaixo na aplicação  e realize os testes.
---
### - 1° passo acrescente o endPoint abaixo na livrariaRoutes.js
```
 router.get('/:id',async(req,res)=>{
  try {
    const { id } = req.params;
    const [[result]] = await livrariadb.findById(id);
    if (result) {
      res.status(200).json(result);
    } else {
      res.status(404).json({ message: 'Pessoa não encontrada' });
    }
  } catch (err) {
    console.log(err);
    res.status(500).json({ message: err.sqlMessage });
  }
})
```
### - 2° passo acrescente a função finBydId no arquivo livrariadb.js

```
 const findById = (id) => conn.execute('SELECT * FROM books WHERE id = ?', [id]);
 
 //module.exports={getAll,
 findById}
```

### agora você pode criar seu test de integração.

---
## Teste o endPoint POST/books,utilizano o mesmo banco de dados,acrescente os códigos abaixo na aplicação  e realize os testes.
---
### 1° passo passo acrescente o endPoint abaixo na livrariaRoutes.js
```

router.post('/', async (req, res) => {
  const book = req.body;
  try {
    const [result] = await livrariadb.insert(book);
    res.status(201).json({
      message: `Pessoa cadastrada com sucesso com o id ${result.insertId}` });
  } catch (err) {
    console.log(err);
    res.status(500).json({ message: 'Ocorreu um erro ao cadastrar uma pessoa' });
  }
});
```
### 2° passo passo acrescente a função finBydId no arquivo livrariadb.js
```
const insert = (book) => conn.execute(
  `INSERT INTO books 
  (name_book, author_name, genre_name, pages,release_year) VALUES (?, ?, ?, ?,?)`,
  [book.nameBook, book.authorName, book.genreName, book.pages,book.releaseYear],
  );
  //module.exports={getAll,
 findById,insert}
  ```
### 3° passo utilize o código abaixo no .send do test  para criar o novo livro
```
  {
    name_book: "Diário de Anne Frank",
    author_name: "Anne Frank",
    genre_name: "Biografia",
    pages": 352,
    release_year:1947
  }
  ```
  ### agora você pode começar a escrever seus tests
  ---
## Teste o endPoint PUT/books/1,utilizano o mesmo banco de dados,acrescente os códigos abaixo na aplicação  e realize os testes.
---
### 1° passo passo acrescente o endPoint abaixo na livrariaRoutes.js
```
router.put('/:id', async (req, res) => {
  try {
    const { id } = req.params;
    const book = req.body;
    const [result] = await livrariadb.update(book, id);
    if (result.affectedRows > 0) {
      res.status(200).json({ message: `Pessoa de id ${id} atualizada com sucesso` });
    } else {
      res.status(404).json({ message: 'Pessoa não encontrada' });
    }
  } catch (err) {
    res.status(500).json({ message: err.sqlMessage });
  }
});

```
### 2° passo passo acrescente a função finBydId no arquivo livrariadb.js
```
const update = (book, id) => conn.execute(
  `UPDATE books
  SET name_book = ?, author_name = ?, genre_name = ?, pages = ?,release_year = ? WHERE id = ?`,
  [book.nameBook, book.authorName, book.genreName, book.pages,book.releaseYear,id],
);
  //module.exports={getAll,
 findById,insert}
  ```
### 3° passo utilize o código abaixo no .send do test  para criar o novo livro
```
{
  name_book: 'Brigerton O visconde que me amava',
  author_name: 'Julia Quinn',
  genre_name: 'Romance',
  pages: '304',
  release_year:'2000'
}

  ```
  ### agora você pode começar a escrever seus tests

---
# Exercício Bônus

## Para terminarmos o CRUD faltou fazer um DELETE, como exercício bônus você vai fazer o endPoint delete , a função e o test.
## vamos fazer utilizando a prática do TDD?

## - Caso tenha alguma dúvida utilize o Gabarito para lhe auxiliar.


---

## Bons Estudos!


















