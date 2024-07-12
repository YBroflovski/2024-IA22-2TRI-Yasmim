# Iniciando um projeto Node.js com TypeScript

Faça uma conta no github, caso ainda não tenha uma. Depois, siga os passos abaixo:

Clique no ícone "+" do canto superior direito e clique em new repository.
Escolha um nome para o repositório e confirme a caixa do Add a README file. Após isso, finalize clicando no botão verde.


Crie um diretório para o projeto e acesse-o pelo vscode, abra o terminal e siga os passos abaixo. 

- Para a instalação do Node js, escreva ou copie o código abaixo no terminal. (OBSERVAÇÃO, PARA ABRIR O TERMINAL, APERTE CTRL + ASPAS). 

```bash
npm init -y
npm install express cors sqlite3 sqlite
npm install --save-dev typescript nodemon ts-node @types/express @types/cors
npx tsc --init
mkdir src
touch src/app.ts
```

## Configurando o `tsconfig.json`

Abra o arquivo e procure a linha onde está escrito ```"outDir": "./",``` e mude para ```"outDir": "./dist",```, da mesma forma, faça o mesmo substituindo ```"rootDirs": [],``` por ```"rootDir": "./src",``` Seu arquivo de configuração do compilador do TypeScript ficará mais ou menos assim:

```json
{
  "compilerOptions": {
    "target": "ES2017",
    "module": "commonjs",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
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

Abra o navegador e acesse `http://localhost:3333` ou aparecerá uma mensagem no canto inferior direito, levando diretamente ao localhost e você verá a mensagem `Hello World`.

## Configurando o banco de dados

Crie um arquivo chamado `database.ts` dentro da pasta `src` e adicione o seguinte código.

```typescript
import { open } from 'sqlite';
import sqlite3 from 'sqlite3';

let instance: sqlite3.Database | null = null;

export async function connect() {
  if (instance) return instance;

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

- C

## Testando a inserção de dados

- Faça uma requisição POST para `http://localhost:3333/users` (para conseguir o local host, é necessário estár com o terminal rodando em ```ndm run dev```, dessa maneira, coloque ecom o seguinte corpo.

```json
{
  "name": "John Doe",
  "email": "
}
```

Se tudo ocorrer bem, você verá a resposta com o usuário inserido.

```json
{
  "id": 1,
  "name": "John Doe",
  "email": "
}
```

## Listando os usuários

Adicione a rota `/users` ao servidor.

```typescript
app.get('/users', async (req, res) => {
  const db = await connect();
  const users = await db.all('SELECT * FROM users');

  res.json(users);
});
```