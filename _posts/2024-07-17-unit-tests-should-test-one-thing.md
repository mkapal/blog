---
layout: post
title: Unit tests should test one thing
tags: testing jest javascript
summary: "This is a cool post"

---

PEREX TBA

Suppose that our application has a feature where users can switch between languages. There is a language service which stores the selected language in a persistent key-value storage:

```ts
function createLanguageService(storage) {
  return {
    getLanguage: () => {
      return storage.get('language') ?? 'en';
    },
    setLanguage: (language) => {
      storage.set('language', language);
    }
  }
}
```

Our goal is to cover this service with unit tests.

If we have a look at the service implementation above, we can come up with the following test case:

1. Create an instance of the language service
2. Change the language by calling `setLanguage("de")`
3. Verify that `getLanguage()` returns `"de"`

The language service requires storage as an argument so we need to mock it in our test.

Here's how the full test case implementation would look:

```ts
const createMockStorage = () => {
  let data = {};

  return {
    get: (key) => {
      return data[key];
    },
    set: (key, value) => {
      data[key] = value;
    }
  }
}

it('should set the language', () => {
  const storage = createMockStorage();
  const languageService = createLanguageService(storage);
  
  languageService.setLanguage('de');  
  
  expect(languageService.getLanguage()).toEqual('de');
});
```

However, there is an issue with this approach.

Our single test doesn't only test `setLanguage` but also `getLanguage`. Moreover, it implicitly assumes that `getLanguage` will return "de". This means that the `getLanguage` could be implemented to always return "de" and the test would still pass, even though it would obviously be incorrect behavior in a real application:

```ts
function createLanguageService(storage) {
  return {
    getLanguage: () => {
      return 'de';
    },
    setLanguage: (language) => {
      storage.set('language', language);
    }
  }
}
```

We can fix this by adding an assertion before calling `setLanguage` to make sure the language is "en" by default.

```ts
  expect(languageService.getLanguage()).toEqual('en');
  
  languageService.setLanguage('de');
  
  expect(languageService.getLanguage()).toEqual('de');
```

But we still test too many things at once. A unit test should focus on testing just one unit, which is the `setLanguage` function in our case. So what are we interested in when we call `languageService.setLanguage('de')`? Well, we want to make sure that "de" is then passed to the storage. In other words, we must check if the `storage.set` function was called with "de" in its argument. To implement this, we can make use of the `jest.fn()` function to keep track of the number of calls of `storage.set`:

```ts
const storage = {
  set: jest.fn(),
}
```

You may notice that the `storage.get` function is missing from the mock storage - that is fine because `storage.get` is never called in the test case, so we can mock only what is required for our test to run without crashing.

So, we want to verify that `storage.set` is never called initially, then call `languageService.setLanguage`, then verify that `storage.set` has been called with "en".  Here's how the test code looks now:

```ts
it('should set the language in storage', () => {
  const storage = {
    set: jest.fn(),
  }
  const languageService = createLanguageService(storage);
  
  expect(storage.set).not.toHaveBeenCalled();
  
  languageService.setLanguage('en');
  
  expect(storage.set).toHaveBeenCalledWith('en');
});
```

This test has a well defined scope, which is the `setLanguage` function.

In order to test the `getLanguage` function, we would write a different set of test cases. Here's an example:

```ts
it('should get the language from storage', () => {
  const storage = {
    get: () => 'de',
  }
  const languageService = createLanguageService(storage);
  
  expect(storage.get).not.toHaveBeenCalled();
  
  const language = languageService.getLanguage();
  
  expect(storage.get).toHaveBeenCalled();
  expect(language).toEqual('de');
});
```

## Conclusion

TBA
