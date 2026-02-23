# 📅 Day 4 — ActiveRecord & 폼(Form) 기초

**날짜 / 日付(ひづけ):** 2026.02.23  
**학습 시간 / 学習時間(がくしゅうじかん):** 2시간 / 2時間(じかん)  
**한 줄 목표:** 하드코딩 배열을 진짜 DB로 교체하고, 데이터를 직접 저장하는 폼을 만든다.  
**一言目標(ひとこともくひょう):** ハードコードの配列(はいれつ)を本物(ほんもの)のDBに切(き)り替(か)えて、データを保存(ほぞん)するフォームを作(つく)る。

---

## 목차 / 目次(もくじ)

1. [ActiveRecord란?](#1-activerecord란)
2. [마이그레이션 — DB 테이블 만들기](#2-마이그레이션--db-테이블-만들기)
3. [ActiveRecord 주요 메서드](#3-activerecord-주요-메서드)
4. [Validation — 유효성 검사](#4-validation--유효성-검사)
5. [컨트롤러 DB 연결](#5-컨트롤러-db-연결)
6. [폼(Form) 만들기](#6-폼form-만들기)
7. [오늘의 실수 & 배운 점](#7-오늘의-실수--배운-점)

---

## 1. ActiveRecord란?

Ruby 코드로 DB를 다루게 해주는 Rails의 핵심 기능입니다. SQL을 직접 쓰지 않아도 됩니다.

RubyのコードでDBを扱(あつか)えるRailsの核心(かくしん)機能(きのう)です。SQLを直接(ちょくせつ)書(か)かなくて済(す)みます。

```ruby
# ❌ SQL 직접 쓰는 방식
"SELECT * FROM posts WHERE id = 1"

# ✅ ActiveRecord 방식
Post.find(1)
```

---

## 2. 마이그레이션 — DB 테이블 만들기

마이그레이션은 DB 테이블을 만들거나 수정하는 설계도 파일입니다.

マイグレーションはDBのテーブルを作(つく)ったり変更(へんこう)したりする設計図(せっけいず)ファイルです。

```bash
# 모델 + 마이그레이션 파일 동시 생성
rails generate model Post title:string content:text author:string

# 실제 DB에 반영
rails db:migrate
```

생성된 마이그레이션 파일:

```ruby
# db/migrate/20260223000000_create_posts.rb
class CreatePosts < ActiveRecord::Migration[7.2]
  def change
    create_table :posts do |t|
      t.string :title      # 짧은 텍스트
      t.text   :content    # 긴 텍스트
      t.string :author

      t.timestamps         # created_at, updated_at 자동 생성
    end
  end
end
```

---

## 3. ActiveRecord 주요 메서드

```ruby
# 전체 조회
Post.all

# 단일 조회
Post.find(1)                   # id로 찾기 (없으면 에러)
Post.find_by(author: "강홍규") # 조건으로 첫 번째 하나 (없으면 nil)
Post.first / Post.last         # 첫 번째 / 마지막

# 조건 조회
Post.where(author: "강홍규")            # 조건에 맞는 전부
Post.where("title LIKE ?", "%Day%")    # 제목에 "Day" 포함된 것
Post.order(created_at: :desc)          # 최신순 정렬
Post.where(id: [1, 3])                 # id가 1 또는 3

# 생성
Post.create(title: "제목", content: "내용", author: "강홍규")

# 수정
post = Post.find(1)
post.update(title: "수정된 제목")

# 삭제
post = Post.find(1)
post.destroy

# 개수
Post.count
```

**find_by vs where vs find 차이:**

| 메서드          | 반환           | 없을 때 |
| :-------------- | :------------- | :------ |
| `find(id)`      | 1개            | 에러    |
| `find_by(조건)` | 1개            | nil     |
| `where(조건)`   | 여러 개 (배열) | 빈 배열 |

---

## 4. Validation — 유효성 검사

이상한 데이터가 DB에 들어오지 못하게 막는 기능입니다.

おかしなデータがDBに入(はい)らないよう防(ふせ)ぐ機能(きのう)です。

```ruby
# app/models/post.rb
class Post < ApplicationRecord
  validates :title,   presence: true
  validates :content, presence: true, length: { minimum: 10 }
  validates :author,  presence: true, length: { maximum: 20 }
end
```

콘솔에서 확인:

```ruby
post = Post.new(title: "", content: "짧음", author: "강홍규")
post.valid?                # → false
post.errors.full_messages  # → ["Title can't be blank", "Content is too short..."]

# 콘솔에서 코드 변경 반영할 때
reload!
```

---

## 5. 컨트롤러 DB 연결

하드코딩 배열 → 진짜 DB 조회로 교체합니다.

ハードコードの配列(はいれつ)→本物(ほんもの)のDB照会(しょうかい)に切(き)り替(か)えます。

```ruby
# app/controllers/posts_controller.rb
class PostsController < ApplicationController
  def index
    @posts = Post.all       # DB에서 전체 조회
  end

  def show
    @post = Post.find(params[:id])  # DB에서 id로 조회
    # params[:id]는 Rails가 자동으로 변환해줌 (.to_i 불필요)
  end

  def new
    @post = Post.new        # 빈 Post 객체 준비
  end
end
```

> **배열 vs DB 모델 접근 방식 차이:**
>
> ```ruby
> post[:title]   # 배열(해시)일 때
> post.title     # DB 모델일 때 (메서드처럼 점으로!)
> ```

---

## 6. 폼(Form) 만들기

`form_with`로 HTML 폼을 Rails답게 만들 수 있습니다.

`form_with`でHTMLフォームをRailsらしく作(つく)れます。

```ruby
# config/routes.rb — new보다 :id가 아래 있어야 함!
get  "/posts/new",  to: "posts#new"    # ← 반드시 위에!
get  "/posts/:id",  to: "posts#show"   # ← 아래에!
post "/posts",      to: "posts#create"
```

```erb
<!-- app/views/posts/new.html.erb -->
<h1>새 게시글 작성</h1>

<%= form_with model: @post, url: "/posts", method: :post do |f| %>
  <p>
    <%= f.label :title, "제목" %><br>
    <%= f.text_field :title %>
  </p>
  <p>
    <%= f.label :content, "내용" %><br>
    <%= f.text_area :content %>
  </p>
  <p>
    <%= f.label :author, "작성자" %><br>
    <%= f.text_field :author %>
  </p>
  <%= f.submit "저장" %>
<% end %>

<a href="/posts">← 목록으로</a>
```

**form_with 핵심 패턴:**

```erb
f.label    :필드명, "라벨텍스트"  ← 콤마 필수!
f.text_field :필드명              ← 한 줄 입력
f.text_area  :필드명              ← 여러 줄 입력
f.submit "버튼텍스트"
```

---

## 7. 오늘의 실수 & 배운 점

今日(きょう)つまずいた部分(ぶぶん)を記録(きろく)します。

| 실수                     | 원인                             | 올바른 방법                 |
| :----------------------- | :------------------------------- | :-------------------------- |
| `vaildates`              | `validates` 철자 오타 (i,l 순서) | `validates`                 |
| `params.id`              | params는 해시, `.`으로 접근 불가 | `params[:id]`               |
| `form with`              | 언더바 빠짐                      | `form_with`                 |
| `f.label :title "제목"`  | 콤마 빠짐                        | `f.label :title, "제목"`    |
| `f.content`, `f.author`  | 없는 메서드                      | `f.label` 사용              |
| `/posts/new` → show 에러 | routes 순서 잘못됨               | `new`를 `:id`보다 위에 선언 |

> **오늘의 핵심 기억:**
>
> 1. `Post.all` → DB 전체 조회, `post.title` → 점으로 접근
> 2. routes에서 `/posts/new`는 `/posts/:id`보다 **반드시 위에** 선언
> 3. `form_with`의 `f.label`은 항상 **콤마** 필요

---

## 🔑 한눈에 보는 키워드

| 키워드                                | 설명                          |
| :------------------------------------ | :---------------------------- |
| `rails generate model 이름 필드:타입` | 모델 + 마이그레이션 파일 생성 |
| `rails db:migrate`                    | 마이그레이션 DB에 반영        |
| `rails console` / `rails c`           | DB를 직접 조작하는 콘솔       |
| `reload!`                             | 콘솔에서 코드 변경사항 반영   |
| `Post.all`                            | 전체 조회                     |
| `Post.find(id)`                       | id로 단일 조회                |
| `Post.where(조건)`                    | 조건으로 여러 개 조회         |
| `Post.create(...)`                    | 생성 + 저장                   |
| `post.update(...)`                    | 수정 + 저장                   |
| `post.destroy`                        | 삭제                          |
| `validates`                           | 유효성 검사                   |
| `form_with`                           | Rails 폼 헬퍼                 |

---

[← 목차로 돌아가기 / 目次(もくじ)に戻(もど)る](../README.md)
