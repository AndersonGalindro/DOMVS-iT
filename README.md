# Processamento de Proposta de Cartão

Este documento descreve o fluxo completo de processamento de uma proposta de cartão, incluindo validação de elegibilidade da oferta, seleção de recompensas, aplicação de benefícios e comunicação final com o cliente.

O objetivo desta documentação é facilitar o entendimento do processo por desenvolvedores, arquitetos de software e equipes de produto.

---

# Visão Geral do Processo

O fluxo de processamento da proposta segue as seguintes etapas:

1. Recebimento da proposta do cliente
2. Validação dos dados pessoais
3. Verificação de elegibilidade da oferta
4. Validação de benefícios disponíveis
5. Seleção do modelo de recompensa (cashback ou pontos)
6. Aplicação de benefícios extras
7. Criação da conta do cartão
8. Registro de eventos e histórico
9. Atualização do status da proposta
10. Comunicação com o cliente

---

# Regras de Elegibilidade das Ofertas

| Oferta | Regras |
|------|------|
Oferta A | Renda > R$ 1.000 |
Oferta B | Renda > R$ 15.000 **e** Investimentos > R$ 5.000 |
Oferta C | Renda > R$ 50.000 **e** Conta corrente > 2 anos |

---

# Benefícios por Oferta

| Benefício | Oferta A | Oferta B | Oferta C |
|------|------|------|------|
Cashback | ✔ | ✔ | ✔ |
Pontos | ✔ | ✔ | ✔ |
Seguro Viagem | ✖ | ✖ | ✔ |
Sala VIP | ✖ | ✔ | ✔ |

---

# Regras de Recompensa

O cliente pode escolher apenas **uma das opções abaixo**:

- Cashback
- Pontos

As opções são **mutuamente exclusivas**, ou seja, não é possível possuir cashback e pontos simultaneamente.

Caso seja selecionada uma opção inválida, o sistema deve registrar um erro de regra de negócio.

---

# Fluxo Arquitetural

Este diagrama representa como os serviços do sistema interagem durante o processamento da proposta.

```mermaid
flowchart LR

subgraph Customer
A[Enviar Proposta]
end

subgraph Proposal_Service
B[Receber Proposta]
C[Validar Dados Pessoais]
end

subgraph Eligibility_Service
D{Oferta Escolhida}
E{Renda > 1.000}
F{Renda > 15.000 e Investimentos > 5.000}
G{Renda > 50.000 e Conta > 2 anos}
end

subgraph Rewards_Service
H{Cashback ou Pontos}
I[Aplicar Cashback]
J[Aplicar Pontos]
end

subgraph Benefits_Service
K[Validar Benefícios Extras]
L[Seguro Viagem]
M[Sala VIP]
end

subgraph Card_Service
N[Criar Conta do Cartão]
end

subgraph Event_Service
O[Registrar Eventos e Histórico]
end

subgraph Notification_Service
P[Atualizar Status da Proposta]
Q[Comunicar Cliente]
end

A --> B
B --> C
C --> D

D --> E
D --> F
D --> G

E --> H
F --> H
G --> H

H --> I
H --> J

I --> K
J --> K

K --> L
L --> M
M --> N

N --> O
O --> P
P --> Q
```

---
# Fluxograma do Processo

```mermaid
flowchart TD

N1[1 Receber Proposta do Cliente] --> N2[2 Validar Dados Pessoais]

N2 --> N3{3 Oferta Escolhida}

N3 -->|Oferta A| N31{3.1 Renda > 1.000}
N31 -->|Não| N311[3.1.1 Reprovar Proposta]
N311 --> N11[11 Registrar Eventos e Histórico]
N31 -->|Sim| N312[3.1.2 Aprovar Proposta]
N312 --> N4

N3 -->|Oferta B| N32{3.2 Renda > 15.000 e Investimentos > 5.000}
N32 -->|Não| N321[3.2.1 Reprovar Proposta]
N321 --> N11
N32 -->|Sim| N322[3.2.2 Aprovar Proposta]
N322 --> N4

N3 -->|Oferta C| N33{3.3 Renda > 50.000 e Conta > 2 anos}
N33 -->|Não| N331[3.3.1 Reprovar Proposta]
N331 --> N11
N33 -->|Sim| N332[3.3.2 Aprovar Proposta]
N332 --> N4

N4[4 Validar Benefício] --> N5{5 Cashback ou Pontos}

N5 -->|Erro Regra| N51[5.1 Erro de regra de negócio]
N51 --> N11

N5 -->|Cashback| N6[6 Cashback]
N6 --> N61[6.1 Validar Benefícios Extras]
N61 --> N8

N5 -->|Pontos| N7[7 Pontos]
N7 --> N71[7.1 Validar Benefícios Extras]
N71 --> N8

N8[8 Seguro Viagem] --> N81{Oferta C}

N81 -->|Sim| N811[8.1.1 Ativar]
N811 --> N9

N81 -->|Não| N821[8.2.1 Ignorar]
N821 --> N9

N9[9 Sala VIP] --> N91{Oferta B ou Oferta C}

N91 -->|Sim| N911[9.1.1 Ativar]
N911 --> N10

N91 -->|Oferta A| N921[9.2.1 Ignorar]
N921 --> N10

N10[10 Criar Conta Cartão] --> N11

N11[11 Registrar Eventos e Histórico] --> N12[12 Atualizar Status da Proposta]

N12 --> N13[13 Comunicar Cliente]
```

---


# Eventos do Sistema

Durante o processamento da proposta, seria interessante colocar eventos para publicados em um sistema de mensageria como:

- RabbitMQ
- AWS SNS/SQS

---

# Objetivo da Documentação

Esta documentação tem como finalidade:

- Registrar as regras de negócio
- Descrever o fluxo de processamento da proposta
- Facilitar o entendimento da arquitetura do sistema
- Servir como referência para desenvolvedores e arquitetos envolvidos no projeto

---

# Diagrama no Figma

O fluxo completo também está disponível no Figma.

Clique para abrir o board:

[Visualizar no Figma](https://www.figma.com/board/Yp2tHEiu2sGynZSZ90qoI9/Sem-t%C3%ADtulo?node-id=0-1&t=YqZqsrqs3NWBpf5l-1)
