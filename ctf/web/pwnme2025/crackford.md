# Crackford



{% hint style="success" %}
### Challenge

### Description

This a website without a lot of fonctionnalites. You can only create and connect to your account.

You need to take over admin account

### Author

* [Eteck](https://x.com/Eteckq)

### Difficulty

* Medium
{% endhint %}

We begin with a simple login page where we can create an account. After creating an account, we log in and discover an email that seems to be associated with the admin.

<figure><img src="../../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Upon further inspection, we notice an unusual GET request being made. This link turns out to be a password reset link for our user.

<figure><img src="../../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

At this point, we hypothesize that if we can figure out how this link is generated, we might be able to create a custom link that would allow us to reset the admin's password. I attempted to use tools like CrackStation, but none of them seemed to work. Instead, I created several user accounts with different emails to observe the variations in the reset links generated for each email. Here are some examples of the links:

* test1@test.com -> `0rsxg4br1b9gk58ufzrw68I5g38d32cqk4h39rjA1nk3g`
* t@t.com -> `0rAh1I7dn4wxymjwgn6fAv90jvcsAq9u1g`
* testtest@testtest.com -> `0rsxg4dumvzx1qdumvzx14df0n9c5y8pnv6dcnrupr1f07sn1uq3gvcg`
* abc@abc.com -> `mfrggqdbmjrs5y8pnv6dcnrvpr1f07sn1uq3gvcg`
* abcdefghijklmnopqrstuvwxyx@test.com -> `mfrggzdfmz7wq9Iknnwg987p0byx358u0v8h06dzpbAh1zI70qxgg88npqy7mn75kbIu57kf3bbv1rc`
* abcd1234@test.com -> `mfrggzbrg1z71qdumvzx1I7dn4wxymjwg46fAv90jvcsAq9u1g`

The key takeaways from this are as follows:

1. The length of the encoded value appears to correlate with the length of the email address. The longer the email, the longer the encoding of the `h` parameter, which suggests a direct relationship between the two.
2. Additionally, every email that starts with "abc" produces an encoding that begins with `mfrgg`. This could imply the existence of a hidden alphabet or encoding scheme.

In order to understand the pattern better, I created a payload containing all the characters, numbers, and symbols from the alphabet:\
[https://capitalizemytitle.com/copy-paste-alphabets/](https://capitalizemytitle.com/copy-paste-alphabets/).

The result of this payload for an email address like `abcdefghijklmnopqrstuvwxyz_ABCDEFGHIJKLMNOPQRSTUVWXYZ_0123456789@gmail.com` is as follows:

* Resulting encoding: `mfrggzdfmz7wq9Iknnwg987p0byx358u0v8h06dzpbAh1zI70qxgg88npqy7mn75kbIu57kf3bbv1rc`

***

