---
name: singleton
description: Aplica (ou questiona) o padrão Singleton. Use quando uma classe deve ter exatamente uma instância compartilhada com ponto de acesso global (ex.: conexão de banco, configuração, logger), quando se quer substituir variáveis globais por algo controlado, ou para revisar singletons existentes quanto a thread safety, testabilidade e acoplamento oculto. Inclui alternativas como injeção de dependência.
---

# Singleton

**Categoria:** criacional · **Também conhecido como:** Carta única

## Propósito

Garantir que uma classe tenha apenas uma instância e oferecer um ponto de acesso global a ela.

Note que o padrão resolve **dois problemas ao mesmo tempo** (instância única + acesso global), e por isso viola o princípio de responsabilidade única. Muita gente chama de singleton qualquer coisa que resolva só um deles.

## Quando usar

- Uma classe precisa ter uma única instância disponível para todos os clientes, como um objeto de banco de dados compartilhado por várias partes do programa.
- Você precisa de um controle mais estrito do que uma variável global oferece: ninguém além da própria classe pode substituir a instância guardada.

## Quando evitar

- Quando o objetivo real é só "ter uma instância por aplicação". Geralmente é melhor criar uma instância na composição da aplicação e **injetá-la** onde for necessária, o que mantém o código testável.
- Quando o singleton está escondendo dependências: componentes que sabem demais uns dos outros (o padrão pode mascarar um design ruim).
- Quando o estado global mutável compartilhado vai complicar testes ou concorrência.

## Como implementar

1. Adicione um campo estático privado para guardar a instância.
2. Declare um método estático público de criação (`getInstance()`).
3. Implemente inicialização preguiçosa: na primeira chamada cria o objeto e guarda no campo; nas seguintes devolve sempre o mesmo.
4. Torne o construtor privado. Só o método estático pode chamá-lo.
5. Substitua no cliente todas as chamadas diretas ao construtor por `getInstance()`.

Em ambiente multithread, proteja a criação (lock com dupla verificação, inicialização estática/eager, ou o mecanismo idiomático da linguagem) para que duas threads não criem duas instâncias.

## Exemplo mínimo

```ts
class Database {
  private static instancia: Database | null = null
  private constructor() { /* abre conexão */ }

  static getInstance(): Database {
    if (!Database.instancia) Database.instancia = new Database()
    return Database.instancia
  }

  query(sql: string) { /* ponto central: cache, throttling etc. */ }
}

Database.getInstance().query("SELECT ...")
```

Em linguagens com módulos (JS/TS, Python), uma instância exportada pelo módulo costuma bastar. Prefira isso a uma classe cheia de cerimônia.

Para relaxar a regra (por exemplo, permitir N instâncias), só o corpo de `getInstance()` precisa mudar.

## Prós e contras

- ✅ Certeza de que só existe uma instância.
- ✅ Ponto de acesso global a ela.
- ✅ Inicialização só quando pedida pela primeira vez.
- ⚠️ Viola responsabilidade única.
- ⚠️ Pode mascarar design ruim e acoplamento.
- ⚠️ Exige cuidado com concorrência.
- ⚠️ Dificulta testes unitários: construtor privado e métodos estáticos são difíceis de simular.

## Checklist de revisão

- [ ] Existe mesmo uma razão de domínio para haver só uma instância?
- [ ] A criação é segura entre threads?
- [ ] Os clientes poderiam receber a instância por injeção em vez de buscá-la globalmente?
- [ ] Os testes conseguem substituir ou reiniciar a instância?

## Padrões relacionados

- Um **Facade** muitas vezes pode ser singleton, pois uma fachada costuma bastar.
- **Flyweight** se parece com Singleton se todo o estado compartilhado fosse reduzido a um objeto, mas: flyweights podem ter várias instâncias com estados intrínsecos diferentes e são imutáveis; o singleton é único e pode ser mutável.
- **Abstract Factory**, **Builder** e **Prototype** podem ser implementados como singletons.
