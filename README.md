# Skills

Coleção de skills para o Claude.

## Padrões de projeto (GoF)

Uma skill por padrão de projeto clássico, escrita em português com base nas boas práticas do livro
*Mergulho nos Padrões de Projeto* (Alexander Shvets, Refactoring.Guru). Cada skill traz propósito,
quando usar e quando evitar, passos de implementação, um exemplo mínimo em TypeScript, prós e contras,
um checklist de revisão e as relações com outros padrões.

### Criacionais

| Skill | Resumo |
| --- | --- |
| [factory-method](skills/factory-method/SKILL.md) | Subclasses decidem qual classe concreta criar. |
| [abstract-factory](skills/abstract-factory/SKILL.md) | Cria famílias de objetos relacionados e compatíveis. |
| [builder](skills/builder/SKILL.md) | Monta objetos complexos passo a passo. |
| [prototype](skills/prototype/SKILL.md) | Copia objetos sem depender de suas classes. |
| [singleton](skills/singleton/SKILL.md) | Garante uma única instância com acesso global. |

### Estruturais

| Skill | Resumo |
| --- | --- |
| [adapter](skills/adapter/SKILL.md) | Faz interfaces incompatíveis colaborarem. |
| [bridge](skills/bridge/SKILL.md) | Separa abstração e implementação em hierarquias independentes. |
| [composite](skills/composite/SKILL.md) | Trata árvores de objetos como objetos individuais. |
| [decorator](skills/decorator/SKILL.md) | Adiciona comportamentos empilhando invólucros. |
| [facade](skills/facade/SKILL.md) | Interface simples para um subsistema complexo. |
| [flyweight](skills/flyweight/SKILL.md) | Compartilha estado para caber mais objetos na memória. |
| [proxy](skills/proxy/SKILL.md) | Substituto que controla o acesso ao objeto real. |

### Comportamentais

| Skill | Resumo |
| --- | --- |
| [chain-of-responsibility](skills/chain-of-responsibility/SKILL.md) | Passa pedidos por uma corrente de handlers. |
| [command](skills/command/SKILL.md) | Transforma pedidos em objetos (fila, histórico, desfazer). |
| [iterator](skills/iterator/SKILL.md) | Percorre coleções sem expor sua estrutura. |
| [mediator](skills/mediator/SKILL.md) | Centraliza a comunicação entre componentes. |
| [memento](skills/memento/SKILL.md) | Salva e restaura estados sem quebrar o encapsulamento. |
| [observer](skills/observer/SKILL.md) | Assinatura de eventos para notificar vários objetos. |
| [state](skills/state/SKILL.md) | Muda o comportamento conforme o estado interno. |
| [strategy](skills/strategy/SKILL.md) | Algoritmos intercambiáveis em tempo de execução. |
| [template-method](skills/template-method/SKILL.md) | Esqueleto de algoritmo com etapas sobrescrevíveis. |
| [visitor](skills/visitor/SKILL.md) | Separa operações da estrutura de objetos. |

## Como usar

Cada pasta em `skills/` contém um `SKILL.md` com frontmatter `name` e `description`. Copie as pastas
desejadas para `~/.claude/skills/` (uso pessoal) ou `.claude/skills/` (no projeto), ou envie-as como
skills na sua conta do Claude. A `description` diz ao Claude quando acionar cada skill.
