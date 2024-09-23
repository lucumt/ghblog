---
title: "[译]从JUnit 4 迁移到JUnit 5 - 权威指南"
date: 2023-06-21T22:21:43+08:00
lastmod: 2023-06-21T22:21:43+08:00
draft: false
keywords: ["junit5"]
description: "JUnit5翻译专题，主要是从JUnit 4 迁移到JUnit 5的简要说明"
tags: ["junit5","java","mockito","junit"]
categories: ["翻译","JUnit5翻译"]
author: "Rosen Lu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
toc: true
autoCollapseToc: true
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false
centerImage: true
borderImage: false

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""

mermaidDiagrams: 
  enable: false
  options: ""

highchartsDiagrams: 
  enable: false
  options: ""
---

本文翻译自[**Migrating From JUnit 4 to JUnit 5: A Definitive Guide**](https://www.arhohuttunen.com/junit-5-migration/)。

<!--more-->

![JUnit 4 to JUnit 5](/blog_img/translate/junit5/junit-5-migration/junit-4-junit-5.webp "JUnit 4 to JUnit 5") 

在教程中，我们将了解从`JUnit 4`迁移到`JUnit 5`所需的步骤，我们将了解如何将现有测试与新版本一起运行，以及必须进行哪些更改才能迁移代码。

本文是[**JUnit 5 教程**](https://www.arhohuttunen.com/junit-5-tutorial/)的一部分。

## 概览

`JUnit 5`不同于之前的版本，它采用模块化设计，这种新架构的关键点是将编写测试、扩展和工具之间的关注点分开。

`JUnit`被拆分为3个不同的子工程：

* **JUnit Platform**是基础，提供构建插件和用于编写测试引擎的API
* **JUnit Jupiter**是在`JUnit 5`中用于编写测试和扩展的新式API
* **JUnit Vintage**允许我们在`JUnit 5`中运行`JUnit 4`编写的测试

以下是`JUnit 5`相对于`JUnit 4`的一些优点：

`JUnit 4`最大的一个缺陷是不支持运行多个Runners(例如不能同时使用`SpringJUnit4ClassRunner`和`Parameterized`)，而在`JUnit 5`中可通过注册多个扩展实现。

此外，`JUnit 5`利用了`Java 8`中的一些特性如lambda用于惰性评估，`JUnit 4`从没使用超过`Java 7`的版本因而错失了`Java 8`的特性。

同样，`JUnit 4`在参数化测试方面有缺陷并且缺乏嵌套测试，由此激发了第三方开发者为这些场景开发专门的Runner。

`JUnit 5`给参数化测试提供了更好的支持，在对嵌套测试提供原生支持的同时也增加了一些新功能。

## 关键迁移步骤

`JUnit`在`JUnit Vintage`测试引擎的帮助下提供了渐进的迁移路径，我们可使用`JUnit Vintage`测试引擎在`JUnit 5`中运行`JUnit 4`相关的测试。

所有`JUnit 4`相关的类都位于`org.junit`包下，所有`JUnit 5`相关的类都位于`org.junit.jupiter`包下，如果`JUnit 4`和`JUnit 5`在当前类路径下同时存在，也不会产生冲突。

因此，我们可以将之前实现的`JUnit 4`测试与`JUnit 5`测试保留在一起，直到完成迁移，同时可逐步规划迁移。

下属表格总结了从`JUnit 4`迁移到`JUnit 5`中的一些关键步骤：

| 步骤                    | 解释                                                         |
| ----------------------- | ------------------------------------------------------------ |
| 替换依赖                | `JUnit 4`使用单个依赖,`JUnit 5`对迁移支持和`JUnit Vintage`引擎有额外的依赖项 |
| 替换注解                | `JUnit 5`中的一些注解与`JUnit 4`中的相同，一些新注解替换了旧的，并且功能略有不同。 |
| 替换测试类和方法        | 断言和假设已被移动到新的类中，方法参数的顺序在某些场景下所有不同。 |
| 替换runners、规则、扩展 | `JUnit 5`用一个扩展模型替换了runners和规则，此步骤相较于其它步骤更耗时。 |

接下来我们将更深入地研究每个步骤。

## 依赖

先看看需要做什么才能在新平台上运行现有测试，为了同时运行`JUnit 4`和`JUnit 5`测试，需要如下操作：

* `JUnit Jupiter`用于编写和运行`JUnit 5`测试
* `JUnit Vintage`测试引擎用于运行`JUnit 4`测试

除此之外，要使用`Maven`运行测试，我们还需要Surefire插件，需要在`pom.xml`中添加所需的全部依赖：

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>2.22.2</version>
</plugin>

<dependencies>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>5.8.0</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.junit.vintage</groupId>
        <artifactId>junit-vintage-engine</artifactId>
        <version>5.8.0</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

相似的，要使用`Gradle`运行测试的话，同样需要在测试中开启`JUnit Platform`支持，同时在`build.gradle`中添加所需的全部依赖：

```groovy
test {
    useJUnitPlatform()
}

dependencies {
    testImplementation('org.junit.jupiter:junit-jupiter:5.8.0')
    testRuntime('org.junit.vintage:junit-vintage-engine:5.8.0')
}
```

## 注解

注解位于`org.junit.jupiter.api`包中而不是`org.junit`。

大部分的注解名称也不相同：

| JUnit 4        | JUnit 5       |
| :------------- | :------------ |
| `@Test`        | `@Test`       |
| `@Before`      | `@BeforeEach` |
| `@After`       | `@AfterEach`  |
| `@BeforeClass` | `@BeforeAll`  |
| `@AfterClass`  | `@AfterAll`   |
| `@Ignore`      | `@Disable`    |
| `@Category`    | `@Tag`        |

在大多数情况下，我们可以查找并替换相关的包名与类名。

但是，`@Test`注解不再具有`expected`和`timeout`属性。

### 异常

我们无法在`@Test`注解中使用`expected`属性。

可用`JUnit 5`中的`assertThrows()`方法来替换`JUnit 4`中的`expected`属性：

```java
public class JUnit4ExceptionTest {
    @Test(expected = IllegalArgumentException.class)
    public void shouldThrowAnException() {
        throw new IllegalArgumentException();
    }
}

class JUnit5ExceptionTest {
    @Test
    void shouldThrowAnException() {
        Assertions.assertThrows(IllegalArgumentException.class, () -> {
            throw new IllegalArgumentException();
        });
    }
}
```

### 超时

同样也无法在`@Test`注解中使用`timeout`属性。

可在`JUnit 5`中使用`assertTimeout()`方法来替换`JUnit 4`中的`timeout`属性。

```java
public class JUnit4TimeoutTest {
    @Test(timeout = 1)
    public void shouldTimeout() throws InterruptedException {
        Thread.sleep(5);
    }
}

class JUnit5TimeoutTest {
    @Test
    void shouldTimeout() {
        Assertions.assertTimeout(Duration.ofMillis(1), () -> Thread.sleep(5));
    }
}
```

## 测试类和方法

如前所述，断言和假设被移动到新的类中，同样的，方法参数的顺序在某些场景下所有不同。

下述表格总结了`JUnit 4`和`JUnit 5`中测试类和测试方法的一些主要的不同点：

|                    | JUnit 4               | JUnit 5                      |
| ------------------ | --------------------- | ---------------------------- |
| **测试包**         | `org.junit`           | `org.junit.jupiter.api`      |
| **断言类**         | `Assert`              | `Assertions`                 |
|                    | `assertThat()`        | `MatcherAssert.assertThat()` |
| **可选的断言消息** | 第一个方法参数        | 最后一个方法参数             |
| **假设类**         | `Assume`              | `Assumptions`                |
|                    | `assumeNotNull()`     | 移除                         |
|                    | `assumeNoException()` | 移除                         |

值得注意的是，我们在`JUnit 4`中自己编写的测试类和方法必须是`public`修饰的。

`JUnit 5`移除了上述限制，测试方法和测试类可以在包中私有，我们可在提供的示例中看到这种差异。

接下来我们详细看看测试类和测试方法中的变化。

### 断言

断言方法位于`org.junit.jupiter.api.Assertions`类中，而不是`org.junit.Assert`类中。

在大多数场景下，我们可直接查找并替换包名。

然而，如果我们给断言提供了自定义消息，在编译时会报错，可选的断言消息现在是最后一个参数，这种参数顺序看起来更自然：

```java
public class JUnit4AssertionTest {
    @Test
    public void shouldFailWithMessage() {
        Assert.assertEquals("numbers " + 1 + " and " + 2 + " are not equal", 1, 2);
    }
}

class JUnit5AssertionTest {
    @Test
    void shouldFailWithMessage() {
        Assertions.assertEquals(1, 2, () -> "numbers " + 1 + " and " + 2 + " are not equal");
    }
}
```

也可以像示例中那样惰性评估断言消息，从而避免构造不必要的复杂消息。

{{% admonition danger "注意" false %}}

当断言一个`String`对象并有自定义消息时，由于所有的参数类型都是`String`我们不会看见编译错误，然而我们可以很容易地发现这些情况，因为当运行它们时测试将会失败。

{{% /admonition %}}

此外，有些遗留的测试代码会基于`JUnit 4`中的`Assert.assertThat()`来使用`Hamcrest`断言，`JUnit 5`没有像`JUnit 4`那样提供`Assert.assertThat()`，相反，我们必须从`Hamcrest`的`MatcherAssert`中导入相关方法：

```java
public class JUnit4HamcrestTest {
    @Test
    public void numbersNotEqual() {
        Assert.assertThat("numbers 1 and 2 are not equal", 1, is(not(equalTo(2))));
    }
}

class JUnit5HamcrestTest {
    @Test
    void numbersNotEqual() {
        MatcherAssert.assertThat("numbers 1 and 2 are not equal", 1, is(not(equalTo(2))));
    }
}
```

### 假设

假设方法位于`org.junit.jupiter.Assumptions`类中，而不是`org.junit.Assume`类中。

这些方法有类似的改动，假设消息现在是最后一个参数。

```java
@Test
public class JUnit4AssumptionTest {
    public void shouldOnlyRunInDevelopmentEnvironment() {
        Assume.assumeTrue("Aborting: not on developer workstation",
                "DEV".equals(System.getenv("ENV")));
    }
}

class JUnit5AssumptionTest {
    @Test
    void shouldOnlyRunInDevelopmentEnvironment() {
        Assumptions.assumeTrue("DEV".equals(System.getenv("ENV")),
                () -> "Aborting: not on developer workstation");
    }
}
```

同样需要注意的是，现在不再有`Assume.assumeNotNUll()`和`Assume.assumeNoException()` 。

## 分类

`JUnit 4`中的`@Category`注解在`JUnit 5`中被`@Tag`注解替换，此外我们不再使用标记接口，而是向注解传递字符串参数。

在`JUnit 4`可通过标记接口来使用分类：

```java
public interface IntegrationTest {}

@Category(IntegrationTest.class)
public class JUnit4CategoryTest {}
```

之后可通过在`pom.xml`中配置基于标签过滤测试。

```xml
<plugin>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>2.22.2</version>
    <configuration>
        <groups>com.example.AcceptanceTest</groups>
        <excludedGroups>com.example.IntegrationTest</excludedGroups>
    </configuration>
</plugin>
```

或者，在使用`Gradle`时在`build.gradle`中进行配置：

```groovy
test {
    useJUnit {
        includeCategories 'com.example.AcceptanceTest'
        excludeCategories 'com.example.IntegrationTest'
    }
}
```

但是在`JUnit 5`中可直接使用标签来实现：

```java
@Tag("integration")
class JUnit5TagTest {}
```

`Maven`中的`pom.xml`配置也更简单：

```xml
<plugin>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>2.22.2</version>
    <configuration>
        <groups>acceptance</groups>
        <excludedGroups>integration</excludedGroups>
    </configuration>
</plugin>
```

相似的，`build.gradle`中的配置也更简洁：

```groovy
test {
    useJUnitPlatform {
        includeTags 'acceptance'
        excludeTags 'integration'
    }
}
```

## Runners

`JUnit 4`中的`@RunWith`注解在`JUnit 5`中不存在，可通过使用`org.junit.jupiter.api.extension`包和`@ExtendWith`注解中的新扩展模型实现相似的功能。

### Spring Runner

`JUnit 4`中使用的一个流行的runner是Spring test runner，在`JUnit 5`中需要将其替换为一个`Spring`扩展。

如果我们使用`Spring 5`，该扩展会与`Spring Test`绑定在一起。

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = SpringTestConfiguration.class)
public class JUnit4SpringTest {

}

@ExtendWith(SpringExtension.class)
@ContextConfiguration(classes = SpringTestConfiguration.class)
class JUnit5SpringTest {

}
```

然而在使用`Spring 4`时该扩展没有与`SpringExtension`绑定，我们仍然可以使用它，但需要JitPack仓库中的额外依赖。

要在`JUnit 4`中使用`SpringExtension`注解我们需要在`pom.xml`中添加如下依赖：

```xml
<repositories>
    <repository>
        <id>jitpack.io</id>
        <url>https://jitpack.io</url>
    </repository>
</repositories>

<dependencies>
    <dependency>
        <groupId>com.github.sbrannen</groupId>
        <artifactId>spring-test-junit5</artifactId>
        <version>1.5.0</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

类似的，在使用`Gradle`时也需要在`build.gradle`中添加相应依赖：

```groovy
repositories {
    mavenCentral()
    maven { url 'https://jitpack.io' }
}

dependencies {
    testImplementation('com.github.sbrannen:spring-test-junit5:1.5.0')
}
```

### Mockito Runner

`JUnit 4`中另一个流行的runner是Mockito runner，在使用`JUnit 5`时需要将其替换为`JUnit 5`中的`Mockito`扩展。

要使用`Mockito`扩展，需要在`pom.xml`中添加相关的依赖：

```xml
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-junit-jupiter</artifactId>
    <version>3.6.28</version>
    <scope>test</scope>
</dependency>
```

类似的，使用`Gradle`时需要在`build.gradle`中添加依赖：

```groovy
dependencies {
    testImplementation('org.mockito:mockito-junit-jupiter:3.12.4')
}
```

现在，可将`MockitoJUnitRunner`简单的替换为`MockitoExtension`：

```java
@RunWith(MockitoJUnitRunner.class)
public class JUnit4MockitoTest {

    @InjectMocks
    private Example example;

    @Mock
    private Dependency dependency;

    @Test
    public void shouldInjectMocks() {
        example.doSomething();
        verify(dependency).doSomethingElse();
    }
}

@ExtendWith(MockitoExtension.class)
class JUnit5MockitoTest {

    @InjectMocks
    private Example example;

    @Mock
    private Dependency dependency;

    @Test
    void shouldInjectMocks() {
        example.doSomething();
        verify(dependency).doSomethingElse();
    }
}
```

## 规则

`JUnit 4`中的`@Rule`和`@ClassRule`注解在`JUnit 5`中不存在，可通过使用`org.junit.jupiter.api.extension`包和`@ExtendWith`注解中的新扩展模型实现相似的功能。

然而，要实现一个平滑的迁移，`junit-jupiter-migrationsupport`模块提供了`JUnit 4`中的规则子集和子类的支持:

* `ExternalResource`(例如`TemporaryFolder`)
* `Verifier`(例如`ErrorCollector`)
* `ExpectedException`

通过使用`org.junit.jupiter.migrationsupport.rules`包中的类级别注释`@EnableRuleMigrationSupport`，可让使用这些规则的已有代码保持不变。

要在`Maven`中开启该支持需要添加相关依赖：

```xml
<dependencies>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-migrationsupport</artifactId>
        <version>5.8.0</version>
    </dependency>
</dependencies>
```

同样的，若使用`Gradle`需要在`build.gradle`中添加相关依赖：

```groovy
dependencies {
    testImplementation('org.junit.jupiter:junit-jupiter-migrationsupport:5.8.0')
}
```

### 预期异常

在`JUnit 4`中使用`@Test(expected = SomeException.class)`注解不允许我们检查异常的详细信息，若需要检查，则要使用`ExpectedException`规则。

`JUnit 5`迁移支持允许我们通过在测试中添加`@EnableRuleMigrationSupport`来仍然使用该规则。

```java
@EnableRuleMigrationSupport
class JUnit5ExpectedExceptionTest {

    @Rule
    public ExpectedException thrown = ExpectedException.none();

    @Test
    void catchThrownExceptionAndMessage() {
        thrown.expect(IllegalArgumentException.class);
        thrown.expectMessage("Wrong argument");

        throw new IllegalArgumentException("Wrong argument!");
    }
}
```

由于我们将所有内容都集中在一起，结果更具有可读性。

### 临时目录

在`JUnit 4`中可通过使用`TemporaryFolder`规则来创建和清除一个临时目录，同样的，`JUnit 5`迁移支持允许我们添加`@EnableRuleMigrationSupport`来继续使用该功能。

```java
@EnableRuleMigrationSupport
class JUnit5TemporaryFolderTest {

    @Rule
    public TemporaryFolder temporaryFolder = new TemporaryFolder();

    @Test
    void shouldCreateNewFile() throws IOException {
        File textFile = temporaryFolder.newFile("test.txt");
        Assertions.assertNotNull(textFile);
    }
}
```

要完全摆脱`JUnit 4`中的规则，我们需要将其替换为`TempDirectory`扩展，可通过给一个`Path`或`File`属性添加`@TempDir`注解来使用此扩展：

```java
class JUnit5TemporaryFolderTest {

    @TempDir
    Path temporaryDirectory;

    @Test
    public void shouldCreateNewFile() {
        Path textFile = temporaryDirectory.resolve("test.txt");
        Assertions.assertNotNull(textFile);
    }
}
```

此扩展与前面的规则类似，一个不同点是我们也可以将其添加到方法参数中：

```java
@Test
public void shouldCreateNewFile(@TempDir Path anotherDirectory) {
    Path textFile = anotherDirectory.resolve("test.txt");
    Assertions.assertNotNull(textFile);
}
```

### 自定义规则

迁移`JUnit 4`中的规则时需要将其重写为`JUnit 5`中的扩展。

可通过实现`BeforeEachCallback`和`AfterEachCallback`接口来重现引用了`@Rule`注解的业务逻辑。

例如，我们有一个实现性能日志的`JUnit 4`规则：

```java
public class JUnit4PerformanceLoggerTest {

    @Rule
    public PerformanceLoggerRule logger = new PerformanceLoggerRule();
}

public class PerformanceLoggerRule implements TestRule {

    @Override
    public Statement apply(Statement base, Description description) {
        return new Statement() {
            @Override
            public void evaluate() throws Throwable {
                // Store launch time
                base.evaluate();
                // Store elapsed time
            }
        };
    }
}
```

反过来，我们可以编写与`JUnit 5`扩展相同的规则：

```java
@ExtendWith(PerformanceLoggerExtension.class)
public class JUnit5PerformanceLoggerTest {

}

public class PerformanceLoggerExtension
        implements BeforeEachCallback, AfterEachCallback {

    @Override
    public void beforeEach(ExtensionContext context) throws Exception {
        // Store launch time
    }

    @Override
    public void afterEach(ExtensionContext context) throws Exception {
        // Store elapsed time
    }
}
```

### 自定义类规则

类似的，可通过实现`BeforeEachCallback`和`AfterEachCallback`接口来重现引用了`@ClassRule`注解的业务逻辑。

在某些场景中，我们可能在`JUnit 4`中将类规则编写为匿名内部类，如下述例子，有一个服务器资源我们希望可以很轻松的在不同的测试中使用设置。

```java
public class JUnit4ServerBaseTest {
    static Server server = new Server(9000);

    @ClassRule
    public static ExternalResource resource = new ExternalResource() {
        @Override
        protected void before() throws Throwable {
            server.start();
        }

        @Override
        protected void after() {
            server.stop();
        }
    };
}

public class JUnit4ServerInheritedTest extends JUnit4ServerBaseTest {
    @Test
    public void serverIsRunning() {
        Assert.assertTrue(server.isRunning());
    }
}
```

可将该规则编写为`JUnit 5`扩展，不幸的是，如果在该扩展中使用`@ExtendWith`注解，我们无法访问该扩展提供的资源，但可通过使用`@RegisterExtension`来替换：

```java
public class ServerExtension implements BeforeAllCallback, AfterAllCallback {
    private Server server = new Server(9000);

    public Server getServer() {
        return server;
    }

    @Override
    public void beforeAll(ExtensionContext context) throws Exception {
        server.start();
    }

    @Override
    public void afterAll(ExtensionContext context) throws Exception {
        server.stop();
    }
}

class JUnit5ServerTest {
    @RegisterExtension
    static ServerExtension extension = new ServerExtension();

    @Test
    void serverIsRunning() {
        Assertions.assertTrue(extension.getServer().isRunning());
    }
}
```

## 参数化测试

在`JUnit 4`中编写参数化测试时需要使用`Parameterized` runner，此外，我们需要通过一个添加了`@Parameterized.Parameters`注解的方法来传递参数化数据：

```java
@RunWith(Parameterized.class)
public class JUnit4ParameterizedTest {
    @Parameterized.Parameters
    public static Collection<Object[]> data() {
        return Arrays.asList(new Object[][] {
                { 1, 1 }, { 2, 1 }, { 3, 2 }, { 4, 3 }, { 5, 5 }, { 6, 8 }
        });
    }

    private int input;
    private int expected;

    public JUnit4ParameterizedTest(int input, int expected) {
        this.input = input;
        this.expected = expected;
    }

    @Test
    public void fibonacciSequence() {
        assertEquals(expected, Fibonacci.compute(input));
    }
}
```

编写`JUnit 4`参数化测试有很多缺点，并且像JUnitParams这样的社区runner将自己描述为并不糟糕的参数化测试。

不幸的是没有`JUnit 4`参数化runner的直接替代品，相反的，`JUnit 5`中提供了一个`@ParameterizedTest`注解，可以为数据提供各种数据源注解，其中最接近`JUnit 4`的是`@MethodSource`注释：

```java
class JUnit5ParameterizedTest {
    private static Stream<Arguments> data() {
        return Stream.of(
                Arguments.of(1, 1),
                Arguments.of(2, 1),
                Arguments.of(3, 2),
                Arguments.of(4, 3),
                Arguments.of(5, 5),
                Arguments.of(6, 8)
        );
    }

    @ParameterizedTest
    @MethodSource("data")
    void fibonacciSequence(int input, int expected) {
        assertEquals(expected, Fibonacci.compute(input));
    }
}
```

{{% admonition  note "注意" false %}}

在`JUnit 5`中与`JUnit 4`参数化测试最接近的是使用`@ParameterizedTest`和`@MethodSource`数据源，但`JUnit 5`中的参数化测试有多项改进，可在[**JUnit 5参数化测试**](/post/translate/junit5/junit-5-parameterized-tests/)中阅读有关改进的更多信息。

{{% /admonition %}}

## 总结

从`JUnit 4`迁移到`JUnit 5`需要一些工作，具体取决于现有测试的编写方式。

* 我们可以将`JUnit 4`测试与`JUnit 5`测试一起运行，以允许逐步迁移。
* 在很多情况下，我们只需查找并替换包名和类名。
* 我们可能必须将自定义运行程序和规则转换为扩展。
* 要转换参数化测试，我们可能需要做一些返工。


本文的示例代码能在[**GitHub**](https://github.com/arhohuttunen/junit5-examples/tree/main/junit5-gradle-kotlin)中找到。