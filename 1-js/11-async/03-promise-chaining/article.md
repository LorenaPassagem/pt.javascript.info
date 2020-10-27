
# Encadeamento de promessas

<<<<<<< HEAD
Vamos retornar ao problema mencionado no capítulo <info:callbacks>: temos uma sequência de tarefas assíncronas a serem executadas uma após a outra — por exemplo, o carregamento de scripts. Como podemos codificar isso bem?
=======
Let's return to the problem mentioned in the chapter <info:callbacks>: we have a sequence of asynchronous tasks to be performed one after another — for instance, loading scripts. How can we code it well?
>>>>>>> e074a5f825a3d10b0c1e5e82561162f75516d7e3

Promessas proveem algumas receitas para isso.

Neste capítulo, vamos cobrir o encadeamento de promessas.

Ele se parece com isto:

```js run
new Promise(function(resolve, reject) {

  setTimeout(() => resolve(1), 1000); // (*)

}).then(function(result) { // (**)

  alert(result); // 1
  return result * 2;

}).then(function(result) { // (***)

  alert(result); // 2
  return result * 2;

}).then(function(result) {

  alert(result); // 4
  return result * 2;

});
```

A ideia é que o resultado seja passado através de uma cadeia de tratadores `.then`.

O fluxo é o seguinte:
1. A promessa inicial é resolvida em 1 segundo `(*)`,
2. Então o tratador de `.then` é chamado `(**)`.
3. O valor retornado por ele é passado ao próximo tratador de `.then` `(***)`
4. ...e assim por diante.

Como o resultado é passado através da cadeia de tratadores, podemos observar a sequência de chamadas `alert`: `1` -> `2` -> `4`.

![](promise-then-chain.svg)

A coisa toda funciona pois a chamada ao `promise.then` retorna uma promessa, assim podemos chamar o próximo `.then` nesse retorno.

Quando um tratador retorna um valor, ele se torna o resultado da promessa, então o próximo `.then` é chamado com ele. 

**Um erro clássico de novatos: tecnicamente, podemos chamar vários `.then` em uma única promessa. Isso não é encadeamento.**

Por exemplo:
```js run
let promise = new Promise(function(resolve, reject) {
  setTimeout(() => resolve(1), 1000);
});

promise.then(function(result) {
  alert(result); // 1
  return result * 2;
});

promise.then(function(result) {
  alert(result); // 1
  return result * 2;
});

promise.then(function(result) {
  alert(result); // 1
  return result * 2;
});
```

<<<<<<< HEAD
O que fizemos aqui é apenas utilizar uma série de tratadores em uma promessa. Eles não passam o resultado uns aos outros; pelo contrário, eles o processam de maneira independente.
=======
What we did here is just several handlers to one promise. They don't pass the result to each other; instead they process it independently.
>>>>>>> e074a5f825a3d10b0c1e5e82561162f75516d7e3

Aqui está uma imagem (compare-a com a cadeia acima):

![](promise-then-many.svg)

Todos os `.then` na mesma promessa obtêm o mesmo resultado -- o resultado dessa promessa. Então no código acima todas as chamadas `alert` mostrarão o mesmo: `1`.

Na prática, raramente precisamos de múltiplos tratadores para uma promessa. O encadeamento é utilizado com mais frequência.

## Retornando promessas

Um tratador "handler", utilizado em `.then(handler)` pode criar e retornar uma promessa.

<<<<<<< HEAD
Nessa caso, tratadores seguintes aguardarão até que essa seja estabelecida, e então pegarão o seu resultado.
=======
In that case further handlers wait until it settles, and then get its result.
>>>>>>> e074a5f825a3d10b0c1e5e82561162f75516d7e3

Por exemplo:

```js run
new Promise(function(resolve, reject) {

  setTimeout(() => resolve(1), 1000);

}).then(function(result) {

  alert(result); // 1

*!*
  return new Promise((resolve, reject) => { // (*)
    setTimeout(() => resolve(result * 2), 1000);
  });
*/!*

}).then(function(result) { // (**)

  alert(result); // 2

  return new Promise((resolve, reject) => {
    setTimeout(() => resolve(result * 2), 1000);
  });

}).then(function(result) {

  alert(result); // 4

});
```
 
Aqui o primeiro `.then` exibe `1` e retorna `new Promise(…)` na linha `(*)`. Após 1 segundo ele é resolvido, e o resultado (o argumento de `resolve`, no caso é `result * 2`) é passado ao tratador do segundo `.then`. O tratador está na linha `(**)`, ele exibe `2` e faz a mesma coisa.

Então a saída é a mesma que no exemplo anterior: 1 -> 2 -> 4, mas agora com 1 segundo de atraso entre as chamadas `alert`.

Retornar promessas nos permite construir um encadeamento de ações assíncronas.

## Exemplo: loadScript

Vamos utilizar esta funcionalidade com o `loadScript` "promisified", definido em [previous chapter](info:promise-basics#loadscript), para carregar scripts um a um, em sequência:

```js run
loadScript("/article/promise-chaining/one.js")
  .then(function(script) {
    return loadScript("/article/promise-chaining/two.js");
  })
  .then(function(script) {
    return loadScript("/article/promise-chaining/three.js");
  })
  .then(function(script) {
    // usando funções declaradas nos scripts
    // para mostrar que de fato foram carregados
    one();
    two();
    three();
  });
```

Esse código pode ser resumido utilizando arrow functions: 

```js run
loadScript("/article/promise-chaining/one.js")
  .then(script => loadScript("/article/promise-chaining/two.js"))
  .then(script => loadScript("/article/promise-chaining/three.js"))
  .then(script => {
    // scripts já carregados, podemos utilizar funções declaradas neles
    one();
    two();
    // three()
  });
```


Aqui cada chamada `loadScript` retorna uma promessa, e o próximo `.then` é executado quando ela é resolvida. Então, é iniciado o carregamento do próximo script. Assim, os scripts são carregados um após o outro.

<<<<<<< HEAD
Podemos adicionar mais ações assíncronas à cadeia. Por favor note que o código ainda está "flat" — ele cresce para baixo, não para direita. Não há sinais da "pyramid of doom". 
=======
We can add more asynchronous actions to the chain. Please note that the code is still "flat" — it grows down, not to the right. There are no signs of the "pyramid of doom".
>>>>>>> e074a5f825a3d10b0c1e5e82561162f75516d7e3

Tecnicamente, poderíamos chamar `.then` diretamente em cada `loadScript`, desta maneira:

```js run
loadScript("/article/promise-chaining/one.js").then(script1 => {
  loadScript("/article/promise-chaining/two.js").then(script2 => {
    loadScript("/article/promise-chaining/three.js").then(script3 => {
      // esta função tem acesso às variáveis script1, script2 e script3
      one();
      two();
      three();
    });
  });
});
```

Esse código faz o mesmo: carrega 3 scripts em sequência. Porém ele "cresce para direita". Então temos o mesmo problema das callbacks.

Pessoas que iniciam a usar promessas às vezes não conhecem o encadeamento, então elas escrevem código dessa maneira. Geralmente, o encadeamento é preferido.

Às vezes, é ok escrever `.then` diretamente, pois funções aninhadas possuem acesso ao escopo exterior. No exemplo acima, a callback mais aninhada possui acesso a todas as variáveis `script1`, `script2`, `script3`. Mas isso é uma exceção ao invés de ser uma regra.


````smart header="Thenables"
<<<<<<< HEAD
Para ser preciso, um tratador pode não retornar exatamente uma promessa, mas um objeto conhecido por "thenable" - um objeto arbitrário que possui um método `.then`. Ele vai ser tratado da mesma maneira que uma promessa.

A ideia é que bibliotecas de terceiros possam implementar seus próprios objetos "compatíveis com promessas". Eles podem possuir um conjunto maior de métodos, mas também ser compatíveis com promessas nativas, pois eles implementam `.then`.
=======
To be precise, a handler may return not exactly a promise, but a so-called "thenable" object - an arbitrary object that has a method `.then`. It will be treated the same way as a promise.

The idea is that 3rd-party libraries may implement "promise-compatible" objects of their own. They can have an extended set of methods, but also be compatible with native promises, because they implement `.then`.
>>>>>>> e074a5f825a3d10b0c1e5e82561162f75516d7e3

Aqui está um exemplo de um objeto "thenable":

```js run
class Thenable {
  constructor(num) {
    this.num = num;
  }
  then(resolve, reject) {
    alert(resolve); // function() { native code }
    // resolve com this.num*2 depois de 1 segundo
    setTimeout(() => resolve(this.num * 2), 1000); // (**)
  }
}

new Promise(resolve => resolve(1))
  .then(result => {
*!*
    return new Thenable(result); // (*)
*/!*
  })
  .then(alert); // exibe 2 depois de 1000ms
```

<<<<<<< HEAD
JavaScript verifica o objeto retornado pelo tratador de `.then` na linha `(*)`: se ele possuir um método executável que possa ser chamado e possua nome `then`, então ele chama o método provendo funções nativas `resolve`, `reject` como argumentos (similar a um executor) e aguarda até que uma delas seja chamada. No exemplo acima, `resolve(2)` é chamada depois de 1 segundo `(**)`. Então o resultado é passado adiante pela cadeia.

Essa funcionalidade nos permite integrar objetos customizáveis com cadeias de promessas sem termos que herdar de `Promise`. 
=======
JavaScript checks the object returned by the `.then` handler in line `(*)`: if it has a callable method named `then`, then it calls that method providing native functions `resolve`, `reject` as arguments (similar to an executor) and waits until one of them is called. In the example above `resolve(2)` is called after 1 second `(**)`. Then the result is passed further down the chain.

This feature allows us to integrate custom objects with promise chains without having to inherit from `Promise`.
>>>>>>> e074a5f825a3d10b0c1e5e82561162f75516d7e3
````


## Maior exemplo: fetch

Em programação frontend, promessas são utilizadas para requisições de rede com frequência. Então vamos ver um exemplo estendido disso.

Vamos usar o método [fetch](info:fetch) para carregar de um servidor remoto informações sobre o usuário. Ele possui vários parâmetros opcionais abordados em [separate chapters](info:fetch), mas a sintaxe básica é bem simples:

```js
let promise = fetch(url);
```

Isso faz uma requisição de rede para a `url` e retorna uma promessa. A promessa é resolvida com um objeto `response` quando o servidor remoto responder com os cabeçalhos, mas *antes do download completo da resposta*.  

<<<<<<< HEAD
Para ler a resposta completa, devemos chamar o método `response.text()`: isso retorna uma promessa que é resolvida quando o texto completo é baixado do servidor remoto, com esse texto como resultado.
=======
To read the full response, we should call the method `response.text()`: it returns a promise that resolves when the full text is downloaded from the remote server, with that text as a result.
>>>>>>> e074a5f825a3d10b0c1e5e82561162f75516d7e3

O código abaixo faz uma requisição a `user.json` e carrega seu texto do servidor:  

```js run
fetch('/article/promise-chaining/user.json')
  // .then abaixo executa quando o servidor remoto responder
  .then(function(response) {
    // response.text() retorna uma nova promessa que é resolvida com o texto completo da resposta
    // quando ela é baixada por completo
    return response.text();
  })
  .then(function(text) {
<<<<<<< HEAD
    // ...e aqui está o conteúdo do arquivo remoto
    alert(text); // {"name": "iliakan", isAdmin: true}
  });
```

O objeto `response` retornado pelo `fetch` também inclui o método `response.json()` que lê os dados remotos e faz o parser deles como JSON. No nosso caso, isso é ainda mais conveniente, então vamos trocar para isso.
=======
    // ...and here's the content of the remote file
    alert(text); // {"name": "iliakan", "isAdmin": true}
  });
```

The `response` object returned from `fetch` also includes the method `response.json()` that reads the remote data and parses it as JSON. In our case that's even more convenient, so let's switch to it.
>>>>>>> e074a5f825a3d10b0c1e5e82561162f75516d7e3

Também vamos utilizar arrow functions para brevidade:

```js run
// mesmo que acima, mas response.json() faz o parser do conteúdo remoto como JSON
fetch('/article/promise-chaining/user.json')
  .then(response => response.json())
  .then(user => alert(user.name)); // iliakan, obtém o nome do usuário
```

Agora, vamos fazer algo com o usuário carregado.

<<<<<<< HEAD
Por exemplo, podemos fazer mais uma requisição ao GitHub, carregar o perfil do usuário e exibir o seu avatar. 
=======
For instance, we can make one more requests to GitHub, load the user profile and show the avatar:
>>>>>>> e074a5f825a3d10b0c1e5e82561162f75516d7e3

```js run
// Fazer uma requisição para user.json
fetch('/article/promise-chaining/user.json')
  // Carregar como json
  .then(response => response.json())
  // Fazer uma requisição ao GitHub
  .then(user => fetch(`https://api.github.com/users/${user.name}`))
  // Carregar a resposta como json
  .then(response => response.json())
  // Exibir a imagem do avatar (githubUser.avatar_url) por 3 segundos (talvez animá-la)
  .then(githubUser => {
    let img = document.createElement('img');
    img.src = githubUser.avatar_url;
    img.className = "promise-avatar-example";
    document.body.append(img);

    setTimeout(() => img.remove(), 3000); // (*)
  });
```

<<<<<<< HEAD
O código funciona; veja os comentários para os detalhes. Porém, há um problema em potencial nele, um erro comum para aqueles que começam a usar promessas.
=======
The code works; see comments about the details. However, there's a potential problem in it, a typical error for those who begin to use promises.
>>>>>>> e074a5f825a3d10b0c1e5e82561162f75516d7e3

Observe a linha `(*)`: como podemos fazer algo *após* o avatar acabar de ser exibido e ser removido? Por exemplo, queremos exibir um formulário para edição do usuário ou outra coisa. Do jeito que está, não há como.

Para tornar a cadeia extensível, precisamos retornar uma promessa que seja resolvida quando o avatar acabar de ser exibido.

Desta forma:

```js run
fetch('/article/promise-chaining/user.json')
  .then(response => response.json())
  .then(user => fetch(`https://api.github.com/users/${user.name}`))
  .then(response => response.json())
*!*
  .then(githubUser => new Promise(function(resolve, reject) { // (*)
*/!*
    let img = document.createElement('img');
    img.src = githubUser.avatar_url;
    img.className = "promise-avatar-example";
    document.body.append(img);

    setTimeout(() => {
      img.remove();
*!*
      resolve(githubUser); // (**)
*/!*
    }, 3000);
  }))
  // disparado após 3 segundoss
  .then(githubUser => alert(`Finalizou exibição de ${githubUser.name}`));
```

<<<<<<< HEAD
Isto é, o tratador do `.then` na linha `(*)` agora retorna `new Promise`, que se torna estabelecida apenas após a chamada de `resolve(githubUser)` em `setTimeout` `(**)`. O próximo `.then` na cadeia vai aguardar por isso.

Como uma boa prática, uma ação assíncrona deve sempre retornar uma promessa. Isso torna possível planejar ações após ela; mesmo que não planejemos estender a cadeia agora, podemos precisar disso depois.
=======
That is, the `.then` handler in line `(*)` now returns `new Promise`, that becomes settled only after the call of `resolve(githubUser)` in `setTimeout` `(**)`. The next `.then` in the chain will wait for that.

As a good practice, an asynchronous action should always return a promise. That makes it possible to plan actions after it; even if we don't plan to extend the chain now, we may need it later.
>>>>>>> e074a5f825a3d10b0c1e5e82561162f75516d7e3

Finalmente, podemos dividir o código em funções reusáveis:

```js run
function loadJson(url) {
  return fetch(url)
    .then(response => response.json());
}

function loadGithubUser(name) {
  return fetch(`https://api.github.com/users/${name}`)
    .then(response => response.json());
}

function showAvatar(githubUser) {
  return new Promise(function(resolve, reject) {
    let img = document.createElement('img');
    img.src = githubUser.avatar_url;
    img.className = "promise-avatar-example";
    document.body.append(img);

    setTimeout(() => {
      img.remove();
      resolve(githubUser);
    }, 3000);
  });
}

// Utilizando as funções:
loadJson('/article/promise-chaining/user.json')
  .then(user => loadGithubUser(user.name))
  .then(showAvatar)
  .then(githubUser => alert(`Finalizou exibição ${githubUser.name}`));
  // ...
```

## Resumo

Se um tratador `.then` (ou `catch/finally`, não importa) retornar uma promessa, o resto da cadeia aguarda até ela ser estabelecida. Quando isso acontece, o resultado (ou erro) é passado adiante.

Aqui está a ideia geral:

![](promise-handler-variants.svg)
