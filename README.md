# yo-java

JVM client for [yo](https://github.com/tamnd/yo), an embedded multi-model database that lives in one `.yo` file. Records are the schema and there is no query language to learn. Java, Kotlin and Scala from one coordinate.

## Status

Nothing to use yet, and the artifact on Maven Central says so out loud.

`com.tamnd:yodb:0.0.0` resolves, compiles and throws:

```java
com.tamnd.yodb.Yo.open("app.yo");
// java.lang.UnsupportedOperationException: yo is not usable yet. This is a reserved
// placeholder at 0.0.0; see https://github.com/tamnd/yo
```

The version is held so the coordinate is held. `com.tamnd` verifies against `tamnd.com`, which is a registered domain, so nobody else can publish under it.

The engine is at `M0`. The record plane and the file format are `M1` and in progress, so there is nothing for this binding to sit on top of yet. Watch the [milestones](https://github.com/tamnd/yo/milestones).

Artifacts on Central are signed with the key whose fingerprint is `F737 055C 3ACD 3956 2FE2 6163 46D1 5643 1C21 8272`. It is published at [yo.tamnd.dev](https://yo.tamnd.dev) and on the public keyservers, and if any two of those three disagree you should trust none of them.

## Install

```xml
<dependency>
  <groupId>com.tamnd</groupId>
  <artifactId>yodb</artifactId>
  <version>0.0.0</version>
</dependency>
```

One coordinate for the whole JVM. Kotlin and Scala get their own idiom layers on top of it rather than their own coordinates to keep in sync.

## What this will be

```java
import com.tamnd.yo.*;

public class Quickstart {
    record User(@Yo.Id long id, String name, @Yo.Index(ord = true) double score) {}

    public static void main(String[] args) throws Exception {
        try (Db db = Db.open("app.yo")) {
            var users = db.docs(User.class, "users");
            users.put(new User(1, "Ada", 99.5));
            users.put(new User(2, "Grace", 97.0));

            var score = users.path(User::score);
            try (var rows = users.orderBy(score.desc()).limit(10).stream()) {
                rows.forEach(u -> System.out.println(u.name() + " " + u.score()));
            }
        }
    }
}
```

`users.path(User::score)` is a method reference to a record accessor, resolved once against the record components, so the field name is never a string in your code and `desc()` exists only on the orderable path type. That is the best the JVM offers, and it exists because records expose component names and types reflectively with no build step.

None of it works today. Note that the placeholder above sits in `com.tamnd.yodb` while the shipped API will be `com.tamnd.yo`; the placeholder is a name reservation and not a preview of the import.

## Planned support

| Item | Version |
|---|---|
| Baseline | Java 25 LTS, where FFM is final |
| JNI provider | Java 17 and 21, a second artifact with the same API |
| Build | Maven Central, one coordinate |

The JNI artifact exists because "we support Java" that turns out to mean "Java 25 only" is a promise a large share of the market cannot take, and because the JNI path is about forty lines of glue over the same C ABI. Its benchmark numbers are published separately and they are worse. The documentation says by how much rather than averaging the two.

From JDK 24 onward `--enable-native-access` is required or the JVM complains, and in a future release refuses. The binding declares the module so the flag can be scoped rather than opened to everything, and it detects the missing flag at `Db.open` time and throws an error naming the exact flag and where to put it. A warning you will ignore followed by a failure you cannot explain is the outcome that is being designed out.

## Design

The full JVM specification, shared with .NET because the two runtimes have the same shape of problem, is `dx/08` in the project specification.

## Licence

Apache 2.0 or MIT, at your option.
