# Requisitos Não Funcionais

Qualidades, limites e restrições do sistema WMS. Os valores numéricos são **metas iniciais** e devem ser calibrados com a carga real da operação e registrados em ADR quando mudarem.

**Legenda de prioridade:** `MVP` primeira entrega · `F2` segunda fase · `F3` terceira fase.

## Desempenho

| ID | Requisito | Verificação | Prio |
|---|---|---|---|
| RNF-01 | Confirmação de leitura no coletor (leitura até resposta) com p95 ≤ 1 s e p99 ≤ 2 s sob carga nominal | Teste de carga | MVP |
| RNF-02 | Telas operacionais e consultas de estoque com p95 ≤ 2 s, sempre paginadas | Teste de carga | MVP |
| RNF-03 | Alocação de pedido de até 100 linhas em ≤ 5 s | Teste de carga | F2 |
| RNF-04 | Relatórios e dashboards pesados usam réplica de leitura ou dados pré-agregados, sem degradar a operação | Teste de carga com consultas pesadas simultâneas | F2 |
| RNF-05 | Dimensionamento alvo documentado: nº de SKUs, endereços, linhas de pedido/dia e usuários simultâneos | ADR de capacidade | MVP |

## Consistência e integridade de dados

| ID | Requisito | Verificação | Prio |
|---|---|---|---|
| RNF-06 | Movimentação e atualização de saldo gravadas na mesma transação (ACID) | Teste: saldo = soma do ledger | MVP |
| RNF-07 | Operações idempotentes (chave de idempotência) na API e nas confirmações do coletor; reenvio não duplica movimento | Teste de reenvio | MVP |
| RNF-08 | Controle de concorrência na reserva: reservas simultâneas nunca excedem o saldo disponível | Teste de concorrência | MVP |
| RNF-09 | Ledger imutável: correção apenas por estorno ou ajuste, sem update/delete | Revisão de schema e permissões do banco | MVP |
| RNF-10 | Timestamps em UTC; exibição no fuso do depósito | Teste de integração | MVP |
| RNF-11 | Integridade referencial e constraints no banco para invariantes críticos (saldo não negativo, quantidades positivas) | Revisão de schema | MVP |

## Disponibilidade e resiliência

| ID | Requisito | Verificação | Prio |
|---|---|---|---|
| RNF-12 | Disponibilidade ≥ 99,5% no horário operacional | Monitoramento | F2 |
| RNF-13 | RPO ≤ 15 min e RTO ≤ 2 h; backups automáticos com teste de restauração periódico | Exercício de restauração | F2 |
| RNF-14 | Indisponibilidade do ERP não interrompe a operação: mensagens de saída ficam em fila e são reenviadas | Teste com ERP simulado fora do ar | F2 |
| RNF-15 | Integrações com retry exponencial, fila de mensagens com falha (dead-letter) e reprocessamento manual | Teste de falha injetada | F2 |
| RNF-16 | App do coletor tolera queda de rede: fila local de operações, sincronização e tratamento de conflito | Teste em modo avião | F3 |

## Escalabilidade

| ID | Requisito | Verificação | Prio |
|---|---|---|---|
| RNF-17 | API sem estado de sessão no servidor, escalável horizontalmente | Teste com múltiplas instâncias | F2 |
| RNF-18 | Tabelas de movimentação e auditoria com particionamento ou arquivamento por data e índices alinhados às consultas críticas | Plano de execução das consultas | F2 |
| RNF-19 | Isolamento por depositante sem duplicar a base (multi-tenant lógico) quando RF-14 estiver ativo | Teste de isolamento | F3 |

## Segurança

| ID | Requisito | Verificação | Prio |
|---|---|---|---|
| RNF-20 | Autenticação com token de expiração curta e renovação; senhas com hash forte (Argon2/bcrypt) | Revisão de segurança | MVP |
| RNF-21 | Autorização RBAC validada no backend em toda rota, nunca apenas no cliente | Testes de autorização por perfil | MVP |
| RNF-22 | TLS 1.2+ em trânsito; criptografia em repouso para dados sensíveis | Revisão de infraestrutura | MVP |
| RNF-23 | Mitigação do OWASP Top 10: validação de entrada, limite de requisições, proteção contra injeção | Análise estática e testes | MVP |
| RNF-24 | Segredos e chaves de integração fora do código, com rotação | Varredura de segredos no CI | MVP |
| RNF-25 | Isolamento de dados por depósito/depositante aplicado na camada de dados, não só na interface | Teste de acesso cruzado | F3 |

## Privacidade e conformidade

| ID | Requisito | Verificação | Prio |
|---|---|---|---|
| RNF-26 | LGPD: coleta mínima de dados pessoais (operadores, motoristas, clientes), retenção definida, exportação e anonimização sob demanda | Inventário de dados pessoais | F2 |
| RNF-27 | Para segmentos regulados (ANVISA, alimentos): trilha de auditoria, segregação de funções, controle de mudanças e rastreabilidade de lote disponíveis para validação | Checklist de conformidade | F3 |

## Auditabilidade e observabilidade

| ID | Requisito | Verificação | Prio |
|---|---|---|---|
| RNF-28 | Trilha de auditoria imutável e consultável, com retenção mínima configurável | Teste de consulta e retenção | MVP |
| RNF-29 | Logs estruturados (JSON) sem dados sensíveis | Revisão de logs | MVP |
| RNF-30 | Health checks de liveness e readiness | Teste automatizado | MVP |
| RNF-31 | ID de correlação propagado entre API, filas e chamadas ao ERP | Rastreio de uma requisição ponta a ponta | F2 |
| RNF-32 | Métricas (latência, erros, tamanho de filas, tarefas pendentes) e alertas | Painel e alertas configurados | F2 |
| RNF-33 | Tracing distribuído | Traces visíveis ponta a ponta | F3 |

## Usabilidade e operação no chão de armazém

| ID | Requisito | Verificação | Prio |
|---|---|---|---|
| RNF-34 | Fluxos do coletor orientados à leitura (scan-first), com no máximo 3 toques por confirmação | Teste de usabilidade com operadores | MVP |
| RNF-35 | Leitura de produto ou endereço errado bloqueia a ação com feedback visual e sonoro/vibratório | Teste funcional | MVP |
| RNF-36 | Interface do coletor com fontes grandes, alto contraste e alvos de toque amplos (uso com luvas) | Teste em campo | MVP |
| RNF-37 | Leitores suportados: câmera do celular e scanner nativo de coletor; simbologias EAN-13, Code 128, GS1-128, QR e DataMatrix | Matriz de dispositivos testada | MVP |
| RNF-38 | Impressão térmica (ZPL) com templates de etiqueta configuráveis | Teste com impressora | MVP |
| RNF-39 | Telas administrativas com acessibilidade WCAG 2.1 AA | Auditoria de acessibilidade | F3 |
| RNF-40 | pt-BR como padrão; estrutura preparada para outros idiomas, formatos de data/número e unidades | Revisão de i18n | F3 |

## Interoperabilidade

| ID | Requisito | Verificação | Prio |
|---|---|---|---|
| RNF-41 | API REST documentada em OpenAPI, versionada (`/v1`), com compatibilidade retroativa dentro da versão | Contract tests no CI | MVP |
| RNF-42 | Formatos padrão: JSON, CSV, XML de NF-e e codificação UTF-8 | Testes de importação | MVP |
| RNF-43 | Testes de contrato com o ERP e demais consumidores | Pipeline de contract tests | F2 |

## Manutenibilidade e testabilidade

| ID | Requisito | Verificação | Prio |
|---|---|---|---|
| RNF-44 | Monólito modular com fronteiras claras por módulo (recebimento, estoque, tarefas, pedidos); um módulo não acessa tabelas de outro | Teste de arquitetura (ex.: ArchUnit) | MVP |
| RNF-45 | Cobertura mínima de 80% nas regras de domínio (estoque, alocação, endereçamento), com testes de concorrência | Relatório de cobertura no CI | MVP |
| RNF-46 | CI com build, testes, análise estática e verificação de vulnerabilidades em dependências | Pipeline verde obrigatório para merge | MVP |
| RNF-47 | Migrations de banco versionadas (Flyway ou Liquibase) | Execução em ambiente limpo no CI | MVP |
| RNF-48 | Documentação viva: README, ADRs das decisões e glossário de domínio | Revisão em cada PR relevante | MVP |

## Configurabilidade

| ID | Requisito | Verificação | Prio |
|---|---|---|---|
| RNF-49 | Regras parametrizáveis (endereçamento, giro, tolerâncias, alçadas) com versionamento e auditoria das mudanças | Teste de alteração de regra sem deploy | F2 |
| RNF-50 | Feature flags por módulo e por depositante | Teste de ativação/desativação | F3 |

## Implantação e operação

| ID | Requisito | Verificação | Prio |
|---|---|---|---|
| RNF-51 | Aplicação containerizada, configuração por variáveis de ambiente e ambientes separados (dev, homologação, produção) | Deploy reproduzível | MVP |
| RNF-52 | Carga inicial de cadastros e saldo com validação e relatório de rejeições | Carga de teste com dados inválidos | F2 |
| RNF-53 | Deploy sem indisponibilidade (rolling ou blue-green) | Deploy em horário de operação simulado | F3 |
| RNF-54 | Matriz de dispositivos, versões de SO e navegadores suportados, definida e testada | Documento + testes | MVP |