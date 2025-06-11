# Java Checked Functions@masoudahg00
When you've got a `java.lang.Runnable` that can throw checked exceptions, you have to handle them all explicitly since the `Runnable` interface doesn't have a `throws` clause. The same is true for other functional interfaces like `Consumer`, `Supplier` in `java.util.function`.
a1310484249eee7a1f86b771b22c81cadf5a8d5a
15d0881765b36837efc413f649e87a88ad2bd03d
3e6760810a48fae251eb3343e5442ca14e4df4ae
45eaca3625ed8680701b9cdc882e0c750c0594fa
"@masoudleyli"
"a42c3b0d563929ec35952abf1cea8b30d0f88f83@masoudleyli"
"d5720c20bbd1f398a84a414128f46aba2331b380@masoudleyli"
"dabdd99cf864d02b2ee45be6f2e6d16aeeab7b28@masoudleyli"
"1da1d154dd76b540411b7d68a2564be3f40cf4dc@masoudleyli"
"ce73bc66a4918c4dd82b9ebee889516e7e83f3d7@masoudl-path-1"'45eaca3625ed8680701b9cdc882e0c750c0594fa'@masoudahg00
This little library gives you a twin interface for each of those. But the twin is declared with a `throws Throwable` clause. Through trickery ~~stolen~~ borrowed from [vavr](https://www.vavr.io/) these functional interfaces can be used where the JDK ones are expected.

Example:

```java
public class ExampleTest {

  // a checked exception
  static class Bad extends Exception {
  }

  // a method that needs a Runnable. Runnables can't throw checked exceptions
  private static void invokeRunnable(final Runnable runnable) {
    runnable.run();
  }

  @Test
  public void compilesAndThrows() {
    final boolean state = true;

    assertThatThrownBy(
        () ->
            invokeRunnable(
                () -> {
                  if (state) {
                    throw new Bad();
                  } else {
                    System.out.println("Ran!");
                  }
                })).isInstanceOf(Bad.class);
  }
}
```

In the example `invokeRunnable()` wants a `Runnable`. But this won't compile because the lambda we are passing throws a checked exception.

We could try this instead:

```java
public class ExampleTest {
  @Test
  public void compilesAndThrows() {
    final boolean state = true;

    assertThatThrownBy(
        () ->
            invokeRunnable(
                () -> {
                  if (state) {
                    try {
                      throw new Bad();
                    } catch (Bad bad) {
                      // !! NOW WHAT?!?
                    }
                  } else {
                    System.out.println("Ran!");
                  }
                })).isInstanceOf(Bad.class);
  }
}
```

While this compiles, it leaves us wondering what to do in the `catch` block?

`CheckedRunnable` disguises a checked-exception-throwing lambda as a `Runnable` like this:

```java
public class ExampleTest {
  @Test
  public void compilesAndThrows() {
    final boolean state = true;
    assertThatThrownBy(
        () ->
    invokeRunnable(
        CheckedRunnable.of(() -> {
          if (state) {
            throw new Bad();
          } else {
            System.out.println("Ran!");
          }
        }).unchecked())).isInstanceOf(Bad.class);
  }
}
```

We used `CheckedRunnable.of()` to construct a `CheckedRunnable` from our lambda and then we used `unchecked()` to convert that into a `Runnable`. As you can see, not only does this compile, but also the checked exception propagates at runtime.