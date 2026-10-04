---
name: prototype
description: Aplica o padrão Prototype (protótipo / clone). Use quando for preciso copiar objetos sem depender de suas classes concretas (ex.: objetos recebidos de terceiros por interface), quando existem muitas subclasses que só diferem na configuração inicial, quando criar um objeto do zero é caro, ou quando se quer um registro de objetos pré-configurados para clonar. Também use para revisar implementações de clone (cópia rasa vs. profunda, referências circulares).
---

# Prototype

**Categoria:** criacional · **Também conhecido como:** Protótipo, Clone

## Propósito

Copiar objetos existentes sem que o código fique dependente de suas classes. O próprio objeto sabe se clonar, por meio de um método `clonar()` declarado numa interface comum.

## Quando usar

- O código não deve depender das classes concretas dos objetos que precisa copiar, por exemplo quando eles chegam de código de terceiros apenas por uma interface.
- Existem muitas subclasses que só diferem na forma como inicializam o objeto. Em vez delas, mantenha um conjunto de objetos pré-configurados (protótipos) e clone o que precisar.
- Recriar um objeto do zero é caro (consultas, cálculos, montagem de estruturas grandes como árvores Composite/Decorator).

**Por que não copiar "por fora"?** Copiar campo a campo de fora exige conhecer a classe concreta e não alcança campos privados. Por isso a cópia é delegada ao próprio objeto.

## Quando evitar

- Os objetos são simples e baratos de construir.
- O grafo de objetos tem referências circulares ou recursos externos (conexões, arquivos) difíceis de duplicar corretamente.

## Como implementar

1. Crie uma interface de protótipo com o método `clonar()` (ou adicione-o à hierarquia existente).
2. Em cada classe protótipo, defina um construtor alternativo que recebe um objeto da mesma classe e copia todos os seus campos. Em subclasses, chame o construtor da classe pai para que ela copie seus campos privados. Sem sobrecarga de construtor, use um método especial de cópia.
3. Implemente `clonar()`, normalmente uma linha: `return new MinhaClasse(this)`. **Toda subclasse precisa sobrescrever** `clonar()`, senão o clone sai com o tipo da superclasse.
4. Opcional: crie um **registro de protótipos** (classe fábrica ou método estático) que guarda protótipos frequentes, busca por um critério (nome, parâmetros) e devolve um clone. Depois substitua chamadas diretas aos construtores por chamadas ao registro.

## Exemplo mínimo

```ts
abstract class Forma {
  x = 0; y = 0; cor = "preto"
  constructor(origem?: Forma) {
    if (origem) { this.x = origem.x; this.y = origem.y; this.cor = origem.cor }
  }
  abstract clonar(): Forma
}

class Circulo extends Forma {
  raio = 1
  constructor(origem?: Circulo) {
    super(origem)
    if (origem) this.raio = origem.raio
  }
  clonar() { return new Circulo(this) }
}

// registro de protótipos
const registro = new Map<string, Forma>()
const grande = new Circulo(); grande.raio = 100; grande.cor = "azul"
registro.set("circulo-azul-grande", grande)
const copia = registro.get("circulo-azul-grande")!.clonar()
```

Decida conscientemente entre **cópia rasa** (compartilha objetos internos) e **cópia profunda** (duplica a estrutura). Documente a escolha.

## Prós e contras

- ✅ Clona sem acoplar às classes concretas.
- ✅ Elimina código de inicialização repetido.
- ✅ Facilita produzir objetos complexos.
- ✅ Alternativa à herança para configurações predefinidas.
- ⚠️ Clonar objetos com referências circulares pode ser bem complicado.

## Checklist de revisão

- [ ] Toda subclasse sobrescreve `clonar()` retornando o próprio tipo?
- [ ] A cópia de campos mutáveis compartilhados é intencional (rasa) ou foi esquecida?
- [ ] Recursos externos (handles, conexões) são tratados no clone?
- [ ] Referências circulares estão cobertas?

## Padrões relacionados

- Evolução comum de **Factory Method**; não usa herança, mas exige inicializar o clone.
- Métodos de uma **Abstract Factory** podem ser compostos com protótipos.
- Útil para guardar cópias de **Commands** no histórico.
- Projetos com muito **Composite** e **Decorator** se beneficiam: clonar estruturas em vez de reconstruí-las.
- Às vezes é uma alternativa mais simples ao **Memento**, se o estado for simples e sem vínculos externos.
- Pode ser implementado como **Singleton** (o registro, por exemplo).
