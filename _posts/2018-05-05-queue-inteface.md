---
layout:     post
title:      Java Queue Interface
summary:    Java Queue Interface exploration
date:       2018-05-05 16-48-50
categories: java
comments: true
---

The goal of this post is to explore the Java Queue Interface. It is basically an interface with the following methods:

```java
public interface Queue<E> extends Collection<E> {
    E element();
    boolean offer(E e);
    E peek();
    E poll();
    E remove();
}
```


* `E element()` -> retrieves the first element (whose priority is higher) from the queue __without removing it__. Throws `NosuchElementException` when empty.
* `boolean offer(E e)` -> adds an element to the queue. It's preferable than `add` when the queue has finite length.
* `E peek()` -> Same as `element()`, but return null when the Queue is empty.
* `E poll()` -> Same as `element()` and `peek()` but removing the object.
* `E remove()` -> Same as `poll()` but will throw `NoSuchElementException` when the queue is empty.


In this example, we are going to simulate a store the waiting line of which is a priority queue where people are served 
according to their age. The youger a person is, the sooner he/she will be served.

The `Person` class is as follows:

```java
public class Person implements Comparable<Person> {

    private final String name;
    private final int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    @Override
    public int compareTo(Person o) {
        return Integer.compare(this.getAge(), o.getAge());
    }
}
```

In order to tell the `PriorityQueue` how the priority is stablished, we must implement 
the `Comparable::compareTo` method. When we use this method, we are ensuring that the
age is what we are taking into account when comparing `Person` objects.

The `Store` class is as follows:

```java
public class Store {

    public static void main(String[] args) {

        final Queue<Person> clients = new PriorityQueue<>(5);

        final Person person1 = new Person("Ignasi", 25);
        final Person person2 = new Person("David", 27);
        final Person person3 = new Person("Pere", 31);
        final Person person4 = new Person("Jorge", 27);
        final Person person5 = new Person("Baby", 2);

        clients.offer(person1);
        clients.offer(person2);
        clients.offer(person3);
        clients.offer(person4);
        clients.offer(person5);

        for (int i = 0; i < 5; i++) {
            final Person person = clients.poll();
            System.out.println(String.format("Serving %s with age %s", person.getName(), person.getAge()));
        }
    }
}
```

Which produces the output:

```text
Serving Baby with age 2
Serving Ignasi with age 25
Serving David with age 27
Serving Jorge with age 27
Serving Pere with age 31
```