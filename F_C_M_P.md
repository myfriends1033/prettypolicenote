## 兩張圖看懂這四個區別
參考 [這裡1](https://openhome.cc/Gossip/CodeData/PythonTutorial/FunctionModuleClassPackagePy3.html)

![c_diff_f](https://myfriends1033.github.io/article/photo/C_F.PNG)
- 類別與函式範例

----
![p_diff_m](https://myfriends1033.github.io/article/photo/M_P.PNG)
- 只要你建立了一個**原始碼檔案 name.py**，你就建立了一個**模組 name**，**原始碼主檔名就是模組名稱**，也就如同圖中的 xmath.py 就可以是一個模組
- 當你要建立一個套件，只需要將各個 .py 檔放在同一個資料夾內，並且在同資料夾內設置一個 \_\_init\_\_ .py 的空檔案，該**資料夾名稱就能成為一個套件的名稱**，如圖中的 openhome

## 函式（Function）

```python
def say_hello():
    print("hello world")
say_hello()
```
\>\> hello world

```python
def addNum(a,b):
    return a+b
print(addNum(1,4))
```
\>\> 5

```python
result = addNum(4,5)
print(result)
print(addNum('one','two'))
```
\>\> 9
\>\> onetwo

```python
def addall(x, *args):
    res = x
    for y in args:
        res +=y
    return res
print(addall(1,2,3,4,5))
```
\>\> 15

```python
def make_two_lists(**kwargs):
    keys,values = [],[]
    for k,v in kwargs.items():
        keys.append(k)
        values.append(v)
    return[keys,values]
print(make_two_lists(david = 'M', Mary = 'F', John = 'M'))
```
\>\> [['david', 'Mary', 'John'], ['M', 'F', 'M']]

### Lambda函式可以使用單一陳述句建立簡單匿名函式，簡單方便用完即丟

```python
def squareA(num):
    result = num**2
    return result
print(squareA(2))
```
\>\> 4

```python
def squareB(num):
    return num**2
print(squareB(2))
```
\>\> 4

```python
def squareC(num): return num**2
print(squareC(2))
```
\>\> 4

```python
squareD = lambda num:num*2
print(squareD(2))
```
\>\> 4

```python
even = lambda x: x%2==0
print(even(3))
print(even(4))
```
\>\> False
\>\> Ture

```python
first = lambda s: s[0]
print(first('hello'))
```
\>\> 'h'

```python
rev = lambda s: s[::-1]
print(rev('hello'))
```
\>\> 'olleh'

```python
adder = lambda x,y: x+y
print(adder(2,3))
```
\>\> 5

```python
x = 25

def printer():
    x = 50
    return x
print(x)
print(printer())
```
\>\> 25
\>\> 50

```python
x = 100
f = lambda x: x**2
print(f(5))

#可以發現函式裡面的 x，不會跟local的 x衝突
````
\>\> 25

```python
name = 'This is a global name'

def greet():
    name = 'Sammy'
    def hello():
        print('Hello' + name)
    hello()

greet()
print(name)

#可以發現外面print(name)是取外面的name
#greet函式hello()後會變成
#def greet():
#    name = 'Sammy'
#    print('Hello' + name)
#所以name就會帶入Sammy
```
\>\> HelloSammy
\>\> This is a global name

```python
x = 50

def func(x):
    print('x is', x)
    x = 2
    print('Changed local x to', x)

func(x)
print('x is still', x)
```
\>\> x is 50
\>\> Changed local x to 2
\>\> x is still 50

```python
x = 50

def funcA():
    global x
    print('This function is now using the global x!')
    print('Because of global x is:', x)
    x = 2
    print('Ran func(), changed global x to', x)

print('Before calling funcA(), x is:', x)
funcA()
print(x)
print('Value of x (outside of funcA()) is:',x)

#因為x使用了global，所以在函式中改動了 x，也會連帶影響函式外的 x
```
\>\> Before calling funcA(), x is: 50
\>\> This function is now using the global x!
\>\> Because of global x is: 50
\>\> Ran func(), changed global x to 2
\>\> 2
\>\> Value of x (outside of funcA()) is: 2

## 類別（Class）
物件是就由建構子 \_\_init\_\_ 建立物件，使用者可以在類別中使用 def \_\_init\_\_

```python
class Dog(object):
    def __init__(self,breed):
        self.breed = breed
sam = Dog(breed = 'Lab')
frank = Dog(breed = 'Huskie')
print(sam.breed)
print(frank.breed)
```
\>\> Lab
\>\> Huskie

```python
lass DogA(object):
    species = 'mammal' #類別屬性
    def __init__(self, breed, name):
        self.breed = breed
        self.name = name
sam = DogA('Lab','Sam')
print(sam.name)
print(sam.species)
print(DogA.species) #類別屬性是公開的
```
\>\> Sam
\>\> mammal
\>\> mammal

```python
class Circle(object):
    pi = 3.14  #類別屬性是公開的
    def __init__(self, radius = 1):
        self.radius = radius
    def area(self):
        return self.radius * self.radius * Circle.pi
    def setRadius(self, radius):
        self.radius = radius
    def getRadius(self):
        return self.radius

c = Circle()
print(c.pi)
print(c.radius)
c.setRadius(2)
print(c.radius)
print(c.getRadius())
print(c.area())
```
\>\> 3.14
\>\> 1
\>\> 2
\>\> 2
\>\> 12.56
- 當然 pi 就是**公開的類別屬性** `.pi` 都會得到 3.14
- 從一開始可以發現 \_\_init\_\_ 的 radius 可以設定預設值 = 1，所以沒有在 objec t內設定數字的話，radius 就等於 1
- 而後用 c.setRadius(2)，radius 就帶入 2，也就變成 c.radius = radius，這時就代表 self.radius 也等於 2
- 當然c.getRadius()就可會回傳 self.radius，也就等於 2

### 繼承（inheritance）
當**新類別與既有的類別有很多共通方法時**，就能用繼承。被繼承為 父類別（superclass），繼承為 子類別（subclass）

```python
class Animal(object):
    def __init__(self):
        print('Animal created')
        
    def whoAmi(self):
        print('Animal')
        
    def eat(self):
        print('Eating')

a = Animal()
a.eat()
```
\>\> Animal created
\>\> Eating

以上是 Animal 類別，以下是建立 Dog 類別並繼承 Animal 類別，只要將（object）改成（Animal），並且可以發現能夠改寫類別內的方法（函式），更能夠增加新方法

```python
class Dog(Animal):
    def __init__(self):
        Animal.__init__(self)
        print('Dog created')
        
    def whoAmi(self):
        print('Dog')
        
    def bark(self):
        print('woof!')

d = Dog()
d.whoAmi()
d.eat() #
d.bark()
```
\>\> Animal created
\>\> Dog created
\>\> Dog
\>\> Eating
\>\> woof!

### 類別的一些特殊方法名稱及用法
參考 [這裡2](https://openhome.cc/Gossip/Python/SpecialMethodNames.html)，以下只擷取特殊方法及定義，剩下的請閱讀參考網站

```python
class Rational:
    def __init__(self, n, d):  # 物件建立之後所要建立的初始化動作
        self.numer = n
        self.denom = d
    
    def __str__(self):   # 定義物件的字串描述
        return str(self.numer) + '/' + str(self.denom)
    
    def __add__(self, that):  # 定義 + 運算
        return Rational(self.numer * that.denom + that.numer * self.denom, 
                        self.denom * that.denom)
    
    def __sub__(self, that):  # 定義 - 運算
        return Rational(self.numer * that.denom - that.numer * self.denom,
                        self.denom * that.denom)
                           
    def __mul__(self, that):  # 定義 * 運算
        return Rational(self.numer * that.numer, 
                        self.denom * that.denom)
        
    def __truediv__(self, that):   # 定義 / 運算
        return Rational(self.numer * that.denom,
                        self.denom * that.denom)

    def __eq__(self, that):   # 定義 == 運算
        return self.numer * that.denom == that.numer * self.denom

x = Rational(1, 2)
y = Rational(2, 3)
z = Rational(2, 3)
print(x)
print(y)
print(x + y)
print(x - y)
print(x * y)
print(x / y)
print(x == y)
print(y == z)
```
\>\> 1/2
\>\> 2/3
\>\> 7/6
\>\> -1/6
\>\> 2/6
\>\> 3/6
\>\> False
\>\> True
-  \_\_init\_\_() 定義物件建立後要執行的初始化過程，相對它的是 \_\_del\_\_() 方法，在物件被回收前會被執行（因為回收物件的時間不一定，所以不建議用在要求立即性的情況）
- 常見的+、-、*、/、==等操作，則分別是由 \_\_add\_\_()、\_\_sub\_\_()、\_\_mul\_\_()、\_\_truediv\_\_()（//則是由 \_\_floordiv\_\_() 定義）與 \_\_eq\_\_() 定義
- \_\_str\_\_() 用來定義傳回物件描述字串，通常用來描述的字串是對使用者友善的說明文字
- 如果要定義對開發人員較有意義的描述，例如傳回產生實例的類別名稱之類的，則可以定義 \_\_repr\_\_()

### 備註
因為 Markdown 內特殊字元皆為特殊指令，如**底線(\_)**、**但星號(\*)** 等等，**單個為斜體、成對為粗體**，如果文章中展示出這些特殊字元，就在特殊字元前加上 \ ，就能乖乖展示出來

## 模組（Module）
- 在終端機中 `python3 -m pip install ...` 中的 -m 就是模組 Module
- Python 模組只是以 .py 做為副檔名的 Python 程式
- 可以在編輯器中使用 `import` 來匯入模組，並且重複 `import` 不會導致函式被重複宣告
- 可透過 `print(dir(mouldname))` 來探索模組內的方法(函式)、透過 `help(pandas.mouldname)` 來看說明

譬如建立一個 sample.py 即建立了一個模組叫做 sample，當寫一個專案 code.py 時，就可在編輯時 `import` 進來用（注意：兩者在同一個目錄下），並且可以發現 sample.py 內的 var 類似 **類別 class** 公開的類別屬性，直接在 專案code 裡用 sample.var 就能取用，而 sample.py 內的 `f()` 方法，在 專案code 用 `sample.f()`
![mould_ex](https://myfriends1033.github.io/article/photo//M1.PNG)


## 套件（Package）




### [<< Back](https://myfriends1033.github.io/)
