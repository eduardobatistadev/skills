---
name: abstract-factory
description: Aplica o padrão Abstract Factory (fábrica abstrata). Use quando o código precisa criar famílias de objetos relacionados que devem ser compatíveis entre si (ex.: widgets de Windows vs. Mac, temas, drivers por banco de dados) sem depender das classes concretas, quando a variante é escolhida por configuração ou ambiente, ou quando uma classe acumulou vários métodos fábrica que desviam sua responsabilidade. Também use para revisar ou refatorar fábricas abstratas existentes.
---

# Abstract Factory

**Categoria:** criacional · **Também conhecido como:** Fábrica abstrata

## Propósito

Produzir famílias de objetos relacionados sem especificar suas classes concretas. Uma interface de fábrica declara um método de criação por tipo de produto; cada fábrica concreta entrega uma variante coerente da família inteira.

## Quando usar

- O código trabalha com várias famílias de produtos relacionados (cadeira + sofá + mesa em estilos Moderno/Vitoriano; botão + checkbox para Windows/Mac) e você não quer depender das classes concretas, seja porque ainda são desconhecidas, seja para permitir crescer depois.
- É importante garantir que produtos de variantes diferentes nunca sejam misturados.
- Uma classe tem um punhado de métodos fábrica que ofuscam sua responsabilidade principal; vale extraí-los para uma fábrica dedicada.

**Sinais no código:** `if (os == "Windows") new WinButton() else new MacButton()` repetido para cada tipo de componente.

## Quando evitar

- Há só um tipo de produto (Factory Method basta) ou só uma variante.
- A família muda de forma (novos *tipos* de produto) com frequência: cada tipo novo obriga a alterar a interface e todas as fábricas.

## Como implementar

1. Monte uma matriz: tipos de produto (linhas) × variantes (colunas).
2. Declare uma interface abstrata para cada tipo de produto e faça todos os produtos concretos implementarem a interface do seu tipo.
3. Declare a interface da fábrica abstrata com um método de criação por produto abstrato.
4. Implemente uma fábrica concreta por variante.
5. Em um único ponto de inicialização, escolha a fábrica concreta conforme configuração/ambiente e injete-a em quem constrói produtos.
6. Substitua todas as chamadas diretas a construtores de produtos por chamadas à fábrica.

## Exemplo mínimo

```ts
interface Botao { render(): void }
interface Checkbox { render(): void }

interface FabricaGUI {
  criarBotao(): Botao
  criarCheckbox(): Checkbox
}
class FabricaWin implements FabricaGUI {
  criarBotao() { return new BotaoWin() }
  criarCheckbox() { return new CheckboxWin() }
}
class FabricaMac implements FabricaGUI {
  criarBotao() { return new BotaoMac() }
  criarCheckbox() { return new CheckboxMac() }
}

class Aplicacao {
  constructor(private fabrica: FabricaGUI) {}
  montarTela() { this.fabrica.criarBotao().render() }
}

// inicialização: o único lugar que conhece as classes concretas
const fabrica = config.os === "mac" ? new FabricaMac() : new FabricaWin()
new Aplicacao(fabrica).montarTela()
```

## Prós e contras

- ✅ Produtos vindos da mesma fábrica são compatíveis entre si.
- ✅ Cliente desacoplado dos produtos concretos.
- ✅ Responsabilidade única e aberto/fechado para novas variantes.
- ⚠️ Muitas interfaces e classes novas; o código pode ficar mais complexo do que o problema pede.

## Checklist de revisão

- [ ] Existe uma única decisão sobre qual fábrica concreta usar?
- [ ] O cliente depende só das interfaces de fábrica e de produto?
- [ ] Todas as fábricas concretas implementam todos os métodos (nenhuma variante "incompleta")?

## Padrões relacionados

- Frequentemente é a evolução de um **Factory Method**; quase sempre é implementada com métodos fábrica, mas pode compor **Prototypes**.
- **Builder** monta um objeto complexo passo a passo; Abstract Factory devolve o produto imediatamente.
- Pode substituir um **Facade** quando o objetivo é só esconder como os objetos do subsistema são criados.
- Combina com **Bridge** quando certas abstrações só funcionam com certas implementações: a fábrica encapsula essas combinações.
- Fábricas costumam ser implementadas como **Singleton**.
