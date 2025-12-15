# Documento de Visão

Documento construído à parte do **Documento de requisitos do sistema: Controle de Compras e Vendas de Cosméticos** que pode ser encontrado no link: https://docs.google.com/document/d/1d2V91BRv_6ZYkVmo_FXOfOeWm__Vs_nRf30rpTrJWyw/edit?tab=t.0

## Descrição do Projeto

Este projeto trata-se de um sistema de informação de controle de compra e venda de cosméticos para revendedores autônomos que busca automatizar as vendas dos produtos disponíveis, proporcionando uma gestão eficiente e automatizando o gerenciamento de clientes, produtos, vendas e estoque.

## Diagrama de class

```mermaid
classDiagram
    class Cliente {
        -codigo : int
        -nome : string
        -cpf : string
        -telefone : int
        -email : string
        -endereco : string
        +cadastrar_cliente(c : Cliente) void
        +excluir_cliente(c : Cliente) void
        +alterar_cliente(c : Cliente) void
        +consultar_cliente(cpf : string) void
    }

    class Venda {
        -codigo_venda : int
        -data_venda : date
        +realizar_venda(v : Venda) void
        +excluir_venda(v : Venda) void
        +alterar_venda(v : Venda) void
        +consultar_venda(codigo_venda : int) void
        +finalizar_venda(v : Venda) void
    }

    class ItensVenda {
        -quantidade_produto : int
        -preco_total : double
        +cadastrar_item(i : ItensVenda) void
        +excluir_item(i : ItensVenda) void
        +alterar_item(i : ItensVenda) void
        +consultar_item(cod_venda : int) void
    }

    class Produto {
        -codigo_produto : int
        -nome_produto : string
        -validade_produto : date
        -preco_produto : double
        -preco_revenda : double
        +cadastrar_produto(p : Produto) void
        +excluir_produto(p : Produto) void
        +alterar_produto(p : Produto) void
        +consultar_produto(codigo_produto : int) void
    }

    class Marca {
        -codigo_marca : int
        -nome_marca : string
        -cnpj : string
        -telefone : int
        -email : string
        +cadastrar_marca(m : Marca) void
        +excluir_marca(m : Marca) void
        +alterar_marca(m : Marca) void
        +consultar_marca(codigo_marca : int) void
    }

    class Cobranca {
        -codigo_cobranca : int
        -valor_cobranca : double
        -data_cobranca : date
        -data_vencimento : date
        -nome_cliente : string
        +gerar_cobranca(v : Venda) void
        +excluir_cobranca(c : Cobranca) void
        +alterar_cobranca(c : Cobranca) void
        +consultar_cobranca(codigo_cobranca : int) void
    }

    class Pagamento {
        -codigo_pagamento : int
        -preco_pagamento : double
        -forma_pagamento : string
        -data_pagamento : date
        +registrar_pagamento(p : Pagamento) void
        +excluir_pagamento(p : Pagamento) void
        +alterar_pagamento(p : Pagamento) void
        +consultar_pagamento(codigo_pagamento : int) void
    }

    Cliente "1" --> "0..*" Venda 
    Venda "1" --> "1..*" ItensVenda 
    ItensVenda "0..*" --> "1" Produto 
    Produto "0..1" --> "1" Marca 
    Venda "1" --> "0..1" Cobranca 
    Cobranca "1" --> "1..*" Pagamento 
```

## Diagrama de Sequência – Manter cliente – Incluir

```mermaid
sequenceDiagram
    autonumber

    actor Gerente as Gerente
    participant IU as Interface com Usuário (Cliente)
    participant Cliente as Cliente

    opt Verificar se o cliente já existe no sistema
        rect rgb(255,230,200)
            IU ->> IU: Manter cliente - consultar
        end
    end

    Gerente ->> IU: Cadastrar Cliente()

    IU ->> Cliente: cadastrar_cliente(nome, endereço, telefone, cpf, email)

    alt Campos preenchidos
        Cliente -->> IU: Cliente cadastrado com sucesso()
    else Campos não preenchidos
        Cliente -->> IU: Exibir mensagem de cliente não cadastrado()
    end
```

## Diagrama de Sequência – Manter cliente – Alterar

```mermaid
sequenceDiagram
    actor Gerente
    participant IU as Interface com Usuário (Cliente)
    participant Cliente

    opt Verificar se o Cliente já existe no sistema
        IU ->> IU: Consultar cliente
    end

    Gerente ->> IU: Escolher informação do cliente para alterar()
    IU ->> Cliente: alterar_cliente(cliente)

    alt Campos válidos
        Cliente -->> IU: Cliente alterado com sucesso()
    else Campos inválidos
        Cliente -->> IU: Exibir mensagem para preencher campos em branco
    end
```

## Diagrama de Sequência – Manter cliente – Consultar

```mermaid
sequenceDiagram
    autonumber

    actor Gerente
    participant IU as Interface com Usuário (Cliente)
    participant Cliente as Cliente

    Gerente ->> IU: ConsultarCliente()
    IU ->> Cliente: consultar_cliente(cpf, nome)

    alt Cliente Existe
        Cliente -->> IU: Informações do Cliente
    else Cliente não existe
        Cliente -->> IU: Mensagem de cliente inexistente
    end
```

## Diagrama de Sequência – Manter cliente – Excluir

```mermaid
sequenceDiagram
    autonumber

    actor Gerente
    participant IU as Interface com Usuário (Cliente)
    participant Cobranca as Cobrança
    participant Cliente as Cliente

    Gerente ->> IU: Escolher o cliente e deletar()
    IU ->> Cliente: excluir_cliente(nome, cpf)

    opt Verificar Se o Cliente Já Existe no Sistema
        IU ->> Cliente: consultar_cliente()
        Cliente -->> IU: dados do cliente
    end

    opt Verificar cobrança
        IU ->> Cobranca: verificar_cobranca()
        Cobranca -->> IU: status da cobrança
    end

    alt Pagamento iniciado
        IU -->> Gerente: Não foi possível excluir...
    else Pagamentos não iniciados
        loop enquanto houver cobrança
            IU ->> Cobranca: excluir_cobranca()
            Cobranca -->> IU: cobrança excluída
        end
    end

    IU -->> Gerente: Cliente excluído com sucesso
```

## Diagrama de Sequência – Realizar Venda

```mermaid
sequenceDiagram
    actor Gerente
    participant IU as Interface_Usuario
    participant Cliente
    participant Venda
    participant ItemVenda
    participant Produto
    participant Cobranca

    Gerente ->> IU: 1. solicitarVenda(cpf, itens)

    rect rgb(255,200,120)
        note over IU,ItemVenda: Manter cliente - consultar
    end

    loop Enquanto houver itens a adicionar na venda
        IU ->> Venda: 2. adicionar_produto(produto)

        rect rgb(255,255,180)
            note over Venda,Produto: Manter produto - consultar
        end

        Venda ->> ItemVenda: 2.1 cadastrar_item(item_venda:Item_venda)

        alt Se item adicionado
            ItemVenda -->> Venda: item adicionado com sucesso
            Venda -->> IU: itens adicionados com sucesso
        else Se item não adicionado
            ItemVenda -->> Venda: 3. item inexistente
            Venda -->> IU: 4. item inexistente
        end
    end

    Gerente ->> IU: 5. confirmar venda()
    IU ->> Venda: 6. cadastrar_venda(venda)

    Venda ->> Cobranca: 6.1 gerar_cobranca(cobranca)

    alt Se gerar cobrança e cadastrar venda
        Cobranca -->> Venda: cobrança gerada com sucesso
        Venda -->> IU: venda cadastrada com sucesso
    else Se não gerar cobrança
        Cobranca -->> Venda: 7. erro ao gerar cobrança
    end
```


## Requisitos Funcionais

| Código | Descrição do requisito             | Prioridade | Tempo estimado | Tempo real | Tamanho funcional | Analista         | Desenvolvedor | Revisor | Testador |
|--------|------------------------------------|------------|----------------|------------|-------------------|------------------|---------------|---------|----------|
| RF01   | Cadastro de produtos               | Essencial  | 8h             |            |                   | Vitória Geovanna |               |         |          |
| RF02   | Cadastro de marcas                 | Desejável  | 8h             |            |                   | Vitória Geovanna |               |         |          |
| RF03   | Cadastro de vendas                 | Essencial  | 8h             |            |                   | Vitória Geovanna |               |         |          |
| RF04   | Cadastro de clientes               | Importante | 9h             |            |                   | Vitória Geovanna |               |         |          |
| RF05   | Realizar compras de mercadorias    | Essencial  | 18h            |            |                   | Vitória Geovanna |               |         |          |
| RF06   | Manter o estoque atualizado        | Importante | 24h            |            |                   | Vitória Geovanna |               |         |          |
| RF07   | Controle de compras                | Essencial  | 29h            |            |                   | Vitória Geovanna |               |         |          |
| RF08   | Controle de vendas                 | Essencial  | 29h            |            |                   | Vitória Geovanna |               |         |          |
| RF09   | Controle de produtos               | Essencial  | 29h            |            |                   | Vitória Geovanna |               |         |          |
| RF10   | Geração de relatórios              | Desejável  | 15h            |            |                   | Vitória Geovanna |               |         |          |

## Testes de Aceitação

### RF01 – Cadastro de Produtos

| Código    | Descrição                                                                                                       |
|-----------|-----------------------------------------------------------------------------------------------------------------|
| TA01.01   | Cadastro de produtos bem-sucedidos com todos os dados preenchidos.                                              |
| TA01.02   | Tentativa de cadastro com campos vazios retorna erro.                                                           |
| TA01.03   | Tentativa de cadastro com código do produto duplicado retorna erro.                                             |
| TA01.04   | Exibe uma mensagem de confirmação após o cadastro ser concluído.                                                |

### RF02 – Cadastro de Marcas

| Código    | Descrição                                                                                                       |
|-----------|-----------------------------------------------------------------------------------------------------------------|
| TA02.01   | Cadastro de marcas bem-sucedidas com todos os dados preenchidos.                                                |
| TA02.02   | Tentativa de cadastro com campos vazios retorna erro.                                                           |
| TA02.03   | Tentativa de cadastro com CNPJ da marca duplicada retorna erro.                                                 |
| TA02.04   | Exibe uma mensagem de confirmação após o cadastro ser concluído.                                                |

### RF03 – Cadastro de Vendas

| Código    | Descrição                                                                                                       |
|-----------|-----------------------------------------------------------------------------------------------------------------|
| TA03.01   | Cadastro de vendas bem-sucedidas com todos os dados preenchidos.                                                |
| TA03.02   | Tentativa de cadastro com campos vazios retorna erro.                                                           |
| TA03.03   | Tentativa de cadastro com id da venda duplicada retorna erro.                                                   |
| TA03.04   | Exibe uma mensagem de confirmação após o cadastro ser concluído.                                                |

### RF04 - Cadastro de clientes

| Código    | Descrição                                                                                                       |
|-----------|-----------------------------------------------------------------------------------------------------------------|
| TA04.01   | Cadastro de clientes bem-sucedidos com todos os dados preenchidos.                                              |
| TA04.02   | Tentativa com campos vazios retorna erro.                                                                       |
| TA04.03   | Tentativa de cadastro com CPF do cliente duplicado retorna erro.                                                |
| TA04.04   | Tentativa de cadastro com CPF do cliente inválido retorna erro.                                                 |
| TA04.05   | Exibe uma mensagem de confirmação após o cadastro ser concluído.                                                |

### RF05 - Realizar compra de mercadorias

| Código    | Descrição                                                                                                       |
|-----------|-----------------------------------------------------------------------------------------------------------------|
| TA05.01   | Realização de compras bem-sucedidas com todos os dados preenchidos.                                             |
| TA05.02   | Tentativa com campos vazios retorna erro.                                                                       |
| TA05.03   | Tentativa de compra de mercadoria com estoque, o sistema informa a quantidade em estoque.                       |
| TA05.04   | Exibe uma mensagem de confirmação após a realização ser concluída.                                              |

### RF06 - Manter o estoque atualizando

| Código    | Descrição                                                                                                       |
|-----------|-----------------------------------------------------------------------------------------------------------------|
| TA06.01   | Realização do estoque bem-sucedido com todos os dados atualizados.                                              |
| TA06.02   | Tentativa com campos vazios retorna erro.                                                                       |
| TA06.03   | Não permite a desatualização do estoque.                                                                        |
| TA06.04   | Exibe uma mensagem de confirmação após a realização ser concluída.                                              |

### RF07 - Controle de compras

| Código    | Descrição                                                                                                       |
|-----------|-----------------------------------------------------------------------------------------------------------------|
| TA07.01   | Realização do controle das compras bem-sucedidas com todos os dados atualizados.                                |
| TA07.02   | Tentativa com campos vazios retorna erro.                                                                       |
| TA07.03   | Não permite a desatualização das compras.                                                                       |
| TA07.04   | Exibe uma mensagem de confirmação após a realização ser concluída.                                              |

### RF08 – Controle de Vendas

| Código    | Descrição                                                                                                       |
|-----------|-----------------------------------------------------------------------------------------------------------------|
| TA08.01   | Realização do controle das vendas bem-sucedidas com todos os dados atualizados.                                 |
| TA08.02   | Tentativa com campos vazios retorna erro.                                                                       |
| TA08.03   | Não permite a desatualização das vendas.                                                                        |
| TA08.04   | Exibe uma mensagem de confirmação após a realização ser concluída.                                              |
| TA08.05   | Tentativa de venda sem estoque retorna erro.                                                                    |
| TA08.06   | Tentativa de venda para clientes com débitos pendentes retorna erro.                                            |

### RF09 – Controle de Produtos (Estoque)

| Código    | Descrição                                                                                                       |
|-----------|-----------------------------------------------------------------------------------------------------------------|
| TA09.01   | Realização do controle dos produtos bem-sucedidos com todos os dados atualizados.                               |
| TA09.02   | Tentativa com campos vazios retorna erro.                                                                       |
| TA09.03   | Não permite a desatualização dos produtos.                                                                      |
| TA09.04   | Exibe uma mensagem de confirmação após a operação ser concluída.                                                |

### RF10 – Gerar Relatórios

| Código    | Descrição                                                                                                       |
|-----------|-----------------------------------------------------------------------------------------------------------------|
| TA10.01   | Realização de relatórios de produtos, marcas, vendas e clientes bem-sucedidos com todos os dados atualizados.   |
| TA10.02   | Tentativa com campos vazios retorna erro.                                                                       |
| TA10.03   | Não permite a desatualização dos relatórios de produtos, marcas, vendas e clientes.                             |
| TA10.04   | Exibe uma mensagem de confirmação após a realização ser concluída.                                              |
