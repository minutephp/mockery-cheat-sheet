# Mockery cheat sheet
Cheat sheet for people getting started with Php's Mockery - https://github.com/padraic/mockery

### Create a new mock

```
//indexed array = constructor args for class 
$mock = \Mockery::mock('SomeClass', ['constructor', 'args']); 
//assoc array = mock method and return values
$mock = \Mockery::mock('SomeClass', ['method1' => 'ret_val1', 'method2' => 'ret_val2']); 
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
echo $mock->method1(); //prints ret_val1

//Never forget to call \Mockery::close(); in tearDown
```

### Expect a function was NEVER fired

```
$mock = \Mockery::mock('SomeClass', ['method1' => 'ret_val1']);
$mock->shouldReceive('method1')->never(); //also possible to use `shouldNotReceive('method1')`
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

Any arguments
```
->withAnyArgs()
```

Without arguments
```
->withNoArgs()
```

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

Create partial mocks like this:

```
class Test {
  function a () { return 'a'; }
  function b () { return 'b'; }
}

$mock = Mockery::mock('Test[a]', ['a' => 'c']);
echo $mock->a(); //prints 'c'
echo $mock->b(); //prints 'b'
```

### How to mock Model Objects? (like Laravel Models)

Create mock aliases (mocks that extend \stdClass)

```
$userMock = \Mockery::mock('alias:App\Model\User');
$userMock->first_name = 'San';
//\Mockery::self() returns the last mock created (i.e. $userMock in this case)
$userMock->shouldReceive('where')->andReturn(\Mockery::self())->mock(); 
$userMock->shouldReceive('first')->andReturn(\Mockery::self())->mock();

echo $userMock->first_name; //prints San also
echo $userMock instanceof \App\Model\User ? 'true' : 'false'; //prints true
echo $userMock->where()->first()->first_name; //prints San also
```

### How to intercept new Class creation with Mocks?

You can overload your classes (i.e. replace your classes with Mocks) using the `overload:` keyword

```
class Test {
    function a () {
        $api = new Api;
        return $api->fetch();
    }
}

$mock = \Mockery::mock('overload:Test\Mailer\Api');
$mock->shouldreceive('fetch')->andReturn('someResult');

$instance = new Test;
echo $instance->a(); //prints 'someResult'
```

### How to silently ignore method not defined in your $mock?

```
$mock = \Mockery::mock('MyClass')->shouldIgnoreMissing()->asUndefined(); //returns null without ->asUndefined()
echo $mock->missingMethod(); //prints Undefined (references to \Mockery\Undefined) 

```

### How to call methods of *Original* Class not defined in your $mock? 

Best way is to create partial mocks like shown above. But TIMTWOTDI

```
\Mockery::mock('MyClass')->shouldDeferMissing();
echo $mock->missingMethod(); //will call original method, i.e. n$MyClass::missingMethod()
```

### How to call the original Class function after assertions?

```
class Test {
 function a () { return 'a'; }
}

$mock = \Mockery::mock('Test');
$mock->shouldreceive('a')->once()->passthru();
echo $mock->a(); //print 'a'
}
```

### How to assert that which function is called after which?

```
$mock = \Mockery::mock('Test', ['first' => '1', 'second' => 2]);
$mock->shouldreceive('first')->once()->ordered();
$mock->shouldreceive('second')->once()->ordered();

echo $mock->first();
echo $mock->second(); //PASSES!

echo $mock->second();
echo $mock->first(); //FAILS [ Method Mockery_0__Test::first()() called out of order: expected order 1, was 2 ]
```

### How to check something in all the *tests* defined in a Test Class by default?

```
public function setUp() {
    parent::setUp();

    $this->mock = \Mockery::mock('Test', ['first' => '1', 'second' => 2]);
    $this->mock->shouldreceive('first')->once()->with('1')->andReturn('1')->byDefault();
}

public function testMock() {
    echo $this->mock->first('1');
}

public function testMock2() {
    //this overrides the Default Check!
    $this->mock->shouldreceive('first')->once()->with('2')->andReturn('1');
    echo $this->mock->first('2');
}
```

### How to Mock `final` classes?

```
$mock = \Mockery::mock(new FinalClass, ['check' => '1']);
echo $mock->check(); //prints 1
```

