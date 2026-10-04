---
name: template-method
description: Aplica o padrão Template Method (método padrão / modelo). Use quando várias classes implementam algoritmos quase idênticos com pequenas diferenças (ex.: importadores de CSV/PDF/DOC, pipelines de processamento, jobs com etapas fixas, geração de relatórios, IA de jogos com turnos), quando se quer permitir que extensões sobrescrevam só certas etapas mantendo a estrutura do algoritmo, ou quando é preciso adicionar ganchos (hooks) num framework. Também use para revisar hierarquias com código duplicado entre subclasses.
---

# Template Method

**Categoria:** comportamental · **Também conhecido como:** Método padrão

## Propósito

Definir o esqueleto de um algoritmo na superclasse e deixar que subclasses sobrescrevam etapas específicas sem mudar sua estrutura. O método template chama as etapas na ordem certa; algumas são abstratas, outras têm implementação padrão, e algumas são ganchos opcionais.

Tipos de etapa:
- **Abstratas:** toda subclasse precisa implementar.
- **Opcionais:** têm implementação padrão e podem ser sobrescritas.
- **Ganchos (hooks):** etapas opcionais com corpo vazio, colocadas antes ou depois de pontos cruciais, para que subclasses possam se encaixar.

## Quando usar

- Você quer deixar clientes estenderem só algumas etapas de um algoritmo, não o algoritmo inteiro nem sua estrutura.
- Você tem várias classes com algoritmos quase idênticos e pequenas diferenças, e teria de mexer em todas sempre que o algoritmo mudasse. Suba as etapas iguais para a superclasse e deixe nas subclasses só o que varia.

**Sinal no código:** `MineradorCSV`, `MineradorPDF` e `MineradorDOC` com os mesmos passos (abrir, extrair, analisar, gerar relatório, fechar), código de análise e relatório copiado entre eles.

## Quando evitar

- As variações são combinatórias ou precisam mudar em tempo de execução. Use **Strategy** (composição).
- O algoritmo tem tantas etapas que a superclasse fica difícil de entender e manter.
- Subclasses precisariam suprimir etapas da superclasse. Isso viola Liskov e indica que a abstração está errada.

## Como implementar

1. Analise o algoritmo e veja se dá para quebrá-lo em etapas. Separe as comuns a todas as subclasses das que serão únicas.
2. Crie a classe base abstrata, declare o método template e as etapas como métodos abstratos. O template chama as etapas em ordem. Considere torná-lo **final** para impedir que subclasses o sobrescrevam.
3. Tudo bem se todas as etapas forem abstratas, mas algumas podem ganhar implementação padrão, e aí as subclasses não precisam implementá-las.
4. Considere adicionar **ganchos** entre as etapas cruciais.
5. Para cada variação, crie uma subclasse concreta que implementa todas as etapas abstratas e, se quiser, sobrescreve as opcionais.

## Exemplo mínimo

```ts
abstract class MineradorDeDados {
  // método template: a estrutura não muda
  minerar(caminho: string): Relatorio {
    const arquivo = this.abrir(caminho)
    const bruto = this.extrair(arquivo)
    const dados = this.converter(bruto)
    const analise = this.analisar(dados)           // comum
    this.depoisDaAnalise(analise)                  // gancho
    const rel = this.gerarRelatorio(analise)       // comum
    this.fechar(arquivo)
    return rel
  }

  protected abstract abrir(c: string): Arquivo
  protected abstract extrair(a: Arquivo): string
  protected abstract converter(bruto: string): Dados
  protected abstract fechar(a: Arquivo): void

  protected analisar(d: Dados): Analise { /* padrão para todos */ return {} as Analise }
  protected gerarRelatorio(a: Analise): Relatorio { /* padrão */ return {} as Relatorio }
  protected depoisDaAnalise(_a: Analise): void {}  // gancho vazio
}

class MineradorCSV extends MineradorDeDados {
  protected abrir(c: string) { /* ... */ return {} as Arquivo }
  protected extrair(a: Arquivo) { return "" }
  protected converter(b: string) { return {} as Dados }
  protected fechar(a: Arquivo) {}
}
```

TypeScript não tem `final`. Documente que `minerar` não deve ser sobrescrito ou use composição.

## Prós e contras

- ✅ Clientes sobrescrevem só partes de um algoritmo grande e ficam menos expostos a mudanças em outras partes.
- ✅ Código duplicado sobe para a superclasse.
- ⚠️ Alguns clientes ficam limitados pelo esqueleto fornecido.
- ⚠️ Pode violar o princípio de substituição de Liskov se uma subclasse suprimir uma etapa padrão.
- ⚠️ Quanto mais etapas, mais difícil de manter.

## Checklist de revisão

- [ ] O método template está protegido contra sobrescrita (final, ou convenção documentada)?
- [ ] Etapas abstratas são realmente obrigatórias para todas as subclasses?
- [ ] Ganchos têm nomes claros (`antesDe...`, `depoisDe...`) e corpo vazio por padrão?
- [ ] Nenhuma subclasse anula comportamento da base de um jeito que quebre o contrato?

## Padrões relacionados

- **Factory Method** é uma especialização do Template Method e pode ser uma etapa dele.
- **Strategy** é o equivalente baseado em composição: Template Method funciona no nível de classe (estático); Strategy, no de objeto (troca em tempo de execução).
