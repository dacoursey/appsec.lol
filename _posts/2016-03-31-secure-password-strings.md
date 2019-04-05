---
layout: post
title: "Secure Password Strings in Java and C#"
author:
modified:
tags: [Development,.NET,Java,AppSec]
---

For the second time in a few months I had a conversation with friends on this Fortify finding - Privacy Violation: Heap Inspection.  

The description reads:

> "Sensitive data (such as passwords, social security numbers, credit card numbers, etc.) stored in memory can be leaked if it is stored in a managed String object."

The threat here is that the string data will remain in memory long enough to be retrieved by an attacker. This is exactly why Heartbleed (TM) was such a big problem--strings in memory could be accessed long after they were no longer being used. If you ran it a bunch of times and were lucky, the exploit would give you passwords or private keys.

## Why is this happening?

When you don't spend much time thinking about how memory works, you may assume that after a password is used it no longer exists in memory. Sadly, this is not true, especially when we talk about a memory-managed language like Java or C#. There are a few things going on here that need to be discussed. We must go deeper!

For starters, we are stuck working with the language's Garbage Collector (GC). The GC is the mechanism that frees memory after it is no longer needed. I'm not going to rehash what has been written about a billion times, just understand one point about the GC: it's like the guy sleeping on your couch. You can ask him to go get a job, and he will tell you he put in all sorts of applications, but you can never be certain that anything is actually happening.

Next, in an effort to conserve memory space, the language runtime makes available this interesting thing called the String Intern Pool. The String Intern Pool is a special storage area that keeps a map of strings your program is using and provides references to a single string in memory. At any time, exactly one copy of any string can exist in the String Pool. When your code has two String objects that are identical, they are really referencing the same location in memory. This happens automatically when a new String is created in the program.

But what happens if one of those interned String objects has its value changed?

HA! Trick question. That string cannot be modified in memory because strings are 'immutable' and can never be changed. When you do something like: `s = s.concat("foo");`

you are actually allocating new memory for storing your data. So we are left with a lazy garbage collector and a storage area that hoards strings that couldn't be modified if we wanted to.

## What to do, what to do?

There is actually a pretty simple fix for secure data handling: instead of `String`, use `Char[]` for any sensitive data. When all operations are finished with `Char[]`, it can be overwritten with zero's or junk text to clear it from memory. There are some academic "what if" scenarios in there, but the risk of exposing data through heap inspection is drastically reduced.

Your implementation will vary a little, but securely handling a password should look something like this in Java:

```java
//Handle the secure password data.

public void actionPerformed(ActionEvent e) {
    String cmd = e.getActionCommand();

    if (OK.equals(cmd)) { //Process the password.
        string username = usernameField.getText();
        char[] password = passwordField.getPassword();
        if (isPasswordCorrect(username, password)) {
            doLoginStuff(); //Build session object and redirect browser.
        } else {
            JOptionPane.showMessageDialog(controllingFrame,
                "Login failed.",
                "Error Message",
                JOptionPane.ERROR_MESSAGE);
        }

        //Zero out the possible password, for security.
        Arrays.fill(password, '0');

        passwordField.selectAll();
        resetFocus();
    } else ...//do other stuff...
}


private static boolean isPasswordCorrect(String username, char[] password) {
    boolean isCorrect = false;

    byte[] salt = getSaltFromDB(username).getBytes();

    byte[] passwordHash = secureHash(password, salt);

    isCorrect = validatePassword(passwordHash);

    return isCorrect;
}
```


## What about C#?

In .NET, this process is handled just a bit differently. As with Java, String objects are immutable, internable, and magically garbage-collected. However, the .NET Framework provides a new type to ease the burden in this scenario,  `System.Security.SecureString`. This new type has a few notable enhancements over plain ol' `System.String`. By default, `SecureString` is encrypted in memory (really it's obfuscation), is pinned in memory, and can be marked read-only.

```c#
protected void LogIn(object sender, EventArgs e)
{
    if (IsValid)
    {
        SecureString password = new SecureString();

        foreach (char c in txtPassword.Text.ToCharArray())
            password.AppendChar(c);

        //Do more stuff.
    }
}
```


## Integration

I know what you're thinking; you are going to work early tomorrow to convert every `String` to `Char[]`. Not so fast! There is one huge drawback with this method - interoperability. Using `Char[]` may not always work with your existing integrations that expect `String`. If you find that you must interface with code or systems that do not handle the data as securely as you do, that's okay. Do the best you can for your own projects, reducing the attack surface is a worthwhile effort. Converting your `Char[]` to `String` just before interfacing with another system is better than having sensitive strings littered throughout your code.

In the ideal scenario, the password is salted and hashed on the client prior to storage. However, not everybody does this; legacy API's, ignorance, and evil voices all contribute to that ideal scenario being not as common as we would like. I saw one company that had a great code library for these kinds of operations, BUT NOBODY UPDATED IT! So that's great, now all of your applications are standardized on the same weak design. So you need to do a little investigation to make sure your new method works in the environment. Go be the person that updates your out-of-date standards!




## Implementation

It's fairly safe to say that bcrypt is the strongest hash function, although Argon2 may soon be the champion. Both Java and C# have third-party libs that support bcrypt - jbcrypt and bcrypt.net. They are both good, but for the purposes of this post I am going to show PBKDF2, because it is a built-in part of both languages. 

**Java**

```java
private static void getHash(char[] chars) 
{
    int iterations = 1000;
    byte[] salt = getSalt().getBytes();
     
    PBEKeySpec spec = new PBEKeySpec(chars, salt, iterations, 64 * 8);
    SecretKeyFactory skf = SecretKeyFactory.getInstance("PBKDF2WithHmacSHA1");
    byte[] hash = skf.generateSecret(spec).getEncoded();
    return iterations + ":" + toHex(salt) + ":" + toHex(hash);
}
```

**C#**

```c#
private static byte[] getHash(byte[] password)
{
    int iterations = 1000;
    byte[] salt = getSalt(16);

    Rfc2898DeriveBytes pbkdf2 = new Rfc2898DeriveBytes(password, temp, iterations);
    return pbkdf2.GetBytes(16);
}
```


You are going to have a more difficult time storing a char or byte array in your credential store. Modern relational DBs will be able to store this as a blob. Not the greatest solution, but it gets the job done. I'm not so sure this will work if you are authenticating against an LDAP directory or a custom AuthN provider. I would love to hear from someone that has made this switch and lived to tell about it.

So, while not perfect, this is a safer way to store any secure data, not just passwords. If your application operates on anything that should be protected, such as SSN's, credit card numbers, or sensitive proprietary data, think about protecting it at every level of the data flow. You never know when the next exploit will come along with a logo, website, and theme song that can pass right through one or more of your defenses.

Last thing: if you need a little more explanation on the overall process of password handling, go check out this great post - [How to Safely Store Your Users' Passwords in 2016](https://paragonie.com/blog/2016/02/how-safely-store-password-in-2016 target=_blank).


## References:

* [https://paragonie.com/blog/2016/02/how-safely-store-password-in-2016](https://paragonie.com/blog/2016/02/how-safely-store-password-in-2016)
* [http://docs.oracle.com/javase/tutorial/uiswing/components/passwordfield.html](http://docs.oracle.com/javase/tutorial/uiswing/components/passwordfield.html)
* [https://msdn.microsoft.com/en-us/library/system.security.securestring(v=vs.110).aspx](https://msdn.microsoft.com/en-us/library/system.security.securestring(v=vs.110).aspx)
* [http://forums.asp.net/t/2070598.aspx?Login+Page+Visual+Studio+2013+NET+4+5](http://forums.asp.net/t/2070598.aspx?Login+Page+Visual+Studio+2013+NET+4+5)
* [http://www.javaxp.com/2011/08/java-strings-are-immutable-objects.html](http://www.javaxp.com/2011/08/java-strings-are-immutable-objects.html)
* [http://java-performance.info/string-intern-in-java-6-7-8/](http://java-performance.info/string-intern-in-java-6-7-8/)
* [https://msdn.microsoft.com/en-us/library/system.string.intern(v=vs.110).aspx](https://msdn.microsoft.com/en-us/library/system.string.intern(v=vs.110).aspx)
* [http://wideskills.com/java-tutorial/java-jtextfield-class-example](http://wideskills.com/java-tutorial/java-jtextfield-class-example)
* [https://msdn.microsoft.com/en-us/library/07b9wyhy.aspx](https://msdn.microsoft.com/en-us/library/07b9wyhy.aspx)
* [http://stackoverflow.com/questions/141203/when-would-i-need-a-securestring-in-net](http://stackoverflow.com/questions/141203/when-would-i-need-a-securestring-in-net)
* [http://stackoverflow.com/questions/18142745/how-do-i-generate-a-salt-in-java-for-salted-hash](http://stackoverflow.com/questions/18142745/how-do-i-generate-a-salt-in-java-for-salted-hash)
* [http://www.hpenterprisesecurity.com/vulncat/en/vulncat/dotnet/privacy_violation_heap_inspection.html](http://www.hpenterprisesecurity.com/vulncat/en/vulncat/dotnet/privacy_violation_heap_inspection.html)

