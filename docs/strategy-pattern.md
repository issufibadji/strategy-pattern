# Padrão de Projeto Strategy — Sistema de Pagamentos

## O que é o Strategy?

O **Strategy** é um padrão comportamental que permite definir uma família de algoritmos, encapsulá-los e torná-los intercambiáveis em tempo de execução — sem alterar o código que os utiliza.

**Problema que resolve:** evitar grandes blocos `if/else` ou `switch` que crescem a cada nova variação de comportamento, violando o princípio Open/Closed (aberto para extensão, fechado para modificação).

---

## Estrutura do Projeto

```
src/
├── NoStrategy.java       # Implementação sem padrão (problema)
├── CommonStrategy.java   # Strategy clássico (interface + classes)
└── ModernStrategy.java   # Strategy moderno (enum + method reference)
```

---

## 1. Sem padrão — `NoStrategy.java`

O ponto de partida: lógica de pagamento espalhada em `if/else` encadeados dentro do `main`.

```java
if (paymentMethod == 1) {
    System.out.println("Validing credit card...");
    System.out.println("Paid R$ " + amount + " with credit card.");
} else if (paymentMethod == 2) {
    System.out.println("Generating barcode...");
    System.out.println("Paid R$ " + amount + " with boleto.");
} else if (paymentMethod == 3) {
    System.out.println("Generating QR Code...");
    System.out.println("Paid R$ " + amount + " with Pix.");
} else {
    throw new IllegalStateException("Payment method not supported.");
}
```

**Problemas:**
- Adicionar um novo método de pagamento exige editar esse bloco.
- A lógica de cada pagamento fica misturada no mesmo lugar.
- Difícil de testar individualmente.

---

## 2. Strategy Clássico — `CommonStrategy.java`

Abordagem tradicional: uma **interface** define o contrato e cada **classe concreta** implementa a estratégia.

### Diagrama de participantes

```
PaymentProcessor ──uses──> «interface» PaymentMethod
                                  ▲
                    ┌─────────────┼─────────────┐
                 CreditCard    Boleto           Pix
```

### Código-chave

```java
// Contrato
interface PaymentMethod {
    void pay(double amount);
}

// Estratégias concretas
class CreditCard implements PaymentMethod { ... }
class Boleto      implements PaymentMethod { ... }
class Pix         implements PaymentMethod { ... }

// Contexto
class PaymentProcessor {
    private PaymentMethod paymentMethod;

    public void pay(double amount) {
        paymentMethod.pay(amount);
    }
}

// Uso
var paymentMethod = switch (paymentMethodCode) {
    case 1 -> new CreditCard();
    case 2 -> new Boleto();
    case 3 -> new Pix();
    default -> throw new IllegalStateException("Payment method not supported.");
};
var processor = new PaymentProcessor(paymentMethod);
processor.pay(amount);
```

**Vantagens sobre `NoStrategy`:**
- Cada estratégia é isolada em sua própria classe.
- Adicionar um novo método de pagamento = criar uma nova classe, sem alterar as existentes.
- Fácil de testar cada estratégia separadamente.

**Desvantagem:** proliferação de classes (uma por estratégia), mesmo para lógicas simples.

---

## 3. Strategy Moderno — `ModernStrategy.java`

Abordagem enxuta: usa **enum** + **referência de método** (`Consumer<Double>`), eliminando as classes intermediárias.

### Código-chave

```java
// Implementações estáticas agrupadas
class PaymentMethods {
    public static void creditCard(Double amount) { ... }
    public static void boleto(Double amount)     { ... }
    public static void pix(Double amount)        { ... }
}

// Enum carrega a estratégia como função
enum PaymentType {
    CREDIT_CARD(PaymentMethods::creditCard),
    BOLETO     (PaymentMethods::boleto),
    PIX        (PaymentMethods::pix);

    private final Consumer<Double> paymentStrategy;

    PaymentType(Consumer<Double> paymentStrategy) {
        this.paymentStrategy = paymentStrategy;
    }

    public void pay(Double amount) {
        paymentStrategy.accept(amount);
    }
}

// Uso
PaymentType.valueOf("PIX").pay(3.0);
```

**Vantagens sobre o Strategy clássico:**
- Nenhuma classe extra de interface/implementação.
- O enum serve de registro central de todos os métodos de pagamento disponíveis.
- `valueOf()` já valida o tipo — entrada inválida lança `IllegalArgumentException` automaticamente.
- Código muito mais conciso e fácil de navegar.

**Quando preferir o clássico:** quando cada estratégia tem estado próprio, dependências injetadas, ou lógica complexa demais para um método estático.

---

## Comparativo rápido

| Critério              | Sem padrão | Strategy Clássico | Strategy Moderno |
|-----------------------|:----------:|:-----------------:|:----------------:|
| Extensível            | ✗          | ✓                 | ✓                |
| Testável isoladamente | ✗          | ✓                 | ✓                |
| Nº de arquivos/classes| 1          | Alto              | Baixo            |
| Suporte a estado      | —          | ✓                 | ✗ (estático)     |
| Legibilidade          | Ruim       | Boa               | Muito boa        |

---

## Como executar

Cada arquivo é autocontido. Compile e execute individualmente:

```bash
# Sem padrão
javac src/NoStrategy.java && java -cp src NoStrategy

# Strategy clássico
javac src/CommonStrategy.java && java -cp src CommonStrategy

# Strategy moderno
javac src/ModernStrategy.java && java -cp src ModernStrategy
```

Ou abra o projeto em qualquer IDE Java (IntelliJ, VS Code + Extension Pack for Java) e execute o `main` desejado.
