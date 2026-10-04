---
name: builder
description: Aplica o padrão Builder (construtor passo a passo). Use quando um objeto tem muitos parâmetros opcionais ou um "construtor telescópico", quando o mesmo processo de construção deve gerar representações diferentes (ex.: carro e manual do carro, casa de pedra e de madeira), ou quando é preciso montar estruturas complexas como árvores Composite. Também use para revisar ou refatorar builders e classes Director existentes.
---

# Builder

**Categoria:** criacional · **Também conhecido como:** Construtor

## Propósito

Construir objetos complexos passo a passo, separando o código de construção do próprio objeto. O mesmo processo pode produzir tipos e representações diferentes do produto.

## Quando usar

- Para eliminar o **construtor telescópico**: um construtor com dez parâmetros opcionais, ou várias sobrecargas que só repassam valores padrão. Com o Builder você chama só as etapas que precisa.
- Quando várias representações do produto seguem etapas parecidas que diferem só nos detalhes. A interface do builder define as etapas; cada builder concreto as implementa à sua maneira; um diretor pode fixar a ordem.
- Para montar **árvores Composite** ou outros objetos complexos: etapas podem ser adiadas ou chamadas recursivamente.
- Quando o cliente nunca deve ver um produto pela metade: o builder só entrega o resultado no final.

## Quando evitar

- O objeto é simples, com poucos campos obrigatórios. Um construtor (ou parâmetros nomeados, ou um objeto de opções) resolve.
- Não há etapas comuns claras entre as representações: sem isso o padrão não se aplica.

## Como implementar

1. Confirme que é possível definir etapas de construção comuns a todas as representações. Sem isso, pare.
2. Declare essas etapas na interface base do builder.
3. Crie um builder concreto por representação e implemente as etapas.
4. Implemente um método para obter o resultado. Ele só pode ir para a interface base se todos os produtos compartilharem uma interface; caso contrário fica em cada builder concreto.
5. Considere uma classe **diretor** que encapsule receitas de construção (ordem das etapas) reutilizáveis com qualquer builder.
6. O cliente cria o builder e o diretor, passa o builder ao diretor (no construtor do diretor ou no método de construção) e, ao final, pega o resultado do builder (ou do diretor, se todos os produtos tiverem a mesma interface).

## Exemplo mínimo

```ts
interface CarroBuilder {
  reset(): void
  setBancos(n: number): void
  setMotor(tipo: string): void
  setGPS(tem: boolean): void
}

class CarroConcretoBuilder implements CarroBuilder {
  private carro = new Carro()
  reset() { this.carro = new Carro() }
  setBancos(n: number) { this.carro.bancos = n }
  setMotor(t: string) { this.carro.motor = t }
  setGPS(g: boolean) { this.carro.gps = g }
  getResultado(): Carro { const c = this.carro; this.reset(); return c }
}
// ManualBuilder implementa as mesmas etapas, mas escreve um Manual.

class Diretor {
  construirEsportivo(b: CarroBuilder) {
    b.reset(); b.setBancos(2); b.setMotor("V8"); b.setGPS(true)
  }
}

const b = new CarroConcretoBuilder()
new Diretor().construirEsportivo(b)
const carro = b.getResultado()
```

Em linguagens com API fluente é comum cada etapa retornar `this` (`builder.motor("V8").bancos(2).build()`), o que funciona bem quando não há diretor.

## Prós e contras

- ✅ Construção passo a passo, com etapas adiáveis ou recursivas.
- ✅ Reuso do mesmo código de construção para representações diferentes.
- ✅ Responsabilidade única: a montagem complexa sai da lógica de negócio do produto.
- ⚠️ Aumenta o número de classes e a complexidade geral.

## Checklist de revisão

- [ ] O builder impede que um produto incompleto seja obtido?
- [ ] O builder é reiniciado (ou descartado) depois de entregar o resultado?
- [ ] O diretor, se existir, depende só da interface do builder?
- [ ] Os campos obrigatórios são validados antes de entregar o produto?

## Padrões relacionados

- Evolução comum de **Factory Method**.
- **Abstract Factory** entrega famílias de produtos de uma vez; Builder monta um produto em etapas.
- Ótimo para construir árvores **Composite** recursivamente.
- Combina com **Bridge**: o diretor faz o papel de abstração e os builders, de implementações.
- Pode ser implementado como **Singleton**.
