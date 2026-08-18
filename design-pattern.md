# أنماط التصميم (Design Patterns) في Python

شرح لأهم أنماط التصميم البرمجية الثلاثة (الإنشائية، الهيكلية، السلوكية) مع مثال Python لكل نمط.

---

## 📦 الأنماط الإنشائية (Creational Patterns)

تهتم بطريقة **إنشاء الكائنات** بشكل مرن ومناسب للسياق.

### 1. Singleton
يضمن وجود **نسخة واحدة فقط** من الكلاس في كامل البرنامج، مع نقطة وصول عامة لها.

```python
class Singleton:
    _instance = None

    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

s1 = Singleton()
s2 = Singleton()
print(s1 is s2)  # True
```

### 2. Factory Method
يفوّض عملية إنشاء الكائنات إلى **دالة مصنع (factory)** بدل استدعاء الكونستركتور مباشرة، ليتحكم الكلاس الفرعي بنوع الكائن الناتج.

```python
from abc import ABC, abstractmethod

class Animal(ABC):
    @abstractmethod
    def speak(self):
        pass

class Dog(Animal):
    def speak(self):
        return "Woof!"

class Cat(Animal):
    def speak(self):
        return "Meow!"

class AnimalFactory:
    def create_animal(self, kind: str) -> Animal:
        if kind == "dog":
            return Dog()
        elif kind == "cat":
            return Cat()
        raise ValueError("نوع غير معروف")

factory = AnimalFactory()
print(factory.create_animal("dog").speak())  # Woof!
```

### 3. Abstract Factory
مصنع ينتج **عائلة من الكائنات المترابطة** دون تحديد كلاساتها الملموسة.

```python
from abc import ABC, abstractmethod

class Button(ABC):
    @abstractmethod
    def render(self): pass

class Checkbox(ABC):
    @abstractmethod
    def render(self): pass

class WindowsButton(Button):
    def render(self): return "زر ويندوز"

class WindowsCheckbox(Checkbox):
    def render(self): return "خانة اختيار ويندوز"

class MacButton(Button):
    def render(self): return "زر ماك"

class MacCheckbox(Checkbox):
    def render(self): return "خانة اختيار ماك"

class GUIFactory(ABC):
    @abstractmethod
    def create_button(self) -> Button: pass
    @abstractmethod
    def create_checkbox(self) -> Checkbox: pass

class WindowsFactory(GUIFactory):
    def create_button(self): return WindowsButton()
    def create_checkbox(self): return WindowsCheckbox()

class MacFactory(GUIFactory):
    def create_button(self): return MacButton()
    def create_checkbox(self): return MacCheckbox()

factory = WindowsFactory()
print(factory.create_button().render())  # زر ويندوز
```

### 4. Builder
يفصل بناء كائن **معقد** خطوة بخطوة عن تمثيله النهائي، مما يسمح بإنشاء نسخ مختلفة بنفس عملية البناء.

```python
class House:
    def __init__(self):
        self.parts = []

    def add(self, part):
        self.parts.append(part)

    def __str__(self):
        return " + ".join(self.parts)

class HouseBuilder:
    def __init__(self):
        self.house = House()

    def build_walls(self):
        self.house.add("جدران")
        return self

    def build_roof(self):
        self.house.add("سقف")
        return self

    def build_garden(self):
        self.house.add("حديقة")
        return self

    def get_result(self) -> House:
        return self.house

builder = HouseBuilder()
house = builder.build_walls().build_roof().get_result()
print(house)  # جدران + سقف
```

### 5. Prototype
ينسخ كائنات موجودة بدل إنشائها من الصفر، خصوصاً عندما يكون الإنشاء مكلفاً.

```python
import copy

class Sheep:
    def __init__(self, name, color):
        self.name = name
        self.color = color

    def clone(self):
        return copy.deepcopy(self)

original = Sheep("Dolly", "أبيض")
clone = original.clone()
clone.name = "Dolly 2"
print(original.name, clone.name)  # Dolly Dolly 2
```

---

## 🧩 الأنماط الهيكلية (Structural Patterns)

تهتم بتنظيم العلاقات بين الكلاسات والكائنات لتكوين **هياكل أكبر وأكثر مرونة**.

### 6. Adapter
يحوّل واجهة كلاس معين إلى واجهة أخرى متوقعة من طرف العميل، ليتمكن كلاسان غير متوافقين من العمل معاً.

```python
class EuropeanSocket:
    def voltage(self):
        return 230

class USASocket:
    def voltage(self):
        return 120

class Adapter:
    def __init__(self, european_socket: EuropeanSocket):
        self.european_socket = european_socket

    def voltage(self):
        return self.european_socket.voltage() / 1.9  # تحويل تقريبي

socket = Adapter(EuropeanSocket())
print(socket.voltage())  # ~121
```

### 7. Bridge
يفصل **التجريد (Abstraction)** عن **التنفيذ (Implementation)** بحيث يمكن تطوير كل منهما بشكل مستقل.

```python
class Device(ABC):
    @abstractmethod
    def turn_on(self): pass

class TV(Device):
    def turn_on(self): return "التلفاز اشتغل"

class Radio(Device):
    def turn_on(self): return "الراديو اشتغل"

class RemoteControl:
    def __init__(self, device: Device):
        self.device = device

    def press_power(self):
        return self.device.turn_on()

remote = RemoteControl(TV())
print(remote.press_power())  # التلفاز اشتغل
```

### 8. Composite
يعامل الكائنات المفردة والمجموعات المركبة منها **بنفس الطريقة**، عبر بنية شجرية.

```python
class Employee(ABC):
    @abstractmethod
    def show(self, indent=0): pass

class Developer(Employee):
    def __init__(self, name):
        self.name = name

    def show(self, indent=0):
        print(" " * indent + f"- مطور: {self.name}")

class Team(Employee):
    def __init__(self, name):
        self.name = name
        self.members = []

    def add(self, member: Employee):
        self.members.append(member)

    def show(self, indent=0):
        print(" " * indent + f"+ فريق: {self.name}")
        for m in self.members:
            m.show(indent + 2)

team = Team("الفريق الرئيسي")
team.add(Developer("أحمد"))
sub_team = Team("فريق فرعي")
sub_team.add(Developer("سارة"))
team.add(sub_team)
team.show()
```

### 9. Decorator
يضيف سلوكيات جديدة لكائن معين **ديناميكياً** دون تعديل الكلاس الأصلي.

```python
class Coffee:
    def cost(self):
        return 2

class MilkDecorator:
    def __init__(self, coffee):
        self.coffee = coffee

    def cost(self):
        return self.coffee.cost() + 0.5

class SugarDecorator:
    def __init__(self, coffee):
        self.coffee = coffee

    def cost(self):
        return self.coffee.cost() + 0.2

coffee = SugarDecorator(MilkDecorator(Coffee()))
print(coffee.cost())  # 2.7
```

### 10. Facade
يوفر **واجهة مبسطة** لنظام فرعي معقد يتكون من عدة كلاسات.

```python
class CPU:
    def start(self): print("تشغيل المعالج")

class Memory:
    def load(self): print("تحميل الذاكرة")

class HardDrive:
    def read(self): print("قراءة القرص الصلب")

class ComputerFacade:
    def __init__(self):
        self.cpu = CPU()
        self.memory = Memory()
        self.hd = HardDrive()

    def start(self):
        self.cpu.start()
        self.memory.load()
        self.hd.read()
        print("الحاسوب جاهز")

computer = ComputerFacade()
computer.start()
```

### 11. Flyweight
يقلل استهلاك الذاكرة عبر **مشاركة** الجزء المشترك (الحالة الداخلية) بين عدة كائنات متشابهة.

```python
class TreeType:
    _types = {}

    def __new__(cls, name, color):
        key = (name, color)
        if key not in cls._types:
            cls._types[key] = super().__new__(cls)
        return cls._types[key]

    def __init__(self, name, color):
        self.name = name
        self.color = color

class Tree:
    def __init__(self, x, y, tree_type: TreeType):
        self.x, self.y = x, y
        self.type = tree_type

t1 = TreeType("بلوط", "أخضر")
t2 = TreeType("بلوط", "أخضر")
print(t1 is t2)  # True - نفس الكائن مُشترك
```

### 12. Proxy
يوفّر كائناً بديلاً يتحكم في الوصول لكائن آخر (تأجيل التحميل، صلاحيات، تسجيل...).

```python
class RealImage:
    def __init__(self, filename):
        self.filename = filename
        print(f"تحميل الصورة {filename}")

    def display(self):
        print(f"عرض {self.filename}")

class ProxyImage:
    def __init__(self, filename):
        self.filename = filename
        self._real_image = None

    def display(self):
        if self._real_image is None:
            self._real_image = RealImage(self.filename)  # تحميل كسول
        self._real_image.display()

img = ProxyImage("photo.png")
img.display()  # يحمّل ثم يعرض
img.display()  # يعرض فقط بدون إعادة تحميل
```

---

## 🔄 الأنماط السلوكية (Behavioral Patterns)

تهتم بطريقة **تواصل وتفاعل الكائنات** فيما بينها وتوزيع المسؤوليات.

### 13. Chain of Responsibility
يمرر الطلب عبر سلسلة من المعالجات حتى يجد من يستطيع التعامل معه.

```python
class Handler(ABC):
    def __init__(self):
        self.next = None

    def set_next(self, handler):
        self.next = handler
        return handler

    @abstractmethod
    def handle(self, request): pass

class LowSupport(Handler):
    def handle(self, request):
        if request <= 10:
            return f"LowSupport عالج الطلب {request}"
        return self.next.handle(request) if self.next else "لم يُعالج"

class HighSupport(Handler):
    def handle(self, request):
        return f"HighSupport عالج الطلب {request}"

low = LowSupport()
high = HighSupport()
low.set_next(high)
print(low.handle(5))    # LowSupport
print(low.handle(50))   # HighSupport
```

### 14. Command
يحوّل الطلب إلى كائن مستقل يحتوي كل معلوماته، مما يسمح بتأجيل التنفيذ أو التراجع عنه أو تسجيله.

```python
class Light:
    def on(self): print("الضوء اشتغل")
    def off(self): print("الضوء انطفأ")

class Command(ABC):
    @abstractmethod
    def execute(self): pass

class LightOnCommand(Command):
    def __init__(self, light: Light):
        self.light = light
    def execute(self):
        self.light.on()

class RemoteControl:
    def submit(self, command: Command):
        command.execute()

remote = RemoteControl()
remote.submit(LightOnCommand(Light()))
```

### 15. Interpreter
يعرّف تمثيلاً لقواعد لغة معينة، مع مفسّر يقيّم الجمل المكتوبة بتلك اللغة.

```python
class Expression(ABC):
    @abstractmethod
    def interpret(self): pass

class Number(Expression):
    def __init__(self, value):
        self.value = value
    def interpret(self):
        return self.value

class Add(Expression):
    def __init__(self, left, right):
        self.left, self.right = left, right
    def interpret(self):
        return self.left.interpret() + self.right.interpret()

expr = Add(Number(5), Number(3))
print(expr.interpret())  # 8
```

### 16. Iterator
يوفّر طريقة موحدة للمرور على عناصر مجموعة دون كشف تفاصيل بنيتها الداخلية.

```python
class NameCollection:
    def __init__(self):
        self.names = []

    def add(self, name):
        self.names.append(name)

    def __iter__(self):
        return iter(self.names)

collection = NameCollection()
collection.add("علي")
collection.add("فاطمة")
for name in collection:
    print(name)
```

### 17. Mediator
ينظم التواصل بين مجموعة كائنات عبر **كائن وسيط** بدل تواصلها المباشر فيما بينها.

```python
class ChatRoom:
    def show_message(self, user, message):
        print(f"[{user}]: {message}")

class User:
    def __init__(self, name, chatroom: ChatRoom):
        self.name = name
        self.chatroom = chatroom

    def send(self, message):
        self.chatroom.show_message(self.name, message)

room = ChatRoom()
u1 = User("عمر", room)
u1.send("السلام عليكم")
```

### 18. Memento
يسمح بحفظ واستعادة الحالة الداخلية لكائن دون كشف تفاصيلها للخارج (تراجع/Undo).

```python
class Memento:
    def __init__(self, state):
        self._state = state

class Editor:
    def __init__(self):
        self.content = ""

    def type(self, words):
        self.content += words

    def save(self) -> Memento:
        return Memento(self.content)

    def restore(self, memento: Memento):
        self.content = memento._state

editor = Editor()
editor.type("مرحبا")
checkpoint = editor.save()
editor.type(" بالعالم")
print(editor.content)   # مرحبا بالعالم
editor.restore(checkpoint)
print(editor.content)   # مرحبا
```

### 19. Observer
يعرّف علاقة "واحد لمتعدد": عندما يتغير كائن (Subject)، تُعلَم كل الكائنات المشتركة (Observers) وتتحدث تلقائياً.

```python
class Subject:
    def __init__(self):
        self._observers = []

    def subscribe(self, observer):
        self._observers.append(observer)

    def notify(self, event):
        for obs in self._observers:
            obs.update(event)

class Observer(ABC):
    @abstractmethod
    def update(self, event): pass

class EmailObserver(Observer):
    def update(self, event):
        print(f"إرسال إيميل بخصوص: {event}")

subject = Subject()
subject.subscribe(EmailObserver())
subject.notify("طلب جديد")
```

### 20. State
يسمح لكائن بتغيير سلوكه عندما تتغير حالته الداخلية، وكأنه يغيّر كلاسه.

```python
class State(ABC):
    @abstractmethod
    def handle(self): pass

class StartState(State):
    def handle(self):
        return "بدأ التشغيل"

class StopState(State):
    def handle(self):
        return "توقف التشغيل"

class Machine:
    def __init__(self):
        self.state: State = StartState()

    def set_state(self, state: State):
        self.state = state

    def request(self):
        return self.state.handle()

machine = Machine()
print(machine.request())       # بدأ التشغيل
machine.set_state(StopState())
print(machine.request())       # توقف التشغيل
```

### 21. Strategy
يعرّف مجموعة خوارزميات قابلة للتبديل فيما بينها، ويجعلها قابلة للاستبدال أثناء التشغيل.

```python
class Strategy(ABC):
    @abstractmethod
    def calculate(self, a, b): pass

class AddStrategy(Strategy):
    def calculate(self, a, b): return a + b

class MultiplyStrategy(Strategy):
    def calculate(self, a, b): return a * b

class Calculator:
    def __init__(self, strategy: Strategy):
        self.strategy = strategy

    def execute(self, a, b):
        return self.strategy.calculate(a, b)

calc = Calculator(AddStrategy())
print(calc.execute(3, 4))  # 7
calc.strategy = MultiplyStrategy()
print(calc.execute(3, 4))  # 12
```

### 22. Template Method
يعرّف الهيكل العام لخوارزمية في دالة رئيسية بالكلاس الأب، ويترك بعض الخطوات لتُنفّذ من الكلاسات الفرعية.

```python
class DataProcessor(ABC):
    def process(self):  # Template Method
        self.read_data()
        self.transform()
        self.save()

    def read_data(self):
        print("قراءة البيانات")

    @abstractmethod
    def transform(self): pass

    def save(self):
        print("حفظ البيانات")

class CSVProcessor(DataProcessor):
    def transform(self):
        print("تحويل بيانات CSV")

CSVProcessor().process()
```

### 23. Visitor
يفصل خوارزمية معينة عن الكلاسات التي تعمل عليها، بحيث يمكن إضافة عمليات جديدة دون تعديل تلك الكلاسات.

```python
class Shape(ABC):
    @abstractmethod
    def accept(self, visitor): pass

class Circle(Shape):
    def accept(self, visitor):
        return visitor.visit_circle(self)

class Square(Shape):
    def accept(self, visitor):
        return visitor.visit_square(self)

class AreaVisitor:
    def visit_circle(self, circle):
        return "حساب مساحة الدائرة"

    def visit_square(self, square):
        return "حساب مساحة المربع"

shapes = [Circle(), Square()]
visitor = AreaVisitor()
for shape in shapes:
    print(shape.accept(visitor))
```

---

## 📚 ملخص سريع

| الفئة | الغرض الأساسي |
|---|---|
| إنشائية | التحكم في طريقة إنشاء الكائنات |
| هيكلية | تنظيم العلاقات بين الكائنات والكلاسات |
| سلوكية | تنظيم التواصل وتوزيع المسؤوليات بين الكائنات |
