---
name: iterator
description: Aplica o padrão Iterator (iterador). Use quando for preciso percorrer os elementos de uma coleção sem expor sua estrutura interna (lista, pilha, árvore, grafo, resultado paginado de API), oferecer várias formas de travessia (profundidade, largura, filtrada), eliminar código de travessia duplicado, ou escrever código que funcione com coleções de tipos desconhecidos de antemão. Também use para revisar iteradores, geradores e coleções customizadas.
---

# Iterator

**Categoria:** comportamental · **Também conhecido como:** Iterador

## Propósito

Percorrer os elementos de uma coleção sem expor sua representação (lista, pilha, árvore...). O algoritmo de travessia sai da coleção e vai para um objeto iterador, que guarda seu próprio estado (posição atual, quanto falta).

## Quando usar

- A coleção tem uma estrutura de dados complexa por baixo e você quer escondê-la dos clientes, por conveniência ou por segurança (o cliente não consegue fazer nada indevido com a estrutura).
- Você quer reduzir a **duplicação de código de travessia**. Algoritmos de iteração não triviais, misturados à lógica de negócio, desfocam a responsabilidade do código.
- Você quer que o código percorra estruturas diferentes, ou tipos desconhecidos de antemão, por meio de interfaces genéricas de coleção e iterador.

## Quando evitar

- A aplicação só trabalha com coleções simples e a linguagem já oferece iteração nativa. Criar um iterador próprio vira preciosismo.
- Desempenho é crítico e percorrer a coleção especializada diretamente é bem mais rápido.

**Na prática:** a maioria das linguagens já tem o padrão embutido (`Iterable`/`Iterator` em Java, `__iter__`/geradores em Python, `Symbol.iterator`/`function*` em JS/TS, `IEnumerable` em C#). Implemente o protocolo da linguagem em vez de inventar uma interface nova.

## Como implementar

1. Declare a interface do iterador. No mínimo um método para obter o próximo elemento; por conveniência, outros como "tem próximo", posição atual ou elemento anterior.
2. Declare a interface da coleção com um método que devolve iteradores (tipo de retorno: a interface do iterador). Se houver grupos distintos de iteradores, declare métodos parecidos para cada um.
3. Implemente iteradores concretos para as coleções. Cada iterador fica ligado a **uma única instância** de coleção, normalmente pelo construtor.
4. Implemente a interface de coleção nas suas classes: a coleção passa a si mesma ao construtor do iterador.
5. No cliente, substitua todo código de travessia por iteradores. Peça um iterador novo a cada travessia.

## Exemplo mínimo

```ts
class Arvore<T> implements Iterable<T> {
  constructor(public valor: T, public filhos: Arvore<T>[] = []) {}

  // iterador padrão (profundidade) usando o protocolo da linguagem
  *[Symbol.iterator](): Iterator<T> {
    yield this.valor
    for (const f of this.filhos) yield* f
  }

  // travessia alternativa
  *emLargura(): Generator<T> {
    const fila: Arvore<T>[] = [this]
    while (fila.length) {
      const n = fila.shift()!
      yield n.valor
      fila.push(...n.filhos)
    }
  }
}

for (const v of arvore) { /* cliente não sabe como a árvore é armazenada */ }
for (const v of arvore.emLargura()) { /* outra ordem, mesma coleção */ }
```

## Prós e contras

- ✅ Responsabilidade única: algoritmos de travessia pesados saem do cliente e da coleção.
- ✅ Aberto/fechado: coleções e iteradores novos funcionam com o código existente.
- ✅ Várias travessias em paralelo sobre a mesma coleção, já que cada iterador tem seu estado.
- ✅ É possível pausar uma iteração e retomá-la depois.
- ⚠️ Preciosismo para coleções simples.
- ⚠️ Pode ser menos eficiente que acessar diretamente coleções especializadas.

## Checklist de revisão

- [ ] Usa o protocolo de iteração nativo da linguagem?
- [ ] O cliente não depende da estrutura interna da coleção?
- [ ] O comportamento ao modificar a coleção durante a iteração está definido (falhar rápido, snapshot...)?
- [ ] Coleções grandes ou remotas são iteradas de forma preguiçosa (sem carregar tudo)?

## Padrões relacionados

- Percorre árvores **Composite**.
- Com **Factory Method**, subclasses de coleção devolvem iteradores compatíveis.
- Com **Memento**, captura e restaura o estado de uma iteração.
- Com **Visitor**, percorre uma estrutura complexa e executa operações sobre elementos de classes diferentes.
