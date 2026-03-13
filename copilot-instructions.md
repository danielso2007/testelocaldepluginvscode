# Padrões de Revisão de Código

Ao revisar o código, atue como um revisor sênior e siga rigorosamente estas diretrizes:

## Formato de Saída Sugerido
O Copilot deve responder seguindo este padrão para cada apontamento:
> [SEVERIDADE] - Local: [path/arquivo](path/arquivo#L10-L12)
> Descrição: breve explicação da violação, com resposta técnica.
> Sugestão: código sugerido ou ação clara.

## Padrão de Referência a Arquivos/Linhas
- Use links Markdown com linhas: [path/arquivo](path/arquivo#L10) ou [path/arquivo](path/arquivo#L10-L12).
- Não use backticks para nomes de arquivos nem referências sem link.
- Para trechos não contíguos, use links separados (não combine múltiplos ranges em um único link).

## Ferramentas e Tasks
- Maven: `mvn checkstyle:check`, `mvn pmd:check`, `mvn verify`.
- VS Code (Terminal → Run Task): "Maven Checkstyle", "Maven PMD", "Maven Verify".

1. **Revisão de Código:**
    * Forneça feedback construtivo, específico e rigoroso.
    * Foque no código, sugira melhorias práticas e reconheça soluções elegantes.
2. **Arquitetura:**
    * Certifique-se de que a lógica de negócio esteja isolada apenas na camada de `Services` ou `Business`.
    * Garanta que as camadas estejam bem definidas e que as regras de negócio sejam independentes de frameworks.
    * O código deve expressar a arquitetura com separação clara de responsabilidades.
3. **Princípios SOLID (Fundamentais):**
    * **SRP (Single Responsibility):** Cada classe/função deve ter um único motivo para mudar. Identifique e sugira a divisão de "God Classes" ou métodos que fazem mais de uma tarefa.
      * **Identificação de Múltiplas Razões para Mudar:**
        * Sinalize classes que misturam lógica de negócio com infraestrutura (ex: um Service que monta queries SQL ou envia e-mails diretamente).
        * Identifique classes com nomes genéricos como Manager, Common ou Utils, que tendem a acumular responsabilidades diversas.
      * **Coesão de Métodos:**
        * Um método deve realizar apenas uma tarefa. Se um método contém comentários para separar "etapas" (ex: `// Step 1: Validate`, `// Step 2: Save`), ele deve ser quebrado em métodos menores ou classes distintas.
      * **Acoplamento Oculto:**
        * Observe se uma classe possui muitas dependências (mais de 5-7). Isso geralmente indica que ela está orquestrando responsabilidades demais.
    * **OCP (Open/Closed):** O código deve estar aberto para extensão, mas fechado para modificação. Prefira o uso de interfaces e herança/polimorfismo para adicionar novos comportamentos sem alterar o código existente.
      * **Diretriz de Extensibilidade**: O código deve permitir a adição de novas funcionalidades sem alterar o código fonte original.
      * **Identificação de Violação**: Sinalize métodos que utilizam estruturas de decisão (switch ou if-else if) para tratar diferentes tipos de uma mesma categoria de negócio. Isso indica que a classe precisará ser modificada toda vez que um novo tipo surgir.
      * **Sugestão de Refatoração**: Recomende o uso de interfaces ou classes abstratas. Cada novo comportamento deve ser implementado em uma nova classe que estenda a base, mantendo o código original "fechado" para modificações.
      * **Exemplos de Contexto:**
        * **Cenário de Violação (Ruim)**: Uma classe ProcessadorDeDesconto que possui um método com `if/else` para verificar se o cliente é "VIP", "REGULAR" ou "ESTUDANTE". Se um novo tipo "PRIME" for criado, você terá que abrir e alterar essa classe, correndo o risco de quebrar os descontos existentes.
        * **Cenário de Conformidade (Bom)**: Criar uma interface EstrategiaDesconto com o método calcular(). Cada categoria de cliente (VIP, Estudante, etc.) torna-se uma classe independente que implementa essa interface. O ProcessadorDeDesconto passa a receber a interface por parâmetro, tornando-se imune a novos tipos de clientes; ele está aberto para extensão (novos descontos) mas fechado para modificação.
    * **LSP (Liskov Substitution):** Subclasses devem ser substituíveis por suas classes base sem quebrar a aplicação. Evite sobrescrever métodos lançando `UnsupportedOperationException`.
      * **Diretriz de Substitutibilidade**: Objetos de uma classe base devem poder ser substituídos por objetos de suas subclasses sem que a aplicação pare de funcionar ou se comporte de forma inesperada.
      * **Identificação de Violação**:
        * Sinalize métodos sobrescritos que lançam exceções do tipo `UnsupportedOperationException` ou `NotImplementedException`.
        * Identifique "refreamentos" de comportamento (ex: uma subclasse que faz menos do que a classe pai ou que retorna `null` onde a pai retornava um objeto válido).
        * Sinalize verificações de tipo manual (`if (obj instanceof SubClass)`) dentro de métodos que deveriam ser genéricos.
      * **Sugestão de Refatoração**: Se uma subclasse não consegue implementar um método da classe pai de forma funcional, a hierarquia de herança está errada. Sugira a segregação de interfaces ou a composição em vez de herança.
      * **Exemplos de Contexto para o Copilot:**
        * **Cenário de Violação (Ruim)**: Imagine uma classe pai chamada `ContaBancaria` com os métodos `depositar()` e `sacar()`. Se criarmos uma subclasse `ContaInvestimentoRendaFixa` que não permite saques antes do vencimento e, por isso, lança uma `RuntimeException` toda vez que o método `sacar()` é chamado, estamos quebrando o LSP. O código que espera uma `ContaBancaria` genérica irá falhar ao receber essa subclasse específica.
        * **Cenário de Conformidade (Bom)**: Em vez de forçar a `ContaInvestimento` a herdar de uma classe que tem saque, devemos criar interfaces mais granulares. Uma interface `Depositavel` e outra `Sacavel`. A `ContaCorrente` implementa ambas, enquanto a `ContaInvestimentoRendaFixa` implementa apenas `Depositavel`. Assim, qualquer código que receba uma classe que implementa `Sacavel` terá a garantia de que o método realmente funciona.
    * **ISP (Interface Segregation):** Prefira várias interfaces específicas em vez de uma interface genérica ("gorda"). Não obrigue uma classe a implementar métodos que ela não utiliza.
      * **Diretriz de Granularidade**: Uma classe não deve ser forçada a depender de métodos que ela não utiliza. Interfaces grandes devem ser divididas em interfaces menores e mais específicas (especializadas).
      * **Identificação de Violação**:
        * Sinalize classes que implementam uma interface, mas deixam métodos vazios ou lançam `UnsupportedOperationException` (isso é frequentemente uma violação conjunta de LSP e ISP).
        * Identifique interfaces que possuem métodos que atendem a contextos de negócio muito distintos (ex: uma interface `Documento` que tem métodos para `imprimir()`, `assinarDigitalmente()` e `enviarViaFax()`).
      * **Sugestão de Refatoração**: Sugira a quebra da interface única em múltiplas interfaces menores. Uma classe pode implementar quantas interfaces forem necessárias, mas não deve herdar o que não consome.
      * **Exemplos de Contexto:**
        * **Cenário de Violação (Ruim)**: Temos uma interface chamada `Funcionario` com os métodos `getSalario()`, `gerarRelatorioVendas()` e `programarCodigo()`. Se tivermos uma classe `Vendedor`, ela será forçada a implementar `programarCodigo()` (mesmo que o corpo do método fique vazio). Se tivermos um `Desenvolvedor`, ele será forçado a implementar `gerarRelatorioVendas()`. A interface é "gorda" e polui as implementações.
        * **Cenário de Conformidade (Bom)**: Segregamos as responsabilidades. Criamos a interface base `Funcionario` (com `getSalario()`), uma interface `Vendedor` (com `gerarRelatorioVendas()`) e uma interface `Programador` (com `programarCodigo()`). A classe `Vendedor` implementa apenas `Funcionario` e `Vendedor`. Isso mantém o código limpo, coeso e evita que mudanças em relatórios de vendas afetem a classe do desenvolvedor.
    * **DIP (Dependency Inversion) & Injeção de Dependência:** * Dependa de abstrações. 
        * **Injeção de Dependência:** Use estritamente a injeção do Spring (preferencialmente via construtor).
        * **Proibição de Singletons Manuais:** Proíba o padrão Singleton manual (construtor privado + `static getInstance()`). Delegue o gerenciamento de escopo e ciclo de vida ao container do Spring (IoC).
        * **Diretriz de Abstração**:
          * Módulos de alto nível (regras de negócio) não devem depender de módulos de baixo nível (infraestrutura, banco de dados, APIs externas). Ambos devem depender de abstrações (interfaces).
          * Não se refira a classes concretas voláteis.
            * Refira-se a interfaces abstratas.
          * Não derive de classes concretas voláteis.
          * Não sobrescreva funções concretas.
            * Funções concretas muitas vezes exigem dependências de código-fonte. Quando houver o override dessas funções, não eliminar essas dependências. Para controlar essas dependências, converta a função em abstrata e crie múltiplas implementações.
          * Nunca mencione o nome de algo que seja concreto e volátil.
            * Essa é apenas uma reafirmação do próprio princípio.
        * **Identificação de Violação:**
          * Sinalize o uso da palavra-chave new para instanciar dependências complexas (Services, Repositories, Clients) dentro de uma classe.
          * Identifique o uso de implementações concretas em vez de interfaces em assinaturas de métodos e atributos.
          * Sinalize o uso de **Field Injection** (`@Autowired` diretamente no atributo), que dificulta testes e oculta o acoplamento.
        * **Sugestão de Refatoração:**
          * Exija a injeção via Construtor com atributos `final`.
          * Sugira a criação de interfaces para desacoplar a lógica de negócio de implementações tecnológicas (ex: em vez de depender de `S3Storage`, depender de `FileStorage`).
        **Exemplos de Contexto**
          * **Cenário de Violação (Ruim)**: Um `ProcessadorPagamento` que instancia diretamente um `MercadoPagoClient` dentro do seu construtor ou método. Se amanhã o provedor mudar para "Stripe", você terá que alterar a classe de negócio. Além disso, você não consegue testar o `ProcessadorPagamento` sem que ele tente se conectar à API do Mercado Pago.
          * **Cenário de Conformidade (Bom)**: O `ProcessadorPagamento` depende da interface `ProvedorPagamento`. O Spring injeta a implementação configurada (ex: `MercadoPagoService`) via construtor. O código de alto nível agora "dita as ordens" (define o contrato), e os detalhes de infraestrutura se adaptam a ele.
4. **Princípio das Dependências Acíclicas (ADP - Acyclic Dependencies Principle)**
    * **Diretriz de Estrutura de Grafo**: O grafo de dependências entre componentes, módulos ou pacotes deve ser um **Grafo Acíclico Dirigido (DAG)**. Ciclos de dependência (A -> B -> A) são terminantemente proibidos.
    * **Identificação de Violação:**
      * Sinalize quando dois pacotes dependem um do outro diretamente ou através de um terceiro pacote.
      * Identifique o uso de "importações cruzadas" entre módulos de diferentes camadas (ex: um pacote `Domain` tentando importar algo de um pacote `Controller`).
      * Sinalize erros de build onde o compilador aponta "Circular dependency" entre Beans do Spring.
    * **Sugestão de Refatoração:**
      * **Inversão de Dependência (DIP)**: Crie uma interface no pacote que precisa ser acessado e mova a implementação para o pacote que originou a chamada.
      * **Criação de um Terceiro Componente**: Mova as classes comuns que ambos os pacotes utilizam para um novo pacote/módulo de nível inferior (ex: common-types).
    * **Exemplos de Contexto:**
      * **Cenário de Violação (Ruim)**: O pacote `Pedido` depende do pacote `Pagamento` para processar uma transação. No entanto, o pacote `Pagamento` precisa enviar um e-mail de confirmação que contém detalhes do `Pedido`, gerando um ciclo: `Pedido -> Pagamento -> Pedido`. Isso impede que você teste ou faça o deploy de `Pagamento` sem levar todo o módulo de `Pedido` junto.
      * **Cenário de Conformidade (Bom)**: Criamos uma interface `ProcessadorPagamento` dentro do pacote `Pedido`. O pacote `Pagamento` implementa essa interface. Se o `Pagamento` precisar de dados do pedido para o e-mail, ele define um DTO ou interface de contrato própria. O ciclo é quebrado e o grafo torna-se linear ou em árvore.
5. **Princípio das Abstrações Estáveis (SAP - Stable Abstractions Principle)**
    * **Diretriz de Estabilidade Arquitetural**: Componentes que são altamente dependidos (possuem alto Afferent Coupling) devem ser compostos majoritariamente por interfaces e classes abstratas. Isso permite que eles sejam estáveis (difíceis de mudar a assinatura), mas ainda assim flexíveis (fáceis de estender o comportamento).
    * **Identificação de Violação:**
      * Sinalize classes concretas com lógica densa que são importadas por quase todos os outros módulos do sistema.
      * Identifique componentes "Rígidos": aqueles que são muito estáveis (muitos dependentes), mas são 100% concretos (difíceis de estender sem modificar o código original).
    * **Sugestão de Refatoração:**
      * Extraia interfaces para as classes core que são amplamente utilizadas.
      * Mova a lógica concreta para componentes periféricos (instáveis), mantendo o "Core" do sistema puramente abstrato.
    * **Exemplos de Contexto:**
      * **Cenário de Violação (Ruim):** Você tem uma biblioteca de "Segurança" que é usada por todos os microserviços da empresa. Toda a lógica de autenticação está em classes concretas e finais. Se você precisar mudar a forma como o token é validado para um serviço específico, você terá que alterar a biblioteca core e forçar o redeploy de toda a empresa. O componente é estável (todos dependem dele), mas não é abstrato (é rígido).
      * **Cenário de Conformidade (Bom):** A biblioteca de "Segurança" define apenas interfaces (ex: `Authenticator`, `TokenValidator`). A lógica pesada é injetada. Assim, a biblioteca permanece estável e imutável (abstração), mas cada serviço pode estender o comportamento conforme necessário sem que o "Core" precise ser modificado.
6. **Eficiência de Design: DRY, KISS e YAGNI**
    * **Diretriz Geral:** O design deve ser o mais simples possível para atender aos requisitos atuais. Evite a complexidade desnecessária sob o pretexto de "prever o futuro", mas não aceite a redundância que gera erros de manutenção.
    * **DRY (Don't Repeat Yourself - Não se Repita):**
        * **Diretriz:** Cada pedaço de conhecimento ou lógica deve ter uma representação única e inequívoca dentro do sistema.
        * **Identificação de Violação:** Sinalize blocos de código idênticos ou lógicas de validação repetidas em múltiplas classes.
        * **Cuidado com a Abstração Prematura:** Se a duplicata ocorre apenas em dois lugares e a abstração tornará o código muito complexo, aplique a "Regra dos Três" (refatore apenas na terceira ocorrência).
        * **Exemplo de Review:** "Esta lógica de cálculo de desconto é idêntica à do `OrderService`. Extraia para um componente comum ou `DomainService` para evitar divergências futuras."
    * **KISS (Keep It Simple, Stupid - Mantenha Simples):**
        * **Diretriz:** A simplicidade deve ser um objetivo fundamental. Código "inteligente" ou "astuto" é difícil de manter e entender.
        * **Identificação de Violação:** Sinalize o uso excessivo de *Design Patterns* para problemas simples, encadeamentos longos de Streams/Lambdas que dificultam o debug, ou algoritmos excessivamente complexos quando uma solução linear resolve.
        * **Refatoração:** Sugira a solução mais direta e legível, mesmo que ocupe mais linhas, se isso aumentar a clareza.
        * **Exemplo de Review:** "Este uso de Reflexão para mapear dois campos pode ser substituído por um simples construtor. Siga o KISS para facilitar o entendimento do time."
    * **YAGNI (You Ain't Gonna Need It - Você não vai precisar disso):**
        * **Diretriz:** Não adicione funcionalidades, classes ou abstrações que não sejam necessárias para os requisitos atuais.
        * **Identificação de Violação:** Sinalize métodos "para uso futuro", parâmetros não utilizados, classes genéricas criadas "por precaução" ou interfaces com apenas uma implementação quando não há previsão real de troca.
        * **Refatoração:** Remova o código morto ou a abstração excessiva. Implemente apenas o necessário para o *ticket* atual.
        * **Exemplo de Review:** "Você criou uma interface e três classes abstratas para um serviço que só tem uma implementação prevista. Remova a abstração seguindo o YAGNI e simplifique o design."
    * **Exemplos Comparativos para Referência:**
        > **❌ VIOLAÇÃO (Over-engineering e Falta de DRY):**
        > ```java
        > // Duplicação e Complexidade Desnecessária
        > public class UserUtils {
        >     public boolean validateEmail(String email) { /* lógica complexa */ return true; }
        > }
        > public class AdminUtils {
        >     public boolean checkEmail(String email) { /* lógica idêntica repetida */ return true; }
        > }
        > ```
        > **✅ CONFORMIDADE (Simples, Único e Atual):**
        > ```java
        > // Centralizado, Simples e Direto
        > public class EmailValidator {
        >     public static boolean isValid(String email) {
        >         return email != null && email.contains("@");
        >     }
        > }
        > ```
7. **Integridade Arquitetural e Fluxo de Dependência**
    * **Diretriz de Dependência:** As dependências devem apontar sempre para dentro (em direção ao Core/Domínio). A lógica de negócio deve ser agnóstica a detalhes de entrada (Web/API) ou saída (Banco de Dados/Mensageria).
    * **Identificação de Violação:**
      * Sinalize o vazamento de entidades JPA (`@Entity`) para a camada de `Controller` ou como resposta direta da API.
      * Sinalize o uso de anotações de infraestrutura (Jackson, Hibernate, Spring Security) dentro das classes de domínio puro.
      * Identifique acessos diretos do `Controller` ao `Repository`, pulando obrigatoriamente a camada de `Service`.
    * **Sugestão de Refatoração:** Exija o uso de **Mappers/DTOs** para isolar as camadas. Se o domínio precisar de um serviço externo, exija a criação de uma **Interface (Port)** no domínio e a implementação na infraestrutura (Adapter).
8. **Resiliência e Microsserviços**
    * **Circuit Breaker:** Sempre que houver chamadas para APIs externas ou serviços de terceiros, sugira o uso de padrões de resiliência (ex: Resilience4j).
    * **Idempotência:** Em processamento de mensagens (Kafka/RabbitMQ) ou endpoints de criação (POST), garanta que o código trate execuções duplicadas através de chaves de idempotência.
    * **Retries:** Verifique se as políticas de re-tentativa possuem *exponential backoff* e *jitter* para evitar o efeito "thundering herd" em sistemas de destino.
9. **Separação por Domínios (Bounded Contexts)**
    * **Diretriz de Isolamento:** Cada sub-domínio (ex: Billing, Catalog, Shipping) deve ser o mais independente possível para evitar o "Big Ball of Mud".
    * **Identificação de Violação:**
        * Sinalize quando uma classe de um domínio acessa diretamente tabelas ou serviços de outro domínio sem um contrato oficial (API ou Interface).
        * Identifique "Modelos Compartilhados" que misturam regras de domínios distintos em uma única classe.
    * **Sugestão de Refatoração:** Sugira DTOs específicos por domínio e comunicação desacoplada (via Eventos ou Interfaces de integração) para reduzir o acoplamento temporal.
10. **Fronteiras: Estabelecendo Limites (Boundaries)**
    * **Diretriz:** Mantenha o código limpo nas fronteiras entre o seu sistema e ferramentas de terceiros. Nunca deixe que APIs externas (como bibliotecas de log, JSON ou utilitários) se espalhem por toda a regra de negócio.
    * **Identificação de Violação:** Sinalize quando tipos de bibliotecas externas são usados como parâmetros em métodos de Service ou Domínio.
    * **Sugestão de Refatoração:** Utilize o padrão **Adapter** ou **Wrappers**. Crie sua própria interface para que, se a biblioteca mudar, você altere apenas um ponto do sistema. Encapsule o uso de mapas (`Map<K,V>`) ou coleções genéricas em classes de domínio para evitar a exposição excessiva de dados internos.
11. **Arquitetura Plug-in (Extensibilidade)**
    * **Diretriz:** O core da aplicação deve tratar funcionalidades secundárias como plug-ins (Banco de dados, UI, Mensageria). O core não deve saber quem o chama ou como os dados são persistidos.
    * **Identificação de Violação:** Sinalize lógica de "tomada de decisão" baseada em qual banco de dados ou qual tipo de interface está sendo usada.
    * **Sugestão de Refatoração:** Use o padrão **Strategy** e **Dependency Injection**. O core define o "soquete" (interface) e o plug-in fornece o "conector" (implementação).
12. **Concorrência e Multithreading (Concurrent Programming)**
    * **Diretriz Geral:** A concorrência é uma estratégia de desacoplamento. Ela ajuda a separar *o que* é feito de *quando* é feito. No entanto, ela deve ser tratada como um domínio separado de responsabilidade (SRP). O código concorrente deve ser isolado do código de lógica de negócio.
    * **Regras de Proteção e Identificação:**
        * **Separação de Responsabilidades (SRP):** Sinalize quando a lógica de concorrência estiver misturada com a lógica de negócio. O código que gerencia threads deve ter sua própria classe ou componente.
        * **Limite de Acesso aos Dados:** Minimize o acesso a dados compartilhados. Prefira o uso de objetos imutáveis e cópias de dados em vez de compartilhar referências mutáveis entre threads.
        * **Conheça sua Biblioteca (Java.util.concurrent):** * Sinalize o uso de coleções padrão (`HashMap`, `ArrayList`) em ambientes multithread. 
            * Sugira coleções thread-safe como `ConcurrentHashMap`, `CopyOnWriteArrayList` ou `BlockingQueue`.
            * Use `AtomicLong`, `AtomicReference` em vez de blocos `synchronized` para operações simples de contagem ou atualização.
        * **Conheça seus Métodos de Execução:** Proíba a criação manual de threads (`new Thread()`). Exija o uso de **Executors**, **Thread Pools** ou as abstrações do Spring (`@Async`).
        * **Dependências entre Métodos Sincronizados:** Sinalize quando uma classe possui mais de um método `synchronized`. O uso de múltiplos métodos sincronizados na mesma classe muitas vezes leva a bugs sutis e deadlocks.
        * **Código de Desligamento (Graceful Shutdown):** Identifique falhas no encerramento de threads. Processos que ficam "pendurados" ou não respondem a interrupções são violações. Garanta que o código lide corretamente com `InterruptedException`.
    * **Testes de Código com Threads:**
        * Exija testes que forcem falhas de concorrência. 
        * Sinalize testes unitários que não falham de forma consistente (flaky tests) em ambientes concorrentes.
        * Sugira ferramentas de monitoramento e testes de estresse para validar se as seções críticas estão realmente protegidas.
    * **Exemplos de Referência:**
        > **❌ VIOLAÇÃO (Concorrência Espalhada e Insegura):**
        > ```java
        > public class OrderService {
        >     private int totalOrders; // Dado compartilhado mutável
        >
        >     public synchronized void processOrder() { // Lock pesado na instância
        >         totalOrders++;
        >         new Thread(() -> { // Criação manual de thread
        >             sendNotification();
        >         }).start();
        >     }
        > }
        > ```
        > **✅ CONFORMIDADE (Concorrência Isolada e Segura):**
        > ```java
        > @Service
        > public class OrderService {
        >     private final AtomicInteger totalOrders = new AtomicInteger(0);
        >     private final NotificationService notificationService;
        >
        >     public void processOrder() {
        >         totalOrders.incrementAndGet(); // Atomicidade sem lock
        >         notificationService.sendAsyncNotification(); // Delegação para executor gerenciado
        >     }
        > }
        >
        > @Service
        > public class NotificationService {
        >     @Async // Abstração do framework (Thread Pool gerenciado)
        >     public void sendAsyncNotification() {
        >         // Lógica de envio
        >     }
        > }
        > ```
13. **Processos Locais e Comunicação**
    * **Diretriz:** Garanta que a comunicação entre processos locais (ou dentro do mesmo SO) seja isolada e síncrona apenas quando necessário.
    * **Identificação de Violação:** Sinalize chamadas de sistema ou execuções de shell script diretamente do código Java.
    * **Sugestão de Refatoração:** Utilize abstrações de IO modernas (`java.nio`) e garanta que recursos como Sockets ou Pipes sejam fechados via `try-with-resources`.
14. **Serviços (EAA - Enterprise Application Architecture)**
    * **Diretriz:** Serviços devem ser independentes, possuir baixo acoplamento e alta coesão. Distinga claramente entre "Serviços de Domínio" (lógica pura) e "Serviços de Aplicação" (orquestração).
    * **Identificação de Violação:** * Sinalize serviços que mantêm estado (stateful) entre requisições. 
        * Identifique serviços que tentam realizar muitas operações de domínios diferentes (God Services).
    * **Sugestão de Refatoração:** Transforme serviços em **Stateless**. Cada requisição deve conter toda a informação necessária para o processamento. Aplique o conceito de "Saga" para transações que envolvem múltiplos serviços.
15. **Nomenclatura:**
    * Variáveis booleanas devem obrigatoriamente começar com `is`, `has` ou `should` (ex: `isActive`).
    * Siga as convenções de `ConstantName`, `MemberName`, `MethodName` e `ParameterName` do Checkstyle.
16. **Segurança:**
    * Proíba a exposição de segredos, tokens ou chaves de API; exija sempre o uso de variáveis de ambiente e arquivos de configuração protegidos.
17. **Tratamento de Erros:**
    * **Diretriz:** O tratamento de erro é uma coisa só. Funções que tratam erros não devem fazer mais nada.
    * **Identificação de Violação:** * Sinalize blocos `try-catch` gigantescos com lógica de negócio misturada.
        * Identifique o retorno de códigos de erro (use Exceções).
        * Sinalize métodos que retornam `null` para indicar erro (prefira `Optional` ou lance uma Exceção).
    * **Sugestão de Refatoração:** Extraia o corpo do `try` para um método específico de negócio, deixando o método original apenas para a orquestração do erro.
    * Exija o uso de blocos try-catch em chamadas externas (APIs e banco de dados) com log adequado. Lançando e sendo capturado por handlers globais (ApiExceptionHandler).
    * Exija o uso de blocos try-catch em chamadas de mensageria (ex: Kafka, RabbitMQ) com log adequado. Usando o bean clientUtil para logs padronizados, quando houver essa classe. Se não, usar log padrão. Exemplo: injetar private final ClientUtil clientUtil e usar throw clientUtil.handleException(e, ABR02, PRODUTO_VIDA_HISTORICO_ABRANGENCIA).
    * Evite retornar null em lógica de negócio; prefira Optional. Em integrações, null pode representar ausência legítima de dados (ex: cliente não cadastrado).
      * Substitua o retorno direto por Optional<T>: Isso força o desenvolvedor que consome o método a lidar com a ausência do dado.
      * Lance Exceções de Negócio: Se a ausência do dado for um erro crítico para o processo (ex: "Conta não encontrada para saque"), lance uma exceção específica em vez de retornar null ou Optional.
    * Separe estritamente a lógica de tratamento de erro da lógica principal.
    * **Em Integrações (Gateways, Clients, Repositórios):**
      * Em integrações (chamadas REST, consultas ao BD), o null é uma resposta técnica válida que indica "vazio". Porém, para evitar que esse null se espalhe ("vaze") para o seu sistema, você deve encapsulá-lo na borda.
      * Em Services de integração (Clients, Gateways), retornar null é aceitável quando representa ausência legítima de dados (ex: BcpService retorna null se cliente não existe). Camadas superiores devem lidar com esses nulls adequadamente.
      * Padrão Null Object: Se for uma lista, retorne Collections.emptyList() em vez de null.
18. **Lançamento de Erros:**
    * Ao lançar erros personalizados, garanta que sejam tratados centralizadamente na classe `ApiExceptionHandler`.
19. **Padrões de Dados e Imutabilidade (Java):**
    * **Records:** Prefira o uso de `Records` para DTOs e Value Objects para garantir imutabilidade e concisão.
    * **DTOs:** Utilize DTOs para entrada e saída de dados da API; evite expor entidades JPA/Hibernate diretamente nos controladores.
    * **Encapsulamento:** Mantenha atributos privados e utilize construtores para inicialização sempre que possível, evitando setters desnecessários.
20. **Nomes Significativos (The Art of Naming)**
    * **Diretriz Geral:** O nome de uma variável, função ou classe deve responder a todas as grandes perguntas: por que ela existe, o que ela faz e como é usada. Se um nome requer um comentário, ele falhou em revelar sua intenção.
    * **Regras de Ouro para Identificação e Refatoração:**
        * **Revele o Propósito:** Evite nomes como `var d; // dias passados`. Use `int daysSinceCreation`.
        * **Evite Desinformação:** Não chame um grupo de contas de `accountList` se ele não for uma `List` (use `accounts` ou `accountGroup`). Evite nomes muito similares como `ControllerManagerForStorage` e `ControllerManagerForDisplay`.
        * **Distinções Significativas:** Não use nomes diferentes para a mesma coisa (ex: `UserInfo` e `UserData` no mesmo projeto). Evite termos redundantes como `Variable` no nome de uma variável ou `Table` no nome de uma classe.
        * **Use Nomes Pronunciáveis e Buscáveis:** Substitua siglas obscuras por palavras reais. Evite variáveis de uma única letra (como `e`, `t`), a menos que sejam contadores locais em loops minúsculos. Nomes grandes e descritivos são melhores que nomes curtos e enigmáticos para busca (`Ctrl+F`).
        * **Abandone Codificações (No Hungarian Notation):** Não use prefixos de tipo (ex: `strName`, `iCount`) ou prefixos de membros (`m_`). Em Java modernas, o tipo já é conhecido pela IDE.
        * **Evite Mapeamento Mental:** O leitor não deve ter que traduzir mentalmente seu nome de variável para o conceito real (ex: `r` para `url_minus_protocol`).
        * **Nomes de Classe:** Devem ser substantivos ou frases substantivadas (ex: `Customer`, `WikiPage`). Evite termos genéricos como `Data` ou `Info`.
        * **Nomes de Métodos:** Devem possuir verbos ou frases verbais (ex: `postPayment`, `deletePage`, `isSettled`). Siga o padrão *get/set* para acessores.
        * **Uma Palavra por Conceito:** Escolha um termo e use-o em todo o projeto. Não misture `fetch`, `retrieve` e `get` para a mesma operação em classes diferentes.
        * **Sem Trocadilhos:** Evite usar a mesma palavra para dois propósitos diferentes. Se `add` insere em uma lista, não use `add` para concatenar valores se o sentido for diferente.
        * **Domínio da Solução vs. Problema:** Use termos da ciência da computação (algoritmos, nomes de padrões como `AccountFactory`) para desenvolvedores, mas use nomes do domínio do negócio (problema) para conceitos de regra de negócio, facilitando a conversa com o Product Owner.
        * **Contexto Significativo:** Adicione contexto apenas se necessário. Em uma classe `Address`, as variáveis podem ser apenas `state` e `city`, em vez de `addressState` e `addressCity`. Não adicione contexto desnecessário a tudo (ex: prefixar todas as classes com o nome da aplicação `MsysAccount`).
    * **Exemplo de Refatoração:**
        * **Ruim:** `public void getNm(List l) { for (Object x : l) { ... } }`
        * **Bom:** `public void retrieveActiveCustomers(List<Customer> customers) { for (Customer customer : customers) { ... } }`
21. **Abreviações:**
    * Proíba abreviações obscuras; prefira nomes por extenso (ex: use `calculateAverage()` em vez de `calcAvg()`).
22. Funções e Métodos (Clean Functions)
    * **Diretriz Geral:** A primeira regra das funções é que elas devem ser pequenas. A segunda regra é que elas devem ser menores ainda. Uma função deve contar uma história clara e focar em apenas uma tarefa.
    * **Regras de Identificação e Refatoração:**
        * **Pequenas e Concisas:** Sinalize funções que ultrapassem 20 linhas. O tamanho ideal é entre 5 e 10 linhas. Cada função deve ser lida como um parágrafo de uma história.
        * **Faça Apenas uma Coisa (SRP):** Se uma função executa passos que podem ser descritos com "e então" (ex: valida E salva E envia e-mail), ela está fazendo coisas demais. Extraia essas responsabilidades.
        * **Um Nível de Abstração por Função:** Não misture conceitos de alto nível (ex: `checkAuthorization`) com conceitos de baixo nível (ex: `.append("\n")`) na mesma função. Isso confunde o leitor sobre a intenção do código.
        * **Instruções Switch/If-Else:** O uso de `switch` ou longas cadeias de `if-else` deve ser limitado a fábricas (Factories) para criar objetos polimórficos. Não permita que a lógica de negócio seja governada por `switches` que precisem ser alterados a cada novo tipo.
        * **Nomes Descritivos:** Não tenha medo de nomes longos se eles forem descritivos. `calculateMonthlyInterestAndFees` é melhor que `calculateFees`. Use verbos e mantenha a consistência com os nomes das variáveis.
        * **Parâmetros de Funções:** * **Mínimo possível:** O ideal é zero (nádico), seguido por um (monádico) ou dois (diádico). 
            * **Três (triádico):** Deve ser evitado. 
            * **Mais de três (poliádico):** Sinalize como erro e sugira a criação de um objeto de argumento (DTO/Record).
        * **Sem Argumentos de Flag:** Proíba parâmetros booleanos (ex: `render(boolean isSuite)`). Eles gritam que a função faz mais de uma coisa. Sugira dividir em duas funções: `renderForSuite()` e `renderForSingle()`.
        * **Sem Efeitos Colaterais (Side Effects):** Uma função não deve fazer mudanças ocultas no estado global ou em variáveis de classe que não estão claras em seu nome. Se `checkPassword` também inicia uma sessão (`initSession`), o nome deve ser `checkPasswordAndInitializeSession` (ou, preferencialmente, ser dividida).
        * **Separação Comando-Consulta (CQS):** Uma função deve fazer algo (comando) ou responder algo (consulta), nunca ambos. `if (set("username", "unclebob"))` é confuso; separe em `set` e depois verifique o estado.
        * **Prefira Exceções a Códigos de Erro:** Não retorne `int` ou `boolean` para indicar sucesso ou erro. Isso força o chamador a lidar com estruturas `if (error == -1)` profundamente aninhadas. Lance exceções com nomes claros.
        * **Evite Repetição (DRY - Don't Repeat Yourself):** O código duplicado é o pecado capital do software. Identifique algoritmos idênticos ou muito similares em diferentes funções e sugira a extração para um método comum ou utilitário.
    * **Exemplo de Review Sugerido pela IA:**
        * **Violação:** Uma função de 50 linhas com um `boolean` que decide se deleta ou atualiza um registro.
        * **Sugestão da IA:** "Esta função viola o SRP e usa um Flag Argument. Recomendo dividir em `deleteRecord()` e `updateRecord()`, movendo a lógica comum para um método privado `saveChange()`."
23. **Comentários (Expressividade vs. Ruído)**
    * **Diretriz Geral:** Comentários são, na melhor das hipóteses, um mal necessário. O uso de um comentário é uma admissão de derrota na tentativa de se expressar através do código. O objetivo é que o código seja autodocumentado.
    * **Regras de Identificação e Refatoração:**
        * **Comentários não compensam código ruim:** Se o código está confuso, sinalize que ele deve ser limpo, e não "explicado" por um comentário. Um comentário nunca justifica uma lógica obscura ou nomes de variáveis ruins.
        * **Explique-se no Código:** Antes de permitir um comentário, exija que o desenvolvedor tente explicar a intenção através de uma nova função ou variável.
            * *Exemplo:* Em vez de `if (employee.flags && HOURLY) // verifica se é horista`, sugira `if (employee.isEligibleForFullBenefits())`.
        * **Identificação de Comentários Ruins (Sinalizar para Remoção):**
            * **Murmúrios:** Comentários escritos apenas por obrigação ou que não adicionam valor real.
            * **Redundantes:** Comentários que apenas repetem o que o nome do método já diz (ex: `// Define o nome` acima de `setName`).
            * **Comentários de Diário ou Autoria:** Registros de quem alterou o quê (o Git já faz isso). Remova tags como `@author`.
            * **Marcadores de Posição:** Comentários como `// ------------- Actions -------------` que apenas poluem a visualização.
            * **Código Comentado (Dead Code):** Esta é a violação mais grave. Sinalize para remoção imediata, pois o código comentado gera confusão e medo de exclusão em outros desenvolvedores.
            * **Comentários de Fechamento:** Sinalize comentários no final de blocos, como `} // end if`. Se o bloco é tão longo que precisa de um comentário no fim, o bloco deve ser diminuído.
        * **Comentários Aceitáveis (O "Porquê", não o "O quê"):**
            * **Comentários Informativos:** Explicar o formato de um Regex ou uma decisão técnica de baixo nível.
            * **Explicação da Intenção:** Quando há uma decisão de negócio complexa ou um "hack" necessário por causa de uma biblioteca externa bugada.
            * **Alertas de Consequências:** `// este teste demora 30 minutos para rodar`.
            * **TODOs:** Apenas se forem tarefas iminentes e rastreáveis.
        * **A Regra da Verdade:** Lembre o desenvolvedor que o código muda, mas os comentários raramente são atualizados. Comentários mentem; o código é a única verdade.
    * **Exemplo de Review Sugerido pela IA:**
        * **Violação:** `// verifica se o cliente é maior de idade e tem saldo` seguido de um `if` complexo.
        * **Sugestão da IA:** "Este comentário está tentando compensar um código complexo. Remova o comentário e extraia a lógica do `if` para um método bem nomeado, como `canCompletePurchase(customer)`."
24. **Formatação e Organização:**
    * **Diretriz:** A formatação deve ser consistente e facilitar a leitura vertical.
    * **Identificação de Violação:** * Sinalize classes com milhares de linhas.
        * Identifique falta de afinidade conceitual (funções relacionadas que estão distantes uma da outra no arquivo).
    * **Sugestão de Refatoração:** Organize variáveis no topo da classe, seguidas pelas funções públicas. Mantenha funções chamadas logo abaixo das funções que as chamam (Stepdown Rule).
    * **Indentação:** Use estritamente 4 espaços para indentação básica, `case`, `throws` e `arrayInit`. Proíba o uso de caracteres de tabulação (`FileTabCharacter`).
    * **Limites:** Máximo de 150 caracteres por linha (`LineLength`).
    * **Espaçamento:** Garanta `GenericWhitespace`, `MethodParamPad` e `WhitespaceAround` adequados. Proíba espaços em branco no final das linhas (`trailing spaces`).
    * **Blocos:** Exija chaves (`NeedBraces`) mesmo para declarações de uma única linha. O uso de `LeftCurly` e `RightCurly` deve seguir o padrão "Same Line".
25. **Objetos e Estruturas de Dados:**
    * **Diretriz:** Objetos escondem dados e expõem funções. Estruturas de dados (DTOs/Records) expõem dados e não possuem funções significativas. Não misture os dois (evite objetos "híbridos").
    * **Identificação de Violação:** Sinalize classes que expõem todos os seus atributos via getters/setters (Anemic Domain Model) mas possuem lógica complexa.
    * **Sugestão de Refatoração:** Mova o comportamento para onde os dados estão ou mantenha a classe como um simples Record para transporte de dados.
    * Use estruturas adequadas para informações complexas; evite o uso excessivo de arrays.
    * Prefira o encapsulamento e evite getters/setters excessivos.
    * Use `Optional` com cautela.
    * Proíba instanciacões ilegais (`IllegalInstantiation`).
26. **Testes Limpos e TDD (The Craft of Testing)**
    * **Diretriz Geral:** O código de teste é tão importante quanto o código de produção. Ele não é um "cidadão de segunda classe". Testes sujos equivalem a não ter testes, pois dificultam a evolução do sistema.
    * **As Três Leis do TDD:**
        1. Você não pode escrever nenhum código de produção até que tenha escrito um teste de unidade que falhe.
        2. Você não pode escrever mais de um teste de unidade do que o suficiente para falhar (e não compilar é falhar).
        3. Você não pode escrever mais código de produção do que o suficiente para fazer o teste que falha passar.
        * *Ação da IA:* Se o código de produção parecer complexo demais para o teste existente, sugira que o desenvolvedor pode ter pulado etapas do TDD.
    * **Mantendo os Testes Limpos:**
        1. **Legibilidade:** É o fator mais importante. Use o padrão **AAA (Arrange, Act, Assert)** ou **Given-When-Then**.
        2. **Domínio do Teste:** Evite detalhes técnicos desnecessários (como setup complexo de mocks) dentro do método de teste. Extraia-os para métodos auxiliares.
        3. **Linguagem de Domínio:** O teste deve ser lido como um requisito de negócio.
    * **Uma Afirmação (Assert) por Teste:**
        * **Diretriz:** Cada teste deve validar apenas um conceito. Embora "um único assert" seja a meta ideal, o foco real é testar apenas uma coisa por método.
        * **Identificação de Violação:** Sinalize testes "misteriosos" que possuem múltiplos asserts validando fluxos diferentes. Divida-os em testes menores.
    * **Princípios F.I.R.S.T.:**
        * **F - Fast (Rápido):** Testes devem rodar rápido para que sejam executados o tempo todo.
        * **I - Independent (Independente):** Um teste não deve depender do resultado de outro. Eles devem poder rodar em qualquer ordem.
        * **R - Repeatable (Repetível):** Deve passar em qualquer ambiente (máquina local, CI, produção) sem falhas intermitentes (flakiness).
        * **S - Self-Validating (Auto-validável):** O teste deve ter um resultado booleano (passa ou falha). Você não deve ler um log para saber se funcionou.
        * **T - Timely (Oportuno):** O teste deve ser escrito *antes* do código de produção (conforme o TDD).
    * **Exemplos de Referência:**
        > **❌ TESTE SUJO (Múltiplos conceitos, difícil leitura):**
        > ```java
        > @Test
        > void testProcess() {
        >     User u = new User("John");
        >     orderService.process(u, new Item("Book"));
        >     assertEquals(1, repo.count());
        >     assertEquals("John", repo.findFirst().getName());
        >     // faz outra coisa totalmente diferente
        >     orderService.cancel(1);
        >     assertEquals(0, repo.count());
        > }
        > ```
        > **✅ TESTE LIMPO (Focado, legível, padrão AAA):**
        > ```java
        > @Test
        > void shouldIncrementRepositoryCountWhenOrderIsProcessed() {
        >     // Arrange
        >     User john = createStandardUser("John");
        >     Item book = createItem("Book");
        >
        >     // Act
        >     orderService.process(john, book);
        >
        >     // Assert
        >     assertEquals(1, repository.count());
        > }
        > ```
27. **Classes (Coesão e Estrutura)**
    * **Diretriz Geral:** Classes devem ser pequenas e ter apenas uma responsabilidade (SRP). O nome da classe deve descrever o que ela é, não apenas o que ela faz. Se não conseguirmos dar um nome preciso, a classe provavelmente está grande demais.
    * **Regras de Organização e Tamanho:**
        * **Organização Interna (Padrão Java):** As classes devem seguir a ordem de "descida" (Stepdown Rule):
            1. Lista de variáveis estáticas e públicas.
            2. Variáveis estáticas e privadas.
            3. Variáveis de instância privadas.
            4. (Pouco uso de variáveis públicas de instância).
            5. Construtores.
            6. Funções públicas.
            7. Funções privadas (devem aparecer logo após a função pública que as chama).
        * **Classes devem ser Pequenas:** * **Métrica de Responsabilidade:** O tamanho de uma classe não é medido apenas por linhas de código, mas por **responsabilidades**. 
            * **Identificação de Violação:** Sinalize classes com nomes genéricos (ex: `SuperManager`, `CommonProcessor`) ou que possuam muitos métodos públicos não relacionados.
            * **Coesão:** Uma classe deve ter um pequeno número de variáveis de instância. Cada método da classe deve manipular uma ou mais dessas variáveis. Se muitos métodos usam apenas uma parte das variáveis, divida a classe.
        * **Encapsulamento e Visibilidade:** * Mantenha a visibilidade de modificadores a mais restrita possível (`private` por padrão).
            * Siga a ordem correta de modificadores (ex: `public static final`).
        * **Composição sobre Herança:** Sinalize hierarquias de herança profundas ou que existam apenas para reaproveitar código. Sugira composição para maior flexibilidade.
        * **Evite God Classes (Classes Deus):** Bloqueie classes que tentam orquestrar todo o sistema ou que conhecem detalhes de muitos outros domínios.
    * **Exemplos de Referência:**
        > **❌ VIOLAÇÃO (Classe "Deus" e Desorganizada):**
        > ```java
        > public class DashboardManager {
        >     public void saveUser(User u) { ... }
        >     private String dbUrl;
        >     public void sendEmail() { ... }
        >     public void calculatePayroll() { ... }
        >     private void connectToDb() { ... }
        >     // Variáveis e métodos misturados, múltiplas responsabilidades
        > }
        > ```
        > **✅ CLASSE LIMPA (Pequena, Coesa e Organizada):**
        > ```java
        > public class PayrollCalculator {
        >     private final TaxService taxService;
        >     private final EmployeeRepository repository;
        >
        >     public PayrollCalculator(TaxService taxService, EmployeeRepository repository) {
        >         this.taxService = taxService;
        >         this.repository = repository;
        >     }
        >
        >     public MonthlyReport calculate(Employee employee) {
        >         double grossSalary = employee.getSalary();
        >         double taxes = taxService.calculateFor(grossSalary);
        >         return new MonthlyReport(grossSalary - taxes);
        >     }
        >     
        >     // Funções privadas viriam aqui, se necessário.
        > }
        > ```
28. **Dependências e Imports:**
    * Proíba `AvoidStarImport` (importações com `*`).
    * Remova `UnusedImports` e `RedundantImport`.
    * Dependa de abstrações e injete via construtor.
29. **Limites (Boundaries):**
    * Isole o código de bibliotecas externas criando wrappers para APIs de terceiros.
30. **Performance:**
    * Priorize código claro.
    * Use `StringBuilder` para concatenação em loops.
    * Evite `MagicNumber` (use constantes).
    * Prefira imutabilidade.
31. **Documentação:**
    * Mantenha JavaDoc/Swagger atualizados.
    * Arquivos devem terminar com uma nova linha (`NewlineAtEndOfFile`).
32. **Boas Práticas em Java:**
    * Use `try-with-resources`. Prefira `List`/`Set` a arrays.
    * Use `java.time` para datas. Use `enum` para constantes.
    * Use `final` sempre que possível. Implemente `equals()` e `hashCode()` juntos.
    * Simplifique expressões booleanas e retornos (`SimplifyBooleanReturn`).
    * **Pattern Matching:** Sugerir o uso de instanceof moderno e switch expressions (Java 17+).
33. **PMD e Análise Estática:**
    * Identifique variáveis não utilizadas e código morto.
    * Alerte sobre complexidade ciclomática elevada e métodos excessivamente longos.
    * Garanta que não existam blocos vazios ou atribuições internas (`InnerAssignment`).

34. **Padrões REST e Web API:**
    * **Semântica de Verbos:** 
      * `GET`: Apenas para leitura, deve ser idempotente e sem efeitos colaterais.
      * `POST`: Criação de recursos.
      * `PUT`: Atualização total (substituição).
      * `PATCH`: Atualização parcial.
      * `DELETE`: Remoção de recursos.
    * **Status Codes:** Use códigos apropriados: `201 Created` para POST bem-sucedido, `204 No Content` para DELETE, `400 Bad Request` para erros de validação, `404 Not Found` quando o recurso não existir.
    * **Nomenclatura de URIs:** Use substantivos no plural e hifens (kebab-case), nunca camelCase ou verbos (ex: `/orders-items` e não `/getOrders`).
    * **DTOs:** Nunca exponha Entidades JPA diretamente nos Controllers. Utilize DTOs distintos para Request e Response.
    * **Filtros e Paginação:** Para listagens (GET), exija paginação para evitar problemas de performance.
35. **OpenAPI & Swagger (API First):**
    * **Documentação:**
      * Toda operação deve ter `summary` e `description` claros. Parâmetros e propriedades de schemas devem possuir `description` explicativo.
    * **Nomenclatura de Schemas:**
      * Use PascalCase para schemas (ex: `CotacaoRequest`). Evite schemas inline; defina-os em `components/schemas` para promover o reuso.
    * **Tipagem Estrita:**
      * Sempre defina `format` para tipos básicos (ex: `type: string, format: date-time` ou `format: uuid`). Use `minimum`, `maximum` e `pattern` (regex) para validações.
    * **Required:**
      * Marque explicitamente as propriedades obrigatórias no schema.
    * **Exemplos:**
      * Forneça `example` para schemas e propriedades para facilitar o entendimento no Swagger UI.
    * **Erros:**
      * Defina modelos de erro padronizados (ex: `ErrorResponse`) para códigos 4xx e 5xx.
    * **Não editar código gerado pelo OpenAPI:**
      * Para estender DTOs gerados, use Composição ou ModelMapper (veja sub-item "Extensão de DTOs Gerados").
    * **Imutabilidade do Código Gerado:** Proíba terminantemente a edição manual de qualquer arquivo dentro de pastas de código gerado (ex: `target/generated-sources` ou pacotes configurados em `openapi-generator`). Se o código gerado não atende à necessidade, o ajuste deve ser feito no arquivo YAML original.
    * **Ciclo de Regeneração:** O código deve ser regenerado (ex: `mvn clean generate-sources`) sempre que houver qualquer alteração no arquivo YAML em `src/main/resources/openapi/`.
    * **Extensão de DTOs Gerados:** * Se for necessário adicionar comportamento ou campos extras a um DTO gerado, use **Composição** em uma nova classe (Wrapper) ou crie um **Mapeador (ModelMapper)** para converter o DTO gerado para um objeto de domínio/negócio.
        * Evite herança de DTOs gerados, a menos que a estratégia de `discriminator` esteja definida no OpenAPI.
    * **Sincronização:** Garanta que os nomes de métodos e parâmetros nos Controllers implementados correspondam exatamente às interfaces geradas pelo OpenAPI para evitar erros de compilação ou de runtime.
36. **Git e Commits:**
    * Mensagens de commit devem ser claras e descritivas, seguindo o padrão "Imperative Mood" (ex: "Add user authentication", "Fix null pointer exception").
    * Evite commits muito grandes; prefira commits pequenos e focados em uma única mudança.
    * Use branches para novas funcionalidades, correções de bugs ou experimentos, mantendo a branch principal estável.
37. **Revisão de Dependências:**
    * Verifique regularmente as dependências do projeto para vulnerabilidades de segurança e atualize-as conforme necessário.
38. **Logging:**
    * Não logue informações sensíveis (CPF completo, senhas, tokens). Dados sensíveis devem ser mascarados ou omitidos.
    * Use níveis de log apropriados: `ERROR` para falhas críticas, `WARN` para situações inesperadas, `INFO` para eventos normais e `DEBUG` para detalhes de diagnóstico.
    * Em Services de integração, sempre logue exceções capturadas.
39. **ModelMapper & Mapeamento de Objetos:**
    * **Configuração Centralizada:** O `ModelMapper` deve ser configurado como um Bean Spring único.
    * **Estratégia de Mapeamento:** Use `STRICT` (MatchingStrategy.STRICT) para evitar mapeamentos ambíguos ou acidentais em objetos complexos.
    * **Explicit Mapping:** Para campos com nomes diferentes ou lógicas complexas, exija a criação de um `PropertyMap` ou `TypeMap` explícito. Não confie apenas no mapeamento automático.
    * **Validação:** Sugira sempre o uso de `modelMapper.validate()` em testes unitários para garantir que todos os campos foram mapeados corretamente.
    * **Conversão de Records:** Como o projeto usa `Records`, certifique-se de que o ModelMapper esteja configurado para lidar com eles ou sugira o uso de construtores manuais caso o mapeamento automático falhe na imutabilidade.
40. **Gatilhos de Erro Crítico:**
    * **Erro Fatal:** Edição manual de código gerado (OpenAPI), exposição de segredos, ou falta de injeção via construtor.
41. **Encapsulamento (Proteção de Estado):**
    * Privacidade Absoluta: Todos os atributos de classe devem ser private. O uso de protected é permitido apenas em classes abstratas de infraestrutura.
    * Tell, Don't Ask: Proíba a extração de dados para tomada de decisão externa. O objeto deve exportar comportamento, não apenas dados. (Ex: Use user.isActive() em vez de user.getStatus().equals(ACTIVE)).
    * Imutabilidade de Coleções: Getters de listas/coleções devem retornar Collections.unmodifiableList() ou similar para evitar manipulação externa do estado interno.
    * Validação na Origem: O construtor (ou métodos de fábrica) deve garantir que o objeto nunca nasça em estado inválido.
42. **Herança (Uso Consciente):**
    * Composition over Inheritance: Priorize sempre a composição. Só aceite herança se houver uma relação clara de "é um" (is-a).
    * Final por Padrão: Classes que não foram expressamente desenhadas para extensão devem ser marcadas como final.
    * Limite de Hierarquia: Proíba hierarquias de herança com mais de 2 níveis de profundidade para evitar complexidade oculta.
    * Template Method: Em classes abstratas, os métodos que definem o fluxo principal devem ser final, deixando apenas os "hooks" (ganchos) para as subclasses.
43. **Polimorfismo e Extensibilidade:**
    * Programação para Interfaces: Variáveis, parâmetros e retornos devem sempre referenciar a Interface (ex: List) e nunca a implementação (ArrayList).
    * Eliminação de Instanceof: O uso de instanceof ou casts manuais é um code smell. Sugira a substituição por métodos polimórficos ou o padrão Strategy.
    * Switch Case vs Strategy: Substitua blocos switch ou if-else que decidem comportamento baseado em "tipo" por implementações da interface Strategy injetadas dinamicamente.
44. **Inversão de Dependência (DIP):**
    * Desacoplamento de Alto Nível: Módulos de negócio não devem depender de detalhes de implementação (ex: um Service não deve depender de um BCryptPasswordEncoder específico, mas de uma abstração de criptografia).
    * Injeção via Construtor Obrigatória: Proíba terminantemente @Autowired em campos (Field Injection). Exija o uso de final nos atributos e injeção via construtor (Lombok @RequiredArgsConstructor é aceitável, mas o construtor explícito é preferível para clareza).
    * Isolamento de Bibliotecas Externas: Proíba a injeção direta de classes de terceiros em Services. Crie um Wrapper ou Adapter de sua propriedade para isolar o core do sistema de mudanças na biblioteca.
    * Prevenção de Ciclos: Identifique e proíba dependências circulares entre Beans. Se A depende de B e B de A, a lógica comum deve ser movida para um terceiro componente (C).
45. **JPA e Hibernate (Persistência e Performance)**
    * **Estratégia de Fetch (N+1)**: * Proíba o uso de FetchType.EAGER em relacionamentos @OneToMany e @ManyToMany. O padrão deve ser sempre LAZY.
      * Identifique potenciais problemas de N+1 e sugira o uso de JOIN FETCH em consultas JPQL ou o uso de EntityGraph.
    * **Mapeamento de Relacionamentos:**
      * Em relacionamentos bidirecionais, exija o uso do atributo mappedBy no lado não proprietário para evitar tabelas de junção desnecessárias.
      * Exija métodos auxiliares (add/remove) em relacionamentos bidirecionais para manter os dois lados da associação sincronizados.
    * **Performance e Queries:**
      * Proíba o uso de findAll() em tabelas grandes sem paginação (Pageable).
      * Para operações de "apenas leitura", sugira o uso de @Transactional(readOnly = true) para que o Hibernate otimize o flush e o dirty checking.
      * Sugira Queries Nativas apenas quando houver necessidade de recursos específicos do banco de dados que o JPQL não suporte.
    * **Entidades e Ciclo de Vida:**
      * Identidade: Prefira GenerationType.SEQUENCE em vez de IDENTITY para permitir o batch insert do Hibernate.
      * Lombok em Entidades: Proíba o uso de @Data em entidades JPA devido a problemas de performance e recursão infinita no hashCode e equals. Use @Getter e @Setter explicitamente.
      * Equals e HashCode: Devem ser implementados usando apenas a chave primária (ID) ou uma chave de negócio estável, nunca campos mutáveis.
    * **Otimização de Memória:**
      * Em processos em lote (batch), exija o uso de entityManager.clear() e flush() periodicamente para evitar estouro da memória (L1 Cache).
      * Sugira Projeções (Interfaces ou DTOs no repositório) em vez de carregar entidades completas quando apenas alguns campos forem necessários.
    * **Tratamento de Transações:**
      * Garanta que a anotação @Transactional seja usada na camada de Service, nunca no Controller ou Repository.
      * Alerte sobre o uso de @Transactional em métodos private (onde não funcionam por causa do proxy do Spring).
46. **Odores e Heurísticas (Code Smells & Heuristics)**
    * **Diretriz Geral:** Identifique padrões que indicam fragilidade, rigidez ou opacidade no design do software. Se o código "cheira mal", ele deve ser refatorado imediatamente para evitar a podridão do software.
    * **Java: Especificidades de Design:**
        * **Não Herde as Constantes (The Constant Interface Pattern):** * **Violação:** Sinalize quando uma classe implementa uma interface apenas para ganhar acesso a constantes (`public interface MyConstants`). Isso é um uso impróprio da herança.
            * **Refatoração:** Sugira o uso de **Static Imports** de uma classe final de constantes ou, preferencialmente, mova as constantes para a classe onde elas são mais relevantes.
        * **Constantes versus Enums:**
            * **Violação:** Identifique o uso de `public static final int` ou `String` para representar um conjunto fixo de opções (ex: `STATUS_OPEN = 1`).
            * **Refatoração:** Exija o uso de **Enums**. Enums em Java são classes poderosas que podem ter métodos, atributos e comportamentos, oferecendo segurança de tipo que constantes primitivas não possuem.
    * **Heurísticas de Implementação (Sinalizar durante o Review):**
        * **G1: Níveis de Abstração Múltiplos:** Sinalize quando uma função mistura detalhes de implementação com regras de alto nível.
        * **G2: Código Morto:** Identifique funções que nunca são chamadas, variáveis nunca lidas ou ramos `if` impossíveis de serem atingidos. Remova-os.
        * **G3: Seletividade de Informação:** Uma classe não deve exportar muitos métodos ou variáveis. Mantenha a interface pequena e o acoplamento baixo.
        * **G4: Funções com Muitos Argumentos:** (Reiterando) Funções com mais de 3 argumentos devem ser sinalizadas como erro de design.
        * **G5: Argumentos de Saída:** Funções não devem alterar o estado de um objeto passado como argumento. Se a função deve alterar algo, que mude o estado do objeto no qual ela é chamada.
    * **Heurísticas de Java (J-Rules):**
        * **J1: Lista de Importações Longas (Wildcards):** Evite `import java.util.*`. Exija importações específicas para evitar colisões de nomes e clareza de dependências.
        * **J2: Não ignore exceções:** Blocos `catch` vazios ou que apenas imprimem o stack trace (`e.printStackTrace()`) sem tratar o erro devem ser bloqueados.
        * **J3: Constantes Simbólicas vs Literais:** Números mágicos (como `86400`) devem ser substituídos por constantes nomeadas (`SECONDS_IN_A_DAY`).
    * **Exemplos de Referência:**
        > **❌ VIOLAÇÃO (Herança de Constantes e Tipos Primitivos):**
        > ```java
        > // Interface de constantes (MÁ PRÁTICA)
        > public interface StatusConstants {
        >     int OPEN = 1;
        >     int CLOSED = 2;
        > }
        >
        > public class OrderService implements StatusConstants { // Herança indevida
        >     public void update(int status) { // Segurança de tipo fraca
        >         if (status == OPEN) { ... }
        >     }
        > }
        > ```
        > **✅ CONFORMIDADE (Enums e Encapsulamento):**
        > ```java
        > public enum OrderStatus {
        >     OPEN, CLOSED;
        >     
        >     public boolean canArchive() {
        >         return this == CLOSED;
        >     }
        > }
        >
        > public class OrderService {
        >     public void update(OrderStatus status) { // Segurança de tipo forte
        >         if (status.canArchive()) { ... }
        >     }
        > }
        > ```
47. **Refatoração: Identificando e Tratando Maus Cheiros (Code Smells)**
    * **Diretriz Geral:** Refatoração é o processo de alterar um sistema de software de modo que não mude o comportamento externo do código, mas melhore sua estrutura interna. Se um "mau cheiro" for detectado, a refatoração é obrigatória.
    * **Catálogo de Maus Cheiros e Ações:**
        | Mau Cheiro | Descrição | Técnica de Refatoração Sugerida |
        | :--- | :--- | :--- |
        | **Nome Misterioso** | Nomes de funções ou variáveis que não explicam o propósito. | **Rename Variable/Function** |
        | **Código Duplicado** | Mesma estrutura de código em mais de um lugar. | **Extract Method** ou **Pull Up Method** (se em subclasses). |
        | **Função Longa** | Métodos com muitas linhas ou responsabilidades. | **Extract Method**; procure por comentários que explicam blocos de código. |
        | **Lista Longa de Parâmetros** | Funções com mais de 3 ou 4 parâmetros. | **Introduce Parameter Object** ou **Preserve Whole Object**. |
        | **Dados Globais** | Variáveis acessíveis de qualquer lugar (ex: `public static`). | **Encapsulate Variable** (mover para classes específicas). |
        | **Dados Mutáveis** | Dados que mudam de valor constantemente, causando efeitos colaterais. | **Split Variable** ou **Replace Derived Variable with Query**. |
        | **Alteração Divergente** | Uma única classe é alterada constantemente por motivos diferentes. | **Extract Class** (separar as responsabilidades que mudam). |
        | **Cirurgia com Rifle** | Uma mudança exige pequenos ajustes em muitas classes diferentes. | **Move Method/Field** para centralizar a alteração em um lugar. |
        | **Agrupamentos de Dados** | Conjuntos de dados que sempre aparecem juntos (ex: DDD e Telefone). | **Extract Class** ou **Introduce Parameter Object**. |
        | **Obsessão por Primitivos** | Uso de strings/ints para conceitos (ex: `String email`, `int moeda`). | **Replace Primitive with Object** (criar classe `Email` ou `Money`). |
        | **Switches Repetidos** | O mesmo `switch` aparece em vários lugares do sistema. | **Replace Conditional with Polymorphism**. |
        | **Laços (Loops)** | Loops complexos que dificultam a leitura da intenção. | **Replace Loop with Pipeline** (uso de Streams/Lambdas). |
        | **Elemento Ocioso** | Classes ou métodos que não fazem quase nada. | **Inline Class** ou **Inline Function** (eliminar a estrutura). |
        | **Generalidade Especulativa** | Código criado para "casos futuros" que nunca ocorreram. | **Collapse Hierarchy** ou **Remove Dead Code**. |
        | **Campos Temporários** | Variáveis de instância que só fazem sentido em certas circunstâncias. | **Extract Class** para conter esses campos órfãos. |
        | **Cadeias de Mensagens** | Navegação longa entre objetos: `a.getB().getC().getD().doSomething()`. | **Hide Delegate** (o cliente não deve conhecer a estrutura interna). |
        | **Intermediário (Trocas Escuras)** | Uma classe que apenas delega todo o trabalho para outra. | **Remove Middleman** (falar diretamente com quem faz o trabalho). |
        | **Classe Grande** | Classes com excesso de variáveis de instância e código. | **Extract Class** ou **Extract Subclass**. |
        | **Classes Alt. com Interfaces Diff** | Duas classes fazem a mesma coisa, mas com métodos diferentes. | **Change Function Declaration** para alinhar as assinaturas. |
        | **Classe de Dados** | Classes que só possuem getters/setters e nenhuma lógica. | **Encapsulate Field** e mover lógica relacionada para dentro dela. |
        | **Herança Recursiva** | Onde cada vez que você cria uma subclasse de uma classe, tem que criar de outra. | **Move Method** e **Move Field** para achatar a hierarquia. |
    ---
    * **Exemplos de Refatoração Aplicar:**
    > **❌ VIOLAÇÃO (Obsessão por Primitivos e Agrupamento de Dados):**
    > ```java
    > // Mau cheiro: Dados que sempre andam juntos mas estão soltos
    > public void createOrder(String customerName, String street, String city, String zipCode, double amount) {
    >     // ...
    > }
    > ```
    > **✅ REFATORADO (Introduce Parameter Object):**
    > ```java
    > public void createOrder(Customer customer, Address deliveryAddress, Money amount) {
    >     // O código agora é tipado, legível e protege as regras de negócio
    > }
    > ```

    > **❌ VIOLAÇÃO (Cadeia de Mensagens / Lei de Demeter):**
    > ```java
    > // O cliente sabe demais sobre a estrutura interna
    > String zipCode = order.getCustomer().getAddress().getZipCode();
    > ```

    > **✅ REFATORADO (Hide Delegate):**
    > ```java
    > // O objeto esconde a delegação
    > String zipCode = order.getDeliveryZipCode();
    > ```**
48. **Refatoração: Encapsulamento e Proteção de Estado**
    * **Diretriz Geral:** Ocultar detalhes de implementação e proteger dados contra modificações indevidas. O código deve interagir com interfaces e comportamentos, não com a estrutura interna dos dados.
    * **Técnicas de Encapsulamento:**
        | Técnica | Quando aplicar | Ação de Refatoração |
        | :--- | :--- | :--- |
        | **Encapsulate Record** | Quando você usa mapas, hashes ou registros brutos onde a estrutura é oculta. | Converta o registro em uma **Classe** ou **Record** com nomes de campos claros. |
        | **Encapsulate Collection** | Quando um getter retorna uma referência direta a uma lista interna. | Retorne uma **cópia imutável** (ex: `List.copyOf`) e crie métodos `add/remove` específicos na classe. |
        | **Replace Primitive with Object** | Quando um dado simples tem lógica associada (ex: CPF, Telefone, Cor). | Crie um **Value Object** que valide o dado na construção. |
        | **Replace Temp with Query** | Quando uma variável temporária armazena o resultado de uma expressão. | Extraia a expressão para um método. Isso permite que a lógica seja reutilizada em outros lugares. |
        | **Extract Class** | Quando uma classe faz o trabalho de duas. | Mova parte dos campos e métodos para uma nova classe separada. |
        | **Inline Class** | Quando uma classe não está fazendo quase nada. | Mova todos os seus membros para outra classe e apague-a. |
        | **Hide Delegate** | Quando o cliente acessa um objeto através de outro (`a.getB().doC()`). | Crie um método delegado em `A` que encapsule a chamada para `B`. |
        | **Remove Middle Man** | Quando uma classe apenas delega todas as chamadas sem adicionar valor. | Faça o cliente chamar o objeto final diretamente (inverso de Hide Delegate). |
        | **Substitute Algorithm** | Quando você encontra uma maneira mais clara ou eficiente de resolver um problema. | Substitua o corpo do método pelo novo algoritmo mais simples. |
    ---
    * **Exemplos de Referência:**
        > **❌ VIOLAÇÃO (Coleção Exposta):**
        > ```java
        > public class Course {
        >     private List<Student> students;
        >     public List<Student> getStudents() { return students; } // Erro: permite modificação externa
        > }
        > ```
        > 
        >
        > **✅ REFATORADO (Encapsulate Collection):**
        > ```java
        > public class Course {
        >     private List<Student> students = new ArrayList<>();
        >     public List<Student> getStudents() { return Collections.unmodifiableList(students); }
        >     public void addStudent(Student student) { this.students.add(student); }
        > }
        > ```

        > **❌ VIOLAÇÃO (Cadeia de Mensagens):**
        > ```java
        > // O cliente precisa saber que o Departamento tem um Gerente
        > manager = person.getDepartment().getManager();
        > ```
        > 
        >
        > **✅ REFATORADO (Hide Delegate):**
        > ```java
        > // O cliente pede diretamente para a Pessoa
        > manager = person.getManager(); 
        > // Dentro de Person: public Manager getManager() { return department.getManager(); }
        > ```

        > **❌ VIOLAÇÃO (Replace Temp with Query):**
        > ```java
        > double basePrice = quantity * itemPrice;
        > if (basePrice > 1000) return basePrice * 0.95;
        > ```
        >
        > **✅ REFATORADO (Replace Temp with Query):**
        > ```java
        > if (basePrice() > 1000) return basePrice() * 0.95;
        > 
        > private double basePrice() { return quantity * itemPrice; }
        > ```
49. **Refatoração: Movendo Recursos e Organizando Instruções**
    * **Diretriz Geral:** O código deve estar onde é mais usado. Se uma função gasta mais tempo interagindo com outra classe do que com a sua própria, ela deve ser movida. Instruções devem ser agrupadas por afinidade para melhorar a clareza.
    * **Técnicas de Movimentação e Fluxo:**
        | Técnica | Quando aplicar | Ação de Refatoração |
        | :--- | :--- | :--- |
        | **Move Function** | Quando uma função interage mais com outra classe do que com a classe onde reside. | Mova a função para a classe de destino e transforme a original em uma delegação ou remova-a. |
        | **Move Field** | Quando um dado é mais utilizado por outra classe do que pela classe que o possui. | Mova o atributo e atualize todos os acessos para o novo local. |
        | **Move Statements into Function** | Quando um grupo de instruções é repetido toda vez que uma função é chamada. | Integre as instruções dentro da própria função chamada. |
        | **Move Statements to Callers** | Quando uma parte da função não faz mais sentido para todos os seus chamadores. | Mova essa parte da lógica para os locais onde a função é invocada. |
        | **Replace Inline Code with Function Call** | Quando você encontra um código que faz o mesmo que uma função já existente. | Remova o código duplicado e chame a função existente (DRY). |
        | **Slide Statements** | Quando instruções relacionadas estão espalhadas dentro de um método. | Mova as linhas de código para que fiquem adjacentes às suas dependências (afinidade). |
        | **Split Loop** | Quando um loop faz duas ou três coisas diferentes ao mesmo tempo. | Divida-o em loops separados (um para cada tarefa) para facilitar a leitura e futura extração. |
        | **Replace Loop with Pipeline** | Quando você usa um `for` ou `while` para processar coleções. | Substitua por operações de **Stream/Lambda** (filter, map, reduce). |
        | **Remove Dead Code** | Quando uma função, variável ou classe não é mais utilizada. | Exclua imediatamente. Se precisar no futuro, o Git terá o histórico. |
    ---
    * **Exemplos de Referência:**
        > **❌ VIOLAÇÃO (Loop com Múltiplas Responsabilidades):**
        > ```java
        > // Mau cheiro: O loop calcula o total e busca o funcionário mais novo
        > int totalSalary = 0;
        > Employee youngest = employees.get(0);
        > for (Employee e : employees) {
        >     totalSalary += e.getSalary();
        >     if (e.getAge() < youngest.getAge()) youngest = e;
        > }
        > ```
        > 
        >
        > **✅ REFATORADO (Split Loop + Pipeline):**
        > ```java
        > // Cada responsabilidade é clara e isolada
        > int totalSalary = employees.stream().mapToInt(Employee::getSalary).sum();
        > Employee youngest = employees.stream().min(Comparator.comparingInt(Employee::getAge)).orElse(null);
        > ```

        > **❌ VIOLAÇÃO (Instruções Desorganizadas):**
        > ```java
        > public void processOrder(Order order) {
        >     double discount = 0.1;
        >     log.info("Starting process"); // Instrução deslocada
        >     double price = order.getAmount() * (1 - discount);
        >     save(order);
        >     log.info("Order saved: " + order.getId());
        > }
        > ```
        >
        > **✅ REFATORADO (Slide Statements):**
        > ```java
        > public void processOrder(Order order) {
        >     log.info("Starting process");
        >     
        >     // Agrupamento por afinidade (cálculo)
        >     double discount = 0.1;
        >     double price = order.getAmount() * (1 - discount);
        >     
        >     // Agrupamento por afinidade (persistência)
        >     save(order);
        >     log.info("Order saved: " + order.getId());
        > }
        > ```

        > **❌ VIOLAÇÃO (Move Function):**
        > ```java
        > class Account {
        >     // Este método usa quase tudo de 'AccountType' e pouco de 'Account'
        >     double overdraftCharge() {
        >         if (type.isPremium()) { ... }
        >         else { ... }
        >     }
        > }
        > ```
        > 
        >
        > **✅ REFATORADO (Move Function):**
        > ```java
        > class AccountType {
        >     // O método agora reside onde os dados que ele mais usa estão
        >     double overdraftCharge(Account account) { ... }
        > }
        > ```
50. **Refatoração: Organizando Dados**
    * **Diretriz Geral:** Dados devem ser estruturados de forma a minimizar confusão e erros de estado. Variáveis devem ter responsabilidades únicas e campos devem ser nomeados com precisão. Evite armazenar dados que podem ser calculados e escolha sabiamente entre objetos de valor e referências.
    * **Técnicas de Organização de Dados:**
        | Técnica | Quando aplicar | Ação de Refatoração |
        | :--- | :--- | :--- |
        | **Split Variable** | Quando uma variável local é reatribuída para diferentes propósitos (exceto variáveis de loop). | Crie uma variável separada para cada responsabilidade. Cada variável deve ter apenas um significado. |
        | **Rename Field** | Quando o nome de um campo não reflete mais sua função ou é ambíguo. | Renomeie o campo e todos os seus acessos para um nome que revele a intenção. |
        | **Replace Derived Variable with Query** | Quando você armazena um dado que pode ser facilmente calculado a partir de outros campos. | Remova o campo e crie um método (query) que realize o cálculo dinamicamente, garantindo a "única fonte da verdade". |
        | **Change Reference to Value** | Quando você tem um objeto de referência que é pequeno, imutável e sua identidade não importa. | Transforme-o em um **Value Object** (imutável). Dois objetos são iguais se seus valores forem iguais. |
        | **Change Value to Reference** | Quando você tem muitos objetos iguais que precisam compartilhar o mesmo estado mutável ou identidade. | Transforme o objeto de valor em um **Objeto de Referência** único (geralmente gerenciado por um repositório ou cache). |
    ---
    * **Exemplos de Referência para a IA:**
        > **❌ VIOLAÇÃO (Variável com Dupla Responsabilidade):**
        > ```java
        > // Mau cheiro: 'temp' armazena o perímetro e depois a área
        > double temp = 2 * (height + width);
        > System.out.println(temp);
        > temp = height * width;
        > System.out.println(temp);
        > ```
        >
        > **✅ REFATORADO (Split Variable):**
        > ```java
        > final double perimeter = 2 * (height + width);
        > System.out.println(perimeter);
        > final double area = height * width;
        > System.out.println(area);
        > ```

        > **❌ VIOLAÇÃO (Variável Derivada/Redundante):**
        > ```java
        > class Order {
        >     private double price;
        >     private double discount;
        >     private double finalPrice; // Mau cheiro: campo derivado que pode ficar obsoleto
        >
        >     public void setPrice(double price) {
        >         this.price = price;
        >         this.finalPrice = price - discount; // Risco de erro de sincronização
        >     }
        > }
        > ```
        > 
        >
        > **✅ REFATORADO (Replace Derived Variable with Query):**
        > ```java
        > class Order {
        >     private double price;
        >     private double discount;
        >
        >     public double getFinalPrice() {
        >         return price - discount; // Sempre atualizado, sem estado redundante
        >     }
        > }
        > ```

        > **❌ VIOLAÇÃO (Reference vs Value):**
        > ```java
        > // Se mudarmos o DDD de uma instância, não queremos que afete outras acidentalmente
        > // mas Telefone aqui está sendo tratado como uma entidade mutável.
        > public class Contact {
        >     private Telephone telephone;
        > }
        > ```
        >
        > **✅ REFATORADO (Change Reference to Value):**
        > ```java
        > // Telefone agora é um Record (imutável) - um Value Object
        > public record Telephone(String ddi, String ddd, String number) {}
        > ```
51. **Refatoração: Simplificando Lógica Condicional**
    * **Diretriz Geral:** A lógica condicional tende a tornar-se complexa com o tempo. O objetivo desta refatoração é tornar a intenção do código óbvia, reduzindo o aninhamento e substituindo verificações procedimentais por estruturas orientadas a objetos.
    * **Técnicas de Simplificação:**
        | Técnica | Quando aplicar | Ação de Refatoração |
        | :--- | :--- | :--- |
        | **Decompose Conditional** | Quando tens um bloco `if-then-else` complexo. | Extrai os métodos para a condição (`isSpecialDeal`), o bloco `then` (`priceCharge`) e o bloco `else` (`regularCharge`). |
        | **Consolidate Conditional Expression** | Quando várias verificações levam ao mesmo resultado/ação. | Une as condições usando `&&` ou `||` e extrai o resultado para um único método com nome descritivo. |
        | **Replace Nested Conditional with Guard Clauses** | Quando tens um `if` dentro de outro `if` que torna o fluxo principal difícil de seguir. | Usa cláusulas de guarda (`return` ou `throw` antecipado) para casos especiais, mantendo o fluxo principal sem aninhamento. |
        | **Replace Conditional with Polymorphism** | Quando tens um `switch` ou vários `if-else` baseados no tipo do objeto. | Cria subclasses ou implementações de interface e move cada ramo da condicional para o método sobrescrito correspondente. |
        | **Introduce Special Case** | Quando tens verificações repetidas por valores nulos ou casos padrão (ex: `customer == null`). | Cria um objeto de caso especial (Null Object Pattern) que implementa o comportamento padrão, eliminando a necessidade do `if`. |
        | **Introduce Assertion** | Quando uma parte do código assume que certas condições são sempre verdadeiras para funcionar. | Adiciona uma asserção (`Assert`) para tornar o pressuposto explícito e ajudar no debug. |
    ---
    * **Exemplos de Referência para a IA:**
        > **❌ VIOLAÇÃO (Condicional Aninhada - Código em Seta):**
        > ```java
        > public double getPayAmount() {
        >     double result;
        >     if (isDead) result = deadAmount();
        >     else {
        >         if (isSeparated) result = separatedAmount();
        >         else {
        >             if (isRetired) result = retiredAmount();
        >             else result = normalPayAmount();
        >         }
        >     }
        >     return result;
        > }
        > ```
        >
        > **✅ REFATORADO (Guard Clauses):**
        > ```java
        > public double getPayAmount() {
        >     if (isDead) return deadAmount();
        >     if (isSeparated) return separatedAmount();
        >     if (isRetired) return retiredAmount();
        >     return normalPayAmount();
        > }
        > ```

        > **❌ VIOLAÇÃO (Condicional Baseada em Tipo):**
        > ```java
        > // Mau cheiro: sempre que surgir um novo tipo, este método tem de ser alterado
        > double getSpeed() {
        >     switch (type) {
        >         case EUROPEAN: return getBaseSpeed();
        >         case AFRICAN: return getBaseSpeed() - getLoadFactor() * numberOfCoconuts;
        >         case NORWEGIAN_BLUE: return (isNailed) ? 0 : getBaseSpeed(voltage);
        >     }
        >     throw new RuntimeException("Should be unreachable");
        > }
        > ```
        > 
        >
        > **✅ REFATORADO (Polymorphism):**
        > ```java
        > abstract class Bird {
        >     abstract double getSpeed();
        > }
        > class European extends Bird {
        >     double getSpeed() { return getBaseSpeed(); }
        > }
        > class African extends Bird {
        >     double getSpeed() { return getBaseSpeed() - getLoadFactor() * numberOfCoconuts; }
        > }
        > ```

        > **❌ VIOLAÇÃO (Verificação de Nulo Repetida):**
        > ```java
        > String name = (customer == null) ? "occupant" : customer.getName();
        > ```
        >
        > **✅ REFATORADO (Special Case / Null Object):**
        > ```java
        > // Onde UnknownCustomer implementa getName() para retornar "occupant"
        > String name = customer.getName(); 
        > ```
52. **Refatoração: Refatorando APIs e Contratos**
    * **Diretriz Geral:** Uma API deve ser clara quanto ao que faz e o que precisa. Evite funções "mágicas" que alteram estados enquanto respondem perguntas, e proteja o encapsulamento através de métodos de criação e acesso controlados.
    * **Técnicas de Refatoração de API:**
        | Técnica | Quando aplicar | Ação de Refatoração |
        | :--- | :--- | :--- |
        | **Separate Query from Modifier** | Quando uma função retorna um valor mas também altera o estado de um objeto. | Divida-a em duas: uma para a consulta (query) e outra para a alteração (modifier). |
        | **Parameterize Function** | Quando várias funções fazem coisas muito parecidas, mudando apenas um valor literal. | Una as funções em uma só, passando o valor diferente como parâmetro. |
        | **Remove Flag Argument** | Quando um parâmetro booleano dita qual caminho a função deve seguir. | Crie funções explícitas para cada caso (ex: `bookStandardOrder()` e `bookPremiumOrder()`). |
        | **Preserve Whole Object** | Quando você extrai vários valores de um objeto para passá-los a uma função. | Passe o objeto inteiro para a função, reduzindo o acoplamento da lista de parâmetros. |
        | **Replace Parameter with Query** | Quando um parâmetro pode ser obtido pela própria função chamando outro método. | Remova o parâmetro e faça a função obter o dado internamente. |
        | **Replace Query with Parameter** | Quando uma função tem uma dependência indesejada para obter um dado. | Passe o dado como parâmetro para tornar a função mais pura e testável. |
        | **Remove Setting Method** | Quando um campo não deve ser alterado após a criação do objeto. | Remova o `setter` e torne o campo `final` (imutável). |
        | **Replace Constructor with Factory** | Quando a criação exige lógica complexa ou você quer retornar subclasses. | Transforme o construtor em `private` e use um método estático de fábrica. |
        | **Replace Function with Command** | Quando uma função é muito complexa e precisa de muitas variáveis locais. | Transforme a função em uma classe própria (`Command Pattern`) com seu próprio estado. |
        | **Replace Command with Function** | Quando uma classe de comando é simples demais para o que faz. | Transforme a classe de volta em uma função simples. |
    ---
    * **Exemplos de Referência:**
        > **❌ VIOLAÇÃO (Mistura de Query e Modifier):**
        > ```java
        > // Mau cheiro: você só queria saber o total, mas a função disparou um alerta
        > public double getTotalAndAlertIfExpensive() {
        >     double total = calculateTotal();
        >     if (total > 1000) sendAlert();
        >     return total;
        > }
        > ```
        >
        > **✅ REFATORADO (Separate Query from Modifier):**
        > ```java
        > public double getTotal() { return calculateTotal(); }
        > public void alertIfExpensive(double total) { if (total > 1000) sendAlert(); }
        > ```

        > **❌ VIOLAÇÃO (Flag Argument):**
        > ```java
        > // O booleano 'isPremium' esconde a lógica
        > void setDimension(String name, boolean isPremium) { ... }
        > ```
        > 
        >
        > **✅ REFATORADO (Explicit Methods):**
        > ```java
        > void setPremiumDimension(String name) { ... }
        > void setStandardDimension(String name) { ... }
        > ```

        > **❌ VIOLAÇÃO (Parameter with Query):**
        > ```java
        > // O chamador não precisa calcular o preço para passar para o método
        > double basePrice = quantity * itemPrice;
        > double finalPrice = discountedPrice(basePrice);
        > ```
        >
        > **✅ REFATORADO (Replace Parameter with Query):**
        > ```java
        > // A própria função calcula o que precisa internamente
        > double finalPrice = discountedPrice(); 
        > ```
53. **Refatoração: Lidando com Herança e Hierarquia**
    * **Diretriz Geral:** A herança deve ser usada para expressar variações de comportamento real, não apenas para compartilhar código. Se uma hierarquia se torna complexa ou confusa, mova recursos para cima (generalização) ou para baixo (especialização), ou substitua a herança por delegação.
    * **Técnicas de Refatoração de Herança:**
        | Técnica | Quando aplicar | Ação de Refatoração |
        | :--- | :--- | :--- |
        | **Pull Up Method** | Quando subclasses possuem métodos com resultados idênticos. | Mova o método para a superclasse para eliminar a duplicação. |
        | **Pull Up Field** | Quando subclasses usam um campo da mesma maneira. | Mova o atributo para a superclasse. |
        | **Pull Up Constructor Body** | Quando subclasses têm construtores com lógica comum. | Mova a lógica comum para o construtor da superclasse e chame `super()`. |
        | **Push Down Method** | Quando um método na superclasse é relevante apenas para algumas subclasses. | Mova o método para as subclasses específicas que o utilizam. |
        | **Push Down Field** | Quando um campo na superclasse é usado apenas por algumas subclasses. | Mova o atributo para as subclasses que realmente precisam dele. |
        | **Replace Type Code with Subclasses** | Quando você tem um campo de "tipo" que altera o comportamento da classe. | Crie subclasses para cada tipo e use polimorfismo (ajuda no OCP). |
        | **Remove Subclass** | Quando uma subclasse faz muito pouco ou não é mais necessária. | Incorpore a lógica da subclasse na superclasse e remova-a. |
        | **Collapse Hierarchy** | Quando uma superclasse e uma subclasse são quase idênticas. | Combine-as em uma única classe para simplificar o design. |
        | **Replace Subclass with Delegate** | Quando a herança está causando rigidez ou você precisa mudar o comportamento em tempo de execução. | Transforme a relação de herança em composição/delegação. |
    ---
    * **Exemplos de Referência para a IA:**
        > **❌ VIOLAÇÃO (Duplicação em Subclasses):**
        > ```java
        > class Salesman extends Employee {
        >     private String name; // Duplicado
        > }
        > class Engineer extends Employee {
        >     private String name; // Duplicado
        > }
        > ```
        >
        > **✅ REFATORADO (Pull Up Field):**
        > ```java
        > class Employee {
        >     protected String name; // Centralizado na Superclasse
        > }
        > ```

        > **❌ VIOLAÇÃO (Replace Type Code with Subclasses):**
        > ```java
        > class Employee {
        >     private int type;
        >     static final int ENGINEER = 0;
        >     static final int SALESMAN = 1;
        >     
        >     double getPay() {
        >         switch (type) { // Mau cheiro: switch por tipo
        >             case ENGINEER: return salary;
        >             case SALESMAN: return salary + bonus;
        >             default: throw new RuntimeException("Invalid type");
        >         }
        >     }
        > }
        > ```
        >
        > **✅ REFATORADO (Subclasses + Polimorfismo):**
        > ```java
        > abstract class Employee {
        >     abstract double getPay();
        > }
        > class Engineer extends Employee {
        >     double getPay() { return salary; }
        > }
        > class Salesman extends Employee {
        >     double getPay() { return salary + bonus; }
        > }
        > ```

        > **❌ VIOLAÇÃO (Herança Rígida):**
        > ```java
        > // Um objeto não pode deixar de ser 'Premium' em tempo de execução se for herança
        > class PremiumBooking extends Booking { ... }
        > ```
        >
        > **✅ REFATORADO (Replace Subclass with Delegate):**
        > ```java
        > // Agora o comportamento pode ser trocado dinamicamente (Composição)
        > class Booking {
        >     private BookingDelegate delegate;
        >     void setPremium(boolean isPremium) {
        >         delegate = isPremium ? new PremiumDelegate(this) : null;
        >     }
        > }
        > ```
