# 📅 Day 23 — Ruby 실전 (날짜·시각 & OOP 기초 & 세탁기로 배우는 설계)

**날짜 / 日付(ひづけ):** 2026.03.15  
**학습 시간 / 学習時間(がくしゅうじかん):** 자율 / 自律(じりつ)  
**한 줄 목표:** Time/Date로 날짜를 다루고, OOP의 핵심 개념(캡슐화·다형성·책임 분리)을 세탁기 예제로 이해한다.  
**一言目標(ひとこともくひょう):** Time/Dateで日付(ひづけ)を扱(あつか)い、OOPの核心(かくしん)(カプセル化(か)・多態性(たたいせい)・責任分離(せきにんぶんり))を洗濯機(せんたくき)の例(れい)で理解(りかい)する。

---

## 목차 / 目次(もくじ)

1. [날짜와 시각 — Time & Date](#1-날짜와-시각--time--date)
2. [OOP 기초 — 캡슐화 & 다형성](#2-oop-기초--캡슐화--다형성)
3. [OOP 실전 — 세탁기로 배우는 객체 설계](#3-oop-실전--세탁기로-배우는-객체-설계)

---

## 1. 날짜와 시각 — Time & Date

**Ruby에는 날짜·시각을 다루는 두 가지 클래스가 있다.**  
`Time` — 날짜와 시각을 함께 다룸 (초 단위까지)  
`Date` — 날짜만 다룸 (년·월·일), 날짜 연산이 편리

**Rubyには日付(ひづけ)・時刻(じこく)を扱(あつか)う2つのクラスがある**。  
`Time` — 日付(ひづけ)と時刻(じこく)を合(あ)わせて扱(あつか)う (秒単位(びょうたんい)まで)  
`Date` — 日付(ひづけ)のみ (年(ねん)・月(つき)・日(にち))、日付計算(ひづけけいさん)が便利(べんり)

### `Time` — 현재 시각

```ruby
require "date"   # Date 클래스를 사용하려면 require 필요 (Time은 불필요)

time = Time.now   # 현재 시각을 Time 객체로
puts time         # → 2026-03-15 14:32:01 +0900

# 각 구성 요소를 개별 메서드로 꺼낼 수 있음
puts time.year    # → 2026
puts time.month   # → 3
puts time.day     # → 15
puts time.hour    # → 14
puts time.min     # → 32
puts time.sec     # → 1
puts time.wday    # → 0 (0=일요일, 1=월요일 ... 6=토요일)
```

### `Time.new` — 특정 시각 생성

```ruby
# Time.new(년, 월, 일, 시, 분, 초)
specific_time = Time.new(2026, 3, 16, 12, 0, 0)

# strftime으로 원하는 형식으로 포맷
puts specific_time.strftime("%Y/%m/%d %H:%M:%S")
# → 2026/03/16 12:00:00

# 주요 포맷 코드
# %Y: 연도(4자리)  %m: 월(2자리)  %d: 일(2자리)
# %H: 시(24h)      %M: 분         %S: 초
# %A: 요일 영문    %a: 요일 약자
```

### `Date` — 날짜 전용

```ruby
date = Date.today   # 오늘 날짜
puts date.year      # → 2026
puts date.month     # → 3
puts date.day       # → 15
puts date.wday      # → 0 (0=일요일)
puts date.strftime("%Y年(ねん)%m月(つき)%d日(にち)")   # → 2026年03月15日

# 특정 날짜 생성
specific_date = Date.new(2026, 2, 12)
puts specific_date   # → 2026-02-12
```

### 날짜 연산

```ruby
date = Date.today

# + / - : 일(day) 단위 덧셈 뺄셈
puts date + 7    # → 7일 후
puts date - 7    # → 7일 전

# >> / << : 월(month) 단위 덧셈 뺄셈
puts date >> 2   # → 2개월 후
puts date << 2   # → 2개월 전

# Time vs Date 비교
time_now = Time.now
date_now = Date.today
puts time_now   # → 2026-03-15 14:32:01 +0900  (시각 포함)
puts date_now   # → 2026-03-15                  (날짜만)
```

---

### 🔥 도전 코드 — 날짜와 시각

**날짜 계산으로 일정 관리 시스템을 만들어보자.**

```ruby
require "date"

# 특정 날짜까지 D-day 계산
def days_until(target_date, label)
  today = Date.today
  diff  = (target_date - today).to_i

  if diff > 0
    puts "#{label}: D-#{diff} (#{target_date.strftime("%Y/%m/%d")})"
  elsif diff == 0
    puts "#{label}: 今日(きょう)です！🎉"
  else
    puts "#{label}: #{diff.abs}日前(にちまえ)に終了(しゅうりょう)しました"
  end
end

# 학습 기간 계산
def study_progress(start_date, end_date, current_day)
  total_days    = (end_date - start_date).to_i
  elapsed_days  = (Date.today - start_date).to_i
  progress_pct  = [(elapsed_days.to_f / total_days * 100).round(1), 100].min

  bar_length  = 20
  filled      = (bar_length * progress_pct / 100).round
  bar         = "█" * filled + "░" * (bar_length - filled)

  puts "学習進捗(がくしゅうしんちょく): [#{bar}] #{progress_pct}%"
  puts "経過(けいか): #{elapsed_days}日(にち) / 全体(ぜんたい): #{total_days}日(にち)"
  puts "現在(げんざい): Day #{current_day}"
end

# 요일별 추천 학습 내용
def todays_schedule
  schedule = {
    0 => "復習(ふくしゅう)デー — 今週(こんしゅう)の内容(ないよう)を振(ふ)り返(かえ)る",
    1 => "新規(しんき)学習(がくしゅう) — 新(あたら)しい概念(がいねん)を学(まな)ぶ",
    2 => "実践(じっせん)コード — 昨日(きのう)の内容(ないよう)を実装(じっそう)する",
    3 => "新規(しんき)学習(がくしゅう) — ステップアップ",
    4 => "実践(じっせん)コード — 応用(おうよう)問題(もんだい)に挑戦(ちょうせん)",
    5 => "総復習(そうふくしゅう) — 週間(しゅうかん)まとめ",
    6 => "自由(じゆう)課題(かだい) — 興味(きょうみ)のあるテーマを深掘(ふかほ)り",
  }
  puts "今日(きょう)のメニュー(#{Date::DAYNAMES[Date.today.wday]}): #{schedule[Date.today.wday]}"
end

# 실행
bootcamp_start = Date.new(2026, 2, 1)
bootcamp_end   = Date.new(2026, 3, 31)

study_progress(bootcamp_start, bootcamp_end, 23)
puts ""

days_until(Date.new(2026, 4, 20), "ロックウッドホテル勤務(きんむ)開始(かいし)")
days_until(Date.new(2026, 12, 31), "ワーホリビザ期限(きげん)")
days_until(Date.new(2027, 4, 1),  "日本(にほん)IT企業(きぎょう)入社(にゅうしゃ)目標(もくひょう)")
puts ""

todays_schedule
```

**포인트:**

- `(date2 - date1).to_i` → 날짜 차이를 정수로 (Date끼리 빼면 Rational 반환)
- `Date::DAYNAMES` → `["Sunday", "Monday", ...]` 요일 이름 배열 (상수)
- `[값, 100].min` → 100을 초과하지 않도록 제한하는 관용구
- 진행률 바(`█░`) → 시각적 피드백, CLI 도구에서 자주 쓰이는 패턴

---

## 2. OOP 기초 — 캡슐화 & 다형성

**OOP(Object-Oriented Programming, 객체 지향 프로그래밍)의 핵심은 3가지.**

1. **캡슐화** — 데이터와 처리를 하나의 클래스로 묶고, 외부에서 직접 건드리지 못하게 보호
2. **상속** — 공통 기능을 부모 클래스에 정의하고, 자식 클래스가 물려받아 재사용
3. **다형성** — 같은 메서드 이름으로 클래스마다 다른 동작을 하게 하는 것

**OOP(オブジェクト指向(しこう)プログラミング)の核心(かくしん)は3つ**。  
①カプセル化(か) ②継承(けいしょう) ③多態性(たたいせい)

### 클래스로 데이터 묶기

```ruby
class Person
  attr_reader :name, :age   # 읽기만 허용 (외부에서 변경 불가)

  def initialize(name, age)
    @name = name
    @age  = age
  end

  def introduce
    "私(わたし)の名前(なまえ)は#{@name}で、#{@age}歳(さい)です。"
  end
end

person = Person.new("カンホンギュ", 26)
puts person.introduce   # → 私の名前はカンホンギュで、26歳です。
puts person.name        # → カンホンギュ (attr_reader로 읽기 가능)
# person.name = "田中" → NoMethodError! (attr_reader는 쓰기 불가)
```

### 상속 + 오버라이드

```ruby
class Animal
  attr_reader :name

  def initialize(name)
    @name = name
  end

  def introduce
    "私(わたし)の名前(なまえ)は#{@name}です。"
  end

  def speak
    "..."   # 기본값 — 자식 클래스에서 오버라이드 예정
  end
end

class Dog < Animal
  def speak = "ワンワン！"   # 부모의 speak를 덮어씀
end

class Cat < Animal
  def speak = "ニャー！"
end

dog = Dog.new("ポチ")
cat = Cat.new("タマ")
puts dog.introduce   # 부모(Animal)의 introduce 그대로 사용
puts dog.speak       # Dog에서 오버라이드한 speak
```

### 다형성 — 같은 메서드, 다른 동작

```ruby
# 다형성의 핵심: 배열에 여러 클래스의 인스턴스를 담고
# 같은 메서드명으로 호출해도 각 클래스에 맞는 동작이 실행됨
animals = [Dog.new("ポチ"), Cat.new("タマ")]

animals.each do |animal|
  puts animal.speak   # Dog면 ワンワン、Cat면 ニャー が自動(じどう)で選(えら)ばれる
end
```

### `private` — 캡슐화 (내부 구현 숨기기)

```ruby
class Car
  def initialize(brand)
    @brand          = brand
    @engine_started = false
  end

  # public 메서드 — 외부에서 사용하는 인터페이스
  def start
    engine_on   # private 메서드를 내부에서 호출
    "#{@brand}のエンジンがスタートしました"
  end

  def stop
    engine_off
    "#{@brand}のエンジンが停止(ていし)しました"
  end

  private   # 이후의 메서드는 외부에서 호출 불가

  # private 메서드 — 내부 구현 세부사항, 외부에서 직접 호출 불가
  def engine_on  = @engine_started = true
  def engine_off = @engine_started = false
end

car = Car.new("トヨタ")
puts car.start   # → トヨタのエンジンがスタートしました
puts car.stop
# car.engine_on  → NoMethodError! private 메서드는 외부 호출 불가
```

> **`private` 를 쓰는 이유**  
> 외부에서 `car.engine_on` 을 직접 호출하면 `car.start` 를 거치지 않고  
> 엔진만 켜지는 예상치 못한 상태가 될 수 있다.  
> `private` 로 숨겨서 반드시 `start` → `engine_on` 순서로만 실행되게 강제하는 것.

---

### 🔥 도전 코드 — OOP

**다형성을 활용한 결제 시스템을 만들어보자.**

```ruby
# 결제 수단 공통 인터페이스
class PaymentMethod
  attr_reader :name

  def initialize(name)
    @name = name
  end

  # 자식 클래스에서 반드시 구현
  def pay(amount)
    raise NotImplementedError, "#{self.class}はpayを実装(じっそう)してください"
  end

  def receipt(amount)
    puts "=== 領収書(りょうしゅうしょ) ==="
    puts "支払方法(しはらいほうほう): #{@name}"
    puts "金額(きんがく): #{amount}円(えん)"
    puts pay(amount)
    puts "==================="
  end
end

class CreditCard < PaymentMethod
  def initialize(card_number)
    super("クレジットカード")
    @card_number = "****-****-****-#{card_number[-4..]}"  # 마지막 4자리만
  end

  def pay(amount)
    "カード番号(ばんごう): #{@card_number} で#{amount}円(えん)を決済(けっさい)しました"
  end
end

class Cash < PaymentMethod
  def initialize(held_amount)
    super("現金(げんきん)")
    @held = held_amount
  end

  def pay(amount)
    if @held >= amount
      @held -= amount
      "#{amount}円(えん)をお支払(しはら)いいただきました。おつり: #{@held}円(えん)"
    else
      raise "残高(ざんだか)が不足(ふそく)しています (所持金(しょじきん): #{@held}円(えん))"
    end
  end
end

class PayPay < PaymentMethod
  def initialize(balance)
    super("PayPay")
    @balance = balance
  end

  def pay(amount)
    if @balance >= amount
      @balance -= amount
      "PayPay残高(ざんだか)より#{amount}円(えん)を支払(しはら)いました。残高(ざんだか): #{@balance}円(えん)"
    else
      raise "PayPay残高(ざんだか)が不足(ふそく)しています"
    end
  end
end

# 다형성 — 같은 receipt 메서드로 모든 결제 수단 처리
payments = [
  CreditCard.new("1234567890123456"),
  Cash.new(10000),
  PayPay.new(5000),
]

payments.each do |method|
  begin
    method.receipt(3000)
  rescue => e
    puts "エラー: #{e.message}"
  end
  puts ""
end
```

**포인트:**

- 부모 `PaymentMethod` 가 공통 인터페이스(`receipt`) 정의, 자식마다 `pay` 구현
- `card_number[-4..]` → 마지막 4자리 슬라이스 (Ruby 2.6+ 무한 범위)
- `super("クレジットカード")` → 부모의 `initialize` 에 인수 전달
- 배열로 결제 수단을 관리하면 새 결제 수단 추가 시 `receipt` 코드 변경 불필요

---

## 3. OOP 실전 — 세탁기로 배우는 객체 설계

**한 가지 시스템을 여러 클래스로 분리해서 설계하는 연습**.  
`Clothing` 과 `WashingMachine` 을 각각 분리한 것이 핵심인데,  
이것이 OOP의 **단일 책임 원칙(SRP)** — "클래스는 하나의 책임만 가져야 한다".

**一(ひと)つのシステムを複数(ふくすう)のクラスに分(わ)けて設計(せっけい)する練習(れんしゅう)**。  
これがOOPの**単一責任原則(たんいつせきにんげんそく)(SRP)** — 「クラスは一(ひと)つの責任(せきにん)だけ持(も)つ」。

### `Clothing` 클래스 — 옷의 책임

```ruby
class Clothing
  MAX_CAPACITY = 30   # 상수: 클래스 전체에서 공유하는 변경 불가 값

  attr_reader :name, :clean

  def initialize(name)
    @name  = name
    @clean = false   # 처음에는 더러운 상태
  end

  # ? 로 끝나는 메서드 — 상태를 확인하는 관례
  def clean?
    @clean
  end

  # ! 로 끝나는 메서드 — 상태를 변경하는 파괴적 메서드 관례
  def wash!
    @clean = true
  end
end
```

### `WashingMachine` 클래스 — 세탁기의 책임

```ruby
class WashingMachine
  def initialize
    @clothes = []   # 세탁 중인 옷 목록
  end

  # 옷 넣기 — 가득 차면 예외 발생
  def add(clothing)
    if full?
      raise "洗濯機(せんたくき)が満杯(まんぱい)です、最大(さいだい)#{Clothing::MAX_CAPACITY}着(ちゃく)までです"
    end
    @clothes << clothing
  end

  # 옷 꺼내기
  def remove(clothing)
    @clothes.delete(clothing)
  end

  # 세탁 — 각 옷에 wash! 메시지 전달 (세탁 방법은 Clothing이 알고 있음)
  def wash
    @clothes.each { |clothing| clothing.wash! }
  end

  # 가득 찼는지 확인 — 내부 로직을 메서드로 분리
  def full?
    @clothes.length >= Clothing::MAX_CAPACITY
  end

  # 상태 출력
  def status
    @clothes.each do |clothing|
      state = clothing.clean? ? "清潔(せいけつ)" : "汚(よご)れている"
      puts "#{clothing.name}: #{state}"
    end
  end
end
```

### 실행 & 예외처리

```ruby
washing_machine = WashingMachine.new
shirt   = Clothing.new("シャツ")
pants   = Clothing.new("ズボン")
jacket  = Clothing.new("ジャケット")

washing_machine.add(shirt)
washing_machine.add(pants)
washing_machine.add(jacket)

washing_machine.status   # 세탁 전 상태
washing_machine.wash
washing_machine.status   # 세탁 후 상태

washing_machine.remove(jacket)
washing_machine.status   # 재킷 제거 후 상태

# 최대 용량 초과 → 예외 발생
begin
  31.times { |i| washing_machine.add(Clothing.new("服(ふく)#{i}")) }
rescue => error
  puts error.message   # → 洗濯機が満杯です、最大30着までです
end
```

### 설계 포인트 분석

```
Clothing의 책임:
  - 자신의 이름과 청결 상태를 관리
  - wash! 로 자신을 세탁하는 방법을 알고 있음
  - 최대 용량(MAX_CAPACITY)을 상수로 정의

WashingMachine의 책임:
  - 옷의 추가/제거를 관리
  - 용량 초과 여부 판단
  - 세탁 명령을 각 옷에 위임 (clothing.wash!)
  - 전체 상태 출력

핵심: WashingMachine은 "어떻게 옷을 세탁하는지" 몰라도 된다.
      그저 clothing.wash! 를 호출하면 Clothing이 알아서 처리한다.
      → 이것이 "책임 분리" 이자 "메시지 전달" 방식의 OOP
```

---

### 🔥 도전 코드 — 세탁기 확장

**세탁 모드와 건조 기능을 추가해서 진짜 세탁기처럼 만들어보자.**

```ruby
class Clothing
  MAX_CAPACITY = 10   # 도전용 예제는 10으로 줄임

  WASH_MODES = {
    standard:  { time: 30, water: 50 },
    delicate:  { time: 15, water: 30 },   # 섬세한 소재
    heavy_duty: { time: 60, water: 80 },  # 두꺼운 소재
  }

  attr_reader :name, :clean, :dry, :material

  def initialize(name, material: :normal)
    @name     = name
    @material = material   # :normal, :delicate, :heavy
    @clean    = false
    @dry      = false
  end

  def wash!(mode)
    @clean = true
    @last_wash_mode = mode
  end

  def dry!
    raise "洗濯(せんたく)してからでないと乾燥(かんそう)できません" unless @clean
    @dry = true
  end

  def recommended_mode
    case @material
    when :delicate then :delicate
    when :heavy    then :heavy_duty
    else                :standard
    end
  end

  def status_line
    clean_status = @clean ? "✅清潔(せいけつ)" : "🔴汚(よご)れている"
    dry_status   = @dry   ? "☀️乾燥済(かんそうず)み" : "💧未乾燥(みかんそう)"
    "#{@name.ljust(12)} #{clean_status} #{dry_status}"
  end
end

class WashingMachine
  attr_reader :mode

  def initialize
    @clothes = []
    @mode    = :standard
    @running = false
  end

  def add(clothing)
    raise "洗濯機(せんたくき)が満杯(まんぱい)です" if full?
    raise "運転中(うんてんちゅう)は追加(ついか)できません" if @running
    @clothes << clothing
    puts "追加(ついか): #{clothing.name} (推奨(すいしょう)モード: #{clothing.recommended_mode})"
  end

  def set_mode(mode)
    raise "不明(ふめい)なモード" unless Clothing::WASH_MODES.key?(mode)
    @mode = mode
    puts "モード設定(せってい): #{mode}"
  end

  def run
    raise "洗濯物(せんたくもの)がありません" if @clothes.empty?
    @running = true
    config   = Clothing::WASH_MODES[@mode]
    puts ""
    puts "=== 洗濯(せんたく)開始(かいし) ==="
    puts "モード: #{@mode} | 時間(じかん): #{config[:time]}分(ふん) | 水量(すいりょう): #{config[:water]}L"
    puts ""
    @clothes.each { |c| c.wash!(@mode) }
    puts "洗濯(せんたく)完了(かんりょう)！"
    @running = false
  end

  def dry_all
    puts ""
    puts "=== 乾燥(かんそう)開始(かいし) ==="
    errors = []
    @clothes.each do |c|
      begin
        c.dry!
      rescue => e
        errors << "#{c.name}: #{e.message}"
      end
    end
    errors.each { |e| puts "⚠️ #{e}" }
    puts "乾燥(かんそう)完了(かんりょう)！"
  end

  def status
    puts ""
    puts "=== 洗濯機(せんたくき)の状態(じょうたい) ==="
    @clothes.each { |c| puts c.status_line }
    puts "(#{@clothes.length}/#{Clothing::MAX_CAPACITY}着(ちゃく))"
  end

  def full? = @clothes.length >= Clothing::MAX_CAPACITY
end

# 실행
machine = WashingMachine.new

clothes = [
  Clothing.new("シャツ",     material: :normal),
  Clothing.new("シルクスカーフ", material: :delicate),
  Clothing.new("デニムジャケット", material: :heavy),
  Clothing.new("ズボン",     material: :normal),
]

clothes.each { |c| machine.add(c) }
machine.status

machine.set_mode(:standard)
machine.run
machine.dry_all
machine.status
```

**포인트:**

- `WASH_MODES` 상수 해시 → 설정값을 코드에 직접 쓰지 않고 데이터로 분리
- `raise ... if @running` → 가드 절(guard clause) — 조건 충족 시 조기 종료
- `.ljust(12)` → 문자열을 지정 길이로 왼쪽 정렬 (출력 정렬에 편리)
- `errors = []` + 루프 안 `rescue` → 에러가 나도 전체를 중단하지 않고 수집

---

## 🔑 한눈에 보는 키워드

| 키워드                         | 설명                                    |
| :----------------------------- | :-------------------------------------- |
| `Time.now`                     | 현재 시각                               |
| `Time.new(y, m, d, h, min, s)` | 특정 시각 생성                          |
| `time.strftime("포맷")`        | 시각을 원하는 형식으로 출력             |
| `Date.today`                   | 오늘 날짜                               |
| `Date.new(y, m, d)`            | 특정 날짜 생성                          |
| `date + n` / `date - n`        | n일 후 / n일 전                         |
| `date >> n` / `date << n`      | n개월 후 / n개월 전                     |
| `(date2 - date1).to_i`         | 날짜 차이 (일 수)                       |
| `attr_reader`                  | 읽기 전용 접근자                        |
| `attr_accessor`                | 읽기 + 쓰기 접근자                      |
| `private`                      | 이후 메서드를 외부에서 호출 불가로 설정 |
| `CONSTANT = 값`                | 상수 (대문자로 시작, 변경 불가)         |
| `클래스::상수`                 | 다른 클래스의 상수에 접근               |
| `메서드?`                      | 진위값 반환 메서드 관례                 |
| `메서드!`                      | 파괴적 메서드 관례 (원본 변경)          |
| `배열.delete(값)`              | 배열에서 특정 값 제거                   |
| `String.ljust(n)`              | 문자열을 n자리로 왼쪽 정렬              |
| `str[-4..]`                    | 마지막 4번째부터 끝까지 슬라이스        |
| SRP (단일 책임 원칙)           | 클래스는 하나의 책임만 가져야 한다      |
| 다형성                         | 같은 메서드명으로 클래스마다 다른 동작  |
| 캡슐화                         | 내부 구현을 숨기고 인터페이스만 노출    |

---

[← 목차로 돌아가기 / 目次(もくじ)に戻(もど)る](../README.md)
