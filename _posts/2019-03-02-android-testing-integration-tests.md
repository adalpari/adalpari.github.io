---
title: "Android Testing - Integration tests"
header:
  teaser: /assets/images/integration_tests.jpg
  overlay_image: /assets/images/integration_tests.jpg
---

In this series of posts we are going to talk about Android Testing. It's a big field to talk about, but here there are some important points and tips.

1. [Introduction.](../android-testing-introduction/)
2. [How to test.](../android-testing-how-to-test/)
3. [Unit tests.](../android-testing-unit-tests/)
4. [Integration tests.](../android-testing-integration-tests/)

Now that we understand how a typical unit test works on Android, it's time to go a step further. It's very important to be sure about every single and atomic part of an app is working correctly, but also it's important to be sure that they are working in the correct way when we use them together and mix them with external code. 

This is why integration tests are necessary. Not only in our classes (or layers) working together, but also integrating our app with external libraries, classes or tools. So, more than testing how our code works when everything is connected, we are going to test how our code works when it's communicating with external code. In other words: we are going to test the integration of our different layers.

## Integration test example: testing SharedPreferences

Let's start with an example of a class which interacts with _SharedPreferences_. In order to call _Android_ code, we need to run an _Android_ container to provide the native _sdk_. So, in this case, tests are going to be slower than _unit tests_ because of the _Android_ dependency. Normally we are going to run these tests over an emulator or a physical device.
In addition, we should think about the structure of these kind of classes since we are going to interact with a heavy core like _Android sdk_. We have talked about the importance of adapting the production code to be testable, and we are going to iterate over that idea.

Imagine that we have a _Fragment_ and we are using _SharedPreferences_ inside it to store and retrieve user preferences. Of course the _fragment_ has other _Android sdk_ interactions, like the _lifecycle_ or the view bindings. So, if we want to test the _SharedPreferences_ methods, we need to run the whole fragment and provide a scope for the other _Android_ requirements, don't we?. 

Yes, that's right. But we can also avoid them by moving the _SharedPreferences_ methods into a _helper_ or _util_ class. That way we will still need to run an _Android_ container (because _SharedPreferences_ belongs to _Android sdk_) but we will avoid the extra _Fragment_ dependencies. In addition, moving logic code out of the _view_ (in this case the _fragment_) fits better with a clean architecture pattern like _Model-View-Presenter_.

Here is the _SettingsHelper_. It's a simple class to just put and get values from _SharedPreferences_. The class is quite simple and we could discuss about wrapping the saver service, but it's not the purpose of this post.

```java
public class SettingsHelper {
    
    private static final String EMPTY_STRING = "";
    
    private final SharedPreferences sharedPreferences;

    public SettingsHelper(SharedPreferences sharedPreferences) {
        this.sharedPreferences = sharedPreferences;
    }

    public void addSetting(String key, String value) {
        if (sharedPreferences == null || key == null || value == null) {
            return;
        }
        
        sharedPreferences.edit().putString(key, value).commit();
    }

    public String getSettingValue(String key) {
        if (sharedPreferences == null || key == null) {
            return EMPTY_STRING;
        }
        return sharedPreferences.getString(key, EMPTY_STRING);
    }
}
```

Easy right?, let's test it!

If you just create a test like we have been doing on the past post, probably you are wondering how can you get the _SharedPreferences_ object in order to inject it to the helper. Where we can get the _Context_?. Well, you can figure out that we need another way to test our helper. Now, instead of put the test on the regular _test_ folder, we have to create it on the _androidTest_ folder. This means that tests are going to be run using the _Android sdk_, so we can access to _Android_ methods. _(Remember these tests are going to be slow)_.

![androidTest folder](/assets/images/androidTest.png)

```java
public class SettingsHelperTest {

    private final static String SETTING_KEY = "my_setting_key";
    private final static String SETTING_VALUE = "my_setting_value";

    private final static String SETTING_NON_EXISTENT_KEY = "my_setting_non_existing_key";

    private SettingsHelper settingsHelper;

    @Before
    public void setUp() throws Exception {
        SharedPreferences sharedPreferences = getSharedPreferences();

        this.mockedAPIClient = new MockedAPIClient();
        this.settingsHelper = new SettingsHelper(sharedPreferences, mockedAPIClient);
    }

    @After
    public void tearDown() throws Exception {
        SharedPreferences sharedPreferences = getSharedPreferences();
        SharedPreferences.Editor editor = sharedPreferences.edit();
        editor.clear().commit();
    }

    private SharedPreferences getSharedPreferences() {
        return getInstrumentation().getTargetContext()
            .getSharedPreferences("SettingsHelperTest", Context.MODE_PRIVATE);
    }

    @Test
    public void shouldReturnBlankForNonExistentSetting() {
        // don't add any value

        String retrievedValue = settingsHelper.getSettingValue(SETTING_NON_EXISTENT_KEY);

        Assert.assertEquals("", retrievedValue);
    }

    @Test
    public void shouldReturnRightSetting() {
        settingsHelper.addSetting(SETTING_KEY, SETTING_VALUE);

        String retrievedValue = settingsHelper.getSettingValue(SETTING_KEY);

        Assert.assertEquals(SETTING_VALUE, retrievedValue);
    }

    @Test(expected = Test.None.class /* no exception expected */)
    public void shouldNotCrashWhenTryToRetrieveNullSetting() {
        settingsHelper.getSettingValue(null);
    }
}
```

As you can see, we are making simple tests for this simple class. Let's see them step by step.

First, on the `setUp()` method, we are preparing the objects we are going to test. In this case, we are able to get the _context_ from our application with `getInstrumentation().getTargetContext()`, then call the _SharedPreferences_ and inject them in our _SettingsHelper_. This is a simple (and relatively fast) way to get the _context_ and use it. When run the tests, we are not going to see anything on the simulator or the phone, because we are not raising up any activity. But we are actually "running" the test over an instance of the app. We will see _UI testing_ in the next post, but right now we don't need any interaction with any activity or fragment, because we are just testing a helper class that we have extracted from them, remember?

Secondly, it's important to keep the state in the same way we found it. So we are clearing _SharedPreferences_ in the `tearDown()` method. __Caution:__ be careful when clean things in tests, in order to not remove production states by mistake.

After the initialisation, we have the test classes. They look quite simple, similar to a unit test. This is not only because the class we are going to test is simple, but also because they should be simple. If we need a complex test in order to check a method or a case, is a symptom that we could (and probably should) refactor the class and make it simpler.

So, we just need to call the helper methods, adding and retrieving setting from it and _asserting_ that the output is correct. There's another test case in the example, in which we just run a method with a _null_ input and don't do anything else. This is done to check our helper is not throwing _Null Pointer Exception_ in that case. In addition, you can add more _NPE_ checks, or even initialise the helper with _null_ instead of _SharedPreferences_ and see what happens.

## Integration test example: testing an external API handling

Now, imagine that we want not only to store locally the settings, but also send them to the server, and retrieve them from it as well if the setting is not present locally. The following code shows the _APIClient_ interface and the modified code in _SettingsHelper_.

```java
public interface APIClient {
    void storeSetting(String key, String value);
    String getSetting(String key);
}
```
```java
public class SettingsHelper {

    private static final String EMPTY_STRING = "";

    private final SharedPreferences sharedPreferences;
    private final APIClient apiClient;

    public SettingsHelper(SharedPreferences sharedPreferences, APIClient apiClient) {
        this.sharedPreferences = sharedPreferences;
        this.apiClient = apiClient;
    }

    public void addSetting(String key, String value) {
        commitToSharedPreferences(key, value);
        storeInAPI(key, value);
    }

    private void commitToSharedPreferences(String key, String value) {
        if (sharedPreferences == null || key == null || value == null) {
            return;
        }

        sharedPreferences.edit().putString(key, value).commit();
    }

    private void storeInAPI(String key, String value) {
        if (apiClient == null || key == null || value == null) {
            return;
        }

        apiClient.storeSetting(key, value);
    }

    public String getSettingValue(String key) {
        if (sharedPreferences == null || apiClient == null || key == null) {
            return EMPTY_STRING;
        }

        String settingInPreferences = sharedPreferences.getString(key, EMPTY_STRING);

        if (settingInPreferences != EMPTY_STRING) {
            return settingInPreferences;
        } else {
            return apiClient.getSetting(key);
        }
    }
}
```

Probably, a real _APIClient_ will handle the request asynchronously, but we have talked about how to mock async calls as well in [Unit tests](../android-testing-unit-tests/).

Now we need to test the interaction with the _APIClient_ as well. We already know the _double tests_ ([How to test](../android-testing-how-to-test/)). Then, the solution is mock the _APIClient_ and check the made calls. We can just replace methods manually, creating a _CustomAPIClient_ implementing the _APIClient_ interface.

```java
public class SettingsHelperTest {

    private final static String SETTING_KEY = "my_setting_key";
    private final static String SETTING_VALUE = "my_setting_value";

    private final static String SETTING_NON_EXISTENT_KEY = "my_setting_non_existing_key";

    private final static String SETTING_KEY_EXISTENT_ONLY_IN_API = "my_setting_key_existent_only_in_api";
    private final static String SETTING_VALUE_EXISTENT_ONLY_IN_API = "my_setting_value_existent_only_in_api";

    public static final String BLANK = "";

    private MockedAPIClient mockedAPIClient;

    private SettingsHelper settingsHelper;

    @Before
    public void setUp() throws Exception {
        SharedPreferences sharedPreferences = getSharedPreferences();

        this.mockedAPIClient = new MockedAPIClient();
        this.settingsHelper = new SettingsHelper(sharedPreferences, mockedAPIClient);
    }

    @After
    public void tearDown() throws Exception {
        SharedPreferences sharedPreferences = getSharedPreferences();
        SharedPreferences.Editor editor = sharedPreferences.edit();
        editor.clear().commit();
    }

    private SharedPreferences getSharedPreferences() {
        return getInstrumentation().getTargetContext().getSharedPreferences("SettingsHelperTest", Context.MODE_PRIVATE);
    }

    @Test
    public void shouldReturnBlankForNonExistentSetting() {
        // don't add any value

        String retrievedValue = settingsHelper.getSettingValue(SETTING_NON_EXISTENT_KEY);

        Assert.assertEquals(BLANK, retrievedValue);
    }

    @Test
    public void shouldReturnRightSetting() {
        settingsHelper.addSetting(SETTING_KEY, SETTING_VALUE);

        String retrievedValue = settingsHelper.getSettingValue(SETTING_KEY);

        Assert.assertEquals(SETTING_VALUE, retrievedValue);
    }

    @Test(expected = Test.None.class /* no exception expected */)
    public void shouldNotCrashWhenTryToRetrieveNullSetting() {
        settingsHelper.getSettingValue(null);
    }

    @Test
    public void shouldStoreSettingInAPI() {
        settingsHelper.addSetting(SETTING_KEY, SETTING_VALUE);

        Assert.assertEquals(1, mockedAPIClient.storeSettingTimesCalled);
        Assert.assertEquals(SETTING_KEY, mockedAPIClient.lastKeyUsed);
        Assert.assertEquals(SETTING_VALUE, mockedAPIClient.lasValueUsed);
    }

    @Test
    public void shouldRetrieveSettingFromAPIWhenIsBlankInSharedPreferences() {
        // don't add any value

        String retrievedValue = settingsHelper.getSettingValue(SETTING_KEY_EXISTENT_ONLY_IN_API);

        Assert.assertEquals(1, mockedAPIClient.getSettingTimesCalled);
        Assert.assertEquals(SETTING_VALUE_EXISTENT_ONLY_IN_API, retrievedValue);
    }

    private class MockedAPIClient implements APIClient {

        public int storeSettingTimesCalled = 0;
        public String lastKeyUsed;
        public String lasValueUsed;

        public int getSettingTimesCalled = 0;

        @Override
        public void storeSetting(String key, String value) {
            storeSettingTimesCalled++;
            lastKeyUsed = key;
            lasValueUsed = value;
        }

        @Override
        public String getSetting(String key) {
            getSettingTimesCalled ++;

            if (SETTING_KEY_EXISTENT_ONLY_IN_API.equals(key)) {
                return SETTING_VALUE_EXISTENT_ONLY_IN_API;
            } else {
                return BLANK;
            }
        }
    }
}
```

Going from top to bottom, we have added the _MockedAPIClient_ in the `setUp()` method, in order to use it in the helper. If you take a look to the manually mocked class, it's just an implementation of the interface, returning static values and keeping the calls states to allow check them.

Then, we have the tests from the previous example, and finally the new test methods and the _MockedAPIClient_ itself.

The test `shouldStoreSettingInAPI()` is calling to the `addSetting()` method, like we did in the previous test (`shouldReturnRightSetting()`). But this time we are going to actually check the call to the _API_ instead of retrieve the value from it to make the assert. This is because the previous test is testing that the value is actually stored in _SharedPreferences_, but now the _API_ is mocked, so we need to check the call instead. Here is where we are using the state values in the mocked object.

Finally `ShouldRetrieveSettingFromAPIWhenIsBlankInSharedPreferences()` is doing something similar. As you see, we are also calling a method and checking the states, but in this case we are also checking the return of the method. The requested setting should be not present in _SharedPreferences_ so it should be returned from the _API_. If you check the method of the mocked _API_ we are returned a value for it, which is the one we assert on.

After this example, we can extract an important conclusion: even for more complex classes or for the ones using external dependencies, the key is be able to mock part of the code and test only the part we are interested on.