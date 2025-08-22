### Diagrama de Sequência: Registro de Venda

Este diagrama ilustra o fluxo de mensagens para registrar uma venda no sistema.

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
