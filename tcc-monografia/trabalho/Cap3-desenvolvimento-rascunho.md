# Rascunho de estrutura — Capítulo 3: Desenvolvimento

---

## BLOCO 1 — Árvore atual

```
DESENVOLVIMENTO
│
├── Visão Geral da Arquitetura Proposta
│     Apresenta a hierarquia IEC 62264 (Níveis 0–4) com o mapeamento dos componentes,
│     delimita o escopo implementado (Níveis 0–2) e posiciona o K8s no Nível 4
│     como destino arquitetural da proposta.
│
├── Simulador da Planta — tep-plant
│     Contextualiza a escolha do Rust e apresenta o repositório como implementação
│     do modelo matemático do TEP + servidor gRPC.
│
│   ├── Estrutura Interna
│   │     Descreve os dois crates: te-core (biblioteca do modelo) e
│   │     tennessee-eastman-service (binário + gRPC).
│   │
│   ├── Loop de Simulação
│   │     Explica o ciclo step(): integração RK4 → ControllerBank → broadcast gRPC,
│   │     e o fator de aceleração 100× (STEP_DELAY_MS=36).
│   │
│   ├── Controladores — ControllerBank
│   │     Apresenta a trait Controller, o PController proporcional e as três malhas
│   │     P do ponto de operação nominal (pressão, nível separador, nível stripper).
│   │
│   ├── Servidor gRPC
│   │     Documenta os quatro métodos do tep.proto: StreamMetrics, GetPlantStatus,
│   │     ListControllers, UpdateController.
│   │
│   └── Sistema de Distúrbios
│         Explica a ativação dos IDV1–IDV20 via variável de ambiente ou IHM,
│         com fidelidade ao FORTRAN original de Downs e Vogel.
│
├── Operator Kubernetes — plant-supervisor
│     Apresenta o Operator Go (Kubebuilder) como observador supervisório da planta,
│     operando em modo de observação pura nos experimentos realizados.
│
│   ├── O CRD PLCMachine
│   │     Descreve spec (plantAddress, variables, controllers) e status
│   │     (phase, plantTime, lastReconcile), com exemplo de manifest YAML.
│   │
│   ├── Loop de Reconciliação
│   │     Documenta as cinco etapas do Reconcile(): leitura → conexão gRPC →
│   │     observação → avaliação de política → atualização de status.
│   │
│   └── Conectividade com a Planta
│         Explica o problema de rede Kind↔Docker e a solução via
│         host.docker.internal:50051.
│
├── Interface Homem-Máquina — tep-ihm
│     Apresenta a IHM Python (FastAPI + WebSocket) como interface de operação
│     em tempo real.
│
│   ├── Arquitetura
│   │     Descreve o pipeline assíncrono: grpc_reader → asyncio.Queue →
│   │     ws_broadcaster → clientes WebSocket.
│   │
│   ├── Dashboard
│   │     Documenta os seis painéis (SVG da planta, gráficos ECharts, controladores,
│   │     distúrbios, analítico, status PLCMachine) e a conformidade com ISA-101.
│   │
│   └── Registro de Dados (CSV)
│         Explica a gravação contínua de XMEAS/XMV ativada por RECORD_CSV=true;
│         os CSVs são a fonte de dados dos experimentos do Cap. 4.
│
├── Infraestrutura Local — tep-lab
│     Reúne os arquivos de configuração para execução em Windows local.
│
│   ├── Modo de Desenvolvimento — Docker Compose
│   │     Descreve os serviços te-plant e tep-ihm no docker-compose.yml e a
│   │     rede interna que permite resolução por nome de serviço.
│   │
│   └── Modo de Produção — Cluster Kind
│         Documenta o roteiro de seis passos para subir o cluster Kind com o
│         Operator ativo, e a tabela de endereços por modo de execução.
│
└── Considerações Finais
      Sintetiza a separação de responsabilidades (planta / controle / IHM),
      o papel da PLCMachine como política declarativa, e encaminha o Cap. 4.
```

---

## BLOCO 2 — Proposta de nova estrutura (rascunho tosco)

> Ideia central: antes de descrever os componentes, contextualizar **por que** o código foi escrito em Rust (em contraste com o FORTRAN original) e **por que** a arquitetura adota OPC UA. Isso dá ao leitor o quadro conceitual antes dos detalhes técnicos.

```
DESENVOLVIMENTO
│
├── Visão Geral da Arquitetura Proposta
│     Apresenta a hierarquia IEC 62264 (Níveis 0–4) com o mapeamento dos componentes,
│     delimita o escopo implementado (Níveis 0–2) e posiciona o K8s no Nível 4
│     como destino arquitetural da proposta.
│     [duas figuras (hierarquia e fluxo de dados), escopo implementado vs. proposta]
│
├── Do FORTRAN ao Rust: mudança de paradigma
│     Seção narrativa que justifica a reimplementação do TEP em Rust.
│     Não descreve código ainda — prepara o leitor para entender as escolhas
│     das subseções seguintes.
│
│     § Parágrafo inicial: contexto histórico do FORTRAN 77.
│       Downs e Vogel (1993) escreveram o código no paradigma imperativo sequencial
│       típico da época — subrotinas globais, estado compartilhado por COMMON blocks,
│       sem tipagem forte. O objetivo deles era portabilidade e reprodutibilidade,
│       não extensibilidade. Isso foi correto para 1993.
│
│     § Rust: origem (Mozilla Research, 2010), foco em segurança de memória sem GC,
│       sistema de tipos algébrico, traits como abstração sem custo de desempenho.
│       Por que é ideal aqui: simulação numérica de longa duração sem vazamento de
│       memória; modelo de ownership que elimina race conditions no loop de threads;
│       compilação para binário estático adequada a contêineres.
│
│   ├── Big picture do código FORTRAN original
│   │     Descreve a estrutura do código de Downs e Vogel em alto nível:
│   │     TEFUNC (dinâmica da planta, ~600 linhas), TESUB1–TESUB8 (subrotinas
│   │     auxiliares de termodinâmica), COMMON blocks como estado global,
│   │     arrays XMEAS[41] e XMV[12] como interface de I/O, IDV[20] como
│   │     vetor de distúrbios. A planta é "caixa-preta" com entradas XMV e
│   │     saídas XMEAS — interface que te-core preserva.
│   │
│   ├── Limitações e problemas da implementação FORTRAN
│   │     — Monólito numérico-procedural: a planta aparece como uma sequência
│   │       rígida de blocos de cálculo, não como equipamentos, streams,
│   │       sensores, atuadores e interfaces físicas nomeadas.
│   │
│   │     — Estado físico, variáveis algébricas e medições misturados:
│   │       temperaturas, pressões, vazões, composições e XMEAS são calculados
│   │       no mesmo corpo de função, dificultando separar dinâmica da planta
│   │       de instrumentação.
│   │
│   │     — Atuadores escondidos no vetor de estado: posições de válvula entram
│   │       como yy[38..50], mas a semântica "comando → dinâmica do atuador →
│   │       posição efetiva → vazão" não aparece explicitamente no modelo.
│   │
│   │     — Dependências físicas implícitas por ordem de execução: o código sabe
│   │       que precisa calcular termodinâmica antes de vazões, vazões antes de
│   │       balanços, pressões antes de fluxos, mas essa dependência não está
│   │       expressa em tipos ou componentes.
│   │
│   │     — Arrays sem identidade industrial: XMEAS, XMV, ftm, fcm, hst e vpos
│   │       são vetores numéricos; o significado de cada posição vive fora do
│   │       tipo, em tabelas, comentários ou memória do programador.
│   │
│   │     — Mistura de estado diferencial e variáveis algébricas: temperaturas,
│   │       pressões e composições são variáveis algébricas derivadas do estado —
│   │       não têm derivada própria. Tratá-las como coordenadas integradas
│   │       (ex: twr/tws em state[36..37]) obscurece o que o integrador propaga.
│   │
│   │     — Fronteira fraca entre modelo e mundo externo: não existe camada
│   │       formal separando estado interno, medição exposta, comando de atuador,
│   │       sinal bruto, unidade, qualidade, limite ou tag supervisória.
│   │
│   │     — Dificuldade de validação modular: como tudo está acoplado em uma
│   │       função grande, é difícil testar isoladamente uma válvula, um sensor,
│   │       uma vazão, uma equação de troca térmica ou um bloco termodinâmico.
│   │
│   └── O que o Rust deve endereçar — design interno do te-core
│         — A primeira coisa que fiz foi tentar comentar a lógica original e 
│           entender melhor seus blocos funcionais. Isso ainda no FORTRAN que se
│           encontra em https://github.com/Green-Cinnamon-Labs/tep-plant/tree/main/tennessee-eastman-process
│ 
│         — A segunda coisa que fiz foi uma versão 'fidedigna' do FORTRAN em
│           Rust; trata-se da relase https://github.com/Green-Cinnamon-Labs/tep-plant/releases/tag/v1.0.0
│
│         — Na segunda release eu já comecei a refatorar tudo para o padrão `facade`
|           abstraindo complexidades e organizando mais o código.
|
│         — Eu comecei a ter uma atenção maior. Passei por uma poderosa
│           revisão de nomenclatura de variáveis e tipos, para tornar o código mais
│           explicito e legível. Além disso, eu separei a lógica em funções e estruturas
│           com responsabilidades mais claras.
│
│         — Daí eu percebi que existe um problema de design

│ 
│         — Equipamentos como tipos concretos: Reactor, Separator, Stripper,
│           Compressor deixam de ser funções stateless e passam a ser structs
│           com estado próprio (moles, energia interna), cada um expondo
│           compute_thermo() → ThermoState e compute_derivatives() → Vec<f64>.
│
│         — Streams como valores tipados: em vez de xst[[f64;13];8] e
│           tst[f64;13], cada stream é um struct Stream { composition,
│           temperature, flow, enthalpy, mol_weight } identificado por nome
│           (AFeed, DFeed, Purge, …), não por índice.
│
│         — Separação explícita entre estado diferencial e variáveis algébricas:
│           o integrador só propaga moles e energia interna. Temperatura,
│           pressão e composição são recalculadas por flash a cada dynamics() —
│           nunca vivem como coordenadas integradas.
│
│         — Atuadores com semântica completa: Valve e Agitator recebem um
│           Command, produzem uma Position via dinâmica de primeira ordem, e
│           expõem position() para o cálculo de vazão. O integrador vê apenas
│           a derivada da posição, não o comando.
│
│         — dynamics() como função pura: recebe &State, devolve Derivatives,
│           sem efeitos colaterais. Logging, medição e atualização de
│           ProcessImage ocorrem no integrador, após o commit do passo.
│
│         — Camada de instrumentação separada: Sensor<T> lê de um ThermoState
│           ou FlowsOut, aplica ganho, offset e ruído, e escreve em um
│           ProcessImage — fora e depois do laço de integração.
│
│         — ProcessImage como fronteira formal: um struct nomeado e tipado
│           (não [f64;41]) que representa exatamente o que o mundo externo
│           enxerga — tag, valor, unidade, qualidade, timestamp. É a única
│           superfície que OPC-UA, IHM e historiador tocam.
│
│         — Grafo de dependências expresso em tipos: a ordem Disturbances →
│           Thermo → Flows → Heat → Derivatives não é uma sequência de chamadas
│           soltas — cada saída é o input tipado da próxima etapa, tornando
│           impossível chamar Flows sem ter um ThermoState válido.
│
│         — Validação modular por construção: cada equipamento, stream e sensor
│           é testável em isolamento com entradas explícitas. Não existe "rodar
│           a planta inteira para ver se a válvula está certa".
│
│         — Camada industrial como cidadã de primeira classe: AddressSpace,
│           NodeId, tag supervisória e unidade de engenharia nascem junto com
│           o modelo — não são adaptadores colados depois.
│
├── Simulador da Planta — tep-plant
│     [mantém estrutura atual, mas pode ser mais enxuta agora que a seção acima
│      fez o trabalho narrativo de justificativa]
│
│   ├── Estrutura Interna (te-core + tennessee-eastman-service)
│   ├── Loop de Simulação
│   ├── Controladores — ControllerBank
│   ├── Servidor gRPC
│   └── Sistema de Distúrbios
│
├── Implementação OPC UA                          ← NOVA SEÇÃO (a desenvolver)
│     [a desenvolver — posicionar a escolha de OPC UA dentro da arquitetura,
│      relação com a norma IEC 62541, como o servidor OPC UA se relaciona com
│      o gRPC já existente ou o substitui, e o que foi implementado vs. proposto]
│
├── Operator Kubernetes — plant-supervisor            [mantém estrutura atual]
│   ├── O CRD PLCMachine
│   ├── Loop de Reconciliação
│   └── Conectividade com a Planta
│
├── Interface Homem-Máquina — tep-ihm             [mantém estrutura atual]
│   ├── Arquitetura
│   ├── Dashboard
│   └── Registro de Dados (CSV)
│
├── Infraestrutura Local — tep-lab         [mantém estrutura atual]
│   ├── Modo de Desenvolvimento — Docker Compose
│   └── Modo de Produção — Cluster Kind
│
└── Considerações Finais
      [atualizar para incorporar a narrativa Fortran→Rust e OPC UA]
```

---

## Notas para formalizar

- A seção "Do FORTRAN ao Rust" deve vir **antes** de tep-plant porque ela fornece o
  quadro conceitual que justifica as escolhas de design — o leitor entende o *porquê*
  antes de ver o *como*.
- A seção OPC UA precisa definir claramente se é **implementação paralela** ao gRPC
  ou **substituição** — isso impacta onde ela se encaixa na hierarquia IEC 62264.
- As subseções 1.1/1.2/1.3 da seção Fortran→Rust podem se transformar em parágrafos
  dentro de uma única seção sem subsections, dependendo da profundidade que o autor
  quiser dar ao tema.
