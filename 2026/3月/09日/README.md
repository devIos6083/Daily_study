# 📅 Day 17 — 셸 스크립트 심화 (출력 분기 & 경로 판별 & 스크립트 호출)

**날짜 / 日付(ひづけ):** 2026.03.09  
**학습 시간 / 学習時間(がくしゅうじかん):** 자율 / 自律(じりつ)  
**한 줄 목표:** stdout/stderr 분기 처리, 경로 판별, 스크립트 간 호출 방식의 차이를 익힌다.  
**一言目標(ひとこともくひょう):** stdout/stderr分岐(ぶんき)処理(しょり)・パス判別(はんべつ)・スクリプト間(かん)の呼(よ)び出(だ)し方式(ほうしき)の違(ちが)いを身(み)につける。

---

## 목차 / 目次(もくじ)

1. [stdout/stderr 분기 처리](#1-stdoutstderr-분기-처리)
2. [경로 판별 — path-check.sh](#2-경로-판별--path-checksh)
3. [스크립트 호출 방식 4가지](#3-스크립트-호출-방식-4가지)
4. [변수 공유 범위 정리](#4-변수-공유-범위-정리)

---

## 1. stdout/stderr 분기 처리

인수(argument)에 따라 **정상 처리와 에러 처리를 분리**해서 각각 다른 파일에 저장하는 패턴.  
실무에서 로그 파일을 success/error로 나눠 관리할 때 그대로 쓰이는 구조다.

引数(ひきすう)によって**正常処理(せいじょうしょり)とエラー処理(しょり)を分離(ぶんり)**し、別(べつ)のファイルに保存(ほぞん)するパターン。

### 입력값 검증 — `exit 1`

```bash
# 인수가 없으면 사용법 안내 후 종료
# exit 1: 비정상 종료 (실패를 나타내는 종료 코드)
if [ $# -eq 0 ]; then
  echo "使用方法: bash mixed-stdout-stderr.sh [ok|ng]"
  exit 1
fi

# 허용된 값이 아니면 에러 메시지 출력 후 종료
# >&2 : 에러 메시지를 stdout이 아닌 stderr로 출력
if [ $1 != "ok" ] && [ $1 != "ng" ]; then
  echo "エラー: 引数が'ok'か'ng'を入力してください" >&2
  exit 1
fi
```

> **`>&2` 란?**  
> 현재 출력(stdout)을 stderr(2번)로 방향을 바꾸는 것.  
> 에러 메시지는 stdout이 아니라 stderr로 보내는 것이 올바른 설계다.  
> → `2>` 로 에러 로그를 분리 저장할 때 이 메시지도 함께 잡힌다.

### ok / ng 분기 처리

```bash
if [ $1 = "ok" ]; then
  # 정상 처리 — stdout을 ok.txt에 저장
  echo "処理が成功しました" > ok.txt
  echo "ユーザー: $USER" >> ok.txt
  echo "日時: $(date)" >> ok.txt
  echo "ディレクトリ: $PWD" >> ok.txt
  echo "ok.txtに保存しました"
  cat ok.txt

elif [ $1 = "ng" ]; then
  # 에러 처리 — stderr를 ng.txt에 저장
  # 주의: echo 자체는 stdout이라서 2> 로는 잡히지 않음
  # → 2> 대신 >> 로 직접 파일에 쓰는 방식이 더 확실함
  echo "エラー: 失敗しました" 2> ng.txt
  echo "エラーコード: 1" >> ng.txt
  echo "ユーザー: $USER" >> ng.txt
  echo "日時: $(date)" >> ng.txt
  echo "ng.txtに保存しました"
  cat ng.txt
fi
```

### `$?` — 직전 명령어 종료 코드 확인

```bash
# echo 명령어는 항상 성공하므로 $? = 0
echo "終了コード: $?"

# 파일 존재 여부에 따른 분기
if [ -f "ok.txt" ] && [ -f "ng.txt" ]; then
  echo "ok.txtとng.txt両方が存在します"
elif [ -f "ok.txt" ]; then
  echo "ok.txtのみ存在します"
elif [ -f "ng.txt" ]; then
  echo "ng.txtのみ存在します"
else
  echo "どちらのファイルも存在しません"
fi
```

### 실행 결과를 result.log에 저장

```bash
echo "=== 実行結果 ===" > result.log
echo "引数: $1" >> result.log
echo "日時: $(date)" >> result.log

if [ $1 = 'ok' ]; then
  echo "結果: 成功" >> result.log
else
  echo "結果: 失敗" >> result.log
fi

cat result.log
```

---

## 2. 경로 판별 — path-check.sh

파일/디렉토리 경로의 **절대/상대 여부와 존재 여부를 자동으로 판별**하는 스크립트.  
`[[ ]]` (이중 대괄호)는 `[ ]` 보다 강력한 조건 검사 문법으로, 패턴 매칭(`*`, `~`)을 지원한다.

ファイル・ディレクトリのパスの**絶対(ぜったい)/相対(そうたい)の判別(はんべつ)と存在確認(そんざいかくにん)**を自動化(じどうか)するスクリプト。

### `[[ ]]` — 패턴 매칭 조건문

```bash
# [[ ]] 이중 대괄호: 패턴 매칭(==, *) 사용 가능
# [ ]  단일 대괄호: 패턴 매칭 불가

path=$1

# /* 패턴: / 로 시작하면 절대 경로
if [[ $path == /* ]]; then
  echo "絶対パス: $path"
else
  echo "相対パス: $path"
fi

# ~* 패턴: ~ 로 시작하면 홈 디렉토리 기준 경로
if [[ $path == ~* ]]; then
  echo "ホームディレクトリからの相対パスです"
fi
```

### 파일/디렉토리 존재 확인

```bash
# -e: 파일 또는 디렉토리 중 하나라도 존재하면 true
# -f: 파일만 true
# -d: 디렉토리만 true
if [ -e $path ]; then
  if [ -f $path ]; then
    echo "$path: ファイルが存在します"
  elif [ -d $path ]; then
    echo "$path: ディレクトリが存在します"
  fi
else
  echo "$path: 存在しません"
fi
```

### 여러 경로를 배열로 받아서 처리

```bash
# $@ 로 모든 인수를 받아 배열에 담기
paths=("$@")

for path in "${paths[@]}"; do
  if [[ $path == /* ]]; then
    echo "$path -> 絶対パス"
  else
    echo "$path -> 相対パス"
  fi
done
```

### `realpath` — 상대 경로를 절대 경로로 변환

```bash
for path in "${paths[@]}"; do
  if [[ $path = /* ]]; then
    echo "$path -> すでに絶対パスです"
  else
    # realpath: 상대 경로를 절대 경로로 변환
    # 2>/dev/null: 변환 실패 시 에러 숨기기
    # || echo "変換できません": 실패 시 대체 메시지 출력
    echo "$path -> $(realpath $path 2>/dev/null || echo "変換できません")"
  fi
done
```

> **`||` 연산자란?**  
> 앞 명령어가 **실패**했을 때만 뒷 명령어를 실행.  
> `명령어A || 명령어B` → A가 성공하면 B 실행 안 함, A가 실패하면 B 실행.  
> 반대로 `&&` 는 앞이 **성공**했을 때만 뒤를 실행.

---

## 3. 스크립트 호출 방식 4가지

셸에서 다른 스크립트를 실행하는 방법은 4가지가 있으며,  
**변수를 공유할 수 있는 범위가 각각 다르다**.

シェルで他(ほか)のスクリプトを実行(じっこう)する方法(ほうほう)は4つあり、  
**変数(へんすう)を共有(きょうゆう)できる範囲(はんい)がそれぞれ異(こと)なる**。

### ① `bash 스크립트` — export 없이 호출

```bash
name="カンホンギュ"
city="新潟"
language="Ruby"

# bash로 호출 → 자식 프로세스 생성
# export하지 않은 변수는 전달되지 않음
bash shell/script-b.sh
# → name, city, language 모두 "未設定"으로 출력됨
```

### ② `export` 후 `bash` 호출

```bash
# export: 변수를 환경변수로 등록 → 자식 프로세스에서 참조 가능
export name city language
bash shell/script-b.sh
# → name, city, language 정상 출력됨
```

### ③ `source` 호출

```bash
# source: 현재 셸에서 직접 실행 (자식 프로세스 생성 안 함)
# export 없이도 모든 변수가 공유됨
# 같은 프로세스에서 실행되므로 B 스크립트의 변수도 현재 셸에 남음
source shell/script-b.sh
# → name, city, language 정상 출력됨
# 주의: B 스크립트에서 변수를 변경하면 현재 셸에도 영향을 줌!
```

### ④ `./` 직접 실행

```bash
# ./: 파일을 직접 실행
# 실행하려면 먼저 실행 권한(chmod +x)이 필요
chmod +x shell/script-b.sh
./shell/script-b.sh
# bash로 호출하는 것과 동일하게 자식 프로세스에서 실행
# export한 변수만 전달됨
```

### 인수 전달

```bash
# bash로 호출할 때 인수를 함께 전달 가능
bash shell/script-b.sh "テスト引数1" "テスト引数2"
# → script-b.sh 안에서 $1, $2, $@ 로 접근 가능
```

### 실행 결과를 파일에 저장

```bash
# B 스크립트의 모든 출력(stdout + stderr)을 파일로 저장
bash shell/script-b.sh > result.log 2>&1
cat result.log
```

---

## 4. 변수 공유 범위 정리

| 호출 방식         | 프로세스               | export 없는 변수 | export 변수 |
| :---------------- | :--------------------- | :--------------: | :---------: |
| `bash 스크립트`   | 자식 프로세스 생성     |  ❌ 전달 안 됨   |  ✅ 전달됨  |
| `export` + `bash` | 자식 프로세스 생성     |  ❌ 전달 안 됨   |  ✅ 전달됨  |
| `source 스크립트` | 현재 프로세스에서 실행 |    ✅ 공유됨     |  ✅ 공유됨  |
| `./스크립트`      | 자식 프로세스 생성     |  ❌ 전달 안 됨   |  ✅ 전달됨  |

```
현재 셸 (부모 프로세스)
    │
    ├── bash script-b.sh   → 자식 프로세스 생성 (변수 복사본 전달)
    │                         자식이 끝나면 부모에 영향 없음
    │
    └── source script-b.sh → 자식 프로세스 없음, 같은 셸에서 실행
                              B의 변수 변경이 현재 셸에도 반영됨
```

### script-b.sh — 변수 수신 측

```bash
echo "=== Bスクリプト開始 ==="

# ${변수:-기본값}: 변수가 없으면 "未設定" 출력
echo "名前: ${name:-"未設定"}"
echo "都市: ${city:-"未設定"}"
echo "言語: ${language:-"未設定"}"

# B 스크립트 내부 변수 (A와 공유 안 됨)
script_name="script-b.sh"
result="Bスクリプトの処理が完了しました"
echo "Bスクリプト: $script_name"
echo "結果: $result"

number=10
echo "計算結果: $((number * 2))"
echo "実行時間: $(date)"

# 인수가 있으면 출력
if [ $# -gt 0 ]; then
  echo "受け取った引数: $@"
fi

echo "=== Bスクリプト終了 ==="
```

---

## 🔑 한눈에 보는 키워드

| 키워드              | 설명                              |
| :------------------ | :-------------------------------- |
| `>&2`               | 현재 출력을 stderr로 방향 전환    |
| `exit 0`            | 정상 종료                         |
| `exit 1`            | 비정상 종료 (실패)                |
| `$?`                | 직전 명령어 종료 코드 (0=성공)    |
| `[[ ]]`             | 이중 대괄호 — 패턴 매칭 지원      |
| `[[ $path == /* ]]` | `/`로 시작하는지 패턴 매칭        |
| `-e 경로`           | 파일 또는 디렉토리 존재 확인      |
| `realpath 경로`     | 상대 경로 → 절대 경로 변환        |
| `A \|\| B`          | A 실패 시 B 실행                  |
| `A && B`            | A 성공 시 B 실행                  |
| `bash 스크립트`     | 자식 프로세스에서 실행            |
| `source 스크립트`   | 현재 셸에서 직접 실행 (변수 공유) |
| `chmod +x 파일`     | 파일에 실행 권한 부여             |
| `./스크립트`        | 실행 권한 있는 스크립트 직접 실행 |
| `export 변수`       | 자식 프로세스에도 변수 전달       |

---

[← 목차로 돌아가기 / 目次(もくじ)に戻(もど)る](../README.md)
