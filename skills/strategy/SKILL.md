---
name: strategy
description: Aplica o padrão Strategy (estratégia). Use quando existem várias variantes intercambiáveis de um algoritmo e se quer escolher ou trocar em tempo de execução (ex.: cálculo de frete, formas de pagamento, regras de desconto, rotas de navegação, algoritmos de ordenação, compressão, validação), quando uma classe tem um condicional grande que alterna entre variantes do mesmo comportamento, ou quando várias classes só diferem na forma de executar algo. Também use para revisar estratégias e decidir entre classes e funções.
---

# Strategy

**Categoria:** comportamental · **Também conhecido como:** Estratégia

## Propósito

Definir uma família de algoritmos, colocar cada um numa classe separada e tornar seus objetos intercambiáveis. A classe **contexto** guarda uma referência a uma estratégia e delega o trabalho a ela, sem saber qual é a concreta. O cliente escolhe a estratégia.

## Quando usar

- Você quer usar variantes diferentes de um algoritmo dentro de um objeto e **trocar de uma para outra em tempo de execução**.
- Você tem muitas classes parecidas que só diferem na forma como executam um comportamento. Extraia a variação para uma hierarquia de estratégias e una as classes originais numa só.
- Você quer isolar a lógica de negócio de detalhes de implementação de algoritmos (código, dados internos e dependências deles).
- A classe tem um **condicional enorme** que escolhe entre variantes do mesmo algoritmo.

## Quando evitar

- Há só um par de algoritmos que raramente muda. Classes e interfaces novas só complicam.
- A linguagem tem funções de primeira classe e as estratégias não têm estado: passe **funções/lambdas** em vez de criar classes. É o mesmo padrão, sem inchaço.

## Como implementar

1. No contexto, identifique um algoritmo sujeito a mudanças frequentes, ou um condicional enorme que escolhe uma variante em tempo de execução.
2. Declare a interface da estratégia, comum a todas as variantes.
3. Extraia, uma a uma, as variantes para classes próprias que implementam a interface.
4. No contexto, adicione um campo para a estratégia e um setter para trocá-la. O contexto fala com ela só pela interface. Se a estratégia precisar de dados do contexto, passe-os por parâmetro ou exponha uma interface para isso.
5. O cliente associa o contexto à estratégia adequada para o que espera dele.

## Exemplo mínimo

```ts
interface EstrategiaDeFrete { calcular(pedido: Pedido): number }

class FretePadrao implements EstrategiaDeFrete { calcular(p: Pedido) { return 10 + p.peso * 2 } }
class FreteExpresso implements EstrategiaDeFrete { calcular(p: Pedido) { return 25 + p.peso * 4 } }
class RetiradaNaLoja implements EstrategiaDeFrete { calcular() { return 0 } }

class Checkout {                                  // contexto
  constructor(private frete: EstrategiaDeFrete) {}
  setFrete(f: EstrategiaDeFrete) { this.frete = f }
  total(p: Pedido) { return p.subtotal + this.frete.calcular(p) }
}

const checkout = new Checkout(new FretePadrao())
checkout.setFrete(new FreteExpresso())            // troca em tempo de execução

// variante funcional, quando não há estado
type Frete = (p: Pedido) => number
const gratisAcimaDe200: Frete = p => (p.subtotal >= 200 ? 0 : 15)
```

Isso substitui:

```ts
if (tipo === "padrao") ... else if (tipo === "expresso") ... else if (tipo === "retirada") ...
```

## Prós e contras

- ✅ Troca de algoritmos em tempo de execução.
- ✅ Detalhes de implementação isolados do código que usa o algoritmo.
- ✅ Composição no lugar de herança.
- ✅ Aberto/fechado: novas estratégias sem mudar o contexto.
- ⚠️ Desnecessário com poucos algoritmos estáveis.
- ⚠️ O cliente precisa conhecer as diferenças entre as estratégias para escolher.
- ⚠️ Em linguagens com suporte funcional, classes podem ser substituídas por funções anônimas.

## Checklist de revisão

- [ ] O contexto depende só da interface da estratégia?
- [ ] O condicional de escolha ficou num único lugar (fábrica, configuração, mapa `tipo → estratégia`)?
- [ ] Estratégias são independentes entre si (se precisam trocar umas às outras, talvez seja State)?
- [ ] Funções simples resolveriam no lugar de classes?

## Padrões relacionados

- **Bridge**, **State**, Strategy (e de certa forma **Adapter**) têm estrutura parecida, mas intenções diferentes.
- **Command** também parametriza um objeto com uma ação, mas transforma qualquer operação em objeto (fila, histórico, remoto); Strategy descreve maneiras diferentes de fazer *a mesma coisa*.
- **Decorator** muda a pele do objeto; Strategy muda as entranhas.
- **Template Method** usa herança e é estático (nível de classe); Strategy usa composição e é dinâmico (nível de objeto).
- **State** é uma extensão do Strategy em que os estados se conhecem e trocam o estado do contexto.
