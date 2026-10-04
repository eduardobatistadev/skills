---
name: chain-of-responsibility
description: Aplica o padrão Chain of Responsibility (CoR / corrente de responsabilidade). Use quando um pedido deve passar por uma sequência de verificações ou processadores que decidem tratá-lo ou repassá-lo (ex.: autenticação → autorização → validação → cache em requisições, middlewares HTTP, pipelines de aprovação por alçada, tratamento de eventos de UI que sobem pela hierarquia), especialmente se a ordem ou o conjunto de handlers muda em tempo de execução. Também use para revisar cadeias existentes.
---

# Chain of Responsibility

**Categoria:** comportamental · **Também conhecido como:** CoR, Corrente de responsabilidade, Corrente de comando

## Propósito

Passar um pedido por uma corrente de handlers. Cada handler decide se processa o pedido e se o repassa ao próximo. O remetente não precisa saber quem, afinal, vai tratar o pedido.

Existem duas variantes comuns:
- **Filtro/pipeline:** cada handler faz sua parte e pode interromper a cadeia (ex.: autenticação falhou, para tudo).
- **Primeiro que puder trata:** o pedido corre até que um handler o assuma e pare (ex.: eventos de UI subindo do botão até a janela).

## Quando usar

- O programa deve processar tipos diferentes de pedidos de maneiras diferentes, mas os tipos exatos e suas sequências não são conhecidos de antemão.
- É essencial executar vários handlers numa ordem específica.
- O conjunto de handlers e sua ordem devem poder mudar em tempo de execução (com setters no campo "próximo").

**Sinal no código:** um método que cresce com verificações sequenciais (`if (!autenticado) ...; if (!temPermissao) ...; if (!valido) ...; if (cache) ...`), copiadas para outros lugares.

## Quando evitar

- Há uma sequência fixa e curta de passos que nunca muda. Chamadas diretas são mais legíveis.
- É crítico garantir que todo pedido seja tratado e não há um handler final de fallback.

## Como implementar

1. Declare a interface do handler com o método de tratamento. Decida como passar os dados: o mais flexível é transformar o pedido num objeto e passá-lo como argumento.
2. Para eliminar código repetido, crie um **handler base abstrato** com o campo para o próximo handler. Considere torná-lo imutável; se a cadeia precisa mudar em execução, forneça um setter. Implemente nele o comportamento padrão de repassar ao próximo, se existir.
3. Crie os handlers concretos. Cada um decide duas coisas: se processa o pedido e se o repassa adiante.
4. O cliente monta a cadeia sozinho ou recebe cadeias prontas. No segundo caso, use fábricas que montam a cadeia conforme configuração ou ambiente.
5. O cliente pode acionar qualquer handler, não só o primeiro.
6. Esteja pronto para três cenários: a cadeia tem um único elo; alguns pedidos não chegam ao fim; outros chegam ao fim sem tratamento.

## Exemplo mínimo

```ts
interface Handler {
  setProximo(h: Handler): Handler
  tratar(req: Requisicao): Resposta | null
}

abstract class HandlerBase implements Handler {
  private proximo?: Handler
  setProximo(h: Handler) { this.proximo = h; return h }        // permite encadear
  tratar(req: Requisicao) { return this.proximo ? this.proximo.tratar(req) : null }
}

class Autenticacao extends HandlerBase {
  tratar(req: Requisicao) {
    if (!req.token) return { status: 401 }                     // interrompe
    return super.tratar(req)
  }
}
class Autorizacao extends HandlerBase { /* ... */ }
class Validacao extends HandlerBase { /* ... */ }

const cadeia = new Autenticacao()
cadeia.setProximo(new Autorizacao()).setProximo(new Validacao())
cadeia.tratar(req)
```

## Prós e contras

- ✅ Você controla a ordem de tratamento.
- ✅ Responsabilidade única: desacopla quem invoca de quem executa.
- ✅ Aberto/fechado: novos handlers sem quebrar o cliente.
- ⚠️ Alguns pedidos podem terminar sem tratamento.

## Checklist de revisão

- [ ] Cada handler tem uma única responsabilidade?
- [ ] Existe um comportamento definido quando ninguém trata o pedido (fallback, erro explícito)?
- [ ] A montagem da cadeia está centralizada (fábrica/configuração)?
- [ ] Não há risco de ciclo na cadeia?

## Padrões relacionados

- **Chain of Responsibility**, **Command**, **Mediator** e **Observer** são formas de ligar remetentes e destinatários. CoR passa o pedido sequencialmente até alguém agir; Command cria ligação unidirecional; Mediator elimina ligações diretas; Observer permite inscrição dinâmica.
- Usado com **Composite**: uma folha pode repassar o pedido aos pais até a raiz.
- Handlers podem ser **Commands** (várias operações sobre o mesmo contexto), ou o pedido pode ser um Command (a mesma operação em vários contextos).
- **Decorator** tem estrutura parecida, mas decoradores não podem interromper o fluxo; handlers podem.
