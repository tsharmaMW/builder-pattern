---
marp: true
theme: default
paginate: true
---

# 🏗 Builder Design Pattern

---

## 1. Why are we here?

- We’re talking about **Builder Design Pattern** today.
- My goal:  
  - Explain what it is.  
  - Show why/when to use it.  
  - Walk through a real-world analogy.  
  - Show working JS code (Pizza example 🍕).  
  - Wrap up with pros, cons, and when *not* to use it.

---

## 2. The Problem

- Sometimes object creation is **messy**.
- Example: You’re creating a complex object with **lots of optional parts**.
- Without Builder, you either:
  - Have **constructors with 10+ parameters** (ugh 😵‍💫)
  - Or end up with **hard-to-read object setup code**.

---

## 3. What is the Builder Pattern?

- A **creational design pattern**.
- Lets you build complex objects step-by-step.
- You can:
  - Create different *representations* of the same object.
  - Avoid huge constructors.
  - Keep object construction logic **in one place**.

---

## 4. Real-World Analogy 🍕

- Imagine ordering a pizza:
  - You don’t tell the chef every detail in one breath.
  - Instead:  
    1. Choose base  
    2. Choose sauce  
    3. Add toppings  
    4. Bake
- You build it **step by step**.
- Same idea in Builder pattern.

---

## 5. Structure (Refactoring Guru style)

Roles:
- **Builder** – Interface defining building steps.
- **ConcreteBuilder** – Implements steps to build the product.
- **Director** – Controls order of building steps (optional).
- **Product** – The final object.

---

## 6. Pizza Example in JS

```javascript
class Pizza {
  constructor() {
    this.size = null;
    this.cheese = false;
    this.pepperoni = false;
    this.bacon = false;
  }
}

class PizzaBuilder {
  constructor() {
    this.pizza = new Pizza();
  }

  setSize(size) {
    this.pizza.size = size;
    return this;
  }

  addCheese() {
    this.pizza.cheese = true;
    return this;
  }

  addPepperoni() {
    this.pizza.pepperoni = true;
    return this;
  }

  addBacon() {
    this.pizza.bacon = true;
    return this;
  }

  build() {
    return this.pizza;
  }
}

// Usage
const myPizza = new PizzaBuilder()
  .setSize("Large")
  .addCheese()
  .addBacon()
  .build();

console.log(myPizza);
```

---

## 7. How this helps

- **Readable**:
  ```javascript
  new PizzaBuilder().setSize("Large").addBacon().build();
  ```
- **Flexible**:
  - Add/remove steps easily.
  - Different builders for different pizza styles.
- **Safe**:
  - No need to pass undefined/null params to constructors.

---

## 8. Pros ✅

- Avoids telescoping constructors.
- Step-by-step object creation.
- Single place for construction logic.
- Can produce different variations of an object.

---

## 9. Cons ❌

- More classes = more code.
- Not always needed for simple objects.
- Slightly more complex structure.

---

## 10. When to Use

- When object construction is complex.
- When you need different representations of the same object.
- When you want immutable objects but still need step-by-step setup.

---

## 11. Quick Recap

- **Problem**: Complex object creation → messy code.
- **Solution**: Builder pattern.
- **Analogy**: Pizza making.
- **Code**: Step-by-step builder with chaining.
- **When to use**: Complex, configurable object creation.

---

## 12. Questions? 🙋
Let’s discuss real-world cases in our codebase where Builder might fit.