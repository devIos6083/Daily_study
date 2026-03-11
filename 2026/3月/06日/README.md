# 📅 Day 14 — 셸 스크립트 심화 (배열 & 인수 & 환경변수)

**날짜 / 日付(ひづけ):** 2026.03.06  
**학습 시간 / 学習時間(がくしゅうじかん):** 자율 / 自律(じりつ)  
**한 줄 목표:** 셸 스크립트의 배열 조작, 인수 처리, 환경변수를 익혀서 실용적인 스크립트를 작성한다.  
**一言目標(ひとこともくひょう):** シェルスクリプトの配列操作(はいれつそうさ)・引数処理(ひきすうしょり)・環境変数(かんきょうへんすう)を習得(しゅうとく)して実用的(じつようてき)なスクリプトを作成(さくせい)する。

---

## 목차 / 目次(もくじ)

1. [배열이란?](#1-배열이란)
2. [배열 조작](#2-배열-조작)
3. [배열 활용 예제](#3-배열-활용-예제)
4. [인수(引数)란?](#4-인수란)
5. [환경변수란?](#5-환경변수란)

---

## 1. 배열이란?

**여러 개의 값을 하나의 변수에 순서대로 담는 구조**.  
일반 변수가 값 하나만 담는다면, 배열은 값 여러 개를 번호(인덱스)로 구분해서 담는다.  
셸에서 인덱스는 **0부터 시작**한다.

**複数(ふくすう)の値(あたい)を一(ひと)つの変数(へんすう)に順番(じゅんばん)に入(い)れる構造(こうぞう)**。  
シェルではインデックスは**0から始(はじ)まる**。

```
일반 변수:  name = "カンホンギュ"     → 값 1개

배열:       fruits[0] = "apple"
            fruits[1] = "banana"    → 값 여러 개를 번호로 관리
            fruits[2] = "orange"
            fruits[3] = "grape"
```

```bash
# 배열 선언 — 소괄호 () 안에 공백으로 구분
fruits=("apple" "banana" "orange" "grape")

# 전체 요소 출력 — [@]는 "전체"를 의미
echo "全体: ${fruits[@]}"
# → apple banana orange grape

# 요소 수 — ${#배열[@]} : # 기호가 "길이/개수"를 의미
echo "要素数: ${#fruits[@]}"
# → 4

# 인덱스로 접근 — 0부터 시작!
echo "最初の要素: ${fruits[0]}"
# → apple

# 마지막 요소 — 길이-1이 마지막 인덱스
length=${#fruits[@]}
echo "最後の要素: ${fruits[$((length-1))]}"
# → grape
```

### 혼합 타입 배열

```bash
# 셸 배열은 문자열, 숫자, 소수 모두 담을 수 있다
# Ruby나 Python처럼 타입을 엄격하게 구분하지 않음
mixed=("カンホンギュ" 25 "Seoul" 3.14)

for item in "${mixed[@]}"; do
  echo "要素: $item"
done
```

### 인덱스와 함께 순회

```bash
languages=("Ruby" "Shell" "Python" "JavaScript")

# ${!배열[@]} → 값이 아니라 인덱스 번호 목록을 반환
# ! 기호가 "인덱스"를 의미 — ${배열[@]}의 반대
for i in "${!languages[@]}"; do
  echo "インデックス$i: ${languages[$i]}"
done
# → インデックス0: Ruby
#   インデックス1: Shell
#   インデックス2: Python
#   インデックス3: JavaScript
```

---

## 2. 배열 조작

배열은 선언 후에도 **자유롭게 추가/변경/삭제**할 수 있다.

```bash
fruits=("apple" "banana" "orange" "grape")

# ① 요소 변경 — 인덱스를 지정해서 덮어쓰기
fruits[1]="strawberry"
echo "変更後: ${fruits[@]}"
# → apple strawberry orange grape

# ② 요소 추가 — += 로 배열 뒤에 덧붙이기
fruits+=("melon")
echo "追加後: ${fruits[@]}"
# → apple strawberry orange grape melon

# ③ 요소 삭제 — unset으로 특정 인덱스 제거
#    주의: 삭제 후에도 나머지 인덱스는 그대로 유지됨
unset fruits[0]
echo "削除後: ${fruits[@]}"
# → strawberry orange grape melon
```

### 두 배열 합치기

```bash
array1=("a" "b" "c")
array2=("d" "e" "f")

# "${배열[@]}"를 나란히 쓰면 새 배열로 합쳐짐
combined=("${array1[@]}" "${array2[@]}")
echo "結合後: ${combined[@]}"
# → a b c d e f
```

---

## 3. 배열 활용 예제

### 합계 & 평균 계산

```bash
scores=(85 92 78 95 63)
total=0

for score in "${scores[@]}"; do
  total=$((score + total))
done

echo "合計点： $total"
# → 413

echo "平均点: $(($total / ${#scores[@]}))"
# ${#scores[@]} = 5 (요소 수)
# 셸은 정수 나눗셈 → 소수점 버려짐 → 82
```

### 최댓값 & 최솟값 찾기

```bash
numbers=(3 17 8 42 5 29 11)

# 핵심 아이디어:
# 첫 번째 값을 기준(maximum)으로 잡고
# 순회하면서 더 큰 값이 나오면 기준을 그 값으로 교체
maximum=${numbers[0]}
for number in "${numbers[@]}"; do
  if [ $number -gt $maximum ]; then
    maximum=$number
  fi
done
echo "最大値: $maximum"   # → 42

minimum=${numbers[0]}
for number in "${numbers[@]}"; do
  if [ $number -lt $minimum ]; then
    minimum=$number
  fi
done
echo "最小値: $minimum"   # → 3
```

### 짝수만 필터링 — 새 배열에 담기

```bash
numbers2=(1 2 3 4 5 6 7 8 9 10)

# 조건을 통과한 값만 새 배열(even_numbers)에 추가
# 이런 패턴을 "필터링"이라고 부름
for number in "${numbers2[@]}"; do
  if [ $((number % 2)) -eq 0 ]; then
    even_numbers+=($number)
  fi
done

echo "偶数: ${even_numbers[@]}"   # → 2 4 6 8 10
```

---

## 4. 인수(引数)란?

**스크립트를 실행할 때 외부에서 값을 전달하는 방법**.  
함수의 파라미터처럼, 실행 시점에 동적으로 값을 주입할 수 있어서  
**같은 스크립트를 다양한 입력값으로 재사용**할 수 있다.

인수가 없으면 같은 작업을 할 때마다 스크립트를 수정해야 하지만,  
인수를 쓰면 실행할 때마다 다른 값을 전달할 수 있다.

**スクリプト実行時(じっこうじ)に外部(がいぶ)から値(あたい)を渡(わた)す方法(ほうほう)**。  
同(おな)じスクリプトを様々(さまざま)な入力値(にゅうりょくち)で再利用(さいりよう)できる。

```bash
# 실행 방법 — 스크립트명 뒤에 공백으로 구분해서 전달
bash script.sh カンホンギュ 新潟 25
#              ↑ $1          ↑ $2  ↑ $3
```

### 기본 인수 변수

```bash
echo "スクリプト名: $0"   # 실행한 스크립트 파일 이름
echo "引数の数: $#"       # 전달받은 인수 개수
echo "全ての引数: $@"     # 모든 인수를 공백 구분으로 나열

# bash script.sh カンホンギュ 新潟 25 로 실행 시
# $0 = script.sh
# $1 = カンホンギュ
# $2 = 新潟
# $3 = 25
# $# = 3
# $@ = カンホンギュ 新潟 25
```

### 기본값 설정 (`:-`)

```bash
# ${변수:-기본값} 패턴
# 인수가 전달되지 않아도 스크립트가 안전하게 동작하도록 보호
# 인수가 있으면 인수를, 없으면 기본값을 사용
name=${1:-"ゲスト"}    # $1이 없으면 "ゲスト" 사용
city=${2:-"不明"}      # $2가 없으면 "不明" 사용
age=${3:-"不明"}       # $3가 없으면 "不明" 사용

echo "名前: $name"
echo "都市: $city"
echo "年齢: $age"
```

### 인수 개수 검증

```bash
# 스크립트 실행 전에 인수가 올바르게 전달됐는지 확인
# 실무에서 입력값 검증(validation)의 기본 패턴
if [ $# -eq 0 ]; then
  echo "引数がありません"
elif [ $# -lt 3 ]; then
  echo "引数が足りません。3つ入力してください"
else
  echo "全ての引数が揃っています"
fi
```

### 인수로 합계 계산

```bash
# "$@" — 모든 인수를 개별적으로 순회
# 2>/dev/null — 숫자가 아닌 값이 들어왔을 때 나오는 에러 메시지를
#               /dev/null(OS의 휴지통)에 버려서 화면에 안 보이게 숨김
total=0
for number in "$@"; do
  if [ $number -gt 0 ] 2>/dev/null; then
    total=$((number + total))
  fi
done
echo "合計: $total"
```

### 가장 긴 인수 찾기

```bash
# 최댓값 찾기와 같은 패턴
# 다만 숫자 크기 대신 문자열 길이 ${#변수} 로 비교
longest=""
for argument in "$@"; do
  if [ ${#argument} -gt ${#longest} ]; then
    longest=$argument
  fi
done
echo "もっと長い引数: $longest"
```

### 역순 출력

```bash
# "$@"를 배열에 담은 뒤 끝 인덱스부터 역순으로 순회
# seq 시작 -1 0 → 스텝이 음수이면 감소하는 수열 생성
arguments=("$@")
length=${#arguments[@]}

for i in $(seq $((length - 1)) -1 0); do
  echo " - ${arguments[$i]}"
done
```

### 특정 값 검색 & 등장 횟수

```bash
target=${1:-""}   # 첫 번째 인수 = 검색할 대상
found=0

for argument in "$@"; do
  if [ "$argument" = "$target" ]; then
    found=$((found + 1))
  fi
done

echo "${target}は${found}回見つかりました"
```

---

## 5. 환경변수란?

**OS 전체에서 공유되는 변수**.  
일반 변수는 선언된 스크립트 안에서만 유효하지만,  
환경변수(`export`)는 **자식 프로세스나 다른 프로그램에서도 참조**할 수 있다.

실무에서는 API 키, DB 접속 정보, 실행 환경(dev/prod) 같은 **민감하거나 환경마다 다른 설정**을 환경변수로 관리한다.  
코드에 직접 값을 쓰지 않아서 **보안**과 **유연성** 두 가지를 모두 챙길 수 있다.

**OSの全体(ぜんたい)で共有(きょうゆう)される変数(へんすう)**。  
`export`した変数(へんすう)は**子(こ)プロセスや他(ほか)のプログラムからも参照(さんしょう)できる**。

```
로컬 변수    → 현재 스크립트 안에서만 유효
환경변수     → 자식 프로세스, 다른 프로그램에서도 유효

Rails에서 ENV["DATABASE_URL"]로 읽는 값이 바로 환경변수!
```

### OS 기본 제공 환경변수

```bash
# OS가 처음부터 설정해두는 환경변수들
echo "ユーザ名: $USER"            # 현재 로그인한 사용자 이름
echo "ホームディレクトリ: $HOME"   # 홈 디렉토리 경로 (/Users/hongkyu 등)
echo "現在のディレクトリ: $PWD"    # 현재 작업 디렉토리 (pwd 명령어와 동일)
echo "シェルの種類: $SHELL"       # 현재 사용 중인 셸 (/bin/zsh 등)
```

### 로컬 변수 vs export 변수

```bash
# 로컬 변수 — 현재 스크립트 내부에서만 사용 가능
name="カンホンギュ"
city="新潟"

# export — 환경변수로 등록
# 자식 프로세스(이 스크립트에서 실행하는 다른 프로그램)에서도 참조 가능
export app_name="bootcamp"
export app_version="1.0.0"
export app_env="development"

# 값 변경 — export한 변수도 재할당 가능
export app_env="production"

# 변수 삭제 — unset으로 완전히 제거
unset app_version
if [ -z $app_version ]; then
  echo "削除されました"   # -z: 비어있으면(길이가 0이면) true
fi
```

### 기본값으로 환경변수 안전하게 사용

```bash
# 환경변수가 설정되어 있으면 그 값을, 없으면 기본값을 사용
# 개발 환경에서 .env 파일 없이도 스크립트가 동작하게 보호
database_url=${DATABASE_URL:-"localhost:5432"}
api_key=${API_KEY:-"default-key"}

echo "データベースURL: $database_url"
echo "APIキー: $api_key"
```

### 실행 환경에 따른 분기

```bash
# Rails에서도 동일한 개념!
# Rails.env == "production" 과 같은 패턴
if [ "$app_env" = "production" ]; then
  echo "本番環境で実行中です"
elif [ "$app_env" = "development" ]; then
  echo "開発環境で実行中です"
else
  echo "不明な環境です"
fi
```

### PATH 변수

```bash
# PATH — 셸이 명령어를 찾아보는 디렉토리 목록
# git, ruby, node 같은 명령어를 입력했을 때 셸이 PATH를 순서대로 탐색해서 찾음
echo "現在のPATH: $PATH"
# → /usr/local/bin:/usr/bin:/bin 처럼 : 로 구분된 경로 목록

# 새 경로 추가 — 기존 PATH 뒤에 덧붙이기
# rbenv, nodenv 설치 후 PATH에 추가하는 것과 같은 원리
export PATH="$PATH:/usr/local/myapp/bin"

# 환경변수 전체 목록 (head -10으로 앞 10개만 표시)
env | head -10
```

### 앱 설정을 환경변수로 관리 (실무 패턴)

```bash
# 숫자나 설정값을 코드에 직접 쓰지 않고 환경변수로 분리
# → 값을 바꿀 때 코드 수정 없이 환경변수만 바꾸면 됨
export MAX_CONNECTION=100
export TIMEOUT=30
export DEBUG_MODE="false"

echo "最大接続数: $MAX_CONNECTION"
echo "タイムアウト: ${TIMEOUT}秒"

if [ $DEBUG_MODE = "true" ]; then
  echo "デバッグモード: ON"
else
  echo "デバッグモード: OFF"
fi
```

---

## 🔑 한눈에 보는 키워드

| 키워드               | 설명                                      |
| :------------------- | :---------------------------------------- |
| `배열=("a" "b" "c")` | 배열 선언 — 소괄호, 공백으로 구분         |
| `${배열[@]}`         | 배열 전체 요소 (`@` = 전체)               |
| `${#배열[@]}`        | 배열 요소 수 (`#` = 길이/개수)            |
| `${!배열[@]}`        | 배열 인덱스 목록 (`!` = 인덱스)           |
| `배열[i]="값"`       | 특정 인덱스 값 변경                       |
| `배열+=("값")`       | 요소 추가                                 |
| `unset 배열[i]`      | 요소 삭제                                 |
| `$0`                 | 스크립트 파일명                           |
| `$1`, `$2`, ...      | 첫 번째, 두 번째 인수                     |
| `$#`                 | 인수 개수                                 |
| `$@`                 | 모든 인수                                 |
| `${변수:-기본값}`    | 값이 없으면 기본값 사용                   |
| `export 변수=값`     | 환경변수 등록 (자식 프로세스에도 전달)    |
| `unset 변수`         | 변수 삭제                                 |
| `env`                | 환경변수 전체 목록                        |
| `env \| head -10`    | 환경변수 앞 10개만                        |
| `2>/dev/null`        | 에러 메시지를 숨기기 (OS 휴지통에 버리기) |
| `${#변수}`           | 문자열 길이                               |
| `$PATH`              | 명령어 탐색 경로 목록                     |

---

[← 목차로 돌아가기 / 目次(もくじ)に戻(もど)る](../README.md)
