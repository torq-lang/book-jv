# Configurators

Actors are not simply constructed. Instead, they are configured, and later, the configuration is spawned. The evolution from source statement to spawned actor can be described in four steps:

1. Generate -- compile the source to kernel code that creates actor records
2. Construct -- execute the kernel code to create a configurator
3. Configure -- invoke the configurator to create a configuration
4. Spawn -- spawn a concurrent process using the configuration

## Generate and Construct

### `HelloWorld`

In this section, we compile the following `HelloWorld` source into kernel code that, when executed, constructs an actor record containing a configurator.

```
01  actor HelloWorld(init_name) in
02      import system.Cell
03      var my_name = Cell.new(init_name)
04      handle tell {'name': name} in
05          my_name := name
06      end
07      handle ask 'hello' in
08          'Hello, World! My name is ' + @my_name + '.'
09      end
10  end
```

Compiling `HelloWorld` generates the following kernel code with code omitted in certain places to keep the example concise. Omissions are noted using `<<...>>`.

```
01  local $actor_cfgtr in
02      $create_actor_cfgtr(proc (init_name, $r) in // free vars: $import, $respond
03          local Cell, my_name, $v0, $v6 in
04              $import('system', ['Cell'])
05              $select_apply(Cell, ['new'], init_name, my_name)
06              $create_proc(proc ($m) <<...>> in end, $v0)
07              $create_proc(proc ($m) <<...>> in end, $v6)
08              $create_tuple('handlers'#[$v0, $v6], $r)
09          end
10      end, $actor_cfgtr)
11      $create_rec('HelloWorld'#{'cfg': $actor_cfgtr}, HelloWorld)
12  end
```

When executed, the kernel code creates the actor record `'HelloWorld'#{'cfg': $actor_cfgtr}` at line `11`. All actor records conform to the pattern `'ACTORNAME'#{'cfg': $actor_cfgtr}` where `ACTORNAME` is the name used in the source code and `$actor_cfgtr` is a configurator compiled from the actor body source.

Next, we discuss how configurators are used to produce configurations.

## Configure and Spawn

Configuring and spawning an actor involves two special Java classes: `ActorCfgtr` and `ActorCfg`.

The configurator discussed previously is an instance of `ActorCfgtr`, and when invoked, produces an instance of `ActorCfg`. The essence of both Java classes are shown below.

```java
public class ActorCfgtr implements Proc {
    private final Closure handlersCtor;

    public ActorCfgtr(Closure handlersCtor) {
        this.handlersCtor = handlersCtor;
    }

    @Override
    public final void apply(List<CompleteOrIdent> ys, Env env, Machine machine) throws WaitException {
        List<Complete> resArgs = new ArrayList<>(ys.size());
        for (int i = 0; i < ys.size() - 1; i++) {
            CompleteOrIdent y = ys.get(i);
            Complete yRes = y.resolveValue(env).checkComplete();
            resArgs.add(yRes);
        }
        CompleteOrIdent target = ys.get(ys.size() - 1);
        ValueOrVar targetRes = target.resolveValueOrVar(env);
        ActorCfg actorCfg = new ActorCfg(resArgs, handlersCtor);
        targetRes.bindToValue(actorCfg, null);
    }
}
```

```java
public final class ActorCfg implements Obj {

    private final List<Complete> args;
    private final Closure handlersCtor;

    public ActorCfg(List<Complete> args, Closure handlersCtor) {
        this.args = args;
        this.handlersCtor = handlersCtor;
    }
}
```

### `PerformHelloWorld`

In this section, we compile `PerformHelloWorld` and discuss how a parent actor configures and spawns its nested actor `HelloWorld` discussed previously.  

```
01  actor PerformHelloWorld() in
02      actor HelloWorld(init_name) in
03          import system.Cell
04          var my_name = Cell.new(init_name)
05          handle tell {'name': name} in
06              my_name := name
07          end
08          handle ask 'hello' in
09              'Hello, World! My name is ' + @my_name + '.'
10          end
11      end
12      handle ask 'perform' in
13          var hello_world = spawn(HelloWorld.cfg('Bob'))
14          var hello_bob = hello_world.ask('hello')
15          hello_world.tell({'name': 'Bobby'})
16          var hello_bobby = hello_world.ask('hello')
17          [hello_bob, hello_bobby]
18      end
19  end
```

Compiling `PerformHelloWorld` generates the following kernel code containing our nested actor `HelloWorld`. Again, omissions are noted using `<<...>>`. Note the two invocations of `$create_actor_cfgtr` at lines `02` and `04`, which create configurators for `PerformHelloWorld` and `HelloWorld`, respectively. 

```
01  local $actor_cfgtr in
02      $create_actor_cfgtr(proc ($r) in // free vars: $import, $respond, $spawn
03          local HelloWorld, $actor_cfgtr, $v9, $v15 in
04              $create_actor_cfgtr(proc (init_name, $r) in // free vars: $import, $respond
05                  local Cell, my_name, $v0, $v6 in
06                      $import('system', ['Cell'])
07                      $select_apply(Cell, ['new'], init_name, my_name)
08                      $create_proc(proc ($m) in <<...>> end, $v0)
09                      $create_proc(proc ($m) in <<...>> end, $v6)
10                      $create_tuple('handlers'#[$v0, $v6], $r)
11                  end
12              end, $actor_cfgtr)
13              $create_rec('HelloWorld'#{'cfg': $actor_cfgtr}, HelloWorld)
14              $create_proc(proc ($m) in // free vars: $respond, $spawn, HelloWorld
15                  local $else in
16                      $create_proc(proc () in <<...>> end, $else)
17                      case $m of 'perform' then
18                          local $v12, hello_world, hello_bob, hello_bobby in
19                              local $v13 in
20                                  $select_apply(HelloWorld, ['cfg'], 'Bob', $v13)
21                                  $spawn($v13, hello_world)
22                              end
23                              $select_apply(hello_world, ['ask'], 'hello', hello_bob)
24                              local $v14 in
25                                  $bind({'name': 'Bobby'}, $v14)
26                                  $select_apply(hello_world, ['tell'], $v14)
27                              end
28                              $select_apply(hello_world, ['ask'], 'hello', hello_bobby)
29                              $create_tuple([hello_bob, hello_bobby], $v12)
30                              $respond($v12)
31                          end
32                      else
33                          $else()
34                      end
35                  end
36              end, $v9)
37              $create_proc(proc ($m) in <<...>> end, $v15)
38              $create_tuple('handlers'#[$v9, $v15], $r)
39          end
40      end, $actor_cfgtr)
41      $create_rec('PerformHelloWorld'#{'cfg': $actor_cfgtr}, PerformHelloWorld)
42  end
```

The nested `HelloWorld` actor is defined between lines `04` and `13`. At run time, `PerformHelloWorld` configures and spawns a `HelloWorld` child as part of its `handle ask 'perform' in <<...>> end` kernel code defined between lines `19` and `22`. The `HelloWorld` configurator (an instance of `ActorCfgtr`) is invoked at line `20` to instantiate an `ActorCfg` and bind it to variable `$v13`. Subsequently, the `$spawn` instruction launches a concurrent actor using the `ActorCfg` bound to `$v13`. The result of the `$spawn` instruction is an `ActorRef` bound to the `hello_world` variable. Later, the `hello_world` variable is used to send messages to the new child actor.

## Native Configurators

In this section, we present a timer example that uses a native actor. The Torq program below configures and spawns a timer at line `05` as part of stream expression.  

```
01 actor IterateTimerTicks() in
02     import system[Cell, Stream, Timer, ValueIter]
03     handle ask 'iterate' in
04         var tick_count = Cell.new(0)
05         var timer_stream = Stream.new(spawn(Timer.cfg(1, 'microseconds')),
06             'request'#{'ticks': 5})
07         for tick in ValueIter.new(timer_stream) do
08             tick_count := @tick_count + 1
09         end
10         @tick_count
11     end
12 end
```

The Java program below runs the example from above and validates that the stream generated five timer ticks.

```java
ActorRef actorRef = Actor.builder()
    .setAddress(Address.create(getClass().getName() + "Actor"))
    .setSource(source)
    .spawn()
    .actorRef();

Object response = RequestClient.builder()
    .setAddress(Address.create("IterateTimerTicksClient"))
    .send(actorRef, Str.of("iterate"))
    .awaitResponse(100, TimeUnit.MILLISECONDS);

assertEquals(Int32.of(5), response);
```

The timer implementation is provided as a `TimerPack`.

```java
final class TimerPack {

    public static final Ident TIMER_IDENT = Ident.create("Timer");
    private static final int TIMER_CFGTR_ARG_COUNT = 3;
    private static final CompleteProc TIMER_CFGTR = TimerPack::timerCfgtr;
    public static final CompleteRec TIMER_ACTOR = createTimerActor();

    private static CompleteRec createTimerActor() {
        return CompleteRec.singleton(Actor.CFG, TIMER_CFGTR);
    }

    private static void timerCfgtr(List<CompleteOrIdent> ys, Env env, Machine machine) throws WaitException {
        if (ys.size() != TIMER_CFGTR_ARG_COUNT) {
            throw new InvalidArgCountError(TIMER_CFGTR_ARG_COUNT, ys, "timerCfgtr");
        }
        Num period = (Num) ys.get(0).resolveValue(env);
        Str timeUnit = (Str) ys.get(1).resolveValue(env);
        TimerCfg config = new TimerCfg(period, timeUnit);
        ys.get(2).resolveValueOrVar(env).bindToValue(config, null);
    }

    private static final class Timer extends AbstractActor {
        // <<...>>
    }

    private static final class TimerCfg extends OpaqueValue implements NativeActorCfg {
        final Num periodNum;
        final Str timeUnitStr;

        TimerCfg(Num periodNum, Str timeUnitStr) {
            this.periodNum = periodNum;
            this.timeUnitStr = timeUnitStr;
        }

        @Override
        public final ActorRef spawn(Address address, ActorSystem system, boolean trace) {
            return new Timer(address, system, trace, periodNum, timeUnitStr);
        }
    }

}
```

Note the following `TimerPack` elements and how they correlate to compiled Torq:

1. `TIMER_ACTOR` is a Torq record, which is the same structure produced when a source statement `actor <<...>> end` is compiled.
2. `TIMER_CFGTR` is a configurator and part of the `TIMER_ACTOR` record.
3. `TimerCfg` is an instance of `NativeActorConfig`

At run time, `LocalActor` recognizes the difference between Torq and Java configurations and spawns appropriately.

```java
ActorRefObj childRefObj;
if (config instanceof ActorCfg actorCfg) {
    childRefObj = spawnActorCfg(actorCfg);
} else {
    childRefObj = spawnNativeActorCfg((NativeActorCfg) config);
}
```

## Spawning an actor from Java

The previous sections describe how Torq and native actors are spawned from Torq. Here, we briefly discuss how to spawn actors from Java and obtain an instance of `ActorRef` to send the actor messages.

To configure an instance of a Torq actor, begin with a fluent builder instance using `ActorBuilder.builder()`. Actor builders are used throughout this book to run examples.

To configure an instance of a native actor, you must use the elements it provides. For example, the `Timer` actor above is held privately in the `TimerPack`, so you must access its configurator from its actor record.
