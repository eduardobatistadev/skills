---
name: visitor
description: Aplica o padrão Visitor (visitante / double dispatch). Use quando for preciso executar operações sobre todos os elementos de uma estrutura de objetos de classes diferentes (ex.: nós de AST em compiladores e linters, exportar um grafo para XML/JSON, calcular impostos sobre itens heterogêneos, relatórios sobre árvores Composite) sem poluir essas classes com comportamento auxiliar, ou quando uma operação faz sentido só para algumas classes da hierarquia. Também use para revisar visitantes e o trade-off de adicionar novos tipos de elemento.
---

# Visitor

**Categoria:** comportamental · **Também conhecido como:** Visitante

## Propósito

Separar algoritmos dos objetos sobre os quais operam. A operação vai para uma classe visitante com um método por tipo de elemento; cada elemento só tem um método `aceitar(visitante)` que chama o método certo do visitante. Essa troca de chamadas é o **double dispatch**: o tipo do elemento e o do visitante decidem juntos qual código executa, sem condicionais por tipo.

## Quando usar

- Você precisa executar uma operação sobre todos os elementos de uma estrutura de objetos complexa (por exemplo, uma árvore), e os elementos são de classes diferentes.
- Você quer **limpar a lógica de negócio** de comportamentos auxiliares (exportação, relatórios, serialização), deixando as classes principais focadas no seu trabalho.
- Um comportamento faz sentido só para algumas classes de uma hierarquia: implemente só os métodos visitantes relevantes e deixe os outros vazios.

**Por que não sobrecarga de método?** A sobrecarga é resolvida em tempo de compilação pelo tipo declarado da variável. Se você tem uma `Forma` que na verdade é um `Circulo`, `exportar(forma)` chama a versão de `Forma`. O `aceitar` resolve isso porque cada classe concreta sabe seu próprio tipo.

## Quando evitar

- A hierarquia de elementos muda com frequência (classes entram e saem): cada mudança obriga a atualizar **todos** os visitantes.
- As operações precisam de muito acesso a estado privado dos elementos.
- Em linguagens com *pattern matching* exaustivo sobre tipos selados/uniões discriminadas, um `match` pode ser uma alternativa mais simples.

**Trade-off central:** Visitor facilita adicionar **operações** e dificulta adicionar **tipos**. Polimorfismo comum faz o contrário.

## Como implementar

1. Declare a interface do visitante com um método "visitar" para cada classe concreta de elemento.
2. Declare a interface do elemento (ou adicione à base de uma hierarquia existente) com o método `aceitar(visitante)`.
3. Implemente `aceitar` em todas as classes concretas de elemento, apenas redirecionando para o método visitante correspondente à própria classe.
4. Elementos conhecem visitantes só pela interface. Visitantes precisam conhecer todas as classes concretas de elemento (são os tipos dos parâmetros).
5. Para cada comportamento que não deve ficar na hierarquia de elementos, crie um visitante concreto e implemente todos os métodos. Se ele precisar de membros privados, você pode torná-los públicos (violando encapsulamento) ou aninhar o visitante no elemento, se a linguagem permitir.
6. O cliente cria os visitantes e os passa aos elementos via `aceitar`.

## Exemplo mínimo

```ts
interface Visitante {
  visitarCirculo(c: Circulo): void
  visitarRetangulo(r: Retangulo): void
  visitarGrupo(g: Grupo): void
}

interface Forma { aceitar(v: Visitante): void }

class Circulo implements Forma {
  constructor(public x: number, public y: number, public raio: number) {}
  aceitar(v: Visitante) { v.visitarCirculo(this) }
}
class Retangulo implements Forma {
  constructor(public x: number, public y: number, public w: number, public h: number) {}
  aceitar(v: Visitante) { v.visitarRetangulo(this) }
}
class Grupo implements Forma {
  constructor(public filhos: Forma[]) {}
  aceitar(v: Visitante) { v.visitarGrupo(this) }
}

class ExportadorXML implements Visitante {     // operação nova, sem tocar nas formas
  saida: string[] = []
  visitarCirculo(c: Circulo) { this.saida.push(`<circulo r="${c.raio}"/>`) }
  visitarRetangulo(r: Retangulo) { this.saida.push(`<ret w="${r.w}" h="${r.h}"/>`) }
  visitarGrupo(g: Grupo) {
    this.saida.push("<grupo>"); g.filhos.forEach(f => f.aceitar(this)); this.saida.push("</grupo>")
  }
}

const xml = new ExportadorXML()
desenho.forEach(f => f.aceitar(xml))
```

## Prós e contras

- ✅ Aberto/fechado: comportamento novo para várias classes sem mudar essas classes.
- ✅ Responsabilidade única: as versões de um mesmo comportamento ficam juntas numa classe.
- ✅ O visitante pode acumular informação enquanto percorre a estrutura (útil em árvores).
- ⚠️ Todos os visitantes precisam ser atualizados quando uma classe entra ou sai da hierarquia.
- ⚠️ Visitantes podem não ter acesso aos campos e métodos privados de que precisariam.

## Checklist de revisão

- [ ] Todo elemento concreto implementa `aceitar` chamando o método visitante do seu próprio tipo?
- [ ] A hierarquia de elementos é estável o bastante para justificar o padrão?
- [ ] A travessia (quem percorre os filhos) está definida em um lugar só: no visitante, no elemento ou num iterador?
- [ ] O encapsulamento foi relaxado só no mínimo necessário?

## Padrões relacionados

- Pode ser visto como uma versão poderosa do **Command**, que opera sobre objetos de várias classes.
- Executa operações sobre uma árvore **Composite** inteira.
- Com **Iterator**, percorre estruturas complexas e opera sobre elementos de classes diferentes.
