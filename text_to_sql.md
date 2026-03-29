[Back](./README.md)

---

## 📑 Table of Contents

| Section | Topic |
|---------|-------|
| [1](#1-understanding-user-input-nlp-side) | Understanding User Input |
| [2](#2-schema-awareness) | Schema Awareness |
| [3](#3-llm-prompting--design) | LLM Prompting |
| [4](#4-sql-generation-challenges) | SQL Generation |

---

# Text to SQL Design
1. **Understanding User Input (NLP side)**
2. **Schema Awareness**
3. **LLM Prompting & Design**
4. **SQL Generation Challenges**
   	1. **SQL complexity**
	2. **Performance**
	3. **Safety: Prevent SQL injection or harmful queries (e.g., `DROP TABLE`)**
    4. **Dialect differences: SQL dialects differ (`Postgres`, `MySQL`, `BigQuery`, `Snowflake`, etc.)**
5. **System Design**
	1. **Interactive clarification: Ask follow-up questions if ambiguity exists instead of guessing.**
	2. **Error handling: If SQL fails, recover gracefully (retry with different phrasing or constraints)**


## 1. Understanding User Input (NLP side)
**Problems**: Ambiguity, vagueness, multilingual phrasing.

**Solutions**:
- LLMs: Directly prompt
- Auto-translate NL → English → SQL (safer if your SQL training data is mostly English).
  
## 2. Schema Awareness
**Problems**: Mapping multilingual input → schema fields, evolving schemas.

**Solutions**: 
- Schema-augmented prompting: Always include schema (table + column names + descriptions) in the prompt.
- Dynamic schema injection: Fetch schema at runtime → insert into prompt → keeps LLM updated.
- Strategies for Handling Large Schemas
	- Schema Retrieval (RAG for schema)
		- Encode table & column names (and descriptions) with embeddings (multilingual if needed).
		- Given the user query, retrieve top-k relevant tables/columns (e.g., K = 5–10).
 	- Store schema relationships (foreign keys, joins) in a graph DB or structured knowledge base.
	- At query time, traverse the graph → retrieve only the join path that connects relevant tables.


## 3. LLM Prompting & Design
**Problems**: Hallucinations, wrong queries, SQL dialects.

**Solutions**: 
- Use SQL grammar to restrict output tokens.
- Validate with `sqlglot` or DB parser before execution.

## 4. SQL Generation Challenges
- **SQL complexity**
- **Performance**
	- Before you run the SQL, analyze and block unsafe queries.
 	- Use tools like `sqlglot` or `[sqlparse]` to parse queries.
	- Automatically block or rewrite:
		- ```SELECT *``` → replace with ```SELECT column1, column2``` …
		- ```JOIN``` without ```ON``` → block (likely Cartesian product)
	- Execution Guards
 		- Dry-run with `EXPLAIN/EXPLAIN ANALYZE`:
   			- Run `EXPLAIN` first to check cost/row estimate.
			- If query plan looks too heavy (e.g., >1M rows scan), reject or ask for refinement.
  		- Queries without LIMIT → add LIMIT 100 by default or 
  	- Interactive User Loop 
  		- If query looks unsafe -> Let user approve or edit the generated SQL.
- **Error Handling**
	- Irretrievable / illogical questions
 		- User asks for data that doesn’t exist (e.g., “customers born on Mars”).
   	- Ambiguous input
   		- uery vague (*“top customers” with no metric*).
   	- Syntax errors in generated SQL
   	 	- LLM outputs invalid SQL (missing comma, bad alias).
  	- Runtime DB errors
  		- Column/table doesn’t exist.
  	 	- Query is too expensive / times out.
  	  	- Data type mismatch (string vs int).
	- Check if all referenced tables/columns exist in schema.
	- Constrained decoding / validation loop:
		- Generate SQL → parse with sqlglot or sqlparse.
		- If invalid → ask LLM: “Fix this SQL, error was: missing comma near...”
		- Retry automatically before showing user.
	- Safety by Contract
 		- Always add LIMIT.
   		- Never return destructive SQL (*Count number of joins → if > N, mark as “potentially expensive.”*, *look for GROUP BY + ORDER BY without LIMIT.*, *Look for no filters in WHERE (full table scan risk).*)
     	- Guarantee schema correctness (static check).
- **Safety**
	- Run DB in read-only mode unless explicitly allowed.
	- Block destructive queries (`DROP`, `ALTER`, `DELETE`, `TRUNCATE`, `RENAME`, `INSERT`, `GRANT`, `REVOKE`, `EXECUTE`) aka Simple Keyword Blocklist
 	- SQL Parsing Use a parser (e.g., `sqlglot`, `sqlparse`) to parse query into AST (abstract syntax tree).
	- Add a SQL validation step: only whitelisted tables and columns can be used.
 	- Block access to internal/system tables like `pg_catalog`, `mysql.user`.

 ### Prompt Example

```python
from datetime import datetime

now = datetime.now()
now = now.strftime("%B %d, %Y at %I:%M %p")

PROMPT = "\n".join([
    "You are given a SQL Server database schema and a user query in natural language.",
    "Your task is to convert the user query into a valid and optimized SQL Server SELECT statement.",
    "Rules:",
    "1. Only generate SELECT statements. Never generate DELETE, UPDATE, or INSERT statements.",
    "2. Ensure that the generated SQL query is optimized for performance and does not unnecessarily increase execution time.",
    "3. If you cannot generate a valid SQL query, return exactly: ```NONE: <reason why SQL cannot be generated>```",
    "4. Always include a TOP ≤ 100 unless user specifies otherwise",
    "5. Today is ",
    str(now),
    "\nDatabase Schema:",
    "{schema}",
    "\nUser Query:",
    "{query}",
    "\nYour SQL Query:",

])
```

### Big Problem:
If you ask about the number of `Clients`, but the LLM generate `SELECT COUNT(*) FROM Integrations` how would you know that was wrong answer?

### Suggestion roadmap to start the solution.
- **Understanding User Input/Intent *(NLP side & Prompt Engineering)***
  
- **Error Handling**
	- Irretrievable / illogical questions / Ambiguous input
 		- User asks for data that doesn’t exist (e.g., “customers born on Mars”).
   	- Syntax errors in generated SQL
   	 	- LLM outputs invalid SQL *(missing comma, bad alias)*.
   	  	- Column/table doesn’t exist.
   	  	  
- **Safety**
	- Block destructive queries (`DROP`, `ALTER`, `DELETE`, `TRUNCATE`, `RENAME`, `INSERT`, `GRANT`, `REVOKE`, `EXECUTE`) aka Simple Keyword Blocklist
   
- **Schema Awareness**
	- Big schemas
   
- **SQL complexity** & **Performance**
    - ...
   
---
[Back](./README.md)