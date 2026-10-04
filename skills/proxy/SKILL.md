---
name: proxy
description: Aplica o padrão Proxy (substituto). Use quando for preciso controlar o acesso a um objeto mantendo a mesma interface, com inicialização preguiçosa de objetos pesados (proxy virtual), controle de acesso/permissões (proxy de proteção), acesso a serviço remoto (proxy remoto), logging de chamadas, cache de resultados ou contagem de referências (referência inteligente). Também use para revisar proxies e diferenciá-los de Decorator, Adapter e Facade.
---

# Proxy

**Categoria:** estrutural · **Também conhecido como:** Substituto

## Propósito

Fornecer um substituto para outro objeto. O proxy implementa a mesma interface do serviço original, recebe os pedidos do cliente e faz algo antes ou depois de repassá-los, controlando o acesso ao objeto real.

## Quando usar

Os usos mais comuns:

- **Proxy virtual (inicialização preguiçosa):** um serviço pesado consome recursos mesmo quando raramente usado. O proxy só o cria quando ele for realmente necessário.
- **Proxy de proteção (controle de acesso):** só certos clientes podem usar o serviço. O proxy verifica credenciais antes de repassar.
- **Proxy remoto:** o serviço está em outro servidor. O proxy cuida dos detalhes de rede.
- **Proxy de log:** manter histórico dos pedidos ao serviço.
- **Proxy de cache:** guardar resultados de pedidos recorrentes (usando os parâmetros como chave) e gerenciar o ciclo de vida do cache.
- **Referência inteligente:** liberar um objeto pesado quando nenhum cliente o usa mais. O proxy rastreia quem tem referências e também pode detectar se o cliente modificou o serviço, permitindo reaproveitar os não modificados.

## Quando evitar

- O controle adicional não justifica uma camada. Uma chamada direta é mais simples e mais rápida.
- A latência extra introduzida pelo proxy é inaceitável.

## Como implementar

1. Se não houver uma interface de serviço, crie uma para que proxy e serviço sejam intercambiáveis. Se extrair a interface não for viável (exigiria mudar todos os clientes), o plano B é fazer o proxy **herdar** da classe de serviço.
2. Crie a classe proxy com um campo que referencia o serviço. Normalmente o proxy **cria e gerencia todo o ciclo de vida** do serviço; raramente o serviço é passado pelo cliente no construtor.
3. Implemente os métodos conforme o propósito do proxy e, na maioria dos casos, delegue ao serviço depois do trabalho extra.
4. Considere um método de criação (estático ou uma fábrica) que decide se o cliente recebe um proxy ou o serviço real.
5. Considere inicialização preguiçosa do serviço.

## Exemplo mínimo

```ts
interface BibliotecaYouTube {
  listarVideos(): Video[]
  infoDoVideo(id: string): Info
}

class APIYouTube implements BibliotecaYouTube { /* chamadas HTTP lentas */ }

class YouTubeComCache implements BibliotecaYouTube {
  private servico?: APIYouTube
  private lista?: Video[]
  private infos = new Map<string, Info>()

  private api() { return this.servico ??= new APIYouTube() }   // preguiçoso

  listarVideos() { return this.lista ??= this.api().listarVideos() }
  infoDoVideo(id: string) {
    if (!this.infos.has(id)) this.infos.set(id, this.api().infoDoVideo(id))
    return this.infos.get(id)!
  }
}

// o cliente não sabe se recebeu o proxy ou o serviço real
function tela(yt: BibliotecaYouTube) { yt.listarVideos() }
tela(new YouTubeComCache())
```

## Prós e contras

- ✅ Controla o serviço sem que os clientes saibam.
- ✅ Gerencia o ciclo de vida do serviço quando os clientes não se importam com isso.
- ✅ Funciona mesmo se o serviço ainda não estiver pronto ou disponível.
- ✅ Aberto/fechado: novos proxies sem mudar serviço nem clientes.
- ⚠️ Mais classes e mais complexidade.
- ⚠️ A resposta do serviço pode atrasar.

## Checklist de revisão

- [ ] O proxy tem exatamente a mesma interface do serviço?
- [ ] Está claro qual dos tipos de proxy é este (virtual, proteção, cache...)? Evite misturar vários propósitos numa só classe.
- [ ] Cache tem política de invalidação?
- [ ] O proxy é seguro com concorrência, se a inicialização for preguiçosa?

## Padrões relacionados

- **Adapter** oferece interface diferente; **Proxy**, a mesma; **Decorator**, uma interface ampliada.
- **Facade** também envolve e inicializa algo complexo, mas com interface própria; o proxy é intercambiável com o serviço.
- **Decorator** tem estrutura semelhante, mas o proxy costuma gerenciar o ciclo de vida do serviço sozinho, enquanto a composição de decoradores é controlada pelo cliente.
