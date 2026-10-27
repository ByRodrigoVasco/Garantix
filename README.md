<p align="center"><img src="docs/marca/logo.png" alt="Garantix" width="420"></p>

# Garantix: Gestão de Garantias e Assistência Técnica

O **Garantix** é uma aplicação Salesforce para **gerenciar garantias, inspeções técnicas e substituição de peças durante o ciclo de atendimento**.

**Ciclo de atendimento:**

```
Garantia ──► Solicitação de Garantia ──► Vistoria Técnica ──► Peças Substituídas / Laudo Técnico
                     │                                                    │
                     └────────── Aprovação em 2 níveis ◄──────────────────┘
                                         │
                     Histórico da Garantia (registrado automaticamente em cada etapa)
                     Endereço do atendimento (completado pela API ViaCEP)
```

1. O cliente possui uma **Garantia** (Fábrica, Estendida ou Premium) com período de vigência.
2. Quando o produto apresenta problema, é aberta uma **Solicitação de Garantia** (Defeito, Recall ou Acidente).
3. Um técnico realiza a **Vistoria Técnica**, registra as **Peças Substituídas** e emite o **Laudo Técnico**.
4. A solicitação passa por **aprovação**: Analista e, quando necessário, Supervisor Técnico.
5. Cada etapa gera um evento no **Histórico da Garantia**. O endereço do atendimento é preenchido pela **API ViaCEP**, com log de auditoria de cada consulta.

## Sumário

1. [Como instalar e executar](#1-como-instalar-e-executar)
2. [Como utilizar (fluxo do dia a dia)](#2-como-utilizar-fluxo-do-dia-a-dia)
3. [Como utilizar cada recurso (critérios de entrega)](#3-como-utilizar-cada-recurso-critérios-de-entrega)
4. [Estrutura do repositório](#4-estrutura-do-repositório)
5. [Manutenção e solução de problemas](#5-manutenção-e-solução-de-problemas)

---

## 1. Como instalar e executar

### 1.1 Pré-requisitos

| Ferramenta | Para que serve | Como instalar |
|---|---|---|
| **Org Salesforce** (Trailhead Playground ou Developer Edition) | Ambiente onde o Garantix roda | Ver passo 1.2 |
| **Node.js LTS** | Necessário para o Salesforce CLI | https://nodejs.org |
| **Salesforce CLI** (`sf`) | Envia os arquivos para a org | `npm install -g @salesforce/cli` |
| **Git** | Baixar o repositório | https://git-scm.com |
| VS Code + *Salesforce Extension Pack* (opcional) | Editar o código | Marketplace do VS Code |

Confira a instalação no terminal:

```bash
sf --version
git --version
```

### 1.2 Criar a org

1. Acesse **trailhead.salesforce.com** e faça login.
2. Clique na foto do perfil → **Organizações práticas** (Hands-On Orgs) → **Criar Playground**. Aguarde alguns minutos.
3. Abra o Playground → no app **Playground Starter**, aba **Get Your Login Credentials**, anote o **Username** e clique em **Reset My Password** para definir uma senha pelo e-mail.

> Também funciona com uma Developer Edition criada em developer.salesforce.com/signup.

### 1.3 Baixar o projeto e conectar à org

```bash
git clone <url-deste-repositorio>
cd <pasta-do-repositorio>
sf org login web --alias garantix --set-default
```

O navegador abre a tela de login do Salesforce: entre com o usuário e a senha da org e clique em **Allow**.

### 1.4 Implantar (deploy)

```bash
sf project deploy start --source-dir force-app --wait 30
```

O resultado esperado é `Status: Succeeded`. Esse comando cria na org todos os objetos, campos, perfis, flows, a aprovação, o código Apex, os componentes, o app, os relatórios e os dashboards.

### 1.5 Rodar os testes Apex

```bash
sf apex run test --test-level RunLocalTests --code-coverage --result-format human --wait 15
```

Resultado esperado: **16 testes, 100% de aprovação, cobertura da org de 99%**.

### 1.6 Criar usuários e dados de demonstração

```bash
sf apex run --file scripts/apex/1_configurar_usuarios.apex
sf apex run --file scripts/apex/2_dados_teste.apex
```

| Script | O que faz |
|---|---|
| `1_configurar_usuarios.apex` | Coloca o administrador no papel **Diretor** e cria 3 usuários de teste: **Olivia Operadora** (Perfil Operacional), **Caio Coordenador** (Perfil Gestor) e **Diana Diretora** (Perfil Administrativo) |
| `2_dados_teste.apex` | Cria 4 garantias, 6 solicitações, vistorias, peças, 1 laudo, 2 endereços com CEP real (consultados no ViaCEP) e envia 2 solicitações para aprovação |

> Rode cada script **uma única vez** por org. O script 1 usa licenças *Salesforce Platform*; se a org não tiver licenças livres, o log mostra `ERRO` para o usuário que não pôde ser criado.

### 1.7 Abrir a aplicação

Na org, clique no **App Launcher** (ícone de 9 pontos, canto superior esquerdo), digite **Garantix** e abra o app.

---

## 2. Como utilizar (fluxo do dia a dia)

Roteiro completo de um atendimento, do cadastro à aprovação:

| # | Ação | Onde | O que acontece automaticamente |
|---|---|---|---|
| 1 | **Cadastrar a garantia** | Aba **Garantias** → **New** → escolher o tipo (Fábrica, Estendida ou Premium) → preencher Cliente, Produto, Data de Início e Data de Fim | Calcula **Dias Restantes**; o card **Status da Garantia** mostra a situação com cor |
| 2 | **Abrir uma solicitação** | Na garantia, botão **Abrir Solicitação** → escolher tipo e prioridade, descrever o problema, informar o custo estimado e, opcionalmente, o CEP e o número | Cria a Solicitação com status **Aberta**; se houver CEP, cria o **Endereço** e consulta o ViaCEP; registra "Solicitação aberta" no **Histórico** |
| 3 | **Agendar a vistoria** | Na solicitação, lista **Vistorias Técnicas** → **New** → Data Agendada, Técnico, Status **Pendente** | A vistoria entra no indicador **Vistorias Pendentes** |
| 4 | **Registrar as peças trocadas** | Lista **Peças Substituídas** → **New** → Nome, Código, Quantidade e Custo | Atualiza **Quantidade de Peças** e **Custo Total das Peças** na solicitação |
| 5 | **Concluir a vistoria** | Editar a vistoria → Status **Concluída** | A solicitação passa sozinha para **Em Análise**; o histórico registra a mudança |
| 6 | **Emitir o laudo** | Lista **Laudos Técnicos** → **New** → Conclusão (Procedente ou Improcedente) e Parecer | |
| 7 | **Enviar para aprovação** | Na solicitação, botão **Submit for Approval** (aparece só em **Em Análise**) | Status vira **Aguardando Aprovação**; a solicitação vai para a fila do Analista |
| 8 | **Aprovar** | Sino de notificações ou lista **Approval History** da solicitação → **Approve** ou **Reject** | Nível 1 (Analista) sempre. Se custo > R$ 2.000 ou tipo Acidente, segue para o nível 2 (Supervisor Técnico). Ao final, status **Aprovada** ou **Rejeitada** |
| 9 | **Acompanhar** | Aba **Dashboards** → *Garantix - Executivo* e *Garantix - Operacional* → **Refresh** | Indicadores atualizados com os dados do dia |

---

## 3. Como utilizar cada recurso (critérios de entrega)

Cada item abaixo indica **o que foi implementado**, **onde ver na org** e **como testar**.

### 3.1 Dados

#### Objetos customizados (8)

| Objeto | API Name | Função |
|---|---|---|
| Garantia | `Garantia__c` | Cobertura de um produto de um cliente |
| Solicitação de Garantia | `Solicitacao_Garantia__c` | Chamado aberto quando o produto apresenta problema |
| Vistoria Técnica | `Vistoria_Tecnica__c` | Inspeção feita pelo técnico |
| Laudo Técnico | `Laudo_Tecnico__c` | Parecer final da solicitação |
| Peça Substituída | `Peca_Substituida__c` | Peças trocadas e seus custos |
| Histórico da Garantia | `Historico_Garantia__c` | Linha do tempo dos eventos |
| Endereço | `Endereco__c` | Local do atendimento (CEP, Logradouro, Número, Complemento, Bairro, Cidade, Estado) |
| Log de Integração | `Log_Integracao__c` | Auditoria das chamadas ao ViaCEP |

- **Onde ver:** Setup → **Object Manager** → pesquisar o objeto. Visão gráfica: Setup → **Schema Builder** → marcar os 8 objetos.
- **Como testar:** usar as abas do app Garantix e a lista **Todos** de cada objeto.

#### Relacionamentos Master-Detail e Lookup

```
Garantia
 ├─ Solicitação de Garantia ........ Lookup (obrigatório)
 │    ├─ Vistoria Técnica ........... Master-Detail
 │    ├─ Peça Substituída ........... Master-Detail
 │    ├─ Laudo Técnico .............. Lookup
 │    └─ Endereço ................... Lookup
 │          └─ Log de Integração .... Lookup
 └─ Histórico da Garantia ........... Lookup (também ligado à Solicitação)
```

- **Por que assim:** Vistoria e Peça são Master-Detail para permitir os **Roll-Ups** na Solicitação. A Solicitação é Lookup da Garantia para ter **dono e compartilhamento próprios** (OWD Privado, sharing rule e aprovação).
- **Onde ver:** Object Manager → objeto → **Fields & Relationships** (coluna *Data Type*).
- **Como testar:** apagar uma solicitação de teste apaga junto as vistorias e peças (Master-Detail); tentar apagar uma garantia com solicitações é bloqueado (Lookup com restrição).

#### Campos fórmula

| Campo | Objeto | Regra |
|---|---|---|
| **Dias Restantes da Garantia** | Garantia | `Data de Fim - hoje` (0 quando vencida) |
| **Elegível para Cobertura** | Solicitação | Garantia **Ativa**, solicitação aberta **dentro da vigência** e, se for **Acidente**, garantia **Premium** |
| Prazo SLA | Solicitação | Data de abertura + 2 dias (Alta), 5 (Média) ou 10 (Baixa) |
| Situação do SLA | Solicitação | *No prazo*, *Atrasada* ou *Encerrada* |

- **Como testar:** com os dados de demonstração, compare **SOL-0004** (Acidente em garantia Fábrica = **não elegível**) com **SOL-0001** (Acidente em garantia Premium = **elegível**). A **SOL-0003** (prioridade Alta aberta há 6 dias) aparece como **Atrasada**.

#### Roll-Up Summary

| Campo (na Solicitação) | Cálculo |
|---|---|
| **Quantidade de Peças Substituídas** | Contagem das Peças |
| **Quantidade de Vistorias** | Contagem das Vistorias |
| Custo Total das Peças | Soma do custo das Peças |

- **Como testar:** abra a **SOL-0000** (2 peças, R$ 710). Adicione uma peça e veja os totais mudarem na hora.

#### Record Types

- **Garantia:** Fábrica, Estendida, Premium.
- **Solicitação:** Defeito, Recall, Acidente.
- **Onde ver:** Object Manager → objeto → **Record Types**.
- **Como testar:** ao clicar em **New** na aba Garantias ou Solicitações, o Salesforce pede para escolher o tipo.

#### Page Layouts específicos por perfil

| Objeto | Layout *Operacional* | Layout *Gestão* |
|---|---|---|
| Garantia | Tipo somente leitura | Permite alterar o tipo e o proprietário |
| Solicitação | Sem a seção financeira | Com a seção **Financeiro e Cobertura** |

- **Atribuição:** Perfil Operacional usa o *Operacional*; Perfil Gestor e Perfil Administrativo usam o *Gestão*.
- **Onde ver:** Object Manager → objeto → **Page Layouts** → **Page Layout Assignment**.
- **Como testar:** faça **Login As** com a Olivia (Perfil Operacional) e clique em **New** em Solicitações: os campos financeiros não aparecem. Na página do registro, a seção **Financeiro e Cobertura** também fica oculta para esse perfil (regra do Dynamic Forms).

### 3.2 Segurança

| Recurso | Implementação | Onde ver |
|---|---|---|
| **Perfis** | Perfil Operacional (lê garantias; cria solicitações, vistorias, peças e laudos), Perfil Gestor (cria e edita garantias, sem acesso aos logs), Perfil Administrativo (vê e altera tudo) | Setup → **Profiles** |
| **Papéis** | Diretor, Coordenador, Operador | Setup → **Roles** |
| **Role Hierarchy** | Diretor → Coordenador → Operador (o superior vê os registros dos subordinados) | Setup → **Roles** (visão em árvore) |
| **Padrão da organização (OWD)** | Solicitação **Private**; Garantia, Laudo, Histórico e Log **Public Read Only**; Vistoria e Peça **Controlled by Parent**; Endereço **Public Read/Write** | Setup → **Sharing Settings** |
| **Sharing Rule por critério** | Solicitações com Prioridade **Alta** compartilhadas (leitura) com o papel **Operador** | Setup → **Sharing Settings** → *Solicitação de Garantia Sharing Rules* |
| **Permission Set** | **Auditor de Integração**: acesso de leitura a todos os Logs de Integração e Endereços, sem trocar o perfil | Setup → **Permission Sets** |

**Como testar:**
1. Setup → **Users** → ao lado de **Olivia Operadora**, clique em **Login**. (Se o link não aparecer: Setup → *Login Access Policies* → habilitar *Administrators Can Log in as Any User*.)
2. Como Olivia: ela vê as solicitações de prioridade **Alta** (sharing rule), não vê a aba **Logs de Integração** (perfil) e não vê a seção financeira (layout e Dynamic Forms).
3. Saia (*Log out* no topo), atribua o permission set **Auditor de Integração** à Olivia (Setup → Users → Olivia → *Permission Set Assignments* → *Edit Assignments*) e entre novamente: a aba de Logs passa a funcionar.

### 3.3 Automação

#### Flows

| Flow | Tipo | Disparo | Ação |
|---|---|---|---|
| **Garantix - Registrar Histórico (Subflow)** | Autolaunched (subflow) | Chamado por outros flows | Cria um registro no Histórico da Garantia |
| **Garantix - Solicitação Criada** | Record-Triggered (após salvar) | Criação de Solicitação | Chama o subflow: "Solicitação aberta" |
| **Garantix - Status da Solicitação Alterado** | Record-Triggered (após salvar) | Mudança no campo Status | Chama o subflow: "Status alterado de X para Y" |
| **Garantix - Vistoria Concluída** | Record-Triggered (após salvar) | Vistoria passa a Concluída | Muda a Solicitação para **Em Análise** |
| **Garantix - Abrir Solicitação (Screen Flow)** | Screen Flow | Botão **Abrir Solicitação** na Garantia | Assistente que cria a Solicitação e, se informado o CEP, o Endereço |

**Encadeamento (flows encadeados):** Vistoria concluída → Flow 3 muda o status → Flow 2 detecta a mudança → Subflow grava o histórico.

- **Onde ver:** Setup → **Flows** → abrir o flow para ver o desenho no **Flow Builder**. Nos flows *Solicitação Criada* e *Status da Solicitação Alterado* aparece o elemento **Subflow**.
- **Como testar:** siga os passos 2 e 5 do [fluxo do dia a dia](#2-como-utilizar-fluxo-do-dia-a-dia) e acompanhe o card **Histórico** na página da solicitação (botão de atualizar no canto do card).

#### Processo de aprovação (2 níveis)

| Nível | Aprovador | Quando entra |
|---|---|---|
| 1. **Analista** | Fila *Fila Analistas* (papel Coordenador) | Sempre |
| 2. **Supervisor Técnico** | Fila *Fila Supervisores Técnicos* (papel Diretor) | **Custo Estimado > R$ 2.000** (valor) **ou** tipo **Acidente** (categoria). Caso contrário, a solicitação é aprovada após o nível 1 |

- **Entrada:** Status = *Em Análise*. Ao enviar: *Aguardando Aprovação*. Ao aprovar: *Aprovada*. Ao rejeitar: *Rejeitada*. Ao retirar (recall): volta para *Em Análise*.
- **Onde ver:** Setup → **Approval Processes** → objeto *Solicitação de Garantia* → *Aprovação de Solicitação de Garantia*.
- **Como testar:** nos dados de demonstração, **SOL-0002** (Recall, R$ 300) foi aprovada só no nível 1; **SOL-0001** (Acidente, R$ 3.200) aguarda o nível 2. Abra a SOL-0001 → lista **Approval History** → **Approve**.

### 3.4 Apex e integração ViaCEP

| Componente | Arquivo | Função |
|---|---|---|
| **Trigger** | `EnderecoTrigger.trigger` | Uma linha: delega ao handler |
| **Trigger Framework** | `TriggerHandler.cls` | Classe base: identifica o evento (before/after insert/update) e chama o método certo; permite desligar um handler (*bypass*) |
| Handler | `EnderecoTriggerHandler.cls` | Marca o endereço como *Pendente* e enfileira a consulta quando o CEP é informado ou alterado |
| **Callout** | `ViaCepService.cls` | `GET https://viacep.com.br/ws/{cep}/json/`; trata CEP inválido, CEP inexistente, erro HTTP e falha de rede |
| **Queueable** | `ViaCepQueueable.cls` | Processamento assíncrono: consulta, preenche o endereço e grava o **Log de Integração** (até 50 por execução, encadeando o restante) |
| Liberação de acesso | Remote Site Setting **ViaCEP** | Autoriza chamadas a `https://viacep.com.br` |
| **Testes** | `*Test.cls` + `ViaCepHttpMock.cls` | 16 testes com respostas simuladas da API |

**Fluxo:** salvar Endereço com CEP → Trigger → Handler → Queueable (em segundo plano) → ViaCEP → campos preenchidos + Log de Integração.

> Callouts não são permitidos dentro de triggers. Por isso a consulta é feita de forma assíncrona, pelo Queueable.

- **Onde ver:** Setup → **Apex Classes** / **Apex Triggers**; execuções em Setup → **Apex Jobs**; logs na aba **Logs de Integração**.
- **Como testar a integração:** aba **Endereços** → **New** → CEP `01310-100` → **Save**. Aguarde alguns segundos e recarregue a página: Logradouro, Bairro, Cidade e Estado são preenchidos e o **Status da Integração** fica *Sucesso*. A lista relacionada **Logs de Integração** mostra o código HTTP 200 e a resposta JSON. Com o CEP `99999-999` o status fica *Erro* e o log registra "CEP não encontrado".
- **Como testar a cobertura:** comando do passo [1.5](#15-rodar-os-testes-apex), ou Setup → **Apex Test Execution** → *Select Tests* → todas as classes → **Run**.

### 3.5 Interface

| Recurso | Implementação | Como ver e testar |
|---|---|---|
| **Lightning App** | App **Garantix** com logo, cor da marca e abas do processo | App Launcher → *Garantix*. Edição: Setup → **App Manager** |
| **Dynamic Forms** | Páginas *Garantia - Página de Registro* e *Solicitação - Página de Registro* com campos em seções e regras de visibilidade | Seção **"Atenção: garantia vencendo"** aparece só com Dias Restantes ≤ 30 e Status Ativa (veja **GAR-0001**). Seção **"Aprovação em andamento"** aparece só em *Aguardando Aprovação*. Seção **"Financeiro e Cobertura"** fica oculta para o Perfil Operacional |
| **Dynamic Actions** | Botões com regra de exibição | **Abrir Solicitação** só aparece em garantias **Ativas** (compare GAR-0001 com a GAR-0003, expirada). **Submit for Approval** só aparece com status **Em Análise** |
| **LWC Status da Garantia** (`garantiaStatusCard`) | Card na página da Garantia | Mostra dias restantes com cor (verde > 90, laranja 31 a 90, vermelho ≤ 30, cinza vencida) e a barra de % da vigência usada |
| **LWC Linha do Tempo** (`historicoTimeline`) | Card nas páginas de Garantia e Solicitação | Lista os eventos do Histórico; botão de atualizar no canto do card |

- **Onde editar as páginas:** abra um registro → engrenagem ⚙ → **Edit Page** (Lightning App Builder). Clique numa seção ou botão para ver a regra em *Set Component Visibility*.

### 3.6 Analytics

| Dashboard | Indicador | Relatório de origem |
|---|---|---|
| **Garantix - Executivo** | Custo de garantia (total e por tipo) | Custo de Garantia por Tipo |
| | Índice de aprovação | Índice de Aprovação |
| | Taxa de cobertura | Taxa de Cobertura |
| **Garantix - Operacional** | Solicitações abertas por prioridade | Solicitações Abertas |
| | SLA (no prazo x atrasadas) | SLA das Solicitações |
| | Vistorias pendentes por data | Vistorias Pendentes |

- **Onde ver:** aba **Dashboards** → pasta *Garantix Dashboards*; relatórios na aba **Reports** → pasta *Garantix Relatórios*.
- **Como usar:** abra o dashboard e clique em **Refresh** para atualizar os números. Os dashboards são dinâmicos: cada usuário vê os dados que tem permissão de acessar.

### 3.7 Marca

- Nome comercial: **Garantix**.
- Logomarca: escudo (proteção/garantia) com check verde (problema resolvido). Arquivos em `docs/marca/` (PNG e SVG). Cores: azul `#0B3D91`, azul `#1F6FEB`, verde `#2BD47D`.
- Aplicada no app (logo e cor do cabeçalho).

### 3.8 Repositório

Todo o projeto (configurações e código) está neste repositório, no formato **Salesforce DX**, e pode ser implantado em qualquer org com os comandos da [seção 1](#1-como-instalar-e-executar).

---

## 4. Estrutura do repositório

```
force-app/main/default/
  objects/               objetos, campos, record types, list views, validações, layouts compactos
  layouts/               page layouts (Operacional e Gestão)
  profiles/              perfis
  roles/                 papéis (hierarquia)
  sharingRules/          regra de compartilhamento por critério
  permissionsets/        permission set Auditor de Integração
  flows/                 subflow, record-triggered flows e screen flow
  quickActions/          botão "Abrir Solicitação"
  approvalProcesses/     processo de aprovação
  workflows/             atualizações de status usadas na aprovação
  queues/                filas de aprovadores
  classes/               Apex (framework, serviço ViaCEP, queueable, controller, testes)
  triggers/              trigger do Endereço
  remoteSiteSettings/    liberação do endereço do ViaCEP
  lwc/                   Lightning Web Components
  applications/          app Lightning Garantix
  flexipages/            páginas Lightning (Dynamic Forms e Dynamic Actions)
  tabs/                  abas dos objetos
  contentassets/         logo usado no app
  reports/               relatórios
  dashboards/            dashboards
scripts/apex/            usuários de teste e dados de demonstração
docs/marca/              logomarca
sfdx-project.json        configuração do projeto Salesforce DX
```

---

## 5. Manutenção e solução de problemas

### Alterações feitas pela tela

Ajustes feitos pelo navegador (Setup, Flow Builder, Lightning App Builder) ficam só na org. Para trazê-los para o repositório antes do commit:

```bash
sf project retrieve start --source-dir force-app
```

Itens **novos** criados pela tela precisam ser trazidos pelo tipo, por exemplo:

```bash
sf project retrieve start --metadata CustomField:Garantia__c.Novo_Campo__c
```

> Rode o `retrieve` antes de qualquer `deploy`: o deploy sobrescreve na org o que estiver diferente nos arquivos.

### Problemas comuns

| Sintoma | Causa provável | Solução |
|---|---|---|
| `sf` não é reconhecido | CLI não instalado ou terminal aberto antes da instalação | Instalar o Node.js e o CLI e reabrir o terminal |
| Erro de login no `sf org login web` | Senha da org não definida | Fazer o *Reset My Password* (passo 1.2) |
| Endereço fica em *Pendente* | A consulta é assíncrona | Aguardar alguns segundos e recarregar; ver Setup → **Apex Jobs** |
| Endereço com *Erro* | CEP inexistente ou ViaCEP indisponível | Ver a mensagem no **Log de Integração**; corrigir o CEP e salvar (uma nova consulta é feita) |
| Dashboard sem dados | Dados ainda não carregados ou cache | Rodar o script de dados (1.6) e clicar em **Refresh** |
| Abas sem registros | Visualização *Recently Viewed* | Trocar para a lista **Todos** |
| Script de usuários com `ERRO` | Falta de licenças *Salesforce Platform* na org | Ver Setup → **Company Information** → *User Licenses* |
