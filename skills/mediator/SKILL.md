---
name: mediator
description: Aplica o padrão Mediator (mediador / controlador). Use quando vários objetos conversam entre si de forma caótica e muito acoplada (ex.: campos de um formulário ou diálogo que se habilitam/escondem uns aos outros, componentes de UI, torre de controle entre aviões, módulos que se chamam em teia), quando um componente não pode ser reutilizado por depender demais de outros, ou quando se criam subclasses só para mudar como componentes colaboram. Também use para revisar mediadores que viraram objetos deus.
---

# Mediator

**Categoria:** comportamental · **Também conhecido como:** Mediador, Intermediário, Controlador

## Propósito

Reduzir dependências caóticas entre objetos. O padrão proíbe a comunicação direta entre componentes e os obriga a colaborar só por meio de um objeto mediador. Cada componente conhece apenas o mediador; o mediador conhece os componentes e decide como reagir.

## Quando usar

- É difícil mudar algumas classes porque estão fortemente acopladas a várias outras. O mediador concentra as relações num só lugar e isola as mudanças.
- Você não consegue reutilizar um componente em outro programa porque ele depende demais de outros. Depois do Mediator, basta fornecer um mediador novo.
- Você está criando muitas subclasses de componentes só para reaproveitar comportamento básico em contextos diferentes. Com o mediador, novas formas de colaboração são só novos mediadores.

**Sinal no código:** um `Checkbox` que conhece o `TextField`, que conhece o `Botao`, que conhece o `Dialogo`...

## Quando evitar

- São só dois ou três objetos com interação simples e estável.
- O mediador começaria a absorver regras de negócio que pertencem aos próprios componentes.

## Como implementar

1. Identifique um grupo de classes fortemente acopladas que se beneficiariam de ser independentes (manutenção, reuso).
2. Declare a **interface do mediador** com o protocolo de comunicação. Normalmente basta um método `notificar(remetente, evento)`. Essa interface é essencial para reutilizar componentes com mediadores diferentes.
3. Implemente o mediador concreto, guardando referências a todos os componentes que gerencia.
4. Opcional: torne o mediador responsável por criar e destruir os componentes. Nesse ponto ele pode lembrar uma fábrica ou uma fachada.
5. Faça os componentes guardarem uma referência ao mediador (normalmente recebida no construtor).
6. Mude os componentes para chamarem `notificar` do mediador em vez de métodos de outros componentes. Mova para o mediador o código que chamava os outros componentes.

## Exemplo mínimo

```ts
interface Mediador { notificar(remetente: Componente, evento: string): void }

abstract class Componente {
  constructor(protected mediador: Mediador) {}
}
class Checkbox extends Componente {
  marcado = false
  clicar() { this.marcado = !this.marcado; this.mediador.notificar(this, "check") }
}
class CampoTexto extends Componente { visivel = false; valor = "" }
class Botao extends Componente {
  clicar() { this.mediador.notificar(this, "submit") }
}

class DialogoCadastro implements Mediador {
  souCachorro = new Checkbox(this)
  nomeDoCachorro = new CampoTexto(this)
  enviar = new Botao(this)

  notificar(remetente: Componente, evento: string) {
    if (remetente === this.souCachorro && evento === "check")
      this.nomeDoCachorro.visivel = this.souCachorro.marcado
    if (remetente === this.enviar && evento === "submit")
      this.validarEEnviar()
  }
  private validarEEnviar() { /* ... */ }
}
```

`Checkbox`, `CampoTexto` e `Botao` não se conhecem e podem ser reaproveitados em qualquer outro diálogo.

## Prós e contras

- ✅ Responsabilidade única: a comunicação entre componentes fica num só lugar.
- ✅ Aberto/fechado: novos mediadores sem mudar os componentes.
- ✅ Menos acoplamento entre componentes.
- ✅ Componentes individuais mais fáceis de reutilizar.
- ⚠️ Com o tempo o mediador pode virar um **objeto deus**.

## Checklist de revisão

- [ ] Componentes não referenciam outros componentes diretamente?
- [ ] Componentes dependem da interface do mediador, não da classe concreta?
- [ ] O mediador só coordena e deixa o comportamento de cada componente dentro dele?
- [ ] O mediador está crescendo demais? Divida por fluxo ou tela.

## Padrões relacionados

- **CoR**, **Command**, Mediator e **Observer** ligam remetentes e destinatários de formas diferentes; o Mediator elimina as ligações diretas.
- **Facade** também organiza classes acopladas, mas não adiciona funcionalidade e o subsistema nem sabe que ela existe; no Mediator, os componentes só conhecem o mediador.
- **Observer**: uma implementação popular de Mediator usa o mediador como publicador e os componentes como assinantes. Mas o Mediator também pode ligar componentes permanentemente, sem eventos. O foco do Mediator é eliminar dependências múltiplas; o do Observer é comunicação dinâmica de mão única.
