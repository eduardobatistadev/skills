---
name: memento
description: Aplica o padrão Memento (snapshot / retrato). Use quando for preciso salvar e restaurar estados anteriores de um objeto sem expor seus detalhes internos (ex.: desfazer/refazer em editores, checkpoints em jogos, rollback de transações ou operações que falharam, histórico de versões, estado de um wizard), especialmente quando acessar campos privados de fora violaria o encapsulamento. Também use para revisar implementações de snapshot e histórico.
---

# Memento

**Categoria:** comportamental · **Também conhecido como:** Lembrança, Retrato, Snapshot

## Propósito

Salvar e restaurar o estado anterior de um objeto sem revelar os detalhes de sua implementação. O próprio objeto produz o snapshot do seu estado, e só ele consegue lê-lo.

Papéis:
- **Originadora:** o objeto cujo estado é salvo. Cria mementos e restaura a partir deles.
- **Memento:** objeto imutável com o snapshot do estado.
- **Cuidadora (caretaker):** sabe *quando* salvar e restaurar e guarda a pilha de mementos, mas não lê o conteúdo.

## Quando usar

- Você quer produzir snapshots do estado de um objeto para poder voltar a um estado anterior. O caso clássico é o desfazer, mas também serve para **transações**: reverter uma operação quando ocorre um erro.
- Acessar diretamente campos, getters e setters do objeto violaria seu encapsulamento. Com o Memento, o objeto é o único responsável por fotografar a si mesmo.

**Por que não copiar de fora?** Para copiar o estado de fora você teria que torná-lo público, expondo a classe e acoplando quem copia a cada mudança interna.

## Quando evitar

- O estado é enorme e muda com frequência, e guardar cópias inteiras consumiria RAM demais. Considere comandos com operação inversa ou snapshots incrementais.
- O estado é simples e público. Um **Prototype** (clone) pode bastar.

## Como implementar

1. Defina a classe originadora. Veja se o programa tem um objeto central desse tipo ou vários pequenos.
2. Crie a classe memento com campos que espelham os campos da originadora.
3. Torne o memento **imutável**: recebe os dados uma única vez, pelo construtor, e não tem setters.
4. Se a linguagem suporta classes aninhadas, aninhe o memento na originadora. Se não, extraia uma interface vazia (ou só com metadados, como data e nome) para os outros objetos usarem, sem nada que exponha o estado.
5. Adicione à originadora um método que produz mementos, passando seu estado ao construtor do memento. O tipo de retorno é a interface extraída, se houver.
6. Adicione à originadora um método de restauração que recebe um memento (convertendo para a classe concreta, se você usou interface).
7. A cuidadora (um comando, um histórico...) decide quando pedir mementos, como armazená-los e quando restaurar.
8. Alternativa: o memento guarda a referência à originadora e oferece ele mesmo o `restaurar()`. Só faz sentido se o memento for aninhado ou a originadora tiver setters suficientes.

## Exemplo mínimo

```ts
class Editor {                                   // originadora
  private texto = ""; private cursor = 0

  salvar(): Snapshot { return new EditorSnapshot(this.texto, this.cursor) }
  restaurar(s: Snapshot) {
    const m = s as EditorSnapshot                 // só a originadora conhece o tipo concreto
    this.texto = m.texto; this.cursor = m.cursor
  }
}

interface Snapshot { readonly criadoEm: Date }   // a cuidadora só vê metadados

class EditorSnapshot implements Snapshot {
  readonly criadoEm = new Date()
  constructor(readonly texto: string, readonly cursor: number) {}  // imutável
}

class Historico {                                // cuidadora
  private pilha: Snapshot[] = []
  constructor(private editor: Editor) {}
  backup() { this.pilha.push(this.editor.salvar()) }
  desfazer() { const s = this.pilha.pop(); if (s) this.editor.restaurar(s) }
}
```

Em TypeScript, `readonly` e `as` não protegem em tempo de execução. Para encapsulamento real, use campos `#privados`, closures ou módulo privado.

## Prós e contras

- ✅ Snapshots do estado sem violar o encapsulamento.
- ✅ A originadora fica mais simples, já que a cuidadora mantém o histórico.
- ⚠️ Pode consumir muita RAM se mementos forem criados com frequência.
- ⚠️ A cuidadora precisa acompanhar o ciclo de vida da originadora para descartar mementos obsoletos.
- ⚠️ Linguagens dinâmicas (PHP, Python, JavaScript) não garantem que o estado do memento permaneça intacto.

## Checklist de revisão

- [ ] O memento é imutável?
- [ ] Só a originadora lê o conteúdo do memento?
- [ ] O histórico tem limite e política de descarte?
- [ ] Objetos mutáveis dentro do estado são copiados em profundidade?

## Padrões relacionados

- Com **Command** para desfazer: comandos executam operações e mementos guardam o estado anterior a cada uma.
- Com **Iterator**: captura e restaura o estado de uma iteração.
- **Prototype** pode ser uma alternativa mais simples quando o estado é direto e sem vínculos com recursos externos.
