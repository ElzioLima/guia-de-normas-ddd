# Guia de Normas DDD (ADR/README do Repositório)

**Status:** Ativo · **Versão:** 1.0 · **Data:** 2025‑09‑01 · **Proprietário:** Arquitetura/Engenharia

**Escopo:** Aplica‑se a todos os Bounded Contexts e serviços deste monorepo/multi‑repo.

**Diretriz permanente do projeto:** *Value Objects não usam herança; são comportamentais e conceituais, com reuso por composição/contratos.*

---

## 1) Propósito & Escopo

Estabelecer decisões arquiteturais e normas táticas de DDD para alinhar linguagem ubíqua, fronteiras de consistência, estilo de código, testes e integração. Esta ADR funciona como README vivo do repositório.

---

## 2) Sumário (navegação rápida)

1. Propósito & Escopo
2. Sumário
3. Vocabulário & Convenções
4. Normas Gerais de Domínio
5. Identidade, Tempo e Valores Monetários
6. Eventos (Domínio vs Integração)
7. Validações, Specifications & Policies
8. Serviços & Fábricas (Domain/Application)
9. Repositórios, Leitura e Mapeamentos
10. Fronteiras, Bounded Contexts & ACL
11. Consistência, Sagas/Process Managers
12. Organização & Modularidade
13. Observabilidade, Segurança & Auditoria
14. Qualidade & Testes (padrões de teste)
15. Versionamento & Evolução (schema/events)
16. Checklists por Camada

    * 16.1 Domain
    * 16.2 Application
    * 16.3 Infrastructure
17. Glossário do Domínio

> As seções 16.1–16.3 (checklists) serão mantidas como listas prescritivas de verificação contínua.

---

## 3) Vocabulário & Convenções

* **Linguagem Ubíqua:** nomes de classes/métodos/arquivos em inglês, refletindo termos do negócio; documentos em PT‑BR.
* **Past tense em eventos de domínio** (ex.: `BookLoaned`).
* **Semântica > Tecnologia:** o domínio não conhece framework/ORM.
* **Nomenclatura:** IDs e VOs com sufixo `Id`, `VO`, `Code`, etc.; repositórios por agregado; casos de uso no imperativo.

---

## 4) Normas Gerais de Domínio

* **VOs imutáveis, sem herança**; construção via fábricas estáticas; igualdade por valor; reuso por composição/utilitários.
* **Entidades** expõem comportamentos que preservam invariantes; sem setters genéricos.
* **Agregado = fronteira transacional**: invariantes fortes dentro; transações não atravessam agregados.
* **Apenas Aggregate Roots possuem repositório**; entidades internas só existem por meio do agregado.
* **Domínio é persistência‑agnóstica**: sem anotações/ORM no core.

---

## 5) Identidade, Tempo e Valores Monetários

* **Identidade como VO** (`BookId`, `UserId`), não strings cruas.
* **Geração de IDs** via porta `IdGenerator` (injetável na Application/Infra).
* **Relógio do domínio** via `Clock` injetável (não usar `now()` diretamente no core).
* **Money/Quantity/Percentage** como VOs com regras explícitas (moeda, escala, arredondamento, comparações).

---

## 6) Eventos (Domínio vs Integração)

* **Domain Events:** fatos passados, imutáveis, com semântica; publicados pelo agregado; handlers idempotentes.
* **Integration Events:** contratos externos; mapeados/adaptados na Application/Infra; não são o mesmo que Domain Events.
* **Versionamento de eventos** previsto; tolerância a mudanças e compatibilidade retroativa quando possível.

---

## 7) Validações, Specifications & Policies

* **Validação sintática** no VO (ex.: formato de email, UUID, limites numéricos).
* **Validação semântica** no agregado/serviço de domínio (ex.: limites de empréstimo, políticas de disponibilidade).
* **Specifications/Policies** para compor regras reutilizáveis sem poluir entidades.
* **Domain Exceptions** com semântica do negócio; não lançar erros técnicos no domínio.

---

## 8) Serviços & Fábricas

* **Factories** para criação de agregados complexos: objeto nasce válido ou falha explicitamente.
* **Domain Services** somente quando a regra não “cabe” em entidade/VO; sem orquestração externa.
* **Application Services** orquestram casos de uso, transações e integração; o domínio decide as regras.

---

## 9) Repositórios, Leitura e Mapeamentos

* **Repositórios retornam Agregados completos** (evitar partials que quebram invariantes).
* **Consultas pesadas** e relatórios via modelos de leitura/projeções (CQRS opcional).
* **Mapeamento ORM** isolado na Infra; evitar lazy‑loading que atravesse limites de agregado.

---

## 10) Fronteiras, Bounded Contexts & ACL

* **Bounded Contexts** com modelos próprios; sem “modelo corporativo único”.
* **ACL (Anti‑Corruption Layer)** para integrar com legados ou outros contextos, com mapeamentos explícitos.
* **Contratos externos** estáveis e versionados; compatibilidade tratada na borda.

---

## 11) Consistência, Sagas/Process Managers

* **Consistência eventual entre agregados** via eventos + outbox/retry.
* **Sagas/Process Managers** para long‑running e coordenação cross‑agregado/contexto.
* **Concorrência otimista** (versão) e **idempotência** em comandos/handlers.

---

## 12) Organização & Modularidade

* **Estratificação clara:** `Domain / Application / Infrastructure` com dependências unidirecionais.
* **Pastas por Bounded Context**; dentro, por táticos (Entities, VOs, Repos, Events, Services, etc.).
* **Ports** explícitas para serviços externos (Clock, Ids, Crypto, Bus, Email, Payments, Storage).

---

## 13) Observabilidade, Segurança & Auditoria

* **Logs/Tracing** na Application/Infra com `CorrelationId` (VO); o domínio não loga.
* **AuthN/AuthZ** fora do domínio; quando necessário, expressar como políticas/decisões abstratas.
* **Auditoria** como projeções/event sourcing ou trilhas de evento; não mesclar com regras de negócio.

---

## 14) Qualidade & Testes

* **Unit no domínio** (rápidos, sem infraestrutura), cobrindo invariantes e comportamentos.
* **Contratos** para adapters/repositórios (test doubles vs impl real).
* **E2E** para fluxos críticos; ferramentas de migração/esquema validadas.
* **ADRs curtas** para registrar e revisar decisões.

---

## 15) Versionamento & Evolução

* **Evolução incremental** do modelo com refactors guiados por testes.
* **Migrações de schema** compatíveis; janelas de transição planejadas.
* **Versionamento de mensagens/eventos** com compatibilidade progressiva e depreciação controlada.

---

## 16) Checklists por Camada (índice)

* **16.1 Domain** — verificação de VOs, Entidades, Agregados, Eventos, Serviços de Domínio, Specifications/Policies, Exceptions, Invariantes.
* **16.2 Application** — casos de uso, transações, orquestração, publicação/assinatura de eventos, mapeamentos entre Domain/Integration, validação de comandos, idempotência.
* **16.3 Infrastructure** — repositórios concretos, ORM/mapeamentos, ACL/adapters, mensageria/outbox, observabilidade, configurações, segurança nas bordas.

---

## 17) Glossário (sintético)

* **Agregado:** conjunto coerente de entidades/VOs com invariantes e raiz única.
* **Specification:** predicado reutilizável de regra de negócio.
* **Policy:** regra/decisão que direciona comportamento do domínio.
* **Outbox:** padrão para publicar eventos de forma confiável com a transação.
* **ACL:** camada que protege o modelo de domínios externos/legados.

# Guia de Normas DDD (ADR/README do Repositório)

**Status:** Ativo · **Versão:** 1.0 · **Data:** 2025‑09‑01 · **Proprietário:** Arquitetura/Engenharia

**Escopo:** Aplica‑se a todos os Bounded Contexts e serviços deste monorepo/multi‑repo.

**Diretriz permanente do projeto:** *Value Objects não usam herança; são comportamentais e conceituais, com reuso por composição/contratos.*

---

## 1) Propósito & Escopo

Estabelecer decisões arquiteturais e normas táticas de DDD para alinhar linguagem ubíqua, fronteiras de consistência, estilo de código, testes e integração. Esta ADR funciona como README vivo do repositório.

---

## 2) Sumário (navegação rápida)

1. Propósito & Escopo
2. Sumário
3. Vocabulário & Convenções
4. Normas Gerais de Domínio
5. Identidade, Tempo e Valores Monetários
6. Eventos (Domínio vs Integração)
7. Validações, Specifications & Policies
8. Serviços & Fábricas (Domain/Application)
9. Repositórios, Leitura e Mapeamentos
10. Fronteiras, Bounded Contexts & ACL
11. Consistência, Sagas/Process Managers
12. Organização & Modularidade
13. Observabilidade, Segurança & Auditoria
14. Qualidade & Testes (padrões de teste)
15. Versionamento & Evolução (schema/events)
16. Checklists por Camada

    * 16.1 Domain
    * 16.2 Application
    * 16.3 Infrastructure
17. Glossário do Domínio

> As seções 16.1–16.3 (checklists) serão mantidas como listas prescritivas de verificação contínua.

---

## 3) Vocabulário & Convenções

* **Linguagem Ubíqua:** nomes de classes/métodos/arquivos em inglês, refletindo termos do negócio; documentos em PT‑BR.
* **Past tense em eventos de domínio** (ex.: `BookLoaned`).
* **Semântica > Tecnologia:** o domínio não conhece framework/ORM.
* **Nomenclatura:** IDs e VOs com sufixo `Id`, `VO`, `Code`, etc.; repositórios por agregado; casos de uso no imperativo.

---

## 4) Normas Gerais de Domínio

* **VOs imutáveis, sem herança**; construção via fábricas estáticas; igualdade por valor; reuso por composição/utilitários.
* **Entidades** expõem comportamentos que preservam invariantes; sem setters genéricos.
* **Agregado = fronteira transacional**: invariantes fortes dentro; transações não atravessam agregados.
* **Apenas Aggregate Roots possuem repositório**; entidades internas só existem por meio do agregado.
* **Domínio é persistência‑agnóstica**: sem anotações/ORM no core.

---

## 5) Identidade, Tempo e Valores Monetários

* **Identidade como VO** (`BookId`, `UserId`), não strings cruas.
* **Geração de IDs** via porta `IdGenerator` (injetável na Application/Infra).
* **Relógio do domínio** via `Clock` injetável (não usar `now()` diretamente no core).
* **Money/Quantity/Percentage** como VOs com regras explícitas (moeda, escala, arredondamento, comparações).

---

## 6) Eventos (Domínio vs Integração)

* **Domain Events:** fatos passados, imutáveis, com semântica; publicados pelo agregado; handlers idempotentes.
* **Integration Events:** contratos externos; mapeados/adaptados na Application/Infra; não são o mesmo que Domain Events.
* **Versionamento de eventos** previsto; tolerância a mudanças e compatibilidade retroativa quando possível.

---

## 7) Validações, Specifications & Policies

* **Validação sintática** no VO (ex.: formato de email, UUID, limites numéricos).
* **Validação semântica** no agregado/serviço de domínio (ex.: limites de empréstimo, políticas de disponibilidade).
* **Specifications/Policies** para compor regras reutilizáveis sem poluir entidades.
* **Domain Exceptions** com semântica do negócio; não lançar erros técnicos no domínio.

---

## 8) Serviços & Fábricas

* **Factories** para criação de agregados complexos: objeto nasce válido ou falha explicitamente.
* **Domain Services** somente quando a regra não “cabe” em entidade/VO; sem orquestração externa.
* **Application Services** orquestram casos de uso, transações e integração; o domínio decide as regras.

---

## 9) Repositórios, Leitura e Mapeamentos

* **Repositórios retornam Agregados completos** (evitar partials que quebram invariantes).
* **Consultas pesadas** e relatórios via modelos de leitura/projeções (CQRS opcional).
* **Mapeamento ORM** isolado na Infra; evitar lazy‑loading que atravesse limites de agregado.

---

## 10) Fronteiras, Bounded Contexts & ACL

* **Bounded Contexts** com modelos próprios; sem “modelo corporativo único”.
* **ACL (Anti‑Corruption Layer)** para integrar com legados ou outros contextos, com mapeamentos explícitos.
* **Contratos externos** estáveis e versionados; compatibilidade tratada na borda.

---

## 11) Consistência, Sagas/Process Managers

* **Consistência eventual entre agregados** via eventos + outbox/retry.
* **Sagas/Process Managers** para long‑running e coordenação cross‑agregado/contexto.
* **Concorrência otimista** (versão) e **idempotência** em comandos/handlers.

---

## 12) Organização & Modularidade

* **Estratificação clara:** `Domain / Application / Infrastructure` com dependências unidirecionais.
* **Pastas por Bounded Context**; dentro, por táticos (Entities, VOs, Repos, Events, Services, etc.).
* **Ports** explícitas para serviços externos (Clock, Ids, Crypto, Bus, Email, Payments, Storage).

---

## 13) Observabilidade, Segurança & Auditoria

* **Logs/Tracing** na Application/Infra com `CorrelationId` (VO); o domínio não loga.
* **AuthN/AuthZ** fora do domínio; quando necessário, expressar como políticas/decisões abstratas.
* **Auditoria** como projeções/event sourcing ou trilhas de evento; não mesclar com regras de negócio.

---

## 14) Qualidade & Testes

* **Unit no domínio** (rápidos, sem infraestrutura), cobrindo invariantes e comportamentos.
* **Contratos** para adapters/repositórios (test doubles vs impl real).
* **E2E** para fluxos críticos; ferramentas de migração/esquema validadas.
* **ADRs curtas** para registrar e revisar decisões.

---

## 15) Versionamento & Evolução

* **Evolução incremental** do modelo com refactors guiados por testes.
* **Migrações de schema** compatíveis; janelas de transição planejadas.
* **Versionamento de mensagens/eventos** com compatibilidade progressiva e depreciação controlada.

---

## 16) Checklists por Camada (índice)

* **16.1 Domain** — verificação de VOs, Entidades, Agregados, Eventos, Serviços de Domínio, Specifications/Policies, Exceptions, Invariantes.
* **16.2 Application** — casos de uso, transações, orquestração, publicação/assinatura de eventos, mapeamentos entre Domain/Integration, validação de comandos, idempotência.
* **16.3 Infrastructure** — repositórios concretos, ORM/mapeamentos, ACL/adapters, mensageria/outbox, observabilidade, configurações, segurança nas bordas.

---

## 17) Glossário (sintético)

* **Agregado:** conjunto coerente de entidades/VOs com invariantes e raiz única.
* **Specification:** predicado reutilizável de regra de negócio.
* **Policy:** regra/decisão que direciona comportamento do domínio.
* **Outbox:** padrão para publicar eventos de forma confiável com a transação.
* **ACL:** camada que protege o modelo de domínios externos/legados.

---

## 16.1 Checklist — Domain

> Objetivo: garantir que o **núcleo de negócio** esteja correto, coeso, imutável onde preciso e livre de vazamentos de framework/persistência. Sem código nesta seção.

### 1) Value Objects (VOs)

* ✅ Imutáveis; criados via fábrica/método estático; igualdade **por valor**.
* ✅ **Sem herança** (regra permanente). Reuso por composição/utilitários/contratos.
* ✅ Validação **sintática** local (formato, faixa, escala, normalização).
* ✅ Transparência semântica: nomes que comunicam unidade e regra (ex.: `Money`, `Email`, `Quantity`).
* ✅ Serialização controlada (apenas para saída); sem vazar detalhes de infra.
* ❌ Não expor setters/mutators; ❌ não referenciar serviços externos.

### 2) Entidades

* ✅ Identidade **como VO** (`XId`) com igualdade por identidade + versão opcional para concorrência.
* ✅ Métodos de negócio preservam invariantes; sem setters genéricos.
* ✅ Estado mínimo necessário; alta coesão; invariantes documentadas no cabeçalho.
* ✅ Eventos de domínio são **produzidos** por métodos quando um fato relevante acontece.
* ❌ Não acessar repositórios/ORM diretamente; ❌ não conter lógica de orquestração cross‑agregado.

### 3) Agregados & Aggregate Roots

* ✅ Definir explicitamente **invariantes fortes** do agregado.
* ✅ Apenas o **Aggregate Root** é exposto; entidades internas protegidas.
* ✅ Operações públicas **atômicas** e consistentes; transação não atravessa agregados.
* ✅ Tamanho adequado (evitar *god aggregate*); particionar por coesão e por invariantes.
* ✅ Emissão de **Domain Events** registrada como parte do estado transitório do AR até o commit.
* ❌ Evitar lazy‑loading que dependa de infra; ❌ não permitir referências diretas a outros ARs (usar Ids/Policies).

### 4) Domain Events

* ✅ Nome no **passado** (`XxxHappened`/`XxxOccurred`/`BookLoaned`). Dados mínimos e imutáveis.
* ✅ Disparados **dentro** do método de negócio que mudou o estado.
* ✅ Sem side‑effects no construtor do evento; processamento feito por handlers fora do agregado.
* ✅ Idempotência e ordenação **não** são garantidas pelo domínio; consumidores devem lidar (handlers/pipelines).
* ✅ Planejar **versionamento** (v1, v2…) e compatibilidade.

### 5) Specifications & Policies

* ✅ Regras **combináveis** (AND/OR/NOT) exprimem políticas do negócio.
* ✅ Sem dependências de infraestrutura no núcleo; se precisar de dados externos, expor via portas.
* ✅ Usadas por entidades/serviços para decidir comportamento sem inflar modelos.
* ❌ Não duplicar validação sintática feita por VOs.

### 6) Domain Services

* ✅ Usados **apenas** quando a regra não pertence naturalmente a uma entidade/VO.
* ✅ Sem estado mutável; dependências injetadas como **portas** (ex.: `PricingRules`, `ExchangeRates`).
* ✅ Focados em **decisão de negócio**, não em orquestração técnica.

### 7) Fábricas (Factories)

* ✅ Criam agregados **válidos desde o nascimento** (all‑or‑nothing).
* ✅ Encapsulam composição de múltiplos VOs/entidades e validações iniciais.
* ✅ Devolvem sucesso/falha clara (sem estados parcialmente válidos).

### 8) Identidade & Tempo no Domínio

* ✅ **IdGenerator** como porta injetável; nenhuma chamada direta a libs no core.
* ✅ **Clock** como porta injetável para decisões temporais, vencimentos e janelas.
* ✅ `Money`, `Percentage`, `Quantity`, `DateRange` etc. como VOs com regras explícitas.

### 9) Erros, Exceções e Invariantes

* ✅ **Domain Exceptions** nomeadas com semântica (ex.: `LoanLimitExceeded`), documentadas por caso.
* ✅ Mensagens orientadas ao negócio; sem detalhes técnicos.
* ✅ Invariantes listadas e testadas (pré/pós‑condições, regras de consistência).

### 10) Limites & Dependências

* ✅ Domínio **não** importa Application/Infra; dependências unidirecionais (seta entra no domínio, nunca sai).
* ✅ Sem anotações de framework/ORM; sem logs no domínio; sem I/O no core.

### 11) Testes do Domínio

* ✅ **Unit tests** focados em comportamento e invariantes; rápidos, determinísticos, sem banco/queue.
* ✅ Testes de **contrato** para policies/specifications quando recebem portas.
* ✅ *Given‑When‑Then* com linguagem ubíqua; cenários representam regras do negócio.

### 12) Anti‑patterns a evitar

* ❌ **Anemic Domain Model** (regras fora do domínio).
* ❌ **Feature envy** (serviços que só manipulam estado de entidades).
* ❌ **God Aggregate** e agregados instáveis.
* ❌ **Herança acidental** em VOs/Entidades quando composição resolve.
* ❌ **Leaky abstractions**: domínio conhecendo ORM, mensageria ou detalhes de rede.

### 13) Revisões & Definition of Done (Domain)

> Objetivo: padronizar a qualidade mínima exigida para *merge* no núcleo do domínio.

#### 13.1 Revisão Rápida (PR Checklist)

* [ ] Linguagem ubíqua consistente (nomes comunicam o negócio).
* [ ] Não há dependências de framework/ORM/log no domínio.
* [ ] VOs são imutáveis, **sem herança**, com igualdade por valor e validação sintática local.
* [ ] Entidades expõem apenas métodos de negócio (sem setters genéricos).
* [ ] Invariantes dos agregados estão claras, testadas e não atravessam fronteiras.
* [ ] Eventos de domínio estão no **passado**, com payload mínimo e propósito claro.
* [ ] Policies/Specifications são composáveis e livres de infra.
* [ ] Domain Services só existem quando a regra não cabe em entidade/VO.
* [ ] Nenhuma referência direta entre Aggregate Roots (apenas Ids/policies/eventos).
* [ ] Testes do domínio rodam rápido e cobrem casos felizes e limites.

#### 13.2 DoD — Value Objects

* [ ] Construtor privado/controle de criação; invariantes locais garantidas.
* [ ] `equals`/comparações por valor definidas e testadas.
* [ ] Normalização (ex.: trim, canonicalização) documentada.
* [ ] Serialização apenas para saída; sem dependências técnicas.

#### 13.3 DoD — Entidades

* [ ] Identidade expressa por VO (`XId`) e igualdade por identidade.
* [ ] Métodos mantêm invariantes; estados intermediários inválidos não escapam.
* [ ] Emitem eventos quando fatos relevantes ocorrem.

#### 13.4 DoD — Agregados

* [ ] Invariantes fortes listadas (pré/pós‑condições) e cobertas por testes.
* [ ] Operações públicas atômicas; transação não cruza agregados.
* [ ] Tamanho/cohesão adequados (evita *god aggregate*).
* [ ] Versão/concurrency prevista quando aplicável.

#### 13.5 DoD — Domain Events

* [ ] Nome no passado; dados mínimos e imutáveis.
* [ ] Produzidos dentro do método que muda estado; sem side‑effects internos.
* [ ] Plano de versionamento (v1, v2…) quando contrato possa evoluir.

#### 13.6 DoD — Policies & Specifications

* [ ] Predicados claros, combináveis (AND/OR/NOT).
* [ ] Sem consulta direta a infra; dependências externas entram por portas.
* [ ] Testes de contrato cobrindo cenários positivos/negativos/limites.

#### 13.7 DoD — Domain Services

* [ ] Regras realmente transversais ao agregado; ausência de *feature envy*.
* [ ] Stateless, determinísticos; dependências como portas.
* [ ] Foco em decisão de negócio (não orquestração técnica).

#### 13.8 DoD — Documentação & ADRs

* [ ] Invariantes e decisões não óbvias registradas (ADR curta <= 1 página).
* [ ] Glossário atualizado quando novos termos surgirem.

#### 13.9 DoD — Testes & Qualidade

* [ ] Cobertura significativa das regras (priorizar cenários críticos > % numérica).
* [ ] *Given‑When‑Then* com linguagem ubíqua.
* [ ] Tempo de execução curto (preferência < 1s por suite do domínio).

#### 13.10 Sinais de Alerta (bloqueiam merge)

* [ ] VO com herança, mutável ou dependência técnica.
* [ ] Entidade com getters/setters anêmicos e lógica fora do domínio.
* [ ] Agregado fazendo *lazy‑load* de infra ou chamando repositório.
* [ ] Evento descrevendo intenção futura (não fato passado) ou com payload inchado.
* [ ] Policies acopladas a banco/filas/APIs.

**Resumo DoD (Domain)**

* [ ] VOs imutáveis, sem herança, validados e testados.
* [ ] Entidades preservam invariantes; sem setters genéricos.
* [ ] Agregados com invariantes explícitas, transação confinada e concorrência prevista.
* [ ] Eventos de domínio corretos (passado, mínimos, versionáveis) emitidos nos pontos certos.
* [ ] Policies/Specifications compostas e puras; Domain Services criteriosos.
* [ ] Nenhuma dependência de framework/ORM/IO no core.
* [ ] Testes verdes e rápidos; ADRs/Glossário atualizados.
