
---
# Gabarito Exercícios do dia
---
## 1. Teste o endPoint GET/books/1,utilizano o mesmo banco de dados,acrescente os códigos abaixo na aplicação  e realize os testes.
---
```
//describe('CRUD na rota /books', function () {
  //...
   it('Testando a listagem do livro com id 1', async function () {
            sinon.stub(connection, 'execute').resolves([[bookList[0]]]);
            const response = await chai
              .request(app)
              .get('/books/1');
        
            expect(response.status).to.equal(200);
            expect(response.body).to.deep.equal(bookList[0]);
          });
          //...
 // }):
```
  ### Usamos o stub na função execute de connection de maneira que quando essa função for chamada no teste , ela retornará um array na posição 0 que seria o objeto com o id 1 que foi pedido no enunciado do exercício.

  ### utilizamos o request passando o app , e utilizamos o método get na rota /books/1

  ### o expect foi definido de acordo com o return res.status 200 e o response.body com a mesma resposta do sinon.stub.
---
  ## 2. Teste o endPoint POST/books,utilizano o mesmo banco de dados,acrescente os códigos abaixo na aplicação  e realize os testes.
  ---
  ```
//describe('CRUD na rota /books', function () {
  //...
  it('Testando o cadastro de um book ', async function () {
    sinon.stub(connection, 'execute').resolves([{ insertId: 6 }]);

    const response = await chai
      .request(app)
      .post('/books')
      .send(
        {
          name_book: 'Diário de Anne Frank',
          author_name: 'Anne Frank',
          genre_name: 'Biografia',
          pages: '352',
          release_year:'1947'
        }
      );

    expect(response.status).to.equal(201);
    expect(response.body).to.
      deep.equal({ message: 'Pessoa cadastrada com sucesso com o id 6' });
  });
          //...
 // }):
  ```
  ### Usamos o stub na função execute de connection de maneira que quando essa função for chamada no teste , ela retornará um array com o objeto insertId:6

  ### utilizamos o request passando o app , e utilizamos o método post na rota /books

  ### utilizamos .send para enviar o objeto do req.body 

  ### o expect foi definido de acordo com o return res.status 201 e o response.body com a mesma mensagem do endPoint com o id do sinon.stub.
---
  ## 3. Teste o endPoint PUT/books/1,utilizano o mesmo banco de dados,acrescente os códigos abaixo na aplicação  e realize os testes.
  ---

  ```
//describe('CRUD na rota /books', function () {
  //...
 it('Testando a alteração de um livro com o id 1', async function () { //put
            sinon.stub(connection, 'execute').resolves([{ affectedRows: 1 }]);
            const response = await chai
              .request(app)
              .put('/books/1')
              .send( {
                name_book: 'Brigerton O visconde que me amava',
                author_name: 'Julia Quinn',
                genre_name: 'Romance',
                pages: '304',
                release_year:'2000'
              }
              
              );
        
            expect(response.status).to.equal(200);
            expect(response.body).to
              .deep.equal({ message: 'Pessoa de id 1 atualizada com sucesso' });
          });
          //...
 // }):
  ```
   ### Usamos o stub na função execute de connection de maneira que quando essa função for chamada no teste , ela retornará um array com o objeto affectedRows: 1(linhas afetadas)

  ### utilizamos o request passando o app , e utilizamos o método put na rota /books

  ### utilizamos .send para enviar o objeto do req.body 

  ### o expect foi definido de acordo com o return res.status 200 e o response.body com a mesma mensagem do endPoint com o número de linhas do sinon.stub.

  ---
  # Gabarito Exercício Bônus
  ## 1° Vamos começar fazendo o teste de integração
  ```
 //describe('CRUD na rota /books', function () {
  //...
 it('Testando a exclusão do book com id 1', async function () { 
            sinon.stub(connection, 'execute').resolves([{ affectedRows: 1 }]);
            const response = await chai
              .request(app)
              .delete('/books/1');
        
            expect(response.status).to.equal(200);
            expect(response.body).to
              .deep.equal({ message: 'Pessoa de id 1 excluída com sucesso' });
          });


          //afterEach(sinon.restore);
   // });
  ```
  ### Usamos o stub na função execute de connection de maneira que quando essa função for chamada no teste , ela retornará um array com o objeto affectedRows: 1(linhas afetadas)

  ### utilizamos o request passando o app , e utilizamos o método delete na rota /books

  ### o expect foi definido de acordo com o return res.status 200 e o response.body com a mesma mensagem do endPoint com o número de linhas do sinon.stub.

  
  ##  2° passo Agora no arquivo livrariadb.js vamos fazer a função delete
  ```
  //...
   const deleteBook = (id) => conn.execute('DELETE  FROM books WHERE id = ?', [id]);


  //module.exports={update,getAll,insert,findById,
  deleteBook};
  ```
  ### usamos o método conn.execute para se comunicar  e fazemos as querys para excluir  livro que for igual o id.

  ### e exportamos a função.

  ## 3° passo por fim vamos fazer o endpoint delete no arquivo livrariaRoutes.js
  ```
  router.delete('/:id',async(req,res)=>{
  try {
    const { id } = req.params;
    const [result] = await livrariadb.deleteBook(id);
    if (result.affectedRows > 0) {
      res.status(200).json({ message: `Pessoa de id ${id} excluída com sucesso` });
    } else {
      res.status(404).json({ message: 'Pessoa não encontrada' });
    }
  } catch (err) {
    res.status(500).json({ message: err.sqlMessage });
  }
})
```
### recuperamos o id do req.params ,e desestruturamos o result do resultado de da função delete,

### utilizamos if/else  caso alguma linha tenha sido afetada ou não com sua respectiva resposta 


---
# Bons estudos! 

