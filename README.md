# **Session9 Decorators**
A decorator takes in a function, addAuthentications some functionality and returns it. In this tutorial, you will learn how you can create a decorator and why you should use it.

## **Assignment**
Write separate decorators that:
1. allows a function to run only on odd seconds 
2. log 
3. single_dispacth 
4. timed (n times)
5. Provides privilege access (has 4 parameters, based on privileges (high, mid, low, no), gives access to all 4, 3, 2 or 1 params) 
6. Write our htmlize code using inbuild singledispatch 

## *Functions**

### 1. odd_execution
Decorator that lets function to execute only on odd seconds
```python
def odd_execution(fn: "Function"):
    """
    Decorator that lets function to execute only on odd seconds
    
    """
    @wraps(fn)
    def inner(*args, **kwargs):
        CURR_TIME = datetime.now()
        print(CURR_TIME)
        if CURR_TIME.second % 2 != 0:
            return fn(*args, **kwargs)
    return inner
```

### 2. log_dec
 Decorator that prints the logs
 ```python
 def log_dec(fn: "Function"):
    """
    Decorator that prints the logs
    
    """
    @wraps(fn)
    def inner(*args, **kwargs):
        run_dt = datetime.now()
        result = fn(*args, **kwargs)
        end_dt = datetime.now()
        print(f"{run_dt}: called {fn.__name__}")
        print(f"Execution time: {end_dt - run_dt}")
        print(f"Function description:\n{fn.__doc__}")
        print(f"Function returned something: {True if result else False}")
        return result
    return inner
```

### 3. set_password
  A closure to store and set password
  ```python
def set_password() -> "Function":
    """
    A closure to store and set password
    """
    password = ""

    def inner():
        nonlocal password
        if password == "":
            password = get_password()
        return password

    return inner

```

### 4. authenticate
Wrapper used to authenticate the function before executing the function.
    curr_password : password from set_password closure
    user_password : input passowrd to compare
```python
def authenticate(curr_password: str, user_password: str):
    """
    Wrapper used to authenticate the function before executing the function.
    curr_password : password from set_password closure
    user_password : input passowrd to compare
    
    """ 

    def decor(fn: "Function"):
        @wraps(fn)
        def inner(*args, **kwargs):
            if user_password == curr_password():
                print("You are authenticated")
                return fn(*args, **kwargs)
            else:
                print("Password Mismatch")

        return inner

    return decor
```

### 5. time_it
Decorator factory to take in an integer 
    and runs for that many times and calculates the average time taken for a function

```python
def time_it(reps: int):
    """
    Decorator factory to take in an integer 
    and runs for that many times and calculates the average time taken for a function
    """
    if type(reps) is not int:
        raise TypeError("Invalid type, expected int!")
    if reps < 1:
        raise ValueError("It should atleast have 1 iteration")
        
    def timed(fn):
        @wraps(fn)
        def inner(*args, **kwargs):
            total_taken = 0
            for _ in range(reps):
                start = perf_counter()
                result = fn(*args, **kwargs)
                end = perf_counter()
                total_taken += (end - start)
            avg_run_time = total_taken / reps
            print(f'Avg Run time: {avg_run_time}s ({reps} reps)')
            return result
        return inner
    return timed
```
### 6. Class Privilege
This decorator class  wraps any function with certain privileges.
    Based on privileges (high, mid, low, no),it provides access to all 4, 3, 2 or 1 parameters.

```python
class Privilege:
    """
    This decorator class  wraps any function with certain privileges.
    Based on privileges (high, mid, low, no),it provides access to all 4, 3, 2 or 1 parameters.
    """

    def __init__(self, privilege="no"):
        if privilege and privilege in ("high", "mid", "low", "no"):
            self.privilege = privilege
        else:
            self.privilege = "no"

    def __call__(self, fn):

        @wraps(fn)
        def inner(base, **kwargs):
            params = list(kwargs.items())
            cnt = {"high": 3, "mid": 2, "low": 1, "no":0}
            if len(params) < cnt[self.privilege] :
                raise ValueError("Invalid number of keyword arguments!")

            if self.privilege == "high":
                return fn(base, **dict(params[:3]))

            elif self.privilege == "mid":
                return fn(base, **dict(params[:2]))

            elif self.privilege == "low":
                return fn(base, **dict(params[:1]))

            elif self.privilege == "no":
                return fn(base)
        return inner
```

### 7. Htmilze 
converts the given type of data to html format using singledispatch wrapper function

```python
@singledispatch
def htmlize(str_esc: str) -> str:
    """
    Converts the newline character in a string to contain a br tag.
    input: string to escape.
    return: string, transformed
    """
    return escape(str(str_esc)).replace("\n", "<br/>\n")
```
