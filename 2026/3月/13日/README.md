# 📅 Day 21 — Ruby 심화 (해시 & 범위 & 컬렉션 & 정규표현식)

**날짜 / 日付(ひづけ):** 2026.03.13  
**학습 시간 / 学習時間(がくしゅうじかん):** 자율 / 自律(じりつ)  
**한 줄 목표:** 해시·범위·컬렉션의 공통점을 파악하고, 정규표현식으로 문자열 패턴을 다루는 법을 익힌다.  
**一言目標(ひとこともくひょう):** ハッシュ・範囲(はんい)・コレクションの共通点(きょうつうてん)を把握(はあく)し、正規表現(せいきひょうげん)で文字列(もじれつ)パターンを扱(あつか)う方法(ほうほう)を身(み)につける。

---

## 목차 / 目次(もくじ)

1. [해시 (Hash)](#1-해시-hash)
2. [범위 (Range)](#2-범위-range)
3. [컬렉션 (Collections)](#3-컬렉션-collections)
4. [정규표현식 (Regexp)](#4-정규표현식-regexp)

---

## 1. 해시 (Hash)

**키(key)와 값(value)의 쌍으로 데이터를 저장하는 구조**.  
배열이 "순서(번호)"로 값을 찾는다면, 해시는 **"이름(키)"으로 값을 찾는다**.  
Rails에서 파라미터, 설정값, DB 데이터를 주고받을 때 가장 자주 쓰이는 구조다.

**キー(key)と値(あたい)のペアでデータを格納(かくのう)する構造(こうぞう)**。  
配列(はいれつ)が「番号(ばんごう)」で値(あたい)を探(さが)すなら、ハッシュは**「名前(なまえ)(キー)」で値(あたい)を探(さが)す**。

### 해시 생성 — 심볼 키 vs 문자열 키

```ruby
# 심볼 키 — 현대 Ruby에서 권장하는 방식
person = { name: "カンホンギュ", age: 26 }
puts person   # → {:name=>"カンホンギュ", :age=>26}

# 문자열 키 — 외부 API 데이터 등에서 자주 등장
person_with_string = { "name" => "カンホンギュ", "age" => 26 }
puts person_with_string   # → {"name"=>"カンホンギュ", "age"=>26}

# 접근 방법이 다르므로 주의!
puts person[:name]              # → カンホンギュ  (심볼 키)
puts person_with_string["name"] # → カンホンギュ  (문자열 키)
```

### 값 변경 / 추가 / 삭제

```ruby
person = { name: "カンホンギュ", age: 26 }

# 값 변경 — 존재하는 키에 대입
person[:name] = "カン"
puts person   # → {:name=>"カン", :age=>26}

# 키 추가 — 없는 키에 대입하면 새로 추가됨
person[:city] = "東京(とうきょう)"
puts person   # → {:name=>"カン", :age=>26, :city=>"東京"}

# 키 삭제
person.delete(:city)
puts person   # → {:name=>"カン", :age=>26}
```

### merge — 해시 합치기

```ruby
hash_one = { name: "カン" }
hash_two = { age: 26 }

# merge: 두 해시를 합친 새 해시 반환 (원본 변경 없음)
puts hash_one.merge(hash_two)   # → {:name=>"カン", :age=>26}
puts hash_one                   # → {:name=>"カン"}  (원본 그대로)

# merge!: 원본을 직접 변경 (파괴적 메서드)
hash_one.merge!(hash_two)
puts hash_one   # → {:name=>"カン", :age=>26}  (원본이 바뀜!)
```

### 해시 순회 — each

```ruby
person = { name: "カンホンギュ", age: 26, city: "東京(とうきょう)" }

# |key, value| 로 키와 값을 동시에 받음
person.each do |key, value|
  puts "#{key}: #{value}"
end
# → name: カンホンギュ
#   age: 26
#   city: 東京
```

### map / select / reject

```ruby
# map: 각 쌍을 변환 → 배열 반환
puts person.map { |key, value| "#{key}: #{value}" }.inspect
# → ["name: カンホンギュ", "age: 26", "city: 東京"]

# select: 조건에 맞는 쌍만 남김 → 해시 반환
puts person.select { |key, value| value.is_a?(Integer) }.inspect
# → {:age=>26}

# reject: 조건에 맞는 쌍을 제거 → 해시 반환
puts person.reject { |key, value| value.is_a?(Integer) }.inspect
# → {:name=>"カンホンギュ", :city=>"東京"}
```

### 키/값 존재 확인 & 목록

```ruby
puts person.key?(:name)     # → true
puts person.key?(:email)    # → false
puts person.keys.inspect    # → [:name, :age, :city]
puts person.values.inspect  # → ["カンホンギュ", 26, "東京"]
```

---

### 🔥 도전 코드 — 해시

**해시를 중첩(nested)시켜서 사용자 관리 시스템을 만들어보자.**

```ruby
# 해시 안에 해시를 담는 중첩 구조
# Rails의 DB 레코드 구조와 유사함
users = {
  user_001: { name: "カンホンギュ", age: 26, role: :admin,  active: true  },
  user_002: { name: "田中(たなか)",  age: 32, role: :editor, active: true  },
  user_003: { name: "佐藤(さとう)",  age: 28, role: :viewer, active: false },
  user_004: { name: "鈴木(すずき)",  age: 22, role: :editor, active: true  },
}

# 활성화된 유저만 필터링
active_users = users.select { |id, info| info[:active] }
puts "アクティブユーザー数(すう): #{active_users.length}名(めい)"

# 역할별로 그룹화
# group_by: 블록의 반환값을 키로 해서 그룹을 묶어줌
grouped = users.group_by { |id, info| info[:role] }
grouped.each do |role, members|
  names = members.map { |id, info| info[:name] }
  puts "#{role}: #{names.join(", ")}"
end

puts ""

# 전체 유저 평균 나이
total_age = users.sum { |id, info| info[:age] }
avg_age   = (total_age / users.length.to_f).round(1)
puts "平均年齢(へいきんねんれい): #{avg_age}歳(さい)"

# 특정 유저 정보 업데이트 — merge로 기존 정보 보존
updated = users[:user_003].merge(active: true, updated_at: Time.now.strftime("%Y-%m-%d"))
puts "更新後(こうしんご): #{updated}"
```

**포인트:**

- 해시 안에 해시 → `info[:role]` 처럼 단계적으로 접근
- `group_by` → 특정 기준으로 그룹화, Rails의 `group_by` 와 동일
- `merge` 로 원본은 유지하면서 특정 키만 덮어씌우기 — 불변(immutable) 업데이트 패턴

---

## 2. 범위 (Range)

**시작값과 끝값을 `..` 또는 `...` 으로 연결해서 범위를 표현하는 구조**.  
배열처럼 변환해서 쓸 수도 있고, 조건 검사나 `case/when` 에서 직접 쓸 수도 있다.

**開始値(かいしち)と終了値(しゅうりょうち)を`..`や`...`でつなぎ範囲(はんい)を表現(ひょうげん)する構造(こうぞう)**。

### `..` (양쪽 포함) vs `...` (끝 제외)

```ruby
# .. : 끝값 포함 (inclusive)
range_inclusive = (1..5)
puts range_inclusive.to_a.inspect   # → [1, 2, 3, 4, 5]

# ... : 끝값 제외 (exclusive)
range_exclusive = (1...5)
puts range_exclusive.to_a.inspect   # → [1, 2, 3, 4]
# 배열 인덱스 접근이나 페이지네이션에서 자주 사용
```

### 범위 메서드

```ruby
range = (1..10)
puts range.first      # → 1
puts range.last       # → 10
puts range.min        # → 1
puts range.max        # → 10
puts range.sum        # → 55
puts range.include?(5)   # → true
puts range.include?(11)  # → false
```

### 범위 순회 & 변환

```ruby
# each, map, select 모두 배열과 동일하게 사용 가능
(1..5).each { |n| puts n }

puts (1..5).map { |n| n * 2 }.inspect       # → [2, 4, 6, 8, 10]
puts (1..5).select { |n| n.odd? }.inspect   # → [1, 3, 5]
```

### 문자 범위

```ruby
# 숫자뿐 아니라 문자도 범위로 표현 가능
puts ("a".."e").to_a.inspect     # → ["a", "b", "c", "d", "e"]
puts ("a".."e").include?("c")    # → true
```

### case/when 에서의 범위

```ruby
# 범위를 when 조건으로 직접 사용 — 매우 자주 쓰임
score = 75
case score
when 80..100
  puts "優秀(ゆうしゅう)"
when 60..79
  puts "合格(ごうかく)"    # → 출력됨
else
  puts "不合格(ふごうかく)"
end
```

---

### 🔥 도전 코드 — 범위

**범위를 활용해서 달력 및 연령대 분류기를 만들어보자.**

```ruby
# 연령대 분류 — case/when + 범위
def age_group(age)
  case age
  when 0..12   then "子供(こども)"
  when 13..19  then "ティーンエイジャー"
  when 20..29  then "20代(だい)"
  when 30..39  then "30代(だい)"
  when 40..59  then "中年(ちゅうねん)"
  when 60..Float::INFINITY then "シニア"   # 60 이상 모두 포함
  else "不明(ふめい)"
  end
end

ages = [8, 15, 24, 35, 52, 70]
ages.each { |age| puts "#{age}歳(さい): #{age_group(age)}" }

puts ""

# 범위로 구구단 생성
puts "=== 구구단 ==="
(2..9).each do |i|
  row = (1..9).map { |j| "#{i}×#{j}=#{i*j}" }.join("  ")
  puts row
end

puts ""

# 범위 + step: 특정 간격으로 반복
puts "=== 짝수만 (1..20) ==="
(1..20).step(2) { |n| print "#{n+1} " }   # step(2): 2씩 건너뜀
puts ""

# 범위로 알파벳 변환 게임
puts "=== A-Z 인덱스 ==="
("A".."Z").each.with_index(1) do |char, i|
  print "#{char}=#{i} "
  puts "" if i % 5 == 0   # 5개마다 줄바꿈
end
```

**포인트:**

- `Float::INFINITY` → "무한대" — 상한선이 없는 범위 표현
- `range.step(n)` → n씩 건너뛰며 순회
- `.each.with_index(1)` → 인덱스를 1부터 시작

---

## 3. 컬렉션 (Collections)

**Ruby의 Array, Hash, Range는 모두 `Enumerable` 모듈을 포함**하고 있다.  
그래서 `each`, `map`, `select`, `reject`, `include?` 같은 메서드를  
**세 종류 모두 동일하게 사용**할 수 있다.

**RubyのArray・Hash・RangeはすべてEnumerableモジュールを含(ふく)む**。  
そのため`each`・`map`・`select`等(とう)のメソッドを**三種類(さんしゅるい)すべてに同(おな)じく使(つか)える**。

### 비교 — 세 컬렉션에서 같은 메서드

```ruby
fruits = ["apple", "banana", "kiwi"]
person = { name: "カンホンギュ", age: 26 }
range  = (1..5)

# include?: 세 가지 모두 사용 가능
puts fruits.include?("apple")   # → true
puts person.include?(:name)     # → true
puts range.include?(3)          # → true

# map: 세 가지 모두 사용 가능
puts fruits.map { |f| f.upcase }.inspect
# → ["APPLE", "BANANA", "KIWI"]

puts person.map { |k, v| "#{k}: #{v}" }.inspect
# → ["name: カンホンギュ", "age: 26"]

puts range.map { |n| n * 2 }.inspect
# → [2, 4, 6, 8, 10]

# select: 세 가지 모두 사용 가능
puts fruits.select { |f| f.length > 5 }.inspect
# → ["banana"]

puts person.select { |k, v| v.is_a?(String) }.inspect
# → {:name=>"カンホンギュ"}

puts range.select { |n| n.even? }.inspect
# → [2, 4]
```

### 타입 간 변환

```ruby
# 배열 ↔ 해시 상호 변환
array = [[:name, "カンホンギュ"], [:age, 26]]
puts array.to_h.inspect   # → {:name=>"カンホンギュ", :age=>26}

hash = { name: "カンホンギュ", age: 26 }
puts hash.to_a.inspect    # → [[:name, "カンホンギュ"], [:age, 26]]

# 범위 → 배열
puts (1..5).to_a.inspect  # → [1, 2, 3, 4, 5]
```

### 접근 방법

```ruby
puts fruits[0]       # 배열 → 인덱스(숫자)로 접근
puts person[:name]   # 해시 → 키(심볼)로 접근
# 범위는 인덱스 직접 접근 없음 → to_a로 배열로 변환 후 접근
```

---

### 🔥 도전 코드 — 컬렉션

**세 가지 컬렉션을 조합해서 학교 수업 일정표를 만들어보자.**

```ruby
# 수업 데이터를 해시 배열로 표현 (Rails의 ActiveRecord 결과와 유사)
timetable = [
  { day: :monday,    period: 1, subject: "数学(すうがく)",   teacher: "田中(たなか)"  },
  { day: :monday,    period: 2, subject: "英語(えいご)",     teacher: "佐藤(さとう)"  },
  { day: :tuesday,   period: 1, subject: "Ruby",            teacher: "カンホンギュ" },
  { day: :tuesday,   period: 2, subject: "Shell",           teacher: "カンホンギュ" },
  { day: :wednesday, period: 1, subject: "数学(すうがく)",   teacher: "田中(たなか)"  },
  { day: :thursday,  period: 1, subject: "英語(えいご)",     teacher: "佐藤(さとう)"  },
  { day: :friday,    period: 1, subject: "Ruby",            teacher: "カンホンギュ" },
]

# 요일별로 그룹화
by_day = timetable.group_by { |lesson| lesson[:day] }

by_day.each do |day, lessons|
  puts "【#{day}】"
  # 교시(period) 순으로 정렬
  lessons.sort_by { |l| l[:period] }.each do |l|
    puts "  #{l[:period]}時限(じげん): #{l[:subject]} (#{l[:teacher]})"
  end
end

puts ""

# 특정 교사의 수업만 필터링
teacher_name = "カンホンギュ"
my_classes = timetable.select { |l| l[:teacher] == teacher_name }
puts "#{teacher_name}の担当(たんとう)授業(じゅぎょう): #{my_classes.length}コマ"
my_classes.each { |l| puts "  #{l[:day]} #{l[:period]}時限(じげん): #{l[:subject]}" }

puts ""

# 과목별 담당 교사 해시 생성
subject_teacher = timetable
  .map { |l| [l[:subject], l[:teacher]] }   # [과목, 교사] 배열로
  .uniq { |pair| pair[0] }                  # 과목 중복 제거
  .to_h                                     # 해시로 변환

puts "科目(かもく)担当(たんとう)一覧(いちらん):"
subject_teacher.each { |subject, teacher| puts "  #{subject}: #{teacher}" }
```

**포인트:**

- `group_by` → 특정 키 기준으로 그룹화 — Rails의 `group_by` 와 완전히 동일
- `.map.uniq.to_h` 처럼 여러 메서드를 체이닝해서 데이터 변환
- `sort_by { |l| l[:period] }` → 해시 배열을 특정 값 기준으로 정렬

---

## 4. 정규표현식 (Regexp)

**문자열의 패턴을 정의해서 검색·추출·치환하는 도구**.  
`/패턴/` 형태로 작성하며, 이메일·전화번호·URL 검증처럼  
**특정 형식을 가진 문자열을 다룰 때** 매우 강력하다.

**文字列(もじれつ)のパターンを定義(ていぎ)して検索(けんさく)・抽出(ちゅうしゅつ)・置換(ちかん)するツール**。  
メール・電話番号(でんわばんごう)・URLの検証(けんしょう)など**特定(とくてい)の形式(けいしき)を持(も)つ文字列(もじれつ)を扱(あつか)う際(さい)に強力(きょうりょく)**。

### 기본 매칭

```ruby
# =~ : 매칭 시 일치한 시작 인덱스 반환, 불일치 시 nil
puts "hello" =~ /ell/   # → 1  ("ell"이 1번 인덱스에 있음)
puts "hello" =~ /xyz/   # → nil (없음)

# match?: 매칭 여부만 true/false로 반환 (가장 가볍고 빠름)
puts "hello".match?(/ell/)   # → true
puts "hello".match?(/xyz/)   # → false

# match: 매칭 객체 반환 (매칭 내용을 꺼낼 때 사용)
puts "hello".match(/ell/)    # → ell
```

### 자주 쓰는 패턴 문자

```ruby
# \d : 숫자 한 글자 (digit)
puts "abc123" =~ /\d/       # → 3  (첫 번째 숫자의 위치)

# \w : 영문자·숫자·밑줄 (word character)
puts "hello world".match?(/\w/)   # → true

# \s : 공백 문자 (space)
puts "hello world".match?(/\s/)   # → true

# .  : 줄바꿈 제외 임의의 한 글자
puts "hallo".match?(/h.llo/)   # → true  (a가 .에 매칭)
puts "hqllo".match?(/h.llo/)   # → true  (q가 .에 매칭)
```

### 수량자 — 반복 횟수 지정

```ruby
# + : 1회 이상
puts "aaa".match?(/a+/)   # → true
puts "aaa".match?(/b+/)   # → false

# * : 0회 이상 (없어도 true)
puts "bbb".match?(/b*/)   # → true
puts "".match?(/b*/)      # → true  (0회이므로 빈 문자열도 매칭!)
```

### 앵커 — 위치 지정

```ruby
# ^ : 줄의 시작
puts "hello".match?(/^h/)   # → true  (h로 시작)
puts "hello".match?(/^b/)   # → false (b로 시작하지 않음)

# $ : 줄의 끝
puts "hello".match?(/o$/)   # → true  (o로 끝남)
puts "hello".match?(/s$/)   # → false (s로 끝나지 않음)

# \A : 문자열의 절대적 시작 (^ 보다 엄격)
# \z : 문자열의 절대적 끝   ($ 보다 엄격)
# → 보안상 이유로 검증에는 \A, \z를 권장
```

### gsub / scan — 치환과 추출

```ruby
# gsub: 패턴에 맞는 부분 모두 치환
puts "hello world".gsub(/\s/, "_")
# → hello_world  (공백을 _로 치환)

# scan: 패턴에 맞는 부분을 모두 배열로 추출
puts "abs123qwer4567".scan(/\d+/).inspect
# → ["123", "4567"]  (\d+: 연속된 숫자)
```

### 실전 — 이메일 / 전화번호 검증

```ruby
# 이메일 검증
email = "test@example.com"
puts email.match?(/\A[\w+\-.]+@[a-z\d\-.]+\.[a-z]+\z/i)
# → true
# \A ~ \z : 문자열 전체 검증
# [\w+\-.]+ : 영문자·숫자·+·-·.으로 이루어진 로컬 파트
# @ : 골뱅이
# [a-z\d\-.]+\.[a-z]+ : 도메인 부분
# i : 대소문자 무시 옵션

# 전화번호 검증 (일본식 090-XXXX-XXXX)
phone = "090-1234-5678"
puts phone.match?(/\A\d{3}-\d{4}-\d{4}\z/)
# → true
# \d{3} : 숫자 정확히 3개
# \d{4} : 숫자 정확히 4개
```

---

### 🔥 도전 코드 — 정규표현식

**정규표현식으로 로그 파일 파서(parser)를 만들어보자.**

```ruby
# 웹 서버 로그 형식 예시
# "2026-03-13 14:32:01 GET /posts 200 152ms"
logs = [
  "2026-03-13 14:32:01 GET /posts 200 152ms",
  "2026-03-13 14:32:05 POST /posts 201 88ms",
  "2026-03-13 14:33:10 GET /posts/99 404 12ms",
  "2026-03-13 14:35:22 DELETE /posts/1 403 5ms",
  "2026-03-13 14:40:01 GET /users 500 320ms",
  "2026-03-13 14:41:55 GET /posts 200 140ms",
]

# 로그 한 줄을 해시로 파싱
def parse_log(line)
  # 각 부분을 캡처 그룹 () 으로 추출
  pattern = /\A(\d{4}-\d{2}-\d{2}) (\d{2}:\d{2}:\d{2}) (\w+) (\S+) (\d{3}) (\d+)ms\z/

  match = line.match(pattern)
  return nil unless match   # 패턴 불일치 시 nil

  {
    date:    match[1],        # 캡처 그룹 1: 날짜
    time:    match[2],        # 캡처 그룹 2: 시각
    method:  match[3],        # 캡처 그룹 3: HTTP 메서드
    path:    match[4],        # 캡처 그룹 4: 경로
    status:  match[5].to_i,   # 캡처 그룹 5: 상태 코드
    elapsed: match[6].to_i,   # 캡처 그룹 6: 응답시간(ms)
  }
end

# 전체 로그 파싱
parsed = logs.map { |line| parse_log(line) }.compact   # compact: nil 제거

# 상태 코드별 분류
grouped = parsed.group_by do |log|
  case log[:status]
  when 200..299 then :success
  when 400..499 then :client_error
  when 500..599 then :server_error
  end
end

puts "=== ログ分析(ぶんせき) ==="
puts "成功(せいこう) (2xx): #{grouped[:success]&.length || 0}件(けん)"
puts "クライアントエラー (4xx): #{grouped[:client_error]&.length || 0}件(けん)"
puts "サーバーエラー (5xx): #{grouped[:server_error]&.length || 0}件(けん)"

# 응답 시간이 100ms 이상인 느린 요청
slow = parsed.select { |log| log[:elapsed] >= 100 }
puts ""
puts "=== 遅(おそ)いリクエスト (100ms以上(いじょう)) ==="
slow.each { |log| puts "  #{log[:method]} #{log[:path]} #{log[:elapsed]}ms [#{log[:status]}]" }

# 전체 평균 응답 시간
avg_ms = parsed.sum { |log| log[:elapsed] } / parsed.length.to_f
puts ""
puts "平均(へいきん)応答時間(おうとうじかん): #{avg_ms.round(1)}ms"
```

**포인트:**

- `match(패턴)` → `match[1]`, `match[2]` 로 캡처 그룹 순서대로 추출
- `compact` → 배열에서 nil을 제거
- `&.` (safe navigation operator) → nil일 수도 있는 객체에 안전하게 메서드 호출
- 로그 파싱은 실무에서 정규표현식을 가장 많이 쓰는 케이스 중 하나

---

## 🔑 한눈에 보는 키워드

| 키워드                      | 설명                                     |
| :-------------------------- | :--------------------------------------- |
| `{ key: value }`            | 심볼 키 해시 (권장)                      |
| `{ "key" => value }`        | 문자열 키 해시                           |
| `hash.merge(other)`         | 해시 합치기 (새 해시 반환)               |
| `hash.merge!(other)`        | 해시 합치기 (원본 변경)                  |
| `hash.key?(:키)`            | 키 존재 여부                             |
| `hash.keys` / `hash.values` | 키/값 배열                               |
| `1..5`                      | 끝값 포함 범위                           |
| `1...5`                     | 끝값 제외 범위                           |
| `range.step(n)`             | n씩 건너뛰며 순회                        |
| `Float::INFINITY`           | 무한대                                   |
| `.group_by`                 | 특정 기준으로 그룹화                     |
| `.sort_by`                  | 특정 값 기준 정렬                        |
| `.compact`                  | 배열에서 nil 제거                        |
| `.uniq`                     | 중복 제거                                |
| `=~`                        | 정규표현식 매칭 (인덱스 반환)            |
| `.match?`                   | 매칭 여부 (true/false)                   |
| `.match`                    | 매칭 객체 반환                           |
| `\d` / `\w` / `\s`          | 숫자 / 영문자·숫자·\_ / 공백             |
| `.`                         | 임의의 한 글자                           |
| `+` / `*`                   | 1회 이상 / 0회 이상                      |
| `\A` / `\z`                 | 문자열 시작 / 끝 (검증에 권장)           |
| `{n}`                       | 정확히 n회                               |
| `.scan(패턴)`               | 패턴에 맞는 부분 전부 배열로 추출        |
| `&.`                        | Safe Navigation Operator — nil 안전 호출 |

---

[← 목차로 돌아가기 / 目次(もくじ)に戻(もど)る](../README.md)
