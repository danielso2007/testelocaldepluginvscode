# Guia Rápido: Code Review com Copilot Chat

Este guia torna imediato o uso do Copilot Chat para revisão de código neste projeto. Para fundamentos e regras completas, veja [.github/copilot-instructions.md](copilot-instructions.md).

## Passo a passo

1. Preparar contexto
   - Abra o(s) arquivo(s) ou PR a revisar e garanta que o projeto compila.
   - (Opcional) Rode as validações estáticas:

```bash
mvn checkstyle:check
mvn pmd:check
mvn verify
```

2. Pedir o review ao Copilot Chat
   - Use um dos prompts de exemplo abaixo, apontando para arquivos/diffs específicos.

3. Exigir formato de saída padronizado
   - Resumo (3–5 bullets)
   - Achados com Severidade (Crítico/Alto/Médio/Baixo/Elogio)
   - Links para arquivos/linhas, evidência e ação prática

## Checklist rápida (prioridades)

1) Arquitetura e camadas: Controller → Service → Ports/Adapters; regra de negócio apenas em Services/Business; sem entidades JPA no Controller (use DTOs/Records).
2) SOLID foco: SRP, OCP (Strategy/Polimorfismo ao invés de if-else por tipo), ISP, LSP, DIP (injeção via construtor, dependa de interfaces).
3) Boundaries/integrações: adapters para libs externas; não usar tipos de terceiros em Services; tratamento de erros/log adequado.
4) JPA: LAZY por padrão; evitar N+1; paginação em listagens; @Transactional somente em Service; equals/hashCode corretos.
5) Erros/Logs: separar fluxo de erro; exceções específicas; null aceitável apenas na borda de integrações; nunca logar dados sensíveis.
6) Testes/legibilidade: AAA/Given-When-Then; uma coisa por teste; nomes claros; evitar flakiness.

## Prompts de exemplo (copiar e colar)

- Revisão de arquivo

"Atue como revisor sênior. Projeto Java/Spring Boot. Siga estritamente as diretrizes em [.github/copilot-instructions.md](.github/copilot-instructions.md). Revise [src/main/.../MinhaClasse.java](../src/main/.../MinhaClasse.java) focando: SRP, OCP/Strategy, DIP (injeção via construtor), DTOs vs Entidades no Controller, JPA (LAZY, N+1), tratamento de erros e logs. Use o formato de saída sugerido e severidade. Inclua links de linhas."

- Revisão de PR/branch

"Faça code review do diff desta branch conforme [.github/copilot-instructions.md](.github/copilot-instructions.md). Classifique achados por severidade, cite arquivos/linhas e proponha refatorações concretas (Strategy/DTO/Mapper). Verifique também PMD/Checkstyle e regras fatais (OpenAPI gerado)."

## Formato de saída (obrigatório)

- Resumo: 3–5 bullets com riscos e impacto.
- Para cada achado:
  - Severidade: Crítico/Alto/Médio/Baixo/Elogio
  - Link para o trecho: [path/arquivo](path/arquivo#L10-L20)
  - Regra violada (ex.: SRP, OCP, DTOs, N+1)
  - Evidência sucinta (o que/por quê)
  - Ação prática (ex.: extrair Strategy, trocar EAGER→LAZY, criar DTO/Mapper)

## Severidade (rubrica)

- Crítico: regras fatais (OpenAPI gerado, segredos, field injection, dependência circular) ou alto risco de produção. Bloqueia merge.
- Alto: violações arquiteturais/SOLID que aumentam rigidez/fragilidade. Refatorar antes de merge.
- Médio: legibilidade, pequenos smells, KISS/DRY. Planejar refino.
- Baixo: estilo/menor impacto. Ajustar oportunisticamente.
- Elogio: aponte boas práticas e soluções elegantes.

## Ferramentas e tasks

- Maven

```bash
mvn checkstyle:check
mvn pmd:check
mvn verify
```

- VS Code (Terminal → Run Task)
  - "Maven Checkstyle"
  - "Maven PMD"
  - "Maven Verify"

## Regras críticas (fail-fast)

- Não editar código gerado pelo OpenAPI; ajuste no YAML e regenere.
- Nunca expor segredos/tokens em código/logs.
- Proibido Field Injection (@Autowired em atributo); use injeção via construtor com campos `final`.
- Evitar dependências circulares entre Beans/módulos.

## Referências a arquivos/linhas

- Sempre usar links Markdown: [caminho/arquivo](caminho/arquivo#L10-L12).
- Para trechos não contíguos, use links separados (não combine múltiplas faixas no mesmo link).
