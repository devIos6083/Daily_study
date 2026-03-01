# 📅 Day 9 — 세션(Session) & 인증(Authentication) 기초

**날짜 / 日付(ひづけ):** 2026.03.01  
**학습 시간 / 学習時間(がくしゅうじかん):** 2시간 / 2時間(じかん)  
**한 줄 목표:** 세션으로 작성자를 기억하고, ApplicationController로 공통 인증 로직을 관리한다.  
**一言目標(ひとこともくひょう):** セッションで作者(さくしゃ)を記憶(きおく)し、ApplicationControllerで共通(きょうつう)認証(にんしょう)ロジックを管理(かんり)する。

---

## 목차 / 目次(もくじ)

1. [세션(Session)이란?](#1-세션session이란)
2. [ApplicationController — 공통 로직](#2-applicationcontroller--공통-로직)
3. [SessionsController — 로그인/로그아웃](#3-sessionscontroller--로그인로그아웃)
4. [인증 적용 — before_action](#4-인증-적용--before_action)
5. [세션 활용 심화](#5-세션-활용-심화)
6. [전체 코드](#6-전체-코드)
7. [오늘의 실수 & 배운 점](#7-오늘의-실수--배운-점)

---

## 1. 세션(Session)이란?

HTTP는 기본적으로 무상태(stateless) — 요청이 끝나면 서버는 누가 접속했는지 잊어버립니다.  
세션은 이 문제를 해결해서 브라우저가 서버를 기억하게 합니다.

HTTPは基本的(きほんてき)にステートレス — リクエストが終(お)わるとサーバは誰(だれ)がアクセスしたか忘(わす)れます。  
セッションはこの問題(もんだい)を解決(かいけつ)します。

```
처음 접속
브라우저 → 서버: "안녕하세요"
서버 → 브라우저: "안녕하세요! 당신의 번호는 ABC123" (쿠키에 저장)

두 번째 접속
브라우저 → 서버: "안녕하세요 (번호: ABC123)"
서버: "ABC123이군요! 저번에 오셨던 분이네요" ← 기억!
```

```ruby
# 세션에 값 저장
session[:author_name] = "강홍규"

# 세션에서 값 꺼내기
session[:author_name]   # → "강홍규"

# 세션 값 삭제
session[:author_name] = nil

# 세션 전체 초기화 (로그아웃)
reset_session

# 없을 때만 초기화 — ||= 연산자
session[:visit_count] ||= 0   # nil이면 0, 값이 있으면 그대로 유지
session[:visit_count] += 1    # 1씩 증가
```

---

## 2. ApplicationController — 공통 로직

모든 컨트롤러의 부모 클래스입니다.  
여기에 메서드를 넣으면 **모든 컨트롤러 + 뷰**에서 사용 가능합니다.

すべてのコントローラの親(おや)クラスです。

```ruby
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base

  # helper_method: 뷰 파일에서도 사용 가능하게 설정
  helper_method :current_author, :logged_in?

  def current_author
    # 세션에서 작성자 이름 꺼내기 — 없으면 nil
    session[:author_name]
  end

  def logged_in?
    # current_author가 nil이 아니면 true
    current_author.present?
  end

  def require_login
    # 로그인 안 했으면 로그인 페이지로 강제 이동
    unless logged_in?
      flash[:alert] = "먼저 이름을 입력해주세요."
      redirect_to "/login"
    end
  end

end
```

---

## 3. SessionsController — 로그인/로그아웃

```ruby
# app/controllers/sessions_controller.rb
class SessionsController < ApplicationController

  def new
    # 로그인 폼 페이지 — 뷰만 렌더링
  end

  def create
    # 폼에서 넘어온 author_name 파라미터 저장
    name = params[:author_name]

    if name.present?
      # 세션에 이름 저장
      session[:author_name] = name
      flash[:notice] = "#{name}님, 환영합니다! 👋"
      redirect_to "/posts"
    else
      flash[:alert] = "이름을 입력해주세요."
      redirect_to "/login"
    end
  end

  def destroy
    # 세션 전체 초기화 → 로그아웃
    reset_session
    flash[:notice] = "로그아웃 됐습니다."
    redirect_to "/posts"
  end

end
```

```erb
<%# app/views/sessions/new.html.erb %>
<h1>이름 입력</h1>
<p>작성자로 사용할 이름을 입력하세요.</p>

<%= form_with url: "/login", method: :post do |f| %>
  <p>
    <%= f.label :author_name, "이름" %><br>
    <%= f.text_field :author_name, placeholder: "이름을 입력하세요" %>
  </p>
  <%= f.submit "시작하기" %>
<% end %>
```

---

## 4. 인증 적용 — before_action

```ruby
# config/routes.rb
get    "/login",  to: "sessions#new"
post   "/login",  to: "sessions#create"
delete "/logout", to: "sessions#destroy"
get    "/my-posts", to: "posts#my_posts"
```

```ruby
# posts_controller.rb
class PostsController < ApplicationController
  # new, create, edit, update, destroy 실행 전 로그인 확인
  before_action :require_login, only: [:new, :create, :edit, :update, :destroy]
  before_action :set_post, only: [:show, :edit, :update, :destroy]

  def new
    @post = Post.new
    # 작성자 필드를 세션 이름으로 미리 채우기
    @post.author = current_author
  end

  def my_posts
    require_login
    return if performed?   # redirect 됐으면 이후 코드 실행 안 함

    # 현재 로그인한 사람의 게시글만 조회
    @posts = Post.where(author: current_author).latest
  end

  # 나머지 액션 동일...
end
```

```ruby
# comments_controller.rb
class CommentsController < ApplicationController
  # create, destroy 실행 전 로그인 확인
  before_action :require_login, only: [:create, :destroy]

  def create
    @post    = Post.find(params[:post_id])
    @comment = @post.comments.new(comment_params)
    # 작성자를 폼 입력 대신 세션에서 자동으로 가져오기
    @comment.author = current_author

    if @comment.save
      flash[:notice] = "댓글이 등록됐습니다! 💬"
      redirect_to "/posts/#{@post.id}"
    else
      flash[:alert] = @comment.errors.full_messages.join(", ")
      redirect_to "/posts/#{@post.id}"
    end
  end

  # 나머지 동일...
end
```

---

## 5. 세션 활용 심화

### 네비게이션 바 — 로그인 상태 표시

```erb
<%# app/views/layouts/application.html.erb %>
<body>
  <nav>
    <% if logged_in? %>
      <%# 로그인 중이면 이름 + 내 글 보기 + 로그아웃 %>
      <span><%= current_author %>님</span>
      <%= link_to "내가 작성한 글", "/my-posts" %>
      <%= button_to "로그아웃", "/logout", method: :delete %>
    <% else %>
      <%# 로그아웃 상태면 로그인 링크 %>
      <%= link_to "이름 입력하기", "/login" %>
    <% end %>
  </nav>

  <% if flash[:notice] %>
    <p style="color: green"><%= flash[:notice] %></p>
  <% end %>
  <% if flash[:alert] %>
    <p style="color: red"><%= flash[:alert] %></p>
  <% end %>

  <%= yield %>
</body>
```

### 내 게시글만 보기 — `my_posts.html.erb`

```erb
<h1><%= current_author %>님의 게시물</h1>

<% if @posts.empty? %>
  <p>아직 게시물이 없습니다.</p>
<% else %>
  <%# 게시글 카드 파셜 재사용 %>
  <%= render partial: "posts/post", collection: @posts, as: :post %>
<% end %>

<a href="/posts">← 전체 목록</a>
```

### 작성자만 수정/삭제 가능 — `show.html.erb`

```erb
<%# current_author와 게시글 작성자가 같을 때만 버튼 표시 %>
<% if current_author == @post.author %>
  <a href="/posts/<%= @post.id %>/edit">수정하기</a>
  <%= button_to "삭제하기", "/posts/#{@post.id}",
      method: :delete,
      data: { turbo_confirm: "정말 삭제할까요?" } %>
<% end %>
```

### 방문 횟수 세션으로 기록

```ruby
# home_controller.rb
def index
  # nil이면 0으로 초기화, 있으면 그대로 유지
  session[:visit_count] ||= 0
  # 방문할 때마다 1씩 증가
  session[:visit_count] += 1
  # 뷰에서 쓸 수 있게 인스턴스 변수에 저장
  @visit_count = session[:visit_count]
  # ...
end
```

---

## 6. 전체 코드 구조

```
app/
├── controllers/
│   ├── application_controller.rb  ← current_author, logged_in?, require_login
│   ├── sessions_controller.rb     ← new, create, destroy
│   ├── posts_controller.rb        ← before_action :require_login 추가
│   └── comments_controller.rb     ← before_action :require_login 추가
└── views/
    ├── sessions/
    │   └── new.html.erb           ← 이름 입력 폼
    ├── posts/
    │   └── my_posts.html.erb      ← 내 게시글 목록
    └── layouts/
        └── application.html.erb   ← 네비게이션 바 추가
```

---

## 7. 오늘의 실수 & 배운 점

今日(きょう)つまずいた部分(ぶぶん)を記録(きろく)します。

| 실수                                     | 원인                 | 올바른 방법                           |
| :--------------------------------------- | :------------------- | :------------------------------------ |
| `required_login`                         | 과거형 오타          | `require_login`                       |
| `before_action required_login`           | 심볼 콜론 빠짐       | `before_action :require_login`        |
| `full_message`                           | 단수                 | `full_messages` 복수                  |
| `current_author = @post.author`          | 대입 연산자          | `current_author == @post.author` 비교 |
| `session[:author_name]` 뷰에서 직접      | MVC 원칙 위반        | `current_author` 헬퍼 메서드 사용     |
| `session[:visit_count] = 0`              | 매번 0으로 초기화    | `session[:visit_count] \|\|= 0`       |
| `render partial: "/posts/post"`          | 슬래시 불필요        | `render partial: "posts/post"`        |
| `_post.html.erb` 파일 없음               | 파일 생성 누락       | `app/views/posts/_post.html.erb` 생성 |
| `Post.where(current_author == @post.id)` | where 조건 문법 오류 | `Post.where(author: current_author)`  |

> **오늘의 핵심 기억:**
>
> 1. `session[]` — 브라우저가 닫히면 사라지는 임시 저장소
> 2. `helper_method` — 컨트롤러 메서드를 뷰에서도 사용 가능하게 설정
> 3. `||=` — nil일 때만 초기화, 값이 있으면 그대로 유지
> 4. 뷰에서 `session` 직접 접근 ❌ → `current_author` 헬퍼 메서드 ✅

---

## 🔑 한눈에 보는 키워드

| 키워드                           | 설명                                |
| :------------------------------- | :---------------------------------- |
| `session[:키]`                   | 세션에 값 저장/조회                 |
| `reset_session`                  | 세션 전체 초기화 (로그아웃)         |
| `session[:키] \|\|= 0`           | nil일 때만 초기값 설정              |
| `helper_method :메서드명`        | 컨트롤러 메서드를 뷰에서도 사용     |
| `current_author`                 | 세션에서 작성자 이름 반환           |
| `logged_in?`                     | 로그인 여부 true/false              |
| `require_login`                  | 비로그인 시 /login 으로 이동        |
| `return if performed?`           | redirect 됐으면 이후 코드 실행 중단 |
| `current_author == @post.author` | 작성자 본인 확인                    |

---

[← 목차로 돌아가기 / 目次(もくじ)に戻(もど)る](../README.md)
