# 📅 Day 2 — 블록(Block) & 이터레이터(Iterator)

**날짜 / 日付(ひづけ):** 2025.02.20  
**학습 시간 / 学習時間(がくしゅうじかん):** 2시간 / 2時間(じかん)  
**한 줄 목표:** 블록과 이터레이터를 익혀서, 배열 데이터를 자유롭게 다룰 수 있게 된다.  
**一言目標(ひとこともくひょう):** ブロックとイテレータを覚(おぼ)えて、配列(はいれつ)データを自由(じゆう)に扱(あつか)えるようになる。

---

## 목차 / 目次(もくじ)

1. [블록 (Block)](#1-블록-block)
2. [each — 순회](#2-each--순회)
3. [map — 변환](#3-map--변환)
4. [select / reject — 필터링](#4-select--reject--필터링)
5. [reduce — 압축](#5-reduce--압축)
6. [체이닝 — 이어붙이기](#6-체이닝--이어붙이기)
7. [클래스 + 이터레이터 함께 쓰기](#7-클래스--이터레이터-함께-쓰기)
8. [오늘의 실수 & 배운 점](#8-오늘의-실수--배운-점)

---

## 1. 블록 (Block)

블록은 메서드에 넘겨주는 **코드 조각**입니다.  
같은 메서드도 블록에 따라 다르게 동작하게 만들 수 있습니다.

ブロックはメソッドに渡(わた)す**コードのかけら**です。  
同(おな)じメソッドでも、ブロックによって動作(どうさ)を変(か)えることができます。

```ruby
# 방법 1: do...end — 여러 줄일 때 주로 사용
[1, 2, 3].each do |num|
  puts num
end

# 방법 2: { } — 한 줄일 때 주로 사용
[1, 2, 3].each { |num| puts num }

# |num| 은 블록 파라미터. 배열의 각 요소가 차례로 들어옴
```

---

## 2. each — 순회

배열의 각 요소를 꺼내서 처리합니다. 반환값은 원본 배열 그대로입니다.  
配列(はいれつ)の各(かく)要素(ようそ)を取(と)り出(だ)して処理(しょり)します。戻(もど)り値(ち)は元(もと)の配列(はいれつ)のままです。

```ruby
fruits = ["사과", "바나나", "오렌지"]

fruits.each do |fruit|
  puts "#{fruit}는 맛있다"
end
# → 사과는 맛있다
# → 바나나는 맛있다
# → 오렌지는 맛있다
```

---

## 3. map — 변환

각 요소를 가공해서 **새로운 배열**을 반환합니다.  
Rails에서 DB 데이터를 화면용으로 가공할 때 자주 씁니다.

各(かく)要素(ようそ)を加工(かこう)して**新(あたら)しい配列(はいれつ)**を返(かえ)します。  
RailsでDBのデータを画面用(がめんよう)に変換(へんかん)するときによく使(つか)います。

```ruby
numbers = [1, 2, 3, 4, 5]
doubled = numbers.map { |num| num * 2 }
puts doubled.inspect   # → [2, 4, 6, 8, 10]

names = ["kang", "kim", "lee"]
capitalized = names.map { |name| name.upcase }
puts capitalized.inspect   # → ["KANG", "KIM", "LEE"]
```

---

## 4. select / reject — 필터링

`select`는 조건이 `true`인 요소만 골라서 새 배열을 반환합니다.  
`reject`는 반대로 조건이 `false`인 요소만 남깁니다.

`select`は条件(じょうけん)が`true`の要素(ようそ)だけ選(えら)んで新(あたら)しい配列(はいれつ)を返(かえ)します。  
`reject`は逆(ぎゃく)に条件(じょうけん)が`false`の要素(ようそ)だけ残(のこ)します。

```ruby
numbers = [1, 2, 3, 4, 5, 6, 7, 8]

evens = numbers.select { |n| n.even? }
puts evens.inspect   # → [2, 4, 6, 8]

odds = numbers.reject { |n| n.even? }
puts odds.inspect    # → [1, 3, 5, 7]
```

---

## 5. reduce — 압축

배열 전체를 순회하면서 **하나의 값으로 누적**합니다.  
합계, 곱, 평균 등 배열 전체를 하나로 만들 때 사용합니다.

配列(はいれつ)全体(ぜんたい)を巡(めぐ)りながら**一(ひと)つの値(ち)に積(つ)み上(あ)げ**ます。  
合計(ごうけい)・積(せき)・平均(へいきん)など、配列(はいれつ)全体(ぜんたい)をひとつにまとめるときに使(つか)います。

```ruby
numbers = [1, 2, 3, 4, 5]

# acc: 누적값 (시작값은 괄호 안 숫자)
total = numbers.reduce(0) { |acc, n| acc + n }
puts total    # → 15

product = numbers.reduce(1) { |acc, n| acc * n }
puts product  # → 120
```

---

## 6. 체이닝 — 이어붙이기

이터레이터를 `.`으로 연결하면 복잡한 처리를 한번에 표현할 수 있습니다.  
Rails 실무에서 자주 보이는 패턴입니다.

イテレータを`.`でつなぐと、複雑(ふくざつ)な処理(しょり)を一度(いちど)に表現(ひょうげん)できます。  
Railsの現場(げんば)でよく見(み)られるパターンです。

```ruby
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# "짝수만 골라서, 각각 3배로 만들어라"
result = numbers.select { |n| n.even? }
                .map    { |n| n * 3 }

puts result.inspect   # → [6, 12, 18, 24, 30]
```

---

## 7. 클래스 + 이터레이터 함께 쓰기

Day 1에서 배운 클래스와 오늘 배운 이터레이터를 함께 쓰면 Rails 코드와 거의 같아집니다.

Day1(ディワン)で学(まな)んだクラスと今日(きょう)のイテレータを組(く)み合(あ)わせると、Railsのコードとほぼ同(おな)じになります。

```ruby
module ScoreReport
  def summary
    puts "===== 성적 리포트 ====="
    puts "이름  : #{@name}"
    puts "과목  : #{@subject}"
    puts "점수  : #{@score}점"
    puts "등급  : #{grade}"                              # 클래스의 grade 메서드 호출
    puts "합격여부: #{passed? ? "합격 ✅" : "불합격 ❌"}"  # 클래스의 passed? 메서드 호출
  end
end

class Student
  attr_accessor :name, :score, :subject
  include ScoreReport

  def initialize(name, score, subject)
    @name    = name
    @score   = score
    @subject = subject
  end

  def passed?
    @score >= 60
  end

  def grade
    if    @score >= 90 then "A"
    elsif @score >= 80 then "B"
    elsif @score >= 70 then "C"
    elsif @score >= 60 then "D"
    else "F"
    end
  end

  def info
    "이름: #{@name} | 과목: #{@subject} | 점수: #{@score} | 등급: #{grade}"
  end
end

students = [
  Student.new("홍길동", 85, "Ruby"),
  Student.new("김철수", 55, "Rails"),
  Student.new("이영희", 92, "Ruby"),
  Student.new("박민준", 73, "Rails"),
  Student.new("최지우", 60, "Ruby")
]

# each: 전체 출력
students.each { |s| puts s.info }

# select: 합격자만
students.select { |s| s.passed? }
        .each   { |s| puts s.info }

# map: 이름만 뽑기
names = students.map { |s| s.name }
puts names.inspect   # → ["홍길동", "김철수", "이영희", "박민준", "최지우"]

# reduce: 평균 점수
total   = students.reduce(0) { |acc, s| acc + s.score }
average = total / students.length
puts "평균 점수: #{average}점"

# 체이닝 + sort_by: Ruby 과목 합격자를 점수 높은 순으로
students.select  { |s| s.subject == "Ruby" && s.passed? }
        .sort_by { |s| -s.score }
        .each    { |s| puts "#{s.name} | #{s.score}점 | #{s.grade}등급" }
# → 이영희 | 92점 | A등급
# → 홍길동 | 85점 | B등급

# summary 모듈 사용
student = Student.new("홍길동", 85, "Ruby")
student.summary
```

---

## 8. 오늘의 실수 & 배운 점

오늘 헷갈렸던 부분들을 기록합니다. 실수도 기록해두면 나중에 가장 도움이 됩니다.  
今日(きょう)つまずいた部分(ぶぶん)を記録(きろく)します。失敗(しっぱい)も記録(きろく)しておくと、後(あと)で一番(いちばん)役(やく)に立(た)ちます。

| 실수                      | 원인                                     | 올바른 방법                           |
| :------------------------ | :--------------------------------------- | :------------------------------------ |
| `@score.select { ... }`   | `@score`는 숫자(Integer)라 `select` 불가 | `@score >= 60` 으로 바로 비교         |
| `students.name`           | `students`는 배열 전체, `.name` 없음     | `student.name` (블록 파라미터에 호출) |
| 모듈에서 `@grade` 사용    | `@grade`는 선언한 적 없어서 `nil` 반환   | `grade` 메서드를 그냥 호출            |
| 모듈에서 직접 조건식 작성 | `passed?` 메서드가 이미 있는데 중복      | `passed?` 메서드를 그대로 호출        |

> **핵심 기억:** 모듈은 `include`되는 순간 클래스의 일부가 됩니다.  
> 클래스에 있는 메서드(`grade`, `passed?`)를 모듈 안에서 **그냥 이름만 써서** 호출할 수 있습니다.

---

## 🔑 한눈에 보는 키워드

| 키워드           | 설명                                      |
| :--------------- | :---------------------------------------- |
| `each`           | 각 요소를 순회. 반환값은 원본 배열        |
| `map`            | 각 요소를 변환해서 새 배열 반환           |
| `select`         | 조건이 true인 요소만 모아 새 배열 반환    |
| `reject`         | 조건이 false인 요소만 모아 새 배열 반환   |
| `reduce(초기값)` | 누적값을 만들어 하나의 값으로 압축        |
| `sort_by`        | 블록의 기준으로 정렬. `-` 붙이면 내림차순 |
| `\|변수\|`       | 블록 파라미터. 각 요소가 차례로 들어옴    |
| `.inspect`       | 배열을 `["a", "b"]` 형태로 출력           |
