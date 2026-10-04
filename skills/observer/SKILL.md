---
name: observer
description: Aplica o padrão Observer (observador / publicador-assinante / listener / eventos). Use quando mudanças no estado de um objeto precisam notificar outros objetos cujo conjunto é desconhecido de antemão ou muda em tempo de execução (ex.: eventos de UI, notificações de estoque ou pedido, atualização de views a partir de um modelo, webhooks internos, sistemas de plugins), ou quando objetos devem observar outros só por um período. Também use para revisar sistemas de eventos, vazamentos de assinantes e ordem de notificação.
---

# Observer

**Categoria:** comportamental · **Também conhecido como:** Observador, Assinante do evento, Event-Subscriber, Listener

## Propósito

Definir um mecanismo de assinatura para notificar vários objetos sobre eventos que acontecem com o objeto que eles observam. O **publicador** mantém uma lista de **assinantes** e os avisa pela interface comum; os assinantes entram e saem da lista quando quiserem.

## Quando usar

- Mudanças no estado de um objeto podem exigir mudanças em outros objetos, e esse conjunto de objetos é desconhecido de antemão ou muda dinamicamente. Exemplo: botões customizados de uma biblioteca de UI onde o cliente pluga seu próprio código no clique.
- Alguns objetos precisam observar outros só por um tempo limitado ou em casos específicos.

**Sinal no código:** o objeto "dono" do evento chama diretamente uma lista crescente de classes interessadas (`email.enviar(); estoque.atualizar(); analytics.registrar()`), ou clientes fazem polling repetido para saber se algo mudou.

## Quando evitar

- Há um único interessado fixo. Uma chamada direta é mais clara.
- A ordem de reação é crítica e precisa ser garantida. Observers não garantem ordem; use uma sequência explícita ou Chain of Responsibility.
- Cadeias de notificação em cascata dificultariam rastrear o fluxo.

## Como implementar

1. Divida a lógica de negócio em duas partes: a funcionalidade central, independente, vira o **publicador**; o resto vira um conjunto de **assinantes**.
2. Declare a interface do assinante, com no mínimo um método `atualizar`.
3. Declare a interface do publicador com métodos para inscrever e desinscrever assinantes. O publicador só conhece assinantes por essa interface.
4. Decida onde ficam a lista e os métodos de inscrição. Como o código é igual para todos os publicadores, uma classe abstrata base é natural. Se você está aplicando o padrão a uma hierarquia existente, prefira **composição**: um objeto gerenciador de eventos que os publicadores usam.
5. Crie os publicadores concretos e notifique os assinantes sempre que algo importante acontecer.
6. Implemente `atualizar` nos assinantes concretos. Dados do evento podem ir como argumentos, ou o publicador pode passar a si mesmo para o assinante buscar o que precisar. Ligar o publicador ao assinante permanentemente pelo construtor é a opção menos flexível.
7. O cliente cria os assinantes e os registra nos publicadores apropriados.

## Exemplo mínimo

```ts
type Ouvinte<E> = (evento: E) => void

class GerenciadorDeEventos<E> {               // composição, reutilizável
  private ouvintes = new Map<string, Set<Ouvinte<E>>>()
  inscrever(tipo: string, o: Ouvinte<E>) {
    if (!this.ouvintes.has(tipo)) this.ouvintes.set(tipo, new Set())
    this.ouvintes.get(tipo)!.add(o)
    return () => this.ouvintes.get(tipo)!.delete(o)   // função para cancelar
  }
  notificar(tipo: string, e: E) { this.ouvintes.get(tipo)?.forEach(o => o(e)) }
}

class Pedido {                                   // publicador
  eventos = new GerenciadorDeEventos<{ id: string }>()
  pagar(id: string) { /* ... */ this.eventos.notificar("pago", { id }) }
}

const pedido = new Pedido()
const cancelar = pedido.eventos.inscrever("pago", e => enviarEmail(e.id))
pedido.eventos.inscrever("pago", e => baixarEstoque(e.id))
// ...mais tarde
cancelar()
```

## Prós e contras

- ✅ Aberto/fechado: novos assinantes sem mudar o publicador (e vice-versa, se houver interface de publicador).
- ✅ Relações entre objetos estabelecidas em tempo de execução.
- ⚠️ Assinantes são notificados em ordem não garantida.

## Checklist de revisão

- [ ] Todo `inscrever` tem um `desinscrever` correspondente (evita vazamento de memória e assinantes "zumbis")?
- [ ] Nenhum assinante depende da ordem de notificação?
- [ ] Uma exceção num assinante não impede os demais de serem notificados?
- [ ] Notificações em cascata (assinante que dispara outro evento) estão sob controle?
- [ ] Notificações assíncronas, se usadas, tratam erros e reentrância?

## Padrões relacionados

- **CoR**, **Command**, **Mediator** e Observer ligam remetentes e destinatários de formas diferentes; o Observer permite inscrição e cancelamento dinâmicos.
- **Mediator** costuma ser implementado com Observer (o mediador publica, os componentes assinam). Sem um mediador central, vira um conjunto distribuído de observers.
