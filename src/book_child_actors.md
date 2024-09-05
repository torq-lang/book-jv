# Child Actors

## Programming `ConcurrentMath` 

```
actor ConcurrentMath() in
    actor Number(n) in
        handle ask 'get' in
            n
        end
    end
    var n1 = spawn(Number.cfg(1)),
        n2 = spawn(Number.cfg(2)),
        n3 = spawn(Number.cfg(3))
    handle ask 'calculate' in
        n1.ask('get') + n2.ask('get') * n3.ask('get')
    end
end
```

## Running `ConcurrentMath`

```java
public static final String SOURCE = """
    actor ConcurrentMath() in
        actor Number(n) in
            handle ask 'get' in
                n
            end
        end
        var n1 = spawn(Number.cfg(1)),
            n2 = spawn(Number.cfg(2)),
            n3 = spawn(Number.cfg(3))
        handle ask 'calculate' in
            n1.ask('get') + n2.ask('get') * n3.ask('get')
        end
    end""";

public static void perform() throws Exception {

    ActorRef actorRef = Actor.builder()
        .spawn(SOURCE);

    Object response = RequestClient.builder()
        .sendAndAwaitResponse(actorRef, Str.of("calculate"),
            100, TimeUnit.MILLISECONDS);
    checkExpectedResponse(Int32.of(7), response);
}
```

## Programming `ConcurrentMath` with stateful behavior

```
actor ConcurrentMath() in
    import system.Cell
    actor Number(n) in
        var value = Cell.new(n)
        handle ask 'get' in
            @value
        end
        handle tell 'incr' in
            value := @value + 1
        end
    end
    var n1 = spawn(Number.cfg(0)),
        n2 = spawn(Number.cfg(0)),
        n3 = spawn(Number.cfg(0))
    handle ask 'calculate' in
        n1.tell('incr')
        n2.tell('incr'); n2.tell('incr')
        n3.tell('incr'); n3.tell('incr'); n3.tell('incr')
        n1.ask('get') + n2.ask('get') * n3.ask('get')
    end
end
```

## Running `ConcurrentMath` with stateful behavior

```java
public static final String SOURCE = """
    actor ConcurrentMath() in
        import system.Cell
        actor Number(n) in
            var value = Cell.new(n)
            handle ask 'get' in
                @value
            end
            handle tell 'incr' in
                value := @value + 1
            end
        end
        var n1 = spawn(Number.cfg(0)),
            n2 = spawn(Number.cfg(0)),
            n3 = spawn(Number.cfg(0))
        handle ask 'calculate' in
            n1.tell('incr')
            n2.tell('incr'); n2.tell('incr')
            n3.tell('incr'); n3.tell('incr'); n3.tell('incr')
            n1.ask('get') + n2.ask('get') * n3.ask('get')
        end
    end""";

public static void perform() throws Exception {

    ActorRef actorRef = Actor.builder()
        .spawn(SOURCE);

    // 1 + 2 * 3
    Object response = RequestClient.builder().sendAndAwaitResponse(actorRef,
        Str.of("calculate"), 100, TimeUnit.MILLISECONDS);
    checkExpectedResponse(Int32.of(7), response);

    // 2 + 4 * 6
    response = RequestClient.builder().sendAndAwaitResponse(actorRef,
        Str.of("calculate"), 100, TimeUnit.MILLISECONDS);
    checkExpectedResponse(Int32.of(26), response);

    // 3 + 6 * 9
    response = RequestClient.builder().sendAndAwaitResponse(actorRef,
        Str.of("calculate"), 100, TimeUnit.MILLISECONDS);
    checkExpectedResponse(Int32.of(57), response);
}
```