---
layout: post
title: Elegant Business Logic
date: 2018-08-15 13:32:20 +0300
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: i-rest.jpg # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Java, OOP, Business Logic]
---

Imagine a simple method that contains some business logic. A method that depicts an use case
in a system. Let's say we have a case where we have to store some payloads to a user devices and send them to a messaging queue.
How would you model this case? I think it would probably contain
some procedural steps backed up with decoupled parts of functionality such as services and
repositories or some other objects.  The use case could pretty much look like this:
```java
public class ProceduralCase implements UseCase
{
    private static final String TOPIC = "payloads";

    private final PayloadAdapter adapter;

    private final Users users;

    private final Kafka kafka;

    @Override
    public Payloads storePayloads(UserSource userSource, String message, String topic)
    {
        User user = fetchOrCreateUser(userSource);
        Payloads payloads = adapter.convert(message);
        for (Device device: user.devices()) {
            device.storePayloads(payloads);
        }
        pushToKafka(topic, new CompressedPayloads(payloads));
        return payloads;
    }

    private User fetchOrCreateUser(UserSource userSource)
    {
        String id = userSource.id();
        User user;
        if (this.users.exists(id)) {
            user  = this.users.get(id);
        } else {
            user = this.users.create(userSource);
        }
        return user;
    }

    private void pushToKafka(String topic, Payloads payloads) {
        if (validTopic(topic)) {
            this.kafka.push(topic, payloads);
        } else {
            this.kafka.push(TOPIC, payloads);
        }
    }

    private boolean validTopic(String topic) {
        return topic != null && !topic.isEmpty();
    }
}
```
Maybe you have something else in your mind, but I think this pretty common design.
I code like this almost all the time. The code looks clean, it feels clean but is it really clean? After all the work
of decomposing a problem into smaller pieces we still end up writing procedures
that can potentially lead to a mess. You may say: "Well, this is perfectly fine, we eventually need to write
procedures somewhere. Java is not entirely OOP nor functional language.", and I agree.
We cannot get rid of procedural code, but we can isolate procedural pieces by moving design more towards OOP style.

Let's first examine what is wrong with the code above:
* **Readability**. The code is pretty readable but there is still a lot of information to digest.
  Information that involves `User`,`Payloads`, `Device`, `CompressedPayloads`, `topic`, fetching of users,
  storing of payloads and pushing it to kafka. Each time we look at this method our brain needs to process
  that much information. We also have to deal with additional noise such as `for` loops, `if` statements and
  variable declaration.
* **Testability**. This is not the code I would like to test. We need to write at least 5 tests to cover all the branches.
  There is also hardcoded dependency towards `CompressedPayloads` class which we cannot mock.
* **Code reuse**.
  Look at those three private methods. They are great for increasing readability by they do not make a favor
  in terms of code reuse. We cannot access them outside the class and there is a big chance that we would need
  their behaviour somewhere else in the code.
* **Maintainability**.




![I and My friends]({{site.baseurl}}/assets/img/we-in-rest.jpg)

>Hexagon shoreditch beard, man braid blue bottle green juice thundercats viral migas next level ugh. Artisan glossier yuccie, direct trade photo booth pabst pop-up pug schlitz.
