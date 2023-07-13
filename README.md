# Mysql-Database
Mysql learning SQL coding and modeling a database

##Diagrama de Entidade Relacionamento

##Criação das tabelas
```sql
CREATE TABLE pessoa(
	id_pessoa int AUTO_INCREMENT,
	nome varchar(50) NOT NULL,
	sobrenome varchar(50),
	cpf char(14) NOT NULL,
	flg_status_pessoa int DEFAULT 0,
	numero_endereco int,
	complemento varchar(55),
	PRIMARY KEY (id_pessoa)
);

CREATE TABLE telefone (
	id_telefone int AUTO_INCREMENT,
	numero varchar(50) NOT NULL,
	descricao varchar(100),
	id_pessoa int NOT NULL,
	PRIMARY KEY (id_telefone),
	CONSTRAINT fk_telefone_pessoa
		FOREIGN KEY (id_pessoa) REFERENCES pessoa(id_pessoa)
);

CREATE TABLE estado (
	id_estado int AUTO_INCREMENT,
	nome varchar(255) NOT NULL,
	localidade varchar(255),
	uf varchar(50),
	PRIMARY KEY (id_estado)
);

CREATE TABLE cidade (
	id_cidade int AUTO_INCREMENT,
	nome varchar(255) NOT NULL,
	id_estado int NOT NULL,
	PRIMARY KEY (id_cidade),
	CONSTRAINT fk_cidade_estado
		FOREIGN KEY (id_estado) REFERENCES estado (id_estado)
);

CREATE TABLE endereco (
	id_endereco int AUTO_INCREMENT,
	logradouro varchar(255) NOT NULL,
	cep varchar(20),
	id_cidade int NOT NULL,
	PRIMARY KEY (id_endereco),
	CONSTRAINT fk_endereco_cidade
		FOREIGN KEY (id_cidade) REFERENCES cidade (id_cidade)
);

CREATE TABLE pessoa_endereco (
	id_pessoa int NOT NULL,
	id_endereco int NOT NULL,
	PRIMARY KEY (id_pessoa, id_endereco),
	CONSTRAINT rl_endereco_pessoa
		FOREIGN KEY (id_endereco) REFERENCES endereco(id_endereco),
	CONSTRAINT rl_pessoa_endereco
		FOREIGN KEY (id_pessoa) REFERENCES pessoa(id_pessoa)
);

CREATE TABLE livro (
	id_livro int AUTO_INCREMENT,
	titulo varchar(255) NOT NULL,
	autor varchar(255) NOT NULL,
	ISBN varchar(50),
	editora varchar(255) NOT NULL,
	ano_publicacao date,
	flg_status_livro tinyint DEFAULT 0,
	PRIMARY KEY (id_livro)
);

CREATE TABLE forma_pagamento(
	id_forma_pagamento int AUTO_INCREMENT,
	descricao varchar(255) NOT NULL,
	tipo enum('Credito', 'Debito', 'A vista'),
	PRIMARY KEY (id_forma_pagamento)
);

CREATE TABLE pagamento(
	id_pagamento int AUTO_INCREMENT,
	valor int NOT NULL,
	id_forma_pagamento int NOT NULL,
	parcelas int DEFAULT 1,
	descricao varchar(255),
	data_pagamento date NOT NULL,
	PRIMARY KEY (id_pagamento),
	CONSTRAINT fk_pagamento_has_forma
		FOREIGN KEY (id_forma_pagamento) REFERENCES forma_pagamento(id_forma_pagamento)
);


CREATE TABLE emprestimo (
	id_emprestimo int AUTO_INCREMENT,
	id_pessoa int NOT NULL,
	data_retirada date NOT NULL,
	data_prevista_devolucao date NOT NULL,
	data_devolucao date,
	id_pagamento int NOT NULL,
	PRIMARY KEY (id_emprestimo),
	CONSTRAINT fk_emprestimo_pessoa
		FOREIGN KEY (id_pessoa) REFERENCES pessoa (id_pessoa),
	CONSTRAINT fk_emprestimo_pagamento
		FOREIGN KEY (id_pagamento) REFERENCES pagamento(id_pagamento)
);

CREATE TABLE livro_emprestimo(
	id_livro int NOT NULL,
	id_emprestimo int NOT NULL,
	quantidade int DEFAULT 1,
	PRIMARY KEY (id_livro, id_emprestimo),
	CONSTRAINT fk_livro_emprestimo
		FOREIGN KEY (id_livro) REFERENCES livro(id_livro),
	CONSTRAINT fk_emprestimo_livro
		FOREIGN KEY (id_emprestimo) REFERENCES emprestimo(id_emprestimo)
);
```

## Função para insert de pessoa
```sql
CREATE TABLE pessoa(
	id_pessoa int auto_increment,
	nome varchar(50) not null,
	sobrenome varchar(50),
	cpf char(14) not null,
	flg_status_pessoa int default 0,
	primary key (id_pessoa)
);

CREATE FUNCTION add_pessoa
	(nome varchar (50),
	 cpf char (11),
	 email varchar (255))
RETURNS varchar(50) DETERMINISTIC
BEGIN
	DECLARE cpf_formatado char(14);
	SET cpf_formatado = format_cpf(cpf);
	INSERT INTO 
		pessoa (nome, email, CPF) 
	VALUES 
		(nome, email, cpf_formatado);
	RETURN ('Pessoa Incluida');
END;

CREATE FUNCTION format_cpf
	(cpf CHAR(11))
RETURNS VARCHAR(14) DETERMINISTIC
BEGIN
	DECLARE cpf_formatado char(14);
	SET cpf_formatado = 
		CONCAT(
			SUBSTR(cpf, 1,3), '.', 
			SUBSTR(cpf, 4,3), '.', 
			SUBSTR(cpf, 7,3), '-', 
			SUBSTR(cpf, 10,2)
		);
	RETURN (cpf_formatado);
END;
```
