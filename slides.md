---
marp: true
theme: default
paginate: true
---

# Builder Design Pattern

*A clean approach to complex object creation that enhances readability and maintainability*

---

## 1) The Problem: Complex Object Creation

Creating objects with many optional parameters leads to several challenges:

- **Telescoping constructors**: Long parameter lists that are hard to read and maintain
  ```javascript
  // What do these booleans mean? Which parameters are required?
  const pizza = new Pizza("Large", "Thin", true, false, true, false);
  ```

- **Object mutation**: Scattered setup code that's difficult to track
  ```javascript
  const chart = new Chart();
  chart.setType('bar');
  chart.setLegend(true);
  chart.setAxes({ x: 'date', y: 'value' });
  // ... many more lines of configuration
  ```

- **Repeated construction patterns**: Same build steps duplicated across the codebase

---

## 2) The Builder Pattern: A Solution

The Builder pattern is a creational design pattern that:

- Constructs complex objects **step by step**
- **Separates** construction process from representation
- Allows the **same construction process** to create different representations

---

## 3) When to Use Builder

Consider using the Builder pattern when:

- Objects have **many optional parameters** or configurations
- Object creation involves **multiple steps** in a specific sequence
- The same construction process should create **different representations**
- You want to **encapsulate** the knowledge of how to build a complex object

---

## 4) Pattern Structure

```
Client
  │
  ├───> Director (optional): Provides standard recipes
  │         │
  │         └───> Builder Interface: Defines construction steps
  │                          │
  │                          └───> Concrete Builders: Implement steps differently
  │                                           │
  │                                           └───> Products: Different representations
```

**Key concept**: The same construction steps can produce entirely different outputs.

---

## 5) Real-World Analogies

- **Restaurant order**: Same ordering process (select size → choose base → add toppings) creates different meals
- **Document generation**: Same data can produce PDF, HTML, or plain text outputs
- **API request construction**: Method → headers → authentication → body → send

---

## 6) Implementing the Pattern: Pizza Example

Let's build a pizza ordering system with two outputs: a Pizza object and a receipt.

### The Product

```javascript
class Pizza {
  constructor() {
    this.size = null;        // required
    this.crust = null;       // required
    this.cheese = false;     // optional
    this.toppings = [];      // optional collection
    this.extras = [];        // optional collection
  }
  
  describe() {
    return `${this.size} ${this.crust} crust pizza` +
           `${this.cheese ? ' with cheese' : ''}` +
           `${this.toppings.length ? ' topped with ' + this.toppings.join(', ') : ''}`;
  }
}
```

---

### The Builder Interface

In TypeScript, we would define an interface. In JavaScript, we'll document the expected methods:

```javascript
/**
 * Pizza Builder Interface
 * 
 * Methods:
 * - reset(): Builder - Prepares builder for a new product
 * - setSize(size: string): Builder - Sets pizza size
 * - setCrust(crust: string): Builder - Sets crust type
 * - addCheese(): Builder - Adds cheese
 * - addTopping(name: string): Builder - Adds a topping
 * - addExtra(name: string): Builder - Adds an extra item
 * - build(): Product - Validates and returns the final product
 */
```

---

### Concrete Builder #1: Pizza Builder

```javascript
class PizzaBuilder {
  constructor() { 
    this.reset(); 
  }

  // Reset for a new pizza
  reset() { 
    this.pizza = new Pizza(); 
    return this; 
  }
  
  // Required parameters
  setSize(size) { 
    this.pizza.size = size; 
    return this; 
  }
  
  setCrust(crust) { 
    this.pizza.crust = crust; 
    return this; 
  }
  
  // Optional parameters
  addCheese() { 
    this.pizza.cheese = true; 
    return this; 
  }
  
  addTopping(name) { 
    this.pizza.toppings.push(name); 
    return this; 
  }
  
  addExtra(name) { 
    this.pizza.extras.push(name); 
    return this; 
  }

  // Final validation and product creation
  build() {
    // Validate required fields
    if (!this.pizza.size || !this.pizza.crust) {
      throw new Error("Pizza requires both size and crust type");
    }
    
    const result = this.pizza;
    this.reset(); // Prepare for next build
    return result;
  }
}
```

---

### Concrete Builder #2: Receipt Builder

```javascript
class PizzaReceiptBuilder {
  constructor() { 
    this.reset(); 
  }

  reset() {
    this.lines = ["===== PIZZA ORDER RECEIPT ====="];
    this.state = { 
      size: null, 
      crust: null, 
      cheese: false, 
      toppings: [], 
      extras: [] 
    };
    return this;
  }
  
  // Same interface as PizzaBuilder
  setSize(size) { 
    this.state.size = size; 
    return this; 
  }
  
  setCrust(crust) { 
    this.state.crust = crust; 
    return this; 
  }
  
  addCheese() { 
    this.state.cheese = true; 
    return this; 
  }
  
  addTopping(name) { 
    this.state.toppings.push(name); 
    return this; 
  }
  
  addExtra(name) { 
    this.state.extras.push(name); 
    return this; 
  }

  build() {
    const s = this.state;
    
    // Same validation as PizzaBuilder
    if (!s.size || !s.crust) {
      throw new Error("Pizza requires both size and crust type");
    }
    
    // Format the receipt
    this.lines.push(`Size: ${s.size}`);
    this.lines.push(`Crust: ${s.crust}`);
    this.lines.push(`Cheese: ${s.cheese ? "Yes" : "No"}`);
    
    if (s.toppings.length > 0) {
      this.lines.push(`Toppings:`);
      s.toppings.forEach(topping => this.lines.push(`  - ${topping}`));
    } else {
      this.lines.push(`Toppings: None`);
    }
    
    if (s.extras.length > 0) {
      this.lines.push(`Extras:`);
      s.extras.forEach(extra => this.lines.push(`  - ${extra}`));
    } else {
      this.lines.push(`Extras: None`);
    }
    
    this.lines.push("==============================");
    
    const result = this.lines.join("\n");
    this.reset();
    return result;
  }
}
```

---

## 7) The Director: Standard Recipes

The Director encapsulates common construction sequences:

---

```javascript
class PizzaDirector {
  constructor() {
    // Could inject dependencies like pricing service, etc.
  }

  buildMargherita(builder) {
    return builder
      .reset()
      .setSize("Medium")
      .setCrust("Thin")
      .addCheese()
      .addTopping("Fresh Tomatoes")
      .addTopping("Fresh Basil")
      .addTopping("Mozzarella")
      .build();
  }

  buildMeatLovers(builder) {
    return builder
      .reset()
      .setSize("Large")
      .setCrust("Deep Dish")
      .addCheese()
      .addTopping("Pepperoni")
      .addTopping("Sausage")
      .addTopping("Bacon")
      .addTopping("Ham")
      .build();
  }
  
  buildVegetarian(builder) {
    return builder
      .reset()
      .setSize("Medium")
      .setCrust("Whole Wheat")
      .addCheese()
      .addTopping("Bell Peppers")
      .addTopping("Mushrooms")
      .addTopping("Onions")
      .addTopping("Olives")
      .build();
  }
}
```

---

## 8) Using the Pattern

### With Director (Standard Recipes)

```javascript
const director = new PizzaDirector();
const pizzaBuilder = new PizzaBuilder();
const receiptBuilder = new PizzaReceiptBuilder();

// Create a Margherita pizza object
const margherita = director.buildMargherita(pizzaBuilder);
console.log(margherita.describe());
// Output: "Medium Thin crust pizza with cheese topped with Fresh Tomatoes, Fresh Basil, Mozzarella"

// Create a receipt for a Meat Lovers pizza
const meatLoversReceipt = director.buildMeatLovers(receiptBuilder);
console.log(meatLoversReceipt);
// Output: Formatted receipt with all the Meat Lovers details
```
---

### Custom Building (Without Director)

```javascript
const pizzaBuilder = new PizzaBuilder();

// Create a custom pizza
const customPizza = pizzaBuilder
  .reset()
  .setSize("Small")
  .setCrust("Gluten-Free")
  .addCheese()
  .addTopping("Pineapple")
  .addTopping("Ham")
  .addExtra("Garlic Dip")
  .build();

console.log(customPizza.describe());
// Output: "Small Gluten-Free crust pizza with cheese topped with Pineapple, Ham"
```

---

## 9) Real-World Applications

### API Request Builders

```javascript
const request = new RequestBuilder()
  .setMethod('POST')
  .setUrl('https://api.example.com/users')
  .setHeader('Content-Type', 'application/json')
  .setHeader('Authorization', 'Bearer token123')
  .setBody({ name: 'John', email: 'john@example.com' })
  .setTimeout(5000)
  .setRetries(3)
  .build();
```

---

### UI Component Configuration

```javascript
const chart = new ChartBuilder()
  .setType('bar')
  .setTitle('Monthly Sales')
  .addDataset('Q1 Sales', [10, 20, 30])
  .addDataset('Q2 Sales', [15, 25, 35])
  .setAxes({ x: 'Months', y: 'Revenue ($)' })
  .setLegend(true)
  .setAnimations(true)
  .build();
```

---

### Test Data Creation

```javascript
const testUser = new TestUserBuilder()
  .withName('Test User')
  .withEmail('test@example.com')
  .withPermissions(['read', 'write'])
  .withVerifiedEmail()
  .build();
```

---

## 10) Builder vs Alternatives

### Options Object

```javascript
const pizza = createPizza({
  size: "Large",
  crust: "Thin",
  cheese: true,
  toppings: ["Pepperoni", "Mushrooms"]
});
```

**Pros:**
- Simple and familiar in JavaScript . All options visible at once.

**Cons:**
- No step-by-step guidance. No enforcement of required parameters. Harder to create multiple representations

---

### Factory Method

```javascript
const margherita = PizzaFactory.createMargherita();
const meatLovers = PizzaFactory.createMeatLovers();
```

**Pros:**
- Simple interface for common objects
- Hides creation details

**Cons:**
- Less flexible for customization
- Typically creates only one type of representation

---

The **Factory** pattern can almost be seen as a simplified version of the **Builder** pattern.

In the **Factory** pattern, the factory is responsible for creating various subtypes of an object depending on the needs.

The user of a factory method doesn't need to know the exact subtype of that object.  
For example, a factory method `createCar()` might return a **Ford** or a **Honda** typed object.

In the **Builder** pattern, different subtypes are also created by a builder method,  
but the **composition of the objects** might differ within the same subclass.

Continuing the car example:  
You might have a `createCar()` builder method which creates:

- A **Honda**-typed object with a **4-cylinder engine**, or  
- A **Honda**-typed object with **6 cylinders**.

The builder pattern allows for this **finer granularity**.

---


### When to Choose Builder

Choose Builder when:
- The object creation process has multiple steps
- You need different representations from the same construction process
- You want to enforce a specific construction sequence
- The object has many optional parameters

---

## 11) Best Practices

1. **Return `this` from each step** to enable method chaining
2. **Validate in the `build()` method** to ensure a complete object
3. **Reset the builder after `build()`** to prevent accidental reuse of partial state
4. **Make required parameters clear** in documentation or method signatures
5. **Consider using the Director** when you have standard configurations
6. **Don't expose the product during construction** to maintain encapsulation

---

## 12) Decision Checklist

Consider using Builder if you answer "yes" to at least two:

- Does the object have 5+ parameters, especially optional ones?
- Do you need to create the same object in multiple places?
- Would you benefit from having standard "recipes" for object creation?
- Do you need multiple representations (e.g., object, JSON, UI) from the same data?
- Is the construction process complex with multiple steps?

---

## 13) Summary

The Builder pattern:

- **Simplifies** complex object creation with a step-by-step approach
- **Separates** construction logic from the final product
- **Enables** multiple representations from the same construction process
- **Improves** code readability and maintainability
- **Reduces** errors in object creation through validation and encapsulation

---

## 14) Further Resources

- [Refactoring Guru: Builder Pattern](https://refactoring.guru/design-patterns/builder)
- [Design Patterns: Elements of Reusable Object-Oriented Software](https://en.wikipedia.org/wiki/Design_Patterns) (Gang of Four)
---