---
layout: cover
defaults:
  background: "assets/background.jpg"
---

# Fabio Dipilato

E il suo amore nei confronti di Python

<!---

- Parlare di Me
- Accennare lavoro in Futura
- Spiegare la potenza dei decoratori entry level
- Spiegare la potenza dei decoratori end level

-->

---
layout: cover
---

# Teoria

<!---

- Nella sezione teoria perliamo dei meccanissimi fondamentali di python che permettono il corretto funzionamento dei decoratori

-->

---

# Teoria dei decoratori

<span style="font-size: 1.6em; line-height: 1.3em">Il design pattern <b>Decorator</b> fornisce un’alternativa flessibile all’ereditarietà per estendere la funzionalità degli oggetti. Tale pattern consente di arricchire dinamicamente, a run-time, un oggetto con nuove funzionalità: è possibile impilare uno o più decorator uno sopra l’altro, ciascuno aggiungendo nuove funzionalità</span>

```python
@questo_e_un_decoratore
def questa_e_una_funzione():
  pass
```

---

# Un altro po' di teoria

In python le funzioni sono first-class object. Questo ci permette di passare la funzione come argomento per un altra funzione passandolo come valore

```python {0|all}
def say_hello(name: str) -> str:
  return f"Hello {name}"
```

```python {0|all}
def say_hi(name: str) -> str:
  return f"Hi {name}"
```

```python {0|all}
def greet_bob(greeter_func: callable) -> str:
  return greeter_func('Bob')
```

<div v-click="4" class="mt-5">
```python {0|0|1-2|4-5}
>>> greet_bob(say_hello)
'Hello Bob'

>>> greet_bob(say_hi)
'Hi Bob'
```
</div>

<!---
First class object
Un first-class object è un entita nel linguaggio di programmazione che permette di:

- Apparire in un espressione
- Essere assegnata come variabile
- Usata come argomento
- Restituita da una funzione

-->

---

# Definire funzioni all'interno di funzioni

```python {0|1-2|4-8|4-5|7-8|10-11|all|0}
def first_function():
  print("Printing from the first function")

  def first_sub_function():
    print("Printing from the first sub function")
  
  def second_sub_function():
    print("Printing from the second sub function")

  first_sub_function()
  second_sub_function()
```
<div class="mt-6 mb-6" style="display: flex; justify-content: space-between; align-items: center">
<div class="w-100">
```python {0|1|1-2|1-3|all}
>>> first_function()
'Printing from the first function'
'Printing from the first sub function'
'Printing from the second sub function'
```
</div>
<div class="w-100">
```python {0|1|all}
>>> first_sub_function()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'first_sub_function' is not defined
```
</div>
</div>


---

# Restituire una funzione da una funzione

```python{0|all}
def parent(num: int) -> callable:
  def first():
    return "Hi, i'm Fabio"
  def second():
    return "Hi, i'm Luca"

  if num == 1: return first
  else: return second
```

```python{0|all}
>>> f = parent(1)
>>> s = parent(2)

>>> f
"<function parent.<locals>.first at 0x000002236C1381F8>"

>>> s
"<function parent.<locals>.second at 0x000002236C138438>"
```

```python{0|all}
>>> f()
"Hi i'm Fabio"

>>> s()
"Hi, i'm Luca"
```

---

# Sì, ma adesso?

```python {0|1|4|2-6|1-6|8-9|11|all}
def my_first_decorator(func: callable) -> callable:
  def wrapper():
    print("Before the function.")
    func()
    print("After the function.")
  return wrapper

def say_hi_to_python():
  print("Hi Python!")

say_hi_to_python = my_first_decorator(say_hi_to_python)
```

```python{0|1|all}
>>> say_hi_to_python()
'Before the function.'
'Hi Python!'
'After the function.'
```

---

# Syntactic Sugar

```python {all|12|8|all}
def my_first_decorator(func: callable) -> callable:
  def wrapper():
    print("Before the function.")
    func()
    print("After the function.")
  return wrapper

@my_first_decorator
def say_hi_to_python():
  print("Hi Python!")

# say_hi_to_python = my_first_decorator(say_hi_to_python)

>>> say_hi_to_python()
'Before the function.'
'Hi Python!'
'After the function.'
```

---

# Il nosto primo vero decoratore

```python {all|1-2|4|6-7|9|all}
@app.route('/user/', methods=['GET'])
def user_route():
  
  user_uid = request.headers.get('Authorization', None)

  if user_uid is None:
    return "", 401

  return jsonify(user_get(user_uid))
```

<div v-click=5>
<div style="display: flex; justify-content: space-between; align-items: center">
<div class="w-100 h-100">
```python {all|2|5|7-8|all}
@app.route('/user/', methods=['GET'])
@login_required
def user_route():
  
  user_uid = request.headers['Authorization']

  #if user_uid is None:
  #  return "", 401

  return jsonify(user_get(user_uid))
```
</div>
<div class="w-100 h-100" v-click="9">
```python {all|2|3-6|all}
def login_required(func: callable) -> callable:
  def wrapper():
    if request.headers.get('Authorization', None):
      return func()
    else:
      return "", 401
  return wrapper
```
</div>
</div>
</div>
---

# Altro caso reale

```python {0|1|3|5|6|7|8|9|4-10|all}
import time

def execution_time(func: callable):
  def wrapper():
    start = time.time()
    result = func()
    total_time = time.time() - start
    print(f"Function took {total_time}s")
    return result
  return wrapper

```

```python {0|1|3|4-8|10|11-13}
import math

@execution_time
def time_consiuming_function():
  results = []
  for i in range(0, 10_000):
    results.append(math.factorial(i))
  return results

>>> time_consiuming_function()
[...]
"Function took 342.438085079193115s"
"~ 5,7 minuti"

```

---

# Altri decoratori

```python {0|1|3-7|all}
# decorators.py

def repeat(func: callable):
  def wrapper_repeat():
    func()
    func()
  return wrapper_repeat
```

```python {0|3|4-6|8|9-10|all}
# main.py
from decorators import repeat

@repeat
def say_hello():
  print("Hello")

>>> say_hello()
"Hello"
"Hello"
```

---

# Attenzione!

```python {0|1|3-5|7|8-10}
from decorators import repeat

@repeat
def greet(name: str):
  print(f"Hello {name}")

>>> greet("Fabio")
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: wrapper_repeat() takes 0 positional arguments but 1 was given
```

```python {0|4|5-6|all}
# decorators.py

def repeat(func: callable):
  def wrapper_repeat(*args, **kwargs):
    func(*args, **kwargs)
    func(*args, **kwargs)
  return wrapper_repeat
```

---

# Funzione, chi sei veramente?

<span v-click="1">
Uno strumento molto utile quando si lavora con Python è, specialmente, la shell interattiva e il suo potere di introspezione.
Questo ci permette di sapere di un oggetto i suoi attributi al runtime.
</span>

<div class="mt-10" style="display: flex; justify-content: space-between; align-items: center" v-click="2">
```python {0|all|1-2|4-5|7-11}
>>> print
<built-in function print>

>>> print.__name___
'print'

>>> help(print)
Help on built-in function print in module builtins:

print(...)
    <full help message>
```
  <img src="https://i.ibb.co/0F3z1xM/posaman-so-lillo.jpg"/>
</div>

---

# Funzione, chi sei veramente?

<span>
Uno strumento molto utile quando si lavora con Python è, specialmente, la shell interattiva e il suo potere di introspezione.
Questo ci permette di sapere di un oggetto i suoi attributi al runtime.
</span>

<div class="mt-10" style="display: flex; justify-content: space-between; align-items: center">
  <img src="https://i.ibb.co/0F3z1xM/posaman-so-lillo.jpg"/>
```python {0|all|1-2|4-5|7-11}
>>> say_hello
<function repeat.<locals>.wrapper_repeat at 0x7f43700e52f0>

>>> say_hello.__name___
'wrapper_repeat'

>>> help(say_hello)
Help on function wrapper_repeat in module decorators:

wrapper_repeat()
```
</div>

---

# Come risolvere?

```python {all|2|5|all}
# decorators.py
import functools

def repeat(func: callable) -> callable:
  @functools.wraps(func)
  def wrapper_repeat():
    func()
    func()
  return wrapper_repeat
```

```python {0|1-2|4-5|7-10|all}
>>> say_hello
<function say_hello at 0x7ff79a60f2f0>

>>> say_hello.__name___
'say_hello'

>>> help(say_hello)
Help on function say_hello in module main:

say_hello()
```

<!--- Dettagli tecnici

Il decoratore @functools.wraps usa la funzione functools.update_wrapper() per aggiornare gli attributi speciali, come __name__ e __doc__
che sono usati nell'introspezione della shell-->

---
layout: cover
---

# Decoratori avanzati

---


# Registrare un addon

I decoratori non devono per forza wrappare la funzione che stanno decorando, possono semplicemente registrare la funzione in un dizionario per creare un mini sistema di addons

<div style="display: flex; justify-content: space-between; align-items: center">
<div class="w-100 h-100">
```python {0|2|4-6|8-14|16-20|all}
import random
ADDONS = dict()

def save_addon(func: callable) -> callable:
    ADDONS[func.__name__] = func
    return func

@save_addon
def say_hello(name):
    return f"Hello {name}"

@save_addon
def say_hi(name):
    return f"Hi {name}"

def randomly_greet(name):
    greeter, greeter_func = 
      random.choice(list(ADDONS.items()))
    print(f"Using {greeter}")
    return greeter_func(name)
```
</div>
<div class="w-100 h-100">

```python {0|1|1-3|5|5-7|all}
>>> ADDONS
{'say_hello': <function say_hello at 0x7f768eae6730>,
 'say_hi': <function say_hi at 0x7f768eae67b8>}

>>> randomly_greet("Lorenzo")
Using 'say_hello'
'Hello Lorenzo'
```

</div>
</div>

---

# Decoratori che accettano parametri

```python {0|1|3|4|5|6-10|2|all}
# users.py
user_target = None
@app.route('/user/', methods=['GET'])
@login_required(is_admin=True)
@check_header(['User-Target'])
def user_admin_route():
  user_uid = user_target
  return jsonify(user_admin_get(user_uid), 200)
```

<div style="display: flex; justify-content: space-between; align-items: center">
<div class="w-100 h-100">

```python {0|1|2-3|12-13|2-12|1-13|4|5-10|all}
def login_required(is_admin=False) -> callable:
  def decorator(func: callable) -> callable:
    def wrapper(*args, **kwargs):
      
      user_auth = request.headers.get('Authorization')

      if user_auth and is_user_admin(user_auth): 
        return func(*args, **kwargs)
      else: 
        return "", 401

    return wrapper
  return decorator
```

</div>
<div class="w-100 h-100">

```python {0|1|2-3|12-13|3-11|2-12|all|4-8|9-10|11|all}
def check_header(headers_to_check: list = []) -> callable:
  def decorator(func: callable) -> callable:
    def wrapper(*args, **kwargs):
      for header in headers_to_check:
        if request.headers.get(header, None) is None:
          return jsonify({
            'error': f"Missing {header} in header"
          }, 400)
      global user_target
      user_target = request.headers.get('User-Target')
      return func(*args, **kwargs)
    return wrapper
  return decorator
```

</div>
</div>

---

```python {all|1|2|26|3|25|4-9|10-16|17-24|all} {maxHeight:'100'}
def decorator_save_function_result(minute_resets: int):
    def decorator(function: callable):
        def wrapped_function(*args, **kwargs):
            args_string = ','.join(args) if isinstance(args, list) else str(args)
            kwargs_string = ','.join(f'{key}={value}' for key, value in kwargs.items())
            function_name = function.__name__
            paremeters_string = function_name + "," + args_string + "," + kwargs_string
            function_result_name = hashlib.sha224(paremeters_string.encode()).hexdigest()
            function_result_file_name = f'{function_name}/{function_result_name}.json'
            try:
                saved_result = S3Resource.get_item(FUNCTION_RESULTS_BUCKET, function_result_file_name)
                if saved_result['time'] + minute_resets * 60 > time.time(): return saved_result['result']
                else: print('result expired')
            except Exception as e:
                print('No cached result found', e)
                pass
            print('Function result name', function_result_name)
            result = function(*args, **kwargs)
            result_json = {
                'time': time.time(), 'args': args,
                'kwargs': kwargs, 'result': result
            }
            S3Resource.upload_json(FUNCTION_RESULTS_BUCKET, function_result_file_name, result_json)
            return result
        return update_wrapper(wrapped_function, function)
    return decorator
```

---

<style>
  .social-card {
    background-color: white;
    padding: 1em;

    border-radius: 1em;

    display: flex;
    justify-content: space-between;
    align-items: center;

    margin-bottom: 1em;

    color: black;

    font-weight: bold;

    width: 18em;
  }

  .social-card .image-container {
    display: flex;
    flex-direction: column;

    align-items: center;
    justify-content: center;

    margin-right: 1.3em;
  }

  .social-card .image-container img {
    background-color: white;
    width: 30px;
    aspect-ratio: 1;

    /* margin-bottom: .5em; */
  }
</style>

<div style="height: 100%; display: flex; justify-content: space-around; align-items: center; padding-top: 8em; padding-bottom: 8em;">
  <h1 style="width: 7.38em; text-align: center">Grazie</h1>
  <div style="display: flex; flex-direction: column">
    <div class="social-card">
      <div class="image-container">
        <img src='/assets/instagram.svg'>
        <!-- <span>Instagram</span> -->
      </div>
        fabio_dipilato
    </div>
    <div class="social-card">
      <div class="image-container">
        <img src='/assets/linkedin.svg'>
        <!-- <span>Linkedin</span> -->
      </div>
        fabio-dipilato
    </div>
    <div class="social-card">
      <div class="image-container">
        <img src='/assets/whatsapp.svg'>
        <!-- <span>Telefono</span> -->
      </div>
        +39 380 766 5609
    </div>
    <div class="social-card">
      <div class="image-container">
        <img src='/assets/gmail.svg'>
        <!-- <span>Contatto</span> -->
      </div>
        fabio@wearefutura.com
    </div>
  </div>
</div>

---

<style>

  .me-img {
    aspect-ratio: 1/1;
    width: 15em;

    border-radius: 100%;

    border: 2px solid white;

    margin-bottom: 1.5em
  }

</style>

<div style="height: 100%; display: flex; justify-content: space-around; align-items: center; padding-top: 8em; padding-bottom: 8em;">
  <div style="display: flex; flex-direction: column; justify-content: center; align-items: center">
    <img class="me-img" src='/assets/me.jpeg'>
    <h1 style="margin-bottom: .1em">Fabio Dipilato</h1>
    <span>Fullstack Developer</span>
  </div>
  <div style="display: flex; flex-direction: column">
    <div class="social-card">
      <div class="image-container">
        <img src='/assets/instagram.svg'>
        <!-- <span>Instagram</span> -->
      </div>
        fabio_dipilato
    </div>
    <div class="social-card">
      <div class="image-container">
        <img src='/assets/linkedin.svg'>
        <!-- <span>Linkedin</span> -->
      </div>
        fabio-dipilato
    </div>
    <div class="social-card">
      <div class="image-container">
        <img src='/assets/whatsapp.svg'>
        <!-- <span>Telefono</span> -->
      </div>
        +39 380 766 5609
    </div>
    <div class="social-card">
      <div class="image-container">
        <img src='/assets/gmail.svg'>
        <!-- <span>Contatto</span> -->
      </div>
        fabio@wearefutura.com
    </div>
  </div>
</div>