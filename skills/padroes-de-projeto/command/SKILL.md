---
name: command
description: Aplica o padrão Command (comando / ação / transação). Use quando for preciso transformar operações em objetos: implementar desfazer/refazer, enfileirar, agendar, registrar em log ou executar remotamente tarefas, parametrizar botões, menus e atalhos com ações, montar macros (comandos compostos), ou desacoplar a UI da lógica de negócio. Também use para revisar implementações de Command, histórico e undo.
---

# Command

**Categoria:** comportamental · **Também conhecido como:** Comando, Ação, Action, Transação

## Propósito

Transformar um pedido num objeto independente que contém toda a informação sobre ele (destinatário, método, argumentos). Isso permite passar pedidos como argumentos, atrasar ou enfileirar sua execução e suportar operações reversíveis.

Papéis:
- **Remetente (invoker):** dispara o comando (botão, atalho, fila). Não sabe o que ele faz.
- **Comando:** encapsula a chamada e seus parâmetros.
- **Destinatário (receiver):** objeto com a lógica de negócio real.
- **Cliente:** cria e liga tudo.

## Quando usar

- Você quer **parametrizar objetos com operações**: um item de menu, botão ou atalho que o usuário configura com a ação que quiser.
- Você quer **enfileirar, agendar ou executar remotamente** operações. Comandos podem ser serializados, gravados, enviados pela rede e restaurados depois.
- Você quer **operações reversíveis** (desfazer/refazer). Mantenha uma pilha de comandos executados, cada um com um backup do estado ou com a operação inversa.

**Sinal no código:** várias classes de UI (botão, menu, atalho) duplicando a mesma chamada de negócio; ou a UI chamando a lógica de negócio diretamente.

## Quando evitar

- A operação é simples, direta e nunca vai ser enfileirada, desfeita ou reconfigurada. A camada extra só atrapalha.
- Na linguagem, uma função/closure já resolve. Não crie classes se não precisar de undo, serialização ou estado.

## Como implementar

1. Declare a interface de comando com um único método de execução (opcionalmente `desfazer()`).
2. Extraia os pedidos para classes concretas de comando. Cada uma guarda os argumentos e a referência ao destinatário, todos recebidos pelo construtor.
3. Identifique os remetentes e dê a eles campos para armazenar comandos. Eles falam com comandos só pela interface e normalmente os recebem do cliente, sem criá-los.
4. Faça os remetentes executarem o comando em vez de chamar o destinatário diretamente.
5. O cliente inicializa nesta ordem: destinatários → comandos (associados aos destinatários) → remetentes (associados aos comandos).

**Para desfazer:** guarde o estado anterior antes de executar (use **Memento** se o estado for privado) ou implemente a operação inversa. Backups consomem RAM; operações inversas podem ser difíceis ou impossíveis. Escolha conforme o caso.

## Exemplo mínimo

```ts
interface Comando { executar(): boolean; desfazer(): void }

class ComandoColar implements Comando {
  private backup = ""
  constructor(private editor: Editor, private clipboard: Clipboard) {}
  executar() {
    this.backup = this.editor.texto
    this.editor.inserir(this.clipboard.conteudo)
    return true                                   // mudou estado: vai para o histórico
  }
  desfazer() { this.editor.texto = this.backup }
}

class Historico {
  private pilha: Comando[] = []
  executar(c: Comando) { if (c.executar()) this.pilha.push(c) }
  desfazer() { this.pilha.pop()?.desfazer() }
}

// botão, atalho Ctrl+V e menu usam o mesmo comando
const historico = new Historico()
botaoColar.onClick = () => historico.executar(new ComandoColar(editor, clipboard))
atalho("Ctrl+Z", () => historico.desfazer())
```

## Prós e contras

- ✅ Responsabilidade única: desacopla quem invoca de quem executa.
- ✅ Aberto/fechado: novos comandos sem quebrar o cliente.
- ✅ Desfazer/refazer.
- ✅ Execução adiada.
- ✅ Comandos simples combinados em um complexo (macro).
- ⚠️ Uma camada nova entre remetentes e destinatários deixa o código mais complexo.

## Checklist de revisão

- [ ] O remetente depende só da interface `Comando`?
- [ ] O comando delega a lógica ao destinatário em vez de reimplementá-la?
- [ ] Comandos que não alteram estado ficam fora do histórico?
- [ ] O histórico tem limite de tamanho (consumo de RAM)?
- [ ] Comandos serializáveis guardam só dados, não referências frágeis?

## Padrões relacionados

- **Chain of Responsibility**, Command, **Mediator** e **Observer** são formas de ligar remetentes e destinatários. Command cria uma ligação unidirecional.
- Handlers de uma **CoR** podem ser comandos.
- Com **Memento** para desfazer: o comando executa e o memento guarda o estado anterior.
- **Strategy** também parametriza um objeto com uma ação, mas descreve maneiras diferentes de fazer *a mesma coisa*; Command transforma *qualquer* operação em objeto.
- **Prototype** ajuda a guardar cópias de comandos no histórico.
- **Visitor** pode ser visto como um Command poderoso, que opera sobre objetos de várias classes.
