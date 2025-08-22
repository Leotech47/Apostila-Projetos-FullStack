O **diagrama de sequência** é um tipo de diagrama de interação da **UML** que mostra a ordem cronológica das interações entre os objetos (ou atores e componentes do sistema) em um cenário específico. Ele se concentra na sequência de mensagens trocadas no tempo para realizar uma tarefa.

-----

### Diagrama de Sequência: Registro de Venda

Este diagrama ilustra o fluxo de mensagens para o caso de uso principal do Vendedor: **"Registrar uma Venda"**.

```mermaid
sequenceDiagram
    title Fluxo: Vendedor Registra uma Nova Venda

    participant Vendedor
    participant Sistema
    participant ModuloEstoque

    Vendedor->>Sistema: Inicia o processo de venda
    activate Sistema
    Sistema-->>Vendedor: Exibe a interface de registro de venda
    
    Vendedor->>Sistema: Busca por um produto (ex: "Smartphone X")
    
    Sistema->>ModuloEstoque: Solicita informações e estoque do produto
    activate ModuloEstoque
    
    alt Produto Encontrado e Disponível
        ModuloEstoque-->>Sistema: Retorna dados do produto e quantidade
        deactivate ModuloEstoque
        Sistema-->>Vendedor: Exibe dados do produto para adicionar à venda
        
        Vendedor->>Sistema: Adiciona o produto ao carrinho
        Sistema-->>Vendedor: Confirma adição e exibe total parcial
        
        Vendedor->>Sistema: Confirma a venda e dados do cliente
        
        Sistema->>ModuloEstoque: Atualiza o estoque (decrementa a quantidade)
        activate ModuloEstoque
        ModuloEstoque-->>Sistema: Confirma a atualização do estoque
        deactivate ModuloEstoque
        
        Sistema-->>Vendedor: Confirmação de venda e gera recibo
        
    else Produto Não Encontrado ou Sem Estoque
        ModuloEstoque-->>Sistema: Retorna mensagem de erro ou indisponibilidade
        deactivate ModuloEstoque
        Sistema-->>Vendedor: Exibe mensagem de erro: "Produto não encontrado ou sem estoque."
    end
    
    deactivate Sistema
```

-----

### Explicação do Fluxo

Este diagrama descreve passo a passo a interação para registrar uma venda:

1.  O **Vendedor** inicia o processo de venda interagindo com o **Sistema**.
2.  O **Sistema** ativa-se e exibe a interface de registro de venda para o **Vendedor**.
3.  O **Vendedor** busca por um produto.
4.  O **Sistema** envia uma mensagem para o **Módulo de Estoque** solicitando as informações e a quantidade disponível do produto.
5.  O fluxo se divide em dois caminhos (`alt`):
      * **Caminho Principal (Produto Encontrado):** O **Módulo de Estoque** retorna os dados. O **Sistema** os exibe para o **Vendedor**, que adiciona o produto e confirma a venda. O **Sistema** então informa ao **Módulo de Estoque** para atualizar a quantidade, recebe a confirmação e finaliza o processo gerando um recibo para o **Vendedor**.
      * **Caminho Alternativo (Produto Não Encontrado):** O **Módulo de Estoque** retorna uma mensagem de erro. O **Sistema** a exibe para o **Vendedor**, encerrando o processo para essa tentativa.

Para visualizar este diagrama, você pode usar uma ferramenta compatível com a sintaxe [Mermaid](https://www.google.com/search?q=https://mermaid-js.github.io/mermaid/%23/), como o [Mermaid Live Editor](https://mermaid.live).
