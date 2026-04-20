# Roadmap: Fundamentos para Especialização em GoLang

> Para desenvolvedores web experientes que querem construir uma base sólida em sistemas antes de mergulhar no Go.
> Foco em: processos do processador, concorrência, paralelismo, uso de RAM e garbage collection.

---

## Visão geral das fases

| Fase | Tema | Duração estimada |
|------|------|-----------------|
| 1 | Como o computador executa seu código | 3–5 semanas |
| 2 | Concorrência e paralelismo de verdade | 4–6 semanas |
| 3 | Gerenciamento de memória & Garbage Collection | 3–4 semanas |
| 4 | Ponte para o Go — runtime & design | 4–5 semanas |
| 5 | Projeto integrador — sistema distribuído | 4–6 semanas |

**Total estimado:** 18–26 semanas (estudo em paralelo com trabalho)

---

## Fase 1 — Como o computador executa seu código

### Tópicos

#### CPU & processos
- Como o processador executa instruções (fetch → decode → execute)
- Pipeline de instruções, caches L1/L2/L3 e sua influência em performance
- Context switching entre processos — o que o kernel salva e restaura
- Registradores e stack frames — o que acontece em cada chamada de função
- System calls: read, write, fork, execve — a fronteira user space / kernel space

#### Modelo de memória
- Stack vs Heap — quem aloca o quê e por quê
- Virtual memory e page tables — como cada processo enxerga "seu" espaço
- Segmentos de um processo: text, data, BSS, stack, heap
- Cache locality e performance — spatial vs temporal locality
- Endianness e alinhamento de dados

#### Processos vs Threads
- PCB (Process Control Block) — o que o SO armazena por processo
- Fork/exec no Unix — como novos processos são criados
- Threads do kernel vs user space — diferenças e trade-offs
- TLS (Thread Local Storage)
- Sinais UNIX — comunicação assíncrona entre processos

#### Ferramentas práticas
- `strace` — interceptar e inspecionar syscalls em tempo real
- `/proc` no Linux — o sistema de arquivos virtual do kernel
- `perf stat` e flamegraphs — profiling de CPU
- `vmstat`, `htop` — o que os números realmente significam
- `gdb` — inspecionar stack frames e registradores

### Recursos

- 📖 **OSTEP** — Operating Systems: Three Easy Pieces (gratuito online): https://pages.cs.wisc.edu/~remzi/OSTEP/
  - Melhor livro para entender virtualização de CPU e memória, concorrência e persistência de forma progressiva.
- 📖 **CS:APP** — Computer Systems: A Programmer's Perspective (Bryant & O'Hallaron): https://csapp.cs.cmu.edu/
  - Excelente para entender a ponte entre código de alto nível e o que a máquina realmente faz.
- ▶ **MIT 6.004** — Computation Structures (YouTube/MIT OpenCourseWare)
- 🌐 **Julia Evans** — Zines sobre strace, /proc e Linux: https://jvns.ca/categories/strace/
- 🌐 **Computer Science from the Bottom Up** (gratuito): https://www.bottomupcs.com/

### Projeto prático — Shell mínimo em C

Implemente um shell Unix com as seguintes funcionalidades:
- `fork` + `exec` para executar comandos externos
- `wait` para aguardar o filho
- Redirecionamento de stdin/stdout com `dup2`
- Pipes simples entre dois comandos
- Built-ins: `cd`, `exit`

**O que você vai aprender na prática:**
- Como processos são criados e destruídos
- O que acontece nos bastidores quando você digita um comando
- File descriptors e herança entre pai e filho

---

## Fase 2 — Concorrência e paralelismo de verdade

### Tópicos

#### Teoria da concorrência
- Concorrência ≠ Paralelismo — a distinção que todo dev precisa entender
- Race conditions e data races — como surgem e como detectar
- Deadlock, livelock e starvation — os três inimigos clássicos
- Happens-before e memory ordering — por que a ordem importa no hardware
- Modelos de concorrência: CSP (Go), Actor (Erlang/Akka), Shared memory (pthreads)

#### Primitivas de sincronização
- Mutex e RWMutex — quando usar cada um
- Semáforos e monitores
- Condition variables — aguardar uma condição sem busy-wait
- Operações atômicas: CAS (Compare-And-Swap), fetch-add
- Memory barriers: acquire/release semantics

#### Schedulers de threads
- Preemptive vs cooperative scheduling
- O scheduler M:N do Go (modelo G-M-P)
- Work stealing — como o Go balanceia goroutines entre threads
- User-space threads (goroutines) vs OS threads
- Event loop (Node.js/nginx) vs thread per request — quando cada um vence

#### I/O assíncrono
- Blocking vs non-blocking I/O
- `epoll` (Linux) / `kqueue` (BSD/macOS) / IOCP (Windows)
- Reactor pattern — o coração dos event loops
- O problema C10k e como o Go o resolve
- Comparativo: async/await (JS/Python) vs goroutines + channels

### Recursos

- 📖 **Learn Concurrent Programming with Go** — James Cutajar (Manning): https://www.manning.com/books/learn-concurrent-programming-with-go
  - Parte da atomic variables, futexes e princípios universais de concorrência.
- 📖 **Concurrency in Go** — Katherine Cox-Buday (O'Reilly): https://www.amazon.com/Concurrency-Go-Tools-Techniques-Developers/dp/1491941197
  - Cobre goroutines, channels, patterns e como compor primitivas em sistemas maiores.
- 📖 **OSTEP** — capítulos de Concorrência: https://pages.cs.wisc.edu/~remzi/OSTEP/
- ▶ **Rob Pike — Concurrency is not parallelism** (YouTube): https://www.youtube.com/watch?v=oV9rvDllKEg
- 🌐 **Go blog: Concurrency is not Parallelism**: https://go.dev/blog/waza-talk
- ▶ **The Little Book of Semaphores** (gratuito): https://greenteapress.com/wp/semaphores/

### Projeto prático — Thread pool do zero

**Etapa 1** — Implemente em C ou Python:
- Fila de tarefas thread-safe (mutex + condition variable)
- N worker threads que consomem tarefas
- Graceful shutdown: drenar fila antes de terminar
- Detect e reporte de race conditions com Valgrind/ThreadSanitizer

**Etapa 2** — Re-implemente o mesmo em Go com goroutines e channels:
- Worker pool com `chan struct{}` como semáforo
- Context para cancelamento
- `errgroup` para propagação de erros

**O que comparar:** número de linhas, complexidade, legibilidade e comportamento sob carga.

---

## Fase 3 — Gerenciamento de memória & Garbage Collection

### Tópicos

#### Alocação de memória
- `malloc`/`free` internamente — como `dlmalloc` gerencia o heap
- Heap fragmentation — interna vs externa
- Arenas e slab allocators — estratégias para reduzir fragmentação
- Escape analysis — quando o compilador aloca na stack vs no heap
- Memory pools e object reuse — técnica usada em engines de alta performance

#### Algoritmos de GC
- Reference counting — simples mas com problema de ciclos (CPython)
- Mark-and-sweep — o algoritmo base
- Generational GC — a hipótese da geração fraca (JVM)
- Tri-color marking — o algoritmo do Go GC (white/grey/black)
- Stop-the-world vs concurrent GC — o trade-off entre latência e throughput

#### O GC do Go em detalhe
- Write barriers no Go — como o GC mantém invariantes durante execução
- GC pauses e como tunar `GOGC` e `GOMEMLIMIT`
- `pprof` para heap profiling — identificar o que está sendo alocado
- `runtime.MemStats` — entender as métricas
- `sync.Pool` — reutilizar objetos e reduzir pressão no GC

#### Debugging de memória
- Valgrind — detectar leaks e erros de memória em C/C++
- AddressSanitizer — detecção em tempo de compilação
- Memory leaks em Go — goroutine leaks, slice retaining, maps que crescem
- Heap dumps e análise com ferramentas externas
- Benchmarks de alocação: `testing.B` com `-benchmem`

### Recursos

- 🌐 **Go GC Guide** (documentação oficial): https://go.dev/doc/gc-guide
- 🌐 **Ardan Labs: Scheduling in Go** (série de 3 partes): https://www.ardanlabs.com/blog/2018/08/scheduling-in-go-part1.html
- 🌐 **Aerospike: Understanding Garbage Collection**: https://aerospike.com/blog/understanding-garbage-collection/
- ▶ **GopherCon: Go GC deep dive** (YouTube): https://www.youtube.com/watch?v=q4HoWwdZgDs
- 🌐 **Go internals: escape analysis**: https://tip.golang.org/doc/asm
- 📖 **The Garbage Collection Handbook** — Jones, Hosking, Moss (referência acadêmica avançada)

### Projeto prático — Profiling de app com vazamento

**Passo 1** — Crie propositalmente os seguintes problemas em um programa Go:
- Goroutine leak (goroutine que nunca termina)
- Slice retaining (slice grande mantido vivo por sub-slice)
- Map que cresce indefinidamente
- Alocações desnecessárias no hot path

**Passo 2** — Use as ferramentas para diagnosticar:
```bash
go tool pprof http://localhost:6060/debug/pprof/heap
go tool pprof http://localhost:6060/debug/pprof/goroutine
go tool trace trace.out
```

**Passo 3** — Corrija e documente antes/depois com métricas de alocação usando `-benchmem`.

---

## Fase 4 — Ponte para o Go: runtime & design idiomático

### Tópicos

#### Runtime do Go por dentro
- Goroutine scheduling: o modelo G-M-P em detalhe
- Stack growth dinâmica — como o Go começa com 2KB e cresce sob demanda
- Channels: implementação interna com mutex + fila circular
- `select` internamente — como o runtime escolhe qual case executar
- `defer`, `panic` e `recover` — o stack unwinding do Go

#### Padrões de concorrência em Go
- Pipeline pattern — encadear stages com channels
- Fan-out / fan-in — distribuir e coletar trabalho
- Context para cancelamento e deadlines
- `errgroup` e semáforos com canais bufferizados
- Rate limiting com `time.Ticker`

#### Performance em Go
- Benchmarks com `testing.B` e interpretação correta
- `go tool pprof` — CPU profiling e heap profiling
- `go tool trace` — visualizar goroutines, GC e syscalls no tempo
- Evitar alocações no hot path — strings, interfaces, closures
- `GOMAXPROCS` e como ajustar para workloads diferentes

#### Design idiomático
- Interfaces pequenas: o poder de `io.Reader` e `io.Writer`
- Error wrapping com `fmt.Errorf("%w")` e `errors.Is`/`errors.As`
- Table-driven tests — o padrão Go para testes
- Composition over inheritance — como substituir herança com embedding
- Effective Go patterns — o que a documentação oficial recomenda

### Recursos

- 🌐 **Ardan Labs: Ultimate Go** (curso e material): https://www.ardanlabs.com/training/ultimate-go/
- 🌐 **Effective Go** (documentação oficial): https://go.dev/doc/effective_go
- 🌐 **Golang Roadmap 2024** (GitHub): https://github.com/baselrabia/Golang-Roadmap
- ▶ **Golang University 301** — scheduler, maps, channels internals: https://www.youtube.com/watch?v=KBZlN0izeiY
- 🌐 **roadmap.sh/golang**: https://roadmap.sh/golang
- 🌐 **Go blog** (artigos oficiais): https://go.dev/blog/

### Projeto prático — HTTP server concorrente do zero

Implemente um servidor HTTP usando apenas a stdlib (`net/http`), sem frameworks:

**Funcionalidades obrigatórias:**
- Middleware de logging com duração da request
- Rate limiting por IP com `time.Ticker` e `sync.Map`
- Graceful shutdown com `context` e `os.Signal`
- Handler com timeout via `context.WithTimeout`

**Benchmark:**
```bash
# instalar hey
go install github.com/rakyll/hey@latest

# testar com 200 conexões concorrentes
hey -n 10000 -c 200 http://localhost:8080/
```

**O que analisar:** latência p50/p95/p99, goroutines ativas durante carga, alocações por request.

---

## Fase 5 — Projeto integrador: sistema distribuído

Esta fase integra tudo que foi aprendido. Escolha um ou mais projetos para implementar e documentar no seu repositório.

> **Contexto do repositório:** cada projeto deve ter seu próprio diretório com um `README.md` explicando a arquitetura, as decisões técnicas tomadas, o que foi aprendido e benchmarks comparativos antes/depois de otimizações.

---

### Opção A — Job Queue distribuída

Um sistema de filas de tarefas com workers concorrentes:

- Worker pool com backpressure (channel bufferizado como semáforo)
- Persistência de jobs em BoltDB ou SQLite
- Retry com exponential backoff e jitter
- Dead letter queue para jobs que falham N vezes
- Métricas exportadas via Prometheus
- Graceful shutdown que drena a fila antes de encerrar

---

### Opção B — Key-Value Store in-memory

Um servidor tipo Redis minimalista:

- Sharding por hash para reduzir contenção de lock
- `RWMutex` por shard (leitura concorrente, escrita exclusiva)
- TTL com background goroutine usando min-heap
- Persistência via Write-Ahead Log (WAL)
- Protocolo binário simples via TCP
- Benchmark contra `sync.Map` e medição de throughput

---

### Opção C — CLI de Monitoramento de Performance

> **Inspiração:** o que ferramentas como RivaTuner ou MSI Afterburner fazem para jogos, mas rodando no terminal — exibindo métricas de CPU, RAM e processos em tempo real.

**O desafio:** construir uma ferramenta de linha de comando que leia e exiba métricas de hardware em tempo real, atualizando o terminal sem travar e sem vazar memória.

**Funcionalidades:**
- Leitura periódica de `/proc/stat` para calcular uso de CPU por core
- Leitura de `/proc/meminfo` para uso de RAM (total, usada, disponível, buffers/cache)
- Listagem dos top-N processos por consumo de CPU e memória (via `/proc/[pid]/stat`)
- Atualização da tela no terminal com ANSI escape codes (sem limpar e redesenhar tudo)
- Intervalo de atualização configurável via flag (`--interval 500ms`)

**O que você vai aprender:**
- Como o Linux expõe métricas do kernel via `/proc` — e o que cada campo realmente significa
- Conversão de CPU ticks (`jiffies`) para percentuais — o mesmo cálculo que o `htop` faz
- Gerenciamento do terminal: cursor, cores, layout sem bibliotecas externas
- Por que você deve evitar alocar variáveis desnecessárias na heap dentro do loop de monitoramento — o GC rodando a cada segundo distorceria suas próprias métricas

**Dica Go crítica:** use structs com campos alinhados na memória e reutilize buffers de leitura com `sync.Pool` ou variáveis pré-alocadas fora do loop. Qualquer alocação dentro do loop de 500ms vai acionar o GC e introduzir jitter nas métricas — um bug delicioso de observar e corrigir.

```
# Estrutura sugerida de flags
go-monitor --interval 500ms --top 10 --no-color
```

**Documentar no repo:** gráfico de alocações/segundo antes e depois de eliminar allocs no loop, via `go tool pprof`.

---

### Opção D — Load Tester Concorrente

> **Inspiração:** uma versão simplificada do `hey` ou `wrk`, construída do zero para entender o que acontece internamente quando você dispara milhares de requisições simultâneas.

**O desafio:** receber uma URL e disparar N requisições HTTP simultâneas, coletando métricas de latência e reportando estatísticas agregadas.

**Funcionalidades:**
- Flag `--concurrency` para definir número de goroutines paralelas
- Flag `--requests` para total de requisições ou `--duration` para tempo fixo
- Coleta de latência por requisição com resolução de microssegundos
- Cálculo de percentis: p50, p75, p90, p95, p99, p99.9
- Reporte de erros agrupados por status HTTP e por tipo de erro de rede
- Barra de progresso em tempo real no terminal

**O que você vai aprender:**
- O modelo fan-out/fan-in do Go: N goroutines produtoras → 1 goroutine agregadora via channel
- Como gerenciar **file descriptors** — o Linux limita FDs por processo (`ulimit -n`); com 10k goroutines você vai bater nesse limite e precisar entender como contorná-lo com semáforos
- `context.WithTimeout` e `context.WithCancel` para evitar goroutines zumbis se o servidor alvo travar ou demorar demais
- Como `http.Transport` reutiliza conexões (keep-alive) e o impacto no throughput

**Dica Go crítica:** goroutines que ficam bloqueadas em I/O sem timeout se tornam zumbis que nunca liberam memória. Um servidor lento pode travar todas as suas goroutines ao mesmo tempo. Use sempre `context` com deadline nas chamadas HTTP.

```go
// Padrão correto — nunca faça http.Get sem context
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()
req, _ := http.NewRequestWithContext(ctx, "GET", url, nil)
```

**Documentar no repo:** comparativo de throughput (req/s) vs concorrência, e o gráfico de latência mostrando onde aparecem os outliers de p99.

---

### Opção E — Cache in-memory com TTL e Eviction (LRU)

> **Inspiração:** implementar do zero o núcleo do que o Redis faz quando embarcado na sua aplicação — sem dependências externas, thread-safe e com política de eviction.

**O desafio:** um cache genérico em Go com expiração de chaves e remoção automática das entradas menos usadas quando a memória encher.

**Funcionalidades:**
- Interface genérica com `Get(key)`, `Set(key, value, ttl)`, `Delete(key)`
- TTL por chave — expiração lazy (na leitura) + ativa (goroutine de sweep periódico)
- Algoritmo LRU (Least Recently Used) com doubly-linked list + hash map para O(1) em get/set/evict
- `sync.RWMutex` — múltiplas goroutines podem ler simultaneamente; escrita é exclusiva
- Sharding opcional: dividir o cache em N shards independentes para reduzir contenção de lock

**O que você vai aprender:**
- `sync.RWMutex` na prática: quando usar `RLock` vs `Lock` e o impacto real no throughput sob carga concorrente
- A diferença entre expiração **lazy** (verificar na leitura, simples mas deixa lixo acumulado) e **ativa** (background goroutine, mais complexa mas previsível)
- Implementação de LRU com linked list em Go — por que você precisa dos dois (map + lista) e não pode usar só um
- Como o sharding reduz contenção: em vez de 1 mutex global, N mutexes independentes — o trade-off entre complexidade e throughput

**Estrutura sugerida:**
```go
type Cache[K comparable, V any] struct {
    shards    []*shard[K, V]
    numShards int
}

type shard[K comparable, V any] struct {
    mu      sync.RWMutex
    items   map[K]*entry[K, V]
    lru     *list.List         // container/list da stdlib
    maxSize int
}
```

**Benchmark obrigatório** (documentar no repo):
```bash
# comparar 3 implementações:
# 1. mutex global
# 2. RWMutex global
# 3. sharded RWMutex
go test -bench=. -benchmem -cpu=1,2,4,8 ./...
```

**Documentar no repo:** gráfico de operações/segundo por número de goroutines concorrentes para cada estratégia de locking. O resultado vai mostrar visualmente por que sharding existe.

---

## Comunidades e fóruns

| Recurso | Link |
|---------|------|
| 💬 Gophers Discord | https://discord.gg/golang |
| 💬 r/golang | https://www.reddit.com/r/golang/ |
| 🎯 Gophercises (exercícios práticos) | https://gophercises.com/ |
| 🌐 Go official blog | https://go.dev/blog/ |
| ▶ GopherCon (YouTube) | https://www.youtube.com/@GopherAcademy |
| 🌐 Go Time Podcast | https://changelog.com/gotime |

---

## Resumo dos projetos por fase

| Fase | Projeto | Linguagem | Foco principal |
|------|---------|-----------|----------------|
| 1 | Shell Unix mínimo | C | fork/exec, file descriptors |
| 2 | Thread pool do zero | C ou Python → Go | mutex, condition vars, channels |
| 3 | Profiling de memory leak | Go | pprof, trace, GC tuning |
| 4 | HTTP server concorrente | Go | concurrency patterns, benchmarks |
| 5A | Job Queue distribuída | Go | worker pool, persistência, backoff |
| 5B | Key-Value Store in-memory | Go | sharding, WAL, protocolo TCP |
| 5C | CLI de Monitoramento de Performance | Go | /proc, CPU ticks, terminal, GC pressure |
| 5D | Load Tester Concorrente | Go | fan-out/fan-in, file descriptors, context |
| 5E | Cache in-memory com TTL e LRU | Go | RWMutex, sharding, linked list, benchmarks |

---

## Dicas de estudo

1. **Leitura sem código não fixa.** Para cada capítulo do OSTEP, escreva um programa pequeno que demonstre o conceito.
2. **Use `strace` obsessivamente.** Todo comando que você rodar no terminal, tente entender quais syscalls ele faz.
3. **Leia código de projetos reais.** O código-fonte do runtime do Go (`src/runtime/`) é surpreendentemente legível.
4. **Benchmarks são sua bússola.** Antes de otimizar qualquer coisa, meça. Depois de otimizar, meça de novo.
5. **Erros são professores.** Race conditions, deadlocks e leaks que você causou propositalmente ensinam mais do que qualquer livro.