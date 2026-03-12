# 📅 Day 20 — Ruby 기초 (조건문 & 메서드 & 심볼 & 배열)

**날짜 / 日付(ひづけ):** 2026.03.12  
**학습 시간 / 学習時間(がくしゅうじかん):** 자율 / 自律(じりつ)  
**한 줄 목표:** Ruby의 조건문·메서드·심볼·배열을 이해하고, 각 개념을 응용한 실전 코드를 작성한다.  
**一言目標(ひとこともくひょう):** Rubyの条件文(じょうけんぶん)・メソッド・シンボル・配列(はいれつ)を理解(りかい)し、各概念(かくがいねん)を応用(おうよう)した実践(じっせん)コードを書(か)く。

---

## 목차 / 目次(もくじ)

1. [조건문 (Conditional Statement)](#1-조건문-conditional-statement)
2. [메서드 (Method)](#2-메서드-method)
3. [심볼 (Symbol)](#3-심볼-symbol)
4. [배열 (Array)](#4-배열-array)

---

## 1. 조건문 (Conditional Statement)

**조건의 참/거짓에 따라 실행할 코드를 나누는 구조**.  
Ruby는 `if` 외에도 `unless`, `case` 등 다양한 조건 표현이 있어서  
코드를 더 **자연어에 가깝게** 읽히도록 작성할 수 있다.

**条件(じょうけん)の真偽(しんぎ)によって実行(じっこう)するコードを分(わ)ける構造(こうぞう)**。  
Rubyは`if`以外(いがい)にも`unless`・`case`など多様(たよう)な条件表現(じょうけんひょうげん)がある。

### if / elsif / else

```ruby
score = 75

if score >= 90
  puts "優秀"
elsif score >= 70
  puts "合格"   # → score가 75이므로 여기 출력
else
  puts "不合格"
end
# 주의: Ruby는 elsif (else if가 아님!)
```

### unless — "~가 아니라면"

```ruby
# unless는 if의 반대
# "score < 70이 아니라면" = "score >= 70이라면"
unless score < 70
  puts "合格しました"   # → 출력됨
end

# 한 줄로도 쓸 수 있음 — 후위 조건문
puts "合格"    if     score >= 70   # 조건이 참이면 실행
puts "不合格"  unless score >= 70   # 조건이 거짓이면 실행
```

### case / when — 여러 분기

```ruby
grade = "B"

case grade
when "A"
  puts "優秀"
when "B"
  puts "良好"   # → 출력됨
when "C"
  puts "普通"
when "D"
  puts "不合格"
end

# case는 범위(Range)와 함께 쓸 수 있음
case score
when 90..100
  puts "優秀"
when 70..89
  puts "合格"   # → score가 75이므로 출력
else
  puts "不合格"
end
```

### Ruby의 진리값 — 중요!

```ruby
# Ruby에서 false와 nil만 거짓 — 나머지는 모두 참!
# !! : 값을 boolean(true/false)으로 변환하는 관용구
puts "false: #{!!false}"   # → false
puts "nil: #{!!nil}"       # → false
puts "0: #{!!0}"           # → true  ← 다른 언어와 다름!
puts "空文字: #{!!""}"     # → true  ← 빈 문자열도 true!
puts "空配列: #{!![]}"     # → true  ← 빈 배열도 true!
```

### if는 값을 반환한다

```ruby
# Ruby에서 if 자체가 값을 반환하므로 변수에 담을 수 있음
result = if score >= 70
  "合格"     # 이 값이 result에 담김
else
  "不合格"
end
puts result   # → 合格
```

---

### 🔥 도전 코드 — 조건문

**여러 조건을 조합해서 세금 계산기를 만들어보자.**

```ruby
# 일본 소비세 계산기
# 식품(food)은 8%, 일반 상품은 10%
# 금액이 0 이하면 에러 처리

def calculate_tax(price, category)
  # 입력값 검증 — 0 이하면 에러
  unless price > 0
    puts "エラー: 金額(きんがく)は0より大(おお)きくしてください"
    return nil   # nil을 반환하고 메서드 종료
  end

  # category에 따라 세율 결정
  tax_rate = case category
  when :food        then 0.08   # 식품 → 8%
  when :general     then 0.10   # 일반 → 10%
  else
    puts "エラー: 不明(ふめい)なカテゴリです"
    return nil
  end

  tax     = (price * tax_rate).round   # 소수점 반올림
  total   = price + tax

  puts "商品(しょうひん)金額(きんがく): #{price}円(えん)"
  puts "税率(ぜいりつ): #{(tax_rate * 100).to_i}%"
  puts "消費税(しょうひぜい): #{tax}円(えん)"
  puts "合計(ごうけい): #{total}円(えん)"
  puts "---"
end

calculate_tax(1000, :food)     # 식품 8%
calculate_tax(1000, :general)  # 일반 10%
calculate_tax(0, :food)        # 에러 처리
calculate_tax(500, :luxury)    # 미지정 카테고리 에러
```

**포인트:**

- `unless price > 0` → `if price <= 0` 과 같은 의미지만 더 자연스럽게 읽힘
- `case`가 값을 반환하므로 `tax_rate =` 에 바로 담을 수 있음
- `return nil` 로 조기 종료 — 이후 코드가 실행되지 않음

---

## 2. 메서드 (Method)

**반복 사용할 처리를 이름 붙여 묶어놓은 것**.  
Ruby에서 메서드는 `def ~ end` 로 정의하고,  
**마지막에 평가된 값이 자동으로 반환**된다 (`return` 생략 가능).

**繰(く)り返(かえ)し使(つか)う処理(しょり)に名前(なまえ)をつけてまとめたもの**。  
Rubyでは最後(さいご)に評価(ひょうか)された値(あたい)が自動(じどう)で返(かえ)される。

### 기본 메서드

```ruby
def greet
  puts "Hello!"
end
greet   # → Hello!
```

### 반환값 — return 생략 가능

```ruby
# return을 명시적으로 쓰는 경우
def add(a, b)
  return a + b
end
puts add(2, 3)   # → 5

# 마지막 식이 자동 반환 — Ruby 스타일
def multiply(a, b)
  a * b   # return 없어도 이 값이 반환됨
end
puts multiply(2, 3)   # → 6
```

### `?` 로 끝나는 메서드 — 진위값 반환 관례

```ruby
# Ruby 관례: 참/거짓을 반환하는 메서드는 이름 끝에 ? 를 붙임
def even?(number)
  number % 2 == 0   # true 또는 false 반환
end
puts even?(4)   # → true
puts even?(3)   # → false
```

### 기본값 파라미터

```ruby
# 인수가 없으면 기본값 사용
def greet_with_name(name = "ゲスト")
  puts "こんにちは#{name}さん"
end
greet_with_name                # → こんにちはゲストさん
greet_with_name("カンホンギュ")  # → こんにちはカンホンギュさん
```

### 키워드 인수 — 순서 자유, 명시적

```ruby
# name:, age: 처럼 키워드로 지정
# 호출할 때 순서를 바꿔도 됨
def profile(name:, age:)
  puts "名前: #{name}, 年齢: #{age}歳"
end
profile(name: "カンホンギュ", age: 26)   # → 名前: カンホンギュ, 年齢: 26歳
profile(age: 26, name: "カンホンギュ")   # 순서 바꿔도 동일하게 동작
```

### `!` 메서드 — 파괴적 메서드 (원본 변경)

```ruby
string = "hello"
puts string.upcase    # → HELLO  (원본 변경 없음)
puts string          # → hello   (원본 그대로)

string.upcase!       # ! 붙이면 원본을 직접 변경
puts string          # → HELLO   (원본이 바뀜!)
```

### 객체 참조와 메서드

```ruby
# Ruby의 문자열은 참조(reference)로 전달됨
# .replace는 같은 객체의 내용을 바꾸는 파괴적 메서드
def replace_string(string)
  string.replace("world")   # 전달받은 객체 자체를 변경
end

string = "hello"
replace_string(string)
puts string   # → world  (원본이 바뀜!)
```

---

### 🔥 도전 코드 — 메서드

**메서드를 조합해서 성적 처리 시스템을 만들어보자.**

```ruby
# 점수 배열을 받아서 통계와 등급을 출력하는 메서드 모음

# 평균 계산 — inject로 합산
def average(scores)
  scores.inject(0) { |sum, score| sum + score } / scores.length.to_f
end

# 최고/최저 점수
def max_score(scores) = scores.max   # Ruby 3.0+ 한 줄 메서드 문법
def min_score(scores) = scores.min

# 점수 → 등급 변환
def grade(score)
  case score
  when 90..100 then "A"
  when 80..89  then "B"
  when 70..79  then "C"
  when 60..69  then "D"
  else              "F"
  end
end

# 합격 여부 (? 관례)
def pass?(score)
  score >= 60
end

# 전체 리포트 출력
def print_report(name, scores)
  avg = average(scores).round(1)

  puts "=== #{name}の成績(せいせき)レポート ==="
  puts "点数(てんすう): #{scores.inspect}"
  puts "平均(へいきん): #{avg}点(てん) → #{grade(avg.to_i)}評価(ひょうか)"
  puts "最高(さいこう): #{max_score(scores)}点(てん)"
  puts "最低(さいてい): #{min_score(scores)}点(てん)"
  puts "合否(ごうひ): #{pass?(avg) ? "合格(ごうかく)" : "不合格(ふごうかく)"}"
  puts ""
end

print_report("カンホンギュ", [88, 72, 95, 63, 80])
print_report("田中(たなか)", [55, 48, 60, 52, 58])
```

**포인트:**

- `inject(초기값) { |누적, 요소| }` → 배열 전체를 누적 계산 (합계, 곱 등)
- `def 메서드명(인수) = 식` → Ruby 3.0+ 한 줄 메서드
- 메서드를 작게 나눠서 조합하는 것이 Ruby 스타일

---

## 3. 심볼 (Symbol)

**콜론(`:`)으로 시작하는 변경 불가능한 이름 표지**.  
문자열과 비슷해 보이지만 **같은 심볼은 항상 같은 객체 ID**를 가져서  
해시 키나 상태 표현에 문자열보다 **메모리 효율이 좋고 비교가 빠르다**.

**コロン(`:`)で始(はじ)まる変更不可(へんこうふか)な名前標識(なまえひょうしき)**。  
同(おな)じシンボルは常(つね)に同(おな)じオブジェクトIDを持(も)ち、ハッシュキーとして効率(こうりつ)がいい。

### 심볼의 특징 — object_id 비교

```ruby
# 문자열: 같은 내용이어도 매번 새로운 객체 생성 → object_id 다름
puts "hello".object_id   # → 어떤 숫자 (매번 다름)
puts "hello".object_id   # → 다른 숫자

# 심볼: 같은 심볼은 항상 같은 객체 → object_id 동일
puts :hello.object_id    # → 어떤 숫자
puts :hello.object_id    # → 완전히 같은 숫자!
```

### 심볼 ↔ 문자열 변환

```ruby
symbol = :hello
puts symbol.to_s         # → "hello"  (심볼 → 문자열)
puts symbol.to_s.class   # → String

string = "hello"
puts string.to_sym       # → :hello   (문자열 → 심볼)
puts string.to_sym.class # → Symbol
```

### 해시 키로 사용 — 가장 흔한 용도

```ruby
# 심볼을 키로 쓰는 해시 (최신 Ruby 스타일)
person = { name: "カンホンギュ", age: 26 }
puts person[:name]   # → カンホンギュ
puts person[:age]    # → 26

# 비교: 문자열 키 vs 심볼 키
hash_string = { "name" => "カンホンギュ" }   # 문자열 키
hash_symbol = { name: "カンホンギュ" }        # 심볼 키 (권장)

puts hash_string["name"]   # → カンホンギュ  (문자열로 접근)
puts hash_symbol[:name]    # → カンホンギュ  (심볼로 접근)
```

### 심볼로 상태 표현

```ruby
# 문자열 대신 심볼로 상태값 표현
# → 오타를 내도 다른 심볼이 되어버리므로 방어적 코딩에 좋음
status = :inactive
puts status   # → inactive
status = :active
puts status   # → active
```

### 심볼에서 사용 가능한 메서드

```ruby
puts :hello.upcase             # → HELLO
puts :hello.length             # → 5
puts :hello.to_s.reverse.to_sym  # → :olleh
```

---

### 🔥 도전 코드 — 심볼

**심볼을 상태 머신(State Machine)으로 활용해보자.**

```ruby
# 주문 상태 관리 시스템
# 상태를 문자열 대신 심볼로 관리 → 비교가 빠르고 오타 위험이 줄어듦

VALID_TRANSITIONS = {
  :pending  => [:confirmed, :cancelled],   # 대기중 → 확정 또는 취소
  :confirmed => [:shipped, :cancelled],    # 확정됨 → 배송 또는 취소
  :shipped   => [:delivered],              # 배송중 → 배달 완료
  :delivered => [],                        # 배달 완료 → 더 이상 변경 불가
  :cancelled => []                         # 취소됨 → 더 이상 변경 불가
}

def transition_order(current_status, next_status)
  # 현재 상태에서 다음 상태로 이동 가능한지 확인
  allowed = VALID_TRANSITIONS[current_status]

  if allowed.nil?
    puts "エラー: 不明(ふめい)な状態(じょうたい) → #{current_status}"
  elsif allowed.include?(next_status)
    puts "#{current_status} → #{next_status} : 遷移(せんい)成功(せいこう)!"
    next_status   # 새로운 상태 반환
  else
    puts "エラー: #{current_status} → #{next_status} は許可(きょか)されていません"
    current_status   # 상태 변경 없이 현재 상태 반환
  end
end

# 정상 흐름
status = :pending
status = transition_order(status, :confirmed)   # pending → confirmed: 성공
status = transition_order(status, :shipped)     # confirmed → shipped: 성공
status = transition_order(status, :delivered)   # shipped → delivered: 성공

puts ""

# 비정상 흐름
status = :pending
transition_order(status, :delivered)   # pending → delivered: 에러 (건너뜀)
transition_order(status, :cancelled)   # pending → cancelled: 성공
```

**포인트:**

- 심볼은 `==` 비교가 문자열보다 빠름 → 상태값으로 최적
- 해시의 키와 값을 모두 심볼로 → 메모리 효율 극대화
- `VALID_TRANSITIONS` 처럼 상수는 대문자로 정의하는 것이 Ruby 관례

---

## 4. 배열 (Array)

**여러 값을 순서대로 담는 컨테이너**.  
Ruby의 배열은 **타입 혼합이 자유롭고, 다양한 메서드**로 데이터를 가공할 수 있다.  
Rails에서 데이터를 다룰 때 가장 자주 쓰이는 구조다.

**複数(ふくすう)の値(あたい)を順番(じゅんばん)に格納(かくのう)するコンテナ**。  
Rubyの配列(はいれつ)は型混合(かたこんごう)が自由(じゆう)で、多様(たよう)なメソッドでデータを加工(かこう)できる。

### 기본 조작 — push / pop / shift / unshift

```ruby
array = [1, 2, 3, 4, 5]

array.push(6)    # 맨 뒤에 추가 → [1, 2, 3, 4, 5, 6]
array.pop        # 맨 뒤에서 제거 → [1, 2, 3, 4, 5]
array.unshift(0) # 맨 앞에 추가 → [0, 1, 2, 3, 4, 5]
array.shift      # 맨 앞에서 제거 → [1, 2, 3, 4, 5]

puts array[0]    # → 1   (첫 번째)
puts array[-1]   # → 5   (마지막 — 음수 인덱스!)
```

### 배열 연산자

```ruby
puts ([1, 2, 3] + [4, 5, 6]).inspect       # → [1,2,3,4,5,6]  (합치기)
puts ([1, 2, 3, 4, 5] - [2]).inspect       # → [1,3,4,5]      (제거)
puts ([1, 2] * 3).inspect                  # → [1,2,1,2,1,2]  (반복)
puts ([1, 2, 3] & [2, 3]).inspect          # → [2,3]           (교집합)
puts ([1, 2, 3] | [3, 4, 5]).inspect       # → [1,2,3,4,5]    (합집합)
```

### each — 순회 (값 반환 없음)

```ruby
[1, 2, 3].each do |number|
  puts number
end
# → 1, 2, 3 출력
# each의 반환값은 원본 배열 그대로 (가공하지 않음)
```

### map — 변환 (새 배열 반환)

```ruby
# 각 요소를 변환한 새 배열 반환
result = [1, 2, 3].map { |number| number * 2 }
puts result.inspect   # → [2, 4, 6]
```

### select / reject — 필터링

```ruby
# select: 조건에 맞는 요소만 남김
puts [1, 2, 3, 4, 5].select { |n| n.even? }.inspect
# → [2, 4]

# reject: 조건에 맞는 요소를 제거
puts [1, 2, 3, 4, 5].reject { |n| n.even? }.inspect
# → [1, 3, 5]
```

### each vs map — 핵심 차이

```ruby
array = [1, 2, 3]

# each: 블록 안에서 무엇을 해도 원본 배열을 반환
each_result = array.each { |n| n * 2 }
puts each_result.inspect   # → [1, 2, 3]  (변화 없음!)

# map: 블록의 반환값으로 새 배열을 만들어 반환
map_result = array.map { |n| n * 2 }
puts map_result.inspect    # → [2, 4, 6]  (새 배열!)
```

---

### 🔥 도전 코드 — 배열

**배열 메서드를 체이닝해서 성적 분석기를 만들어보자.**

```ruby
students = [
  { name: "カンホンギュ", scores: [88, 72, 95, 63, 80] },
  { name: "田中(たなか)",   scores: [55, 48, 60, 52, 58] },
  { name: "佐藤(さとう)",   scores: [92, 88, 95, 97, 90] },
  { name: "鈴木(すずき)",   scores: [70, 65, 73, 68, 72] },
]

# 각 학생의 평균 점수 계산
averages = students.map do |student|
  avg = student[:scores].sum / student[:scores].length.to_f
  { name: student[:name], average: avg.round(1) }
end

# 평균 점수 내림차순 정렬
ranked = averages.sort_by { |s| -s[:average] }  # - 를 붙이면 내림차순

# 합격자만 필터링 (평균 60점 이상)
passed = ranked.select { |s| s[:average] >= 60 }
failed = ranked.reject { |s| s[:average] >= 60 }

puts "=== 成績(せいせき)ランキング ==="
ranked.each.with_index(1) do |student, rank|
  # with_index(1): 인덱스를 1부터 시작
  status = student[:average] >= 60 ? "合格(ごうかく)" : "不合格(ふごうかく)"
  puts "#{rank}位(い): #{student[:name]} #{student[:average]}点(てん) [#{status}]"
end

puts ""
puts "合格者数(ごうかくしゃすう): #{passed.length}名(めい)"
puts "不合格者数(ふごうかくしゃすう): #{failed.length}名(めい)"

# 전체 평균
all_avg = averages.map { |s| s[:average] }.sum / averages.length
puts "クラス平均(へいきん): #{all_avg.round(1)}点(てん)"
```

**포인트:**

- `map` → `sort_by` → `select` 처럼 메서드 체이닝으로 데이터 파이프라인 구성
- `sort_by { |s| -s[:average] }` → 음수를 붙여서 내림차순 정렬하는 Ruby 관용구
- `.each.with_index(1)` → 인덱스를 1부터 세는 방법
- `배열.sum` → Ruby 내장 합계 메서드 (inject 없이 간결하게)

---

## 🔑 한눈에 보는 키워드

| 키워드                          | 설명                               |
| :------------------------------ | :--------------------------------- |
| `if / elsif / else / end`       | 기본 조건문 (elsif는 하나의 단어!) |
| `unless 조건`                   | `if !조건` 과 동일                 |
| `puts "..." if 조건`            | 후위 조건문 — 한 줄로              |
| `case / when / else / end`      | 다중 분기 — 범위(`..`)도 사용 가능 |
| `!!값`                          | 값을 boolean으로 변환              |
| `false`, `nil`                  | Ruby에서 거짓인 값 (0, "" 도 참!)  |
| `def 메서드명 / end`            | 메서드 정의                        |
| `메서드명?`                     | 진위값 반환 메서드 관례            |
| `메서드명!`                     | 파괴적 메서드 관례 (원본 변경)     |
| `def 메서드(파라미터 = 기본값)` | 기본값 파라미터                    |
| `def 메서드(키워드:)`           | 키워드 인수                        |
| `:심볼`                         | 변경 불가능한 이름 표지            |
| `.object_id`                    | 객체의 고유 ID                     |
| `.to_s` / `.to_sym`             | 심볼↔문자열 변환                   |
| `array.push(값)`                | 맨 뒤에 추가                       |
| `array.pop`                     | 맨 뒤에서 제거                     |
| `array.unshift(값)`             | 맨 앞에 추가                       |
| `array.shift`                   | 맨 앞에서 제거                     |
| `array[-1]`                     | 마지막 요소                        |
| `.each`                         | 순회 (원본 반환)                   |
| `.map`                          | 변환 (새 배열 반환)                |
| `.select`                       | 조건에 맞는 요소만 남김            |
| `.reject`                       | 조건에 맞는 요소를 제거            |
| `.sort_by`                      | 기준으로 정렬                      |
| `.inject`                       | 누적 계산 (합계, 곱 등)            |
| `.sum`                          | 배열 합계                          |
| `.with_index(n)`                | 인덱스를 n부터 시작                |

---

[← 목차로 돌아가기 / 目次(もくじ)に戻(もど)る](../README.md)
