## Rotas

Todos os arquivos dentro de pages ou src/pages, tornam-se uma nova rota na aplicação, com excessão do arquivo _app e document.

## Estilos
Para usar um css global, utilizar apenas .css

Para usar um css que respeite a um escopo, utilizar o module.css

Nos arquivos css que sejam module.css, não podemos estilizar diretamente um elemento html. Exemplo:
- Caso queira estilizar um elemento h1, precisa colocar uma classe no elemento h1 e estilizar a classe.

Além disso, é necessário no arquivo que deseja implementar o module.css, importar com um nome. Por exemplo: import styles from "../styles/home.module.css"

Digamos que você criou uma estilização para a classe .title dentro do arquivo home.module.css. Para dar essa classe ao elemento que deseja estilizar, se utiliza no formato javascript e não como string. Por exemplo : '<h1 className={styles.title}>meu h1 estilizado</h1>'

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
