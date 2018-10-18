---
title: "Android Testing - Introduction"
header:
  teaser: /assets/images/android-testing.jpg
  overlay_image: /assets/images/android-testing.jpg
---

In this series of posts we are going to talk about Android Testing. It's a big field to talk about, bet here there are some important points and tips.

1. [Introduction.](../android-testing-introduction/)
2. [How to test.](../android-testing-how-to-test/)

First of all we need a brief introduction to general testing. 

## What is testing? 

Well, open our app and manually try all the functionalities is testing, right? But think about every time you make a change in you app. In order to have a good testing coverage you would need to try all the functionalities of the app every time you make a new change. And repeat it over and over again in your develop cycle.
Don't get me wrong, manual testing is necessary to know how our app looks and how our app works. But in addition to this, having automatic tests, and running them in every change we make, will help us with the project development and maintenance. So better we spend some time developing tests than spend a lot of time doing only manual testing.

On the other hand, one tend to think that testing help us to know if out app works correctly. But it does much more. When we develop a new feature, adding tests help us to know if it works. But also they should tell us if it does not work, or if it crash. And more important: where or even why it does not work. 
But it goes beyond there. While we keep developing the project, those tests assure that we are not breaking the previous developed features (recession bugs). We can now even change the code or refactor, being more confident about doing it right.
In addition, when a new bug is found, we can fix it and make a new test to be sure that bug is not going to happen again.

## F.I.R.S.T. principles

Acording to [_Robert C. Martin_](../Clean-Code/), tests should follow the _FIRST_ rule:

- __Fast__: tests should be fast when running. We want to run tests as much as possible on every iteration. If they don't run fast, it's going to be a bit painful.
- __Independent__: tests should not depend on each other. This is, every test should be individual, they shouldn't expect results from other test or actions. This way we can run test individually or in random order. (In fact, in _Android Studio_ they are run in random order to assure this property)
- __Repeatable__: tests should be repeatable in every run. If a test is repeatable means that we can trust it. If not, we won't be sure about its results.
- __Self-Validating__: tests should have a boolean output. True if they pass or false if not. This way we can run multiple tests expecting all of them are true, instead of manually inspect the result of each one.
- __Thorough__: They should cover all user cases. Having reliable tests means you are simulating all possible scenarios and you can be sure they are working.

## Tests clasification

There are multiple ways to classify tests depending on the level we want to achieve. _E.g: White box tests, black box tests, acceptance tests, user tests, scope tests, and so on.

However we are going to use one of the most used classification when we talk about Android testing: _Unit tests, integration tests and UI Tests_

- __Unit tests__: are tests running single operations or methods. They are really fast, small and easy to run. We should create as much unit tests as we need to cover our core methods.
- __Integration tests__: are test running different modules together. This means, we have to prepare the infrastructure to make them work. They are bigger and slower than the first ones, so we will have less test on this level.
- __UI tests__: are test that interact directly with what the user see and can do. These test are the slowest ones and require to set up the environment. Because of this, we will have just a few UI tests.

These points are well represented in the following pyramid. Remember: lots of Unit test, less integration tests and even less UI tests.

![testing pyramid](/assets/images/testing-pyramid.png)

This way we will cover almost all our code flow. Testing from the core logic to the user presentation of the app.


## Tests in Android

We need a test runner to be able to run the tests against our Android app. The default test runner is JUnit. We can use other runners, but JUnit should be enough to run our tests, so no change is needed. If you are not familiar with testing, we will see some basic interactions with the runner in the following posts.

And last but not least, let's see how test are organised in Android. When we create an Android applications, along with some configuration files, a source folder is also created in _/appName/src/main/java/package_structure_. In addition to this, we can create two types of tests in two different folders (we will see how they are created automatically by the IDE):

- _/appName/src/test/java/package_structure_: these test will run over Java virtual machine. So, we will test here classes that use only pure Java (or Kotlin) code. We won't include here any test that contains Android code.
- _/appName/src/androidTest/java/package_structure_: these tests need an Android emulator or physical device to run with. So, we will include here all tests containing any Android code (also non-UI Android tests that need objects like _Context, Android resources and so on_).

Note: to follow good practise, tests should be included in the same package structure than its tested classes.

In order to create a test automatically, put the cursor on the class name, push _alt + enter_ and select _Create test_

![create test](/assets/images/create-test.png)

You will see a wizard to create a test, asking you for the test name and some optional configurations. After that, you can choose where to create the test, inside _test_ or _androidTest_ folder.

![test config](/assets/images/test-config.png)