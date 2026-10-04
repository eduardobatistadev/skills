---
name: adapter
description: Aplica o padrão Adapter (adaptador / wrapper). Use quando for preciso integrar uma classe existente, legada ou de terceiros cuja interface ou formato de dados é incompatível com o resto do código (ex.: API que devolve XML onde o sistema espera JSON, SDK externo com assinatura diferente), ou quando várias subclasses existentes precisam ganhar uma funcionalidade comum sem duplicação. Também use para revisar adaptadores existentes.
---

# Adapter

**Categoria:** estrutural · **Também conhecido como:** Adaptador, Wrapper

## Propósito

Permitir que objetos com interfaces incompatíveis colaborem. O adaptador implementa a interface que o cliente espera e traduz as chamadas para o objeto de serviço envolvido.

## Quando usar

- Você quer usar uma classe existente, mas a interface dela não é compatível com o resto do código. O adaptador vira o tradutor entre seu código e a classe antiga, de terceiros ou de interface estranha.
- Você quer reaproveitar várias subclasses existentes às quais falta uma funcionalidade comum que não pode ir para a superclasse. Em vez de estender cada uma (duplicando código), coloque a funcionalidade num adaptador que envolve qualquer objeto da hierarquia (abordagem parecida com o Decorator).

## Quando evitar

- Você controla a classe de serviço e mudá-la para se encaixar é simples e seguro. Às vezes isso é mais barato que uma camada nova.
- A interface já é compatível e você só quer adicionar comportamento: isso é Decorator ou Proxy.

## Como implementar

1. Identifique as duas partes: uma **classe de serviço** útil que você não pode mudar (de terceiros, legada, com muitas dependências) e uma ou mais **classes cliente** que se beneficiariam dela.
2. Declare a **interface do cliente**: como o cliente quer se comunicar.
3. Crie a classe adaptadora implementando essa interface (métodos vazios por enquanto).
4. Adicione ao adaptador um campo com a referência ao serviço, normalmente recebida pelo construtor (às vezes faz mais sentido recebê-la a cada chamada).
5. Implemente os métodos um a um, **delegando o trabalho real ao serviço**. O adaptador só converte interface e formato de dados, sem lógica de negócio.
6. Faça o cliente usar o adaptador sempre pela interface do cliente. Assim é possível trocar ou estender adaptadores sem afetar o cliente.

## Exemplo mínimo

```ts
// o que o sistema espera
interface ProvedorDeCotacao { cotacao(ticker: string): number }

// SDK de terceiros que não podemos mudar
class LegacyMarketXml { fetchQuoteXml(symbol: string): string { /* <q price="12.3"/> */ return "" } }

class AdaptadorMarketXml implements ProvedorDeCotacao {
  constructor(private sdk: LegacyMarketXml) {}
  cotacao(ticker: string): number {
    const xml = this.sdk.fetchQuoteXml(ticker.toUpperCase())
    return parseFloat(/price="([\d.]+)"/.exec(xml)![1])    // só tradução
  }
}

function painel(p: ProvedorDeCotacao) { console.log(p.cotacao("petr4")) }
painel(new AdaptadorMarketXml(new LegacyMarketXml()))
```

Há duas variantes: **adaptador de objeto** (composição, como acima, a recomendada) e **adaptador de classe** (herança múltipla do cliente e do serviço, só em linguagens que a suportam).

## Prós e contras

- ✅ Responsabilidade única: conversão de interface/dados separada da lógica de negócio.
- ✅ Aberto/fechado: novos adaptadores entram sem quebrar o cliente, desde que ele use a interface.
- ⚠️ Mais interfaces e classes. Às vezes é mais simples ajustar o próprio serviço.

## Checklist de revisão

- [ ] O adaptador só traduz, sem regras de negócio?
- [ ] O cliente depende da interface do cliente, não do adaptador concreto nem do serviço?
- [ ] Erros e exceções do serviço são convertidos para o vocabulário do cliente?

## Padrões relacionados

- **Bridge** é planejado de antemão; **Adapter** é aplicado a código existente para fazer peças incompatíveis conversarem.
- **Adapter** muda a interface; **Proxy** mantém a mesma; **Decorator** mantém ou estende e permite composição recursiva.
- **Facade** cria uma interface nova para um subsistema inteiro; Adapter torna utilizável uma interface existente, geralmente de um único objeto.
- **Bridge**, **State**, **Strategy** e, de certa forma, Adapter têm estrutura parecida (composição), mas resolvem problemas diferentes.
