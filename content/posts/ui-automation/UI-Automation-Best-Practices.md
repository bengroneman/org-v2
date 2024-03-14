---
categories:
  - Automation
date: "2023-04-09"
description: "A remote friendly leadership excercise for the whole team"
slug: practical-class-on-leadership-styles
tags:
  - automation
  - code
  - learning
title: UI Automation Best Practices
---
# E2E Automation Best Practices

## Background

Below is an outline of best practices to follow when creating or updating any UI test in our suites.

## **Use accessible selectors**

### Definition

Using good selectors is essential to ensuring a healthy test suite. If the AUT is lacking good selectors, be sure to write a story for the team describing the elements that should have a data-testid attribute.

### Pros

Using the right selector will ensure the accuracy of the test, and if the selector is unique we can be sure that we will get the same element everytime, making the test cases deterministic.

### Cons

It can be challenging to find good selectors, and creating them requires external dependencies.

### Pattern

At LIO a majority of our services are offered as CRM plugins, which can leave us short handed when searching for good selectors. In general, you’ll want to avoid *elements of style* *(CSS classes and ID’s)*, this is because they are subject to change, and have a specific purpose. Instead opt for accessible selectors that are **unique**, **describe** the element, and are as **specific** as possible. Here are some examples of good selectors.
```python
# English agent version
page.get_by_role("button", name="Re-translate using the alternative MT engine")
# Spanish agent version
page.get_by_role("button", name="Vuelva a traducir utilizando el motor MT alternativo")
page.get_by_test_id("filter-showTranslatedMessages")
```
By **specifying** the role of button to narrow the search, and by using a **unique** name that **describes** the element, it can generates a stable selector that documents itself. The get_by_role function can get you a long way, but when you are working within the scope of our internal plugins and don’t have an accessible selector to use, its best to turn to using data-testids.
In this example we have a dedicated and **unique** selector, that has one target function (to filter to show translated messages). This gives us the **specifics** of the element and **describes** it to us.

For further information refer to the Playwright locator philosophy:

[https://playwright.dev/docs/best-practices#testing-philosophy](https://playwright.dev/docs/best-practices#testing-philosophy)

## **Use the Page Object Model**

### Definition

The page object model is a great way to separate concerns, and keep your test cases organized. Ultimately the test function should handle the assertions and test documentation, while the page object should house any element selectors, or common patterns for a particular section/page of the AUT.

### Pros

Allows for reusability of the elements on the page, while also enabling encapsulation of data in order to make the code more readable, and maintainable.

### Cons

Encourages a class based approach.

### Pattern

To get started with building a page object, create instance variables for data you may want to use frequently (a base url is a great addition here). Then identify elements on the page you’ll be interacting with and build getter methods (class methods with a prefix of get_) for those elements.

While you develop test cases, be sure to identify patterns that you’re likely to do regularly (navigate to page, fill out form, translate message, etc..); once you identify these patterns you can hoist them into the page object model for reusability. Here's an example of the start of a page object:

```python
# TODO: add example here
```

## **Type your variables and return types**

### Definition

Since [Python 3.5](https://docs.python.org/3.5/library/typing.html) developers have had the opportunity to implement static typing. By leveraging a modern IDE such as Visual Studio Code, or Pycharm you’ll unlock documentation and hints as to what properties exist on a function, variable, or parameter. In addition you’ll also turn runtime errors into build time errors using the right tools.

### Pros

Improves the documentation of the code, while increasing developer efficiency. Using type checkers like MyPy will surface runtime errors as build time errors, saving time in the long run.

### Cons

You’ll need to spend some extra time *typing*, and updating when something changes within the *type*.

### Pattern

```python
# Variables types
translations: [Locator] = self.get_translations(page=self.page)
# Function return, and parameters types
def get_translations(page: Page) -> [Locator]:
    return page.locator("[data-is-translation]").all()
```

Typing your Python code is strongly encouraged. To get started here is a good reference sheet:

[https://mypy.readthedocs.io/en/stable/cheat_sheet_py3.html](https://mypy.readthedocs.io/en/stable/cheat_sheet_py3.html)

Depending on your editor of choice you may or may not have baked in support for type checking.

## **Document your test cases**

### Definition

Documentation can take place in many ways within a code base, it can show up in docstrings, naming conventions, object and class models, or even external dependencies found in Jira.

### Pros

Allowing for documentation and code to coexist together makes it simpler when maintaining any given test case, or triaging a failed set of test cases.

### Cons

It can be a challenge to keep documentation up to date and in sync with other systems.

### Pattern

Start with a manually documented test case, and use the test cases steps to document each action in your test case. The codebase uses Allure for reporting, and so a good way of documenting a test case is using the `allure.step()` method. See below for an example of this:

```python
with step("Customer setups up chat with Agent"):
    self.setup_manual_customer_chat(page=customer_page, first_name=first_name)
    self.agent_accept_recent_chat_invite(self.page)
```

As shown above, you’ll want to use allure steps for the actionable pieces of your code. Doing so will allow you to also see at which step the test failed/passed, helping to narrow down the issue.

Another great way of documenting is to add a docstring to a function, class, or test case that needs extra context. Here are some examples of how docstring should be used:

```python
# Using docstrings to document parameters and return values
@staticmethod
def setup_manual_customer_chat(
    page: Page,
    lang: str = "",
		email: str = get_random_email(),
		subject: str = "",
		first_name: str = "Tim",
) -> None:
		"""
		@param page: the global page object to use
		@param lang: the ISOCode value for the desired language
		@param email: will generate and default to a random email
		@param first_name: the first name to fill out in the form
		@param subject: the subject of the customer chat
		"""
		# TODO: implement me

```

## **Separate data from your test cases**

### Definition

Fixtures are a [pytest feature](https://docs.pytest.org/en/6.2.x/fixture.html) that can be used in a variety of use cases, ranging from generator functions that setup settings changes, to methods that provide multilingual content.

### Pros

Avoids hard-coding data, and allows for reusability of code.

### Cons

Fixtures can’t be called directly by design, which in some situations reduces their flexibility.

### Pattern

Currently the WebPage class is a low level class that has tons of customer responses in a variety of languages, you can easily import them by adding them to the parameters of your test case (your editor should pick them up). Whereas in the SFPage class there are lots of fixtures that are specific to changing settings in the Salesforce packages. Here's an example of both of those:

```python
# pages/WebPage.py
@staticmethod
@pytest.fixture()
def japanese_description() -> str:
		return "パスワードを保存した後に動作することを確認する方法を教えてください。"
# pages/sf/SFPage.py
@pytest.fixture()
def toggle_translation_enabled_off(self) -> Generator:
		assert self.settings.set_chat_setting(self.settings.translation_enabled, False)
		yield
		assert self.settings.set_chat_setting(self.settings.translation_enabled)

# Here is an example of these fixtures being used by a test function
@pytest.mark.usefixtures(
		"toggle_translation_enabled_off"
)
@pytest.mark.translation_toggle
@pytest.mark.smoke
def test_translation_toggle_begin_translation_once_enabled(
		self, japanese_description
) -> None:
```