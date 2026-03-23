---
layout: post
title: Happy little unit tests
---

Do unit tests make you happy? When they work, they make me ecstatic. For agentic coding, they let you fire and forget, knowing that as long as the unit tests pass, your existing code (probably) won't be broken... Right?



# To fail or not to fail
At work we use a lot of common patterns. A lot of our classes look like this:

```csharp
public class EvenAndPositiveChecker(INumberRulesService service)
{
    public bool IsEvenAndPositive(int value)
    {
        return service.IsEven(value) && service.IsPositive(value);
    }
}
```

So your test cases will look like:

```csharp
public class EvenAndPositiveCheckerTests
{
    private INumberRulesService _rules;
    private EvenAndPositiveChecker _sut;

    [Setup]
    public void Setup()
    {
        _rules = Substitute.For<INumberRulesService>();
        _sut = new EvenAndPositiveChecker(_rules);
    }

    [Test]
    public void IsEvenAndPositive_OddNumber_ReturnsFalse()
    {
        _rules.IsEven(1).Returns(false);

        var result = _sut.IsEvenAndPositive(1);

        Assert.False(result);
    }

    [Test]
    public void IsEvenAndPositive_NegativeNumber_ReturnsFalse()
    {
        _rules.IsPositive(-2).Returns(false);

        var result = _sut.IsEvenAndPositive(-2);

        Assert.False(result);
    }
}
```

But wait -- the tests pass, but they aren't technically correct. We're only ever proving one branch, because the substitute returns `false` by default for any call we haven't set up, including `_rules.IsEven` in the second test. And it's not very obvious from reading the test that there is a problem. We can introduce a bug like this, and the tests will still pass:

```csharp
public class EvenAndPositiveChecker(INumberRulesService service)
{
    public bool IsEvenAndPositive(int value)
    {
        return service.IsEven(value) && !service.IsPositive(value);
    }
}
```

# What the fix?
One approach is to add helpers that set up the green path for each service call.

```csharp
public class EvenAndPositiveCheckerTests
{
    private INumberRulesService _rules;
    private EvenAndPositiveChecker _sut;

    [Setup]
    public void Setup()
    {
        _rules = Substitute.For<INumberRulesService>();
        _sut = new EvenAndPositiveChecker(_rules);
    }

    [Test]
    public void IsEvenAndPositive_OddNumber_ReturnsFalse()
    {
        SetUpIsPositive();
        _rules.IsEven(Arg.Any<int>()).Returns(false);

        var result = _sut.IsEvenAndPositive(Some.Int());

        Assert.False(result);
    }

    [Test]
    public void IsEvenAndPositive_NegativeNumber_ReturnsFalse()
    {
        SetUpIsEven();
        _rules.IsPositive(Arg.Any<int>()).Returns(false);

        var result = _sut.IsEvenAndPositive(Some.Int());

        Assert.False(result);
    }

    private void SetUpIsEven()
    {
        _rules.IsEven(Arg.Any<int>()).Returns(true);
    }

    private void SetUpIsPositive()
    {
        _rules.IsPositive(Arg.Any<int>()).Returns(true);
    }
}
```

I've seen legacy code that does this, and it reeks of an author who wanted to be as precise as possible, regardless of the boilerplate it introduces. It does make it explicit how every service is set up for each test, but you end up with a lot of repeated code, and the tedious job of checking everything is configured and nothing is missed.

# A modern (lazy but simple) solution
I like a solution that's less precise but simpler. We set up the green path in the setup, and each test mutates just enough to hit a sad path:

```csharp
public class EvenAndPositiveCheckerTests
{
    private INumberRulesService _rules;
    private EvenAndPositiveChecker _sut;

    [Setup]
    public void SetupHappyPath()
    {
        _rules = Substitute.For<INumberRulesService>();
        _rules.IsEven(Arg.Any<int>()).Returns(true);
        _rules.IsPositive(Arg.Any<int>()).Returns(true);
        _sut = new EvenAndPositiveChecker(_rules);
    }

    [Test]
    public void IsEvenAndPositive_HappyPath_ReturnsTrue()
    {
        var result = _sut.IsEvenAndPositive(Some.Int());

        Assert.True(result);
    }

    [Test]
    public void IsEvenAndPositive_OddNumber_ReturnsFalse()
    {
        _rules.IsEven(Arg.Any<int>()).Returns(false);

        var result = _sut.IsEvenAndPositive(Some.Int());

        Assert.False(result);
    }

    [Test]
    public void IsEvenAndPositive_NegativeNumber_ReturnsFalse()
    {
        _rules.IsPositive(Arg.Any<int>()).Returns(false);

        var result = _sut.IsEvenAndPositive(Some.Int());

        Assert.False(result);
    }
}
```

The lack of precision comes from reassigning the substitute -- in NSubstitute, later `.Returns()` calls override earlier ones for the same argument. Not ideal, but the tradeoff is worth it for the simplicity.

Naming is also worth thinking about. I like calling the setup `SetupHappyPath` explicitly. It tells the reader what the default state is, so there are no surprises.

The example above is contrived, but a lot of services I write tend to look more like this:

```csharp
public class DatabaseSizeProvider 
{
    public int? GetSize(DatabaseCredentials credentials)
    {
        return dbService.CreateConnection(credentials) is Connection connection
            && sqlService.CreateGetSizeCommand(connection) is Command command
            && queryExecutor.ExecuteQuery(command) is ResultSet results
            && resultParser.ParseResult(results) is int size
            ? size
            : null;
    }
}
```

I keep coming back to this pattern. It's precise, elegant, and reads cleanly top to bottom, everything a class should be. Ironically, happy path first unit tests are none of these things. They lack precision because of all the reassigning of substitutes. They're not elegant, they rely on state in the class. And they don't read top to bottom, because they hide logic in the setup. And honestly, I'm happy for that. Good tests should be practical, implicit, and hide boilerplate. Write code you're proud of. Write tests that work. They don't have to be the same thing.