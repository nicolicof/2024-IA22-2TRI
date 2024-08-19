# 2024-IA22-2TRI
Vamos fazer um projeto.

Se você quizer terminar mais rápido:

**NÃO ABRA O VSCODE POR CONTA PRÓPRIA**
**NÃO FECHE NENHUMA JANELA** que você abrir;
**SEMPRE SALVE OS ARQUIVOS** com *CTRL+S*;

# ALGUNS PRÉ-REQUISITOS
1. Verifique se o programa Node está instalado.

    1. Busque por "cmd" no Windows. Vai abrir essa janela👇 
    ![Terminal](./imgs/terminal.png)

    2. **NUNCA FECHE ESSA JANELA PELO AMOR DE DEUS**

    3. Copie e cole esse código abaixo👇 e aperte ENTER:
    ```
    node -v
    ```
    
    4. Deve aparecer a versão do node, talvez n seja igual como esta na imagem, mas n importa:
    ![node_version](./imgs/nodev.png)


1. Ainda no cmd, rode o comando abaixo👇
```
mkdir projeto && cd projeto
```

2. Inicie o projeto👇
```
npm init -y
```

3. Instale alguns pacotes👇 **[VAI DEMORAR UM POUCO]**
```
npm install express cors sqlite3 sqlite
```

5. Instale mais pacotes👇
```
npm install --save-dev typescript nodemon ts-node @types/express @types/cors
```

6. Rode o comando abaixo👇 para criar o arquivo de configuração
```
npx tsc --init
```

6. Rode o comando abaixo👇
```
mkdir src
```

7. Rode o comando abaixo👇 para abrir o vscode. **VAI DEMORAR UM POUCO**. *[Não feche o cmd ainda, vamos usa-lo mais tarde]*
```
code .
```

# HORA DE COMEÇARMOS O PROJETO
Se tudo tiver ocorrido bem, vc vai ver o vscode assim:
![vscode](./imgs/code.png)

1. Crie um arquivo dentro da pasta "src" e nomeie ele assim: `app.ts`👇
![app.ts](./imgs/appts.png)

2. Abra o arquivo **tsconfig.json**

3. *APAGUE TODO O CÓDIGO*

4. Depois de apagar, copie o código abaixo👇 e cole
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

5. Abra o arquivo "package.json"

6. Copie esse código: `"dev": "npx nodemon src/app.ts",`

7. Olhe pra essa imagem👇. E deixe o seu código igual a ela
![package.json](./imgs/package.png)

8. Crie um arquivo chamado `database.ts` dentro da pasta "src" 👇
![database-ts](./imgs/database-ts.png)

9. E cole esse código, dentro do `database.ts`:
```typescript
import { open } from 'sqlite';
import sqlite3 from 'sqlite3';

let instance: any | null = null;

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

10. Abra o `app.ts` que vc criou dentro da pasta "src" e cole o código abaixo👇
```typescript
import express from 'express';
import cors from 'cors';
import { connect } from './database';

const port = 1111;
const app = express();

app.use(cors());
app.use(express.json());
app.use(express.static(__dirname + '/../public'))

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

app.get('/users', async (req, res) => {
    const db = await connect();
    const users = await db.all('SELECT * FROM users');
  
    res.json(users);
});

app.put('/users/:id', async (req, res) => {
  const db = await connect();
  const { name, email } = req.body;
  const { id } = req.params;

  await db.run('UPDATE users SET name = ?, email = ? WHERE id = ?', [name, email, id]);
  const user = await db.get('SELECT * FROM users WHERE id = ?', [id]);

  res.json(user);
});

app.delete('/users/:id', async (req, res) => {
  const db = await connect();
  const { id } = req.params;

  await db.run('DELETE FROM users WHERE id = ?', [id]);

  res.json({ message: 'User deleted' });
});
```

# ESTÁ PERTO DO FIM...
1. Crie um arquivo dentro da pasta "src" chamado `test.http`

2. E instale uma extensão no seu vscode chamada "REST Client". Tutorial fodão abaixo👇
![extension](./imgs/extension.png)

---
# Agora vamos rodar e testar o nosso servidor

1. Naquele cmd que vc n fechou. Cole e rode esse comando:
```sh
npm run dev
```

> Se vc fechou, ainda há esperanças. Clique nessas teclas: **CTRL** + **"** . Ou pesquise como abrir o "terminal" dentro do vscode

2. Pesquise esse link: `http://localhost:1111/`

3. Vc vai ver uma tela branca e bem no cantinho superior duas palavras: "Hello Word"

4. Abra o `test.http` e cole esse código
```http
POST http://localhost:1111/users HTTP/1.1
content-type: application/json

{
  "name": "John Doe",
  "email": "johndoe@mail.com"
}

####

PUT http://localhost:1111/users/1 HTTP/1.1
content-type: application/json

{
  "name": "Thomas Turbando",
  "email": "thomas@mail.com"
}

####

DELETE http://localhost:1111/users/1 HTTP/1.1
```

7. Em cima do **POST**, **PUT** e **DELETE**, temos duas palavrinhas "Send Request". *(Elas só aparecem se vc tiver instalado aquela extensão que eu falei(REST Client). Se não instalou, INSTALE AGORA*

## Vamos testar agora
1. Clique no "Send Request" que está acima do `POST`

2. Pesquise esse link: `http://localhost:1111/users`. E verifique se aparece o nome: `Jonh Doe` e o email: `johndoe@mail.com`

4. Clique no "Send Request" que está acima do `PUT`

5. Atualize a página `localhost`. E verifique se aparece o nome: `Thomas Turbando` e o email: `thomas@mail.com`

7. Clique no ultimo "Send Request", que está acima do `DELETE`

8. Atualize a página `localhost`. E verifique se sumiu tudo

## Agora vamos adicionar botões, etc..
1. Crie uma pasta chamada: `public` [tem q ser dentro da pasta "projeto"]:

2. Crie um arquivo chamado: `index.html"` dentro da pasta public. Deve ficar assim 👇
![public](./imgs/public.png).

3. E cole esse código 👇
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>

<body>
  <form>
    <input type="text" required name="name" placeholder="Nome">
    <input type="email" required name="email" placeholder="Email">
    <button type="submit">Cadastrar</button>
  </form>
  <table>
    <thead>
      <tr>
        <th>Id</th>
        <th>Name</th>
        <th>Email</th>
        <th>Ações</th>
      </tr>
    </thead>
    <tbody>
      <!--  -->
    </tbody>
  </table>
  <script>
    const form = document.querySelector('form')
    form.addEventListener('submit', async (event) => {
      event.preventDefault()
      const name = form.name.value
      const email = form.email.value
      await fetch('/users', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ name, email })
      })
      form.reset()
      fetchData()
    })
    const tbody = document.querySelector('tbody')
    async function fetchData() {
      const resp = await fetch('/users')
      const data = await resp.json()
      tbody.innerHTML = ''
      data.forEach(user => {
        const tr = document.createElement('tr')
        tr.innerHTML = `
          <td>${user.id}</td>
          <td>${user.name}</td>
          <td>${user.email}</td>
          <td>
            <button class="excluir">excluir</button>
            <button class="editar">editar</button>
          </td>
        `
        const btExcluir = tr.querySelector('button.excluir')
        const btEditar = tr.querySelector('button.editar')
        btExcluir.addEventListener('click', async () => {
          await fetch(`/users/${user.id}`, { method: 'DELETE' })
          tr.remove()
        })
        btEditar.addEventListener('click', async () => {
          const name = prompt('Novo nome:', user.name)
          const email = prompt('Novo email:', user.email)
          await fetch(`/users/${user.id}`, {
            method: 'PUT',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ name, email })
          })
          fetchData()
        })
        tbody.appendChild(tr)
      })
    }
    fetchData()
  </script>
</body>
</html>
```
4. Salve o arquivo.

5. **E FINALMENTE** abra outra guia e cole esse link: `http://localhost:1111/`. E teste os botões, adicione pessoas e remova-as, teste tudo ai e parabéns😊👍

Um jogo daorinha pra jogar: (https://therace.montblancexplorer.com/)
