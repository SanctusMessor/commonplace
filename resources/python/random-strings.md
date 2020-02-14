---
description: For secrets and other useful things
---

# Random Strings

{% hint style="danger" %}
**Do Not Do This**: This Stack Overflow question is the current top Google result for "random string Python". The current top answer is "wrong." 

```text
''.join(random.choice(string.ascii_uppercase + string.digits) for _ in range(N))
```
{% endhint %}

This is an excellent method, but the [PRNG](http://en.wikipedia.org/wiki/Pseudorandom_number_generator) in random is not cryptographically secure. I assume many people researching this question will want to generate random strings for **encryption or passwords**. You can do this **securely** by making a small change in the above code:

```text
''.join(random.SystemRandom().choice(string.ascii_uppercase + string.digits) for _ in range(N))
```

Using `random.SystemRandom()` instead of just random uses /dev/urandom on \*nix machines and `CryptGenRandom()` in Windows. These are cryptographically secure PRNGs. Using `random.choice` instead of `random.SystemRandom().choice` in an application that requires a secure PRNG could be potentially devastating, and given the popularity of this question, I bet that mistake has been made many times already.

{% hint style="success" %}
If you're using python3.6 or above, you can use the new [secrets](https://docs.python.org/3/library/secrets.html) module as mentioned in [MSeifert's answer](https://stackoverflow.com/a/41464693/7851470):
{% endhint %}

```text
''.join(secrets.choice(string.ascii_uppercase + string.digits) for _ in range(N))
```

The module docs also discuss convenient ways to [generate secure tokens](https://docs.python.org/3/library/secrets.html#generating-tokens) and [best practices](https://docs.python.org/3/library/secrets.html#recipes-and-best-practices).

