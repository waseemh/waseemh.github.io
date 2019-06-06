---
layout: post
title: "Thread-safe Message Authentication Code Calculation"
comments: true
tags: java, crypto, concurrency
permalink: thread-safe-mac-calculation
---
In this short paper I would like to present a thread-safety issue when using MAC (Message Authentication Code) in Java crypto library.

### MAC and HMAC overview

MAC is a widely used mechanism for enhancing API security by signing request messages, to verify their integrity and authenticity by the receiver. Messages are signed using a secret key that is shared between the requester and receiver. As opposed to traditional API authentications with tokens, the secret key is never transmitted over the wire in each request. Instead, it is used to sign the request message. 
HMAC (Keyed-Hash Message Authentication Code) is a specific type of MAC where secret key is used along with a cryptographically secure hash function (typically SHA-256) to generate a hash for signing the message.

### MAC in Java

In Java, Mac class (javax.crypto.Mac) can create a message authentication code (MAC) from binary data and a secret key. Following snippet demonstrates this: 
{% highlight java %} 
//create MAC instance
Mac mac = Mac.getInstance("HmacSHA256");

//initialize MAC instance with secret key and hash algorithm
byte[] keyBytes   = new byte[]{0,1,2,3,4,5,6,7,8 ,9,10,11,12,13,14,15};
SecretKeySpec key = new SecretKeySpec(keyBytes, "HmacSHA256");
mac.init(key);

//calculate MAC of message
byte[] data  = "this is a message".getBytes("UTF-8");
byte[] macBytes = mac.doFinal(data);
{% endhighlight %}
 
### Concurrency Issue
 
However, problems arise when we calculate MAC values in parallel. Let’s take for example an HTTP service that communicates with a 3rd-party API which requires MAC-based authentication. When serving multiple concurrent requests, our HTTP service fails to calculate the correct MAC value, thus authentication with 3rd-party API fails.

This behaviour should make sense because:
1- Mac object is stateful because it retains internal state about the encryption process 
2- Most MAC algorithms are sequential (including HMAC). 

Since javax.crypto.Mac is not thread-safe, new MAC instance should be used for each calculation. For example:

{% highlight java %} 

{% endhighlight %}

Moreover, since Mac object is cloneable, we can clone existing Mac object instead of initiating it for each calculation:

{% highlight java %} 
    /**
     * javax.crypto.Mac is not thread-safe. MAC instance should be cloned for each calculation.
     */
    private Mac getMac() {
        try {
            return (Mac) macInstance.clone();
        } catch (CloneNotSupportedException e) {
            throw new IllegalStateException(e);
        }
    }
{% endhighlight %}
