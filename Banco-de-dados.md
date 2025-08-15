Em um **projeto fullstack**, o **banco de dados** é a camada responsável por **armazenar, organizar e disponibilizar** as informações que o sistema utiliza.
Ele funciona como um “arquivo digital estruturado” onde ficam dados de usuários, produtos, pedidos, relatórios etc., sendo acessado e manipulado pelo **backend** por meio de consultas.

---

## 1. **Função do banco de dados em um projeto fullstack**

* **Armazenar dados** de forma estruturada (tabelas, documentos, grafos).
* **Fornecer informações** ao backend quando solicitado.
* **Permitir inserção, atualização e exclusão** de registros.
* **Garantir integridade e segurança** dos dados.
* **Possibilitar consultas complexas** para geração de relatórios.

---

## 2. **Principais tecnologias para criar e manipular bancos de dados**

### **Bancos de Dados Relacionais (SQL)**

Usam tabelas com linhas e colunas.
Exemplos:

* **MySQL / MariaDB**
* **PostgreSQL**
* **Microsoft SQL Server**
* **Oracle Database**

📌 Acesso e manipulação via **SQL** (Structured Query Language).

---

### **Bancos de Dados Não Relacionais (NoSQL)**

Armazenam dados em formatos mais flexíveis (documentos, chave-valor, grafos).
Exemplos:

* **MongoDB** (documentos JSON)
* **Redis** (chave-valor, alta velocidade)
* **Cassandra** (colunas distribuídas)
* **Neo4j** (grafos)
* **CouchDB** (documentos distribuídos)

📌 Manipulados via APIs próprias ou query languages específicos.

---

### **Ferramentas e tecnologias de manipulação**

* **Linguagens**: SQL, PL/pgSQL, T-SQL, Mongo Query Language.
* **ORMs** (Object Relational Mappers): Sequelize (Node.js), Prisma, TypeORM, Hibernate (Java), Entity Framework (.NET), SQLAlchemy (Python).
* **Ferramentas gráficas**: phpMyAdmin, pgAdmin, DBeaver, MySQL Workbench.
* **Migrações e versionamento**: Flyway, Liquibase, Migrate.

---

## 3. **Aplicações dos bancos de dados**

* **Cadastro de usuários e autenticação**.
* **Controle de estoque e vendas**.
* **Geração de relatórios gerenciais**.
* **Histórico de transações**.
* **Armazenamento de conteúdo dinâmico** (posts, comentários, mídias).
* **Integração com APIs** (salvando dados recebidos ou processados).

---

## 4. **Limitações e desafios**

* **Escalabilidade**: bancos relacionais podem ter dificuldade em lidar com grandes volumes de dados distribuídos.
* **Custo**: bancos corporativos (Oracle, SQL Server) têm licenciamento caro.
* **Complexidade de manutenção**: precisa de backup, tuning e segurança constantes.
* **Tempo de resposta**: consultas mal otimizadas podem gerar lentidão.
* **Segurança**: vulnerabilidade a ataques como SQL Injection se não houver boas práticas.
* **Disponibilidade**: quedas no banco comprometem todo o sistema.

---

