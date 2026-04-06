# clean-code

# Clean Code — Guia Completo com TypeScript

> Baseado no livro *Clean Code: A Handbook of Agile Software Craftsmanship* de Robert C. Martin (Uncle Bob)

---

## 1. O QUE É CÓDIGO LIMPO?

Código limpo é código que **qualquer desenvolvedor consegue ler e entender facilmente**. Não basta funcionar — precisa comunicar intenção com clareza.

> *"Código ruim pode funcionar, mas ele vai te derrubar ao longo do tempo."* — Robert C. Martin

---

## 2. NOMES SIGNIFICATIVOS

Escolher bons nomes é a habilidade mais fundamental.

### Use nomes que revelam intenção

```typescript
// Ruim
const d = 86400;
const list1: number[] = [];

// Bom
const SECONDS_PER_DAY = 86400;
const activeUserIds: number[] = [];
```

### Evite desinformação

```typescript
// Ruim — não é uma lista, é um Map
const userList = new Map<number, User>();

// Bom
const usersById = new Map<number, User>();
```

### Faça distinções significativas

```typescript
// Ruim — o que diferencia a1 de a2?
function copy(a1: string[], a2: string[]) { ... }

// Bom
function copy(source: string[], destination: string[]) { ... }
```

### Use nomes pronunciáveis e buscáveis

```typescript
// Ruim
const ymdhms = new Date();
const MAX_U = 7;

// Bom
const timestamp = new Date();
const MAX_USERS_PER_CLASS = 7;
```

### Nomes de classes: substantivos. Nomes de funções: verbos.

```typescript
// Classes são coisas
class UserRepository {}
class PaymentProcessor {}

// Funções fazem coisas
function sendEmail(to: string) {}
function calculateTax(price: number) {}
```

---

## 3. FUNÇÕES

### Devem ser pequenas — e fazer UMA coisa só

```typescript
// Ruim — faz várias coisas
function processUser(user: User) {
  if (!user.email.includes('@')) throw new Error('Invalid email');
  user.name = user.name.trim();
  db.save(user);
  emailService.send(user.email, 'Welcome!');
}

// Bom — cada função tem uma responsabilidade
function validateUser(user: User): void {
  if (!user.email.includes('@')) throw new Error('Invalid email');
}

function normalizeUser(user: User): User {
  return { ...user, name: user.name.trim() };
}

function registerUser(user: User): void {
  validateUser(user);
  const normalized = normalizeUser(user);
  db.save(normalized);
  emailService.send(normalized.email, 'Welcome!');
}
```

### Poucos argumentos — idealmente 0, 1 ou 2

```typescript
// Ruim — 5 parâmetros são difíceis de lembrar e ordenar
function createUser(name: string, email: string, age: number, role: string, active: boolean) {}

// Bom — use um objeto de configuração
interface CreateUserDTO {
  name: string;
  email: string;
  age: number;
  role: string;
  active: boolean;
}

function createUser(dto: CreateUserDTO): User {}
```

### Sem efeitos colaterais ocultos

```typescript
// Ruim — o nome diz "verificar senha", mas cria sessão como efeito colateral
function checkPassword(username: string, password: string): boolean {
  const user = db.findUser(username);
  if (bcrypt.compare(password, user.hash)) {
    sessionService.initialize(user); // efeito colateral oculto!
    return true;
  }
  return false;
}

// Bom — separar responsabilidades
function isPasswordValid(username: string, password: string): boolean {
  const user = db.findUser(username);
  return bcrypt.compare(password, user.hash);
}

function login(username: string, password: string): void {
  if (!isPasswordValid(username, password)) throw new Error('Invalid credentials');
  sessionService.initialize(db.findUser(username));
}
```

### Prefira exceções a códigos de erro

```typescript
// Ruim — forçar o chamador a verificar código de retorno
function deleteUser(id: number): number {
  if (!db.exists(id)) return -1;
  db.delete(id);
  return 0;
}

// Bom
function deleteUser(id: number): void {
  if (!db.exists(id)) throw new UserNotFoundError(id);
  db.delete(id);
}
```

---

## 4. COMENTÁRIOS

> *"O único comentário verdadeiramente bom é aquele que você encontrou uma maneira de não escrever."*

### Comentários ruins são desculpa para código ruim

```typescript
// Ruim — comentário explicando código confuso
// verifica se o funcionário tem direito a benefícios
if (employee.flags & HOURLY_FLAG && employee.age > 65) {}

// Bom — o código se explica
if (employee.isEligibleForBenefits()) {}
```

### Comentários que são aceitáveis

```typescript
// Explicando uma decisão não óbvia
// Usamos SHA-256 aqui porque MD5 tem colisões conhecidas
const hash = crypto.createHash('sha256').update(data).digest('hex');

// TODO legítimos
// TODO: substituir por chamada à API quando o endpoint estiver disponível
function getFeatureFlag(name: string): boolean {
  return true;
}

// Aviso de consequências
// ATENÇÃO: este método deleta registros permanentemente sem soft-delete
function hardDeleteUser(id: number): void { ... }
```

### Nunca deixe código comentado

```typescript
// Nunca faça isso — use controle de versão (Git)
// function oldProcessPayment(amount: number) {
//   const fee = amount * 0.05;
//   return amount + fee;
// }
```

---

## 5. FORMATAÇÃO

### Arquivos pequenos são melhores

- Uncle Bob recomenda **200 linhas** como ideal, máximo de 500.

### Espaçamento vertical como parágrafos

```typescript
// Bom — conceitos separados por linha em branco
class OrderService {

  constructor(private readonly repo: OrderRepository) {}

  async createOrder(dto: CreateOrderDTO): Promise<Order> {
    const order = Order.create(dto);
    return this.repo.save(order);
  }

  async cancelOrder(id: string): Promise<void> {
    const order = await this.repo.findById(id);
    order.cancel();
    await this.repo.save(order);
  }
}
```

### Funções dependentes devem ficar próximas

```typescript
// Bom — chamada e chamado próximos no arquivo
function buildReport(data: ReportData): string {
  const header = buildHeader(data.title);
  const body = buildBody(data.rows);
  return `${header}\n${body}`;
}

function buildHeader(title: string): string {
  return `=== ${title.toUpperCase()} ===`;
}

function buildBody(rows: string[][]): string {
  return rows.map(row => row.join(' | ')).join('\n');
}
```

---

## 6. OBJETOS E ESTRUTURAS DE DADOS

### A diferença fundamental

| | **Objeto** | **Estrutura de Dados** |
|---|---|---|
| Expõe | Comportamento (métodos) | Dados |
| Esconde | Dados internos | Lógica |
| Fácil de... | Adicionar novos tipos | Adicionar novas funções |

```typescript
// Estrutura de dados (procedural)
interface Circle { radius: number; }
interface Rectangle { width: number; height: number; }

function area(shape: Circle | Rectangle): number {
  if ('radius' in shape) return Math.PI * shape.radius ** 2;
  return shape.width * shape.height;
}

// Objeto (OOP) — fácil de adicionar novos tipos
abstract class Shape {
  abstract area(): number;
}

class Circle extends Shape {
  constructor(private radius: number) { super(); }
  area() { return Math.PI * this.radius ** 2; }
}

class Rectangle extends Shape {
  constructor(private width: number, private height: number) { super(); }
  area() { return this.width * this.height; }
}
```

### Lei de Demeter — não fale com estranhos

```typescript
// Ruim — o código sabe demais sobre a estrutura interna
const city = user.getAddress().getCity().getName();

// Bom — User expõe o que o chamador precisa
const city = user.getCityName();
```

---

## 7. TRATAMENTO DE ERROS

### Use exceções tipadas

```typescript
// Hierarquia de erros
class AppError extends Error {
  constructor(message: string, public readonly code: string) {
    super(message);
    this.name = this.constructor.name;
  }
}

class NotFoundError extends AppError {
  constructor(resource: string, id: string | number) {
    super(`${resource} with id ${id} not found`, 'NOT_FOUND');
  }
}

class ValidationError extends AppError {
  constructor(message: string) {
    super(message, 'VALIDATION_ERROR');
  }
}
```

### Não retorne null — prefira objetos especiais ou exceções

```typescript
// Ruim — força verificação de null em todo lugar
function findUser(id: number): User | null {
  return db.find(id) ?? null;
}
const user = findUser(1);
if (user !== null) { ... } // fácil de esquecer

// Opção 1 — lançar exceção
function getUser(id: number): User {
  const user = db.find(id);
  if (!user) throw new NotFoundError('User', id);
  return user;
}

// Opção 2 — Null Object Pattern
class GuestUser implements User {
  name = 'Guest';
  email = '';
  hasPermission(_action: string) { return false; }
}
```

### Não ignore exceções

```typescript
// Ruim
try {
  processPayment(order);
} catch (e) {
  // silêncio... 
}

// Bom
try {
  processPayment(order);
} catch (error) {
  logger.error('Payment failed', { orderId: order.id, error });
  notifySupport(order, error);
  throw new PaymentError('Payment could not be processed');
}
```

---

## 8. LIMITES (BOUNDARIES)

Ao usar código de terceiros, isole a dependência:

```typescript
// Ruim — Axios espalhado por toda a aplicação
import axios from 'axios';

class UserService {
  async getUser(id: number) {
    return axios.get(`/users/${id}`); // acoplado ao Axios
  }
}

// Bom — dependência isolada atrás de uma interface
interface HttpClient {
  get<T>(url: string): Promise<T>;
}

class AxiosHttpClient implements HttpClient {
  async get<T>(url: string): Promise<T> {
    const response = await axios.get<T>(url);
    return response.data;
  }
}

class UserService {
  constructor(private http: HttpClient) {}

  async getUser(id: number): Promise<User> {
    return this.http.get<User>(`/users/${id}`);
  }
}
```

---

## 9. TESTES UNITÁRIOS (TDD)

### As 3 leis do TDD

1. Não escreva código de produção antes de ter um teste que falha
2. Não escreva mais testes do que o suficiente para falhar
3. Não escreva mais código do que o suficiente para passar o teste

### Testes limpos: um assert por conceito

```typescript
// Ruim — testa muitas coisas de uma vez
test('user', () => {
  const user = new User('Alice', 'alice@example.com');
  expect(user.name).toBe('Alice');
  expect(user.email).toBe('alice@example.com');
  user.deactivate();
  expect(user.active).toBe(false);
  expect(user.deactivatedAt).toBeDefined();
});

// Bom — cada teste tem um foco claro
describe('User', () => {
  test('should store name and email on creation', () => {
    const user = new User('Alice', 'alice@example.com');
    expect(user.name).toBe('Alice');
    expect(user.email).toBe('alice@example.com');
  });

  test('should mark user as inactive when deactivated', () => {
    const user = new User('Alice', 'alice@example.com');
    user.deactivate();
    expect(user.active).toBe(false);
  });

  test('should record deactivation timestamp', () => {
    const user = new User('Alice', 'alice@example.com');
    user.deactivate();
    expect(user.deactivatedAt).toBeDefined();
  });
});
```

### F.I.R.S.T — qualidades de bons testes

| Letra | Princípio | Significado |
|---|---|---|
| **F** | Fast | Devem rodar rápido |
| **I** | Independent | Não devem depender uns dos outros |
| **R** | Repeatable | Mesmo resultado em qualquer ambiente |
| **S** | Self-validating | Retornam verdadeiro ou falso, sem análise manual |
| **T** | Timely | Escritos antes do código de produção (TDD) |

---

## 10. CLASSES

### Pequenas e com responsabilidade única (SRP)

```typescript
// Ruim — faz tudo: lógica de negócio, formatação e persistência
class Employee {
  calculatePay(): number { ... }
  generateReport(): string { ... }
  save(): void { ... }
}

// Bom — cada classe tem uma razão para mudar
class Employee {
  calculatePay(): number { ... }
}

class EmployeeReporter {
  generateReport(employee: Employee): string { ... }
}

class EmployeeRepository {
  save(employee: Employee): void { ... }
}
```

### Alta coesão

Todos os atributos da classe devem ser usados pela maioria dos métodos.

```typescript
// Baixa coesão — atributos usados por métodos diferentes
class DataProcessor {
  private data: string[] = [];
  private dbConnection: Connection;
  private logger: Logger;

  processData() { /* usa só data */ }
  saveToDb() { /* usa só dbConnection */ }
  logError() { /* usa só logger */ }
}

// Alta coesão — divisão de responsabilidades
class DataProcessor {
  constructor(private data: string[]) {}
  process(): ProcessedData { ... }
}

class DataRepository {
  constructor(private db: Connection) {}
  save(data: ProcessedData): void { ... }
}
```

### Prefira composição à herança

```typescript
// Herança para reutilizar comportamento
class Animal {
  breathe() { console.log('breathing'); }
}
class Car extends Animal {} // Um carro não é um animal!

// Composição
class Engine {
  start() { console.log('vroom'); }
}

class Car {
  constructor(private engine: Engine) {}
  drive() { this.engine.start(); }
}
```

---

## 11. OS PRINCÍPIOS SOLID

### S — Single Responsibility (Responsabilidade Única)
Uma classe deve ter apenas um motivo para mudar. *(Ver exemplo de Employee acima)*

### O — Open/Closed (Aberto/Fechado)

```typescript
// Ruim — precisa modificar a função para cada novo desconto
function applyDiscount(order: Order, type: string): number {
  if (type === 'vip') return order.total * 0.9;
  if (type === 'student') return order.total * 0.85;
  return order.total;
}

// Bom — aberto para extensão, fechado para modificação
interface DiscountStrategy {
  apply(total: number): number;
}

class VipDiscount implements DiscountStrategy {
  apply(total: number) { return total * 0.9; }
}

class StudentDiscount implements DiscountStrategy {
  apply(total: number) { return total * 0.85; }
}

function applyDiscount(order: Order, strategy: DiscountStrategy): number {
  return strategy.apply(order.total);
}
```

### L — Liskov Substitution (Substituição de Liskov)

Subclasses devem poder substituir a classe pai sem quebrar o comportamento.

```typescript
// Quebra Liskov — Square muda o comportamento herdado
class Rectangle {
  constructor(protected width: number, protected height: number) {}
  setWidth(w: number) { this.width = w; }
  setHeight(h: number) { this.height = h; }
  area() { return this.width * this.height; }
}

class Square extends Rectangle {
  setWidth(w: number) { this.width = this.height = w; } // comportamento diferente
  setHeight(h: number) { this.width = this.height = h; }
}

// Bom — usar composição ou interface separada
interface Shape { area(): number; }

class Rectangle implements Shape {
  constructor(private w: number, private h: number) {}
  area() { return this.w * this.h; }
}

class Square implements Shape {
  constructor(private side: number) {}
  area() { return this.side ** 2; }
}
```

### I — Interface Segregation (Segregação de Interfaces)

```typescript
// Ruim — interface gorda que força implementações desnecessárias
interface Worker {
  work(): void;
  eat(): void;
  sleep(): void;
}

class Robot implements Worker {
  work() { ... }
  eat() { throw new Error("Robots don't eat!"); } 
  sleep() { throw new Error("Robots don't sleep!"); } 
}

// Bom — interfaces segregadas
interface Workable { work(): void; }
interface Eatable { eat(): void; }
interface Sleepable { sleep(): void; }

class Robot implements Workable {
  work() { ... }
}

class Human implements Workable, Eatable, Sleepable {
  work() { ... }
  eat() { ... }
  sleep() { ... }
}
```

### D — Dependency Inversion (Inversão de Dependência)

```typescript
// Ruim — depende de implementação concreta
class OrderService {
  private mailer = new SendGridMailer(); // 💥 acoplado

  completeOrder(order: Order) {
    this.mailer.send(order.userEmail, 'Order complete!');
  }
}

// Bom — depende de abstração
interface Mailer {
  send(to: string, message: string): Promise<void>;
}

class OrderService {
  constructor(private mailer: Mailer) {} // injeção de dependência

  async completeOrder(order: Order): Promise<void> {
    await this.mailer.send(order.userEmail, 'Order complete!');
  }
}
```

---

## 12. SISTEMAS

### Separar construção de uso

```typescript
// Ruim — construção misturada com lógica
class App {
  private service = new UserService(new UserRepository(new DbConnection()));
  // ...
}

// Bom — composição centralizada (IoC Container / Factory)
function buildApp() {
  const db = new DbConnection(process.env.DATABASE_URL);
  const repo = new UserRepository(db);
  const service = new UserService(repo);
  return new App(service);
}
```

### Emergência — As 4 regras de Kent Beck (em ordem de prioridade)

1. **Passa todos os testes**
2. **Sem duplicação** (DRY — Don't Repeat Yourself)
3. **Expressa a intenção do programador**
4. **Minimiza o número de classes e métodos**

---

## 13. CONCORRÊNCIA

### Princípios básicos

```typescript
// Perigoso — estado compartilhado mutável
let counter = 0;
async function increment() {
  const current = counter;
  await someAsyncWork();
  counter = current + 1; // race condition!
}

// Bom — evitar estado compartilhado ou usar sincronização
class Counter {
  private value = 0;

  async increment(): Promise<void> {
    // operação atômica
    this.value += 1;
  }

  get count() { return this.value; }
}
```

### Mantenha seções sincronizadas pequenas e limite o escopo dos dados compartilhados.

---

## 14. CHEIROS DE CÓDIGO (CODE SMELLS)

Os principais sinais de que algo está errado:

| Smell | Descrição | Solução |
|---|---|---|
| **Código duplicado** | Mesma lógica em dois lugares | Extrair função/classe |
| **Função longa** | Mais de 20 linhas | Extrair funções |
| **Lista longa de parâmetros** | Mais de 3 params | Objeto de configuração |
| **God Class** | Classe que faz tudo | Dividir por responsabilidade |
| **Feature Envy** | Método usa mais dados de outra classe | Mover o método |
| **Números mágicos** | `if (status === 4)` | Usar constantes nomeadas |
| **Comentários óbvios** | `// incrementa i` sobre `i++` | Remover |
| **Switch statements** | Switch no tipo do objeto | Polimorfismo |
| **Nomes sem significado** | `x`, `temp`, `data2` | Renomear |

---

## 15. REFATORAÇÃO

Refatorar é melhorar o design do código **sem alterar o comportamento**.

```typescript
// Antes
function calc(e: any) {
  let r = 0;
  for (let i = 0; i < e.items.length; i++) {
    r += e.items[i].p * e.items[i].q;
  }
  if (e.type === 'vip') r = r * 0.9;
  return r;
}

// Depois — após refatoração progressiva
function calculateOrderTotal(order: Order): number {
  const subtotal = sumItemPrices(order.items);
  return applyCustomerDiscount(subtotal, order.customer);
}

function sumItemPrices(items: OrderItem[]): number {
  return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
}

function applyCustomerDiscount(total: number, customer: Customer): number {
  if (customer.isVip) return total * 0.9;
  return total;
}
```

---

## RESUMO FINAL — Os Mandamentos do Clean Code

```
- Nomes revelam intenção
- Funções fazem UMA coisa
- Quanto menor, melhor (funções, classes, arquivos)
- Sem efeitos colaterais ocultos
- Trate erros com exceções tipadas, nunca ignore
- Testes limpos: rápidos, independentes, legíveis
- SOLID: SRP, OCP, LSP, ISP, DIP
- Prefira composição à herança
- Sem duplicação (DRY)
- Deixe o código mais limpo do que encontrou (Regra do Escoteiro)
```

> *"A regra do escoteiro: sempre deixe o acampamento mais limpo do que você o encontrou. Se você checar um código sujo, limpe-o."* — Robert C. Martin

---
