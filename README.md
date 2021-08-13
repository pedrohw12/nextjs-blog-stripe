# Conhecimentos necessários do Nextjs

## Rotas

Todos os arquivos dentro de pages ou src/pages, tornam-se uma nova rota na aplicação, com excessão do arquivo _app e document.

## Estilos
Para usar um css global, utilizar apenas .css

Para usar um css que respeite a um escopo, utilizar o module.css

Nos arquivos css que sejam module.css, não podemos estilizar diretamente um elemento html. Exemplo:
- Caso queira estilizar um elemento h1, precisa colocar uma classe no elemento h1 e estilizar a classe.

Além disso, é necessário no arquivo que deseja implementar o module.css, importar com um nome. Por exemplo: import styles from "../styles/home.module.css"

Digamos que você criou uma estilização para a classe .title dentro do arquivo home.module.css. Para dar essa classe ao elemento que deseja estilizar, se utiliza no formato javascript e não como string. Por exemplo : `<h1 className={styles.title}>meu h1 estilizado</h1>`

## Arquivo _app
Existe um <Component /> neste arquivo e o que ele faz é, em cada página acessada, o componente <Component /> é substituído pelo componente da página acessada.

Então toda vez que uma nova rota for acessada, esta funcão do arquivo _app será executada novamente, montando novamente o componente contido nela.

## Arquivo _document
Este arquivo funciona de forma semelhante ao arquivo _app, mas ele é carregado apenas uma única vez. É como se fosse o arquivo index.html do create-react app

## getServerSideProps
Quando o request a api é feito no lado do servidor e a tela é montada já com todos os dados

## getStaticProps
É a basicamente mesma coisa do getServerSideProps, mas ao fazer o request, ele salva os dados vindos deste request para que quando
esta mesma tela for acessada novamente, não precise fazer um novo request a api. 

Muito útil quando tem uma tela que faz um request a api e esta tela é acessada por inúmeros clientes. Para evitar inúmeros requests
que retorna uma informação que pouco muda, usa-se o getStaticProps.

Ele retorna o revalidate, que é o tempo em segundos que quer que a tela atualize os dados no próximo request.

Por exemplo:
return {
  props: {},
  revalidate: 3000 -> se dois clientes acessam essa tela dentro do intervalo de 3s, eles geraram o mesmo html daquela página.
  caso o terceiro cliente acesse a tela após os 3s, ele irá disparar um novo request que gera novamente um html da página.
}

## Api routes
Todos os arquivos que forem criados dentro de pages/api, serão uma nova rota de api do next. As funções criadas nas api routes são serverless, ou seja, rodam e depois morrem. O servidor não fica de pé.

Obs.: Pastas dentro de api que contenham um underline na frente, não são tratadas como uma rotas. Por exemplo - _lib

Por exemplo, se for criado um arquivo em pages/api/users.ts, que retorna uma lista de usuários, 
ao acessar localhost:3000/api/users, estaremos acessando a rota criada, e vendo o retorno desta lista usuários.

Exemplo da rota users (pages/api/users.ts):

import { NextApiRequest, NextApiResponse } from "next";

export defaul (request: NextApiRequest, response: NextApiResponse) => {
  const users = [
    {id: 1, name: "Pedro"},
    {id: 2, name: "Wanderley"},
  ]

  return response.json(users);
}

### Rota com parametro
Quando for necessário criar uma rota com parâmetro (/users/1, por exemplo), basta criar um arquivo neste formato:
pages
  api
    users
      index.ts - a rota criada fica sendo /api/users
      [id].ts  - a rota criada fica sendo /api/users/1

Para ter acesso ao parâmetro enviado na rota:

import { NextApiRequest, NextApiResponse } from "next";

export defaul (request: NextApiRequest, response: NextApiResponse) => {

  const userId = request.query.id ///// Valor do parametro enviado
                                  ///// id é o nome que foi passado na criação do arquivo ([id].ts)

  const users = [
    {id: 1, name: "Pedro"},
    {id: 2, name: "Wanderley"},
  ]

  return response.json(users);
}

Caso for criado um arquivo assim:
[...params].ts, no lugar de [id.ts], isso significa que todo parâmetro que for enviado na rota vai ficar dentro de request.query.params.

## Estratégias de autenticação

### Estratégia 1 - JWT (storage)
Gerar um JWT e salvar este.

### Estratégia 2 - Next Auth
É utilizado quando se quer um sistema de autenticação simples, quando se precisa de um login social (por meio de github, google e etc) e quando não quer se preocupar com armazenar credenciais de acesso do usuário dentro do backend.

### Estratégia 3 - Serviço externos (providers de autenticação)
Cognito, Auth0

## Autenticando com Next Auth e Github
### 1 passo
#### Criar a pasta /pages/api/auth/[...nextauth].ts
Criar esta pasta e instalar o next-auth
yarn add next-auth

### 2 passo
#### Criar uma aplicação no Github
É necessário criar uma aplicação no Github.
Obs.: Criar uma aplicação para ambiente dev e outra para prod.

1 - Vá em settings
2 - Developer settings
3 - OAuth Apps
4 - New OAuth App
5 - Em authorization callback URL:
http://localhost:3000/api/auth/callback

api/auth, é porque esta são as pastas criadas no projeto e /callback
é o nome da rota. Ele vai criar um parâmetro dentro do [...nextauth]

6 - Em Homepage URl:
http://localhost:3000/home

7 - Copiar o github client_id e criar uma chave no .env GITHUB_CLIENT_ID e o GITHUB_CLIENT_SECRET (gerar o client secret)

8 - Após criada as chaves, colocar no arquivo [...nextauth]

9 - Criar uma nova propriedade nas configurações de autenticação com o github - scope

O scope se refere a quais informações quer ter acesso do usuário

Adicionar - scope: 'read:user' 

### 3 Passo

#### Fazer a função de login
Ir no local onde tem a ação de fazer login (geralmente é no botão login, que fica na tela de login), e importar a função signIn do next-auth/client

import {signIn} from "next-auth/client";

A ação do clique no botão para fazer login, será apenas isso:
onClick={() => signIn('github')}

O parâmetro da função é para dizer qual o provider, porque poderia ser google, facebook e etc. Neste caso é github.

### 4 passo

#### Usar o useSession para verificar se tem uma sessão ativa
Ao usar este hook, podemos fazer as verificações de se o usuário esta logado ou não, e com base nisso, tomar ações na aplicação (enviar para as páginas internas e etc).

Fazendo o login, pode pegar o nome do usuário em session.user.name;

import {useSession} from "next-auth/client";

const [session] = useSession();

if (session) {
  redirect ...
}

### 5 passo (último passo)

#### Englobar toda a aplicação com o provider do next-auth
Com a mesma lógica do provider da context api, vamos englovar todo o app com o provider do next-auth, para que todo o app saiba as informações de autenticação

import {Provider} from "next-auth/client"

function MyApp({ Component, pageProps }: AppProps) {
  return (
    <Provider session={pageProps.session}>
      <Header />
      <Component {...pageProps} />
    </Provider>
  );
}

Passando o session do pageProps, significa que ao dar um reload na página, as informações da sessão ativa (se está logado ou não) irão chegar através do pageProps.

Para fazer logout:
import {signOut} from "next-auth/client"

onClick={() => signOut()}

Essas informações de sessão do usuário são salvas nos cookies do navegador.

## Stripe
Depois de toda a configuração dos webhooks feita, rodar o comando stripe listen --forward-to localhost:3000/api/webhooks para escuta-los.
