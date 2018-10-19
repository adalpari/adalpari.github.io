---
title: "Android Testing - How to test"
header:
  teaser: /assets/images/basic-testing.jpg
  overlay_image: /assets/images/basic-testing.jpg
---

In this series of posts we are going to talk about Android Testing. It's a big field to talk about, but here there are some important points and tips.

1. [Introduction.](../android-testing-introduction/)
2. [How to test.](../android-testing-how-to-test/)

## Test structure

Now that we know how to create a test, let's start with simple tests inside _test_ directory. This is how a typical test looks:

```java
public class MyTest {

    @Before
    public void setUp() {
        // Init variables and prepare whatever I need
    }

    @After
    public void tearDown() {
        // Reset state to be sure about keep everything like it was before the test
    }

    @Test
    public void shouldDoXWhenY() {
        // Prepare and run the test
    }

    @Test
    public void shouldReturnXOnY() {
        // Prepare and run the test
    }
}
```

- _@Before setUp()_: this method is automatically __run before every test__. So, you can add here every common initialisation code you need: initialise variables, create objects and so on.
- _@After tearDown()_: this is the opposite to the previous one. It's automatically __run after every test__. So, you can add here every common code related to finish test cycle: reset states, remove items and so on.
- _@Test testName_: this is the test itself. You can create as much tests as you need. So, try to create one test per each atomic tasks. This way we will keep all tests as simple as possible. Try yo use a name which says what the test does or what it tests. Don't be shy about long names, it's a good way to quickly understand the test. There are multiple test names conventions. Personally I use something similar to:

`should[do/return/throw]Whatever[When/On]SomethingHappens()`.

_E.g. `shouldReturnTrueWhenIntroduceRightUserLogin();`. We will see how a test is structured on the next point.

If I run the tests, this is the output I should get if everything is OK. All green.

![tests outout green](/assets/images/example-test-passing.png)

However, if some test fails, I will see the red state of the failing test, what's happening (_It expected to get 1, but it got 2_) and the stack-trace if there was any crash. _(Notice that here the error is simulated, so don't pay attention to the actual reason)_

![tests output red](/assets/images/example-test-failing.png)

## Given-When-Then

Well, let's talk about how every single test is organised inside it. In order to keep a generic and organised structure, and again according to [_Robert C. Martin_](../Clean-Code/), we can use a three block structure. This will help us, in addition to descriptive title, to easily understand all tests from a project. Because all tests will work in the same way.

```java
@Test
public void shouldDoXWhenY() {
    // Given

    // When

    // Then
}
```

- __Given__: we will set the test conditions here and create necessary objects. Do not confuse this with _setUpMethod()_. This block will affect only to the current test, while the _setUp_ method will affect to every test.
- __When__: this block is used to run the actions to test. This is basically, call the methods we want to test and get the results from them.
- __Then__: finally, we will use this block to check that the results from method executions are correct. In other words: the __asserts__. A useful tip is having only one assert per test. This way we will keep tests simple, and will get direct test information about each atomic tests. If we mix functionalities while testing, we can misunderstand results and some actions can modify the behaviour of other ones. Remember the _I_ from _FIRST_ principles: _Independent_.

## Dependency injection in tests

This and the following one (mocks) are some of the very basic topics in testing. It turned out that lot of coding strategies we do with our code base, are highly related with testing. It's very important make our code testable rather than just test the code. Let's see an example of how important is dependency injection.

```java
// Don't do this
public boolean savePhoneNumber(String number) {
    boolean phoneCreated = false;
    
    if (isValidNumber(number)) {
        PhoneNumberDao phoneNumberDao = new PhoneNumberDao();
        phoneCreated = phoneNumberDao.create(number);
    }
    
    return phoneCreated;
}

private boolean isValidNumber(String number) {
    return number != null && number.length() == 9;
}
```

These methods are quite easy. Let's say we can use `savePhoneNumber(number)` to save a phone number into database with a _DAO_.
Then, we can test the method `isValidNumber(number)` with different parameters (we will se how in the next post). But, what happens with `savePhoneNumber(number)`, can we test it?. Well, te answer is: yes we can. But the right answer is: no, we shouldn't. 

We should not test it, because it's creating a new object inside it, which we can not control in the test. What happens is the test fails? is it failing in the _saver_ method? or is it failing in the _whole PhoneNumberDao_?
Rememeber this phrase: __a key for successful testing is having a completely controlled test environment__.

So, the correct way to write these methods in order to make them testable is the following one.

```java
public boolean savePhoneNumber(String number, PhoneNumberDao phoneNumberDao) {
    boolean phoneCreated = false;
    
    if (isValidNumber(number)) {
        phoneCreated = phoneNumberDao.create(number);
    }
    
    return phoneCreated;
}

private boolean isValidNumber(String number) {
    return number != null && number.length() == 9;
}
```

We are injecting the external class (_dao_) using the arguments in order to have the control over it. We could also inject it using the class constructor. If you have any doubt with dependency injection please refer to [Dependency Inversion principle from SOLID](../SOLID-D/).

You may ask, why I should do this? how is it helping me to control the external object? Let's talk about the following point: _mocks and test doubles_.

##  Mocks and Test doubles

Now that we have our code organised pretty much, let's answer how to "control" the external _dao_. Remember that we want to test the number saver method apart from the _dao_. We can, of course test the _PhoneNumberDao_ in a separated test.

In order to get rid and control the external injection of the _PhoneNumberDao_, we are going to create a _fake_ object. Or, in other words, create a _mock_ or _test double_.

It's really easy. We can create a new object extending from _PhoneNumberDao_, mocking all it's content. Or we can use [Mockito](https://site.mockito.org) to do it. Mockito is a testing framework to create _fake_ or mocked object and interact with them. 

The definition for a basic mocked object, is a copy of the real object but not doing any operation; an empty object. In the case of the _MockedDao_ it won't save anything if we use `create(..)` method, and we won't know the returned result until we set it manually. So, if we use the mocked object in the test, calling `savePhoneNumber(number, phoneNumberDaoMocked)`, we have to set also the behaviour of the mocked object. 

As we mentioned before, we can create the mock manually, and use it for the test. We will know the return of the _create()_ method, so we can create a simple test. 

Just see how the test creates a mocked object, a correct number, try to save it and expect to be true. Because our mocked object is going to return true, the test is passing. 

Remember that we are testing _savePhoneNumber()_ method, so it doesn't matter what the _dao_ is doing, we just want to be sure the saver method does what it's supposed to do. Notice that we can test the validation method as well, but we will see the complete example in the next post.

```java
public class MyTest {
    @Test
    public void shouldReturnTrueWhenSaveValidNumber() {
        // Given
        PhoneNumberDaoMocked phoneNumberDaoMocked = new PhoneNumberDaoMocked();
        String correctPhoneNumber = "726384982";
        
        // When
        boolean saved = savePhoneNumber(correctPhoneNumber, phoneNumberDaoMocked);
        
        // Then
        assertTrue(saved);
    }

    class PhoneNumberDaoMocked extends PhoneNumberDao {
        @Override
        public boolean create(String number) {
            return true;
        }
    }
}
```

On the other hand, the same example but using _Mockito_ is a bit different. We need to create the mocked object first with the _Mockito_ method, and set its behaviour after creation. In this case, we specify that when the method `create(..)` is called with any string, it will return _true_. Note: we can create the object with this syntax or use the `@Mock` annotation if we set it as a class variable.

```java
@RunWith(MockitoJUnitRunner.class)
public class MyTest {
    @Test
    public void shouldReturnTrueWhenSaveValidNumber() {
        // Given
        String correctPhoneNumber = "726384982";
        PhoneNumberDao phoneNumberDaoMocked = mock(PhoneNumberDao.class);  // create the mock
        when(phoneNumberDaoMocked.create(anyString())).thenReturn(true);   // set mock behaviour

        // When
        boolean saved = savePhoneNumber(correctPhoneNumber, phoneNumberDaoMocked);

        // Then
        assertTrue(saved);
    }

    class PhoneNumberDaoMocked extends PhoneNumberDao {
        @Override
        public boolean create(String number) {
            return true;
        }
    }
}
```

Notice the `@RunWith(MockitoJUnitRunner.class)` specifying to use the mockito runner.
