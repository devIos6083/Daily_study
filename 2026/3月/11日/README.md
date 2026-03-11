# 📅 Day 19 — Ruby 기초 (출력 & 문자열 & 숫자 & 연산자)

**날짜 / 日付(ひづけ):** 2026.03.11  
**학습 시간 / 学習時間(がくしゅうじかん):** 자율 / 自律(じりつ)  
**한 줄 목표:** Ruby의 출력 메서드 차이를 이해하고, 문자열·숫자·연산자의 기본 사용법을 익힌다.  
**一言目標(ひとこともくひょう):** Rubyの出力(しゅつりょく)メソッドの違(ちが)いを理解(りかい)し、文字列(もじれつ)・数値(すうち)・演算子(えんざんし)の基本(きほん)を身(み)につける。

---

## 목차 / 目次(もくじ)

1. [출력 메서드 — puts / print / p](#1-출력-메서드--puts--print--p)
2. [문자열 (String)](#2-문자열-string)
3. [숫자 (Numeric)](#3-숫자-numeric)
4. [연산자 (Operator)](#4-연산자-operator)

---

## 1. 출력 메서드 — puts / print / p

Ruby에는 출력 메서드가 3가지 있으며, **목적에 따라 구분해서 사용**한다.  
Rubyには出力(しゅつりょく)メソッドが3つあり、**目的(もくてき)に応(おう)じて使(つか)い分(わ)ける**。

### 비교 한눈에 보기

| 메서드  |  줄바꿈   |   배열 처리    |  따옴표  | 주 용도               |
| :-----: | :-------: | :------------: | :------: | :-------------------- |
| `puts`  | 자동 추가 | 요소를 한 줄씩 |   없음   | 일반적인 출력         |
| `print` |   없음    |  배열 그대로   |   없음   | 줄바꿈 없이 이어 출력 |
|   `p`   | 자동 추가 |  배열 그대로   | **있음** | 디버깅 (타입 확인)    |

### `puts` — 자동 줄바꿈, 배열은 한 줄씩

```ruby
puts "puts: Hello! World"   # → puts: Hello! World\n
puts 1234                   # → 1234
puts 3.14                   # → 3.14

# 배열을 puts하면 요소를 한 줄씩 출력
puts ["Ruby", "Shell", "Python"]
# → Ruby
#   Shell
#   Python
```

### `print` — 줄바꿈 없음, `\n`으로 직접 제어

```ruby
print "print: Hello "
print "World"
print "\n"          # 줄바꿈을 직접 써야 함
print 123
print "\n"

# 배열을 print하면 그대로 출력
print ["Ruby", "Shell", "Python"]
print "\n"
# → ["Ruby", "Shell", "Python"]
```

### `p` — 타입 정보 포함 출력, 디버깅용

```ruby
p "p: Hello World"          # → "p: Hello World"  ← 따옴표 포함!
p 123                       # → 123
p 3.14                      # → 3.14
p nil                       # → nil
p true                      # → true
p ["Ruby", "Shell", "Python"]  # → ["Ruby", "Shell", "Python"]
```

### 실제 차이 비교

```ruby
puts "putsは引用なし"   # → putsは引用なし     (따옴표 없음)
p    "pは引用あり"      # → "pは引用あり"      (따옴표 있음 → 문자열임을 명시)
print "printは改行なし" # → printは改行なし    (다음 출력이 바로 이어짐)
puts "putsは改行あり"   # → putsは改行あり\n
```

> **언제 어떤 걸 쓰는가?**  
> `puts` — 화면에 보여주는 일반 출력  
> `p` — 개발 중 값의 타입·내용 확인 (디버깅)  
> `print` — 줄바꿈 없이 이어 붙여야 할 때

---

## 2. 문자열 (String)

### 싱글쿼트 vs 더블쿼트

```ruby
single = 'シングルクォート: 文字列をそのまま出力する'
double = "ダブルクォート: 変数展開や特殊文字が使える"

name = "カンホンギュ"

# #{} 변수 전개는 더블쿼트에서만 동작!
puts "私の名前は#{name}です"   # → 私の名前はカンホンギュです
puts '私の名前は#{name}です'   # → 私の名前は#{name}です  (그대로 출력)
```

### 문자열 길이

```ruby
puts "hello".length   # → 5
puts "hello".size     # → 5  (length와 동일, 별칭)
```

### 대소문자 변환

```ruby
puts "hello".upcase    # → HELLO
puts "HELLO".downcase  # → hello
puts "hello".reverse   # → olleh
```

### 포함 여부 확인

```ruby
puts "hello world".include?("world")  # → true
puts "hello world".include?("study")  # → false
```

### 치환

```ruby
# gsub: 일치하는 모든 부분을 치환 (global substitution)
puts "hello world".gsub("l", "r")   # → herro worrd

# sub: 처음 일치하는 부분만 치환 (substitution)
puts "hello world".sub("l", "r")    # → herlo world
```

### 분리 / 공백 제거

```ruby
# split: 구분자로 나눠서 배열로 반환
puts "hello world ruby".split(" ").inspect
# → ["hello", "world", "ruby"]

# strip: 앞뒤 공백 제거
puts " hello ".strip   # → "hello"
```

### 결합 / 반복

```ruby
puts "hello" + "world"   # → helloworld
puts "hello" * 3         # → hellohellohello
```

### 시작/끝 확인

```ruby
puts "hello world".start_with?("hello")   # → true
puts "hello world".end_with?("world")     # → true
```

### 기타 유용한 메서드

```ruby
# chars: 문자 하나씩 배열로
puts "hello".chars.inspect   # → ["h", "e", "l", "l", "o"]

# count: 특정 문자 등장 횟수
puts "hello world".count("l")   # → 3

# replace: 문자열 내용을 직접 교체 (파괴적 메서드)
str = "hello"
str.replace("world")
puts str   # → world
```

---

## 3. 숫자 (Numeric)

### 정수 사칙연산

```ruby
puts 10 + 3    # → 13
puts 10 - 3    # → 7
puts 10 * 3    # → 30
puts 10 / 3    # → 3  ← 정수끼리 나누면 소수점 버려짐!
puts 10 % 3    # → 1  (나머지)
puts 10 ** 2   # → 100 (거듭제곱)
```

### 소수(Float) 연산

```ruby
puts 10.0 / 3    # → 3.3333...  (Float끼리면 소수점 유지)
puts 1.5 + 2.3   # → 3.8
puts 3.14 * 2    # → 6.28
```

### 타입 변환

```ruby
puts 3.14.to_i   # → 3      (Float → Integer, 소수점 버림)
puts 3.to_f      # → 3.0    (Integer → Float)
```

### 반올림 / 올림 / 내림

```ruby
puts 3.14.round   # → 3   (반올림)
puts 3.14.ceil    # → 4   (올림 — ceiling)
puts 3.14.floor   # → 3   (내림 — floor)
```

### 짝수 / 홀수 판별

```ruby
puts 10.even?   # → true  (짝수)
puts 11.odd?    # → true  (홀수)
puts 15.even?   # → false
puts 16.odd?    # → false
```

### 절댓값

```ruby
puts -5.abs   # → 5
puts  5.abs   # → 5
```

### 유리수 / 복소수

```ruby
# Rational: 분수를 정확하게 표현
r = Rational(1, 3)
puts r              # → 1/3
puts r + Rational(1, 6)   # → 1/2  (분수 계산이 정확함)
puts r * 3          # → 1

# Complex: 복소수 (실수부 + 허수부)
c = Complex(2, 3)
puts c                     # → 2+3i
puts c + Complex(1, 2)     # → 3+5i
puts c * Complex(1, 2)     # → -4+7i
```

---

## 4. 연산자 (Operator)

### 산술 연산자

```ruby
puts 10 + 3    # → 13
puts 10 - 3    # → 7
puts 10 * 3    # → 30
puts 10 / 3    # → 3
puts 10 % 3    # → 1
puts 10 ** 3   # → 1000
```

### 문자열에서의 연산

```ruby
puts "Hello" + "World"   # → HelloWorld  (연결)
puts "Hello" * 3         # → HelloHelloHello  (반복)
```

### 복합 대입 연산자

```ruby
x = 10
x += 10;  puts x   # → 20  (x = x + 10)
x -= 3;   puts x   # → 17  (x = x - 3)
x *= 2;   puts x   # → 34  (x = x * 2)
x /= 4;   puts x   # → 8   (x = x / 4)
x **= 2;  puts x   # → 64  (x = x ** 2)
x %= 3;   puts x   # → 1   (x = x % 3)
```

### 비교 연산자

```ruby
puts 10 > 5    # → true
puts 10 < 5    # → false
puts 10 == 10  # → true
puts 10 != 5   # → true
puts 10 >= 10  # → true
puts 9  <= 10  # → true
```

### `<=>` — 우주선 연산자 (Spaceship Operator)

```ruby
# 왼쪽이 크면 1, 같으면 0, 작으면 -1 반환
# 정렬(sort) 로직에서 내부적으로 사용됨
puts 10 <=> 5    # → 1   (10 > 5)
puts 10 <=> 10   # → 0   (같음)
puts 5  <=> 10   # → -1  (5 < 10)
```

### 논리 연산자

```ruby
puts true  && true    # → true   (AND: 둘 다 true)
puts true  && false   # → false
puts true  || false   # → true   (OR: 하나라도 true)
puts false || false   # → false
puts !true            # → false  (NOT: 반전)
puts !false           # → true
```

### 삼항 연산자

```ruby
# 조건 ? 참일 때 값 : 거짓일 때 값
puts 10 > 5 ? "10は5より大きい" : "10は5より小さい"
# → 10は5より大きい

age = 20
puts age >= 20 ? "成人です" : "未成年です"
# → 成人です
```

### 연산자 우선순위

```ruby
# 수학과 같은 규칙: * / 이 + - 보다 먼저!
puts 2 + 3 * 4        # → 14  (3*4 먼저)
puts (2 + 3) * 4      # → 20  (괄호 먼저)
puts 2 + 3 * 4 - 1    # → 13
puts 10 / 2 + 3 * 4   # → 17
```

---

## 🔑 한눈에 보는 키워드

| 키워드                        | 설명                                            |
| :---------------------------- | :---------------------------------------------- |
| `puts`                        | 출력 + 자동 줄바꿈, 배열은 한 줄씩              |
| `print`                       | 출력, 줄바꿈 없음                               |
| `p`                           | 타입 포함 출력 (디버깅용), 문자열은 따옴표 포함 |
| `"#{변수}"`                   | 더블쿼트 안 변수 전개                           |
| `.length` / `.size`           | 문자열 길이                                     |
| `.upcase` / `.downcase`       | 대소문자 변환                                   |
| `.include?`                   | 문자열 포함 여부                                |
| `.gsub(a, b)`                 | 전체 치환                                       |
| `.sub(a, b)`                  | 첫 번째만 치환                                  |
| `.split(구분자)`              | 문자열 → 배열로 분리                            |
| `.strip`                      | 앞뒤 공백 제거                                  |
| `.chars`                      | 문자 하나씩 배열로                              |
| `.count("문자")`              | 특정 문자 등장 횟수                             |
| `.to_i` / `.to_f`             | Float→Integer / Integer→Float 변환              |
| `.round` / `.ceil` / `.floor` | 반올림 / 올림 / 내림                            |
| `.even?` / `.odd?`            | 짝수 / 홀수 판별                                |
| `.abs`                        | 절댓값                                          |
| `**`                          | 거듭제곱                                        |
| `%`                           | 나머지 (modulo)                                 |
| `<=>`                         | 우주선 연산자 (-1 / 0 / 1 반환)                 |
| `조건 ? A : B`                | 삼항 연산자                                     |

---

[← 목차로 돌아가기 / 目次(もくじ)に戻(もど)る](../README.md)
