[Back](./README.md)

# 🎭  What is Moq?

👉 Moq is a library used to create **fake dependencies**

---

### Why do we need it?

Because we don’t want:

* Real database ❌
* Real API ❌
* Real file system ❌

---

### Example

```csharp
var mockRepo = new Mock<IUserRepository>();

mockRepo.Setup(r => r.GetById(1))
        .Returns(new User { Name = "Ali" });
```

👉 You control the behavior without real DB.

---

# 🧩 5. Important Terms (Simple)

### Mock

👉 Fake object + verify behavior

```csharp
mock.Verify(x => x.Save(), Times.Once);
```

---

### Stub

👉 Just returns data

```csharp
mock.Setup(x => x.Get()).Returns(data);
```

---

### Fake

👉 Simple working implementation (like in-memory DB)

---

### ❌ You DON’T need testing when:

* Code is trivial:
* No logic
* Prototype / throwaway code

---

# ⚠️ 11. Common Beginner Mistakes

### ❌ Testing everything

👉 Not needed

---

### ❌ Testing framework behavior

```csharp
_dbContext.SaveChanges();
```

👉 EF already tested this

---

### ❌ Not isolating dependencies

👉 Leads to slow tests

---

# 🧠 12. Simple Mental Model

Think like this:

> Unit test = “Does my logic work alone?”

> Integration test = “Do all parts work together?”

---

# 🚀 Final Advice

If you remember only this:

1. **Write testable code (use interfaces + DI)**
2. **Mock external dependencies**
3. **Test behavior, not implementation**
4. **Keep tests small and fast**


---

# 🧼 Best Practices

### 1. Follow AAA Pattern

```csharp
// Arrange
// Act
// Assert
```

---

### 2. One Behavior per Test

Bad:

```csharp
TestEverything()
```

Good:

```csharp
ShouldReturnFalse_WhenOutOfStock()
```

---

### 3. Use Meaningful Names

```csharp
MethodName_State_ExpectedResult
```

---

### 4. Keep Tests Fast & Isolated

* No real DB
* No real API
* No file system

---

### 5. Don’t Over-Mock

👉 Mock only *external dependencies*, not everything.

---

### 6. Test Behavior, Not Implementation

Bad:

```csharp
Verify method X was called 3 times internally
```

Good:

```csharp
Verify correct result/output
```

---

# 🧭 Big Picture

Think of unit testing like this:

* You are testing **your logic in isolation**
* Everything else = **fake/mock**
* Focus on **what the code should do**, not how it does it

---

# 🚨 2. Most Common Mistakes

| Mistake                         | Why It’s Bad                  | Better Approach                 |
| ------------------------------- | ----------------------------- | ------------------------------- |
| Testing everything              | Wastes time                   | Test logic, not trivial code    |
| Over-mocking                    | Tests become fake/unrealistic | Mock only external dependencies |
| Testing implementation details  | Breaks when refactoring       | Test behavior/output            |
| Using real DB/API in unit tests | Slow & flaky                  | Use mocks/fakes                 |
| Poor test names                 | Hard to understand            | Use descriptive names           |
| One test = many things          | Hard to debug                 | One behavior per test           |
| Ignoring edge cases             | Bugs in production            | Test boundaries & failures      |
| Not running tests often         | Late bug detection            | Run on every build              |

---

# 🧠 3. Rules of Thumb (Very Important)

| Rule                                | Meaning                         |
| ----------------------------------- | ------------------------------- |
| “If it’s logic → test it”           | Business rules must be tested   |
| “If it’s framework → don’t test it” | EF, ASP.NET already tested      |
| “One reason to fail per test”       | Keep tests focused              |
| “Tests should be boring”            | Simple, predictable             |
| “Fast tests = good tests”           | Slow tests won’t be used        |
| “If it’s hard to test → bad design” | Testing reveals design problems |
| “Mock boundaries, not everything”   | Only external systems           |

---


# 🧪 6. Good vs Bad Tests

| Bad ❌                 | Good ✅                                  |
| --------------------- | --------------------------------------- |
| `Test1()`             | `Register_ShouldFail_WhenNameIsEmpty()` |
| Tests multiple things | Tests one behavior                      |
| Uses real DB          | Uses mock                               |
| Hard to read          | Clear Arrange-Act-Assert                |
| Breaks on refactor    | Stable                                  |

---

# 🏗️ 7. Test Design Tips

| Tip                    | Why              |
| ---------------------- | ---------------- |
| Use AAA pattern        | Clear structure  |
| Keep tests independent | No side effects  |
| Use builders/factories | Clean setup      |
| Avoid duplication      | Maintainability  |
| Use meaningful data    | Easier debugging |

---

# 🧠 8. How to Know You’re Doing It Right

You’re on the right track if:

* ✅ Tests run in seconds
* ✅ You can refactor safely
* ✅ Tests are easy to read
* ✅ Failures clearly explain the problem

---

# ⚠️ 9. Red Flags (Very Important)

| Red Flag 🚩                | Meaning                |
| -------------------------- | ---------------------- |
| Test takes seconds/minutes | Probably not unit test |
| Needs DB setup             | Integration test       |
| Random failures            | Bad isolation          |
| Too many mocks             | Over-engineering       |
| Hard to write test         | Bad design             |

---

# 🧭 10. Practical Developer Mindset

When writing code, always ask:

* “How will I test this?”
* “Can I mock dependencies?”
* “Is this doing too many things?”

---

# Interview Questions


# 🧠 1. Fundamentals

### ❓ What is unit testing?

✅ **Answer:**
Unit testing is the practice of testing small, isolated pieces of code (usually methods) to ensure they behave correctly. External dependencies like databases or APIs are replaced with mocks or fakes so the test focuses only on the logic.

---

### ❓ What makes a good unit test?

✅ **Answer:**
A good unit test is:

* Fast
* Independent
* Repeatable
* Easy to read
* Tests one behavior

It should follow the **Arrange–Act–Assert** pattern and verify behavior, not implementation.

---

# ⚖️ 2. Unit vs Integration

### ❓ What is the difference between unit testing and integration testing?

✅ **Answer:**

* Unit testing focuses on a single component in isolation using mocks.
* Integration testing verifies how multiple components work together, often using real dependencies like databases.

Unit tests are fast and isolated; integration tests are slower but more realistic.

---

### ❓ When would you use integration tests instead of unit tests?

✅ **Answer:**
When you want to verify:

* Database interactions
* API endpoints
* Configuration issues
* End-to-end flows between components

---

# 🎭 3. Mocking & Moq

### ❓ What is mocking?

✅ **Answer:**
Mocking is creating fake implementations of dependencies so you can control their behavior and isolate the unit under test.

---

### ❓ Why do we use Moq?

✅ **Answer:**
Moq allows us to:

* Simulate dependencies
* Control return values
* Verify interactions (e.g., method calls)

This helps isolate business logic from external systems.

---

### ❓ What is the difference between mock and stub?

✅ **Answer:**

* Stub: Provides predefined data
* Mock: Verifies interactions (e.g., method calls)

---

# 🧩 4. Design for Testability

### ❓ What makes code testable?

✅ **Answer:**

* Uses dependency injection
* Depends on interfaces, not concrete classes
* Has clear inputs and outputs
* Avoids static/global state

---

### ❓ How would you improve this code?

```csharp
public void Save()
{
    var db = new Database();
    db.Insert();
}
```

✅ **Answer:**
Introduce dependency injection:

```csharp
public class Service
{
    private readonly IDatabase _db;

    public Service(IDatabase db)
    {
        _db = db;
    }

    public void Save()
    {
        _db.Insert();
    }
}
```

Now it’s testable using mocks.

---

# 🧪 5. Practical Scenarios

### ❓ How do you test a method that writes to a file?

✅ **Answer:**
Abstract the file system behind an interface (e.g., `IFileSystem`) and mock it. Do not use real file operations in unit tests.

---

### ❓ How do you test database logic?

✅ **Answer:**

* Unit test: mock repository
* Integration test: use in-memory DB or test DB

---

### ❓ How do you test external APIs?

✅ **Answer:**
Wrap the API in an interface and mock it. Never call real APIs in unit tests.

---

# ⚠️ 6. Edge Cases & Thinking

### ❓ What should NOT be unit tested?

✅ **Answer:**

* Simple getters/setters
* Framework code (EF, ASP.NET)
* Third-party libraries
* Trivial code with no logic

---

### ❓ What is over-mocking?

✅ **Answer:**
Mocking too many internal details, making tests fragile and unrealistic. Only external dependencies should be mocked.

---

# 🧠 7. Advanced Thinking

### ❓ How do you test exception handling?

✅ **Answer:**

```csharp
Assert.Throws<ArgumentException>(() => service.DoSomething(null));
```

---

### ❓ How do you test async methods?

✅ **Answer:**

```csharp
var result = await service.GetDataAsync();
Assert.NotNull(result);
```

---

### ❓ How do you ensure tests are maintainable?

✅ **Answer:**

* Use clear naming
* Avoid duplication
* Use helper methods/builders
* Keep tests small and focused

---

# 🏗️ 8. Project Structure

### ❓ How should a test project be structured?

✅ **Answer:**

```
MyApp/
├── Services/
├── Controllers/
├── Tests/
    ├── Services/
    ├── Controllers/
```

Tests should mirror the main project structure.

---

# 🔥 9. Real Interview Challenge

### ❓ Write a test for this:

```csharp
public bool IsAdult(int age)
{
    return age >= 18;
}
```

✅ **Answer:**

```csharp
[Theory]
[InlineData(17, false)]
[InlineData(18, true)]
[InlineData(25, true)]
public void IsAdult_ShouldReturnCorrectResult(int age, bool expected)
{
    var result = service.IsAdult(age);
    Assert.Equal(expected, result);
}
```

👉 Shows parameterized testing (good signal in interviews)

---

# 🎯 10. Trick Questions

### ❓ Should you aim for 100% test coverage?

✅ **Answer:**
No. Focus on **critical logic**, not coverage numbers. High coverage with poor tests is useless.

---

### ❓ If a test is hard to write, what does it mean?

✅ **Answer:**
Usually the code is poorly designed (too many responsibilities or tight coupling).

---

# 🧠 Final Interview Tip

If you get stuck, say this:

> “I would isolate dependencies using interfaces and mock them to focus on the business logic.”

👉 This sentence alone scores points.

---
[Back](./README.md)
