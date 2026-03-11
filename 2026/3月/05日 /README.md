# 📅 Day 13 — 셸 스크립트 기초 (조건문 & 반복문)

**날짜 / 日付(ひづけ):** 2026.03.05  
**학습 시간 / 学習時間(がくしゅうじかん):** 자율 / 自律(じりつ)  
**한 줄 목표:** 셸 스크립트의 조건문과 반복문을 익혀서 실용적인 자동화 스크립트를 작성한다.  
**一言目標(ひとこともくひょう):** シェルスクリプトの条件文(じょうけんぶん)とループを習得(しゅうとく)して、実用的(じつようてき)な自動化(じどうか)スクリプトを作成(さくせい)する。

---

## 목차 / 目次(もくじ)

1. [조건문 (if/elif/else)](#1-조건문-ifelifelse)
2. [비교 연산자](#2-비교-연산자)
3. [반복문 (for)](#3-반복문-for)
4. [break & continue](#4-break--continue)
5. [실전 예제 — mixed.sh](#5-실전-예제--mixedsh)

---

## 1. 조건문 (if/elif/else)

### 기본 구조

```bash
if [ 조건 ]; then
  # 조건이 참일 때
elif [ 다른조건 ]; then
  # 다른 조건이 참일 때
else
  # 모두 아닐 때
fi
```

### 숫자 비교

```bash
score=70

if [ $score -ge 90 ]; then
  echo "優秀です"
elif [ $score -ge 70 ]; then
  echo "合格です"       # → 출력됨
else
  echo "不合格です"
fi
```

### 문자열 비교

```bash
name="カンホンギュ"

if [ $name = "カンホンギュ" ]; then
  echo "名前が一致しました"   # → 출력됨
else
  echo "名前が一致しませんでした"
fi
```

### AND / OR 조건

```bash
age=20

# AND — 두 조건 모두 참일 때
if [ $age -ge 18 ] && [ $age -le 65 ]; then
  echo "働くことができます"   # → 출력됨
fi

day="Saturday"

# OR — 둘 중 하나라도 참일 때
if [ $day = "Saturday" ] || [ $day = "Sunday" ]; then
  echo "今日は休みです"   # → 출력됨
fi
```

### 파일 존재 확인

```bash
# -f: 파일이 존재하는지 확인
if [ -f "echo.sh" ]; then
  echo "echo.shが存在します"
else
  echo "echo.shが存在しません"
fi

# -z: 문자열이 비어있는지 확인
input=""
if [ -z $input ]; then
  echo "入力が空です"   # → 출력됨
fi
```

---

## 2. 비교 연산자

### 숫자 비교

| 연산자 | 의미                           | 예시           |
| :----: | :----------------------------- | :------------- |
| `-eq`  | 같다 (equal)                   | `[ $a -eq 5 ]` |
| `-ne`  | 다르다 (not equal)             | `[ $a -ne 5 ]` |
| `-gt`  | 크다 (greater than)            | `[ $a -gt 5 ]` |
| `-ge`  | 크거나 같다 (greater or equal) | `[ $a -ge 5 ]` |
| `-lt`  | 작다 (less than)               | `[ $a -lt 5 ]` |
| `-le`  | 작거나 같다 (less or equal)    | `[ $a -le 5 ]` |

### 문자열 비교

| 연산자 | 의미                   |
| :----: | :--------------------- |
|  `=`   | 같다                   |
|  `!=`  | 다르다                 |
|  `-z`  | 비어있다 (zero length) |
|  `-n`  | 비어있지 않다          |

### 파일 확인

| 연산자 | 의미                     |
| :----: | :----------------------- |
|  `-f`  | 파일이 존재한다          |
|  `-d`  | 디렉토리가 존재한다      |
|  `-e`  | 파일/디렉토리가 존재한다 |

---

## 3. 반복문 (for)

### 기본 목록 순회

```bash
for fruit in apple banana orange; do
  echo "果物：$fruit"
done
# → 果物：apple
#   果物：banana
#   果物：orange
```

### seq로 숫자 범위 반복

```bash
# seq 시작 끝
for i in $(seq 1 5); do
  echo "数字： $i"
done
# → 1, 2, 3, 4, 5

# seq 시작 스텝 끝
for i in $(seq 1 2 10); do
  echo "ステップ: $i"
done
# → 1, 3, 5, 7, 9
```

### 배열 순회

```bash
names=("カンホンギュ" "田中" "佐藤")

for name in "${names[@]}"; do
  echo "名前： $name"
done
# "${배열명[@]}" — 배열의 모든 요소
```

### 합계 계산

```bash
total=0

for i in $(seq 1 10); do
  total=$((total + i))   # 산술 연산은 $(( )) 안에서!
done

echo "合計： $total"   # → 合計： 55
```

### 짝수만 출력

```bash
for i in $(seq 1 10); do
  if [ $((i % 2)) -eq 0 ]; then   # % = 나머지 연산
    echo "偶数： $i"
  fi
done
# → 2, 4, 6, 8, 10
```

### 파일 존재 여부 확인

```bash
for file in echo.sh loops.sh conditional.sh; do
  if [ -f $file ]; then
    echo "${file}が存在します"
  else
    echo "${file}が存在しません"
  fi
done
```

---

## 4. break & continue

```bash
# break — 조건 충족 시 루프 완전 종료
total=0
for i in $(seq 1 10); do
  total=$((total + i))
  if [ $total -gt 30 ]; then
    echo "合計が30を超えました: $total"
    break   # 루프 탈출!
  fi
  echo "現在の合計: $total"
done

# continue — 현재 반복만 건너뛰고 다음 반복 계속
for i in $(seq 1 10); do
  if [ $((i % 2)) -eq 0 ]; then
    continue   # 짝수면 건너뜀
  fi
  echo "奇数: $i"   # 홀수만 출력
done
```

|   키워드   | 동작                      |
| :--------: | :------------------------ |
|  `break`   | 루프 전체 종료            |
| `continue` | 현재 반복만 건너뛰고 계속 |

---

## 5. 실전 예제 — mixed.sh

### FizzBuzz

```bash
for i in $(seq 1 20); do
  if [ $((i % 15)) -eq 0 ]; then
    echo "$i: 3と5の倍数"
  elif [ $((i % 3)) -eq 0 ]; then
    echo "$i: 3の倍数"
  elif [ $((i % 5)) -eq 0 ]; then
    echo "$i: 5の倍数"
  else
    echo "$i: 該当なし"
  fi
done
```

### 점수 배열로 합격/불합격 판정

```bash
scores=(45 72 88 55 91 63)

for score in "${scores[@]}"; do
  if [ $score -ge 70 ]; then
    echo "点数: $score → 合格"
  else
    echo "点数: $score → 不合格"
  fi
done
```

### 기온 판정

```bash
temperatures=(5 15 25 30 38 12)

for temp in "${temperatures[@]}"; do
  if [ $temp -ge 30 ]; then
    echo "気温: ${temp}度 → 暑いです"
  elif [ $temp -ge 20 ]; then
    echo "気温: ${temp}度 → 適温です"
  else
    echo "気温: ${temp}度 → 寒いです"
  fi
done
```

### 이름 길이 판정

```bash
names=("カンホンギュ" "田中(たなか)" "アレキサンダー" "佐藤(さとう)")

for name in "${names[@]}"; do
  length=${#name}   # ${#변수명} → 문자열 길이!
  if [ $length -ge 7 ]; then
    echo "${name}: 長い名前です"
  else
    echo "${name}: 短い名前です"
  fi
done
```

---

## 🔑 한눈에 보는 키워드

| 키워드                 | 설명                           |
| :--------------------- | :----------------------------- |
| `if [ 조건 ]; then`    | 조건문 시작                    |
| `elif [ 조건 ]; then`  | 추가 조건                      |
| `else`                 | 모든 조건이 거짓일 때          |
| `fi`                   | 조건문 종료 (if 거꾸로!)       |
| `-ge` / `-le` / `-eq`  | 크거나같다 / 작거나같다 / 같다 |
| `&&` / `\|\|`          | AND / OR                       |
| `-f 파일명`            | 파일 존재 확인                 |
| `-z 변수`              | 문자열이 비어있는지 확인       |
| `for 변수 in 목록; do` | 반복문 시작                    |
| `done`                 | 반복문 종료                    |
| `$(seq 시작 끝)`       | 숫자 범위 생성                 |
| `$(seq 시작 스텝 끝)`  | 스텝 지정 숫자 범위            |
| `배열[@]`              | 배열 전체 요소                 |
| `$((계산식))`          | 산술 연산                      |
| `$((i % 2))`           | 나머지 연산                    |
| `${#변수명}`           | 문자열 길이                    |
| `break`                | 루프 완전 종료                 |
| `continue`             | 현재 반복만 건너뜀             |

---

[← 목차로 돌아가기 / 目次(もくじ)に戻(もど)る](../README.md)
