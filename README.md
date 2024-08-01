# Iniciando um projeto Node.js com TypeScript

### Faça uma conta no github, caso ainda não tenha uma. Depois, siga os passos abaixo:

Clique no ícone "+" do canto superior direito e clique em new repository.
Escolha um nome para o repositório e confirme a caixa do ```Add a README file```. Após isso, finalize clicando no botão verde.

Crie um diretório para o projeto e acesse-o pelo codespace do Github, para abri-lo, acesse ```Code```, clique em ```Codespaces``` e clique no + para criar um arquivo que abrirá automaticamente. (ele tem nomes aleatórios). Para acessá-lo novamente ao sair, faça o mesmo procedimento que fora usado para criar, dessa vez criando no nome abaixo de ```On current branch```(Ele possuirá um ```main*``` pequeno abaixo, indicando seu códico primário), abra o terminal e siga os passos abaixo. 

- Para a instalação do Node js, escreva ou copie o código abaixo no terminal. (OBSERVAÇÃO, PARA ABRIR O TERMINAL, APERTE CTRL + ASPAS). 

```bash
npm init -y
npm install express cors sqlite3 sqlite
npm install --save-dev typescript nodemon ts-node @types/express @types/cors
npx tsc --init
mkdir src
touch src/app.ts
```
- #### Crie um novo arquivo chamado .gitignore. Nele, adicione o seguinte spricpt:
```typescript 
node_modules/
dist/

database.sqlite
```

database.sqlite

## Configurando o `tsconfig.json`

Abra o arquivo e procure a linha onde estará escrito ```"outDir": "./",``` e substitua por ```"outDir": "./dist"``` sem ser comentário ```//```,```, da mesma forma, faça o mesmo substituindo ```"rootDirs": [],``` por ```"rootDir": "./src",```. Seu arquivo de configuração do compilador do TypeScript ficará mais ou menos assim:

```json
{
  "compilerOptions": {
    "target": "ES2017",
    "module": "commonjs",
    "rootDir": "./src",
    "outDir": "./dist", 
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    true"strict": true,
  }
}
```

## Configurando o `package.json`

Substitua o seguinte script ao arquivo chamado `package.json`. No código, estará   ```"test": "echo \"Error: no test specified\" && exit 1"
  },```, copie e cole sobre ele:

```json
"scripts": {
  "dev": "npx nodemon src/app.ts"
},
```

## Criando arquivo inicial do servidor

No src, crie um arquivo chamado `src/app.ts` e adicione o seguinte código:

```typescript
import express from 'express';
import cors from 'cors';

const port = 3333;
const app = express();

app.use(cors());
app.use(express.json());

app.get('/', (req, res) => {
  res.send('Hello World');
});

app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});
```

## Inicializando o servidor

No terminal, escreva:

```bash
npm run dev
```

Se tudo ocorrer bem, você verá a mensagem `Server running on port 3333` no terminal.

## Testando o servidor

Abra o navegador e acesse `http://localhost:3333` ou no momento em que dar a ordem de ```npm run dev``` aparecerá uma mensagem no canto inferior direito escrito ```abrir no navegador```, isso te levará diretamente ao localhost e você verá a mensagem `Hello World`.

## Configurando o banco de dados

Crie um arquivo chamado `database.ts` dentro da pasta `src` e adicione o seguinte código:

```typescript
import { open, Database } from 'sqlite';
import sqlite3 from 'sqlite3';
let instance: Database | null = null;
export async function connect() {
  if (instance !== null) 
      return instance;
  const db = await open({
     filename: './src/database.sqlite',
     driver: sqlite3.Database
   });
  await db.exec(`
    CREATE TABLE IF NOT EXISTS users (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      name TEXT,
      email TEXT
    )
  `);
  instance = db;
  return db;
}
```

## Adicionando o banco de dados ao servidor

Substitua o código do arquivo ```src/app.ts``` para:

```typescript
import express from 'express';
import cors from 'cors';
import { connect } from './database';

const port = 3333;
const app = express();

app.use(cors());
app.use(express.json());

app.get('/', (req, res) => {
  res.send('Hello World');
});

app.get('/users', async (req, res) => {
  const db = await connect();
  const users = await db.all('SELECT * FROM users');
  res.json(users);
});

app.post('/users', async (req, res) => {
  const db = await connect();
  const { name, email } = req.body;
  const result = await db.run('INSERT INTO users (name, email) VALUES (?, ?)', [name, email]);
  const user = await db.get('SELECT * FROM users WHERE id = ?', [result.lastID]);
  res.json(user);
});

app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});
```

## Postman

- Primeiramente, para instalá-lo acesse https://www.postman.com. Instale, execute e crie uma conta.

- Após feito, clique no + e crie uma New Collection, criando um New Request.

- Clique em ```Get```, trocando para ```Post```.

- Clique em ```Body``` e aperte na bolinha escrito ```raw```, desse jeito, abrirá um JSON.

## Testando a inserção de dados

- Faça uma requisição POST para `http://localhost:3333/users` (para conseguir o local host, é necessário que o terminal ainda esteja rodando a porta, caso não estiver, escreva novamente ```npm run dev```), dessa maneira, coloque o seguinte corpo na parte JSON:

```json
{
  "name": "John Doe",
  "email": "johndoe@mail.com"
}
```
Para que o código funcione, é necessário tornar a porta do localhost pública. (Como fazer: ao lado de ```terminal```, está escrito ```portas```. Se estiver rodando, terá um link para a porta 3333. Clique com o botão direito sobre o link e vá em visibilidade da porta, a tornando ```pública```). Após feito, volte para o Postman e clique no botão laranja escrito ```send```. Se tudo ocorrer bem, você verá a resposta com o usuário inserido.

```json
{
  "id": 1,
  "name": "John Doe",
  "email": "johndoe@mail.com"
}
```

## Listando os usuários

Adicione a rota `/users` ao servidor, no arquivo ```src/app.ts```, abaixo do ({}); do res.json(user);, colocando o código seguinte:


```typescript
app.get('/users', async (req, res) => {
  const db = await connect();
  const users = await db.all('SELECT * FROM users');
  res.json(users);
});
```

## Editando um usuário

Adicione a rota `/users/:id` ao servidor.

```typescript
app.put('/users/:id', async (req, res) => {
  const db = await connect();
  const { name, email } = req.body;
  const { id } = req.params;
  await db.run('UPDATE users SET name = ?, email = ? WHERE id = ?', [name, email, id]);
  const user = await db.get('SELECT * FROM users WHERE id = ?', [id]);
  res.json(user);
});
```
## Deletando um usuário

Adicione a rota `/users/:id` ao servidor.

```typescript
app.delete('/users/:id', async (req, res) => {
  const db = await connect();
  const { id } = req.params;

  await db.run('DELETE FROM users WHERE id = ?', [id]);

  res.json({ message: 'User deleted' });
});
```