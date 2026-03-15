# 📅 Day 22 — Ruby 심화 (클래스 & 모듈 & 예외처리 & 라이브러리)

**날짜 / 日付(ひづけ):** 2026.03.15  
**학습 시간 / 学習時間(がくしゅうじかん):** 자율 / 自律(じりつ)  
**한 줄 목표:** 클래스 상속·모듈 믹스인·예외처리 구조를 이해하고, 표준 라이브러리로 파일을 다루는 법을 익힌다.  
**一言目標(ひとこともくひょう):** クラス継承(けいしょう)・モジュールMixin・例外処理(れいがいしょり)の構造(こうぞう)を理解(りかい)し、標準(ひょうじゅん)ライブラリでファイルを扱(あつか)う方法(ほうほう)を身(み)につける。

---

## 목차 / 目次(もくじ)

1. [클래스 (Class)](#1-클래스-class)
2. [모듈 (Module)](#2-모듈-module)
3. [예외처리 (Exception)](#3-예외처리-exception)
4. [표준 라이브러리 (Library)](#4-표준-라이브러리-library)

---

## 1. 클래스 (Class)

**데이터(인스턴스 변수)와 처리(메서드)를 하나로 묶는 설계도**.  
`new` 로 설계도를 바탕으로 실제 객체(인스턴스)를 만들고,  
`initialize` 는 객체가 만들어질 때 자동으로 호출되는 초기화 메서드다.

**データ(インスタンス変数(へんすう))と処理(しょり)(メソッド)を一(ひと)つにまとめた設計図(せっけいず)**。  
`new`で設計図(せっけいず)をもとに実際(じっさい)のオブジェクト(インスタンス)を作(つく)り、  
`initialize`はオブジェクト生成時(せいせいじ)に自動(じどう)で呼(よ)ばれる初期化(しょきか)メソッド。

### 기본 클래스

```ruby
class Animal
  # initialize: 객체 생성 시 자동 호출
  # @name, @age: 인스턴스 변수 (@ 가 붙으면 인스턴스 변수)
  def initialize(name, age)
    @name = name
    @age  = age
  end

  def introduce
    "私(わたし)の名前(なまえ)は#{@name}で、#{@age}歳(さい)です。"
  end
end

animal = Animal.new("Dog", 3)   # initialize(name="Dog", age=3) 호출
puts animal.introduce
```

### attr_accessor / attr_reader — 접근자

```ruby
class Person
  # attr_accessor: 읽기 + 쓰기 모두 허용 (getter + setter 자동 생성)
  attr_accessor :name, :age

  # attr_reader: 읽기만 허용 (getter만 생성, 외부에서 변경 불가)
  attr_reader :id

  def initialize(id, name, age)
    @id   = id
    @name = name
    @age  = age
  end

  def introduce
    "私(わたし)の名前(なまえ)は#{@name}で、#{@age}歳(さい)です。"
  end

  # to_s: puts로 출력할 때 자동 호출 (객체를 문자열로 표현)
  def to_s
    "Person(id: #{@id}, name: #{@name}, age: #{@age})"
  end
end

person = Person.new(1, "カンホンギュ", 26)
puts person.introduce

person.name = "田中(たなか)"   # attr_accessor 덕분에 변경 가능
person.age  = 31
# person.id = 99  → NoMethodError! attr_reader라서 쓰기 불가

puts person   # to_s 자동 호출 → Person(id: 1, name: 田中, age: 31)
```

| 접근자          | 읽기 | 쓰기 |
| :-------------- | :--: | :--: |
| `attr_reader`   |  ✅  |  ❌  |
| `attr_writer`   |  ❌  |  ✅  |
| `attr_accessor` |  ✅  |  ✅  |

### 클래스 메서드 — `self.`

```ruby
class MathHelper
  # self.메서드명: 인스턴스 없이 클래스 이름으로 직접 호출
  def self.square(number)
    number ** 2
  end
  def self.cube(number)
    number ** 3
  end
end

# 인스턴스 불필요 — 클래스명.메서드명() 으로 바로 호출
puts MathHelper.square(3)   # → 9
puts MathHelper.cube(4)     # → 64
```

### 상속 (Inheritance) — `<`

```ruby
# Dog와 Cat은 Animal을 상속 (공통 기능을 부모에서 물려받음)
class Animal
  attr_accessor :name, :age
  def initialize(name, age)
    @name = name
    @age  = age
  end
  def introduce
    "私(わたし)の名前(なまえ)は#{@name}で、#{@age}歳(さい)です"
  end
  def speak
    "..."   # 기본 구현 (자식 클래스에서 오버라이드 예정)
  end
end

class Dog < Animal
  # 오버라이드: 부모의 speak를 자식에서 덮어씀
  def speak
    "ワンワン！"
  end
  def fetch
    "#{@name}がボールを持(も)ってきました!"
  end
end

class Cat < Animal
  def speak
    "ニャー！"
  end
  def purr
    "#{@name}がゴロゴロしています"
  end
end

dog = Dog.new("ポチ", 3)
cat = Cat.new("タマ", 5)

puts dog.introduce   # 부모 Animal의 introduce 그대로 사용
puts dog.speak       # Dog에서 오버라이드한 speak
puts dog.fetch       # Dog 고유 메서드
puts cat.speak       # Cat에서 오버라이드한 speak
```

### `super` — 부모 메서드 호출

```ruby
class Dog < Animal
  def introduce
    # super: 부모의 introduce를 그대로 실행하고 + 추가
    super + "そして私(わたし)は犬(いぬ)です"
  end
end

dog = Dog.new("ポチ", 3)
puts dog.introduce
# → 私の名前はポチで、3歳です + そして私は犬です
```

---

### 🔥 도전 코드 — 클래스

**클래스 상속과 다형성(Polymorphism)을 활용한 도형 계산기를 만들어보자.**

```ruby
# 도형의 공통 인터페이스를 부모 클래스로 정의
class Shape
  attr_reader :color

  def initialize(color = "白(しろ)")
    @color = color
  end

  # 자식 클래스에서 반드시 오버라이드해야 하는 메서드
  def area
    raise NotImplementedError, "#{self.class}はareaを実装(じっそう)してください"
  end

  def perimeter
    raise NotImplementedError, "#{self.class}はperimeterを実装(じっそう)してください"
  end

  def describe
    "図形(ずけい): #{self.class} | 色(いろ): #{@color} | " \
    "面積(めんせき): #{area.round(2)} | 周囲(しゅうい): #{perimeter.round(2)}"
  end
end

class Circle < Shape
  def initialize(radius, color = "赤(あか)")
    super(color)          # 부모의 initialize 호출
    @radius = radius
  end
  def area      = Math::PI * @radius ** 2
  def perimeter = 2 * Math::PI * @radius
end

class Rectangle < Shape
  def initialize(width, height, color = "青(あお)")
    super(color)
    @width  = width
    @height = height
  end
  def area      = @width * @height
  def perimeter = 2 * (@width + @height)
end

class Triangle < Shape
  def initialize(a, b, c, color = "緑(みどり)")
    super(color)
    @a, @b, @c = a, b, c
  end
  def perimeter = @a + @b + @c
  def area
    # 헤론의 공식: s = (a+b+c)/2, 넓이 = √(s(s-a)(s-b)(s-c))
    s = perimeter / 2.0
    Math.sqrt(s * (s - @a) * (s - @b) * (s - @c))
  end
end

shapes = [
  Circle.new(5),
  Rectangle.new(4, 6),
  Triangle.new(3, 4, 5),
]

# 다형성: 같은 메서드 이름으로 각 클래스가 다르게 동작
shapes.each { |shape| puts shape.describe }

puts ""
# 가장 넓은 도형 찾기
largest = shapes.max_by { |s| s.area }
puts "最大面積(さいだいめんせき): #{largest.class} (#{largest.area.round(2)})"
```

**포인트:**

- `raise NotImplementedError` → 추상 메서드처럼 강제 구현 요구
- `super(인수)` → 부모의 initialize에 인수 전달
- `def 메서드 = 식` (한 줄 메서드) + 다형성 → 각 클래스가 같은 이름으로 다른 계산
- `max_by` → 특정 값이 최대인 요소 찾기

---

## 2. 모듈 (Module)

**메서드나 상수를 묶어두는 그릇**.  
클래스와 다르게 **인스턴스를 만들 수 없고**, 두 가지 용도로 쓰인다:  
① `include` — 인스턴스 메서드로 추가 (믹스인, Mixin)  
② `extend` — 클래스 메서드로 추가  
③ `::` — 네임스페이스(이름 충돌 방지)로 사용

**メソッドや定数(ていすう)をまとめる入(い)れ物(もの)**。  
クラスと違(ちが)い**インスタンスは作(つく)れず**、主(おも)に二(ふた)つの用途(ようと)がある：  
① `include` — インスタンスメソッドとして追加(ついか)(Mixin)  
② `extend` — クラスメソッドとして追加(ついか)  
③ `::` — 名前空間(なまえくうかん)(名前衝突(なまえしょうとつ)防止(ぼうし))として使用(しよう)

### `include` — 믹스인 (인스턴스 메서드)

```ruby
module Greetable
  def greet
    "こんにちは、私(わたし)は#{@name}です"
  end
end

module Farewell
  def farewell
    "さようなら、私(わたし)は#{@name}です"
  end
end

class Person
  include Greetable   # Greetable의 메서드를 인스턴스 메서드로 추가
  include Farewell
  def initialize(name)
    @name = name
  end
end

class Robot
  include Greetable   # 다른 클래스에서도 동일하게 재사용!
  def initialize(name)
    @name = name
  end
end

person = Person.new("カンホンギュ")
puts person.greet    # → こんにちは、私はカンホンギュです
puts person.farewell

robot = Robot.new("R2D2")
puts robot.greet     # → こんにちは、私はR2D2です
# puts robot.farewell  → NoMethodError! Farewell을 include 안 했으니까
```

### `extend` — 클래스 메서드로 추가

```ruby
module Describable
  def describe
    "これはメソッドです"
  end
end

class Car
  extend Describable   # 클래스 메서드로 추가
end

# 인스턴스 불필요 — 클래스명으로 직접 호출
puts Car.describe   # → これはメソッドです
# Car.new.describe  → NoMethodError!
```

### `include` vs `extend` 비교

```ruby
module Walkable
  def walk
    "歩(ある)いています"
  end
end

class Human
  include Walkable   # 인스턴스 메서드로
end

class AnimalClass
  extend Walkable    # 클래스 메서드로
end

puts Human.new.walk   # → 歩いています  (인스턴스로 호출)
puts AnimalClass.walk # → 歩いています  (클래스로 호출)
```

### `::` — 네임스페이스 (이름 충돌 방지)

```ruby
# 같은 이름의 클래스를 모듈로 구분
module DomesticAnimal
  class Dog
    def speak = "ワンワン！"
  end
  class Cat
    def speak = "ニャー！"
  end
end

module RobotAnimal
  class Dog
    def speak = "ビービー！"
  end
  class Cat
    def speak = "ジージー！"
  end
end

# :: 로 어느 모듈의 클래스인지 명시
puts DomesticAnimal::Dog.new.speak   # → ワンワン！
puts RobotAnimal::Dog.new.speak      # → ビービー！
```

---

### 🔥 도전 코드 — 모듈

**모듈로 공통 기능을 분리해서 Rails의 Concern과 비슷한 구조를 만들어보자.**

```ruby
# 타임스탬프 자동 관리 (Rails의 ActiveRecord 타임스탬프와 유사)
module Timestamps
  def self.included(base)
    # include된 시점에 실행 — 클래스에 인스턴스 변수 초기화 추가
    puts "Timestamps が #{base}にincludeされました"
  end

  def touch_created
    @created_at = Time.now.strftime("%Y-%m-%d %H:%M:%S")
  end

  def touch_updated
    @updated_at = Time.now.strftime("%Y-%m-%d %H:%M:%S")
  end

  def timestamps
    "作成(さくせい): #{@created_at || "未設定(みせってい)"} | " \
    "更新(こうしん): #{@updated_at || "未設定(みせってい)"}"
  end
end

# 유효성 검사 모듈
module Validatable
  def valid?
    errors.empty?
  end

  def errors
    @errors ||= []   # ||= : 처음 호출 시에만 초기화
  end

  def add_error(message)
    errors << message
  end

  def validate!
    raise "バリデーションエラー: #{errors.join(", ")}" unless valid?
    true
  end
end

# 두 모듈을 함께 사용
class User
  include Timestamps
  include Validatable

  attr_accessor :name, :email

  def initialize(name, email)
    @name  = name
    @email = email
    touch_created   # 생성 시 타임스탬프 기록
    run_validations
  end

  private

  def run_validations
    add_error("名前(なまえ)は必須(ひっす)です") if @name.to_s.empty?
    add_error("メールは必須(ひっす)です")       if @email.to_s.empty?
    add_error("メール形式(けいしき)が不正(ふせい)です") unless @email.match?(/\A\S+@\S+\z/)
  end
end

user1 = User.new("カンホンギュ", "kang@example.com")
puts user1.valid?       # → true
puts user1.timestamps   # 생성 시각 표시

user2 = User.new("", "invalid-email")
puts user2.valid?       # → false
puts user2.errors.inspect
```

**포인트:**

- `self.included(base)` → 모듈이 include된 시점에 실행되는 훅(hook)
- `@errors ||= []` → nil일 때만 초기화하는 관용구 (lazy initialization)
- `private` → 이후 메서드를 외부에서 호출 불가로 설정
- Rails의 `ActiveSupport::Concern` 과 동일한 발상

---

## 3. 예외처리 (Exception)

**프로그램 실행 중 발생할 수 있는 에러를 안전하게 처리하는 구조**.  
에러가 발생해도 프로그램이 갑자기 종료되지 않고,  
`rescue` 블록에서 **어떻게 대응할지 정의**할 수 있다.

**プログラム実行中(じっこうちゅう)に発生(はっせい)するエラーを安全(あんぜん)に処理(しょり)する構造(こうぞう)**。  
エラーが発生(はっせい)してもプログラムが突然終了(とつぜんしゅうりょう)せず、  
`rescue`ブロックで**どう対応(たいおう)するか定義(ていぎ)できる**。

### 기본 구조

```ruby
begin
  # 에러가 발생할 수 있는 코드
  puts 10 / 0
rescue => error
  # 모든 에러를 받아서 처리
  puts "エラー発生(はっせい)しました: #{error.message}"
end
# → エラー発生しました: divided by 0
```

### 에러 종류별 rescue

```ruby
begin
  puts 10 / 0
rescue ZeroDivisionError => error   # 0으로 나누기
  puts "ゼロ除算(じょざん): #{error.message}"
end

begin
  puts "hello" + 1
rescue TypeError => error           # 타입 불일치
  puts "タイプエラー: #{error.message}"
end

begin
  puts undefined_variable
rescue NameError => error           # 미정의 변수/메서드
  puts "名前(なまえ)エラー: #{error.message}"
end
```

### 여러 에러를 순서대로 rescue

```ruby
begin
  puts 10 / 0
rescue ZeroDivisionError => error
  puts "ゼロ除算(じょざん)エラー: #{error.message}"
rescue TypeError => error
  puts "タイプエラー: #{error.message}"
rescue StandardError => error
  # StandardError는 대부분의 에러의 부모 → 마지막에 배치
  puts "標準(ひょうじゅん)エラー: #{error.message}"
end
# 주의: 더 구체적인 에러를 먼저, 더 일반적인 에러를 나중에 쓸 것!
```

### `ensure` — 반드시 실행

```ruby
begin
  puts 10 / 0
rescue ZeroDivisionError => error
  puts "ゼロ除算(じょざん)エラー: #{error.message}"
ensure
  # 에러 발생 여부와 관계없이 항상 실행
  # 파일 닫기, DB 연결 해제 등에 사용
  puts "ensureは必(かなら)ず実行(じっこう)されます。"
end
```

### `raise` — 의도적으로 에러 발생

```ruby
# 에러 메시지만
begin
  raise "意図的(いとてき)なエラーです"
rescue => error
  puts error.message   # → 意図的なエラーです
end

# 에러 종류 지정
begin
  raise ArgumentError, "引数(ひきすう)が不正(ふせい)です"
rescue ArgumentError => error
  puts "引数(ひきすう)エラー: #{error.message}"
end
```

### 커스텀 에러 클래스

```ruby
# StandardError를 상속해서 자신만의 에러 정의
class AgeError < StandardError
  def initialize(message: "年齢(ねんれい)が不正(ふせい)です")
    super   # 부모(StandardError)의 initialize에 message 전달
  end
end

begin
  age = -1
  raise AgeError if age < 0
rescue AgeError => error
  puts "カスタムエラー: #{error.message}"
  # → カスタムエラー: 年齢が不正です
end
```

### `retry` — 재시도

```ruby
retry_count = 0

begin
  retry_count += 1
  puts "試行(しこう)回数(かいすう): #{retry_count}"
  raise "エラー発生(はっせい)" if retry_count < 3   # 3번 미만이면 에러
  puts "成功(せいこう)しました"
rescue => error
  puts "エラー: #{error.message}"
  retry if retry_count < 3   # 3번 미만이면 begin 블록으로 돌아가 재시도
end
# → 試行回数: 1 → 2 → 3 → 成功しました
```

---

### 🔥 도전 코드 — 예외처리

**재시도 로직과 커스텀 에러로 API 클라이언트처럼 동작하는 코드를 만들어보자.**

```ruby
# 커스텀 에러 계층 구조
class AppError < StandardError; end
class NetworkError  < AppError
  def initialize(msg = "ネットワークエラーが発生(はっせい)しました")
    super
  end
end
class RateLimitError < AppError
  attr_reader :retry_after
  def initialize(retry_after: 5)
    @retry_after = retry_after
    super("レートリミット: #{retry_after}秒(びょう)後(ご)に再試行(さいしこう)してください")
  end
end

# 불안정한 API를 흉내내는 메서드
def fake_api_call(attempt)
  case attempt
  when 1 then raise NetworkError
  when 2 then raise RateLimitError.new(retry_after: 2)
  when 3 then { status: 200, data: "成功(せいこう)！" }
  end
end

# 재시도 로직이 있는 API 호출 래퍼
def call_with_retry(max_retries: 3)
  attempt = 0

  begin
    attempt += 1
    puts "試行(しこう) #{attempt}/#{max_retries}..."
    result = fake_api_call(attempt)
    puts "成功(せいこう): #{result[:data]}"

  rescue RateLimitError => e
    puts "警告(けいこく): #{e.message}"
    if attempt < max_retries
      puts "#{e.retry_after}秒(びょう)待機(たいき)します..."
      sleep(0) # 테스트용: 실제 코드에서는 sleep(e.retry_after)
      retry
    else
      puts "最大(さいだい)リトライ回数(かいすう)に達(たっ)しました"
    end

  rescue NetworkError => e
    puts "エラー: #{e.message}"
    retry if attempt < max_retries

  rescue AppError => e
    # 모든 AppError의 부모 → 마지막에
    puts "アプリエラー: #{e.message}"

  ensure
    puts "--- 試行(しこう)#{attempt}完了(かんりょう) ---"
  end
end

call_with_retry(max_retries: 3)
```

**포인트:**

- 에러 클래스 계층: `AppError < StandardError`, `NetworkError < AppError` → 세분화
- `attr_reader :retry_after` → 에러 객체에 추가 정보를 담을 수 있음
- `rescue` 순서: 더 구체적인 에러(RateLimitError) → 덜 구체적인 에러(AppError) 순
- `ensure` 는 retry할 때도 매번 실행됨

---

## 4. 표준 라이브러리 (Library)

**Ruby에 기본으로 포함된 라이브러리**.  
`require "라이브러리명"` 으로 불러와서 사용하며,  
파일, CSV, YAML, JSON 처리 등 실무에서 자주 쓰이는 기능을 제공한다.

**Rubyに標準(ひょうじゅん)で含(ふく)まれるライブラリ**。  
`require`で読(よ)み込(こ)み、ファイル・CSV・YAML・JSON処理(しょり)など実務(じつむ)でよく使(つか)う機能(きのう)を提供(ていきょう)する。

### `Dir` — 디렉토리 조작

```ruby
require "csv"    # 사용할 라이브러리를 파일 상단에 선언
require "yaml"
require "json"

puts Dir.pwd                    # 현재 디렉토리 경로
Dir.mkdir("test_dir")           # 디렉토리 생성
puts Dir.exist?("test_dir")     # → true
puts Dir.entries(".").inspect   # 현재 폴더의 모든 파일/폴더 목록
puts Dir.glob("*.rb").inspect   # *.rb 패턴에 맞는 파일 목록 (와일드카드)
```

### `File` — 파일 읽기/쓰기

```ruby
File.write("test.txt", "こんにちは/世界(せかい)")  # 파일 쓰기 (없으면 생성)
puts File.read("test.txt")              # 파일 전체 읽기
puts File.exist?("test.txt")            # → true
puts File.basename("path/to/test.txt")  # → "test.txt"  (파일명만)
puts File.extname("test.txt")           # → ".txt"      (확장자만)
File.delete("test.txt")                 # 파일 삭제
puts File.exist?("test.txt")            # → false
```

### `CSV` — CSV 파일 처리

```ruby
# CSV 쓰기
CSV.open("test.csv", "w") do |csv|
  csv << ["name", "age", "city"]         # 헤더
  csv << ["カンホンギュ", 26, "東京(とうきょう)"]
  csv << ["田中(たなか)", 30, "大阪(おおさか)"]
end

# 한 줄씩 읽기
CSV.foreach("test.csv") do |row|
  puts row.inspect
end

# 전체 읽기 → 2차원 배열
puts CSV.read("test.csv").inspect
```

### `YAML` / `JSON` — 설정 파일 처리

```ruby
data = { name: "カンホンギュ", age: 26, city: "東京(とうきょう)" }

# YAML 저장 & 읽기 — 설정 파일에 자주 사용
File.write("test.yml", data.to_yaml)
puts YAML.load_file("test.yml").inspect
# → {"name"=>"カンホンギュ", "age"=>26, "city"=>"東京"}

# JSON 저장 & 읽기 — API 응답에 자주 사용
File.write("test.json", data.to_json)
puts JSON.parse(File.read("test.json")).inspect
# → {"name"=>"カンホンギュ", "age"=>26, "city"=>"東京"}

# 사용 후 정리
File.delete("test.csv")
File.delete("test.yml")
File.delete("test.json")
Dir.delete("test_dir")
```

---

### 🔥 도전 코드 — 라이브러리

**CSV + JSON + 예외처리를 조합한 데이터 변환 파이프라인을 만들어보자.**

```ruby
require "csv"
require "json"

# 샘플 CSV 데이터 생성
CSV.open("students.csv", "w") do |csv|
  csv << ["id", "name", "score", "email"]
  csv << [1, "カンホンギュ",  88, "kang@example.com"]
  csv << [2, "田中(たなか)",   72, "tanaka@example.com"]
  csv << [3, "佐藤(さとう)",   95, "invalid-email"]       # 잘못된 이메일
  csv << [4, "鈴木(すずき)",   -5, "suzuki@example.com"]  # 잘못된 점수
end

# CSV를 읽어서 검증 후 JSON으로 저장하는 파이프라인
valid_students   = []
invalid_students = []

CSV.foreach("students.csv", headers: true) do |row|
  begin
    # 데이터를 해시로 변환
    student = {
      id:    Integer(row["id"]),
      name:  row["name"],
      score: Integer(row["score"]),
      email: row["email"],
    }

    # 유효성 검사
    raise ArgumentError, "点数(てんすう)は0〜100の範囲(はんい)です" unless (0..100).include?(student[:score])
    raise ArgumentError, "メール形式(けいしき)が不正(ふせい)です"   unless student[:email].match?(/\A\S+@\S+\.\S+\z/)

    valid_students << student

  rescue ArgumentError => e
    invalid_students << { name: row["name"], error: e.message }
  rescue => e
    invalid_students << { name: row["name"] || "不明(ふめい)", error: "予期(よき)せぬエラー: #{e.message}" }
  end
end

# 결과를 JSON으로 저장
output = {
  summary: {
    total:   valid_students.length + invalid_students.length,
    valid:   valid_students.length,
    invalid: invalid_students.length,
  },
  students: valid_students,
  errors:   invalid_students,
}

File.write("result.json", JSON.pretty_generate(output))

puts "=== 処理結果(しょりけっか) ==="
puts JSON.pretty_generate(output)

# 정리
File.delete("students.csv")
File.delete("result.json")
```

**포인트:**

- `CSV.foreach("파일", headers: true)` → 첫 행을 헤더로 처리, `row["컬럼명"]` 으로 접근
- `Integer(문자열)` → `to_i` 와 달리 변환 실패 시 `ArgumentError` 발생 → 검증에 적합
- `JSON.pretty_generate(hash)` → 들여쓰기된 JSON 출력
- begin/rescue를 루프 안에 넣으면 에러가 나도 다음 줄 처리를 계속할 수 있음

---

## 🔑 한눈에 보는 키워드

| 키워드                          | 설명                                |
| :------------------------------ | :---------------------------------- |
| `class 클래스명 / end`          | 클래스 정의                         |
| `def initialize / end`          | 생성자 — `new` 호출 시 자동 실행    |
| `@변수`                         | 인스턴스 변수                       |
| `attr_accessor`                 | 읽기 + 쓰기 접근자 자동 생성        |
| `attr_reader`                   | 읽기 전용 접근자                    |
| `def self.메서드`               | 클래스 메서드                       |
| `class 자식 < 부모`             | 상속                                |
| `super`                         | 부모의 동일 메서드 호출             |
| `to_s`                          | `puts` 시 자동 호출되는 문자열 변환 |
| `module 모듈명 / end`           | 모듈 정의                           |
| `include 모듈`                  | 인스턴스 메서드로 추가              |
| `extend 모듈`                   | 클래스 메서드로 추가                |
| `모듈::클래스`                  | 네임스페이스로 접근                 |
| `begin / rescue / ensure / end` | 예외처리 구조                       |
| `rescue 에러클래스 => e`        | 특정 에러 처리                      |
| `ensure`                        | 에러 여부 상관없이 항상 실행        |
| `raise "메시지"`                | 에러 강제 발생                      |
| `retry`                         | begin 블록 처음으로 재시도          |
| `class 에러 < StandardError`    | 커스텀 에러 정의                    |
| `require "라이브러리"`          | 라이브러리 로드                     |
| `File.write / read / delete`    | 파일 쓰기 / 읽기 / 삭제             |
| `Dir.mkdir / exist? / glob`     | 디렉토리 생성 / 확인 / 패턴 검색    |
| `CSV.open / foreach / read`     | CSV 쓰기 / 순회 / 전체 읽기         |
| `YAML.load_file`                | YAML 파일 읽기                      |
| `JSON.parse / pretty_generate`  | JSON 파싱 / 예쁘게 출력             |

---

[← 목차로 돌아가기 / 目次(もくじ)に戻(もど)る](../README.md)
