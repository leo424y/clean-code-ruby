# clean-code-ruby

Clean Code concepts adapted for Ruby.

Inspired by [clean-code-javascript](https://github.com/ryanmcdermott/clean-code-javascript).

*Note: This is still a WIP. The examples are largely ported over from JavaScript so they may not be idiomatic. Feel free to point out any non-idiomatic Ruby code by submitting an issue and I'll correct it right away. Also, pull requests are always welcome!*

## Table of Contents
  1. [Introduction](#introduction)
  2. [Variables](#variables)
  3. [Functions](#functions)
  4. [Objects and Data Structures](#objects-and-data-structures)
  5. [Classes](#classes)
  6. [SOLID](#solid)
  7. [Testing](#testing)
  9. [Error Handling](#error-handling)
  10. [Formatting](#formatting)
  11. [Comments](#comments)
  12. [Translations](#translations)

## Introduction
![Humorous image of software quality estimation as a count of how many expletives
you shout when reading code](http://www.osnews.com/images/comics/wtfm.jpg)

Software engineering principles, from Robert C. Martin's book
[*Clean Code*](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882),
adapted for Ruby. This is not a style guide. It's a guide to producing
[readable, reusable, and refactorable](https://github.com/ryanmcdermott/3rs-of-software-architecture) software in Ruby.

Not every principle herein has to be strictly followed, and even fewer will be
universally agreed upon. These are guidelines and nothing more, but they are
ones codified over many years of collective experience by the authors of
*Clean Code*.

Our craft of software engineering is just a bit over 50 years old, and we are
still learning a lot. When software architecture is as old as architecture
itself, maybe then we will have harder rules to follow. For now, let these
guidelines serve as a touchstone by which to assess the quality of the
Ruby code that you and your team produce.

One more thing: knowing these won't immediately make you a better software
developer, and working with them for many years doesn't mean you won't make
mistakes. Every piece of code starts as a first draft, like wet clay getting
shaped into its final form. Finally, we chisel away the imperfections when
we review it with our peers. Don't beat yourself up for first drafts that need
improvement. Beat up the code instead!

## **Variables**
### 變數用有意義且可發音的字

**Bad:**
```ruby
yyyymmdstr = Time.now.strftime('%Y/%m/%d')
```

**Good:**
```ruby
current_date = Time.now.strftime('%Y/%m/%d')
```
**[⬆ back to top](#table-of-contents)**

### 同類型的變數用同一個字詞

Pick one word for the concept and stick to it.


**Bad:**
```ruby
user_info
user_data
user_record

starts_at
start_at
start_time
```

**Good:**
```ruby
user

starts_at
```
**[⬆ back to top](#table-of-contents)**

### 使用常數且易於搜尋
你讀程式的時間比寫的時間多，讓你的變數能被搜尋且好懂

**Bad:**
```ruby
# What the heck is 86400 for?
status = Timeout::timeout(86_400) do
  # ...
end
```

**Good:**
```ruby
# Declare them as capitalized globals.
SECONDS_IN_A_DAY = 86_400

status = Timeout::timeout(SECONDS_IN_A_DAY) do
  # ...
end
```
**[⬆ back to top](#table-of-contents)**

### 解釋變數
**Bad:**
```ruby
address = 'One Infinite Loop, Cupertino 95014'
city_zip_code_regex = /^[^,\\]+[,\\\s]+(.+?)\s*(\d{5})?$/
save_city_zip_code(city_zip_code_regex.match(address)[1], city_zip_code_regex.match(address)[2])
```

**Good:**
```ruby
address = 'One Infinite Loop, Cupertino 95014'
city_zip_code_regex = /^[^,\\]+[,\\\s]+(.+?)\s*(\d{5})?$/
_, city, zip_code = city_zip_code_regex.match(address).to_a
save_city_zip_code(city, zip_code)
```
**[⬆ back to top](#table-of-contents)**

### 明確不靠意會

**Bad:**
```ruby
locations = ['Austin', 'New York', 'San Francisco']
locations.each do |l|
  do_stuff
  do_some_other_stuff
  # ...
  # ...
  # ...
  # Wait, what is `l` for again?
  dispatch(l)
end
```

**Good:**
```ruby
locations = ['Austin', 'New York', 'San Francisco']
locations.each do |location|
  do_stuff
  do_some_other_stuff
  # ...
  # ...
  # ...
  dispatch(location)
end
```
**[⬆ back to top](#table-of-contents)**

### 別重工

**Bad:**
```ruby
car = {
  car_make: 'Honda',
  car_model: 'Accord',
  car_color: 'Blue'
}

def paint_car(car)
  car[:car_color] = 'Red'
end
```

**Good:**
```ruby
car = {
  make: 'Honda',
  model: 'Accord',
  color: 'Blue'
}

def paint_car(car)
  car[:color] = 'Red'
end
```
**[⬆ back to top](#table-of-contents)**

### 給定預設值
false 或 nil 則不會被轉成預設

**Bad:**
```ruby
def create_micro_brewery(name)
  brewery_name = name || 'Hipster Brew Co.'
  # ...
end
```

**Good:**
```ruby
def create_micro_brewery(brewery_name = 'Hipster Brew Co.')
  # ...
end
```
**[⬆ back to top](#table-of-contents)**

## **Functions**
### 函式參數最多兩個，或使 Simbol

**Bad:**
```ruby
def create_menu(title, body, button_text, cancellable)
  # ...
end
```

**Good:**
```ruby
def create_menu(title:, body:, button_text:, cancellable:)
  # ...
end

create_menu(
  title: 'Foo',
  body: 'Bar',
  button_text: 'Baz',
  cancellable: true
)
```
**[⬆ back to top](#table-of-contents)**


### 單一職責

**Bad:**
```ruby
def email_clients(clients)
  clients.each do |client|
    client_record = database.lookup(client)
    email(client) if client_record.active?
  end
end
```

**Good:**
```ruby
def email_active_clients(clients)
  clients
    .select(&method(:active_client?))
    .each(&method(:email))
end

def active_client?(client)
  client_record = database.lookup(client)
  client_record.active?
end
```
**[⬆ back to top](#table-of-contents)**

### 明確表示職責

**Bad:**
```ruby
def add_to_date(date, month)
  # ...
end

date = DateTime.now

# It's hard to to tell from the function name what is added
add_to_date(date, 1)
```

**Good:**
```ruby
def add_month_to_date(date, month)
  # ...
end

date = DateTime.now
add_month_to_date(date, 1)
```
**[⬆ back to top](#table-of-contents)**

### 抽象最多一層
以便測試

**Bad:**
```ruby
def interpret(code)
  regexes = [
    # ...
  ]

  statements = code.split(' ')
  tokens = []
  regexes.each do |regex|
    statements.each do |statement|
      # ...
    end
  end

  ast = []
  tokens.each do |token|
    # lex...
  end

  result = []
  ast.each do |node|
    # result.push(...)
  end

  result
end
```

**Good:**
```ruby
def interpet(code)
  tokens = tokenize(code)
  ast = lex(tokens)
  parse(ast)
end

def tokenize(code)
  regexes = [
    # ...
  ]

  statements = code.split(' ')
  tokens = []
  regexes.each do |regex|
    statements.each do |statement|
      # tokens.push(...)
    end
  end

  tokens
end

def lex(tokens)
  ast = []
  tokens.each do |token|
    # ast.push(...)
  end

  ast
end

def parse(ast)
  result = []
  ast.each do |node|
    # result.push(...)
  end

  result
end
```
**[⬆ back to top](#table-of-contents)**

### 別重覆程式

**Bad:**
```ruby
def show_developer_list(developers)
  developers.each do |developer|
    data = {
      expected_salary: developer.expected_salary,
      experience: developer.experience,
      github_link: developer.github_link
    }

    render(data)
  end
end

def show_manager_list(managers)
  managers.each do |manager|
    data = {
      expected_salary: manager.expected_salary,
      experience: manager.experience,
      portfolio: manager.mba_projects
    }

    render(data)
  end
end
```

**Good:**
```ruby
def show_employee_list(employees)
  employees.each do |employee|
    data = {
      expected_salary: employee.expected_salary,
      experience: employee.experience
    }

    case employee.type
    when 'manager'
      data[:portfolio] = employee.mba_projects
    when 'developer'
      data[:github_link] = employee.github_link
    end

    render(data)
  end
end
```
**[⬆ back to top](#table-of-contents)**

### 參數不要用旗標
旗標代表會做兩件事以上，一個函數只做一件事

**Bad:**
```ruby
def create_file(name, temp)
  if temp
    fs.create("./temp/#{name}")
  else
    fs.create(name)
  end
end
```

**Good:**
```ruby
def create_file(name)
  fs.create(name)
end

def create_temp_file(name)
  create_file("./temp/#{name}")
end
```
**[⬆ back to top](#table-of-contents)**

### 避免副作用
一個變數指一件事，函數作用後是另件事了

**Bad:**
```ruby
# Global variable referenced by following function.
# If we had another function that used this name, now it'd be an array and it could break it.
$name = 'Ryan McDermott'

def split_into_first_and_last_name
  $name = $name.split(' ')
end

split_into_first_and_last_name()

puts $name # ['Ryan', 'McDermott']
```

**Good:**
```ruby
def split_into_first_and_last_name(name)
  name.split(' ')
end

name = 'Ryan McDermott'
new_name = split_into_first_and_last_name(name)

puts name # 'Ryan McDermott'
puts new_name # ['Ryan', 'McDermott']
```
**[⬆ back to top](#table-of-contents)**

### 再避免副作用
再次避免搞混變數及函式

**Bad:**
```ruby
def add_item_to_cart(cart, item)
  cart.push(item: item, time: Time.now)
end
```

**Good:**
```ruby
def add_item_to_cart(cart, item)
  cart + [{ item: item, time: Time.now }]
end
```
**[⬆ back to top](#table-of-contents)**

### 函數式地開發吧
讓你好好測試

**Bad:**
```ruby
programmer_output = [
  {
    name: 'Uncle Bobby',
    lines_of_code: 500
  }, {
    name: 'Suzie Q',
    lines_of_code: 1500
  }, {
    name: 'Jimmy Gosling',
    lines_of_code: 150
  }, {
    name: 'Grace Hopper',
    lines_of_code: 1000
  }
]

total_output = 0

programmer_output.each do |output|
  total_output += output[:lines_of_code]
end
```

**Good:**
```ruby
programmer_output = [
  {
    name: 'Uncle Bobby',
    lines_of_code: 500
  }, {
    name: 'Suzie Q',
    lines_of_code: 1500
  }, {
    name: 'Jimmy Gosling',
    lines_of_code: 150
  }, {
    name: 'Grace Hopper',
    lines_of_code: 1000
  }
]

INITIAL_VALUE = 0

total_output = programmer_output
               .reduce(INITIAL_VALUE) { |acc, output| acc + output[:lines_of_code] }
```
**[⬆ back to top](#table-of-contents)**

### 包裝判斷式

**Bad:**
```ruby
if params[:message].present? && params[:recipient].present?
  # ...
end
```

**Good:**
```ruby
def send_message?(params)
  params[:message].present? && params[:recipient].present?
end

if send_message?(params)
  # ...
end
```
**[⬆ back to top](#table-of-contents)**

### 避免反判斷

**Bad:**
```ruby
if !genres.blank?
  # ...
end
```

**Good:**
```ruby
unless genres.blank?
  # ...
end

# or

if genres.present?
  # ...
end
```
**[⬆ back to top](#table-of-contents)**

### 抽象取代判斷

**Bad:**
```ruby
class Airplane
  # ...
  def cruising_altitude
    case @type
    when '777'
      max_altitude - passenger_count
    when 'Air Force One'
      max_altitude
    when 'Cessna'
      max_altitude - fuel_expenditure
    end
  end
end
```

**Good:**
```ruby
class Airplane
  # ...
end

class Boeing777 < Airplane
  # ...
  def cruising_altitude
    max_altitude - passenger_count
  end
end

class AirForceOne < Airplane
  # ...
  def cruising_altitude
    max_altitude
  end
end

class Cessna < Airplane
  # ...
  def cruising_altitude
    max_altitude - fuel_expenditure
  end
end
```
**[⬆ back to top](#table-of-contents)**

### 避免型別確認

**Bad:**
```ruby
def travel_to_texas(vehicle)
  if vehicle.is_a?(Bicycle)
    vehicle.pedal(@current_location, Location.new('texas'))
  elsif vehicle.is_a?(Car)
    vehicle.drive(@current_location, Location.new('texas'))
  end
end
```

**Good:**
```ruby
def travel_to_texas(vehicle)
  vehicle.move(@current_location, Location.new('texas'))
end
```
**[⬆ back to top](#table-of-contents)**

### 再避免型別確認
工具 [contracts.ruby](https://github.com/egonSchiele/contracts.ruby)

**Bad:**
```ruby
def combine(val1, val2)
  if (val1.is_a?(Numeric) && val2.is_a?(Numeric)) ||
     (val1.is_a?(String) && val2.is_a?(String))
    return val1 + val2
  end

  raise 'Must be of type String or Numeric'
end
```

**Good:**
```ruby
def combine(val1, val2)
  val1 + val2
end
```
**[⬆ back to top](#table-of-contents)**

### 沒呼叫的殺掉

**Bad:**
```ruby
def old_request_module(url)
  # ...
end

def new_request_module(url)
  # ...
end

req = new_request_module(request_url)
inventory_tracker('apples', req, 'www.inventory-awesome.io')
```

**Good:**
```ruby
def new_request_module(url)
  # ...
end

req = new_request_module(request_url)
inventory_tracker('apples', req, 'www.inventory-awesome.io')
```
**[⬆ back to top](#table-of-contents)**

## **Objects and Data Structures**
### 使用 Getter, Setter, attr_accessor

**Bad:**
```ruby
def make_bank_account
  # ...

  {
    balance: 0
    # ...
  }
end

account = make_bank_account
account[:balance] = 100
account[:balance] # => 100
```

**Good:**
```ruby
class BankAccount
  def initialize
    # this one is private
    @balance = 0
  end

  # a "getter" via a public instance method
  def balance
    # do some logging
    @balance
  end

  # a "setter" via a public instance method
  def balance=(amount)
    # do some logging
    # do some validation
    @balance = amount
  end
end

account = BankAccount.new
account.balance = 100
account.balance # => 100
```

**Good:**
```ruby
class Toy
  attr_accessor :price
end

toy = Toy.new
toy.price = 50
toy.price # => 50
```

**[⬆ back to top](#table-of-contents)**


## **Classes**

### Avoid fluent interfaces
A [Fluent interface](https://en.wikipedia.org/wiki/Fluent_interface) is an object
oriented API that aims to improve the readability of the source code by using
[method chaining](https://en.wikipedia.org/wiki/Method_chaining).

While there can be some contexts, frequently builder objects, where this
pattern reduces the verbosity of the code (e.g., ActiveRecord queries),
more often it comes at some costs:

1. Breaks [Encapsulation](https://en.wikipedia.org/wiki/Encapsulation_%28object-oriented_programming%29)
2. Breaks [Decorators](https://en.wikipedia.org/wiki/Decorator_pattern)
3. Is harder to [mock](https://en.wikipedia.org/wiki/Mock_object) in a test suite
4. Makes diffs of commits harder to read

For more informations you can read the full [blog post](https://ocramius.github.io/blog/fluent-interfaces-are-evil/)
on this topic written by [Marco Pivetta](https://github.com/Ocramius).

**Bad:**
```ruby
class Car
  def initialize(make, model, color)
    @make = make
    @model = model
    @color = color
    # NOTE: Returning self for chaining
    self
  end

  def set_make(make)
    @make = make
    # NOTE: Returning self for chaining
    self
  end

  def set_model(model)
    @model = model
    # NOTE: Returning self for chaining
    self
  end

  def set_color(color)
    @color = color
    # NOTE: Returning self for chaining
    self
  end

  def save
    # save object...
    # NOTE: Returning self for chaining
    self
  end
end

car = Car.new('Ford','F-150','red')
  .set_color('pink')
  .save
```

**Good:**
```ruby
class Car
  attr_accessor :make, :model, :color

  def initialize(make, model, color)
    @make = make
    @model = model
    @color = color
  end

  def save
    # Save object...
  end
end

car = Car.new('Ford', 'F-150', 'red')
car.color = 'pink'
car.save
```
**[⬆ back to top](#table-of-contents)**

### Prefer composition over inheritance
As stated famously in [*Design Patterns*](https://en.wikipedia.org/wiki/Design_Patterns) by the Gang of Four,
you should prefer composition over inheritance where you can. There are lots of
good reasons to use inheritance and lots of good reasons to use composition.
The main point for this maxim is that if your mind instinctively goes for
inheritance, try to think if composition could model your problem better. In some
cases it can.

You might be wondering then, "when should I use inheritance?" It
depends on your problem at hand, but this is a decent list of when inheritance
makes more sense than composition:

1. Your inheritance represents an "is-a" relationship and not a "has-a"
relationship (Human->Animal vs. User->UserDetails).
2. You can reuse code from the base classes (Humans can move like all animals).
3. You want to make global changes to derived classes by changing a base class.
(Change the caloric expenditure of all animals when they move).

**Bad:**
```ruby
class Employee
  def initialize(name, email)
    @name = name
    @email = email
  end

  # ...
end

# Bad because Employees "have" tax data. EmployeeTaxData is not a type of Employee
class EmployeeTaxData < Employee
  def initialize(ssn, salary)
    super()
    @ssn = ssn
    @salary = salary
  end

  # ...
end
```

**Good:**
```ruby
class EmployeeTaxData
  def initialize(ssn, salary)
    @ssn = ssn
    @salary = salary
  end

  # ...
end

class Employee
  def initialize(name, email)
    @name = name
    @email = email
  end

  def set_tax_data(ssn, salary)
    @tax_data = EmployeeTaxData.new(ssn, salary)
  end
  # ...
end
```
**[⬆ back to top](#table-of-contents)**

## **SOLID**
### Single Responsibility Principle (SRP) 單一職責原則
As stated in Clean Code, "There should never be more than one reason for a class
to change". It's tempting to jam-pack a class with a lot of functionality, like
when you can only take one suitcase on your flight. The issue with this is
that your class won't be conceptually cohesive and it will give it many reasons
to change. Minimizing the amount of times you need to change a class is important.
It's important because if too much functionality is in one class and you modify
a piece of it, it can be difficult to understand how that will affect other
dependent modules in your codebase.

**Bad:**
```ruby
class UserSettings
  def initialize(user)
    @user = user
  end

  def change_settings(settings)
    return unless valid_credentials?
    # ...
  end

  def valid_credentials?
    # ...
  end
end
```

**Good:**
```ruby
class UserAuth
  def initialize(user)
    @user = user
  end

  def valid_credentials?
    # ...
  end
end

class UserSettings
  def initialize(user)
    @user = user
    @auth = UserAuth.new(user)
  end

  def change_settings(settings)
    return unless @auth.valid_credentials?
    # ...
  end
end
```
**[⬆ back to top](#table-of-contents)**

### Open/Closed Principle (OCP)
As stated by [Bertrand Meyer](https://en.wikipedia.org/wiki/Bertrand_Meyer), "software entities (classes, modules, functions,
etc.) should be open for extension, but closed for modification." What does that
mean though? This principle basically states that you should allow users to
add new functionalities without changing existing code.

**Bad:**
```ruby
class Adapter
  attr_reader :name
end

class AjaxAdapter < Adapter
  def initialize
    super()
    @name = 'ajaxAdapter'
  end
end

class NodeAdapter < Adapter
  def initialize
    super()
    @name = 'nodeAdapter'
  end
end

class HttpRequester
  def initialize(adapter)
    @adapter = adapter
  end

  def fetch(url)
    adapter_name = @adapter.name

    if adapter_name == 'ajaxAdapter'
      make_ajax_call(url)
    elsif adapter_name == 'httpNodeAdapter'
      make_http_call(url)
    end
  end

  def make_ajax_call(url)
    # ...
  end

  def make_http_call(url)
    # ...
  end
end
```

**Good:**
```ruby
class Adapter
  attr_reader :name
end

class AjaxAdapter < Adapter
  def initialize
    super()
    @name = 'ajaxAdapter'
  end

  def request(url)
    # ...
  end
end

class NodeAdapter < Adapter
  def initialize
    super()
    @name = 'nodeAdapter'
  end

  def request(url)
    # ...
  end
end

class HttpRequester
  def initialize(adapter)
    @adapter = adapter
  end

  def fetch(url)
    @adapter.request(url)
  end
end
```
**[⬆ back to top](#table-of-contents)**

### Liskov Substitution Principle (LSP)
This is a scary term for a very simple concept. It's formally defined as "If S
is a subtype of T, then objects of type T may be replaced with objects of type S
(i.e., objects of type S may substitute objects of type T) without altering any
of the desirable properties of that program (correctness, task performed,
etc.)." That's an even scarier definition.

The best explanation for this is if you have a parent class and a child class,
then the base class can always be replaced by the child class without getting
incorrect results. This might still be confusing, so let's take a look at the
classic Square-Rectangle example. Mathematically, a square is a rectangle, but
if you model it using the "is-a" relationship via inheritance, you quickly
get into trouble.

**Bad:**
```ruby
class Rectangle
  def initialize
    @width = 0
    @height = 0
  end

  def color=(color)
    # ...
  end

  def render(area)
    # ...
  end

  def width=(width)
    @width = width
  end

  def height=(height)
    @height = height
  end

  def area
    @width * @height
  end
end

class Square < Rectangle
  def width=(width)
    @width = width
    @height = width
  end

  def height=(height)
    @width = height
    @height = height
  end
end

def render_large_rectangles(rectangles)
  rectangles.each do |rectangle|
    rectangle.width = 4
    rectangle.height = 5
    area = rectangle.area # BAD: Returns 25 for Square. Should be 20.
    rectangle.render(area)
  end
end

rectangles = [Rectangle.new, Rectangle.new, Square.new]
render_large_rectangles(rectangles)
```

**Good:**
```ruby
class Shape
  def color=(color)
    # ...
  end

  def render(area)
    # ...
  end
end

class Rectangle < Shape
  def initialize(width, height)
    super()
    @width = width
    @height = height
  end

  def area
    @width * @height
  end
end

class Square < Shape
  def initialize(length)
    super()
    @length = length
  end

  def area
    @length * @length
  end
end

def render_large_shapes(shapes)
  shapes.each do |shape|
    area = shape.area
    shape.render(area)
  end
end

shapes = [Rectangle.new(4, 5), Rectangle.new(4, 5), Square.new(5)]
render_large_shapes(shapes)
```
**[⬆ back to top](#table-of-contents)**

### Interface Segregation Principle (ISP)
Ruby doesn't have interfaces so this principle doesn't apply as strictly
as others. However, it's important and relevant even with Ruby's lack of
type system.

ISP states that "Clients should not be forced to depend upon interfaces that
they do not use." Interfaces are implicit contracts in Ruby because of
duck typing.

When a client depends upon a class that contains interfaces that the client does not use, but that other clients do use, then that client will be affected by the changes that those other clients force upon the class.

The following example is taken from [here](http://geekhmer.github.io/blog/2015/03/18/interface-segregation-principle-in-ruby/).

**Bad:**
```ruby
class Car
  # used by Driver
  def open
    # ...
  end

  # used by Driver
  def start_engine
    # ...
  end

  # used by Mechanic
  def change_engine
    # ...
  end
end

class Driver
  def drive
    @car.open
    @car.start_engine
  end
end

class Mechanic
  def do_stuff
    @car.change_engine
  end
end

```

**Good:**
```ruby
# used by Driver only
class Car
  def open
    # ...
  end

  def start_engine
    # ...
  end
end

# used by Mechanic only
class CarInternals
  def change_engine
    # ...
  end
end

class Driver
  def drive
    @car.open
    @car.start_engine
  end
end

class Mechanic
  def do_stuff
    @car_internals.change_engine
  end
end
```

**[⬆ back to top](#table-of-contents)**

### Dependency Inversion Principle (DIP)
This principle states two essential things:
1. High-level modules should not depend on low-level modules. Both should
depend on abstractions.
2. Abstractions should not depend upon details. Details should depend on
abstractions.

Simply put, DIP keeps high-level
modules from knowing the details of its low-level modules and setting them up.
It can accomplish this through DI. A huge benefit of this is that it reduces
the coupling between modules. Coupling is a very bad development pattern because
it makes your code hard to refactor.

As stated previously, Ruby doesn't have interfaces so the abstractions
that are depended upon are implicit contracts. That is to say, the methods
and properties that an object/class exposes to another object/class. In the
example below, the implicit contract is that any Request module for an
`InventoryTracker` will have a `request_items` method.

**Bad:**
```ruby
class InventoryRequester
  def initialize
    @req_methods = ['HTTP']
  end

  def request_item(item)
    # ...
  end
end

class InventoryTracker
  def initialize(items)
    @items = items

    # BAD: We have created a dependency on a specific request implementation.
    @requester = InventoryRequester.new
  end

  def request_items
    @items.each do |item|
      @requester.request_item(item)
    end
  end
end

inventory_tracker = InventoryTracker.new(['apples', 'bananas'])
inventory_tracker.request_items
```

**Good:**
```ruby
class InventoryTracker
  def initialize(items, requester)
    @items = items
    @requester = requester
  end

  def request_items
    @items.each do |item|
      @requester.request_item(item)
    end
  end
end

class InventoryRequesterV1
  def initialize
    @req_methods = ['HTTP']
  end

  def request_item(item)
    # ...
  end
end

class InventoryRequesterV2
  def initialize
    @req_methods = ['WS']
  end

  def request_item(item)
    # ...
  end
end

# By constructing our dependencies externally and injecting them, we can easily
# substitute our request module for a fancy new one that uses WebSockets.
inventory_tracker = InventoryTracker.new(['apples', 'bananas'], InventoryRequesterV2.new)
inventory_tracker.request_items
```
**[⬆ back to top](#table-of-contents)**

## **Testing**
推薦工具 [good coverage tool](https://coveralls.io/).

### 一個期望一句

**Bad:**
```ruby
require 'rspec'

describe 'Calculator' do
  let(:calculator) { Calculator.new }

  it 'performs addition, subtraction, multiplication and division' do
    expect(calculator.calculate('1 + 2')).to eq(3)
    expect(calculator.calculate('4 - 2')).to eq(2)
    expect(calculator.calculate('2 * 3')).to eq(6)
    expect(calculator.calculate('6 / 2')).to eq(3)
  end
end
```

**Good:**
```ruby
require 'rspec'

describe 'Calculator' do
  let(:calculator) { Calculator.new }

  it 'performs addition' do
    expect(calculator.calculate('1 + 2')).to eq(3)
  end

  it 'performs subtraction' do
    expect(calculator.calculate('4 - 2')).to eq(2)
  end

  it 'performs multiplication' do
    expect(calculator.calculate('2 * 3')).to eq(6)
  end

  it 'performs division' do
    expect(calculator.calculate('6 / 2')).to eq(3)
  end
end
```
**[⬆ back to top](#table-of-contents)**

## **Error Handling**
掌握錯誤

### 可能出錯的地方 log 且導向相關 user 或統計工具

**Bad:**
```ruby
require 'logger'

logger = Logger.new(STDOUT)

begin
  function_that_might_throw()
rescue StandardError => err
  logger.info(err)
end
```

**Good:**
```ruby
require 'logger'

logger = Logger.new(STDOUT)
# Change the logger level to ERROR to output only logs with ERROR level and above
logger.level = Logger::ERROR

begin
  function_that_might_throw()
rescue StandardError => err
  # Option 1: Only log errors
  logger.error(err)
  # Option 2: Notify end-user via an interface
  notify_user_of_error(err)
  # Option 3: Report error to a third-party service like Honeybadger
  report_error_to_service(err)
  # OR do all three!
end
```

### 說明錯誤

***Bad:***
```ruby
def initialize(user)
  fail unless user
  ...
end
```

***Good:***
```ruby
def initialize(user)
  fail ArgumentError, 'Missing user' unless user
  ...
end
```

**[⬆ back to top](#table-of-contents)**


## **Formatting**
推薦[RuboCop](https://github.com/bbatsov/rubocop)

### 大小寫統一

**Bad:**
```ruby
DAYS_IN_WEEK = 7
daysInMonth = 30

songs = ['Back In Black', 'Stairway to Heaven', 'Hey Jude']
Artists = ['ACDC', 'Led Zeppelin', 'The Beatles']

def eraseDatabase; end

def restore_database; end

class ANIMAL; end
class Alpaca; end
```

**Good:**
```ruby
DAYS_IN_WEEK = 7
DAYS_IN_MONTH = 30

SONGS = ['Back In Black', 'Stairway to Heaven', 'Hey Jude'].freeze
ARTISTS = ['ACDC', 'Led Zeppelin', 'The Beatles'].freeze

def erase_database; end

def restore_database; end

class Animal; end
class Alpaca; end
```
**[⬆ back to top](#table-of-contents)**


### 按照關聯順序排函式

**Bad:**
```ruby
class PerformanceReview
  def initialize(employee)
    @employee = employee
  end

  def lookup_peers
    db.lookup(@employee, 'peers')
  end

  def lookup_manager
    db.lookup(@employee, 'manager')
  end

  def peer_reviews
    peers = lookup_peers
    # ...
  end

  def perf_review
    peer_reviews
    manager_review
    self_review
  end

  def manager_review
    manager = lookup_manager
    # ...
  end

  def self_review
    # ...
  end
end

review = PerformanceReview.new(employee)
review.perf_review
```

**Good:**
```ruby
class PerformanceReview
  def initialize(employee)
    @employee = employee
  end

  def perf_review
    peer_reviews
    manager_review
    self_review
  end

  def peer_reviews
    peers = lookup_peers
    # ...
  end

  def lookup_peers
    db.lookup(@employee, 'peers')
  end

  def manager_review
    manager = lookup_manager
    # ...
  end

  def lookup_manager
    db.lookup(@employee, 'manager')
  end

  def self_review
    # ...
  end
end

review = PerformanceReview.new(employee)
review.perf_review
```

**[⬆ back to top](#table-of-contents)**

## **Comments**


### 沒用的 Code 就刪，git有存了

**Bad:**
```ruby
do_stuff
# do_other_stuff
# do_some_more_stuff
# do_so_much_stuff
```

**Good:**
```ruby
do_stuff
```
**[⬆ back to top](#table-of-contents)**

### Git log

**Bad:**
```ruby
# 2016-12-20: Removed monads, didn't understand them (RM)
# 2016-10-01: Improved using special monads (JP)
# 2016-02-03: Removed type-checking (LI)
# 2015-03-14: Added combine with type-checking (JR)
def combine(a, b)
  a + b
end
```

**Good:**
```ruby
def combine(a, b)
  a + b
end
```
**[⬆ back to top](#table-of-contents)**

## Translations

This is also available in other languages:

  - [Brazilian Portuguese](https://github.com/uohzxela/clean-code-ruby/blob/master/translations/pt-BR.md)

**[⬆ back to top](#table-of-contents)**
