---
title: "Android Testing - Unit tests"
header:
  teaser: /assets/images/unit-testing.jpg
  overlay_image: /assets/images/unit-testing.jpg
---

In this series of posts we are going to talk about Android Testing. It's a big field to talk about, but here there are some important points and tips.

1. [Introduction.](../android-testing-introduction/)
2. [How to test.](../android-testing-how-to-test/)
3. [Unit tests.](../android-testing-unit-tests/)
4. [Integration tests.](../android-testing-integration-tests/)

We are going to do some examples to easily understand how typical unit tests are done in Android. So, in addition to read this post I encourage you to try the examples by yourself.

## Basic Unit Test example

The following class can be used to save a number in a persistent storage. It has a public method, which receive a number. If the number is valid, it's saved on the "database" using a _DAO_. The _dao_ can throw an exception, so it's thrown in the saver method as well. Note: we don't need a real `PhoneNumberDao` class, because we are testing the saver helper and not the _dao_. So, the database saving is just a supposition.

```java
public class NumberSaverHelper {

    private PhoneNumberDao phoneNumberDao;

    public NumberSaverHelper(PhoneNumberDao phoneNumberDao) {
        this.phoneNumberDao = phoneNumberDao;
    }

    public boolean savePhoneNumber(int number) throws PhoneNumberDao.PhoneNumberDaoException {
        boolean phoneCreated = false;

        if (isValidNumber(number)) {
            phoneCreated = phoneNumberDao.create(number);
        }

        return phoneCreated;
    }

    private boolean isValidNumber(int number) {
        if (number < 0) {
            return false;
        }
        
        return String.valueOf(number).length() == 9;
    }
}
```

```java
public class PhoneNumberDao {

    public boolean create(int number) throws PhoneNumberDaoException {
        try {
            // simulate saving the number on the database
            Thread.sleep(200);
        } catch (InterruptedException e) {
            throw new PhoneNumberDaoException();
        }
        return true;
    }

    class PhoneNumberDaoException extends Exception {
    }
}
```

I invite you to try the following points by yourself before check the answers.

Test the use of `savePhoneNumber(int)`:
1. Negative number returns false.
2. Number with less than 9 digits returns false.
3. Number with more than 9 digits returns false.
4. Number with 9 digits returns true.
5. Number with 9 digits returns false if the _dao_ can not create the number.
6. `PhoneNumberDaoException` is properly thrown if the _dao_ throws it.

...

Done? Let's see the solution!

```java
public class NumberSaverHelperTest {

    private final static int NEGATIVE_NUMBER = -123456789;
    private final static int SHORT_NUMBER = 12345678;
    private final static int LONG_NUMBER = 1234567890;
    private final static int CORRECT_NUMBER = 123456789;

    @Test
    public void shouldReturnFalseForNegativeNumber() throws PhoneNumberDao.PhoneNumberDaoException {
        NumberSaverHelper numberSaverHelper = givenNumberSaverHelperWithSucessDao();

        boolean result = numberSaverHelper.savePhoneNumber(NEGATIVE_NUMBER);

        Assert.assertFalse(result);
    }

    @Test
    public void shouldReturnFalseForShortNumbers() throws PhoneNumberDao.PhoneNumberDaoException {
        NumberSaverHelper numberSaverHelper = givenNumberSaverHelperWithSucessDao();

        boolean result = numberSaverHelper.savePhoneNumber(SHORT_NUMBER);

        Assert.assertFalse(result);
    }

    @Test
    public void shouldReturnFalseForLongNumbers() throws PhoneNumberDao.PhoneNumberDaoException {
        NumberSaverHelper numberSaverHelper = givenNumberSaverHelperWithSucessDao();

        boolean result = numberSaverHelper.savePhoneNumber(LONG_NUMBER);

        Assert.assertFalse(result);
    }

    @Test
    public void shouldReturnTrueForCorrect() throws PhoneNumberDao.PhoneNumberDaoException {
        NumberSaverHelper numberSaverHelper = givenNumberSaverHelperWithSucessDao();

        boolean result = numberSaverHelper.savePhoneNumber(CORRECT_NUMBER);

        Assert.assertTrue(result);
    }

    @Test
    public void shouldReturnFalseWhenDaoReturnFalse() throws PhoneNumberDao.PhoneNumberDaoException {
        NumberSaverHelper numberSaverHelper = givenNumberSaverHelperWithErrorDao();

        boolean result = numberSaverHelper.savePhoneNumber(CORRECT_NUMBER);

        Assert.assertFalse(result);
    }

    @Test(expected = PhoneNumberDao.PhoneNumberDaoException.class)
    public void shouldThrowExceptionFromDao() throws PhoneNumberDao.PhoneNumberDaoException {
        NumberSaverHelper numberSaverHelper = givenNumberSaverHelperWithExceptionInDao();

        boolean result = numberSaverHelper.savePhoneNumber(CORRECT_NUMBER);

        Assert.assertTrue(result);
    }

    private NumberSaverHelper givenNumberSaverHelperWithSucessDao() {
        PhoneNumberDaoMocked phoneNumberDao = new PhoneNumberDaoMocked(true);
        NumberSaverHelper numberSaverHelper = new NumberSaverHelper(phoneNumberDao);
        return numberSaverHelper;
    }

    private NumberSaverHelper givenNumberSaverHelperWithErrorDao() {
        PhoneNumberDaoMocked phoneNumberDao = new PhoneNumberDaoMocked(false);
        NumberSaverHelper numberSaverHelper = new NumberSaverHelper(phoneNumberDao);
        return numberSaverHelper;
    }

    private NumberSaverHelper givenNumberSaverHelperWithExceptionInDao() {
        PhoneNumberDaoMocked phoneNumberDao = new PhoneNumberDaoMocked(true);
        phoneNumberDao.setForceException(true);
        NumberSaverHelper numberSaverHelper = new NumberSaverHelper(phoneNumberDao);
        return numberSaverHelper;
    }

    class PhoneNumberDaoMocked extends PhoneNumberDao {

        private boolean mockedReturn;
        private boolean forceException = false;

        public PhoneNumberDaoMocked(boolean mockedReturn) {
            this.mockedReturn = mockedReturn;
        }

        @Override
        public boolean create(int number) throws PhoneNumberDaoException {
            if (forceException) {
                throw new PhoneNumberDao.PhoneNumberDaoException();
            }

            return mockedReturn;
        }

        public void setForceException(boolean forceException) {
            this.forceException = forceException;
        }
    }
}
```

Before the explanation, remember that you can (should) organise the test like they were a regular class. You can re-use methods, refactor the code and so on.

Starting from the end of the test class, we have created a mocked `PhoneNumberDao`. The mock can be instantiated with the expected return from the _create_ method. In the same way we can force it to throw `PhoneNumberDaoException` when use the _create_ method instead of return a value. We do this with an additional setter.

If you keep looking some lines above, you can see three methods to create the `NumberSaverHelper`, depending on if we want to create a successful one, a failing one or even the one which raise the exception. We can do this on many other ways, but in this example we have extracted the creation into these simple methods.

And last but not least, let's see the tests. As you can see, all of them have the _given-when-then_ structure.
- Test with non-valid numbers: they instantiates a success _NumberSaverHelper_ and pass to _create_ method a non-valid number, expecting the method to return false.
- Test with valid number: it instantiates a success _NumberSaverHelper_ and pass a valid number. Everything should be ok, so it expects true.
- Test with a failing _dao_: it instantiates a _NumberSaverHelper_ with the _dao_ returning false. It use a valid number but expect false in the hepler as well.
- Test with with an error in _dao_: it instantiates a _NumberSaverHelper_ with the _dao_ throwing the exception and expect it. Notice the annotation `@Test(expected = PhoneNumberDao.PhoneNumberDaoException.class)`


## View Presenter Unit Test example

Now that we have seen a very basic unit test, let's do it something more related with Android. Probably you are familiar with _Model View Presenter_, so in this case we are going to test the presenter with unit testing. For this purpose is very important to keep the presenter free of any Android reference. This is not always possible, but is a good practise, because we can test our presenter in a really quick and easier ways than having Android references. Android references implies that we need an emulator to run with, and we would need to mock much more objects as well.

Let's say we have the _NumberHandler_ presenter and its view interface. The prsenter is used to be the communicator between the view and the _NumberSaverHelper_.

```java
public class NumberHandlerPresenter {

    private NumberHandlerView view;
    private NumberSaverHelper numberSaverHelper;

    public NumberHandlerPresenter(NumberSaverHelper numberSaverHelper) {
        this.numberSaverHelper = numberSaverHelper;
    }

    public void onAttach(NumberHandlerView numberHandlerView) {
        this.view = numberHandlerView;
    }

    public void onDetach() {
        this.view = null;
    }

    public void onSaveNumber(int number) {
        if (view == null) {
            return;
        }

        try {
            boolean saved = numberSaverHelper.savePhoneNumber(number);
            if (saved) {
                view.showNumberSaved();
            } else {
                view.showNumberSavingError("The number is not valid");
            }
        } catch (PhoneNumberDao.PhoneNumberDaoException exception) {
            view.showNumberSavingError("Error while saving the number");
        }
    }
}
```

```java
public interface NumberHandlerView {
    void showNumberSaved();
    void showNumberSavingError(String error);
}
```

Again, I invite you to try the following points by yourself before check the answers. Remember that we are going to test the presenter and not the _NumberSaverHelper_. We do not need any activity or fragment either for the view as long as we can mock it. _Tip: use test doubles to check the presenter callbacks_

Note: in this case, to show other variants, we are going to use [Mockito](https://site.mockito.org) to mock the objects and spy the calls.

1. When a valid number is entered to the presenter, the view is notified with the positive result.
2. When a non-valid number is entered, the view is notified about the error.
3. When the _PhoneNumberSaver_ throws an error, the view is notified about the error.
4. When the view is not attached, there's no crash if we try to use the presenter method.

...

Done? Let's go with the solution!

```java
@RunWith(MockitoJUnitRunner.class)
public class NumberHandlerPresenterTest {

    private final static int ANY_NUMBER = 123456789;

    @Test
    public void shouldShowSucessMessageWhenSaveAValidNumber() throws PhoneNumberDao.PhoneNumberDaoException {
        NumberHandlerPresenter numberHandlerPresenter = givenSuccessNumberHandlerPresenter();
        NumberHandlerView view = givenNumberHandlerView();
        
        numberHandlerPresenter.onAttach(view);
        numberHandlerPresenter.onSaveNumber(ANY_NUMBER);

        verify(view, times(1)).showNumberSaved();
    }

    @Test
    public void shouldShowErrorMessageWhenSaveNonValidNumber() throws PhoneNumberDao.PhoneNumberDaoException {
        NumberHandlerPresenter numberHandlerPresenter = givenFailingNumberHandlerPresenter();
        NumberHandlerView view = givenNumberHandlerView();
        
        numberHandlerPresenter.onAttach(view);
        numberHandlerPresenter.onSaveNumber(ANY_NUMBER);

        verify(view, times(1)).showNumberSavingError(anyString());
    }

    @Test
    public void shouldShowErrorMessageWhenExceptionOccurs() throws PhoneNumberDao.PhoneNumberDaoException {
        NumberHandlerPresenter numberHandlerPresenter = givenExceptionNumberHandlerPresenter();
        NumberHandlerView view = givenNumberHandlerView();
        
        numberHandlerPresenter.onAttach(view);
        numberHandlerPresenter.onSaveNumber(ANY_NUMBER);

        verify(view, times(1)).showNumberSavingError(anyString());
    }

    @Test
    public void shouldNotCrashIfViewIsNotAttached() throws PhoneNumberDao.PhoneNumberDaoException {
        NumberHandlerPresenter numberHandlerPresenter = givenSuccessNumberHandlerPresenter();
        NumberHandlerView view = givenNumberHandlerView();

        // not attaching the view
        numberHandlerPresenter.onSaveNumber(ANY_NUMBER);

        verify(view, times(0)).showNumberSaved();
    }

    NumberHandlerPresenter givenSuccessNumberHandlerPresenter() throws PhoneNumberDao.PhoneNumberDaoException {
        NumberSaverHelper numberSaverHelper = Mockito.mock(NumberSaverHelper.class);
        when(numberSaverHelper.savePhoneNumber(anyInt())).thenReturn(true);
        return new NumberHandlerPresenter(numberSaverHelper);
    }

    NumberHandlerPresenter givenFailingNumberHandlerPresenter() throws PhoneNumberDao.PhoneNumberDaoException {
        NumberSaverHelper numberSaverHelper = Mockito.mock(NumberSaverHelper.class);
        when(numberSaverHelper.savePhoneNumber(anyInt())).thenReturn(false);
        return new NumberHandlerPresenter(numberSaverHelper);
    }

    NumberHandlerPresenter givenExceptionNumberHandlerPresenter() throws PhoneNumberDao.PhoneNumberDaoException {
        NumberSaverHelper numberSaverHelper = Mockito.mock(NumberSaverHelper.class);
        when(numberSaverHelper.savePhoneNumber(anyInt())).thenThrow(PhoneNumberDao.PhoneNumberDaoException.class);
        return new NumberHandlerPresenter(numberSaverHelper);
    }

    NumberHandlerView givenNumberHandlerView() {
        return Mockito.mock(NumberHandlerView.class);
    }
}
```

Again, we start (at the bottom of the class) setting up the _given_ methods. In this case we want to get different variants of the presenter depending on the mocked _PhoneSaverHelper_ inside it. One of them returns true when we try to save a number, the other one returns false, and the last one throws an exception. Notice here how we have set the behaviour of the mocked objects. In the previous example we did it manually, but here we are using _Mockito_ to do it. It's pretty simple and verbose syntax. _When_ a method is called with _anyInt_ argument, _then return_ a result (or _throw_ an exception).

Test are pretty simple. Basically we are getting the necessary objects in the _given_ block: the presenter with the specified helper and the mocked view. Then we are attaching (or not) the view to the presenter and calling the method to tests. Notice that here we are not expecting any result but we are expecting a call to the view. That check is done in the _then_ block using _Mockito_ and spying the callbacks. The syntax is pretty simple in the same way it was specifying the behaviour. _Verify_ and object gets _x times_ calls to a _method_.

- Test saving the number should call the success method of the view.
- Test not saving the number (because it's not valid or because of an exception) should call the error method of the view.
- Test with non-attached view should not do anything (this implies do not crash)

Notice that, when the method of the view to show an error is called (`showNumberSavingError(String error)`), it receive a string as an argument. We are not checking which string it's using, and we are specifying `anyString()`. We could use the same strings used in the presenter, but actually it should use the _string resources_ to fit with any translation. We haven't used to not include Android code in the presenter. But we can include the translations in the presenter instead of use static vstrings for the message. In this case we would need to test the presenter using the Android emulator. The advantage is that, instead of check the method call with `anyString()`, we can check that the actual error message is being used. Check this [StackOverflow answer](https://stackoverflow.com/questions/39453986/android-espresso-assert-text-on-screen-against-string-in-resources/39454118#39454118) I gave about how to get string resources in Android tests.

Another option is create different method in the view depending on the error you want to show. This way the string resource will be retrieved in the view and you won't need to include Android references in the presenter. In this case, you can test the proper message is shown in the UI tests.

### Bonus1: testing background thread executors

If you are familiar with Android programming, you probably know that thread handling in one of the daily basis in the environment. This is because we want to keep heavy tasks out of the _UI thread_. So probably, in addition to the presenter, you would have some background runner, executor or similar. Let's add a basic _thread executor_. The task we are running is actually a dummy task, so we will just add a small delay to simulate de delay in the background thread.

```java
public class BackgroundExecutor {

    public void execute(Runnable runnable) {
        // This is a simple thread creator to simulate background working
        try {
            Thread.sleep(100);
            Thread runnerThread = new Thread(runnable);
        } catch (InterruptedException e) { }
    }
}

```

Now, we can use it in `NumberHandlerPresenter`. Let's change the `onSaveNumber` method to run the task in the `BackgroundExecutor`. We are not redirecting the response to the UI thread (because it's not important now), but we will do on the UI part.

```java
public class NumberHandlerPresenter {

    ...

    private NumberSaverHelper numberSaverHelper;
    private BackgroundExecutor backgroundExecutor;

    public NumberHandlerPresenter(NumberSaverHelper numberSaverHelper, BackgroundExecutor backgroundExecutor) {
        this.numberSaverHelper = numberSaverHelper;
        this.backgroundExecutor = backgroundExecutor;
    }

    ...

    public void onSaveNumber(final int number) {
        if (view == null) {
            return;
        }

        backgroundExecutor.execute(new Runnable() {
            @Override
            public void run() {
                try {
                    boolean saved = numberSaverHelper.savePhoneNumber(number);
                    if (saved) {
                        view.showNumberSaved();
                    } else {
                        view.showNumberSavingError(THE_NUMBER_IS_NOT_VALID_MSG);
                    }
                } catch (PhoneNumberDao.PhoneNumberDaoException exception) {
                    view.showNumberSavingError(ERROR_WHILE_SAVING_THE_NUMBER_MSG);
                }
            }
        });
    }
}
```

Now, try to run the tests again. What happens? Are the test "in green"?

Probably not, they fail since you introduced the background thread handling. This is happening because of a simple reason: test are being run in the main thread without waiting for any other thread to finish. As soon as the main thread finish to run the assigned code, the _test runner_ executes the _asserts_, but the results/callbacks are not finished yet. Although the test are correctly written they are failing because of the parallel execution of the thread. How can we fix this?

There's an easy way. It's actually the same way we solve most of the test issues: mock the _BackgroundExecutor_. If we mock it, we can override the execute method and run the received task in the same thread instead of create a new one. This way, we keep the thread handling in the app, but we test it in a single thread. (This is applicable to executors, handler, async tasks and so on).

This is how the mocked object looks inside the test class. And this is how we create the presenter now we have the _BackgroundExecutor_ (mocked object in the test).

```java
@RunWith(MockitoJUnitRunner.class)
public class NumberHandlerPresenterTest {

    ...

    private BackgroundExecutorMocked backgroundExecutorMocked = new BackgroundExecutorMocked();

    ...

    NumberHandlerPresenter givenWHATEVERTYPEWENEEDPresenter() throws PhoneNumberDao.PhoneNumberDaoException {
        NumberSaverHelper numberSaverHelper = Mockito.mock(NumberSaverHelper.class);
        when(numberSaverHelper.savePhoneNumber(anyInt())).thenReturn(true);
        return new NumberHandlerPresenter(numberSaverHelper, backgroundExecutorMocked);
    }

    class BackgroundExecutorMocked extends BackgroundExecutor {
        @Override
        public void execute(Runnable runnable) {
            runnable.run();
        }
    }
}
```

Now, all the test should pass correctly.

### Bonus2: static helpers

Static helpers are very common in some scenarios: we have the ability to access to some repetitive code or common methods without the necessity of instantiate them over and over again. But they are a big problem for testing. I mean, we can test them with no problem: we just need to call their methods and check they return succesfully. But what if we want to mock them? what if the _BackgroundExecutor_  was a static helper? We could not test our presenter just because of the static executor. Too bad..

Best solution is avoid static classes and use _Dependency Injection_ (like we have done in the example with _BackgroundExecutor_). That way we will get the same helper everywhere, and we can mock it in the tests. However, this is not possible in some stablished projects which are already using the static helper (and we can not change them).

There are two alternative "solutions" (notice they are not the best ones) in order to make static helpers compatible with tests:
1. Keep the helper static and don't mock it. This is risky because we can not simulate the different helper responses inside a class which is using it. (This solution won't work for helpers which need to be mocked to make the test pass, like _BackgroundExecutor_).
2. Make the test singleton with an injectable _Instance_ and _getInstance()_ method. In this case we need to change all calls to the helper to use the instance, and inject a mocked one in the tests. This will fix the problem with mocking the helper in tests and its use.

Note how important is adapt the base code to tests in order to be able to run them. In other words __make the code testable__.
