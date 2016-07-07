# Mockery cheat sheet
Cheat sheet for people getting started with Php's Mockery

### Create a new mock

```
$mock = \Mockery::mock('SomeClass', ['constructor', 'args']); //indexed array = constructor 
$mock = \Mockery::mock('SomeClass', ['method1' => 'ret_val1', 'method2' => 'ret_val2']); //assoc array = mock method
```

### Always remember!

Always call \Mockery::close in the `tearDown` method of your tests. Otherwise it won't be able to verify your assertions.

```
\Mockery::close();
```

### Expect a function was fired

```
$mock = \Mockery::mock('SomeClass', ['method1' => 'ret_val1']);
$mock->shouldReceive('method1')->once()->andReturn('Honeybunny'); //->once() or ->twice() or ->times(num) or ->atLeast()->times(1);
echo $mock->method1();

//And dont' forget to call \Mockery::close(); in tearDown
```

### Expect a function was NEVER fired

```
$mock = \Mockery::mock('SomeClass', ['method1' => 'ret_val1']);
$mock->shouldReceive('method1')->never(); //also possible to use `shouldNotReceive` instead of ->never()
echo $mock->method1('honey', 'bunny');
```

### Expect a function was fired with some parameters

```
$mock = \Mockery::mock('SomeClass', ['method1' => 'ret_val1']);
$mock->shouldReceive('method1')->once()->with('honey', 'bunny')->andReturn('Honeybunny');
echo $mock->method1('honey', 'bunny');
```

### Expect a function was fired with a particlar object of Class or type of param like number, string, etc

Full list of matchers can be found here: http://www.php.net/manual/en/ref.var.php (matcher doesn't have the leading `is_` part)

```
$mock = \Mockery::mock('SomeClass', ['method1' => 'ret_val1']);
$mock->shouldReceive('method1')->once()->with(\Mockery::type('numeric'))->andReturn('Honeybunny');
echo $mock->method1(3);
```

#### Other useful matchers that can be used with `with`

Regualr expressions
```
->with('/some\-regex/i')
```

Selection
```
->with(\Mockery::anyOf('option1', 'option2'))
```

Hamcrest expressions

```
 ->with(stringValue()),  ->with(arrayValue()),  ->with(nullValue())
 ```

### How to only mock only a few functions inside a Class?

```
class Test {
  function a () { return 'a'; }
  function b () { return 'b'; }
}

$mock = Mockery::mock('Test[a]', ['a' => 'c']);
echo $mock->a(); //prints 'c'
echo $mock->b(); //prints 'b'
```


