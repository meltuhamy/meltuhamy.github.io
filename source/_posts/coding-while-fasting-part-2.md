---
title: Coding While Fasting Part 2
tags:
  - coding
  - coffee
  - fasting
  - food
  - islam
  - ramadan
  - front-end
  - js
  - testing
categories:
  - productivity
banner: /images/ramadan-dates-banner.png
thumbnail: /images/dates-cup.jpg
date: 2016-06-14 21:56:00
---

_[Read Part 1: General productivity tips for coding during Ramadan](/productivity/coding-while-fasting/)_

The assumption is that when you're _tired_, you're more likely to lose focus and therefore make mistakes. It is important to combat this tendency by first not blaming your lack of productivity and mistakes on fasting, and second by finding and using productivity tools and techniques. In the previous post, [I mentioned 5 tips that can help you with general productivity during Ramadan](/productivity/coding-while-fasting/). In this post, I focus on technical techniques and tools to help you be a more productive coder during Ramadan.

## 1. Catch errors fast.

![Squash 'em before they become a problem](/images/bugsquashing.jpg)

### Linting
Sometimes, you want to have a safety net to catch common typos and errors. You can use _linting_ to catch out obvious errors and typos. Think of it as a spell checker for code. Here's an example showing the use of [eslint](http://eslint.org/) to lint JavaScript code.

```js
var foo = bar;
```
```
1:5 - 'foo' is defined but never used (no-unused-vars)
1:11 - 'bar' is not defined. (no-undef)
```

When coding, having such safety nets is useful in general, and when you're fasting they certainly help you catch common mistakes.

We can attach a linter to a pre-commit hook. This means you won't be able to commit code if there are any linting issues like above. [Here](https://coderwall.com/p/zq8jlq/eslint-pre-commit-hook)'s a pre-commit hook that uses eslint.

## 2. Focus on one thing at a time. Tests.
![Tests make you focus!](/images/tests-focus.png)

Test driven development becomes more useful when you're fasting because of a couple of reasons:

1. It allows you to focus on one thing at a time.
2. It allows you to catch errors before you commit them.

I find that it is always useful to _write down what you're doing_ - it allows you to focus on the task at hand and not get distracted by other things. Writing tests allows you to do this, it gives you a 'task' that you must fix before moving on to the next thing.

There are tools to help you with testing. [PhantomFlow](https://github.com/Huddle/PhantomFlow) allows you to write front end user flows. Here's an example flow:

```js
flow("Get a coffee", () => {
  step("Go to the kitchen", goToKitchen);
  step("Go to the coffee machine", goToMachine);
  decision({
    "Wants Latte": () => {
      chance({
        "There is no milk": () => {
          step("Request Latte", requestLatte_fail);
          decision({
            "Give up": () => {
              step("Walk away from the coffee machine", walkAway);
            },
            "Wants Espresso instead": wantsEspresso
          });
        },
        "There is milk": () => {
          step("Request Latte", requestLatte_success);
        }
      });
    },
    "Wants Cappuccino": () => {
      // ...
    },
    "Wants Espresso": wantsEspresso
  });
});
```

Now that the user flow is written, even if it is just stubs like above, you have in your head a general plan of what the feature will look like. You can then implement each step and focus on it. For example, you can implement `goToMachine`. For more details about PhantomFlow, check out the GitHub repo [here](https://github.com/Huddle/PhantomFlow).

## 3. Code reviews and pair programming
![Pair programming](/images/minions-programming.gif)

A good way to keep productive while fasting is to _work with someone_.

>Pair programming is an agile software development technique in which two programmers work together at one workstation. One, the driver, writes code while the other, the observer or navigator, reviews each line of code as it is typed in. The two programmers switch roles frequently.

*From [Wikipedia](https://en.wikipedia.org/wiki/Pair_programming)*

Pair programming and code reviews have many [benefits](http://blog.codinghorror.com/pair-programming-vs-code-reviews/). When fasting, they are extra useful because:

1. When in the driving seat, they force you to be constantly focussed. Someone else is watching you. Better not make mistakes.
2. When in the observer seat they allow you to 'rest' from actual coding, but still allow you to contribute to the solution.
2. They catch out problems in your code, and allow you to learn from someone else's experience.

## Conclusion
![Ramdan Mubarak!](/images/ramadan-sublime-screenshot.png)

Ramadan is a month of spirituality, but it is also a month of productivity. It forces you to focus - whether it is on your worship, or on your work. Finding tools and techniques to help you focus during Ramadan is not only good to get you through the month, but the things you learn can be applied throughout the year.
