# Mysql-Database
Mysql learning SQL coding and modeling a database

## Diagrama de Entidade Relacionamento

![alt text](/Imagens/Biblioteca1.png)

## Criação das tabelas
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
CREATE PROCEDURE add_pessoa
	(nome varchar (50),
	 sobrenome varchar(50),
	 cpf char (14),
	 email varchar (255))
BEGIN
	DECLARE cpf_formatado char(14);
	cpf_formatado = format_cpf(cpf);
	INSERT INTO 
		pessoa (nome, sobrenome, email, CPF)
	VALUES 
		(nome, sobrenome, email, cpf_formatado);	
END;

CREATE FUNCTION format_cpf
	(cpf CHAR(14))
RETURNS VARCHAR(14) DETERMINISTIC
BEGIN
	DECLARE cpf_formatado char(14);
	DECLARE custom_exception CONDITION FOR SQLSTATE '45000';
	DECLARE EXIT HANDLER FOR custom_exception
	BEGIN
		SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'CPF deve ter tamanho 11 ou 14';
	END;
	IF length(cpf) = 11 THEN
		SET cpf_formatado = 
		CONCAT(
			SUBSTR(cpf, 1,3), '.', 
			SUBSTR(cpf, 4,3), '.', 
			SUBSTR(cpf, 7,3), '-', 
			SUBSTR(cpf, 10,2)
		);
		RETURN (cpf_formatado);
	ELSEIF length(cpf) = 14 THEN
		RETURN (cpf);
	ELSE
		RESIGNAL custom_exception;
	END IF;
END;

CREATE PROCEDURE read_pessoa ()
BEGIN
	SELECT 
		p.id_pessoa,
		concat(nome, ' ', sobrenome) nome_completo,
		cpf,
		email,
		e.logradouro,
		e.cep
	FROM pessoa p
		LEFT JOIN pessoa_endereco pe ON p.id_pessoa = pe.id_pessoa
		LEFT JOIN endereco e ON pe.id_endereco = e.id_endereco;
END;

CREATE PROCEDURE update_pessoa (
	 IN v_id int,
	 IN v_nome varchar(50),
	 IN v_sobrenome varchar(50),
	 IN v_cpf varchar(14),
	 IN v_flg_status int,
	 IN v_email varchar(255),
	 IN	v_cep varchar(20),
	 IN	v_numero int,
	 IN	v_complemento varchar(55))
BEGIN
	DECLARE v_aux int;
	IF v_nome IS NOT NULL THEN
		UPDATE pessoa SET nome = v_nome
		WHERE id_pessoa = v_id;
	END IF;
	IF v_sobrenome IS NOT NULL THEN
		UPDATE pessoa set sobrenome = v_sobrenome
		WHERE id_pessoa = v_id;
	END IF;
	IF v_cpf IS NOT NULL THEN
		UPDATE pessoa set cpf = format_cpf(v_cpf)
		WHERE id_pessoa = v_id;
	END IF;
	IF v_flg_status IS NOT NULL THEN
		UPDATE pessoa set flg_status_pessoa = v_flg_status
		WHERE id_pessoa = v_id;
	END IF;
	IF v_email IS NOT NULL THEN
		UPDATE pessoa set email = v_email
		WHERE id_pessoa = v_id;
	END IF;
	IF  v_cep IS NOT NULL THEN
		SELECT id_endereco INTO v_aux FROM endereco WHERE cep = v_cep;
		IF v_aux IS NOT NULL THEN
		UPDATE pessoa SET
			numero_endereco = v_numero,
			complemento = v_complemento
		WHERE
			id_pessoa = v_id;
		INSERT INTO
			pessoa_endereco (id_pessoa, id_endereco)
		VALUES
			(v_id, v_aux);
		ELSE
		SIGNAL SQLSTATE '44000' set MESSAGE_TEXT = 'Endereço não cadastrado';
		END IF;
	END IF;
END;

CREATE PROCEDURE delete_pessoa (
	IN v_id int)
BEGIN
	DELETE FROM pessoa_endereco WHERE id_pessoa = v_id;
	DELETE FROM pessoa WHERE id_pessoa = v_id;
END;

CREATE PROCEDURE read_endereco(
	IN v_cep varchar(20)
)
BEGIN
	SELECT 
		id_endereco,
		logradouro,
		cep,
		cidade.nome,
		estado.nome,
		estado.uf
	FROM endereco
		JOIN cidade ON endereco.id_cidade = cidade.id_cidade
		JOIN estado ON cidade.id_estado = estado.id_estado
	WHERE
		endereco.cep = v_cep;
END;

CREATE PROCEDURE insert_endereco (
	v_logradouro varchar(255),
	v_cep varchar(20),
	v_id_cidade int
)
BEGIN 
	DECLARE v_aux int;
	SELECT count(id_endereco) INTO v_aux FROM endereco WHERE cep = v_cep;
	IF v_aux = 0 THEN
		INSERT INTO 
			endereco (logradouro, cep, id_cidade)
		VALUES
			(v_logradouro, v_cep, v_id_cidade);
	ELSE
		SIGNAL SQLSTATE '45000' set MESSAGE_TEXT = 'Endereço já existente';
	END IF;
END;

-- Chamada das Procedure
CALL read_pessoa();
CALL add_pessoa(:nome, :sobrenome, :cpf, :email); 
CALL update_pessoa(:v_id, :v_nome, :v_sobrenome, :v_cpf, :v_flg_status, :v_email, :v_cep, :v_numero, :v_complemento); 
CALL delete_pessoa(:v_id); 

CALL read_endereco(:v_cep); 
CALL insert_endereco(:v_logradouro, :v_cep, :v_id_cidade); 
```
