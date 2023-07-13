# Mysql-Database
Mysql learning SQL coding and modeling a database

## SQL Codes
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
