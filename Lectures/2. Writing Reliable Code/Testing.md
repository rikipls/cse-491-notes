[Video link](https://www..youtube.com/watch?v=SAM4rWaIvUQ)
Only watched 1:15 - 21:25

### Testing levels

#### Unit tests
- Test a single unit (e.g. a function or class)
- Isolated as much as possible
#### Component Tests
- Test a single component in isolation
#### Integration Tests
-  Test several units together
#### System Tests
- Test the whole system
- Might involve external components (DB, UI, etc.)

### Unit tests 
- Testing *individual units* of behavior
- An *individual unit* of behavior is the **smallest** possible unit of behavior that can be individually tested in isolation
- You can write unit tests before, while, or after you write your code

![[Pasted image 20230920103209.png]]

**Motivations for Unit Testing**
- Test the code as early as possible
	- Easier to fix
	- Easier to analyze: less complexities, earlier in the process
- Automatic regression
	- Others can touch the code with less fear
	- Allows refactoring, even for old production code
- Encourages clear API, better decoupling and organized development
	- API must be clear when test is written
	- Flows and scenarios must be defined

#### TDD
**Test Driven Development**
- Write the tests **first**
- Review the tests with the team/product/system engineer
- Make the tests compile, but *fail*
- Work on the feature implementation so the tests start to **pass**

**Pros**
- Better understanding of the feature and the API
- Actual control over the implementation progress
- Makes sure we test! - We don't push it to the end when there's no time left...

**Cons**
You may end up at the deadline with 100% tests and only 40% features

#### Good Unit Tests are
**Maintainable**
- If there is a need to change the test, it shouldn't be too difficult

**Readable**
- Tests themselves should be easy to understand and intuitive

**Trustworthy**
- When they pass, it actually shows that the feature works as intended
- When they fail, it highlights a **real** problem with the feature
	- It should never be **okay** for a test to fail, otherwise it's a bad test

Test according to the **spec**, not the actual implementation
Test **public behavior**