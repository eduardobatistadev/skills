---
name: facade
description: Aplica o padrão Facade (fachada). Use quando o código cliente precisa usar um subsistema, biblioteca ou framework complexo (muitas classes, inicialização trabalhosa, ordem de chamadas) e só precisa de uma fração das funcionalidades, quando se quer isolar o código de mudanças numa dependência de terceiros, ou quando se quer organizar um subsistema em camadas com pontos de entrada. Também use para revisar fachadas que viraram "objetos deus".
---

# Facade

**Categoria:** estrutural · **Também conhecido como:** Fachada

## Propósito

Oferecer uma interface simplificada para uma biblioteca, um framework ou qualquer conjunto complexo de classes. A fachada expõe só o que o cliente realmente precisa e cuida de orquestrar os objetos do subsistema.

## Quando usar

- Você precisa de uma interface limitada, mas simples, para um subsistema complexo. Com o tempo subsistemas ganham classes e configuração; a fachada vira um atalho para os usos mais comuns.
- Você quer estruturar um subsistema em **camadas**: crie uma fachada como ponto de entrada de cada camada e faça as camadas se comunicarem só por elas, reduzindo o acoplamento.
- Você quer que o código de negócio não dependa diretamente de dezenas de classes de uma biblioteca de terceiros (conversão de vídeo, pagamento, envio de e-mail...).

## Quando evitar

- O cliente realmente precisa de todo o poder do subsistema. A fachada pode coexistir, mas não deve bloquear o acesso direto quando necessário.
- A fachada começaria a acumular regras de negócio de áreas diferentes. Nesse caso divida em fachadas menores.

## Como implementar

1. Verifique se é possível oferecer uma interface mais simples do que a do subsistema. Bom sinal: ela torna o cliente independente de muitas classes do subsistema.
2. Declare e implemente essa interface numa classe fachada que redireciona as chamadas aos objetos certos do subsistema. Ela deve **inicializar o subsistema e gerenciar seu ciclo de vida**, a menos que o cliente já faça isso.
3. Para o benefício completo, faça todo o código cliente falar com o subsistema só pela fachada. Quando o subsistema mudar de versão, só a fachada muda.
4. Se a fachada ficar grande demais, extraia parte do comportamento para uma nova **fachada refinada**.

## Exemplo mínimo

```ts
// subsistema complexo (biblioteca de terceiros)
// VideoFile, CodecFactory, OggCodec, MPEG4Codec, BitrateReader, AudioMixer...

class ConversorDeVideo {                // fachada
  converter(arquivo: string, formato: "mp4" | "ogg"): Buffer {
    const fonte = new VideoFile(arquivo)
    const codecOrigem = CodecFactory.extract(fonte)
    const destino = formato === "mp4" ? new MPEG4Codec() : new OggCodec()
    const buffer = BitrateReader.read(arquivo, codecOrigem)
    const resultado = BitrateReader.convert(buffer, destino)
    return new AudioMixer().fix(resultado)
  }
}

// cliente: uma linha, nenhuma classe do subsistema
const mp4 = new ConversorDeVideo().converter("gato.ogg", "mp4")
```

A fachada não acrescenta funcionalidade nova ao subsistema e o subsistema nem sabe que ela existe.

## Prós e contras

- ✅ Isola o código da complexidade do subsistema.
- ⚠️ A fachada pode virar um **objeto deus** acoplado a todas as classes da aplicação.

## Checklist de revisão

- [ ] A fachada expõe um vocabulário do cliente, e não espelha a API do subsistema método a método?
- [ ] Ela só orquestra, sem regras de negócio escondidas?
- [ ] O tamanho está sob controle (ou foi dividida em fachadas refinadas)?
- [ ] O cliente não importa mais classes internas do subsistema?

## Padrões relacionados

- **Facade** define uma interface nova para um subsistema; **Adapter** torna usável uma interface existente, geralmente de um único objeto.
- **Abstract Factory** pode substituir a fachada quando o objetivo é só esconder como os objetos do subsistema são criados.
- **Flyweight** cria muitos objetos pequenos; Facade cria um objeto que representa um subsistema inteiro.
- **Mediator** também organiza colaboração entre classes acopladas, mas centraliza a comunicação (os componentes só conhecem o mediador). Na fachada, o subsistema não a conhece e seus objetos se falam diretamente.
- Uma fachada costuma poder ser **Singleton**.
- **Proxy** também envolve e inicializa uma entidade complexa, mas tem a mesma interface do serviço; a fachada não.
