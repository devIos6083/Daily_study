# 📅 Day 5 — CRUD 완성 & Flash 메시지

**날짜 / 日付(ひづけ):** 2026.02.24  
**학습 시간 / 学習時間(がくしゅうじかん):** 2시간 / 2時間(じかん)  
**한 줄 목표:** create / edit / update / destroy 를 완성해서 진짜 게시판을 만든다.  
**一言目標(ひとこともくひょう):** create・edit・update・destroyを完成(かんせい)させて、本物(ほんもの)の掲示板(けいじばん)を作(つく)る。

---

## 목차 / 目次(もくじ)

1. [CRUD란?](#1-crud란)
2. [Strong Parameters — 보안 필터](#2-strong-parameters--보안-필터)
3. [before_action — 중복 코드 제거](#3-before_action--중복-코드-제거)
4. [redirect_to vs render](#4-redirect_to-vs-render)
5. [전체 코드](#5-전체-코드)
6. [Flash 메시지](#6-flash-메시지)
7. [오늘의 실수 & 배운 점](#7-오늘의-실수--배운-점)

---

## 1. CRUD란?

웹 서비스의 기본 동작 4가지입니다. Rails 실무의 80%가 CRUD입니다.

Webサービスの基本(きほん)動作(どうさ)4つです。Railsの現場(げんば)の80%がCRUDです。

|    이름    | 동작 | HTTP 메서드 |    Rails 액션     |
| :--------: | :--: | :---------: | :---------------: |
| **C**reate | 생성 |    POST     | `new` / `create`  |
|  **R**ead  | 조회 |     GET     | `index` / `show`  |
| **U**pdate | 수정 |    PATCH    | `edit` / `update` |
| **D**elete | 삭제 |   DELETE    |     `destroy`     |

---

## 2. Strong Parameters — 보안 필터

폼에서 넘어온 데이터를 그대로 DB에 저장하면 위험합니다.  
허용할 필드만 명시해서 보안을 지킵니다.

フォームから送(おく)られたデータをそのままDBに保存(ほぞん)するのは危険(きけん)です。  
許可(きょか)するフィールドだけ明示(めいじ)してセキュリティを守(まも)ります。

```ruby
# ❌ 위험한 방식 — 모든 필드를 그대로 저장
Post.create(params[:post])

# ✅ Strong Parameters — 허용할 필드만 명시
def post_params
  params.require(:post).permit(:title, :content, :author)
end
Post.create(post_params)
```

---

## 3. before_action — 중복 코드 제거

`show`, `edit`, `update`, `destroy` 에서 매번 `@post`를 선언하는 중복을 제거합니다.  
지정한 액션 실행 전에 자동으로 호출됩니다.

`show`・`edit`・`update`・`destroy`での`@post`宣言(せんげん)の重複(じゅうふく)をなくします。  
指定(してい)したアクションの前(まえ)に自動(じどう)で呼(よ)ばれます。

```ruby
class PostsController < ApplicationController
  # only: 로 특정 액션에서만 실행되도록 지정
  before_action :set_post, only: [:show, :edit, :update, :destroy]

  private

  def set_post
    @post = Post.find(params[:id])
  end
end
```

---

## 4. redirect_to vs render

```ruby
# redirect_to: 브라우저에게 "이 URL로 다시 요청해" 라고 지시
# → 저장/수정/삭제 성공 후 페이지 이동할 때 사용
redirect_to "/posts/#{@post.id}"

# render: 지금 바로 이 뷰를 화면에 그려라
# → 저장/수정 실패 시 폼을 다시 보여줄 때 사용
render :new   # new.html.erb 렌더링
render :edit  # edit.html.erb 렌더링
```

---

## 5. 전체 코드

### routes.rb

```ruby
Rails.application.routes.draw do
  root "home#index"
  get  "/about",          to: "home#about"

  get  "/posts",          to: "posts#index"
  get  "/posts/new",      to: "posts#new"      # ← :id 보다 반드시 위에!
  get  "/posts/:id",      to: "posts#show"
  post "/posts",          to: "posts#create"
  get  "/posts/:id/edit", to: "posts#edit"
  patch  "/posts/:id",    to: "posts#update"
  delete "/posts/:id",    to: "posts#destroy"
end
```

### posts_controller.rb

```ruby
class PostsController < ApplicationController
  # show, edit, update, destroy 실행 전 set_post 자동 호출
  before_action :set_post, only: [:show, :edit, :update, :destroy]

  def index
    # 최신순으로 전체 게시글 조회
    @posts = Post.order(created_at: :desc)
  end

  # before_action이 @post를 처리하므로 비워둠
  def show
  end

  def new
    # 빈 Post 객체 준비
    @post = Post.new
  end

  def create
    # post_params로 Post 객체 생성 (아직 DB 저장 안 함)
    @post = Post.new(post_params)

    # 저장 성공 → 상세 페이지로 이동 / 실패 → new 폼 다시 렌더링
    if @post.save
      flash[:notice] = "게시글이 저장되었습니다! ✅"
      redirect_to "/posts/#{@post.id}"
    else
      render :new
    end
  end

  # before_action이 @post를 처리하므로 비워둠
  def edit
  end

  def update
    # @post를 post_params로 수정 후 저장
    if @post.update(post_params)
      flash[:notice] = "게시글이 수정되었습니다! ✏️"
      redirect_to "/posts/#{@post.id}"
    else
      # 실패하면 edit 폼 다시 렌더링
      render :edit
    end
  end

  def destroy
    # @post 삭제 후 목록 페이지로 이동
    @post.destroy
    flash[:notice] = "게시글이 삭제되었습니다! 🗑️"
    redirect_to "/posts"
  end

  private

  # params[:id]로 DB에서 게시글 찾기
  def set_post
    @post = Post.find(params[:id])
  end

  # 허용할 필드만 명시
  def post_params
    params.require(:post).permit(:title, :content, :author)
  end
end
```

### show.html.erb

```erb
<h1><%= @post.title %></h1>
<p><%= @post.content %></p>
<p>작성자: <%= @post.author %></p>
<p>작성일: <%= @post.created_at.strftime("%Y.%m.%d") %></p>

<!-- 수정 페이지로 이동 -->
<a href="/posts/<%= @post.id %>/edit">수정하기</a>

<!-- 삭제 버튼 — method: :delete 심볼 필수! -->
<%= button_to "삭제하기", "/posts/#{@post.id}",
    method: :delete,
    data: { turbo_confirm: "정말 삭제할까요?" } %>

<a href="/posts">← 목록으로</a>
```

### edit.html.erb

```erb
<h1>게시글 수정</h1>

<!-- method: :patch 로 수정 요청 전송 -->
<%= form_with model: @post, url: "/posts/#{@post.id}", method: :patch do |f| %>
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
  <%= f.submit "수정하기" %>
<% end %>

<a href="/posts/<%= @post.id %>">← 취소</a>
```

### index.html.erb

```erb
<h1>게시글 목록</h1>

<!-- 새 게시글 작성 링크 — 루프 밖에! -->
<a href="/posts/new">새 게시글 작성</a>

<!-- 게시글이 없을 때 안내 문구 -->
<% if @posts.empty? %>
  <p>아직 게시물이 없습니다.</p>
<% else %>
  <% @posts.each do |post| %>
    <p>
      [<%= post.id %>]
      <a href="/posts/<%= post.id %>"><%= post.title %></a>
      <small>(<%= post.created_at.strftime("%Y.%m.%d") %>)</small>
    </p>
  <% end %>
<% end %>

<a href="/">홈으로</a>
```

---

## 6. Flash 메시지

저장 / 수정 / 삭제 후 화면에 알림 메시지를 띄웁니다.  
`app/views/layouts/application.html.erb` 의 `<body>` 바로 아래에 추가합니다.

保存(ほぞん)・修正(しゅうせい)・削除(さくじょ)の後(あと)に画面(がめん)に通知(つうち)メッセージを表示(ひょうじ)します。

```erb
<body>
  <!-- notice: 초록색 / alert: 빨간색 -->
  <% if flash[:notice] %>
    <p style="color: green;"><%= flash[:notice] %></p>
  <% end %>

  <%= yield %>
</body>
```

---

## 7. 오늘의 실수 & 배운 점

今日(きょう)つまずいた部分(ぶぶん)を記録(きろく)します。

| 실수                       | 원인                           | 올바른 방법         |
| :------------------------- | :----------------------------- | :------------------ |
| `method: delete`           | 심볼 콜론 빠짐                 | `method: :delete`   |
| `created_at: desc`         | 심볼 콜론 빠짐                 | `created_at: :desc` |
| `@posts.empty`             | `?` 빠짐                       | `@posts.empty?`     |
| `<p>post</p>`              | ERB 태그 없이 변수 출력 시도   | `<%= post.title %>` |
| 링크를 each 루프 안에 선언 | 게시글 수만큼 링크가 반복 출력 | 루프 밖으로 꺼내기  |

> **오늘의 핵심 기억:**
>
> 1. 심볼은 항상 콜론부터 — `:delete`, `:desc`, `:patch`
> 2. `before_action`으로 중복 코드 제거
> 3. 성공 → `redirect_to` / 실패 → `render`

---

## 🔑 한눈에 보는 키워드

| 키워드                               | 설명                                 |
| :----------------------------------- | :----------------------------------- |
| `before_action :메서드, only: [...]` | 지정 액션 실행 전 자동 호출          |
| `params.require(:post).permit(...)`  | Strong Parameters — 허용 필드만 통과 |
| `@post.save`                         | DB에 저장, 성공 시 true              |
| `@post.update(params)`               | DB 수정, 성공 시 true                |
| `@post.destroy`                      | DB에서 삭제                          |
| `redirect_to "경로"`                 | 브라우저를 다른 URL로 이동           |
| `render :뷰이름`                     | 해당 뷰를 바로 렌더링                |
| `flash[:notice]`                     | 다음 요청까지 유지되는 알림 메시지   |
| `button_to`                          | DELETE/PATCH 요청 버튼 생성          |
| `@posts.empty?`                      | 배열이 비어있으면 true               |

---

[← 목차로 돌아가기 / 目次(もくじ)に戻(もど)る](../README.md)
