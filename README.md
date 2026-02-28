# Documentação Técnica
## Plataforma de Entrevistas Online com Feedback Inteligente — CIEE
**Ticket de origem:** PK-28240 | **Versão:** 1.0.0 | **Data:** Fevereiro/2026

---

# 1. Visão Geral do Projeto

## 1.1 Introdução e Contexto de Negócio

O CIEE (Centro de Integração Empresa-Escola) é uma das maiores organizações de intermediação de estágios e aprendizagem profissional do Brasil, conectando anualmente centenas de milhares de estudantes a empresas parceiras. O processo seletivo tradicional envolve triagem de currículos, entrevistas presenciais ou telefônicas e devolutivas manuais, gerando gargalos operacionais significativos, inconsistência na qualidade do feedback e dificuldade de escala.

A **Plataforma de Entrevistas Online com Feedback Inteligente** nasce da necessidade de modernizar e digitalizar completamente o ciclo de entrevistas e avaliações, incorporando inteligência artificial para automatizar tarefas repetitivas, padronizar a qualidade do feedback e gerar insights acionáveis para estudantes, empresas e gestores do CIEE.

## 1.2 Problema que o Sistema Resolve

| Problema | Impacto Atual |
|---|---|
| Feedback pós-entrevista inconsistente ou ausente | Estudantes não sabem como melhorar; taxa de rejeição silenciosa alta |
| Agendamento manual e descentralizado | Perda de vagas por falhas de comunicação |
| Ausência de dados estruturados sobre entrevistas | Impossibilidade de análise de padrões e melhoria contínua |
| Dificuldade de escala geográfica | Empresas em outros estados não conseguem entrevistar estudantes locais |
| Tempo elevado de retorno de feedback | Média de 5–10 dias úteis para devolutiva; candidato já aceitou outra vaga |
| Falta de ferramenta de desenvolvimento contínuo | Nenhum mecanismo formal de feedback entre gestores e estagiários no dia a dia |

## 1.3 Objetivos Gerais e Específicos

**Objetivo Geral:** Criar uma plataforma integrada que permita a realização de entrevistas online, a geração automatizada de feedback por IA e o acompanhamento contínuo do desenvolvimento de estudantes e colaboradores no ecossistema CIEE.

**Objetivos Específicos:**
- Reduzir o tempo médio de retorno de feedback de entrevistas de 5–10 dias para menos de 2 horas (automático) ou 24 horas (com revisão humana).
- Aumentar a taxa de feedbacks fornecidos de ~40% para >90% dos processos seletivos.
- Permitir entrevistas remotas com qualidade equivalente ao presencial, utilizando videochamada P2P com baixa latência.
- Oferecer transcrição automática e análise semântica do conteúdo das entrevistas, preservando a privacidade dos dados conforme a LGPD.
- Criar um ciclo de feedback contínuo (360°) entre estudantes, empresas, gestores e colaboradores CIEE.
- Gerar planos de desenvolvimento individualizados (PDIs) automaticamente baseados em análises de desempenho.
- Disponibilizar um dashboard preditivo para gestores identificarem riscos de evasão e oportunidades de desenvolvimento.

## 1.4 Público-Alvo e Stakeholders

| Perfil | Papel no Sistema | Necessidades Primárias |
|---|---|---|
| **Estudante/Candidato** | Realiza entrevistas, recebe e emite feedbacks | Feedback claro e acionável; acesso ao próprio histórico de desenvolvimento |
| **Empresa Parceira (Recrutador)** | Agenda e conduz entrevistas, avalia candidatos | Agilidade no processo seletivo; relatórios padronizados; conformidade legal |
| **Gestor CIEE** | Supervisiona processos, acessa dashboards | Visibilidade macro dos processos; KPIs; alertas de risco |
| **Colaborador CIEE** | Recebe feedbacks de gestores; participa de PDIs | Transparência no desenvolvimento profissional; planos claros de crescimento |
| **Administrador da Plataforma** | Configura o sistema, gerencia tenants | Controle de acesso, auditoria, configurações de IA |

## 1.5 Escopo do Projeto

**Dentro do Escopo:**
- Agendamento e realização de videochamadas P2P com sinalização via WebSocket
- Gravação de entrevistas com consentimento explícito e armazenamento seguro
- Transcrição automática com Whisper
- Geração de feedback estruturado via LLM (Mistral 3B / Claude API)
- Plataforma de feedback bidirecional (candidato ↔ empresa, colaborador ↔ gestor)
- Análise de sentimento de feedbacks submetidos
- Geração de Planos de Desenvolvimento Individual (PDI)
- Assistente de redação de feedback (IA)
- Dashboard preditivo para gestores
- Multi-tenancy (separação de dados por empresa/unidade CIEE)
- Conformidade com LGPD

**Fora do Escopo (v1.0):**
- Integração com ATS (Applicant Tracking Systems) de terceiros
- Módulo de testes técnicos/codificação ao vivo
- Análise de linguagem não-verbal (expressão facial, postura) — previsto para v2.0
- Aplicativo mobile nativo (Android/iOS)
- Integração com folha de pagamento ou sistemas de RH legados
- Suporte a múltiplos idiomas (v1.0 apenas Português Brasileiro)

## 1.6 Premissas e Restrições

**Premissas:**
- Todos os usuários têm acesso à internet banda larga (mínimo 1 Mbps para videochamada)
- A infraestrutura inicial será hospedada on-premises ou em nuvem privada, com possibilidade de migração para cloud pública
- Os modelos de IA (Whisper, Mistral 3B) podem ser executados localmente; Claude API é consumida via internet
- O sistema deve suportar múltiplas unidades do CIEE com isolamento de dados

**Restrições:**
- Orçamento inicial limita o uso de APIs pagas (Claude API) — necessidade de rate limiting inteligente e fallback para Mistral 3B local
- As gravações de entrevistas devem ser armazenadas por no mínimo 6 meses (conformidade interna) e excluídas após 2 anos ou mediante solicitação do titular
- O sistema deve ser acessível via navegador (Chrome/Firefox/Edge/Safari) sem instalação de plugins

---

> ⚠️ **Pontos de Atenção — Seção 1**
> - A definição de "colaborador CIEE" vs "estudante" deve ser formalizada no contrato de desenvolvimento para evitar ambiguidade de escopo.
> - A restrição de uso de API externa (Claude) exige que o rate limiting seja implementado desde o MVP para evitar custos inesperados.
> - O escopo de análise de linguagem não-verbal é altamente demandado pelo negócio — registrar formalmente como fora do escopo v1.0 e endereçar no roadmap.

---

# 2. Arquitetura do Sistema

## 2.1 Diagrama Textual da Arquitetura

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          CLIENTES (Browser)                              │
│  ┌──────────────┐   ┌──────────────┐   ┌──────────────┐                │
│  │  Candidato   │   │  Recrutador  │   │  Gestor CIEE │                │
│  └──────┬───────┘   └──────┬───────┘   └──────┬───────┘                │
│         │ Angular + PrimeNG + TailwindCSS       │                        │
└─────────┼──────────────────┼───────────────────┼────────────────────────┘
          │ HTTPS/WSS        │ HTTPS/WSS          │ HTTPS
          ▼                  ▼                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                          API GATEWAY / NGINX                             │
│              (TLS Termination, Rate Limiting, Auth Forward)              │
└───────────────────────────────┬─────────────────────────────────────────┘
                                │
        ┌───────────────────────┼───────────────────────┐
        ▼                       ▼                       ▼
┌───────────────┐   ┌───────────────────┐   ┌───────────────────┐
│  Auth Service │   │  Interview Service│   │  Feedback Service │
│  (Spring Boot)│   │  (Spring Boot)    │   │  (Spring Boot)    │
│  JWT + OAuth2 │   │  Módulo 1         │   │  Módulo 2         │
└───────┬───────┘   └────────┬──────────┘   └────────┬──────────┘
        │                    │                        │
        └────────────────────┼────────────────────────┘
                             │
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
        ┌──────────┐  ┌──────────┐  ┌──────────────┐
        │PostgreSQL│  │  Redis   │  │  RabbitMQ    │
        │(Multi-   │  │  Cache   │  │  (Filas de   │
        │ tenant)  │  │  + Sess. │  │   eventos)   │
        └──────────┘  └──────────┘  └──────┬───────┘
                                           │
              ┌────────────────────────────┼───────────────────┐
              ▼                            ▼                   ▼
        ┌──────────────┐         ┌──────────────┐    ┌────────────────┐
        │  AI Worker   │         │  Signaling   │    │  Storage Svc   │
        │  (Whisper +  │         │  Server      │    │  (Gravações)   │
        │  Mistral 3B) │         │  (PeerJS/WS) │    │  S3-compatible │
        └──────┬───────┘         └──────────────┘    └────────────────┘
               │
        ┌──────┴───────┐
        ▼              ▼
 ┌────────────┐  ┌────────────┐
 │ Mistral 3B │  │ Claude API │
 │  (Local)   │  │ (Anthropic)│
 └────────────┘  └────────────┘

         ◄── P2P WebRTC (PeerJS) entre Candidato e Recrutador ──►
              (negociado via Signaling Server, dados diretos P2P)
```

## 2.2 Descrição Detalhada de Cada Camada

### Camada de Apresentação (Frontend)
- **Framework:** Angular 17+ (standalone components, signals)
- **UI Components:** PrimeNG para componentes ricos (tabelas, calendários, modais); TailwindCSS para layout e customização
- **Comunicação em tempo real:** WebSocket (SockJS + STOMP) para sinalização de chamadas e notificações; PeerJS encapsulando WebRTC para a stream de vídeo/áudio P2P
- **Estado:** NgRx para gerenciamento de estado global; RxJS para streams reativos
- **Responsividade:** Layout responsivo para desktop e tablets (mobile não prioritário em v1.0)

### Camada de Aplicação (Backend)
- **Runtime:** Java 21 com Spring Boot 3.2+
- **Interview Service:** Gerencia agendamentos, sessões de entrevista, consentimento, integração com fila de processamento de IA
- **Feedback Service:** Gerencia submissão, análise, planos de desenvolvimento e dashboard
- **Auth Service:** Autenticação via JWT (access token 15min + refresh token 7 dias); suporte a OAuth2 para empresas parceiras
- **Comunicação interna:** REST síncrono entre serviços para operações críticas; RabbitMQ para operações assíncronas (processamento de IA, notificações)

### Camada de IA (AI Worker)
- Serviço Python (FastAPI) que consome mensagens do RabbitMQ
- Orquestra o pipeline: áudio → Whisper → transcrição → Mistral 3B / Claude API → feedback estruturado
- Gerencia o fallback entre modelos e o rate limiting da API Claude

### Camada de Dados
- **PostgreSQL:** Banco relacional principal com schema por tenant (estratégia schema-per-tenant)
- **Redis:** Cache de sessões, tokens de autenticação, resultados de IA recentes, filas de rate limiting
- **Armazenamento de mídia:** Servidor de arquivos S3-compatible (MinIO on-premises ou AWS S3 em produção)

## 2.3 Comunicação entre os Módulos 1 e 2

Os módulos compartilham o banco de dados (schemas isolados por tenant) e se comunicam via RabbitMQ para eventos assíncronos e via REST interno para consultas síncronas.

**Fluxo de integração típico:**
1. Módulo 1 finaliza processamento de entrevista e publica evento `interview.completed` no RabbitMQ com payload contendo `interviewId`, `transcription`, `feedbackDraft`
2. Módulo 2 consome o evento e cria automaticamente um registro de feedback no sistema de feedback contínuo, vinculando ao candidato e à empresa
3. O gestor CIEE vê no dashboard (Módulo 2) as métricas agregadas de todas as entrevistas processadas (Módulo 1)
4. O PDI gerado pelo Módulo 2 pode referenciar pontos identificados nas transcrições do Módulo 1

## 2.4 Padrões Arquiteturais Utilizados

**Event Sourcing:** Todos os eventos relevantes (agendamento criado, entrevista iniciada, gravação consentida, feedback submetido, PDI gerado) são armazenados como eventos imutáveis em uma tabela `domain_events`. O estado atual é derivado pela projeção desses eventos. Isso garante auditoria completa e possibilidade de replayar eventos.

**Multi-tenancy (Schema-per-Tenant):** Cada empresa parceira ou unidade CIEE possui seu próprio schema no PostgreSQL (ex: `tenant_ciee_sp`, `tenant_empresa_xyz`). O tenant é identificado via subdomínio ou header `X-Tenant-ID`. A camada de repositório injeta dinamicamente o schema correto usando Spring's `AbstractRoutingDataSource`.

**P2P com WebRTC:** A stream de vídeo/áudio viaja diretamente entre os navegadores dos participantes, reduzindo latência e custos de infraestrutura. O servidor de sinalização (PeerJS) apenas negocia a conexão inicial (troca de SDP e ICE candidates) e não processa mídia.

**CQRS Parcial:** Operações de escrita passam pelos serviços de domínio; operações de leitura para dashboard usam views materializadas no PostgreSQL, atualizadas por eventos do RabbitMQ.

## 2.5 Estratégias de Fallback e Resiliência

| Componente | Falha | Estratégia de Fallback |
|---|---|---|
| Claude API | Timeout / rate limit excedido | Redirecionar para Mistral 3B local; sinalizar feedback como "gerado por modelo alternativo" |
| Mistral 3B | Serviço indisponível | Gerar feedback template baseado em regras fixas; notificar administrador |
| Whisper | Falha na transcrição | Notificar entrevistador para inserção manual; manter gravação para retry posterior |
| PeerJS (P2P) | Falha na conexão direta | Fallback automático para relay TURN; se TURN falhar, oferecer chamada via telefone |
| RabbitMQ | Fila indisponível | Retry com exponential backoff (3 tentativas); persistir em dead-letter queue |
| Redis | Cache miss | Carregar dados do PostgreSQL; reconstruir cache |
| Servidor de sinalização | Indisponível | Health check a cada 30s; restart automático via systemd/k8s |

## 2.6 Justificativa para Cada Escolha Tecnológica

| Tecnologia | Justificativa |
|---|---|
| **Angular** | Padrão já adotado pelo time CIEE; tipagem forte com TypeScript; ecossistema maduro para aplicações empresariais |
| **PeerJS/WebRTC** | Elimina custo de servidor de mídia (alternativa: Jitsi/Mediasoup custam 10x mais em infra); latência menor; sem plugin |
| **Spring Boot** | Ecossistema Java maduro, time já familiarizado, excelente suporte a multi-tenancy e integração com RabbitMQ |
| **RabbitMQ** | Processamento de IA é custoso e assíncrono; fila garante que nenhum resultado seja perdido mesmo sob carga |
| **Redis** | Rate limiting em memória com alta performance; sessões distribuídas; Lua scripts para operações atômicas |
| **Whisper** | Melhor custo-benefício para ASR em português; pode ser executado localmente (privacidade); modelo `medium` equilibra velocidade e precisão |
| **Mistral 3B** | Modelo open-source que roda localmente, eliminando custo de API para feedbacks simples; boa performance em português |
| **Claude API** | Qualidade superior para feedback complexo e assistente de redação; usado apenas quando Mistral não é suficiente |
| **PostgreSQL** | ACID compliant; excelente suporte a schemas múltiplos (multi-tenancy); JSONB para dados semi-estruturados de IA |
| **PrimeNG + TailwindCSS** | PrimeNG oferece componentes enterprise prontos; Tailwind permite customização rápida e consistente |

---

> ⚠️ **Pontos de Atenção — Seção 2**
> - A estratégia schema-per-tenant escala bem até ~100 tenants; acima disso, avaliar migração para row-level security (RLS) com discriminador de tenant.
> - O fallback TURN para WebRTC requer servidor TURN dedicado (ex: Coturn) — este custo de infraestrutura deve ser previsto desde o início.
> - O AI Worker em Python (FastAPI) introduz uma linguagem adicional na stack; garantir que o time tenha capacidade de manutenção ou encapsular como serviço com interface bem definida.

---

# 3. Especificação dos Módulos

## 3.1 Módulo 1 — Videochamada com Feedback por IA

### Fluxo Detalhado

**Fase 1: Agendamento**
1. Recrutador acessa o painel de vagas e seleciona candidato aprovado na triagem
2. Sistema verifica disponibilidade do candidato via calendário integrado (ou envia formulário de disponibilidade)
3. Recrutador define data, hora, duração estimada (30/45/60 min) e seleciona avaliadores adicionais (opcional)
4. Backend gera `interview_token` único (UUID v4), URL de acesso única para cada participante e calendário ICS
5. Sistema envia e-mail/notificação para candidato e recrutador com:
   - Link personalizado com token: `https://ciee.org.br/entrevista/{interview_token}?role=candidate`
   - Data, hora e instruções de acesso
   - Checklist pré-entrevista (câmera, microfone, internet)
6. Registro de evento `interview.scheduled` no Event Store

**Fase 2: Pré-Entrevista e Validação**
1. T-15min: Sistema envia lembrete automático via e-mail e notificação push (se habilitado)
2. Candidato clica no link; backend valida:
   - Token existe e não expirou (validade: 24h após horário agendado)
   - Entrevista não foi cancelada ou reagendada
   - IP/device não está bloqueado por segurança
3. Sistema exibe tela de pré-entrevista com:
   - Teste de câmera e microfone (MediaDevices API)
   - Seleção de dispositivos de entrada/saída
   - Verificação de velocidade de conexão (WebRTC stats API)
4. **Tela de Consentimento de Gravação** (obrigatória):
   - Exibe texto legal claro sobre: o que será gravado, quem terá acesso, por quanto tempo será armazenado
   - Checkbox explícito: "Li e concordo com a gravação desta entrevista"
   - Opção de recusar (entrevista ocorre sem gravação; feedback automático não estará disponível)
   - Registro do consentimento com timestamp, IP e user-agent no banco
5. Registro de evento `interview.checkin` no Event Store

**Fase 3: Entrevista (Videochamada)**
1. Candidato e recrutador entram na sala virtual; o primeiro a entrar aguarda o outro
2. Frontend solicita ao backend via REST: `GET /api/v1/interviews/{id}/peer-token`
3. Backend retorna token de peer assinado e endereço do servidor de sinalização
4. PeerJS inicia conexão: troca de SDP offer/answer e ICE candidates via WebSocket (servidor de sinalização)
5. Conexão P2P estabelecida; stream de vídeo/áudio flui diretamente entre navegadores
6. Interface exibe: vídeo do outro participante, preview próprio (canto), timer da entrevista, botões (mudo, câmera, encerrar)
7. Se gravação foi consentida: `MediaRecorder API` grava a stream local em chunks de 5 segundos, enviados via WebSocket para o backend (streaming upload)
8. Backend armazena chunks no servidor de mídia (MinIO/S3) em diretório isolado por tenant/interview
9. Registro de evento `interview.recording.started` no Event Store (se consentimento dado)

**Fase 4: Encerramento e Processamento**
1. Recrutador ou candidato clica em "Encerrar Entrevista"
2. Frontend envia sinal de encerramento via WebSocket; ambos os lados fecham a conexão P2P
3. Backend finaliza o upload de chunks e consolida o arquivo de áudio/vídeo
4. Registro de evento `interview.ended` no Event Store
5. Backend publica mensagem na fila RabbitMQ `queue.interview.process`:
   ```json
   {
     "interviewId": "uuid",
     "tenantId": "tenant_ciee_sp",
     "mediaPath": "s3://ciee-recordings/tenant_ciee_sp/interviews/uuid/recording.webm",
     "participants": { "candidateId": "uuid", "recruiterId": "uuid" },
     "duration": 2734,
     "consentGiven": true
   }
   ```
6. Notificação imediata ao recrutador: "Entrevista encerrada. O feedback estará disponível em breve."

**Fase 5: Processamento de IA**
1. AI Worker consome mensagem da fila
2. Extrai áudio do arquivo WebM usando FFmpeg
3. Envia áudio ao Whisper (modelo `medium`): `whisper.transcribe(audio_path, language="pt")`
4. Recebe transcrição em formato JSON com timestamps por segmento
5. Pré-processa transcrição: identifica turnos de fala (candidato/recrutador) por sobreposição de timestamps
6. Monta prompt para Mistral 3B (análise inicial):
   ```
   Você é um especialista em recrutamento e desenvolvimento de carreira.
   Analise a transcrição abaixo de uma entrevista de estágio e forneça:
   1. Resumo executivo (3-5 pontos principais)
   2. Pontos fortes demonstrados pelo candidato
   3. Áreas de desenvolvimento identificadas
   4. Adequação ao perfil de estágio (Alta/Média/Baixa) com justificativa
   Transcrição: {transcription}
   ```
7. Se análise Mistral for classificada como "Alta Complexidade" (score heurístico < 0.7) → escalona para Claude API
8. Monta prompt Claude para feedback estruturado detalhado (ver Seção 6)
9. Compila resultado final: transcrição + análise Mistral + feedback Claude (se aplicável)
10. Publica evento `interview.feedback.ready` no RabbitMQ
11. Backend persiste feedback no banco de dados
12. Notificação enviada ao recrutador e ao candidato (se habilitado)

**Fase 6: Disponibilização do Feedback**
1. Recrutador acessa painel e revisa feedback gerado pela IA
2. Recrutador pode: aprovar como está, editar, ou rejeitar e escrever manualmente
3. Após aprovação, candidato recebe notificação e acessa seu feedback personalizado
4. Feedback fica disponível no histórico do candidato e nos relatórios do gestor CIEE

### Regras de Negócio

- Um `interview_token` é válido por exatamente 25 horas após o horário de início agendado
- A entrevista não pode ser iniciada mais de 15 minutos antes do horário agendado
- O consentimento de gravação deve ser registrado individualmente por cada participante
- Se o candidato não comparecer em 15 minutos após o horário agendado, o sistema gera alerta de "no-show" automaticamente
- Máximo de 3 tentativas de reagendamento por processo seletivo
- Gravações ficam disponíveis para o recrutador por 6 meses; candidato pode solicitar cópia da sua própria gravação

### Casos de Uso e Atores

| ID | Caso de Uso | Ator Principal | Atores Secundários |
|---|---|---|---|
| UC-01 | Agendar Entrevista | Recrutador | Sistema, Candidato |
| UC-02 | Aceitar/Recusar Agendamento | Candidato | Sistema |
| UC-03 | Realizar Pré-teste de Dispositivos | Candidato, Recrutador | Sistema |
| UC-04 | Dar Consentimento de Gravação | Candidato, Recrutador | Sistema |
| UC-05 | Participar de Videochamada | Candidato, Recrutador | Sistema (Sinalização P2P) |
| UC-06 | Visualizar Feedback Gerado | Recrutador | IA Worker |
| UC-07 | Aprovar/Editar Feedback | Recrutador | Sistema |
| UC-08 | Receber Feedback | Candidato | Sistema |
| UC-09 | Cancelar/Reagendar Entrevista | Recrutador, Candidato | Sistema |

### Tratamento de Erros e Exceções

| Cenário | Comportamento do Sistema |
|---|---|
| Token inválido ou expirado | HTTP 401; tela de erro com link para contato com o CIEE |
| Falha na câmera/microfone | Alerta em tela; link para tutorial de solução; opção de continuar sem vídeo |
| Queda de conexão P2P durante entrevista | Tentativa automática de reconexão por 30s; se falhar, oferece relay TURN; se falhar, modal para reagendar |
| Falha no upload de gravação | Chunks são enfileirados localmente (IndexedDB) e reprocessados quando conexão restabelece |
| Whisper retorna transcrição vazia ou erro | Notifica recrutador; mantém gravação disponível; permite upload manual de transcrição |
| Claude API timeout (>30s) | Fallback para Mistral 3B; feedback marcado como `source: "mistral_fallback"` |
| Entrevista excede duração máxima (2h) | Aviso em tela a cada 15min após 1h45; encerramento automático se ultrapassar 2h |

---

## 3.2 Módulo 2 — Plataforma de Feedback Contínuo

### Funcionalidades Detalhadas

**1. Feedback Bidirecional**
Sistema que permite a qualquer usuário (dentro das permissões do seu papel) enviar feedback para outro usuário. Tipos suportados:
- Candidato → Empresa (avaliação do processo seletivo)
- Empresa → Candidato (avaliação de desempenho no estágio)
- Gestor CIEE → Colaborador CIEE
- Colaborador CIEE → Gestor CIEE
- Pares (colaboradores entre si)

Cada feedback possui: destinatário, remetente (pode ser anônimo para pares), tipo (elogio, desenvolvimento, reconhecimento), competências avaliadas (escala 1-5), texto livre, visibilidade (privado/compartilhado com gestor).

**2. Análise de Sentimento**
Cada feedback textual submetido passa por pipeline de análise:
- Classificação de sentimento: Positivo / Neutro / Negativo / Misto
- Intensidade: Leve / Moderado / Forte
- Categorias temáticas detectadas: Comunicação, Trabalho em Equipe, Proatividade, Técnico, Comportamental, Pontualidade
- Flags de atenção: linguagem agressiva, discriminatória ou inadequada

Metodologia: Mistral 3B com few-shot prompting em português; validação por regras heurísticas (lista de termos sensíveis).

**3. Assistente de Redação de Feedback**
Interface de chat integrada que auxilia o usuário a redigir um feedback construtivo:
- Usuário descreve o comportamento em linguagem informal
- IA (Claude API) reformula para feedback estruturado no padrão SBI (Situação-Comportamento-Impacto)
- Usuário pode ajustar, aprovar ou pedir nova sugestão
- Limite: 5 interações por feedback por dia (rate limiting)

Exemplo de transformação:
- Input: "Ele sempre chega atrasado e não avisa"
- Output: "Em múltiplas ocasiões nas últimas duas semanas, percebi que você chegou após o horário de início das reuniões sem comunicar previamente. Isso impactou o andamento das discussões, que precisaram ser reiniciadas ou resumidas para incluí-lo."

**4. Planos de Desenvolvimento Individual (PDI)**
Gerado automaticamente ou sob demanda, o PDI agrega:
- Padrões identificados nos feedbacks recebidos nos últimos 90 dias
- Competências com avaliações abaixo da meta (< 3.5/5.0)
- Sugestões de ações de desenvolvimento com prazo e recursos (cursos, leituras, mentores)
- Metas SMART geradas pela IA com base no histórico
- Acompanhamento de progresso (check-ins mensais)

**5. Dashboard Preditivo**
Disponível para Gestores CIEE com as seguintes visualizações:
- Mapa de calor de sentimento por departamento/empresa
- Tendência de engajamento de estagiários (baseado em frequência e qualidade de feedbacks)
- Score de risco de evasão por candidato (modelo baseado em: ausência de feedbacks, sentimento negativo recorrente, ausência em entrevistas)
- Ranking de empresas parceiras por qualidade de experiência do estagiário
- Alertas automáticos para situações de risco (ex: estagiário sem feedback por 30 dias)

### Fluxo de Dados (Submissão ao Dashboard)

```
Usuário submete feedback (formulário ou assistente IA)
        ↓
Feedback Service valida dados (campos obrigatórios, permissões de papel)
        ↓
Persistência imediata no banco (status: pending_analysis)
        ↓
Evento publicado: feedback.submitted → RabbitMQ
        ↓
AI Worker consome evento:
  → Análise de sentimento (Mistral 3B)
  → Detecção de temas e categorias
  → Flag de conteúdo inadequado (se detectado → notifica moderador)
        ↓
Resultado retorna via RabbitMQ: feedback.analyzed
        ↓
Feedback Service atualiza registro: status: analyzed, adiciona campos de IA
        ↓
View materializada do dashboard é invalidada no Redis (TTL: 5min)
        ↓
Próxima requisição ao dashboard recarrega dados do PostgreSQL
        ↓
Dados agregados ficam disponíveis no dashboard do gestor
```

### Casos de Uso por Perfil

| Perfil | Casos de Uso |
|---|---|
| **Candidato** | Enviar feedback sobre empresa; visualizar feedbacks recebidos; acessar PDI próprio; usar assistente de redação |
| **Recrutador/Empresa** | Enviar feedback sobre candidato/estagiário; visualizar feedbacks recebidos sobre a empresa; acessar relatório de equipe |
| **Gestor CIEE** | Acessar dashboard preditivo; visualizar todos os feedbacks (com flag de moderação); gerar relatórios; criar/atribuir PDIs |
| **Colaborador CIEE** | Enviar e receber feedbacks de pares e gestores; acessar PDI próprio; participar de check-ins |

---

> ⚠️ **Pontos de Atenção — Seção 3**
> - O fluxo de consentimento de gravação deve ser validado juridicamente pela equipe legal do CIEE antes da implementação, especialmente o texto exibido ao usuário.
> - O feedback anônimo entre pares cria risco de uso inadequado; definir política clara de moderação e limites de anonimato.
> - O modelo de risco de evasão do dashboard (Módulo 2) requer dados históricos de pelo menos 6 meses para ter precisão aceitável; no lançamento, o modelo deve ser apresentado como "experimental" com baixa confiança.

---

# 4. Especificação Técnica da API REST

## 4.1 Convenções e Padrões da API

- **Versionamento:** Via path — `/api/v1/...`. Versões anteriores mantidas por 12 meses após deprecação.
- **Autenticação:** Bearer Token (JWT) no header `Authorization: Bearer {token}`. Endpoints públicos usam token de entrevista (`X-Interview-Token`).
- **Formato de resposta:** JSON em todos os endpoints. Content-Type: `application/json; charset=utf-8`
- **Multi-tenancy:** Header obrigatório `X-Tenant-ID: {tenantSlug}` em todas as requisições autenticadas
- **Paginação:** Query params `page` (0-based) e `size` (padrão 20, máximo 100); resposta inclui `totalElements`, `totalPages`, `currentPage`
- **Ordenação:** Query param `sort=campo,asc|desc` (ex: `sort=createdAt,desc`)
- **Timestamps:** ISO 8601 com timezone: `2026-02-15T14:30:00-03:00`
- **IDs:** UUID v4 em todos os recursos
- **Envelope de resposta padrão:**
```json
{
  "success": true,
  "data": { },
  "meta": { "timestamp": "2026-02-15T14:30:00-03:00", "requestId": "uuid" },
  "errors": []
}
```

## 4.2 Endpoints — Módulo 1 (Videochamada)

### Agendamento

**POST /api/v1/interviews**
Cria um novo agendamento de entrevista.

Request:
```json
{
  "candidateId": "3f7a1b2c-...",
  "jobId": "9a8b7c6d-...",
  "scheduledAt": "2026-03-01T10:00:00-03:00",
  "durationMinutes": 45,
  "additionalRecruiters": ["uuid1", "uuid2"],
  "notes": "Entrevista técnica para vaga de Dev Jr."
}
```

Response (201 Created):
```json
{
  "success": true,
  "data": {
    "interviewId": "1a2b3c4d-...",
    "status": "SCHEDULED",
    "candidateAccessUrl": "https://ciee.org.br/entrevista/TOKEN_CAND?role=candidate",
    "recruiterAccessUrl": "https://ciee.org.br/entrevista/TOKEN_REC?role=recruiter",
    "scheduledAt": "2026-03-01T10:00:00-03:00",
    "expiresAt": "2026-03-02T11:00:00-03:00"
  }
}
```

**GET /api/v1/interviews/{interviewId}**
Retorna detalhes de uma entrevista.

**PATCH /api/v1/interviews/{interviewId}**
Atualiza agendamento (reagendamento, cancelamento).

Request:
```json
{
  "action": "RESCHEDULE",
  "newScheduledAt": "2026-03-05T14:00:00-03:00",
  "reason": "Conflito de agenda do candidato"
}
```

**GET /api/v1/interviews?status=SCHEDULED&from=2026-03-01&to=2026-03-31**
Lista entrevistas com filtros.

### Sessão de Entrevista

**POST /api/v1/interviews/{interviewId}/validate-token**
Valida o token de acesso e retorna dados da sessão.

Request:
```json
{ "token": "interview_access_token", "role": "candidate" }
```

Response:
```json
{
  "success": true,
  "data": {
    "valid": true,
    "interviewId": "uuid",
    "scheduledAt": "2026-03-01T10:00:00-03:00",
    "otherParticipantReady": false,
    "signalingServerUrl": "wss://signal.ciee.org.br",
    "peerToken": "signed_peer_token"
  }
}
```

**POST /api/v1/interviews/{interviewId}/consent**
Registra consentimento de gravação.

Request:
```json
{
  "userId": "uuid",
  "role": "candidate",
  "consentGiven": true,
  "consentText": "Li e concordo com os termos de gravação...",
  "deviceInfo": { "userAgent": "...", "ip": "..." }
}
```

**POST /api/v1/interviews/{interviewId}/events**
Registra eventos da sessão (start, end, participant-joined, etc).

```json
{ "eventType": "PARTICIPANT_JOINED", "participantId": "uuid", "timestamp": "..." }
```

### Gravação e Feedback

**POST /api/v1/interviews/{interviewId}/recordings/chunks**
Upload de chunk de gravação (multipart/form-data). Header: `X-Chunk-Index: 0`, `X-Chunk-Total: 47`

**POST /api/v1/interviews/{interviewId}/recordings/finalize**
Sinaliza fim do upload e inicia processamento.

**GET /api/v1/interviews/{interviewId}/feedback**
Retorna feedback gerado pela IA.

Response:
```json
{
  "success": true,
  "data": {
    "feedbackId": "uuid",
    "status": "PENDING_REVIEW",
    "generatedBy": "claude_api",
    "transcript": { "text": "...", "segments": [] },
    "analysis": {
      "executiveSummary": "...",
      "strengths": ["Boa comunicação verbal", "Conhecimento técnico sólido"],
      "developmentAreas": ["Poderia ser mais objetivo nas respostas"],
      "fitScore": "HIGH",
      "fitJustification": "..."
    },
    "generatedAt": "2026-03-01T11:23:00-03:00"
  }
}
```

**PATCH /api/v1/interviews/{interviewId}/feedback**
Recrutador aprova, edita ou rejeita feedback.

```json
{
  "action": "APPROVE_WITH_EDITS",
  "editedAnalysis": { "developmentAreas": ["..."] },
  "releaseToCandidate": true
}
```

## 4.3 Endpoints — Módulo 2 (Feedback Contínuo)

**POST /api/v1/feedbacks**
Submete novo feedback.

Request:
```json
{
  "recipientId": "uuid",
  "recipientType": "CANDIDATE",
  "feedbackType": "DEVELOPMENT",
  "competencies": [
    { "id": "communication", "score": 3 },
    { "id": "teamwork", "score": 4 }
  ],
  "text": "Texto do feedback...",
  "isAnonymous": false,
  "visibility": "RECIPIENT_AND_MANAGER"
}
```

Response (202 Accepted — processamento assíncrono):
```json
{
  "success": true,
  "data": {
    "feedbackId": "uuid",
    "status": "PENDING_ANALYSIS",
    "estimatedAnalysisTime": "30s"
  }
}
```

**GET /api/v1/feedbacks/received?userId={id}&from=...&to=...**
Lista feedbacks recebidos por um usuário.

**GET /api/v1/feedbacks/{feedbackId}/analysis**
Retorna análise de sentimento de um feedback.

Response:
```json
{
  "data": {
    "sentiment": "POSITIVE",
    "intensity": "MODERATE",
    "themes": ["communication", "proactivity"],
    "flags": [],
    "confidenceScore": 0.87
  }
}
```

**POST /api/v1/feedbacks/writing-assistant**
Usa assistente de redação de feedback.

Request:
```json
{
  "draftText": "Ele sempre chega atrasado e não avisa",
  "context": { "recipientRole": "INTERN", "feedbackType": "DEVELOPMENT" },
  "sessionId": "uuid"
}
```

Response:
```json
{
  "data": {
    "suggestion": "Em múltiplas ocasiões nas últimas duas semanas...",
    "remainingInteractions": 4,
    "sessionId": "uuid"
  }
}
```

**GET /api/v1/pdis/{userId}**
Retorna PDI atual do usuário.

**POST /api/v1/pdis/{userId}/generate**
Dispara geração de novo PDI baseado em feedbacks recentes.

**GET /api/v1/dashboard/overview**
Retorna dados do dashboard para gestores. Query params: `tenantId`, `departmentId`, `period`.

**GET /api/v1/dashboard/risk-scores**
Retorna scores de risco de evasão.

## 4.4 Códigos de Erro Padronizados

```json
{
  "success": false,
  "data": null,
  "errors": [
    {
      "code": "INTERVIEW_TOKEN_EXPIRED",
      "message": "O link de acesso à entrevista expirou. Entre em contato com o recrutador.",
      "field": null,
      "httpStatus": 401
    }
  ]
}
```

| Código | HTTP | Descrição |
|---|---|---|
| `INTERVIEW_TOKEN_EXPIRED` | 401 | Token de entrevista expirado |
| `INTERVIEW_TOKEN_INVALID` | 401 | Token não encontrado ou inválido |
| `INTERVIEW_NOT_FOUND` | 404 | Entrevista não encontrada |
| `CONSENT_NOT_GIVEN` | 403 | Gravação solicitada sem consentimento registrado |
| `FEEDBACK_NOT_READY` | 202 | Feedback ainda em processamento |
| `RATE_LIMIT_EXCEEDED` | 429 | Limite de requisições atingido (header Retry-After incluído) |
| `TENANT_NOT_FOUND` | 404 | Tenant não identificado |
| `INSUFFICIENT_PERMISSIONS` | 403 | Papel do usuário não tem acesso ao recurso |
| `VALIDATION_ERROR` | 400 | Campos obrigatórios ausentes ou inválidos (campo `field` preenchido) |
| `AI_SERVICE_UNAVAILABLE` | 503 | Serviço de IA temporariamente indisponível |
| `RECORDING_UPLOAD_FAILED` | 500 | Falha no upload de gravação; tentar novamente |

---

> ⚠️ **Pontos de Atenção — Seção 4**
> - O endpoint de upload de chunks de gravação (`POST .../recordings/chunks`) deve implementar validação de tamanho de chunk (máximo 5MB por chunk) e hash MD5 para integridade.
> - O endpoint do assistente de redação deve respeitar rate limiting por usuário (5 interações/dia) — implementar via Redis com TTL de 24h.
> - Versionar a API desde o início; mudanças em endpoints de feedback podem impactar integrações com dashboards externos futuros.

---

# 5. Modelagem de Dados

## 5.1 Entidades Principais

### Entidade: `interviews`
| Campo | Tipo | Descrição |
|---|---|---|
| `id` | UUID PK | Identificador único |
| `tenant_id` | VARCHAR(100) | Identificador do tenant |
| `job_id` | UUID FK | Referência à vaga |
| `candidate_id` | UUID FK | Referência ao candidato |
| `recruiter_id` | UUID FK | Recrutador responsável |
| `status` | ENUM | SCHEDULED, IN_PROGRESS, COMPLETED, CANCELLED, NO_SHOW |
| `scheduled_at` | TIMESTAMPTZ | Data/hora agendada |
| `started_at` | TIMESTAMPTZ | Data/hora real de início |
| `ended_at` | TIMESTAMPTZ | Data/hora de encerramento |
| `duration_minutes` | INT | Duração estimada |
| `actual_duration_seconds` | INT | Duração real |
| `candidate_token` | VARCHAR(255) | Token de acesso do candidato |
| `recruiter_token` | VARCHAR(255) | Token de acesso do recrutador |
| `token_expires_at` | TIMESTAMPTZ | Expiração dos tokens |
| `recording_path` | TEXT | Caminho no storage |
| `recording_size_bytes` | BIGINT | Tamanho do arquivo |
| `created_at` | TIMESTAMPTZ | Data de criação |
| `updated_at` | TIMESTAMPTZ | Última atualização |

### Entidade: `recording_consents`
| Campo | Tipo | Descrição |
|---|---|---|
| `id` | UUID PK | Identificador |
| `interview_id` | UUID FK | Entrevista relacionada |
| `user_id` | UUID FK | Usuário que consentiu |
| `role` | ENUM | CANDIDATE, RECRUITER |
| `consent_given` | BOOLEAN | Se consentiu |
| `consent_text_hash` | VARCHAR(64) | SHA-256 do texto exibido (auditoria) |
| `ip_address` | INET | IP do usuário |
| `user_agent` | TEXT | User-agent do browser |
| `consented_at` | TIMESTAMPTZ | Timestamp do consentimento |

### Entidade: `interview_feedbacks`
| Campo | Tipo | Descrição |
|---|---|---|
| `id` | UUID PK | Identificador |
| `interview_id` | UUID FK | Entrevista relacionada |
| `status` | ENUM | PENDING, PROCESSING, GENERATED, REVIEWED, RELEASED, FAILED |
| `source_model` | VARCHAR(50) | claude_api, mistral_3b, mistral_fallback, manual |
| `transcript_text` | TEXT | Transcrição completa |
| `transcript_segments` | JSONB | Segmentos com timestamps |
| `executive_summary` | TEXT | Resumo executivo |
| `strengths` | JSONB | Array de pontos fortes |
| `development_areas` | JSONB | Array de áreas de desenvolvimento |
| `fit_score` | ENUM | HIGH, MEDIUM, LOW |
| `fit_justification` | TEXT | Justificativa da adequação |
| `ai_confidence_score` | DECIMAL(3,2) | 0.00–1.00 |
| `reviewed_by` | UUID FK | Recrutador que revisou |
| `reviewed_at` | TIMESTAMPTZ | Data da revisão |
| `released_to_candidate` | BOOLEAN | Se foi liberado ao candidato |
| `released_at` | TIMESTAMPTZ | Data da liberação |
| `created_at` | TIMESTAMPTZ | Data de criação |

### Entidade: `continuous_feedbacks`
| Campo | Tipo | Descrição |
|---|---|---|
| `id` | UUID PK | Identificador |
| `tenant_id` | VARCHAR(100) | Tenant |
| `sender_id` | UUID FK | Remetente (null se anônimo) |
| `recipient_id` | UUID FK | Destinatário |
| `feedback_type` | ENUM | PRAISE, DEVELOPMENT, RECOGNITION, NEUTRAL |
| `sender_role` | ENUM | CANDIDATE, RECRUITER, MANAGER, PEER |
| `recipient_role` | ENUM | CANDIDATE, RECRUITER, MANAGER, PEER |
| `is_anonymous` | BOOLEAN | Anonimato do remetente |
| `visibility` | ENUM | PRIVATE, RECIPIENT_AND_MANAGER, PUBLIC_TEAM |
| `text` | TEXT | Conteúdo do feedback |
| `competency_scores` | JSONB | `{"communication": 4, "teamwork": 3}` |
| `status` | ENUM | PENDING_ANALYSIS, ANALYZED, FLAGGED, ARCHIVED |
| `sentiment` | ENUM | POSITIVE, NEGATIVE, NEUTRAL, MIXED |
| `sentiment_intensity` | ENUM | LOW, MODERATE, HIGH |
| `themes` | JSONB | Array de temas detectados |
| `flags` | JSONB | Array de flags de moderação |
| `ai_confidence_score` | DECIMAL(3,2) | Confiança da análise |
| `created_at` | TIMESTAMPTZ | Data de criação |

### Entidade: `development_plans` (PDI)
| Campo | Tipo | Descrição |
|---|---|---|
| `id` | UUID PK | Identificador |
| `user_id` | UUID FK | Usuário dono do PDI |
| `tenant_id` | VARCHAR(100) | Tenant |
| `period_start` | DATE | Início do período |
| `period_end` | DATE | Fim do período |
| `status` | ENUM | ACTIVE, COMPLETED, CANCELLED |
| `competency_gaps` | JSONB | Competências abaixo da meta |
| `goals` | JSONB | Array de metas SMART |
| `actions` | JSONB | Ações de desenvolvimento sugeridas |
| `generated_by` | ENUM | AI, MANAGER, MANUAL |
| `progress_score` | DECIMAL(3,2) | Progresso geral (0–1) |
| `last_checkin_at` | TIMESTAMPTZ | Último check-in |
| `created_at` | TIMESTAMPTZ | Data de criação |

### Entidade: `domain_events` (Event Store)
| Campo | Tipo | Descrição |
|---|---|---|
| `id` | UUID PK | Identificador do evento |
| `aggregate_id` | UUID | ID do agregado (ex: interviewId) |
| `aggregate_type` | VARCHAR(100) | Tipo do agregado (Interview, Feedback) |
| `event_type` | VARCHAR(100) | Tipo do evento (interview.scheduled) |
| `tenant_id` | VARCHAR(100) | Tenant |
| `payload` | JSONB | Dados completos do evento |
| `metadata` | JSONB | Usuário, IP, correlationId |
| `occurred_at` | TIMESTAMPTZ | Timestamp do evento |
| `version` | INT | Versão para otimistic locking |

## 5.2 Diagrama Entidade-Relacionamento (Descritivo)

```
users ──< recording_consents >── interviews ──< interview_feedbacks
  │                                  │
  │                              domain_events (polimórfico)
  │
  ├──< continuous_feedbacks >── users (recipient)
  │
  └──< development_plans
```

- Um `user` pode ter múltiplos `continuous_feedbacks` (enviados e recebidos)
- Uma `interview` tem exatamente dois `recording_consents` (candidato + recrutador)
- Uma `interview` tem no máximo um `interview_feedback` por vez
- Um `user` tem no máximo um `development_plan` ativo por vez
- `domain_events` é uma tabela append-only particionada por `aggregate_type` para performance

## 5.3 Estratégia de Multi-Tenancy

Utiliza-se **schema-per-tenant** no PostgreSQL:
- Cada tenant possui um schema dedicado: `CREATE SCHEMA tenant_{slug}`
- Todas as tabelas acima existem dentro do schema do tenant
- Uma tabela global `public.tenants` mantém o catálogo de tenants com metadados
- A aplicação Spring Boot usa `AbstractRoutingDataSource` com ThreadLocal para rotear dinamicamente para o schema correto baseado no header `X-Tenant-ID`
- Migrations executadas via Flyway com suporte a múltiplos schemas (callback `FlywayMigrationStrategy`)

```sql
-- Exemplo de isolamento
SET search_path TO tenant_ciee_sp;
SELECT * FROM interviews WHERE status = 'SCHEDULED';
```

## 5.4 Política de Retenção e Auditoria (Event Sourcing)

- **Feedbacks de entrevista:** Retidos por 2 anos; após esse prazo, anonimizados (candidate_id substituído por hash irreversível)
- **Gravações de vídeo:** Retidas por 6 meses; exclusão automática via job agendado
- **domain_events:** Retidos indefinidamente para auditoria; particionados por ano no PostgreSQL para performance
- **Dados de consentimento:** Retidos pelo período máximo de retenção da gravação associada, mais 6 meses adicionais como comprovação legal
- **Solicitação de exclusão (LGPD Art. 18):** Aciona processo de anonimização que: substitui dados pessoais por hashes, exclui arquivos de mídia, preserva events com dados anonimizados

---

> ⚠️ **Pontos de Atenção — Seção 5**
> - A tabela `domain_events` crescerá rapidamente; implementar particionamento por `occurred_at` (range partitioning anual) desde o início.
> - O JSONB para `competency_scores`, `goals` e `actions` oferece flexibilidade mas dificulta queries analíticas — criar índices GIN e considerar views materializadas para relatórios frequentes.
> - O processo de anonimização para LGPD deve ser auditado: criar tabela `gdpr_requests` para rastrear todas as solicitações de exclusão e seu status.

---

# 6. Integração com IA

## 6.1 Whisper — Transcrição de Áudio

**Como funciona:**
O arquivo de áudio extraído da gravação WebM (via FFmpeg, convertido para WAV 16kHz mono) é processado pelo modelo `whisper-medium` rodando localmente via biblioteca `openai-whisper` (Python).

**Formato de entrada:** Arquivo WAV, 16kHz, mono, máximo 2GB
**Formato de saída:**
```json
{
  "text": "Transcrição completa como string única",
  "segments": [
    {
      "id": 0,
      "start": 0.0,
      "end": 4.2,
      "text": " Olá, obrigado por participar do processo seletivo.",
      "avg_logprob": -0.21,
      "no_speech_prob": 0.05
    }
  ],
  "language": "pt"
}
```

**Latência esperada:** Aproximadamente 1/3 do tempo real de áudio para `whisper-medium` em GPU Tesla T4. Para entrevista de 45min: ~15min de processamento. Em CPU: ~3x o tempo real (135min) — GPU é altamente recomendada.

**Limitações:**
- Precisão menor em sotaques regionais muito marcados (nordestino, gaúcho)
- Dificuldade com jargões técnicos muito específicos ou siglas — mitigado com prompt inicial ao modelo: `"A seguinte entrevista é sobre tecnologia/marketing/etc"`
- Sobreposição de falas (dois falando simultaneamente) gera transcrição mesclada — identificação de turnos é pós-processada por heurística de timestamps

## 6.2 Mistral 3B — Resumos e Insights

**Utilização:** Análise inicial de transcrições de entrevistas e análise de sentimento de feedbacks contínuos.

**Infraestrutura:** Modelo servido localmente via `Ollama` ou `llama.cpp` em servidor dedicado. Endpoint interno: `http://ai-worker:11434/api/generate`

**Prompt Engineering — Análise de Entrevista:**
```
<s>[INST] Você é um especialista sênior em recrutamento e desenvolvimento humano com 15 anos de experiência no mercado brasileiro.

Analise a transcrição abaixo de uma entrevista de estágio e retorne EXCLUSIVAMENTE um JSON válido com a seguinte estrutura:
{
  "executiveSummary": "string com 3-5 frases resumindo a entrevista",
  "strengths": ["array", "de", "pontos", "fortes"],
  "developmentAreas": ["array", "de", "áreas", "para", "desenvolver"],
  "fitScore": "HIGH | MEDIUM | LOW",
  "fitJustification": "string justificando o fit score",
  "complexityScore": 0.0-1.0  // 0=simples, 1=muito complexo/necessita revisão humana
}

TRANSCRIÇÃO:
{transcription_text}

Retorne apenas o JSON, sem texto adicional. [/INST]
```

**Prompt Engineering — Análise de Sentimento:**
```
<s>[INST] Classifique o feedback abaixo em português. Retorne apenas JSON:
{
  "sentiment": "POSITIVE | NEGATIVE | NEUTRAL | MIXED",
  "intensity": "LOW | MODERATE | HIGH",
  "themes": ["lista de temas: communication, teamwork, technical, behavioral, punctuality, proactivity"],
  "flags": ["aggressive_language | discriminatory | inappropriate"],
  "confidenceScore": 0.0-1.0
}

FEEDBACK: "{feedback_text}" [/INST]
```

## 6.3 Claude API — Feedback Detalhado e Assistente de Redação

**Quando é utilizado:**
1. Quando o `complexityScore` da análise Mistral é ≥ 0.7 (caso complexo)
2. Sempre para o assistente de redação de feedback (qualidade superior necessária)

**Modelo utilizado:** `claude-3-5-haiku-20241022` para feedback de entrevistas (balanceia custo e qualidade); `claude-3-5-sonnet-20241022` para assistente de redação (mais criativo e preciso).

**Prompt — Feedback Estruturado de Entrevista:**
```
Você é um especialista em desenvolvimento humano e recrutamento, atuando como consultor do CIEE.

Com base na transcrição e na análise prévia abaixo, gere um feedback profissional, empático e acionável para o candidato. O feedback deve:
- Ser escrito em português brasileiro formal-amigável
- Seguir a estrutura SBI (Situação-Comportamento-Impacto)
- Destacar 3 pontos fortes com exemplos específicos da entrevista
- Apontar 2-3 áreas de desenvolvimento com sugestões práticas
- Terminar com uma mensagem encorajadora personalizada

ANÁLISE PRÉVIA (Mistral): {mistral_analysis_json}
TRANSCRIÇÃO: {transcription_excerpt}  [máximo 2000 tokens de transcrição]
PERFIL DA VAGA: {job_description}

Retorne um JSON com: {"candidateFeedback": "...", "internalRecruiterNotes": "..."}
```

**Prompt — Assistente de Redação:**
```
Você é um coach de comunicação especializado em feedback construtivo no ambiente corporativo brasileiro.

O usuário quer enviar um feedback para um {recipient_role}. Transforme o rascunho abaixo em um feedback profissional no padrão SBI (Situação-Comportamento-Impacto), mantendo a intenção original.

Rascunho: "{draft_text}"
Tipo de feedback: {feedback_type}

Retorne apenas o feedback reformulado, sem explicações. Máximo 200 palavras.
```

**Rate Limiting Claude API:**
- Orçamento mensal definido pelo CIEE; limite diário configurável via Redis counter
- Threshold: se >80% do limite diário atingido → redirecionar para Mistral 3B
- Threshold: se >95% do limite → bloquear novas requisições até meia-noite (reset do counter)
- Alertas automáticos ao administrador em 60%, 80% e 95% do limite

## 6.4 Análise de Sentimento — Metodologia e Exemplos

**Categorias de sentimento:**
- `POSITIVE`: Elogios claros, reconhecimento, satisfação expressa
- `NEGATIVE`: Críticas, insatisfação, problemas reportados
- `NEUTRAL`: Informativo, descritivo, sem carga emocional
- `MIXED`: Combinação de aspectos positivos e negativos

**Exemplo de saída:**
```json
{
  "feedbackText": "A Ana demonstra muita vontade de aprender, mas tem dificuldade em cumprir prazos, o que impacta o time.",
  "analysis": {
    "sentiment": "MIXED",
    "intensity": "MODERATE",
    "themes": ["proactivity", "punctuality", "teamwork"],
    "flags": [],
    "confidenceScore": 0.91
  }
}
```

## 6.5 Política de Fallback quando a IA Falha

| Situação | Ação |
|---|---|
| Whisper falha (erro de processo) | Retry 2x com intervalo de 5min; se persistir, notificar admin e manter gravação para reprocessamento manual |
| Mistral não responde (timeout 30s) | Retornar template de feedback baseado em regras; marcar como `source: rule_based`; notificar admin |
| Claude API retorna erro 5xx | Retry 1x imediatamente, depois 1x após 60s; se persistir, fallback para Mistral |
| Claude API retorna erro 429 | Implementar queue de espera com backoff; nunca deixar requisição cair |
| `confidenceScore < 0.5` em qualquer análise | Marcar feedback como `LOW_CONFIDENCE`; incluir aviso ao recrutador para revisão obrigatória |
| JSON inválido retornado pelo LLM | Retry com prompt que reforça o formato JSON; se falhar 2x, extrair campos via regex e marcar como `parsing_recovered` |

---

> ⚠️ **Pontos de Atenção — Seção 6**
> - O uso de GPU para Whisper é crítico para SLA de feedback em 2 horas; o dimensionamento de hardware deve ser feito antes do lançamento.
> - Os prompts para Mistral 3B devem ser testados extensivamente com dados reais de entrevistas em português antes do go-live; qualidade pode variar.
> - Manter logs de todos os prompts enviados e respostas recebidas das APIs de IA (com dados anonimizados) para auditoria e melhoria contínua dos prompts.
> - Custos da Claude API devem ser monitorados diariamente; uma entrevista complexa pode consumir ~2000-4000 tokens de entrada + ~800 de saída.

---

# 7. Infraestrutura e DevOps

## 7.1 Componentes de Infraestrutura

| Componente | Tecnologia | Propósito | Alta Disponibilidade |
|---|---|---|---|
| Servidor de Sinalização | PeerJS Server (Node.js) | Negociação WebRTC | 2 instâncias com Load Balancer sticky sessions |
| Message Broker | RabbitMQ 3.12 | Filas assíncronas de IA | Cluster de 3 nós (quorum queues) |
| Cache / Session | Redis 7.2 | Cache, rate limiting, sessões | Redis Sentinel (1 master + 2 replicas) |
| Banco de Dados | PostgreSQL 16 | Dados persistentes | Primary + 1 Read Replica |
| Storage de Mídia | MinIO (on-prem) / AWS S3 | Gravações de entrevistas | MinIO distribuído ou S3 com versionamento |
| AI Worker | Python / FastAPI | Pipeline de IA | 2 workers configuráveis |
| API Gateway | NGINX / AWS ALB | TLS, roteamento, rate limiting | Múltiplas instâncias |

## 7.2 Estratégia de Filas no RabbitMQ

**Exchanges e Filas:**
```
Exchange: ciee.interviews (topic)
  ├── Routing Key: interview.process → Queue: interview.ai.processing
  ├── Routing Key: interview.completed → Queue: interview.notifications
  └── Routing Key: interview.failed → Queue: interview.dlq (dead-letter)

Exchange: ciee.feedbacks (topic)
  ├── Routing Key: feedback.submitted → Queue: feedback.ai.analysis
  ├── Routing Key: feedback.analyzed → Queue: feedback.dashboard.update
  └── Routing Key: feedback.flagged → Queue: feedback.moderation

Exchange: ciee.notifications (fanout)
  └── Queue: notification.email
  └── Queue: notification.push
```

**Configurações:**
- Todas as filas são `durable: true` e `persistent: true` (mensagens sobrevivem a restart)
- Dead-letter queue para mensagens que falham após 3 retries
- Prefetch count: 1 por consumer (processamento sequencial no AI Worker para não sobrecarregar GPU)
- TTL de mensagens na DLQ: 72 horas; após isso, alerta ao administrador

## 7.3 Estratégia de Cache no Redis

| Dado em Cache | Chave Redis | TTL | Estratégia de Invalidação |
|---|---|---|---|
| Sessão de usuário | `session:{sessionId}` | 15min (renovável) | Sliding window; logout explícito |
| Token de entrevista (validação) | `interview:token:{token}` | 25h | Expiração natural ou cancelamento |
| Dados de tenant (metadados) | `tenant:{tenantId}:meta` | 1h | Invalidação manual em updates |
| Dashboard (dados agregados) | `dashboard:{tenantId}:{period}` | 5min | Evento `feedback.analyzed` invalida |
| Rate limit (Claude API) | `ratelimit:claude:daily:{date}` | TTL até meia-noite | Reset automático por TTL |
| Rate limit (assistente) | `ratelimit:assistant:{userId}:{date}` | TTL até meia-noite | Reset automático por TTL |
| Resultado de sentimento | `sentiment:{feedbackId}` | 7 dias | Nunca (resultado imutável) |
| Peer token (WebRTC) | `peer:token:{interviewId}:{userId}` | 3h | Fim da entrevista |

## 7.4 Rate Limiting Inteligente

**Camadas de Rate Limiting:**

**Camada 1 — NGINX (global por IP):**
- Máximo 100 req/min por IP para endpoints de API
- Máximo 10 req/min por IP para endpoints de autenticação
- Máximo 5 req/min para upload de chunks (por interview_id)

**Camada 2 — Aplicação (por usuário/tenant):**
- Assistente de redação: 5 interações/usuário/dia
- Geração de PDI: 1 geração/usuário/semana
- Exportação de relatórios: 10/usuário/dia

**Camada 3 — IA (por quota de API):**
- Claude API: configurável por tenant; padrão 1000 requisições/mês
- Escalonamento automático: se >80% quota → Mistral fallback

**Implementação Redis:**
```lua
-- Script Lua atômico para rate limiting
local key = KEYS[1]
local limit = tonumber(ARGV[1])
local ttl = tonumber(ARGV[2])
local current = redis.call('INCR', key)
if current == 1 then
  redis.call('EXPIRE', key, ttl)
end
if current > limit then
  return 0  -- limite excedido
end
return 1  -- permitido
```

## 7.5 Ambientes

| Ambiente | Propósito | Configurações Chave |
|---|---|---|
| **Desenvolvimento** | Desenvolvimento local individual | Docker Compose com todos os serviços; Whisper modelo `tiny`; Mistral via Ollama local; Claude API com chave de desenvolvimento |
| **Homologação (QA)** | Testes integrados e UAT | Infraestrutura similar à produção, escala reduzida; dados sintéticos; Claude API com quota limitada |
| **Produção** | Sistema em produção | Alta disponibilidade; monitoramento completo; Whisper modelo `medium` em GPU; backup automatizado |

**Docker Compose (Desenvolvimento):**
```yaml
services:
  postgres:
    image: postgres:16
    environment:
      POSTGRES_DB: ciee_dev
  redis:
    image: redis:7.2-alpine
  rabbitmq:
    image: rabbitmq:3.12-management
  peerjs:
    image: peerjs/peerjs-server
  ai-worker:
    build: ./ai-worker
    environment:
      WHISPER_MODEL: tiny
      OLLAMA_HOST: http://ollama:11434
  backend:
    build: ./backend
  frontend:
    build: ./frontend
```

## 7.6 Estratégia de Escalabilidade

**Fase 1 — On-premises / Cloud Privada:**
- Todos os serviços em VMs ou bare metal
- PostgreSQL single node com backup diário
- AI Worker em servidor com GPU dedicada

**Fase 2 — Cloud Pública (AWS/GCP):**
- Backend Spring Boot em Kubernetes (EKS/GKE); HPA baseado em CPU/RPS
- RabbitMQ → migração para Amazon MQ ou CloudAMQP
- PostgreSQL → RDS PostgreSQL com Multi-AZ
- Storage → AWS S3 com lifecycle policies
- AI Worker → EC2 com GPU (g4dn.xlarge) ou AWS SageMaker para Whisper

**Indicadores para escalar horizontalmente:**
- CPU > 70% por 5 minutos → scale up backend
- Fila `interview.ai.processing` > 50 mensagens → adicionar AI Worker
- Latência média de API > 500ms → investigar gargalo

---

> ⚠️ **Pontos de Atenção — Seção 7**
> - O servidor TURN para WebRTC deve ser provisionado desde o MVP; sem TURN, ~15% das conexões P2P falham em redes corporativas com NAT restritivo.
> - O MinIO em produção requer storage distribuído para alta disponibilidade; planejar armazenamento para crescimento de 50GB/mês de gravações.
> - Definir SLA de processamento de IA (ex: feedback em até 2h) e implementar alertas quando a fila ultrapassar o threshold.

---

# 8. Segurança, Privacidade e LGPD

## 8.1 Autenticação e Autorização

**Autenticação:**
- JWT com algoritmo RS256 (chave assimétrica); access token válido por 15 minutos; refresh token válido por 7 dias (rotativo)
- OAuth2 / OIDC para SSO com empresas parceiras (Google Workspace, Microsoft Azure AD)
- 2FA opcional para gestores CIEE e administradores (TOTP via Google Authenticator/Authy)
- Tokens de entrevista (stateless, assinados com HMAC-SHA256): acesso temporário sem autenticação completa

**Autorização — RBAC (Role-Based Access Control):**

| Role | Permissões |
|---|---|
| `CANDIDATE` | Leitura do próprio perfil; enviar/receber feedbacks sobre si; acessar próprio PDI |
| `RECRUITER` | CRUD de entrevistas para suas vagas; visualizar/editar feedbacks das suas entrevistas; relatórios da empresa |
| `COMPANY_ADMIN` | Todas as permissões de RECRUITER para todos recrutadores da empresa; configurações do tenant |
| `CIEE_MANAGER` | Acesso de leitura a todos os dados do tenant CIEE; dashboard preditivo; moderação de feedbacks |
| `PLATFORM_ADMIN` | Acesso total; gerenciamento de tenants; configurações de IA; auditoria |

## 8.2 Criptografia

**Dados em trânsito:**
- TLS 1.3 obrigatório para todos os endpoints HTTP (HSTS habilitado; HSTS max-age 1 ano)
- WebSocket (WSS) com TLS 1.3 para sinalização e upload de gravações
- Stream WebRTC criptografada nativamente com SRTP/DTLS (padrão do protocolo)

**Dados em repouso:**
- Gravações de vídeo: cifradas com AES-256-GCM no storage (server-side encryption no S3/MinIO)
- Dados sensíveis no banco (CPF, e-mail): cifrados com pgcrypto (AES-256) antes de persistir; chave de cifra armazenada em AWS KMS ou HashiCorp Vault
- Backups do PostgreSQL: cifrados com AES-256 antes de upload ao storage de backup

## 8.3 Consentimento Explícito para Gravação

**Fluxo Técnico:**
1. Frontend exibe modal com texto legal completo (version-controlled; cada mudança gera novo hash)
2. Usuário deve rolar até o final do texto (scroll tracking no frontend)
3. Checkbox habilitado apenas após rolar até o final
4. Ao marcar checkbox e clicar em "Confirmar", frontend envia POST para `/api/v1/interviews/{id}/consent` com:
   - `consentGiven: true`
   - `consentTextHash`: SHA-256 do texto exibido
5. Backend valida que o hash corresponde à versão atual do texto de consentimento
6. Registro salvo em `recording_consents` com timestamp, IP e user-agent
7. A gravação só é iniciada se AMBOS os participantes deram consentimento

**Fluxo de Recusa:**
- Se qualquer participante recusar, a entrevista ocorre normalmente sem gravação
- O sistema exibe aviso claro: "A entrevista ocorrerá sem gravação. O feedback automático por IA não estará disponível."
- Feedback pode ser inserido manualmente pelo recrutador após a entrevista

## 8.4 Conformidade com a LGPD

| Aspecto LGPD | Implementação |
|---|---|
| **Base legal** | Consentimento explícito (Art. 7º, I) para gravações; Legítimo interesse para dados de processo seletivo |
| **Dados coletados** | Nome, e-mail, CPF (candidato), áudio/vídeo (com consentimento), transcrição, avaliações de competências |
| **Finalidade** | Processo seletivo, desenvolvimento profissional, análise de qualidade do CIEE |
| **Tempo de retenção** | Gravações: 6 meses; Feedbacks: 2 anos (anonimizados após); Eventos de auditoria: indefinido |
| **Direito de acesso (Art. 18, I)** | Endpoint `GET /api/v1/privacy/my-data` retorna todos os dados do titular em JSON |
| **Direito de exclusão (Art. 18, VI)** | Endpoint `DELETE /api/v1/privacy/my-data` inicia processo de anonimização em 30 dias |
| **Portabilidade (Art. 18, V)** | Export em JSON/CSV disponível para candidatos |
| **DPO** | Informar contato do DPO na plataforma e na política de privacidade |
| **Transferência internacional** | Claude API (Anthropic, EUA): incluir cláusulas contratuais padrão; dados enviados são anonimizados (sem nome/CPF) |

## 8.5 Auditoria e Rastreabilidade

Toda ação relevante gera um evento no Event Store (`domain_events`) com:
- Quem fez (`userId`)
- O quê fez (`event_type`)
- Quando (`occurred_at`)
- De onde (`metadata.ip`, `metadata.userAgent`)
- ID de correlação para rastrear fluxos distribuídos (`metadata.correlationId`)

Logs de auditoria são imutáveis (sem UPDATE/DELETE na tabela `domain_events`). Acesso ao Event Store é restrito a `PLATFORM_ADMIN` e ao processo automatizado de relatórios.

## 8.6 Análise de Riscos de Segurança

| Risco | Mitigação |
|---|---|
| Acesso não autorizado a gravações | Signed URLs com expiração de 1h para download; sem URLs públicas permanentes |
| Replay de token de entrevista | Tokens de entrevista são invalidados após uso (one-time para início da sessão) |
| Injeção de prompt (LLM) | Sanitizar transcrições antes de incluir em prompts; limitar tamanho do input; usar delimitadores explícitos |
| Enumeração de IDs | Todos os IDs são UUID v4 (não sequenciais) |
| IDOR (Insecure Direct Object Reference) | Toda requisição valida que o recurso pertence ao tenant e ao usuário autenticado |
| Vazamento de dados entre tenants | Schema isolation + testes de penetração em cada release |
| Ataque de gravação sem consentimento | Verificação server-side do consentimento antes de aceitar qualquer chunk de upload |

---

> ⚠️ **Pontos de Atenção — Seção 8**
> - Antes do go-live, realizar um DPIA (Data Protection Impact Assessment) formal, exigido pela LGPD para tratamentos de dados em larga escala.
> - O envio de transcrições para Claude API (servidor nos EUA) configura transferência internacional de dados — garantir que transcrições sejam anonimizadas (sem nome, CPF, dados identificáveis) antes do envio.
> - Implementar testes de penetração (pentest) a cada 6 meses, com foco em isolamento multi-tenant e segurança do fluxo de consentimento.

---

# 9. Roadmap de Implementação

## Fase 1 — MVP (12 semanas / ~120 story points)

**Objetivo:** Sistema funcional para entrevistas e feedback básico.

**Entregáveis:**
- Módulo de agendamento de entrevistas (sem integração de calendário externo)
- Videochamada P2P funcional (PeerJS/WebRTC) para 2 participantes
- Gravação de entrevista com consentimento básico
- Transcrição via Whisper (modelo `tiny` no desenvolvimento, `medium` em produção)
- Análise e feedback via Mistral 3B (Claude API como opcional/configurável)
- Fluxo de revisão e aprovação de feedback pelo recrutador
- Notificações por e-mail (agendamento, lembrete, feedback pronto)
- Autenticação JWT; roles básicos (CANDIDATE, RECRUITER, CIEE_MANAGER)
- Multi-tenancy básico (schema-per-tenant com 2 tenants iniciais)
- Infraestrutura Docker Compose para desenvolvimento

**Critérios de Aceite:**
- Entrevista de 30min pode ser realizada de ponta a ponta sem erros críticos
- Feedback gerado em até 30min após término da entrevista (target < 2h em produção)
- Consentimento de gravação registrado e auditável
- Zero vazamento de dados entre tenants (teste manual)

**Estimativa de Esforço por Frente:**
- Backend (agendamento + sessão): 35 SP
- Frontend (videochamada + UX): 30 SP
- AI Worker (Whisper + Mistral): 25 SP
- Infraestrutura + DevOps: 15 SP
- QA + Ajustes: 15 SP

**Dependências:** Definição do tenant inicial, chave Claude API, servidor GPU disponível.

---

## Fase 2 — Expansão (8 semanas / ~80 story points)

**Objetivo:** Completar o Módulo 2 e melhorar robustez do Módulo 1.

**Entregáveis:**
- Plataforma de feedback contínuo (bidirecional) completa
- Análise de sentimento em feedbacks contínuos
- Assistente de redação de feedback (Claude API)
- PDI básico (geração manual com suporte de IA)
- Dashboard básico para gestores (métricas de feedbacks, sentimento geral)
- Fallback P2P (servidor TURN)
- Reagendamento e cancelamento de entrevistas
- Rate limiting completo (todas as camadas)
- Event Sourcing completo (domain_events)

**Critérios de Aceite:**
- Ciclo completo de feedback bidirecional funcional
- Assistente de redação gera sugestões em menos de 5 segundos
- Dashboard exibe dados de até 90 dias sem degradação de performance
- Rate limiting bloqueia corretamente acesso excessivo

**Dependências:** Fase 1 concluída; definição das competências avaliadas com o negócio.

---

## Fase 3 — Inteligência Avançada (10 semanas / ~100 story points)

**Objetivo:** Melhorar qualidade de IA e adicionar features preditivas.

**Entregáveis:**
- PDI automatizado (gerado por IA com base em histórico de feedbacks)
- Dashboard preditivo com score de risco de evasão
- Análise de padrões cross-tenant para gestores CIEE (dados agregados/anonimizados)
- Integração com Claude API para feedbacks complexos (com fallback inteligente)
- Melhoria da identificação de turnos de fala na transcrição
- 2FA para administradores
- Módulo de exportação de dados (LGPD - portabilidade)
- Processo automatizado de anonimização (LGPD - exclusão)

**Critérios de Aceite:**
- Score de risco de evasão com acurácia validada em dados históricos (>70% de precisão)
- PDI gerado automaticamente revisado e aprovado por amostra de gestores CIEE (>80% de aprovação)
- Processo de exclusão de dados LGPD auditado por equipe jurídica

**Dependências:** Fase 2 concluída; dados históricos de 3+ meses de feedbacks reais.

---

## Fase 4 — Escala (Ongoing / Backlog)

**Objetivo:** Preparar a plataforma para crescimento e novos casos de uso.

**Entregáveis:**
- Migração para Kubernetes (EKS/GKE) com auto-scaling
- Integração com calendários externos (Google Calendar, Outlook)
- Entrevistas com mais de 2 participantes (painel de entrevistas)
- API pública para integração com ATSs de empresas parceiras
- Análise de linguagem não-verbal (fase experimental com OpenCV/MediaPipe)
- Suporte a múltiplos idiomas (Inglês em seguida ao Português)
- Aplicativo mobile web progressivo (PWA)

**Dependências:** Fase 3 concluída; validação de negócio das features adicionais.

---

> ⚠️ **Pontos de Atenção — Seção 9**
> - O modelo de risco de evasão (Fase 3) requer dados históricos que só existirão após 6+ meses de uso real — não lançar como feature definitiva antes de validar a acurácia.
> - Incluir no planejamento da Fase 1 pelo menos 2 sessões de testes de usabilidade com candidatos reais antes do go-live.
> - O cronograma acima assume um time completo; com time reduzido, estimar expansão de 40-60% nos prazos.

---

# 10. Métricas de Sucesso e KPIs

## 10.1 KPIs Quantitativos

| KPI | Linha de Base | Meta (6 meses) | Meta (12 meses) | Método de Medição |
|---|---|---|---|---|
| Taxa de feedbacks entregues pós-entrevista | ~40% (processo manual) | 85% | 95% | (entrevistas com feedback aprovado) / (total de entrevistas concluídas) |
| Tempo médio de entrega do feedback | 5-10 dias úteis | < 4 horas | < 2 horas | Média de `feedback.released_at - interview.ended_at` |
| Taxa de sucesso da conexão P2P | — | > 92% | > 95% | (conexões P2P estabelecidas) / (tentativas de conexão) |
| Precisão da transcrição Whisper (WER) | — | < 15% WER | < 10% WER | Word Error Rate em amostra de 100 entrevistas |
| Aprovação de feedbacks de IA sem edição | — | > 60% | > 75% | (feedbacks aprovados sem edições) / (total de feedbacks revisados) |
| Tempo de processamento de IA | — | < 20min (45min entrevista) | < 15min | Mediana de `feedback.generated_at - interview.ended_at` |
| Taxa de uso do assistente de redação | — | > 30% dos feedbacks | > 50% | (feedbacks usando assistente) / (total de feedbacks manuais) |
| Score médio de sentimento de feedbacks | — | Baseline (mês 3) | Crescimento 10% positivos | Média mensal da distribuição de sentimentos |
| Uptime da plataforma | — | 99,5% | 99,9% | Monitoramento contínuo (Uptime Robot / Datadog) |
| NPS dos candidatos | — | > 30 | > 50 | Pesquisa mensal pós-entrevista (n > 50) |

## 10.2 KPIs Qualitativos

| KPI | Como será avaliado | Frequência |
|---|---|---|
| Qualidade percebida do feedback de IA | Escala 1-5 pelos recrutadores ao revisar feedbacks; meta: média > 3.8 | Mensal |
| Satisfação dos recrutadores com a plataforma | SUS (System Usability Scale); meta: score > 70 | Trimestral |
| Percepção de privacidade pelos candidatos | Pesquisa com 3 perguntas sobre confiança; meta: >80% "confiam" | Semestral |
| Utilidade dos PDIs | Taxa de check-ins concluídos; meta: >60% dos PDIs com ao menos 1 check-in em 30 dias | Mensal |

## 10.3 Frequência de Monitoramento

- **Tempo real:** Uptime, latência de API, taxa de erros, tamanho de filas RabbitMQ
- **Diário:** Processamentos de IA concluídos/falhados, consumo de quota Claude API, feedbacks pendentes há mais de 4h
- **Semanal:** KPIs de taxa de feedback, tempo médio de entrega, NPS parcial
- **Mensal:** Relatório completo de KPIs, análise de tendências, revisão de alertas

## 10.4 Dashboard de Métricas

**Para Gestores CIEE (Módulo 2):**
- Quantidade de entrevistas por período (gráfico de barras)
- Distribuição de sentimentos de feedbacks (pie chart)
- Ranking de empresas por NPS
- Mapa de calor de engajamento por área
- Lista de candidatos com score de risco alto (tabela com alertas)

**Para Administradores da Plataforma:**
- Métricas de infraestrutura (CPU, memória, latência de banco)
- Consumo de API Claude (gauge com threshold visual)
- Tamanho das filas RabbitMQ em tempo real
- Taxa de erros por serviço (últimas 24h)
- Eventos de auditoria recentes (últimas ações críticas)

---

> ⚠️ **Pontos de Atenção — Seção 10**
> - Coletar baseline de todos os KPIs quantitativos nos primeiros 30 dias de operação antes de definir metas definitivas.
> - O WER (Word Error Rate) do Whisper deve ser medido especificamente em entrevistas de estágio em português, pois pode variar muito por área de conhecimento e sotaque regional.

---

# 11. Análise de Riscos

## 11.1 Tabela de Riscos

| # | Risco | Probabilidade | Impacto | Mitigação | Responsável |
|---|---|---|---|---|---|
| R01 | Qualidade insatisfatória da transcrição Whisper em PT-BR | Alta | Alto | Testar modelos `medium` e `large-v3`; coletar feedback de usuários; permitir correção manual | Tech Lead IA |
| R02 | Falha na conexão P2P em redes corporativas restritivas | Média | Alto | Implementar servidor TURN; teste com VPNs comuns; instruções de configuração de rede | DevOps |
| R03 | Custo da Claude API exceder orçamento | Média | Alto | Rate limiting desde o MVP; fallback para Mistral; alertas em 60%/80%/95% da quota | Product Owner |
| R04 | Não conformidade com LGPD | Baixa | Crítico | DPIA antes do lançamento; revisão jurídica do fluxo de consentimento; DPO designado | Jurídico + Tech Lead |
| R05 | Baixa adoção pelos recrutadores (resistência à mudança) | Alta | Médio | Treinamento; UX simplificada; período de transição paralelo ao processo manual | Product Owner + CS |
| R06 | Vazamento de dados entre tenants | Baixa | Crítico | Testes de isolamento em CI/CD; pentest; code review obrigatório em queries | Tech Lead Backend |
| R07 | Latência de processamento de IA acima do SLA | Média | Médio | GPU dedicada; otimização de chunks de áudio; alertas de fila | DevOps + Tech Lead IA |
| R08 | Feedback de IA com viés ou conteúdo inadequado | Média | Alto | Revisão obrigatória por humano antes de liberar ao candidato; logs de todos os outputs | Tech Lead IA + PO |
| R09 | Indisponibilidade do servidor de sinalização P2P | Baixa | Alto | Alta disponibilidade (2 instâncias); health checks; restart automático | DevOps |
| R10 | Candidatos sem equipamento adequado (câmera/microfone) | Alta | Médio | Teste de dispositivos pré-entrevista; suporte técnico; opção de entrevista por áudio apenas | CS / UX |

## 11.2 Plano de Contingência — Top 5 Riscos Críticos

**R01 — Transcrição de baixa qualidade:**
- Contingência imediata: feedback baseado apenas em notas do recrutador (manual)
- Longo prazo: testar `whisper-large-v3` e avaliar fine-tuning com dados próprios de entrevistas CIEE
- Indicador de disparo: WER > 20% em amostra semanal de 20 entrevistas

**R02 — Falha P2P em redes corporativas:**
- Contingência imediata: ativar servidor TURN (Coturn) automaticamente
- Se TURN também falhar: oferecer link para videochamada via Google Meet / Zoom como failsafe temporário
- Indicador de disparo: taxa de falha P2P > 8% em uma semana

**R03 — Custo Claude API:**
- Contingência imediata: desativar Claude API; operar 100% com Mistral 3B + assistente de redação desabilitado
- Negociar com Anthropic pacote empresarial com desconto por volume
- Indicador de disparo: quota diária 80% consumida até 12h (metade do dia)

**R04 — Não conformidade LGPD:**
- Contingência imediata: suspender novos agendamentos; notificar ANPD; acionar DPO
- Pré-launch: DPIA e revisão jurídica são gates obrigatórios antes do go-live da Fase 1
- Indicador de disparo: qualquer incidente de segurança que exponha dados pessoais

**R06 — Vazamento entre tenants:**
- Contingência imediata: suspender acesso à plataforma; análise forense via Event Store; notificar tenants afetados
- Pré-launch: testes automatizados de isolamento em pipeline CI/CD como gate de qualidade
- Indicador de disparo: qualquer query retornando dados de tenant diferente do autenticado

---

> ⚠️ **Pontos de Atenção — Seção 11**
> - R08 (viés de IA) é frequentemente subestimado; o feedback automático pode refletir vieses presentes nos dados de treinamento dos LLMs. A revisão humana obrigatória antes da liberação ao candidato é uma salvaguarda não negociável no MVP.
> - R05 (resistência dos recrutadores) é o risco de negócio mais provável; planejar um programa estruturado de change management com o time de CS antes do lançamento.

---

# 12. Glossário

| Termo | Definição |
|---|---|
| **AI Worker** | Serviço Python/FastAPI responsável por executar o pipeline de inteligência artificial: extração de áudio, transcrição (Whisper), análise (Mistral/Claude) e publicação de resultados via RabbitMQ. |
| **ASR** | Automatic Speech Recognition — reconhecimento automático de fala; tecnologia que converte áudio em texto. No sistema, implementado via Whisper. |
| **CQRS** | Command Query Responsibility Segregation — padrão arquitetural que separa operações de escrita (Commands) de operações de leitura (Queries), otimizando cada uma separadamente. |
| **Dead-Letter Queue (DLQ)** | Fila especial no RabbitMQ que recebe mensagens que não puderam ser processadas com sucesso após o número máximo de tentativas. |
| **DTLS** | Datagram Transport Layer Security — protocolo de criptografia utilizado pelo WebRTC para cifrar os dados de mídia transmitidos entre peers. |
| **Event Sourcing** | Padrão arquitetural onde o estado do sistema é derivado de uma sequência imutável de eventos armazenados. Permite auditoria completa e reconstrução de estados passados. |
| **FFmpeg** | Ferramenta open-source de processamento de mídia utilizada no AI Worker para extrair o áudio do arquivo WebM da gravação. |
| **HPA** | Horizontal Pod Autoscaler — recurso do Kubernetes que escala automaticamente o número de réplicas de um deployment com base em métricas de CPU/memória/customizadas. |
| **ICE Candidate** | Interactive Connectivity Establishment — mecanismo do WebRTC para descobrir os melhores caminhos de rede entre dois peers, incluindo endereços IP diretos, reflexivos (NAT) e relay (TURN). |
| **IDOR** | Insecure Direct Object Reference — vulnerabilidade de segurança onde um usuário consegue acessar objetos de outro usuário manipulando IDs em requisições. |
| **JWT** | JSON Web Token — padrão aberto (RFC 7519) para transmissão segura de informações entre partes como objeto JSON assinado digitalmente. |
| **LGPD** | Lei Geral de Proteção de Dados (Lei 13.709/2018) — legislação brasileira que regula o tratamento de dados pessoais, análoga ao GDPR europeu. |
| **LLM** | Large Language Model — modelo de linguagem de grande escala treinado em vastos conjuntos de dados textuais, capaz de gerar e analisar texto. No sistema: Mistral 3B e Claude API. |
| **MediaRecorder API** | API nativa do browser que permite gravar streams de mídia (áudio/vídeo) capturadas pela câmera e microfone do usuário. |
| **Mistral 3B** | Modelo de linguagem open-source com 3 bilhões de parâmetros, utilizado localmente no AI Worker para análise de transcrições e análise de sentimento. |
| **Multi-tenancy** | Arquitetura de software onde uma única instância da aplicação serve múltiplos clientes (tenants), com isolamento de dados entre eles. |
| **NAT** | Network Address Translation — técnica de roteamento que traduz endereços IP privados em públicos, comum em redes corporativas e domésticas, podendo dificultar conexões P2P. |
| **NPS** | Net Promoter Score — métrica de lealdade e satisfação do cliente baseada na pergunta "Em uma escala de 0-10, qual a probabilidade de recomendar?". |
| **Ollama** | Plataforma open-source para executar modelos de linguagem localmente, utilizada para servir o Mistral 3B no AI Worker. |
| **P2P** | Peer-to-Peer — arquitetura de comunicação onde os dados fluem diretamente entre os participantes sem passar por um servidor intermediário. |
| **PDI** | Plano de Desenvolvimento Individual — documento personalizado que define metas de desenvolvimento profissional, ações e prazo para um colaborador ou estagiário. |
| **PeerJS** | Biblioteca JavaScript que simplifica o uso de WebRTC para conexões P2P, abstraindo a troca de SDP e ICE candidates. |
| **Prompt Engineering** | Prática de elaborar instruções (prompts) para modelos de linguagem de forma a obter respostas mais precisas, relevantes e bem formatadas. |
| **RBAC** | Role-Based Access Control — modelo de controle de acesso baseado em papéis (roles) atribuídos aos usuários. |
| **Rate Limiting** | Mecanismo que limita o número de requisições que um usuário ou sistema pode fazer em um período de tempo, prevenindo abuso e controlando custos. |
| **SBI** | Situação-Comportamento-Impacto — framework para dar feedback construtivo descrevendo o contexto específico, o comportamento observado e o impacto que causou. |
| **SDP** | Session Description Protocol — formato usado no WebRTC para negociar parâmetros de sessão (codecs, capacidades de rede) entre peers. |
| **SRTP** | Secure Real-time Transport Protocol — protocolo que fornece criptografia, autenticação e proteção ao stream de mídia em comunicações WebRTC. |
| **SUS** | System Usability Scale — questionário padronizado de 10 itens para avaliar a usabilidade percebida de um sistema. Score > 70 indica boa usabilidade. |
| **Tenant** | No contexto de multi-tenancy, representa uma organização ou unidade de negócio com dados completamente isolados dos demais (ex: uma empresa parceira ou unidade regional do CIEE). |
| **TURN** | Traversal Using Relays around NAT — servidor de relay que atua como intermediário quando uma conexão P2P direta não é possível devido a NATs ou firewalls restritivos. |
| **UUID v4** | Universally Unique Identifier versão 4 — identificador de 128 bits gerado aleatoriamente, utilizado como chave primária de todos os recursos do sistema. |
| **WebM** | Formato de vídeo open-source baseado em VP8/VP9, gerado pelo MediaRecorder API nos navegadores modernos. |
| **WebRTC** | Web Real-Time Communication — conjunto de APIs e protocolos que permitem comunicação de áudio, vídeo e dados em tempo real diretamente entre navegadores, sem plugins. |
| **WER** | Word Error Rate — métrica de qualidade para sistemas de reconhecimento de fala; percentual de palavras incorretas na transcrição em relação ao texto de referência. |
| **Whisper** | Modelo de reconhecimento de fala (ASR) open-source desenvolvido pela OpenAI, utilizado no sistema para transcrever gravações de entrevistas. |
| **XSS** | Cross-Site Scripting — vulnerabilidade que permite injeção de scripts maliciosos em páginas web vistas por outros usuários. |

---

*Documento gerado em Fevereiro/2026 | Versão 1.0.0 | CIEE — Plataforma de Entrevistas Online com Feedback Inteligente | Ticket PK-28240*

*Este documento deve ser revisado a cada início de fase de implementação para refletir decisões tomadas durante o desenvolvimento.*
