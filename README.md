# Arquitetura ConsumerIA

## Introdução

Este documento detalha a arquitetura proposta para uma solução avançada de Inteligência Artificial (IA) que atua como consumidor real. O objetivo é fornecer um serviço automatizado abrangente, capaz de:

- Simular com precisão comportamentos reais e variados de consumidores.
- Navegar e interagir com plataformas digitais de maneira natural.
- Avaliar a eficácia dos fluxos de compra, atendimento e experiência do usuário.
- Monitorar e medir tempos de resposta, desempenho e qualidade da interação.
- Operar com diversos perfis e cenários de consumo para uma análise completa.

Este sistema visa gerar insights detalhados e contínuos sobre experiências de consumo, contribuindo diretamente para melhorias estratégicas em processos, produtos, serviços e atendimento ao cliente.

---

## TL;DR

Um agente inteligente que simula consumidores reais para interagir com plataformas digitais, testar fluxos, avaliar concorrentes e parceiros, medir performance, UX e atendimento, além de coletar dados estratégicos de forma contínua para gerar inteligência de produto e mercado.

---

## Proposta de Solução

A ConsumerIA opera a partir de perfis configuráveis de consumidores, simulando visitas reais e interações com sites, aplicativos ou plataformas digitais. A cada requisição, o agente executa ações predeterminadas — como navegar por fluxos de compra, acionar o atendimento, testar respostas automatizadas ou comparar produtos. Ao final do processo, a IA gera um relatório estruturado com métricas de desempenho, registros de interação e insights estratégicos sobre usabilidade, atendimento e comportamento do ambiente analisado.

---

## Fluxo Operacional Simplificado

```jsx
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   GPT Interface │────▶│  Mission Queue  │────▶│ Browser Service │
│   (Requisição)  │     │  (Async Proc)   │     │  (Navigation)   │
└─────────────────┘     └─────────────────┘     └─────────────────┘
                                                          │
┌─────────────────┐     ┌─────────────────┐              │
│ Report Service  │◀────│  AI Decision    │◀─────────────┘
│  (Intelligence) │     │    Engine       │
└─────────────────┘     └─────────────────┘
```

### Descritivo:

O funcionamento da ConsumerIA é organizado em etapas sequenciais, combinando controle por microserviços com simulação realista de consumo. 

1. **Recebimento da Requisição**
    
    O `ConsumerController` recebe a solicitação contendo o perfil do consumidor, a URL alvo e os objetivos da missão (ex: testar o checkout, avaliar o suporte, comparar preços).
    
2. **Execução da Simulação**
    
    O `NavigationService` inicia uma sessão autônoma em navegador headless, simulando a navegação de um consumidor real — incluindo busca por produtos, login, rolagem de página, clique em botões, preenchimento de formulários, etc.
    
3. **Interação com Canais de Atendimento**
    
    O `InteractionService` conduz interações com chatbots, formulários ou canais como WhatsApp. Mede o tempo de resposta, qualidade da interação e possíveis falhas ou scripts mal configurados.
    
4. **Coleta de Dados e Cálculo de Métricas**
    
    O `PerformanceScoringService` coleta dados estruturados e não estruturados (tempo de carregamento, códigos HTTP, mensagens recebidas, screenshots, etc.) e aplica algoritmos de scoring para gerar KPIs de UX e atendimento.
    
5. **Geração e Retorno do Relatório**
    
    O relatório final, contendo métricas e insights estratégicos, é retornado ao solicitante. Simultaneamente, os dados são publicados para auditoria e análises comparativas futuras.
    
6. **Comunicação Assíncrona e Tratamento de Erros:**
A `MissionEventQueue` utiliza uma tecnologia de mensageria robusta e escalável, como RabbitMQ ou Kafka, para garantir a entrega dos relatórios. O `ConsumerController` captura exceções e falhas, registrando-as no `MissionReport` e implementa um timeout global para a missão. A orquestração dos serviços é sequencial, pois a interação depende da navegação.

## Componentes da Arquitetura

- **ConsumerController:** Este é o componente de entrada da arquitetura, responsável por coordenar a lógica da missão. Ele recebe requisições HTTP contendo os detalhes de uma simulação (definidos no `ConsumerRequestDTO`), orquestra a chamada para os demais serviços (`NavigationService`, `InteractionService`, `PerformanceScoringService`) e consolida os resultados em um `MissionReport`. Sua responsabilidade é garantir que o fluxo operacional seja executado de ponta a ponta, de forma controlada e rastreável. Após o processamento, ele publica o resultado em uma fila assíncrona para auditoria e persistência.
- **NavigationService:** Responsável por simular o comportamento de navegação do usuário em um ambiente virtual. Utiliza um navegador headless para acessar a URL alvo, interagir com elementos da página (botões, campos, links), preencher formulários e seguir o fluxo de navegação conforme definido na missão. Este serviço é crucial para replicar a experiência real do usuário em diferentes plataformas web ou mobile.
- **InteractionService:** Componente focado na comunicação com canais de atendimento, como chatbots e sistemas de suporte. Ele é capaz de interpretar o contexto da conversa e gerar respostas realistas. Durante a interação, mede a latência de resposta do chatbot, avalia a precisão das informações e identifica possíveis erros de roteiro ou falhas na comunicação.
- **PerformanceScoringService:** Este serviço é o núcleo analítico da arquitetura. Ele coleta uma vasta gama de dados durante a simulação (tempos de carregamento de página, screenshots, logs de interação, respostas de APIs, etc.) e os processa para gerar métricas de desempenho e risco. Utilizando algoritmos de scoring, ele atribui notas para a experiência do usuário (UX), tempo de resposta e eficácia do atendimento.
- **ProfileConfigService:** A camada de serviço responsável por interagir com o banco de dados. Ele tem métodos para buscar a versão mais recente e ativa de um perfil, garantindo que as versões ativas de cada perfil estejam sempre disponíveis para consulta rápida por meio de um CacheService.

## **Diagrama de Arquitetura**

O diagrama a seguir ilustra o fluxo de dados e a interação dos principais componentes da arquitetura ConsumerIA. Ele foi projetado para demonstrar como uma requisição de missão é processada de forma síncrona e como os dados de resultado são tratados de forma assíncrona.

```jsx
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              CONSUMER AI AGENT                                 │
│                                                                                 │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐                 │
│  │   GPT Interface │  │ Webhook Handler │  │  Admin API      │                 │
│  │   Controller    │  │ (WhatsApp/Email)│  │  (Management)   │                 │
│  └─────────┬───────┘  └─────────┬───────┘  └─────────┬───────┘                 │
│            │                    │                    │                         │
│  ┌─────────▼─────────────────────▼────────────────────▼─────────┐                │
│  │                Consumer Agent Controller                    │                │
│  │              (Orchestration & Mission Control)             │                │
│  └─────────────────────────┬───────────────────────────────────┘                │
│                            │                                                   │
│  ┌─────────────────────────▼───────────────────────────────────┐                │
│  │                    Mission Event Queue                     │                │
│  │                  (Bull/BullMQ - Async)                     │                │
│  └─────────────────────────┬───────────────────────────────────┘                │
│                            │                                                   │
│  ┌─────────────────────────▼───────────────────────────────────┐                │
│  │                   CORE SERVICES LAYER                      │                │
│  │                                                             │                │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────┐   │                │
│  │  │Browser Automation│  │MultiChannel     │  │AI Decision  │   │                │
│  │  │Service          │  │Communicator     │  │Engine       │   │                │
│  │  │(Playwright)     │  │(WhatsApp/Email) │  │(GPT-4/Claude│   │                │
│  │  └─────────┬───────┘  └─────────┬───────┘  └─────────┬───┘   │                │
│  │            │                    │                    │       │                │
│  │  ┌─────────▼───────┐  ┌─────────▼───────┐  ┌─────────▼───┐   │                │
│  │  │Anti-Detection   │  │Report Generation│  │Competitor   │   │                │
│  │  │Service          │  │Service          │  │Analysis     │   │                │
│  │  └─────────────────┘  └─────────────────┘  └─────────────┘   │                │
│  └─────────────────────────────────────────────────────────────┘                │
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐                │
│  │                   SUPPORT SERVICES                         │                │
│  │                                                             │                │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────┐   │                │
│  │  │Persona Config   │  │Cache Service    │  │Tenant       │   │                │
│  │  │Service          │  │(Redis)          │  │Management   │   │                │
│  │  └─────────────────┘  └─────────────────┘  └─────────────┘   │                │
│  └─────────────────────────────────────────────────────────────┘                │
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐                │
│  │                    DATA & ML LAYER                         │                │
│  │                                                             │                │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────┐   │                │
│  │  │PostgreSQL       │  │Feedback Loop    │  │Model        │   │                │
│  │  │(Prisma ORM)     │  │Job Processor    │  │Registry     │   │                │
│  │  └─────────────────┘  └─────────────────┘  └─────────────┘   │                │
│  └─────────────────────────────────────────────────────────────┘                │
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐                │
│  │                 OBSERVABILITY LAYER                        │                │
│  │                                                             │                │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────┐   │                │
│  │  │Winston Logging  │  │Prometheus       │  │Health Check │   │                │
│  │  │(Structured)     │  │Metrics          │  │Service      │   │                │
│  │  └─────────────────┘  └─────────────────┘  └─────────────┘   │                │
│  └─────────────────────────────────────────────────────────────┘                │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Stack Tecnológica

**Backend Core**: Node.js + TypeScript + Express

- Performance superior para I/O intensivo
- Ecossistema maduro para automação web
- TypeScript garante type safety em sistema complexo

**Browser Automation**: Playwright + stealth plugins

- Melhor anti-detection nativo do mercado
- Suporte completo a SPA modernas
- APIs assíncronas otimizadas para performance

**AI/LLM Integration**: OpenAI GPT-4 + Anthropic Claude

- GPT-4 para decisões complexas e análise
- Claude para conversação multicanal natural
- Fallback entre provedores para reliability

**Caching & Performance**: Redis

- Sub-millisecond latency para configurações
- Session management para personas
- Rate limiting distribuído

**Message Queue**: Bull/BullMQ

- Processamento assíncrono confiável
- Retry logic avançado
- Dashboard nativo para monitoring

**Database**: PostgreSQL + Prisma ORM

- ACID compliance para dados críticos
- Type-safe queries com Prisma
- Escalabilidade horizontal via partitioning

**Communication**: WhatsApp Business API + Nodemailer

- Integração nativa com WhatsApp Business
- Templates de email contextuais
- Análise de sentiment das respostas

**Monitoring**: Winston + Prometheus

- Logging estruturado para debugging
- Métricas customizadas de negócio
- Integração com Grafana/AlertManager

## Funcionalidades Chave

### Feedback Loop

O sistema ConsumerIA implementa um mecanismo de feedback contínuo para aprender com os resultados reais das simulações. Esse processo permite refinar o comportamento dos agentes, calibrar parâmetros de scoring e adaptar as estratégias de navegação de forma ágil. O `FeedbackLoopJob` processa relatórios de missões concluídas e utiliza o `Gemini 2.5 Pro` para gerar insights que são traduzidos em ajustes automáticos nas configurações dos perfis.

Toda a análise e adaptação do sistema ocorre de forma assíncrona, consumindo dados da `MissionEventQueue` e do `Database`, garantindo que o fluxo síncrono de avaliação de missões não seja impactado.

**Ajuste Dinâmico de Parâmetros de Perfil**
O objetivo é ter uma arquitetura adaptativa, onde os parâmetros de simulação de cada perfil de consumidor são ajustados automaticamente com base na performance real. Isso garante que as simulações se tornem cada vez mais realistas e eficazes.

| Comportamento | Consequência | Consequência para a Simulação |
| --- | --- | --- |
| Simulações legítimas que falham frequentemente em um checkout complexo. | Aumento do `delay` entre os passos de navegação e aumento do tempo de espera por elementos. | Maior taxa de sucesso em fluxos de navegação complexos, tornando a simulação mais robusta. |
| Altas taxas de erro (4xx, 5xx) em requisições feitas por um perfil específico. | Redução da velocidade de navegação e adição de verificações de status de API. | O agente se torna mais cauteloso, refletindo um comportamento de usuário real que recarregaria a página. |
| O tempo de resposta do chatbot é consistentemente alto para um `targetUrl`. | Ajuste do parâmetro de `timeout` da interação para que a missão não falhe prematuramente. | A avaliação da qualidade do suporte se torna mais justa, permitindo que a missão seja concluída mesmo com latência elevada. |
| O `UXScore` de um perfil está sempre baixo, mas a equipe de produto não consegue replicar o problema. | Geração de um "cenário de ajuste" que prioriza a captura de screenshots e logs mais detalhados para aquela URL. | Maior observabilidade em áreas problemáticas, fornecendo à equipe de produto dados mais ricos para depuração. |

### Perfis Dinâmicos

A arquitetura do ConsumerIA foi concebida para operar com perfis de consumidores dinâmicos e altamente configuráveis. Isso permite que a solução simule uma ampla gama de cenários de consumo, de usuários novos a clientes frequentes, de forma a obter uma análise abrangente e representativa da base de clientes real.

A configuração de cada perfil é encapsulada em uma entidade versionada chamada `ConsumerProfileConfig`. Este objeto é armazenado em um banco de dados e gerenciado por um `ProfileConfigService`, que garante que as versões ativas de cada perfil estejam sempre disponíveis para consulta rápida por meio de um `CacheService`.

**Componentes e Parâmetros Chave:**
Cada `ConsumerProfileConfig` contém os parâmetros essenciais para guiar a simulação. Esses valores podem ser ajustados via feedback loop ou manualmente, garantindo a adaptabilidade da plataforma.

```jsx
interface ConsumerProfileConfig {
  profileId: string; // Ex: "new-user-br", "frequent-shopper-us"
  version: number;
  status: 'ACTIVE' | 'AB_TEST' | 'DEPRECATED';
  description: string;
  // Parâmetros de navegação
  navigationSpeed: 'FAST' | 'MEDIUM' | 'SLOW';
  minClickDelayMs: number; // Atraso mínimo entre cliques
  maxClickDelayMs: number; // Atraso máximo entre cliques
  // Parâmetros de interação
  chatStyle: 'FORMAL' | 'INFORMAL'; // Estilo de interação do chatbot
  maxChatTurns: number; // Limite de interações
  // Pesos para o scoring
  uxWeight: number; // Peso do UXScore no resultado final
  supportWeight: number; // Peso do SupportScore no resultado final
}
```

O `ConsumerController` consulta o `ConsumerProfileConfig` no início de cada missão para obter as diretrizes de navegação, a estratégia de interação com o chatbot e os pesos que devem ser aplicados no cálculo do `MissionReport`.

| Conceito | Descrição |
| --- | --- |
| Estratégia de navegação | Define a velocidade e os padrões de clique para simular usuários mais cautelosos ou mais apressados. |
| Estratégia de interação | Personaliza o estilo de linguagem e o número de turnos da conversa com chatbots. |
| Pesos de scoring | Define a importância de cada métrica (UX, Suporte, etc.) no score final, permitindo que a análise se concentre no que é mais relevante para o cliente. |

### Observabilidade em Tempo Real

A arquitetura do ConsumerIA é construída com um forte foco em observabilidade, garantindo que cada missão seja rastreável e que o desempenho do sistema possa ser monitorado continuamente. Tracing e logging detalhados são implementados para fornecer uma visão clara do que acontece em cada etapa do fluxo, permitindo que a equipe técnica identifique gargalos, depure falhas rapidamente e assegure a qualidade das simulações e a confiabilidade dos insights.

- **Arquitetura de Logging:** Todos os componentes do sistema geram logs estruturados (em formato JSON, por exemplo), facilitando a indexação e a busca em plataformas de monitoramento (como ELK Stack ou Datadog). Os logs incluem metadados essenciais como `missionId`, `profileId`, `component` e `executionTime`, o que permite reconstruir a linha do tempo de uma missão específica.
- **Tracing e Métricas:** O `tracing` de ponta a ponta é aplicado para visualizar o fluxo completo de uma requisição, desde o `ConsumerController` até o `PerformanceScoringService`. Isso ajuda a identificar latências em cada serviço. Além disso, as seguintes métricas-chave são coletadas e monitoradas:
    - **Métricas de Performance:**
        - **Taxa de Sucesso/Falha da Missão:** Percentual de missões que são concluídas com sucesso.
        - **Latência por Componente:** Tempo de execução de cada serviço (`NavigationService`, `InteractionService`, etc.).
        - **Latência Externa:** Tempo de resposta dos ambientes externos (sites, chatbots).
        - **Consumo de Recursos:** Utilização de CPU e memória dos containers/servidores.
    - **Métricas de Negócio:**
        - **UX Score Médio:** Média dos scores de experiência do usuário gerados.
        - **Support Score Médio:** Média dos scores de suporte ao cliente gerados.
        - **Ajustes Dinâmicos:** Frequência com que o `FeedbackLoopJob` realiza ajustes nos perfis.

### Benefícios

- **Detecção Proativa:** Identificação de problemas de performance ou falhas antes que afetem a qualidade das simulações.
- **Depuração Rápida:** Capacidade de isolar a causa-raiz de um erro ou de um resultado inesperado em segundos.
- **Validação da Inteligência:** Confirmação de que os ajustes feitos pelo `Feedback Loop` estão tendo o impacto esperado.
- **Análise de Tendências:** Coleta de dados históricos para identificar tendências de desempenho e regressões ao longo do tempo.
