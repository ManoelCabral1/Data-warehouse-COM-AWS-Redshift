# Data warehouse COM AWS Redshift
![modelagem domensional dw](https://github.com/ManoelCabral1/Prints/blob/main/redshift.png)

## O que é Redshift?
O Amazon Redshift é um produto de data warehouse que faz parte da maior plataforma de computação em nuvem Amazon Web Services . [1] Ele é construído em cima da tecnologia da empresa de armazenamento de dados de processamento paralelo massivo (MPP) ParAccel (mais tarde adquirida pela Actian ), [2] para lidar com conjuntos de dados em grande escala e migrações de banco de dados.(wikipedia)

O Amazon Redshift usa SQL para analisar dados estruturados e semiestruturados em data warehouses, bancos de dados operacionais e data lakes. É baseado no postgresql e orientado a coluna. Portanto realiza a consulta de grande volume de dados com rapidez.

Tendo em vista essa rapidez e capacidade de processamento pensei numa estrutura de DW moderna, onde os dados para consulta ficam armazenados numa tabela fato desnormalizada, sendo de mais fácil consulta por usuários sem conhecimentos em SQL, uma vez que não há necessidade de realizar joins entre as tabelas do DW.

Os dados usados para análise são de vendas de um ecomerce de material para ciclismo. Os arquivos foram armazenados num bucket do AWS S3.

### Modelagem dimensional
![modelagem domensional dw](https://github.com/ManoelCabral1/Prints/blob/main/dimensional.png)

### Script de criação e carga das tabelas

```
set datestyle to 'SQL,DMY';

CREATE TABLE Vendedores(
  IDVendedor int  PRIMARY KEY,
  Nome Varchar(50)
);

CREATE TABLE Produtos(
  IDProduto int  PRIMARY KEY,
  Produto Varchar(100),
  Preco Numeric(10,2)
);

CREATE TABLE Clientes(
  IDCliente int   PRIMARY KEY,
  Cliente Varchar(50),
  Estado Varchar(2),
  Sexo Char(1),
  Status Varchar(50)
);

CREATE TABLE Vendas(
  IDVenda int   PRIMARY KEY,
  IDVendedor int references Vendedores (IDVendedor),
  IDCliente int references Clientes (IDCliente),
  Data Date,
  Total Numeric(10,2)
);

CREATE TABLE ItensVenda (
    IDProduto int ,
    IDVenda int ,
    Quantidade int,
    ValorUnitario decimal(10,2),
    ValorTotal decimal(10,2),
	Desconto decimal(10,2),
    PRIMARY KEY (IDProduto, IDVenda)
);
```

### Script de carga nas tabelas dimensão

```
copy vendas from 's3://bikeshopbucket/dados_bikes_shop/vendas.csv'
credentials 'aws_access_key_id=xxxxxxxxxxxxxxxxxxx;aws_secret_access_key=xxxxxxxxxxxxxxxxxxxxxx'
region 'sa-east-1'
delimiter ';'
IGNOREHEADER 1
DATEFORMAT 'DD/MM/YYYY';

```
O processo de carga nas tabelas é o mesmo somente alterando o nome da tabela e o URI de acesso ao arquivo no datalake.
### Script de carga na tabela fato

```
select cliente, data, nome as vendedor, produto, quantidade, total 
into fatovendas
from vendas v
inner join clientes c on (c.idcliente = v.idcliente)
inner join itensvenda i on (i.idvenda = v.idvenda)
inner join produtos p on (p.idproduto = i.idproduto)
inner join vendedores vd on (vd.idvendedor = v.idvendedor)
```
### consulta vendas por vendedor

```
SELECT vendedor, sum(quantidade) as total_produtos, sum(total) as total_vendido  FROM "ed"."public"."fatovendas"
group by 1 order by 3 desc;
```
Essas consultas podem ser acessadas por vários aplicativos de BI como: PowerBI, DataStudio, ou acessados via python ou outra linguagem, desde que se trenha as credenciais de acesso do cluster do redshift.

### Exemplo de relatório no Google Datastudio com os dado do DW.
![relatório](C:\Users\EBMquintto\Pictures\relatorio-dw.png)

