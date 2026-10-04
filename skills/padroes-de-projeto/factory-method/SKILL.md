---
name: factory-method
description: Aplica o padrão Factory Method (método fábrica / construtor virtual). Use quando o código instancia classes concretas com `new` espalhado e precisa suportar novos tipos de produto sem mexer no cliente, quando um framework ou biblioteca deve permitir que usuários troquem componentes internos por subclasses, ou quando a criação deve poder reaproveitar objetos existentes (pool, cache). Também use para revisar ou refatorar implementações existentes de Factory Method.
---

# Factory Method

**Categoria:** criacional · **Também conhecido como:** Método fábrica, Construtor virtual

## Propósito

Definir, numa classe criadora, um método responsável por criar objetos, e deixar que subclasses decidam qual classe concreta será instanciada. O código cliente passa a depender só da interface comum dos produtos, nunca das classes concretas.

## Quando usar

- Você não sabe de antemão todos os tipos (e dependências) dos objetos com os quais o código vai trabalhar, e novos tipos devem ser fáceis de adicionar.
- Você escreve uma biblioteca ou framework e quer que quem o usa possa substituir um componente interno (ex.: um `Botão`) por uma subclasse própria, sobrescrevendo apenas o método que o cria.
- Você quer economizar recursos reaproveitando objetos caros (conexões, arquivos, sockets). Um construtor sempre devolve um objeto novo; um método fábrica pode devolver um existente.

**Sinais no código:** `switch`/`if` escolhendo qual classe instanciar repetidos em vários lugares; classes de negócio acopladas a uma classe concreta (`Caminhão`) que agora precisa conviver com outra (`Navio`).

## Quando evitar

- Só existe um tipo de produto e não há perspectiva real de variação: o padrão só adiciona subclasses.
- A variação é de *família* de produtos relacionados (use Abstract Factory) ou de *montagem passo a passo* (use Builder).

## Como implementar

1. Faça todos os produtos implementarem a mesma interface, com métodos que façam sentido para todos eles.
2. Adicione à classe criadora um método fábrica cujo tipo de retorno seja a interface do produto.
3. Troque, uma a uma, as chamadas diretas a construtores dentro da criadora por chamadas ao método fábrica. Se preciso, use temporariamente um parâmetro para escolher o tipo (o método pode ficar feio com um `switch` por enquanto).
4. Crie uma subclasse criadora por tipo de produto e sobrescreva o método fábrica, movendo para ela o trecho de construção correspondente.
5. Se houver tipos demais para uma subclasse cada, reaproveite o parâmetro de controle nas subclasses em vez de multiplicar classes.
6. Se o método base ficou vazio, torne-o abstrato; se sobrou algo, mantenha como comportamento padrão.

## Exemplo mínimo

```ts
interface Transporte { entregar(carga: string): void }
class Caminhao implements Transporte { entregar(c: string) { /* por estrada */ } }
class Navio implements Transporte { entregar(c: string) { /* por mar */ } }

abstract class Logistica {
  protected abstract criarTransporte(): Transporte   // método fábrica
  planejarEntrega(carga: string) {                   // lógica de negócio não muda
    const t = this.criarTransporte()
    t.entregar(carga)
  }
}
class LogisticaTerrestre extends Logistica { protected criarTransporte() { return new Caminhao() } }
class LogisticaMaritima extends Logistica { protected criarTransporte() { return new Navio() } }
```

Lembre: a criadora normalmente tem lógica de negócio própria; criar produtos não é sua responsabilidade principal.

## Prós e contras

- ✅ Remove o acoplamento entre a criadora e os produtos concretos.
- ✅ Responsabilidade única: a criação fica concentrada em um lugar.
- ✅ Aberto/fechado: novos produtos entram sem quebrar o cliente.
- ⚠️ Pode exigir muitas subclasses. Funciona melhor quando já existe uma hierarquia de criadoras.

## Checklist de revisão

- [ ] O tipo de retorno do método fábrica é a interface do produto, não uma classe concreta?
- [ ] O cliente usa os produtos apenas via interface?
- [ ] Não sobraram `new ProdutoConcreto()` fora das fábricas?
- [ ] O número de subclasses criadoras se justifica (ou um parâmetro resolveria)?

## Padrões relacionados

- Costuma ser o ponto de partida; projetos evoluem para **Abstract Factory**, **Prototype** ou **Builder** quando precisam de mais flexibilidade.
- **Abstract Factory** geralmente é um conjunto de métodos fábrica.
- É uma especialização do **Template Method** e pode ser uma etapa dele.
- Combina com **Iterator**: subclasses de coleção devolvem iteradores compatíveis.
- **Prototype** evita herança, mas exige inicializar o clone; Factory Method usa herança e dispensa essa etapa.
