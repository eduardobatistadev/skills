---
name: bridge
description: Aplica o padrão Bridge (ponte). Use quando uma classe ou hierarquia cresce em duas ou mais dimensões independentes e começa a explodir em combinações (ex.: Forma × Cor, Controle remoto × Dispositivo, Relatório × Formato de saída, Lógica × Banco de dados), quando uma classe monolítica tem várias variantes da mesma funcionalidade, ou quando a implementação precisa ser trocada em tempo de execução. Também use para revisar separações abstração/implementação.
---

# Bridge

**Categoria:** estrutural · **Também conhecido como:** Ponte

## Propósito

Dividir uma classe grande, ou um conjunto de classes muito ligadas, em duas hierarquias separadas, **abstração** e **implementação**, que evoluem de forma independente. A abstração guarda uma referência à implementação e delega a ela o trabalho de baixo nível.

"Abstração" aqui não é classe abstrata: é a camada de controle de alto nível (a UI, o controle remoto). "Implementação" é a plataforma que faz o trabalho (a API do SO, o dispositivo).

## Quando usar

- Você quer dividir e organizar uma classe monolítica que tem várias variantes da mesma funcionalidade (por exemplo, funcionar com vários bancos de dados). Mudanças numa variante não devem exigir mexer na classe toda.
- Você precisa estender uma classe em várias **dimensões ortogonais**. Extraia uma hierarquia por dimensão e delegue.
- Você precisa trocar a implementação em tempo de execução (basta atribuir outro objeto ao campo).

**Sinal clássico:** nomes de classe que são produtos cartesianos, como `CirculoVermelho`, `CirculoAzul`, `QuadradoVermelho`, `QuadradoAzul`. Cada dimensão nova multiplica as classes.

## Quando evitar

- A classe é altamente coesa e não tem dimensões independentes de verdade: dividi-la só complica.
- Há uma única implementação e nenhuma perspectiva de outra.

## Como implementar

1. Identifique as dimensões ortogonais: abstração/plataforma, domínio/infraestrutura, front-end/back-end, interface/implementação.
2. Defina, na abstração base, as operações de que o cliente precisa.
3. Determine as operações disponíveis em todas as plataformas e declare as necessárias numa **interface de implementação** comum.
4. Crie implementações concretas para cada plataforma, todas seguindo essa interface.
5. Na abstração, adicione um campo do tipo da interface de implementação e delegue a ele a maior parte do trabalho.
6. Se houver variantes de lógica de alto nível, crie **abstrações refinadas** estendendo a base.
7. O cliente passa a implementação para o construtor da abstração e daí em diante trabalha só com a abstração.

## Exemplo mínimo

```ts
interface Dispositivo {               // implementação
  ligado(): boolean
  ligar(): void; desligar(): void
  setVolume(v: number): void; getVolume(): number
}
class TV implements Dispositivo { /* ... */ }
class Radio implements Dispositivo { /* ... */ }

class ControleRemoto {                // abstração
  constructor(protected disp: Dispositivo) {}
  alternarEnergia() { this.disp.ligado() ? this.disp.desligar() : this.disp.ligar() }
  maisVolume() { this.disp.setVolume(this.disp.getVolume() + 10) }
}
class ControleAvancado extends ControleRemoto {   // abstração refinada
  mudo() { this.disp.setVolume(0) }
}

new ControleAvancado(new Radio()).mudo()
```

Duas hierarquias de tamanho N e M ficam com N + M classes em vez de N × M.

## Prós e contras

- ✅ Classes e aplicações independentes de plataforma.
- ✅ O cliente trabalha em alto nível, sem ver detalhes de plataforma.
- ✅ Aberto/fechado: abstrações e implementações novas entram independentemente.
- ✅ Responsabilidade única: lógica de alto nível de um lado, detalhes de plataforma do outro.
- ⚠️ Complica o código se aplicado a uma classe altamente coesa.

## Checklist de revisão

- [ ] As duas hierarquias variam de verdade por motivos diferentes?
- [ ] A abstração fala com a implementação só pela interface?
- [ ] A interface de implementação contém operações primitivas, e a abstração as combina em operações de alto nível?

## Padrões relacionados

- **Bridge** é desenhado de antemão; **Adapter** remenda código existente.
- **Bridge**, **State**, **Strategy** (e de certa forma Adapter) têm estrutura parecida, mas intenções diferentes. O padrão também comunica o problema resolvido.
- **Abstract Factory** pode encapsular quais implementações combinam com quais abstrações.
- Com **Builder**: o diretor é a abstração e os builders, as implementações.
