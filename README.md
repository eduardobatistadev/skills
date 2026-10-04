# Skills

Coleção de skills para o Claude.

## Padrões de projeto (GoF)

Pasta: [`skills/padroes-de-projeto/`](skills/padroes-de-projeto/). Uma skill por padrão de projeto clássico, escrita em português com base nas boas práticas do livro
*Mergulho nos Padrões de Projeto* (Alexander Shvets, Refactoring.Guru). Cada skill traz propósito,
quando usar e quando evitar, passos de implementação, um exemplo mínimo em TypeScript, prós e contras,
um checklist de revisão e as relações com outros padrões.

### Criacionais

| Skill | Resumo |
| --- | --- |
| [factory-method](skills/padroes-de-projeto/factory-method/SKILL.md) | Subclasses decidem qual classe concreta criar. |
| [abstract-factory](skills/padroes-de-projeto/abstract-factory/SKILL.md) | Cria famílias de objetos relacionados e compatíveis. |
| [builder](skills/padroes-de-projeto/builder/SKILL.md) | Monta objetos complexos passo a passo. |
| [prototype](skills/padroes-de-projeto/prototype/SKILL.md) | Copia objetos sem depender de suas classes. |
| [singleton](skills/padroes-de-projeto/singleton/SKILL.md) | Garante uma única instância com acesso global. |

### Estruturais

| Skill | Resumo |
| --- | --- |
| [adapter](skills/padroes-de-projeto/adapter/SKILL.md) | Faz interfaces incompatíveis colaborarem. |
| [bridge](skills/padroes-de-projeto/bridge/SKILL.md) | Separa abstração e implementação em hierarquias independentes. |
| [composite](skills/padroes-de-projeto/composite/SKILL.md) | Trata árvores de objetos como objetos individuais. |
| [decorator](skills/padroes-de-projeto/decorator/SKILL.md) | Adiciona comportamentos empilhando invólucros. |
| [facade](skills/padroes-de-projeto/facade/SKILL.md) | Interface simples para um subsistema complexo. |
| [flyweight](skills/padroes-de-projeto/flyweight/SKILL.md) | Compartilha estado para caber mais objetos na memória. |
| [proxy](skills/padroes-de-projeto/proxy/SKILL.md) | Substituto que controla o acesso ao objeto real. |

### Comportamentais

| Skill | Resumo |
| --- | --- |
| [chain-of-responsibility](skills/padroes-de-projeto/chain-of-responsibility/SKILL.md) | Passa pedidos por uma corrente de handlers. |
| [command](skills/padroes-de-projeto/command/SKILL.md) | Transforma pedidos em objetos (fila, histórico, desfazer). |
| [iterator](skills/padroes-de-projeto/iterator/SKILL.md) | Percorre coleções sem expor sua estrutura. |
| [mediator](skills/padroes-de-projeto/mediator/SKILL.md) | Centraliza a comunicação entre componentes. |
| [memento](skills/padroes-de-projeto/memento/SKILL.md) | Salva e restaura estados sem quebrar o encapsulamento. |
| [observer](skills/padroes-de-projeto/observer/SKILL.md) | Assinatura de eventos para notificar vários objetos. |
| [state](skills/padroes-de-projeto/state/SKILL.md) | Muda o comportamento conforme o estado interno. |
| [strategy](skills/padroes-de-projeto/strategy/SKILL.md) | Algoritmos intercambiáveis em tempo de execução. |
| [template-method](skills/padroes-de-projeto/template-method/SKILL.md) | Esqueleto de algoritmo com etapas sobrescrevíveis. |
| [visitor](skills/padroes-de-projeto/visitor/SKILL.md) | Separa operações da estrutura de objetos. |

## Qual padrão usar?

Comece pelo problema que você tem, não pelo padrão. Encontre abaixo a situação mais parecida com a sua.

### Criar objetos

| Situação | Padrão |
| --- | --- |
| O código faz `new ClasseConcreta()` espalhado e precisa aceitar novos tipos sem mudar o cliente. | [Factory Method](skills/padroes-de-projeto/factory-method/SKILL.md) |
| Você cria **famílias** de objetos que precisam combinar entre si (tema claro/escuro, Windows/Mac, um driver por banco). | [Abstract Factory](skills/padroes-de-projeto/abstract-factory/SKILL.md) |
| O construtor tem muitos parâmetros opcionais, ou o objeto é montado em etapas. | [Builder](skills/padroes-de-projeto/builder/SKILL.md) |
| Você precisa copiar um objeto sem conhecer sua classe, ou criar a partir de modelos pré-configurados. | [Prototype](skills/padroes-de-projeto/prototype/SKILL.md) |
| Deve existir exatamente uma instância compartilhada (configuração, conexão). Antes, considere injeção de dependência. | [Singleton](skills/padroes-de-projeto/singleton/SKILL.md) |

### Organizar a estrutura

| Situação | Padrão |
| --- | --- |
| Uma biblioteca de terceiros ou legada tem interface diferente da que seu código espera. | [Adapter](skills/padroes-de-projeto/adapter/SKILL.md) |
| As classes se multiplicam em combinações de duas dimensões (`CirculoVermelho`, `QuadradoAzul`...). | [Bridge](skills/padroes-de-projeto/bridge/SKILL.md) |
| Os dados formam uma árvore (pastas e arquivos, menus, caixas com produtos) e você quer tratar item e grupo do mesmo jeito. | [Composite](skills/padroes-de-projeto/composite/SKILL.md) |
| Você quer empilhar comportamentos extras (log, cache, compressão, criptografia) sem criar uma subclasse por combinação. | [Decorator](skills/padroes-de-projeto/decorator/SKILL.md) |
| Usar um subsistema exige muitas classes e passos, mas você só precisa de algumas operações. | [Facade](skills/padroes-de-projeto/facade/SKILL.md) |
| Há milhões de objetos parecidos e a memória está acabando. | [Flyweight](skills/padroes-de-projeto/flyweight/SKILL.md) |
| Você precisa controlar o acesso a um objeto (carregar só quando usar, checar permissão, cache, chamada remota) sem mudar a interface. | [Proxy](skills/padroes-de-projeto/proxy/SKILL.md) |

### Distribuir comportamento

| Situação | Padrão |
| --- | --- |
| Um pedido passa por uma sequência de verificações ou processadores (middlewares, validações, alçadas de aprovação). | [Chain of Responsibility](skills/padroes-de-projeto/chain-of-responsibility/SKILL.md) |
| Você quer desfazer/refazer, enfileirar, agendar ou registrar operações, ou ligar botões e atalhos a ações. | [Command](skills/padroes-de-projeto/command/SKILL.md) |
| Você precisa percorrer uma coleção complexa sem expor como ela é guardada. | [Iterator](skills/padroes-de-projeto/iterator/SKILL.md) |
| Vários componentes se chamam uns aos outros numa teia (campos de formulário que se afetam). | [Mediator](skills/padroes-de-projeto/mediator/SKILL.md) |
| Você precisa salvar e restaurar o estado de um objeto (undo, checkpoint, rollback) sem expor campos privados. | [Memento](skills/padroes-de-projeto/memento/SKILL.md) |
| Quando algo acontece num objeto, outros precisam ser avisados, e quem quer ouvir muda com o tempo. | [Observer](skills/padroes-de-projeto/observer/SKILL.md) |
| O comportamento depende de um status, e há `switch (status)` repetido em vários métodos. | [State](skills/padroes-de-projeto/state/SKILL.md) |
| Há várias formas de fazer a mesma coisa (frete, pagamento, desconto) e você quer escolher ou trocar em tempo de execução. | [Strategy](skills/padroes-de-projeto/strategy/SKILL.md) |
| Várias classes seguem o mesmo algoritmo e só diferem em alguns passos. | [Template Method](skills/padroes-de-projeto/template-method/SKILL.md) |
| Você precisa adicionar operações (exportar, calcular, validar) sobre objetos de classes diferentes sem mexer nelas. | [Visitor](skills/padroes-de-projeto/visitor/SKILL.md) |

### Padrões que costumam ser confundidos

- **Strategy × State:** no Strategy o cliente escolhe o algoritmo e as estratégias não se conhecem. No State os próprios estados trocam o estado do objeto.
- **Strategy × Template Method:** Strategy troca o comportamento por composição, em tempo de execução. Template Method varia passos por herança e é fixo por classe.
- **Decorator × Proxy × Adapter:** o Decorator acrescenta comportamento com a mesma interface e o cliente monta a pilha. O Proxy controla o acesso com a mesma interface e gerencia o objeto real. O Adapter muda a interface.
- **Facade × Mediator:** a Facade simplifica o acesso a um subsistema que nem sabe que ela existe. O Mediator é o único canal de comunicação entre os componentes.
- **Observer × Mediator:** o Observer cria assinaturas dinâmicas de mão única. O Mediator elimina dependências cruzadas centralizando a coordenação, e às vezes usa o Observer por baixo.
- **Command × Strategy:** o Command transforma qualquer operação em objeto (fila, histórico, undo). O Strategy oferece formas diferentes de fazer a mesma coisa.
- **Abstract Factory × Builder:** a Abstract Factory entrega na hora uma família de objetos compatíveis. O Builder monta um único objeto complexo em etapas.

> Dica: se o problema ainda não existe, não aplique o padrão. Todos eles adicionam classes e indireção, e só se pagam quando a variação ou a complexidade é real.

## Como usar

As skills ficam agrupadas por tema em `skills/<tema>/`; os padrões de projeto estão em `skills/padroes-de-projeto/`. Cada pasta de skill contém um `SKILL.md` com frontmatter `name` e `description`. Copie as pastas
desejadas para `~/.claude/skills/` (uso pessoal) ou `.claude/skills/` (no projeto), ou envie-as como
skills na sua conta do Claude. A `description` diz ao Claude quando acionar cada skill.
