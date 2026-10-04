---
name: state
description: Aplica o padrão State (estado / máquina de estados orientada a objetos). Use quando um objeto se comporta de maneira diferente conforme seu estado atual e o código está cheio de if/switch sobre um campo de status (ex.: pedido rascunho → pago → enviado → entregue, documento rascunho/moderação/publicado, player tocando/pausado/bloqueado, conexão TCP, fluxo de aprovação), quando há muitos estados ou as regras por estado mudam com frequência. Também use para revisar máquinas de estado e transições.
---

# State

**Categoria:** comportamental · **Também conhecido como:** Estado

## Propósito

Permitir que um objeto altere seu comportamento quando seu estado interno muda, como se mudasse de classe. Cada estado vira uma classe; o objeto **contexto** guarda uma referência ao estado atual e delega a ele o comportamento dependente de estado.

## Quando usar

- Um objeto se comporta de forma diferente conforme o estado atual, o número de estados é grande e o código específico de cada estado muda com frequência.
- Uma classe está poluída por **condicionais gigantes** que mudam o comportamento conforme os valores dos campos. O padrão leva cada ramo para o método da classe de estado correspondente e tira da classe principal campos temporários e métodos auxiliares.
- Há muito código duplicado entre estados parecidos e transições baseadas em condições. Hierarquias de estados com classes base eliminam a duplicação.

**Sinal no código:** `switch (this.status)` repetido em vários métodos (`publicar()`, `cancelar()`, `editar()`), cada um com os mesmos casos.

## Quando evitar

- A máquina tem poucos estados ou muda raramente. Um `switch` bem organizado (ou um enum com tabela de transições) é mais simples.

## Como implementar

1. Escolha a classe **contexto**: uma classe existente com código dependente de estado, ou uma nova se esse código estiver espalhado.
2. Declare a **interface do estado**. Ela espelha métodos do contexto, mas inclua só os que têm comportamento específico de estado.
3. Para cada estado real, crie uma classe que implementa a interface e mova para ela todo o código do contexto relativo àquele estado. Se esse código depender de membros privados do contexto, você pode: torná-los públicos; expor um método público no contexto e chamá-lo do estado (rápido, mas feio, ajuste depois); ou aninhar as classes de estado no contexto, se a linguagem permitir.
4. No contexto, adicione um campo do tipo da interface do estado e um setter público para trocá-lo.
5. Nos métodos do contexto, troque as condicionais por chamadas ao objeto de estado.
6. Para mudar de estado, crie uma instância do novo estado e passe ao contexto. Isso pode ser feito pelo próprio contexto, pelos estados ou pelo cliente. Quem instanciar passa a depender daquela classe concreta.

## Exemplo mínimo

```ts
interface EstadoDoPedido {
  pagar(p: Pedido): void
  enviar(p: Pedido): void
  cancelar(p: Pedido): void
}

class Pedido {                                       // contexto
  constructor(private estado: EstadoDoPedido = new AguardandoPagamento()) {}
  mudarPara(e: EstadoDoPedido) { this.estado = e }
  pagar() { this.estado.pagar(this) }
  enviar() { this.estado.enviar(this) }
  cancelar() { this.estado.cancelar(this) }
}

class AguardandoPagamento implements EstadoDoPedido {
  pagar(p: Pedido) { /* cobra */ p.mudarPara(new Pago()) }
  enviar() { throw new Error("Pedido ainda não foi pago") }
  cancelar(p: Pedido) { p.mudarPara(new Cancelado()) }
}
class Pago implements EstadoDoPedido {
  pagar() { throw new Error("Já pago") }
  enviar(p: Pedido) { /* despacha */ p.mudarPara(new Enviado()) }
  cancelar(p: Pedido) { /* estorna */ p.mudarPara(new Cancelado()) }
}
// Enviado, Cancelado...
```

Estados sem campos próprios podem ser compartilhados (instâncias únicas) para evitar alocações.

## Prós e contras

- ✅ Responsabilidade única: código de cada estado em sua própria classe.
- ✅ Aberto/fechado: novos estados sem mudar os existentes nem o contexto.
- ✅ Contexto mais simples, sem condicionais pesadas de máquina de estados.
- ⚠️ Exagero se houver poucos estados ou se eles raramente mudarem.

## Checklist de revisão

- [ ] O contexto não tem mais `switch`/`if` sobre o estado?
- [ ] Transições inválidas são tratadas explicitamente (erro, no-op documentado)?
- [ ] Está claro quem é responsável pelas transições (estados ou contexto)?
- [ ] Existe um diagrama ou tabela das transições para documentar a máquina?

## Padrões relacionados

- **Bridge**, State, **Strategy** (e de certa forma **Adapter**) têm estrutura parecida, mas resolvem problemas diferentes.
- State pode ser visto como uma extensão do **Strategy**: estratégias são independentes e não se conhecem; estados podem conhecer uns aos outros e trocar o estado do contexto.
