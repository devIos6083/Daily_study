# 📅 Day 16 — 셸 스크립트 심화 (표준 출력 & 표준 에러)

**날짜 / 日付(ひづけ):** 2026.03.08  
**학습 시간 / 学習時間(がくしゅうじかん):** 자율 / 自律(じりつ)  
**한 줄 목표:** 리다이렉션으로 출력 결과와 에러를 파일에 저장하고 분리해서 관리하는 방법을 익힌다.  
**一言目標(ひとこともくひょう):** リダイレクションで出力結果(しゅつりょくけっか)とエラーをファイルに保存(ほぞん)し、分離(ぶんり)して管理(かんり)する方法(ほうほう)を身(み)につける。

---

## 목차 / 目次(もくじ)

1. [리다이렉션이란?](#1-리다이렉션이란)
2. [표준 출력 (stdout)](#2-표준-출력-stdout)
3. [표준 에러 (stderr)](#3-표준-에러-stderr)
4. [stdout & stderr 조합 패턴](#4-stdout--stderr-조합-패턴)
5. [실전 예제](#5-실전-예제)

---

## 1. 리다이렉션이란?

**명령어의 출력 방향을 바꾸는 기능**.  
기본적으로 모든 출력은 터미널 화면에 표시되는데,  
리다이렉션을 사용하면 그 출력을 **파일에 저장**하거나 **버려버리거나** 할 수 있다.

셸에는 3가지 기본 스트림(데이터의 흐름)이 있다:

**コマンドの出力先(しゅつりょくさき)を変(か)える機能(きのう)**。  
デフォルトでは全(すべ)ての出力(しゅつりょく)はターミナル画面(がめん)に表示(ひょうじ)されるが、  
リダイレクションを使(つか)えばその出力(しゅつりょく)を**ファイルに保存(ほぞん)**したり**捨(す)てたり**できる。

```
0번 — stdin  (표준 입력)  → 키보드로부터 입력
1번 — stdout (표준 출력)  → 정상 실행 결과를 터미널에 출력
2번 — stderr (표준 에러)  → 에러 메시지를 터미널에 출력

리다이렉션 = 이 스트림의 방향을 파일 등으로 바꾸는 것
```

|     기호      | 의미                                 |
| :-----------: | :----------------------------------- |
|      `>`      | stdout을 파일에 저장 (덮어쓰기)      |
|     `>>`      | stdout을 파일에 추가 (이어쓰기)      |
|     `2>`      | stderr를 파일에 저장                 |
|     `2>>`     | stderr를 파일에 추가                 |
|    `2>&1`     | stderr를 stdout과 같은 곳으로 합치기 |
| `2>/dev/null` | stderr를 버리기 (OS 휴지통)          |

---

## 2. 표준 출력 (stdout)

### `>` — 덮어쓰기

```bash
# 명령어의 출력 결과를 파일에 저장
# 파일이 없으면 새로 생성, 있으면 내용을 완전히 덮어씀
echo "標準出力のテスト" > output.txt
cat output.txt
# → 標準出力のテスト
```

### `>>` — 이어쓰기 (추가)

```bash
# 기존 파일 내용은 유지하고 뒤에 덧붙임
# 로그 파일처럼 계속 내용을 쌓아야 할 때 사용
echo "追加の内容１" >> output.txt
echo "追加の内容２" >> output.txt
cat output.txt
# → 標準出力のテスト
#   追加の内容１
#   追加の内容２
```

### 명령어 결과를 파일로 저장

```bash
# ls 결과를 파일에 저장
ls > file-list.txt

# 파이프와 함께 사용 — sh 파일 목록만 저장
ls | grep "sh" > sh-files.txt

# 파일 줄 수를 파일에 저장
wc -l stdout.sh > line-count.txt
```

### 시스템 정보 파일 만들기

```bash
# > 로 파일 새로 만들고, >> 로 계속 추가하는 패턴
# 처음에만 > 를 써서 기존 내용을 지우고 시작
echo "=== 環境情報 ===" > system-info.txt
echo "ユーザー: $USER" >> system-info.txt
echo "ディレクトリ: $PWD" >> system-info.txt
echo "日時: $(date)" >> system-info.txt

cat system-info.txt
```

### 배열 내용을 파일에 저장 후 정렬

```bash
languages=("Ruby" "Shell" "Python" "JavaScript")

# 루프로 각 요소를 한 줄씩 파일에 추가
for language in "${languages[@]}"; do
  echo $language >> languages.txt
done

# 저장된 파일을 정렬해서 새 파일로 저장
sort languages.txt > sorted-languages.txt
cat sorted-languages.txt
# → JavaScript, Python, Ruby, Shell
```

### 조건 결과를 파일에 저장

```bash
scores=(45 72 88 55 91 63)

for score in "${scores[@]}"; do
  if [ $score -ge 70 ]; then
    echo "合格: $score" >> results.txt
  else
    echo "不合格: $score" >> results.txt
  fi
done

cat results.txt
```

---

## 3. 표준 에러 (stderr)

**정상 출력(stdout)과 에러 출력(stderr)은 별개의 스트림**이다.  
`>` 로는 에러 메시지를 잡을 수 없고, 반드시 `2>` 를 사용해야 한다.  
에러와 정상 출력을 분리하면 **로그 분석이 훨씬 쉬워진다**.

**正常出力(せいじょうしゅつりょく)とエラー出力(しゅつりょく)は別(べつ)のストリーム**。  
`>`ではエラーを捕(と)らえられず、必(かなら)ず`2>`を使(つか)う必要(ひつよう)がある。

```bash
# 존재하지 않는 파일을 cat하면 에러 발생
# 2> 로 에러 메시지를 파일에 저장
cat not-exist.txt 2> error.log
cat error.log
# → cat: not-exist.txt: No such file or directory
```

### stdout과 stderr를 각각 다른 파일로 분리

```bash
# > : 정상 출력 → output.log
# 2>: 에러 출력 → error.log
ls stderr.sh not-exist.txt > output.log 2> error.log

cat output.log   # → stderr.sh (존재하는 파일)
cat error.log    # → ls: not-exist.txt: No such file or directory
```

### `2>&1` — stdout과 stderr를 하나의 파일로 합치기

```bash
# 2>&1 : stderr(2번)를 stdout(1번)이 가는 곳으로 보내기
# 즉, 정상 출력과 에러 출력을 같은 파일에 함께 저장
ls stderr.sh not-exist.txt > all.log 2>&1

cat all.log
# → stderr.sh
#   ls: not-exist.txt: No such file or directory
```

### `2>/dev/null` — 에러 메시지 숨기기

```bash
# /dev/null : OS의 휴지통 — 여기로 보낸 데이터는 완전히 버려짐
# 에러가 나도 화면에 표시하지 않고 싶을 때 사용
ls not-exist.txt 2>/dev/null
echo "エラーを非表示にしました"
# 에러 메시지 없이 다음 줄로 넘어감
```

### stderr 파일에 이어쓰기

```bash
# 기존 에러 로그에 새 에러를 추가
echo "追加エラー１" >> error.log
echo "追加エラー２" >> error.log
cat error.log
```

---

## 4. stdout & stderr 조합 패턴

### 0으로 나누기 — 에러와 정상 결과 분리

```bash
rm -f output.log error.log   # 기존 파일 초기화

numbers=(10 0 5 0 2)
for number in "${numbers[@]}"; do
  if [ $number -eq 0 ]; then
    # 0으로 나누면 에러 → error.log에 기록
    echo "エラー: 0では割り算できません" >> error.log
  else
    # 정상 계산 결과 → output.log에 기록
    echo "計算結果: $((100 / number))" >> output.log
  fi
done

echo "正常出力:"
cat output.log
# → 計算結果: 10, 計算結果: 20, 計算結果: 50

echo "エラーログ:"
cat error.log
# → エラー: 0では割り算できません (2회)
```

### `$?` — 직전 명령어의 성공/실패 확인

```bash
# $? : 직전에 실행한 명령어의 종료 코드
# 0이면 성공, 0이 아니면 실패
# 명령어 성공/실패에 따라 다른 로그 파일에 기록하는 패턴

rm -f error.log output.log

commands=("ls stderr.sh" "ls not-exist.txt" "cat stderr.sh" "cat not-exist.txt")

for command in "${commands[@]}"; do
  # > /dev/null 2>&1 : 화면 출력은 모두 숨기고 종료 코드만 확인
  $command > /dev/null 2>&1

  if [ $? -eq 0 ]; then
    echo "$command: 成功" >> output.log   # 성공 → output.log
  else
    echo "$command: 失敗" >> error.log    # 실패 → error.log
  fi
done

echo "成功ログ:"
cat output.log
# → ls stderr.sh: 成功
#   cat stderr.sh: 成功

echo "失敗ログ:"
cat error.log
# → ls not-exist.txt: 失敗
#   cat not-exist.txt: 失敗
```

---

## 5. 실전 예제 — 리다이렉션 전체 흐름

```
명령어 실행
    │
    ├─ 정상 결과 (stdout, 1번) ─→ > output.log   (저장)
    │                            >> output.log   (추가)
    │
    └─ 에러 결과 (stderr, 2번) ─→ 2> error.log   (저장)
                                  2>/dev/null    (버리기)
                                  2>&1           (stdout과 합치기)
```

```bash
# 실무에서 자주 쓰는 패턴 정리
# 1. 정상/에러 분리 저장
some_command > output.log 2> error.log

# 2. 전부 한 파일에 저장
some_command > all.log 2>&1

# 3. 에러만 숨기고 정상 출력만 표시
some_command 2>/dev/null

# 4. 정상/에러 모두 숨기기 (종료 코드만 확인할 때)
some_command > /dev/null 2>&1
```

---

## 🔑 한눈에 보는 키워드

| 키워드              | 설명                                       |
| :------------------ | :----------------------------------------- |
| `>`                 | stdout → 파일 저장 (덮어쓰기)              |
| `>>`                | stdout → 파일 추가 (이어쓰기)              |
| `2>`                | stderr → 파일 저장                         |
| `2>>`               | stderr → 파일 추가                         |
| `2>&1`              | stderr를 stdout과 같은 곳으로 합치기       |
| `2>/dev/null`       | 에러 메시지 버리기                         |
| `> /dev/null 2>&1`  | 정상/에러 출력 모두 버리기                 |
| `/dev/null`         | OS의 휴지통 — 보낸 데이터는 사라짐         |
| `$?`                | 직전 명령어의 종료 코드 (0=성공, 비0=실패) |
| `rm -f 파일`        | 파일 강제 삭제 (없어도 에러 안 남)         |
| `cat 파일`          | 파일 내용 출력                             |
| `sort 파일 > 파일2` | 파일 내용 정렬 후 새 파일로 저장           |

---

[← 목차로 돌아가기 / 目次(もくじ)に戻(もど)る](../README.md)
