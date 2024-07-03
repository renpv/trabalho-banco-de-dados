# DISCIPLINA | Experiência Profissional: Banco de Dados Relacional

O presente trabalho faz parte da disciplina Experiência Profissional: Banco de Dados Relacional (Uniasselvi) e tem como orientador o professor [Lucas Martins dos Santos](http://lattes.cnpq.br/4116998659541235).

## Membros do grupo:
 - Francisco Diogo Gomes Viana - Sistema da Informação (5487452) 
 - Mateus Meneses Neco - Engenharia de Software (5571108)
 - Matheus Nunes Barbosa - Engenharia de Software (5045033)
 - Renato Farias de Paiva - Sistema da Informação (6449757)

## Modelagem física

Abaixo segue o código para criação do banco de dados, suas tabelas e respectivos relacionamentos

```sql
CREATE DATABASE ecommerce;
USE ecommerce;


CREATE TABLE Categoria (
    id_categoria INT PRIMARY KEY AUTO_INCREMENT,
    nome VARCHAR(100) NOT NULL
);
CREATE TABLE Produto (
    id_produto INT PRIMARY KEY AUTO_INCREMENT,
    nome VARCHAR(100) NOT NULL,
    descricao TEXT,
    preco DECIMAL(10, 2) NOT NULL,
    estoque INT NOT NULL,
    id_categoria INT,
    FOREIGN KEY (id_categoria) REFERENCES Categoria(id_categoria)
);
CREATE TABLE Carrinho (
    id_carrinho INT PRIMARY KEY AUTO_INCREMENT,
    data_criacao TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
CREATE TABLE Item_Carrinho (
    id_item INT PRIMARY KEY AUTO_INCREMENT,
    id_carrinho INT,
    id_produto INT,
    quantidade INT NOT NULL,
    FOREIGN KEY (id_carrinho) REFERENCES Carrinho(id_carrinho),
    FOREIGN KEY (id_produto) REFERENCES Produto(id_produto)
);
CREATE TABLE Pedido (
    id_pedido INT PRIMARY KEY AUTO_INCREMENT,
    data_pedido TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    total DECIMAL(10, 2) NOT NULL,
    status VARCHAR(50) NOT NULL
);
CREATE TABLE Item_Pedido (
    id_item INT PRIMARY KEY AUTO_INCREMENT,
    id_pedido INT,
    id_produto INT,
    quantidade INT NOT NULL,
    preco_unitario DECIMAL(10, 2) NOT NULL,
    FOREIGN KEY (id_pedido) REFERENCES Pedido(id_pedido),
    FOREIGN KEY (id_produto) REFERENCES Produto(id_produto)
);
CREATE TABLE Envio (
    id_envio INT PRIMARY KEY AUTO_INCREMENT,
    id_pedido INT,
    endereco VARCHAR(255) NOT NULL,
    cidade VARCHAR(100) NOT NULL,
    estado VARCHAR(50) NOT NULL,
    cep VARCHAR(20) NOT NULL,
    data_envio TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    metodo_envio VARCHAR(50) NOT NULL,
    FOREIGN KEY (id_pedido) REFERENCES Pedido(id_pedido)
);
CREATE TABLE Pagamento (
    id_pagamento INT PRIMARY KEY AUTO_INCREMENT,
    id_pedido INT,
    metodo_pagamento VARCHAR(50) NOT NULL,
    status_pagamento VARCHAR(50) NOT NULL,
    data_pagamento TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (id_pedido) REFERENCES Pedido(id_pedido)
);

```

## Consultas

Consultas Básicas
1.	Listar todos os produtos:
```sql
SELECT * FROM Produto;
```

2.	Listar todas as categorias:
```sql
SELECT * FROM Categoria;
```

3.	Listar todos os pedidos:
```sql
SELECT * FROM Pedido;
```

Consultas com Filtros
4.	Listar produtos de uma categoria específica:
```sql
SELECT * FROM Produto
WHERE id_categoria = 1; -- Substitua 1 pelo ID da categoria desejada
```

5.	Listar produtos cujo estoque está abaixo de um valor específico:
```sql
SELECT * FROM Produto
WHERE estoque < 10; -- Substitua 10 pelo valor de estoque desejado
```

6.	Buscar pedidos de um determinado status:
```sql
SELECT * FROM Pedido
WHERE status = 'PROCESSANDO'; -- Substitua 'PROCESSANDO' pelo status desejado
```

### Consultas com Junções
7.	Listar produtos com suas categorias:
```sql
SELECT p.id_produto, p.nome AS produto_nome, c.nome AS categoria_nome
FROM Produto p
JOIN Categoria c ON p.id_categoria = c.id_categoria;
```

8.	Listar itens no carrinho com detalhes do produto:
```sql
SELECT ic.id_item, c.id_carrinho, p.nome AS produto_nome, ic.quantidade
FROM Item_Carrinho ic
JOIN Carrinho c ON ic.id_carrinho = c.id_carrinho
JOIN Produto p ON ic.id_produto = p.id_produto;
```

9.	Listar itens de pedido com detalhes do produto e pedido:
```sql
SELECT ip.id_item, p.id_pedido, pr.nome AS produto_nome, ip.quantidade, ip.preco_unitario
FROM Item_Pedido ip
JOIN Pedido p ON ip.id_pedido = p.id_pedido
JOIN Produto pr ON ip.id_produto = pr.id_produto;
```

### Consultas de Agregação
10.	Calcular o subtotal, impostos e total do carrinho:
```sql
SELECT c.id_carrinho,
       SUM(p.preco * ic.quantidade) AS subtotal,
       SUM(p.preco * ic.quantidade) * 0.1 AS impostos, -- Supondo 10% de impostos
       SUM(p.preco * ic.quantidade) + SUM(p.preco * ic.quantidade) * 0.1 AS total
FROM Carrinho c
JOIN Item_Carrinho ic ON c.id_carrinho = ic.id_carrinho
JOIN Produto p ON ic.id_produto = p.id_produto
GROUP BY c.id_carrinho;
```
### Consultas de Atualização e Exclusão

11.	Atualizar informações de um produto:
```sql
UPDATE Produto
SET nome = 'Novo Nome', preco = 19.99, descricao = 'Nova Descrição', estoque = 50
WHERE id_produto = 1; -- Substitua 1 pelo ID do produto a ser atualizado
```

12.	Remover um produto do catálogo:
```sql
DELETE FROM Produto
WHERE id_produto = 1; -- Substitua 1 pelo ID do produto a ser removido
```

### Consultas para Relatórios

13.	Listar o histórico de pedidos com detalhes de envio e pagamento:
```sql
SELECT p.id_pedido, p.data_pedido, p.total, p.status,
       e.endereco, e.cidade, e.estado, e.cep, e.data_envio, e.metodo_envio,
       pg.metodo_pagamento, pg.status_pagamento, pg.data_pagamento
FROM Pedido p
LEFT JOIN Envio e ON p.id_pedido = e.id_pedido
LEFT JOIN Pagamento pg ON p.id_pedido = pg.id_pedido;
```

14.	Listar produtos mais vendidos:
```sql
SELECT pr.nome AS produto_nome, SUM(ip.quantidade) AS quantidade_vendida
FROM Item_Pedido ip
JOIN Produto pr ON ip.id_produto = pr.id_produto
GROUP BY pr.nome
ORDER BY quantidade_vendida DESC;
```

### Inserções

1.	Adicionar uma nova categoria:
```sql
INSERT INTO Categoria (nome) VALUES ('Eletrônicos');
```

2.	Adicionar um novo produto:
```sql
INSERT INTO Produto (nome, descricao, preco, estoque, id_categoria) 
VALUES ('Smartphone', 'Smartphone de última geração', 999.99, 50, 1); -- Substitua 1 pelo ID da categoria correspondente
```

3.	Adicionar um novo carrinho:
```sql
INSERT INTO Carrinho (data_criacao) VALUES (CURRENT_TIMESTAMP);
```

4.	Adicionar um item ao carrinho:
```sql
INSERT INTO Item_Carrinho (id_carrinho, id_produto, quantidade)
VALUES (1, 1, 2); -- Substitua 1 pelos IDs correspondentes do carrinho e produto, e 2 pela quantidade desejada
```

5.	Criar um novo pedido:
```sql
INSERT INTO Pedido (data_pedido, total, status) 
VALUES (CURRENT_TIMESTAMP, 1999.98, 'PROCESSANDO'); -- Substitua 1999.98 pelo total do pedido e 'PROCESSANDO' pelo status desejado
```

6.	Adicionar um item ao pedido:
```sql
INSERT INTO Item_Pedido (id_pedido, id_produto, quantidade, preco_unitario)
VALUES (1, 1, 2, 999.99); -- Substitua 1 pelos IDs correspondentes do pedido e produto, 2 pela quantidade e 999.99 pelo preço unitário
```

7.	Adicionar informações de envio:
```sql
INSERT INTO Envio (id_pedido, endereco, cidade, estado, cep, data_envio, metodo_envio)
VALUES (1, '123 Rua Principal', 'Cidade Exemplo', 'Estado Exemplo', '12345-678', CURRENT_TIMESTAMP, 'Correios'); -- Substitua pelos valores apropriados
```

8.	Adicionar informações de pagamento:
```sql
INSERT INTO Pagamento (id_pedido, metodo_pagamento, status_pagamento, data_pagamento)
VALUES (1, 'Cartão de Crédito', 'Pago', CURRENT_TIMESTAMP); -- Substitua pelos valores apropriados
Atualizações
```

9.	Atualizar informações de um produto:
```sql
UPDATE Produto
SET nome = 'Smartphone Atualizado', preco = 1099.99, descricao = 'Descrição atualizada', estoque = 45
WHERE id_produto = 1; -- Substitua 1 pelo ID do produto a ser atualizado
```

10.	Atualizar quantidade de um item no carrinho:
```sql
UPDATE Item_Carrinho
SET quantidade = 3
WHERE id_item = 1; -- Substitua 1 pelo ID do item no carrinho a ser atualizado
```

11.	Atualizar status de um pedido:
```sql
UPDATE Pedido
SET status = 'ENVIADO'
WHERE id_pedido = 1; -- Substitua 1 pelo ID do pedido a ser atualizado
```

Exclusões
12.	Remover um produto do catálogo:
```sql
DELETE FROM Produto
WHERE id_produto = 1; -- Substitua 1 pelo ID do produto a ser removido
```

13.	Remover um item do carrinho:
```sql
DELETE FROM Item_Carrinho
WHERE id_item = 1; -- Substitua 1 pelo ID do item no carrinho a ser removido
```

14.	Remover um pedido:
```sql
DELETE FROM Pedido
WHERE id_pedido = 1; -- Substitua 1 pelo ID do pedido a ser removido
```

15.	Remover uma categoria:
```sql
DELETE FROM Categoria
WHERE id_categoria = 1; -- Substitua 1 pelo ID da categoria a ser removida
Consultas para Verificação
```

16.	Verificar produtos na categoria 'Eletrônicos':
```sql
SELECT * FROM Produto
WHERE id_categoria = (SELECT id_categoria FROM Categoria WHERE nome = 'Eletrônicos');
```

17.	Verificar itens no carrinho:
```sql
SELECT * FROM Item_Carrinho
WHERE id_carrinho = 1; -- Substitua 1 pelo ID do carrinho desejado
```

18.	Verificar detalhes de um pedido:
```sql
SELECT * FROM Pedido
WHERE id_pedido = 1; -- Substitua 1 pelo ID do pedido desejado
```



