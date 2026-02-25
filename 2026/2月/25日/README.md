# 📅 Day 6 — 모델 관계(Association) & 댓글 기능

**날짜 / 日付(ひづけ):** 2026.02.25  
**학습 시간 / 学習時間(がくしゅうじかん):** 2시간 / 2時間(じかん)  
**한 줄 목표:** has_many / belongs_to 관계를 이해하고, 게시판에 댓글 기능을 직접 구현한다.  
**一言目標(ひとこともくひょう):** has_many・belongs_toの関係(かんけい)を理解(りかい)して、掲示板(けいじばん)にコメント機能(きのう)を実装(じっそう)する。

---

## 목차 / 目次(もくじ)

1. [Association이란?](#1-association이란)
2. [has_many / belongs_to](#2-has_many--belongs_to)
3. [중첩 라우팅 (resources)](#3-중첩-라우팅-resources)
4. [CommentsController](#4-commentscontroller)
5. [View — 댓글 폼 & 목록](#5-view--댓글-폼--목록)
6. [커스텀 Validation](#6-커스텀-validation)
7. [검색 기능](#7-검색-기능)
8. [오늘의 실수 & 배운 점](#8-오늘의-실수--배운-점)

---

## 1. Association이란?

테이블 간의 관계를 Rails가 자동으로 연결해주는 기능입니다.

テーブル間(かん)の関係(かんけい)をRailsが自動(じどう)でつないでくれる機能(きのう)です。

```
Posts 테이블              Comments 테이블
┌─────────────┐           ┌──────────────────┐
│ id: 1       │◄──────┐   │ id: 1            │
│ title: ...  │       │   │ body: "좋아요!"   │
└─────────────┘       └───│ post_id: 1       │
                          └──────────────────┘
```

관계 선언만으로 아래 메서드가 전부 자동 생성됩니다.

関係(かんけい)を宣言(せんげん)するだけで、下(した)のメソッドがすべて自動(じどう)で使(つか)えます。

```ruby
post.comments              # 이 게시글의 댓글 전체
post.comments.create(...)  # 댓글 생성
post.comments.count        # 댓글 수
comment.post               # 이 댓글이 속한 게시글 (역방향 접근)
comment.post.title         # 댓글 → 게시글 → 제목
```

---

## 2. has_many / belongs_to

```ruby
# app/models/post.rb
class Post < ApplicationRecord
  # 게시글은 댓글을 여러 개 가진다
  # dependent: :destroy → 게시글 삭제 시 댓글도 함께 삭제
  has_many :comments, dependent: :destroy

  validates :title,   presence: true
  validates :content, presence: true, length: { minimum: 10 }
  validates :author,  presence: true, length: { maximum: 20 }
end
```

```ruby
# app/models/comment.rb
class Comment < ApplicationRecord
  # 댓글은 반드시 게시글 하나에 속한다
  belongs_to :post

  validates :body, presence: true

  # 커스텀 validation — 댓글 최대 5개 제한
  validate :maximum_comments

  private

  def maximum_comments
    if post.comments.count >= 5
      errors.add(:base, "댓글은 최대 5개만 달 수 있어요")
    end
  end
end
```

**핵심 차이:**

|           |    `has_many`    |    `belongs_to`     |
| :-------: | :--------------: | :-----------------: |
| 사용 위치 | 부모 모델 (Post) | 자식 모델 (Comment) |
|   의미    | 여러 개를 가진다 |    하나에 속한다    |
|  DB 컬럼  |       없음       |   `post_id` 필요    |

---

## 3. 중첩 라우팅 (resources)

`resources` 한 줄로 CRUD 라우트 전체를 자동 생성합니다.

`resources`一行(いちぎょう)でCRUDルートを全(すべ)て自動(じどう)生成(せいせい)します。

```ruby
# config/routes.rb
Rails.application.routes.draw do
  root "home#index"
  get "/about", to: "home#about"

  resources :posts do
    # 댓글은 posts 안에 중첩 — only로 필요한 것만 생성
    resources :comments, only: [:create, :destroy]
  end
end
```

생성되는 주요 라우트:

```
GET    /posts                          → posts#index
GET    /posts/new                      → posts#new
POST   /posts                          → posts#create
GET    /posts/:id                      → posts#show
GET    /posts/:id/edit                 → posts#edit
PATCH  /posts/:id                      → posts#update
DELETE /posts/:id                      → posts#destroy
POST   /posts/:post_id/comments        → comments#create
DELETE /posts/:post_id/comments/:id    → comments#destroy
```

> **주의:** `resources :comment` (단수) ❌ → `resources :comments` (복수) ✅

---

## 4. CommentsController

```ruby
# app/controllers/comments_controller.rb
class CommentsController < ApplicationController

  def create
    # URL의 post_id로 게시글 찾기
    @post = Post.find(params[:post_id])

    # @post의 댓글로 연결해서 새 댓글 객체 생성
    @comment = @post.comments.new(comment_params)

    # 저장 성공 → 게시글 상세 페이지로 이동
    if @comment.save
      flash[:notice] = "댓글이 등록됐습니다! 💬"
      redirect_to "/posts/#{@post.id}"
    # 저장 실패 → 에러 메시지 표시
    else
      flash[:alert] = @comment.errors.full_messages.join(", ")
      redirect_to "/posts/#{@post.id}"
    end
  end

  def destroy
    # URL의 post_id로 게시글 찾기
    @post = Post.find(params[:post_id])

    # @post의 댓글 중 params[:id]에 해당하는 댓글 찾기
    @comment = @post.comments.find(params[:id])

    # 댓글 삭제 후 게시글 상세 페이지로 이동
    @comment.destroy
    flash[:notice] = "댓글이 삭제되었습니다."
    redirect_to "/posts/#{@post.id}"
  end

  private

  # body, author 필드만 허용
  def comment_params
    params.require(:comment).permit(:body, :author)
  end
end
```

---

## 5. View — 댓글 폼 & 목록

```erb
<!-- app/views/posts/show.html.erb -->
<h1><%= @post.title %></h1>
<p><%= @post.content %></p>
<p>작성자: <%= @post.author %></p>
<p>작성일: <%= @post.created_at.strftime("%Y.%m.%d") %></p>

<a href="/posts/<%= @post.id %>/edit">수정하기</a>
<%= button_to "삭제하기", "/posts/#{@post.id}",
    method: :delete,
    data: { turbo_confirm: "정말 삭제할까요?" } %>

<hr>

<!-- 댓글 개수 표시 -->
<h2>댓글 <%= @post.comments.count %>개</h2>

<!-- 댓글 목록 순회 -->
<% @post.comments.each do |comment| %>
  <div>
    <p>작성자: <%= comment.author %></p>
    <p><%= comment.body %></p>
    <small><%= comment.created_at.strftime("%Y.%m.%d %H:%M") %></small>
    <%= button_to "삭제", "/posts/#{@post.id}/comments/#{comment.id}",
        method: :delete,
        data: { turbo_confirm: "댓글을 삭제할까요?" } %>
  </div>
  <hr>
<% end %>

<!-- 댓글 작성 폼 — scope: :comment 로 comment[body] 형태로 전송 -->
<h3>댓글 작성</h3>
<%= form_with url: "/posts/#{@post.id}/comments",
              method: :post,
              scope: :comment do |f| %>
  <p>
    <%= f.label :author, "작성자" %><br>
    <%= f.text_field :author %>
  </p>
  <p>
    <%= f.text_area :body, placeholder: "댓글을 입력하세요" %>
  </p>
  <%= f.submit "등록" %>
<% end %>

<a href="/posts">← 목록으로</a>
```

> **핵심:** `form_with` 에 `scope: :comment` 를 추가해야  
> `{"comment"=>{"body"=>"..."}}` 형태로 전송됩니다.  
> 없으면 `{"body"=>"..."}` 로 전송되어 `comment_params` 에러 발생!

---

## 6. 커스텀 Validation

```ruby
# validates (복수) — 필드 검증
validates :body, presence: true

# validate (단수) — 커스텀 메서드 검증
validate :maximum_comments

private

def maximum_comments
  if post.comments.count >= 5
    errors.add(:base, "댓글은 최대 5개만 달 수 있어요")
  end
end
```

|      |            `validates`            |          `validate`          |
| :--: | :-------------------------------: | :--------------------------: |
| 철자 |               복수                |             단수             |
| 용도 |             필드 검증             |      커스텀 메서드 등록      |
| 예시 | `validates :body, presence: true` | `validate :maximum_comments` |

---

## 7. 검색 기능

```ruby
# app/controllers/posts_controller.rb
def index
  # 검색어가 있으면 필터링, 없으면 전체 조회
  if params[:search].present?
    @posts = Post.where("title LIKE ?", "%#{params[:search]}%")
                 .order(created_at: :desc)
  else
    @posts = Post.order(created_at: :desc)
  end
end
```

```erb
<!-- index.html.erb 검색 폼 -->
<%= form_with url: "/posts", method: :get do |f| %>
  <%= f.text_field :search, placeholder: "검색어를 입력하세요",
      value: params[:search] %>
  <%= f.submit "검색" %>
<% end %>
```

---

## 8. 오늘의 실수 & 배운 점

今日(きょう)つまずいた部分(ぶぶん)を記録(きろく)します。

| 실수                        | 원인                                | 올바른 방법                       |
| :-------------------------- | :---------------------------------- | :-------------------------------- |
| `resources :comment`        | 단수로 선언                         | `resources :comments` 복수        |
| `form_with` 에 `scope` 없음 | `body`가 `comment` 안으로 안 들어감 | `scope: :comment` 추가            |
| `validates :maximum`        | validates는 필드 검증용             | `validate :maximum_comments` 단수 |
| `post.comment.count`        | 단수                                | `post.comments.count` 복수        |
| `params[:search].presence?` | 없는 메서드                         | `params[:search].present?`        |
| `@post = Post.where(...)`   | 단수 변수명                         | `@posts = Post.where(...)` 복수   |

> **오늘의 핵심 기억:**
>
> 1. `has_many` (부모) / `belongs_to` (자식) — DB에는 자식 쪽에 `post_id` 컬럼 필요
> 2. `resources` 한 줄 = CRUD 라우트 전체 자동 생성
> 3. `validates` (복수) 필드 검증 / `validate` (단수) 커스텀 메서드
> 4. `form_with` 에 `scope: :모델명` 없으면 params 구조가 달라짐

---

## 🔑 한눈에 보는 키워드

| 키워드                        | 설명                                      |
| :---------------------------- | :---------------------------------------- |
| `has_many :comments`          | 게시글이 댓글 여러 개를 가짐              |
| `belongs_to :post`            | 댓글이 게시글 하나에 속함                 |
| `dependent: :destroy`         | 부모 삭제 시 자식도 함께 삭제             |
| `resources :posts`            | CRUD 라우트 자동 생성                     |
| `only: [:create, :destroy]`   | 지정한 액션만 라우트 생성                 |
| `post.comments.new(...)`      | 게시글에 연결된 댓글 객체 생성            |
| `scope: :comment`             | form_with 전송 데이터를 comment 안에 담기 |
| `validate :메서드명`          | 커스텀 검증 메서드 등록                   |
| `errors.add(:base, "메시지")` | 커스텀 에러 메시지 추가                   |
| `params[:search].present?`    | 검색어가 있는지 확인                      |
| `LIKE ?`                      | SQL 부분 일치 검색                        |

---

[← 목차로 돌아가기 / 目次(もくじ)に戻(もど)る](../README.md)
