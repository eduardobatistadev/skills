---
name: flyweight
description: Aplica o padrão Flyweight (peso mosca / cache de estado compartilhado). Use quando o programa precisa manter um número enorme de objetos parecidos que esgotam a memória (ex.: partículas em jogos, árvores numa floresta, caracteres num editor, marcadores num mapa, células de planilha) e esses objetos repetem dados que podem ser compartilhados. Também use para revisar a separação entre estado intrínseco e extrínseco e o uso de fábricas de flyweights.
---

# Flyweight

**Categoria:** estrutural · **Também conhecido como:** Peso mosca, Cache

## Propósito

Caber mais objetos na RAM disponível compartilhando entre eles as partes comuns do estado, em vez de cada objeto guardar todos os seus dados.

- **Estado intrínseco:** dados imutáveis e repetidos em muitos objetos (textura, cor, sprite). Fica dentro do flyweight e é compartilhado.
- **Estado extrínseco:** dados únicos de cada objeto e dependentes de contexto (posição, velocidade). Sai do flyweight e é passado como parâmetro ou guardado num objeto de contexto.

## Quando usar

Use **apenas** quando o programa precisa suportar uma quantidade enorme de objetos que mal cabem na memória. O ganho aparece quando:

- a aplicação gera muitos objetos semelhantes,
- isso drena a RAM do dispositivo alvo,
- e os objetos contêm estado duplicado que pode ser extraído e compartilhado.

## Quando evitar

- Não há problema medido de memória. É otimização prematura com custo alto de legibilidade.
- O estado "compartilhável" precisa mudar por objeto: flyweights têm de ser imutáveis.

## Como implementar

1. Divida os campos da classe em **intrínsecos** (imutáveis, duplicados) e **extrínsecos** (contextuais, únicos).
2. Mantenha os intrínsecos na classe e torne-os imutáveis: valores definidos só no construtor.
3. Nos métodos que usam campos extrínsecos, troque cada campo por um parâmetro.
4. Opcional, mas comum: crie uma **fábrica de flyweights** que verifica se já existe um com o estado intrínseco pedido antes de criar outro. Os clientes passam a pedir flyweights só a ela.
5. O cliente guarda ou calcula o estado extrínseco para chamar os métodos do flyweight. Por conveniência, mova o estado extrínseco e a referência ao flyweight para uma **classe de contexto**.

## Exemplo mínimo

```ts
class TipoDeArvore {                         // flyweight: intrínseco e imutável
  constructor(readonly nome: string, readonly cor: string, readonly textura: Bitmap) {}
  desenhar(tela: Canvas, x: number, y: number) { /* usa extrínseco por parâmetro */ }
}

class FabricaDeTipos {
  private static cache = new Map<string, TipoDeArvore>()
  static obter(nome: string, cor: string, textura: Bitmap) {
    const chave = `${nome}|${cor}`
    if (!this.cache.has(chave)) this.cache.set(chave, new TipoDeArvore(nome, cor, textura))
    return this.cache.get(chave)!
  }
}

class Arvore {                               // contexto: extrínseco + referência
  constructor(private x: number, private y: number, private tipo: TipoDeArvore) {}
  desenhar(tela: Canvas) { this.tipo.desenhar(tela, this.x, this.y) }
}

// um milhão de árvores, poucos TipoDeArvore em memória
floresta.push(new Arvore(x, y, FabricaDeTipos.obter("pinheiro", "verde", texturaPinheiro)))
```

## Prós e contras

- ✅ Economiza muita RAM quando há muitos objetos similares.
- ⚠️ Pode trocar RAM por CPU se dados de contexto precisarem ser recalculados a cada chamada.
- ⚠️ O código fica bem mais complicado, e novos membros do time vão estranhar por que o estado foi separado assim. Documente o motivo.

## Checklist de revisão

- [ ] O ganho de memória foi medido ou estimado com números reais?
- [ ] O estado intrínseco é realmente imutável (sem setters, campos `readonly`)?
- [ ] Todos os clientes obtêm flyweights pela fábrica?
- [ ] A fábrica é segura em ambiente concorrente, se aplicável?

## Padrões relacionados

- Folhas compartilhadas de uma árvore **Composite** podem ser flyweights.
- **Facade** faz um objeto representar um subsistema inteiro; Flyweight faz muitos objetos pequenos.
- Seria parecido com **Singleton** se todo o estado compartilhado coubesse num objeto, mas flyweights podem ter várias instâncias (com estados intrínsecos diferentes) e são imutáveis.
