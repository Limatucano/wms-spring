# Requisitos Funcionais

Sistema WMS (Warehouse Management System) generalista, para qualquer tipo de produto e segmento.

## Escopo e decisões

**Dentro do escopo:** recebimento, armazenagem, estoque, tarefas, separação, expedição, inventário, qualidade/rastreabilidade, devoluções, indicadores, segurança e integrações.

**Fora do escopo:** emissão fiscal (NF-e, CT-e, MDF-e), TMS (roteirização e gestão de frete), WCS (controle de equipamentos de automação), compras/MRP e financeiro.

**Decisões de projeto:**

| Decisão | Definição |
|---|---|
| Fiscal | O ERP é dono dos documentos fiscais. O WMS importa a NF-e de entrada e guarda número/chave da NF de saída, mas não emite. |
| Generalidade | Comportamento por **flags de produto** (lote, validade, série) e **estratégias configuráveis** (endereçamento, giro, picking), nunca por "tipo de segmento". |
| Estoque | Saldo é **derivado de um ledger de movimentações**. Nenhum fluxo edita saldo diretamente. |
| Multi-depositante (3PL) | Em aberto. Requisitos do módulo RF-14 são condicionais a essa decisão. |
| Canal de operação | Web (gestão) e dispositivo móvel/coletor (chão de armazém). Tecnologia a definir. |

**Legenda de prioridade:** `MVP` primeira entrega · `F2` segunda fase · `F3` terceira fase.

## Regras de negócio transversais

| ID | Regra |
|---|---|
| RN-01 | O saldo de estoque nunca é alterado diretamente; toda mudança nasce de uma movimentação registrada. |
| RN-02 | Toda movimentação registra origem, destino, quantidade, usuário, data/hora, motivo e documento de origem. |
| RN-03 | Saldo negativo é bloqueado por padrão (comportamento configurável por depósito). |
| RN-04 | Estoque em status bloqueado, quarentena, avaria ou vencido nunca é alocado a pedido. |
| RN-05 | Produto com flag de lote, validade ou série exige esses dados em toda transação relevante (recebimento, movimentação, separação, expedição). |
| RN-06 | A reserva de estoque acontece antes da geração da tarefa de separação. |
| RN-07 | A baixa definitiva do estoque só ocorre na confirmação de expedição. |
| RN-08 | O volume só é fechado quando a conferência de saída não tem divergência pendente. |
| RN-09 | Pedido não pode ser alterado ou cancelado abaixo da quantidade já separada. |
| RN-10 | Ajustes, descartes e divergências acima da tolerância exigem aprovação conforme alçada. |

## RF-01 Cadastros e estrutura do armazém

| ID | Requisito | Prio |
|---|---|---|
| RF-01.01 | Hierarquia configurável do armazém: depósito > zona > rua > módulo > nível > endereço | MVP |
| RF-01.02 | Tipos de endereço: picking, pulmão, doca, staging, quarentena, avaria, expedição | MVP |
| RF-01.03 | Capacidade e dimensões do endereço (peso, volume, nº de paletes/caixas) | MVP |
| RF-01.04 | Bloqueio e desbloqueio de endereço com motivo | MVP |
| RF-01.05 | Cadastro de produto/SKU: descrição, GTIN/EAN, família, dimensões e peso | MVP |
| RF-01.06 | Flags de controle por produto: lote, validade, nº de série, peso variável | MVP |
| RF-01.07 | Unidades de medida, conversões (unidade, caixa, palete) e múltiplos de embalagem | MVP |
| RF-01.08 | Parâmetros por produto: estratégia de giro, endereço fixo opcional, mín/máx de picking, shelf life mínimo | MVP |
| RF-01.09 | Restrições de armazenagem por produto (empilhamento, temperatura, perigoso, incompatibilidade) | F2 |
| RF-01.10 | Cadastro de parceiros: fornecedor, cliente e transportadora | MVP |
| RF-01.11 | Cadastro de motivos (avaria, ajuste, divergência, devolução, bloqueio, descarte) | MVP |
| RF-01.12 | Importação de cadastros em lote (CSV) com relatório de rejeições | F2 |

## RF-02 Recebimento

| ID | Requisito | Prio |
|---|---|---|
| RF-02.01 | Criar ordem de recebimento por importação de NF-e (XML), ASN ou cadastro manual | MVP |
| RF-02.02 | Conferência cega ou assistida por leitura de código de barras | MVP |
| RF-02.03 | Captura na conferência: quantidade por embalagem, lote, validade e série | MVP |
| RF-02.04 | Recebimento parcial, múltiplas conferências por ordem e cancelamento | MVP |
| RF-02.05 | Divergências (falta, sobra, avaria, produto trocado) com motivo, evidência opcional e aprovação | MVP |
| RF-02.06 | Geração de etiquetas de produto/volume com informações completas | MVP |
| RF-02.07 | Confirmação de recebimento enviada ao ERP | MVP |
| RF-02.08 | Validação de shelf life mínimo, validade inválida e lote duplicado | F2 |
| RF-02.09 | Tolerância configurável para recebimento acima do pedido | F2 |
| RF-02.10 | Agendamento de recebimento e gestão de docas (janelas e disponibilidade) | F2 |
| RF-02.11 | Portaria: registro de chegada, liberação e tempo de permanência do veículo | F2 |
| RF-02.12 | Geração de LPN (identificador de palete/volume) no recebimento | F2 |
| RF-02.13 | Envio automático a quarentena/inspeção conforme regra do produto ou fornecedor | F2 |

## RF-03 Armazenagem e endereçamento

| ID | Requisito | Prio |
|---|---|---|
| RF-03.01 | Tarefa de armazenagem gerada automaticamente ao fim da conferência | MVP |
| RF-03.02 | Sugestão de endereço por regras priorizadas e configuráveis (família, curva, capacidade, restrições, mesmo SKU/lote) | MVP |
| RF-03.03 | Validação de capacidade e compatibilidade; troca de endereço pelo operador com justificativa | MVP |
| RF-03.04 | Modos de endereçamento: fixo e dinâmico | MVP |
| RF-03.05 | Transferência interna entre endereços (manual ou por tarefa) | MVP |
| RF-03.06 | Mapa de ocupação por zona (cheio, parcial, vazio) | F2 |
| RF-03.07 | Transferência entre armazéns/filiais com status "em trânsito" | F2 |
| RF-03.08 | Consolidação e desfragmentação (agrupar paletes parciais do mesmo SKU/lote) | F3 |
| RF-03.09 | Recomendação de re-slotting por giro e curva ABC | F3 |
| RF-03.10 | Cross-docking: item recebido com demanda aberta vai direto para staging/expedição | F3 |

## RF-04 Gestão de estoque

| ID | Requisito | Prio |
|---|---|---|
| RF-04.01 | Saldo por produto, endereço, lote, validade, série e status | MVP |
| RF-04.02 | Ledger imutável de movimentações; saldo derivado (RN-01, RN-02) | MVP |
| RF-04.03 | Status de estoque: disponível, reservado, bloqueado, quarentena, avaria | MVP |
| RF-04.04 | Reserva/alocação por pedido, parcial ou total, reserva suave e reserva firme | MVP |
| RF-04.05 | Ajuste de estoque com motivo e alçada de aprovação | MVP |
| RF-04.06 | Consulta de estoque (por SKU, endereço, lote) e kardex | MVP |
| RF-04.07 | Alertas de vencimento e shelf life | F2 |
| RF-04.08 | Unidades logísticas hierárquicas (palete > caixa > unidade) com pack/unpack | F2 |
| RF-04.09 | Descarte/baixa (vencido, avariado) com aprovação | F2 |
| RF-04.10 | Relatório de divergência de saldo WMS × ERP | F2 |
| RF-04.11 | Kitting, desmontagem, retrabalho e reembalagem com rastreio de lote | F3 |

## RF-05 Gestão de tarefas

| ID | Requisito | Prio |
|---|---|---|
| RF-05.01 | Fila única de tarefas: armazenagem, movimentação, separação, ressuprimento, contagem, conferência | MVP |
| RF-05.02 | Prioridade por tarefa | MVP |
| RF-05.03 | Execução em dispositivo móvel com confirmação por leitura de endereço e produto | MVP |
| RF-05.04 | Pausa, retomada, reatribuição e cancelamento com trilha | MVP |
| RF-05.05 | Atribuição automática (Convocação Ativa) por perfil, disponibilidade, equipamento e zona; modo "pegar tarefa" | F2 |
| RF-05.06 | Prazo/SLA por tarefa com alerta de atraso | F2 |
| RF-05.07 | Intercalação de tarefas na mesma rota (ex.: armazenar e em seguida separar) | F3 |
| RF-05.08 | Cadastro de equipamentos e habilitação operador × equipamento | F3 |
| RF-05.09 | Padrões de produtividade e medição por operador e tipo de tarefa | F3 |

## RF-06 Pedidos e separação (picking)

| ID | Requisito | Prio |
|---|---|---|
| RF-06.01 | Receber pedido por API, arquivo ou cadastro manual (linhas, prioridade, transportadora, data limite) | MVP |
| RF-06.02 | Alocação conforme estratégia de giro: FIFO, FEFO, LIFO, LEFO | MVP |
| RF-06.03 | Tratamento de ruptura: corte, aguardar ressuprimento ou backorder | MVP |
| RF-06.04 | Separação por pedido | MVP |
| RF-06.05 | Separação com confirmação por leitura de endereço, produto, lote/série e quantidade | MVP |
| RF-06.06 | Alteração e cancelamento de pedido respeitando RN-09 | MVP |
| RF-06.07 | Restrições por cliente (lote, validade mínima restante) | F2 |
| RF-06.08 | Substituição de produto mediante aprovação | F2 |
| RF-06.09 | Separação por lote (batch), zona e cluster | F2 |
| RF-06.10 | Roteirização do picking (sequência de endereços de menor trajeto) | F2 |
| RF-06.11 | Ressuprimento do picking: mín/máx, sob demanda (por pedidos) e emergencial | F2 |
| RF-06.12 | Ondas por transportadora, rota, prazo ou prioridade; liberação contínua (sem onda) | F3 |
| RF-06.13 | Consolidação de pedidos do mesmo cliente e divisão em múltiplas remessas | F3 |

## RF-07 Conferência, embalagem e expedição

| ID | Requisito | Prio |
|---|---|---|
| RF-07.01 | Conferência de saída item a item, bloqueando divergência (RN-08) | MVP |
| RF-07.02 | Embalagem: criação de volumes, itens por volume e peso real | MVP |
| RF-07.03 | Etiquetas de volume/transporte e romaneio | MVP |
| RF-07.04 | Vínculo do número/chave da NF-e (emitida pelo ERP) ao pedido e aos volumes | MVP |
| RF-07.05 | Confirmação de expedição ao ERP e baixa definitiva do estoque (RN-07) | MVP |
| RF-07.06 | Staging de expedição por doca ou rota | F2 |
| RF-07.07 | Carregamento: conferência de volumes por veículo e fechamento de carga/manifesto | F2 |
| RF-07.08 | Agendamento de coleta/carregamento em doca de saída | F2 |
| RF-07.09 | Cubagem e sugestão de embalagem (cartonização) | F3 |
| RF-07.10 | Integração com transportadoras (cotação, etiqueta, rastreio) | F3 |

## RF-08 Inventário

| ID | Requisito | Prio |
|---|---|---|
| RF-08.01 | Inventário geral (depósito ou zona) com congelamento de movimentação nos endereços contados | MVP |
| RF-08.02 | Ajuste gerado a partir da contagem, mediante aprovação | MVP |
| RF-08.03 | Inventário cíclico: por endereço, produto, amostragem, curva ABC ou após movimentação/ajuste | F2 |
| RF-08.04 | Contagem cega com recontagem automática quando a divergência excede a tolerância | F2 |
| RF-08.05 | Indicador de acuracidade por endereço, produto e operador | F2 |
| RF-08.06 | Geração automática de contagens por regra e calendário | F3 |

## RF-09 Qualidade, lote e rastreabilidade

| ID | Requisito | Prio |
|---|---|---|
| RF-09.01 | Estados de lote: aprovado, quarentena, bloqueado, reprovado | F2 |
| RF-09.02 | Inspeção de entrada e saída (amostragem) com resultado e bloqueio automático | F2 |
| RF-09.03 | Notificação e tarefa de reconferência em caso de não conformidade | F2 |
| RF-09.04 | Rastreabilidade bidirecional por lote: origem (fornecedor/NF) e destino (clientes/NF) | F2 |
| RF-09.05 | Consulta de recall: dado um lote, listar saldo atual por endereço e expedições por cliente | F2 |
| RF-09.06 | Leitura de padrão GS1 (GTIN, lote AI 10, validade AI 17, série) em GS1-128 e DataMatrix | F2 |
| RF-09.07 | Rastreio por número de série (entrada, movimentação, saída) | F3 |

## RF-10 Devoluções e logística reversa

| ID | Requisito | Prio |
|---|---|---|
| RF-10.01 | Ordem de devolução (RMA) vinculada ao pedido/NF original, lote e série | F2 |
| RF-10.02 | Recebimento em quarentena, inspeção e destino: reestocar, descartar, retrabalhar ou devolver ao fornecedor | F2 |
| RF-10.03 | Retorno ao fornecedor (RTV) com trilha do lote | F3 |
| RF-10.04 | Indicadores de devolução por motivo, produto e lote | F3 |

## RF-11 Indicadores, relatórios e notificações

| ID | Requisito | Prio |
|---|---|---|
| RF-11.01 | Relatórios operacionais (estoque, movimentações, divergências, pendências) com exportação CSV/PDF | MVP |
| RF-11.02 | Dashboards em tempo real: pedidos por etapa, tarefas, ocupação e estoque (Gestão à Vista) | F2 |
| RF-11.03 | KPIs: acuracidade, OTIF, lead time de pedido, linhas/hora, taxa de erro, dock-to-stock, giro, ocupação | F2 |
| RF-11.04 | Notificações configuráveis por perfil (vencimento, ruptura, divergência, tarefa atrasada, endereço cheio) | F2 |
| RF-11.05 | Produtividade por operador e equipe | F3 |
| RF-11.06 | Agendamento e envio automático de relatórios | F3 |

## RF-12 Segurança, administração e auditoria

| ID | Requisito | Prio |
|---|---|---|
| RF-12.01 | Autenticação e gestão de usuários | MVP |
| RF-12.02 | Perfis e permissões (RBAC) por operação e por depósito | MVP |
| RF-12.03 | Trilha de auditoria (quem, quando, valor anterior e novo) em cadastros, ajustes, bloqueios e aprovações | MVP |
| RF-12.04 | Parametrização de regras (endereçamento, giro, tolerâncias) sem deploy | F2 |
| RF-12.05 | Alçadas e aprovações configuráveis (ajuste, divergência, descarte) | F2 |
| RF-12.06 | Segregação de funções (quem conta não ajusta; quem solicita não aprova) | F2 |
| RF-12.07 | Operação multi-depósito | F2 |

## RF-13 Integrações

| ID | Requisito | Prio |
|---|---|---|
| RF-13.01 | API REST versionada e documentada (OpenAPI) | MVP |
| RF-13.02 | ERP → WMS: produtos, parceiros, ordens de recebimento, pedidos e cancelamentos | MVP |
| RF-13.03 | WMS → ERP: confirmação de recebimento, status de pedido, confirmação de expedição, saldo e ajustes | MVP |
| RF-13.04 | Eventos/webhooks de mudança de status (ex.: `pedido.separado`, `estoque.ajustado`) | F2 |
| RF-13.05 | Importação e exportação por arquivo (CSV/XML) como alternativa à API | F2 |
| RF-13.06 | Monitor de integração: mensagens com falha, reprocessamento e idempotência | F2 |
| RF-13.07 | Adaptadores para automação (esteira, sorter, voice picking, pick-to-light) | F3 |

## RF-14 Multi-depositante / 3PL (condicional)

Só entra no escopo se o sistema atender operador logístico com vários donos de estoque.

| ID | Requisito | Prio |
|---|---|---|
| RF-14.01 | Isolamento lógico de estoque, endereços, regras e usuários por depositante, com espaço físico compartilhado | F3 |
| RF-14.02 | Parâmetros por depositante (etiquetas, documentos, estratégia de giro) | F3 |
| RF-14.03 | Tarifação por atividade (armazenagem por palete/dia, recebimento, separação, expedição, serviços adicionais) e geração de ordem de faturamento | F3 |
| RF-14.04 | Portal do cliente: estoque, pedidos, recebimentos e documentos | F3 |
| RF-14.05 | Modelo fiscal de armazém geral (NF de remessa/retorno de armazenagem) via integração com o ERP | F3 |

## Matriz de cobertura do fluxo

Cada etapa do ciclo do armazém tem entrada, saída e requisitos que a sustentam. Se uma etapa do fluxo principal (Cadastro até Expedição) ficar sem requisito MVP, o fluxo não fecha de ponta a ponta. Devolução fica para a F2 por decisão de escopo; no MVP, uma devolução entra como recebimento manual (RF-02.01) e a reintegração ao estoque como ajuste aprovado (RF-04.05).

| Etapa | Entrada | Saída | Requisitos |
|---|---|---|---|
| Cadastro | Dados mestres do ERP | Produto, endereço e parceiro prontos | RF-01, RF-13.02 |
| Recebimento | NF-e/ASN | Mercadoria conferida e etiquetada | RF-02, RF-13.03 |
| Armazenagem | Tarefa de armazenagem | Estoque endereçado e disponível | RF-03, RF-04 |
| Pedido | Pedido do ERP | Estoque reservado | RF-06.01 a RF-06.03, RF-04.04, RF-13.02 |
| Separação | Tarefa de separação | Itens separados e confirmados | RF-05, RF-06 |
| Conferência/embalagem | Itens separados | Volumes fechados e etiquetados | RF-07.01 a RF-07.03 |
| Expedição | Volumes + NF do ERP | Baixa definitiva e confirmação ao ERP | RF-07.04, RF-07.05, RF-13.03 |
| Inventário | Calendário/regra | Saldo ajustado e acuracidade medida | RF-08, RF-04.05 |
| Devolução | RMA | Estoque reintegrado ou descartado | RF-10 |
| Controle | Todas as etapas | Auditoria, KPIs, alertas e acesso controlado | RF-11, RF-12 |