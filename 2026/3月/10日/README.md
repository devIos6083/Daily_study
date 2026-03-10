# 📅 Day 18 — 셸 스크립트 실전 (파일 조작 & 자동 생성 & 백업)

**날짜 / 日付(ひづけ):** 2026.03.10  
**학습 시간 / 学習時間(がくしゅうじかん):** 자율 / 自律(じりつ)  
**한 줄 목표:** 파일/디렉토리 조작 명령어를 익히고, 이중 루프와 날짜 포맷을 활용한 실전 스크립트를 완성한다.  
**一言目標(ひとこともくひょう):** ファイル・ディレクトリ操作(そうさ)コマンドを習得(しゅうとく)し、二重(にじゅう)ループと日付(ひづけ)フォーマットを活用(かつよう)した実践(じっせん)スクリプトを完成(かんせい)させる。

---

## 목차 / 目次(もくじ)

1. [파일/디렉토리 조작 명령어](#1-파일디렉토리-조작-명령어)
2. [file-operations.sh — 디렉토리 & 파일 이동](#2-file-operationssh--디렉토리--파일-이동)
3. [generate-files.sh — 이중 루프로 파일 자동 생성](#3-generate-filessh--이중-루프로-파일-자동-생성)
4. [backup.sh — 날짜 기반 자동 백업](#4-backupsh--날짜-기반-자동-백업)

---

## 1. 파일/디렉토리 조작 명령어

셸 스크립트에서 파일과 디렉토리를 다루는 기본 명령어들.  
이 명령어들을 조합하면 수작업으로 하던 반복 작업을 자동화할 수 있다.

シェルスクリプトでファイルとディレクトリを扱(あつか)う基本(きほん)コマンド。

| 명령어           | 의미                                        | 자주 쓰는 옵션                            |
| :--------------- | :------------------------------------------ | :---------------------------------------- |
| `mkdir 디렉토리` | 디렉토리 생성                               | `-p`: 중간 경로까지 한 번에 생성          |
| `touch 파일`     | 빈 파일 생성 (이미 있으면 수정 시각만 갱신) | —                                         |
| `cd 경로`        | 디렉토리 이동                               | `..`: 한 단계 위로                        |
| `mv 원본 목적지` | 파일/디렉토리 이동 또는 이름 변경           | —                                         |
| `cp 원본 목적지` | 파일/디렉토리 복사                          | `-r`: 디렉토리 전체 복사                  |
| `rm 파일`        | 파일 삭제                                   | `-r`: 디렉토리 전체 삭제, `-f`: 강제 삭제 |
| `ls 경로`        | 경로 내 파일 목록 확인                      | —                                         |
| `tree 경로`      | 디렉토리 구조를 트리 형태로 출력            | —                                         |

### 경로 표기

```
../  → 현재 디렉토리의 한 단계 위 (부모 디렉토리)
./   → 현재 디렉토리
/    → 루트 디렉토리 (절대 경로의 시작)

예: 현재 위치가 child-directory-a/ 일 때
  ../child-directory-b/test.txt
  → 부모로 올라간 뒤 child-directory-b 안의 test.txt
```

---

## 2. file-operations.sh — 디렉토리 & 파일 이동

현재 디렉토리에 `child-directory-a`와 `child-directory-b`를 만들고,  
파일을 디렉토리 간에 이동시키는 스크립트.

カレントディレクトリに2つのディレクトリを作(つく)り、ファイルを移動(いどう)させるスクリプト。

### 전체 코드

```bash
# 이전 실행 결과 초기화 — 항상 깨끗한 상태에서 시작
rm -rf child-directory-a child-directory-b

# 디렉토리 생성
mkdir child-directory-a
mkdir child-directory-b
echo "ディレクトリを作成しました"

# child-directory-a 안에 test.txt 생성
touch child-directory-a/test.txt
echo "child-directory-aにtest.txtを作成しました"

# child-directory-a로 이동
cd child-directory-a
echo "child-directory-aに移動しました: $PWD"

# test.txt를 child-directory-b로 이동
# ../ 로 부모 디렉토리를 거쳐서 형제 디렉토리로 이동
mv test.txt ../child-directory-b/test.txt
echo "test.txtをchild-directory-bに移動しました"

# child-directory-b로 이동
cd ../child-directory-b
echo "child-directory-bに移動しました: $PWD"

# test.txt를 부모 디렉토리로 이동
mv test.txt ../test.txt
echo "test.txtを親ディレクトリに移動しました"
```

### 실행 흐름

```
[시작]
  shell/
  └── file-operations.sh

[실행 중 — touch 후]
  shell/
  ├── file-operations.sh
  ├── child-directory-a/
  │   └── test.txt         ← 여기에 생성
  └── child-directory-b/

[mv 1회 후 — child-b로 이동]
  shell/
  ├── child-directory-a/   ← 비어있음
  └── child-directory-b/
      └── test.txt         ← 이동됨

[mv 2회 후 — 부모로 이동]
  shell/
  ├── test.txt             ← 최종 위치
  ├── child-directory-a/
  └── child-directory-b/
```

---

## 3. generate-files.sh — 이중 루프로 파일 자동 생성

인수로 받은 정수 N만큼 디렉토리를 생성하고,  
각 디렉토리 안에 **인덱스 수만큼 파일**을 생성하는 스크립트.

受(う)け取(と)った整数(せいすう)Nの数(かず)だけディレクトリを生成(せいせい)し、  
各(かく)ディレクトリに**インデックス数(かず)分(ぶん)のファイル**を生成(せいせい)するスクリプト。

### 입력값 검증

```bash
# 인수가 없으면 사용법 출력 후 종료
if [ $# -eq 0 ]; then
  echo "使用方法: bash generate-files.sh [整数]"
  exit 1
fi

# 정규식으로 정수 여부 확인
# =~: 정규식 매칭 연산자 → [[ ]] 이중 대괄호에서만 사용 가능
# ^[0-9]+$: 처음부터 끝까지 숫자만 있는 문자열
# !: 조건 반전 → 숫자가 아니면 에러
if ! [[ $1 =~ ^[0-9]+$ ]]; then
  echo "エラー: 整数を入力してください"
  exit 1
fi

number=$1
```

### 이중 루프로 파일 생성

```bash
# 기존 out 디렉토리 초기화 후 새로 생성
rm -rf out
mkdir out

# 외부 루프: dir-1, dir-2, ..., dir-N 생성
for i in $(seq 1 $number); do
  mkdir out/dir-$i

  # 내부 루프: dir-i 안에 file-1 ~ file-i 생성
  # 핵심: j의 범위가 1~i → i가 클수록 파일 수도 많아짐
  #   dir-1 → file-1 (1개)
  #   dir-2 → file-1, file-2 (2개)
  #   dir-3 → file-1, file-2, file-3 (3개)
  for j in $(seq 1 $i); do
    touch out/dir-$i/file-$j
  done
done

# tree: 디렉토리 구조를 트리로 시각화
tree out
```

### 실행 결과 예시 (N=3)

```
out/
├── dir-1/
│   └── file-1
├── dir-2/
│   ├── file-1
│   └── file-2
└── dir-3/
    ├── file-1
    ├── file-2
    └── file-3
```

> **이중 루프의 핵심 포인트**  
> 외부 루프 변수 `i`를 내부 루프의 **끝 범위**로 사용하는 것이 핵심.  
> `seq 1 $i` → i가 커질수록 내부 루프 횟수도 함께 늘어남.

---

## 4. backup.sh — 날짜 기반 자동 백업

지정한 파일/디렉토리를 **날짜+시각 이름의 폴더에 자동 백업**하는 스크립트.  
실행할 때마다 다른 폴더명이 생기므로 **백업 이력이 자동으로 쌓인다**.  
실무에서 배포 전 백업, DB 덤프 보관 등에 그대로 쓸 수 있는 패턴이다.

指定(してい)したファイル・ディレクトリを**日時付(ひじつつ)きフォルダに自動(じどう)バックアップ**するスクリプト。  
実行(じっこう)のたびに異(こと)なるフォルダ名(めい)が生(う)まれ、**バックアップ履歴(りれき)が自動(じどう)で蓄積(ちくせき)される**。

### 날짜 포맷 — `date +형식`

```bash
# date +포맷: 날짜/시각을 원하는 형식의 문자열로 출력
backup_date=$(date +%Y%m%d_%H%M%S)
# 2026년 3월 10일 14시 30분 45초라면 → 20260310_143045

# 자주 쓰는 포맷 기호
# %Y: 4자리 연도     (2026)
# %m: 2자리 월       (03)
# %d: 2자리 일       (10)
# %H: 2자리 시 (24h) (14)
# %M: 2자리 분       (30)
# %S: 2자리 초       (45)
```

### 전체 코드

```bash
# 인수 검증
if [ $# -eq 0 ]; then
  echo "使用方法: bash backup.sh [バックアップするファイル/ディレクトリ]"
  exit 1
fi

target=$1

# ! -e: 파일/디렉토리가 존재하지 않으면(not exist) true
if [ ! -e $target ]; then
  echo "エラー: ${target}が存在しません"
  exit 1
fi

# 실행 시각을 백업 폴더 이름으로 사용
backup_date=$(date +%Y%m%d_%H%M%S)

# mkdir -p: backup/ 폴더가 없어도 중간 경로까지 한 번에 생성
mkdir -p backup

backup_dir="backup/$backup_date"
mkdir $backup_dir
echo "バックアップディレクトリを作成しました: $backup_dir"

# cp -r: 디렉토리도 포함해서 재귀적으로 전체 복사
cp -r $target $backup_dir
echo "バックアップ完了しました"
echo "バックアップ(元): $target"
echo "バックアップ(先): $backup_dir"

echo "バックアップ内容"
ls $backup_dir
```

### 실행 결과 예시

```bash
bash backup.sh myproject/

# 실행 결과:
backup/
└── 20260310_143045/
    └── myproject/
        ├── file1.txt
        └── file2.txt

# 한 번 더 실행하면 이력이 쌓임:
backup/
├── 20260310_143045/
│   └── myproject/
└── 20260310_150012/    ← 새로운 백업
    └── myproject/
```

> **`mkdir -p` 가 중요한 이유**  
> `mkdir backup/20260310_143045` 처럼 중간 경로인 `backup/` 이 없으면 에러가 난다.  
> `-p` 옵션을 쓰면 경로 중간에 없는 디렉토리를 모두 자동으로 만들어 준다.  
> 스크립트 첫 실행 시처럼 `backup/` 자체가 없는 상황에도 안전하게 동작한다.

---

## 🔑 한눈에 보는 키워드

| 키워드                        | 설명                                        |
| :---------------------------- | :------------------------------------------ |
| `mkdir 경로`                  | 디렉토리 생성                               |
| `mkdir -p 경로`               | 중간 경로까지 포함해서 한 번에 생성         |
| `touch 파일`                  | 빈 파일 생성                                |
| `cd 경로`                     | 디렉토리 이동                               |
| `../`                         | 부모 디렉토리                               |
| `mv 원본 목적지`              | 파일/디렉토리 이동 또는 이름 변경           |
| `cp -r 원본 목적지`           | 디렉토리 포함 재귀적 복사                   |
| `rm -rf 경로`                 | 디렉토리 전체 강제 삭제                     |
| `tree 경로`                   | 디렉토리 구조 트리로 시각화                 |
| `date +%Y%m%d_%H%M%S`         | 날짜+시각을 `20260310_143045` 형식으로 출력 |
| `! -e 경로`                   | 파일/디렉토리가 존재하지 않으면 true        |
| `=~`                          | 정규식 매칭 (`[[ ]]` 안에서만 사용 가능)    |
| `^[0-9]+$`                    | 숫자만 있는 문자열인지 확인하는 정규식      |
| `for i; do for j; done; done` | 이중 루프                                   |

---

[← 목차로 돌아가기 / 目次(もくじ)に戻(もど)る](../README.md)
