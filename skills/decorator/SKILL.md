---
name: decorator
description: Aplica o padrão Decorator (decorador / envoltório / wrapper). Use quando for preciso adicionar ou combinar comportamentos a objetos em tempo de execução sem alterar o código que os usa (ex.: compressão + criptografia em streams, notificações por e-mail + SMS + Slack, logging, cache, retry, middlewares), quando a herança causaria explosão de subclasses para cada combinação, ou quando a classe é final. Também use para revisar pilhas de decoradores.
---

# Decorator

**Categoria:** estrutural · **Também conhecido como:** Decorador, Envoltório, Wrapper

## Propósito

Acoplar novos comportamentos a um objeto colocando-o dentro de um invólucro que implementa a mesma interface. O decorador repassa as chamadas ao objeto envolvido e faz algo antes ou depois. Como a interface é a mesma, decoradores podem ser empilhados.

## Quando usar

- Você precisa atribuir comportamentos extras a objetos **em tempo de execução**, sem quebrar o código que os usa. A lógica fica em camadas: um decorador por camada, combinados como for preciso.
- Estender por herança é complicado ou impossível: a classe é `final`, ou cada combinação de recursos exigiria uma subclasse (`NotificadorEmailSMS`, `NotificadorEmailSlack`, ...).

**Herança vs. composição:** herança é estática e uma classe só tem um pai. Com composição/agregação, o objeto delega e pode trocar o ajudante em tempo de execução. O Decorator é esse princípio aplicado.

## Quando evitar

- Só há uma variação fixa, conhecida em tempo de compilação.
- O comportamento depende muito da ordem dos invólucros e isso vai confundir quem usa.
- O que você quer é mudar a lógica *interna* do objeto, não envolvê-la: isso é Strategy.

## Como implementar

1. Confirme que o domínio pode ser visto como um componente principal com várias camadas opcionais por cima.
2. Identifique os métodos comuns ao componente e às camadas e declare-os numa **interface componente**.
3. Crie o **componente concreto** com o comportamento base.
4. Crie um **decorador base** com um campo do tipo da interface componente (o objeto envolvido) que delega tudo a ele.
5. Garanta que todas as classes implementem a interface componente.
6. Crie **decoradores concretos** estendendo o base; cada um faz seu trabalho antes ou depois de chamar o método pai (que delega).
7. O cliente monta a pilha de decoradores na ordem de que precisa.

## Exemplo mínimo

```ts
interface Fonte { escrever(dados: string): void; ler(): string }

class ArquivoFonte implements Fonte { /* lê/escreve em disco */
  escrever(d: string) {} ; ler() { return "" } }

class DecoradorFonte implements Fonte {
  constructor(protected envolvido: Fonte) {}
  escrever(d: string) { this.envolvido.escrever(d) }
  ler() { return this.envolvido.ler() }
}
class Criptografia extends DecoradorFonte {
  escrever(d: string) { super.escrever(cifrar(d)) }
  ler() { return decifrar(super.ler()) }
}
class Compressao extends DecoradorFonte {
  escrever(d: string) { super.escrever(comprimir(d)) }
  ler() { return descomprimir(super.ler()) }
}

// o cliente decide a composição: comprime, depois criptografa
const fonte: Fonte = new Criptografia(new Compressao(new ArquivoFonte()))
fonte.escrever("dados sensíveis")
```

## Prós e contras

- ✅ Estende comportamento sem criar subclasses.
- ✅ Adiciona e remove responsabilidades em tempo de execução.
- ✅ Combina vários comportamentos empilhando decoradores.
- ✅ Responsabilidade única: divide uma classe monolítica com muitas variantes em classes menores.
- ⚠️ Difícil remover um invólucro do meio da pilha.
- ⚠️ Difícil fazer decoradores cujo comportamento não dependa da ordem.
- ⚠️ O código de configuração das camadas pode ficar feio (encapsule-o numa fábrica ou builder).

## Checklist de revisão

- [ ] Todo decorador implementa a mesma interface do componente e sempre delega ao envolvido?
- [ ] Nenhum decorador interrompe o fluxo (isso seria Chain of Responsibility)?
- [ ] A dependência de ordem está documentada ou eliminada?
- [ ] A montagem da pilha está centralizada?

## Padrões relacionados

- **Adapter** muda a interface; **Decorator** mantém ou amplia e permite composição recursiva; **Proxy** mantém a interface.
- **Chain of Responsibility** tem estrutura parecida, mas seus handlers podem parar o pedido. Decoradores não podem quebrar o fluxo.
- **Composite** também usa composição recursiva, porém com vários filhos e agregando resultados. Decorator pode estender um nó específico de uma árvore Composite.
- **Prototype** ajuda a clonar estruturas decoradas.
- **Decorator** muda a "pele" do objeto; **Strategy** muda as "entranhas".
- **Proxy** costuma gerenciar o ciclo de vida do serviço sozinho; a composição de decoradores é sempre controlada pelo cliente.
