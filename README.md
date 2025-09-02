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
