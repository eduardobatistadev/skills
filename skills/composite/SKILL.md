---
name: composite
description: Aplica o padrão Composite (árvore de objetos). Use quando o modelo do domínio é naturalmente uma árvore de elementos simples e contêineres (ex.: arquivos e pastas, produtos e caixas, componentes de UI aninhados, menus e submenus, organogramas, expressões) e o cliente deve tratar folhas e grupos de forma uniforme, normalmente com operações recursivas como somar preço ou renderizar. Também use para revisar hierarquias Composite existentes.
---

# Composite

**Categoria:** estrutural · **Também conhecido como:** Árvore de objetos, Object tree

## Propósito

Compor objetos em estruturas de árvore e trabalhar com essas estruturas como se fossem objetos individuais. Folhas e contêineres compartilham uma interface; o contêiner delega o trabalho aos filhos e agrega os resultados.

## Quando usar

- Você precisa implementar uma estrutura de objetos tipo árvore: folhas simples e contêineres que podem conter folhas ou outros contêineres.
- Você quer que o cliente trate objetos simples e compostos da mesma forma, sem se preocupar com a classe concreta.

**Teste rápido:** só faz sentido se o modelo central da aplicação puder ser representado como árvore. Se não puder, o padrão não se aplica.

## Quando evitar

- Folhas e contêineres têm comportamentos tão diferentes que uma interface comum ficaria genérica demais e difícil de entender.
- A estrutura é plana (uma lista simples).

## Como implementar

1. Confirme que o modelo pode ser uma árvore. Separe elementos simples e contêineres; contêineres devem aceitar ambos.
2. Declare a **interface componente** com métodos que façam sentido tanto para simples quanto para complexos.
3. Crie uma ou mais classes **folha** para os elementos simples.
4. Crie a classe **contêiner** com uma coleção de filhos tipada pela interface componente. Ao implementar os métodos, delegue a maior parte do trabalho aos filhos.
5. Defina métodos para adicionar e remover filhos no contêiner.

**Decisão de design:** colocar `adicionar`/`remover` na interface componente permite tratar tudo de forma idêntica, inclusive ao montar a árvore, mas viola o princípio de segregação de interface (as folhas ficam com métodos vazios). Deixá-los só no contêiner é mais seguro em tipos, mas o cliente precisa saber o que é contêiner. Escolha conscientemente.

## Exemplo mínimo

```ts
interface Item { preco(): number }

class Produto implements Item {
  constructor(private valor: number) {}
  preco() { return this.valor }
}

class Caixa implements Item {
  private filhos: Item[] = []
  adicionar(i: Item) { this.filhos.push(i); return this }
  remover(i: Item) { this.filhos = this.filhos.filter(f => f !== i) }
  preco() { return this.filhos.reduce((s, f) => s + f.preco(), 0) }   // recursão
}

const pedido = new Caixa()
  .adicionar(new Produto(50))
  .adicionar(new Caixa().adicionar(new Produto(10)).adicionar(new Produto(5)))
pedido.preco()   // 65, sem o cliente saber o que tem dentro
```

## Prós e contras

- ✅ Estruturas de árvore complexas ficam convenientes: polimorfismo e recursão trabalham a seu favor.
- ✅ Aberto/fechado: tipos novos de elemento entram sem quebrar o código que percorre a árvore.
- ⚠️ Difícil criar uma interface comum para classes muito diferentes. A interface pode ficar genérica demais.

## Checklist de revisão

- [ ] A coleção de filhos é tipada pela interface componente (aceita folhas e contêineres)?
- [ ] O contêiner delega aos filhos em vez de inspecionar tipos concretos?
- [ ] Há proteção contra ciclos (um contêiner contendo a si mesmo)?
- [ ] Há uma decisão explícita sobre onde ficam `adicionar`/`remover`?

## Padrões relacionados

- **Builder** constrói árvores Composite complexas de forma recursiva.
- **Chain of Responsibility**: uma folha pode passar um pedido pela cadeia de pais até a raiz.
- **Iterator** percorre a árvore; **Visitor** executa uma operação sobre a árvore toda.
- Folhas compartilhadas podem ser **Flyweights** para economizar RAM.
- **Decorator** tem estrutura parecida, mas com um único filho, e acrescenta responsabilidades em vez de agregar resultados. Os dois cooperam bem.
- **Prototype** ajuda a clonar árvores em vez de reconstruí-las.
