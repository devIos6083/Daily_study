# 📅 Day 7 — Rails 심화 (카테고리, 페이지네이션, 조회수, 검색)

**날짜 / 日付(ひづけ):** 2026.02.26  
**학습 시간 / 学習時間(がくしゅうじかん):** 3시간 / 3時間(じかん)  
**한 줄 목표:** enum, scope, kaminari를 활용해서 게시판을 실무 수준으로 업그레이드한다.  
**一言目標(ひとこともくひょう):** enum・scope・kaminariを使(つか)って掲示板(けいじばん)を実務(じつむ)レベルにアップグレードする。

---

## 목차 / 目次(もくじ)

1. [link_to 헬퍼](#1-link_to-헬퍼)
2. [enum — 카테고리 관리](#2-enum--카테고리-관리)
3. [scope — 쿼리에 이름 붙이기](#3-scope--쿼리에-이름-붙이기)
4. [kaminari — 페이지네이션](#4-kaminari--페이지네이션)
5. [조회수 기능](#5-조회수-기능)
6. [연관 게시글 추천](#6-연관-게시글-추천)
7. [폼 에러 메시지 출력](#7-폼-에러-메시지-출력)
8. [전체 코드](#8-전체-코드)
9. [오늘의 실수 & 배운 점](#9-오늘의-실수--배운-점)

---

## 1. link_to 헬퍼

HTML `<a>` 태그 대신 Rails 방식으로 링크를 만듭니다.

HTMLの`<a>`タグの代(か)わりにRailsの方法(ほうほう)でリンクを作(つく)ります。

```erb
<%# ❌ 기존 방식 %>
<a href="/posts/<%= post.id %>"><%= post.title %></a>

<%# ✅ Rails 방식 %>
<%= link_to post.title, "/posts/#{post.id}" %>

<%# 클래스 추가도 가능 %>
<%= link_to "새 게시글", "/posts/new", class: "btn" %>
```

---

## 2. enum — 카테고리 관리

숫자를 이름으로 관리하는 Rails 기능입니다.  
DB에는 숫자로 저장되지만 코드에서는 이름으로 사용합니다.

数字(すうじ)を名前(なまえ)で管理(かんり)するRailsの機能(きのう)です。

```ruby
# app/models/post.rb
enum :category, { ruby: 0, rails: 1, other: 2 }
```

```ruby
# 자동 생성되는 메서드들
post.ruby?          # → true / false
post.rails?         # → true / false
post.category       # → "ruby"

# 카테고리별 조회
Post.ruby.count     # ruby 카테고리 게시글 수
Post.rails.count    # rails 카테고리 게시글 수
Post.other.count    # other 카테고리 게시글 수
```

폼에서 드롭다운으로 선택:

```erb
<%= f.select :category, [["Ruby", "ruby"], ["Rails", "rails"], ["기타", "other"]] %>
```

---

## 3. scope — 쿼리에 이름 붙이기

자주 쓰는 쿼리에 이름을 붙여서 재사용합니다.  
체이닝으로 조합하면 더 강력해집니다.

よく使(つか)うクエリに名前(なまえ)をつけて再利用(さいりよう)します。

```ruby
# app/models/post.rb
scope :latest,      -> { order(created_at: :desc) }
scope :by_category, ->(cat) { where(category: cat) if cat.present? }

# 체이닝으로 조합
Post.latest.by_category("ruby")   # ruby 카테고리를 최신순으로
```

---

## 4. kaminari — 페이지네이션

게시글이 많을 때 페이지를 나눠서 보여주는 gem입니다.

記事(きじ)が多(おお)いときにページを分(わ)けて表示(ひょうじ)するgemです。

```ruby
# Gemfile에 추가 후 bundle install + 서버 재시작 필요
gem 'kaminari'
```

```ruby
# 컨트롤러 — 한 페이지에 3개씩
@posts = Post.latest.page(params[:page]).per(3)
```

```erb
<%# 뷰 — 페이지 번호 자동 출력 %>
<%= paginate @posts %>
```

> **주의:** `gem` 추가 후 반드시 `bundle install` + **서버 재시작** 필요!

---

## 5. 조회수 기능

```bash
# 마이그레이션으로 view_count 컬럼 추가
rails generate migration AddViewCountToPosts view_count:integer
rails db:migrate

# 기본값 0으로 설정하는 마이그레이션
rails generate migration ChangeViewCountDefaultInPosts
```

```ruby
# db/migrate/날짜_change_view_count_default_in_posts.rb
class ChangeViewCountDefaultInPosts < ActiveRecord::Migration[7.2]
  def change
    # view_count 기본값을 nil → 0으로 변경
    change_column_default :posts, :view_count, from: nil, to: 0
  end
end
```

```ruby
# posts_controller.rb — show 액션
def show
  # 페이지를 볼 때마다 view_count 1씩 증가
  @post.increment!(:view_count)
end
```

```erb
<%# 조회수 출력 %>
<p>조회수: <%= @post.view_count %>회</p>
```

> **핵심:** `view_count`의 기본값이 `nil`이면 `increment!` 실행 시 에러 발생!  
> 반드시 기본값을 `0`으로 설정해야 합니다.

---

## 6. 연관 게시글 추천

```ruby
# posts_controller.rb — show 액션
def show
  # 같은 카테고리의 다른 게시글 3개 (자기 자신 제외, 최신순)
  @related_posts = Post.where(category: @post.category)
                       .where.not(id: @post.id)
                       .order(created_at: :desc)
                       .limit(3)
  @post.increment!(:view_count)
end
```

```erb
<%# show.html.erb %>
<h3>같은 카테고리 게시물</h3>
<% @related_posts.each do |post| %>
  <p><%= link_to post.title, "/posts/#{post.id}" %></p>
<% end %>
```

---

## 7. 폼 에러 메시지 출력

저장 실패 시 어떤 조건이 틀렸는지 화면에 표시합니다.

保存(ほぞん)失敗(しっぱい)時(じ)にどの条件(じょうけん)が間違(まちが)っているか画面(がめん)に表示(ひょうじ)します。

```erb
<%# new.html.erb, edit.html.erb 상단에 추가 %>
<%# @post에 에러가 있을 때만 표시 %>
<% if @post.errors.full_messages.any? %>
  <% @post.errors.full_messages.each do |message| %>
    <p style="color: red"><%= message %></p>
  <% end %>
<% end %>
```

> **핵심:** 에러는 `@post.errors.full_messages`에 배열로 담겨 있습니다.  
> `render :new` 로 폼을 다시 그릴 때 `@post` 객체가 유지되기 때문에 사용 가능합니다.

---

## 8. 전체 코드

### post.rb

```ruby
class Post < ApplicationRecord
  has_many :comments, dependent: :destroy

  validates :title,   presence: true
  validates :content, presence: true, length: { minimum: 10 }
  validates :author,  presence: true, length: { maximum: 20 }

  # 0 → ruby, 1 → rails, 2 → other
  enum :category, { ruby: 0, rails: 1, other: 2 }

  # 자주 쓰는 쿼리에 이름 붙이기
  scope :latest,      -> { order(created_at: :desc) }
  scope :by_category, ->(cat) { where(category: cat) if cat.present? }
end
```

### posts_controller.rb

```ruby
class PostsController < ApplicationController
  before_action :set_post, only: [:show, :edit, :update, :destroy]

  def index
    # 카테고리 + 검색 + 정렬 scope 체이닝
    @posts = Post.latest.by_category(params[:category])

    if params[:search].present?
      @posts = @posts.where("title LIKE ?", "%#{params[:search]}%")
    end

    # 한 페이지에 3개씩 페이지네이션
    @posts = @posts.page(params[:page]).per(3)
  end

  def show
    # 같은 카테고리 연관 게시글 3개
    @related_posts = Post.where(category: @post.category)
                         .where.not(id: @post.id)
                         .order(created_at: :desc)
                         .limit(3)
    # 조회수 1 증가
    @post.increment!(:view_count)
  end

  def new
    @post = Post.new
  end

  def create
    @post = Post.new(post_params)
    if @post.save
      flash[:notice] = "게시글이 저장됐습니다 ✅"
      redirect_to "/posts/#{@post.id}"
    else
      render :new
    end
  end

  def edit
  end

  def update
    if @post.update(post_params)
      flash[:notice] = "게시물이 수정됐습니다 ✅"
      redirect_to "/posts/#{@post.id}"
    else
      render :edit
    end
  end

  def destroy
    @post.destroy
    flash[:notice] = "게시물이 삭제됐습니다 🥹"
    redirect_to "/posts"
  end

  private

  def set_post
    @post = Post.find(params[:id])
  end

  def post_params
    params.require(:post).permit(:title, :content, :author, :category)
  end
end
```

### home_controller.rb

```ruby
class HomeController < ApplicationController
  def index
    @title        = "나의 Ruby 학습 블로그"
    @message      = "매일 2시간, 꾸준히 기록합니다"
    @count        = Post.count
    @recent_posts = Post.latest.limit(3)

    # enum 덕분에 카테고리별 count가 한 줄로!
    @ruby_count  = Post.ruby.count
    @rails_count = Post.rails.count
    @other_count = Post.other.count
  end
end
```

### index.html.erb

```erb
<h1>게시글 목록</h1>

<%# 카테고리 필터 링크 %>
<div>
  <%= link_to "전체",  "/posts" %> |
  <%= link_to "Ruby",  "/posts?category=ruby" %> |
  <%= link_to "Rails", "/posts?category=rails" %> |
  <%= link_to "기타",  "/posts?category=other" %>
</div>

<%# 검색 폼 %>
<%= form_with url: "/posts", method: :get do |f| %>
  <%= f.text_field :search, placeholder: "검색어를 입력하세요", value: params[:search] %>
  <%= f.submit "검색" %>
<% end %>

<%= link_to "새 게시글 작성", "/posts/new" %>

<% if @posts.empty? %>
  <p>아직 게시물이 없습니다.</p>
<% else %>
  <% @posts.each do |post| %>
    <p>
      <span>[<%= post.category %>]</span>
      <%= link_to post.title, "/posts/#{post.id}" %>
      <small>(댓글 <%= post.comments.count %>개)</small>
      <small>(<%= post.created_at.strftime("%Y.%m.%d") %>)</small>
    </p>
  <% end %>
<% end %>

<%# 페이지 번호 자동 출력 %>
<%= paginate @posts %>

<a href="/">홈으로</a>
```

### new.html.erb

```erb
<h1>새 게시글 작성</h1>

<%# 에러 메시지 — 저장 실패 시 빨간색으로 출력 %>
<% if @post.errors.full_messages.any? %>
  <% @post.errors.full_messages.each do |message| %>
    <p style="color: red"><%= message %></p>
  <% end %>
<% end %>

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
  <p>
    <%= f.label :category, "카테고리" %><br>
    <%= f.select :category, [["Ruby", "ruby"], ["Rails", "rails"], ["기타", "other"]] %>
  </p>
  <%= f.submit "저장" %>
<% end %>

<a href="/posts">← 목록으로</a>
```

---

## 9. 오늘의 실수 & 배운 점

今日(きょう)つまずいた部分(ぶぶん)を記録(きろく)します。

| 실수                           | 원인                                     | 올바른 방법                                  |
| :----------------------------- | :--------------------------------------- | :------------------------------------------- |
| `gem 'kaminari'` 터미널에 입력 | gem은 Gemfile에 추가 후 `bundle install` | Gemfile → `bundle install` → 서버 재시작     |
| `paginate @posts` 에러         | 컨트롤러에서 `.page().per()` 누락        | `@posts = @posts.page(params[:page]).per(3)` |
| `view_count: nil` 에러         | 기본값 미설정으로 `nil + 1` 불가         | 마이그레이션으로 기본값 0 설정               |
| `redirect_to = "/posts/..."`   | `=` 때문에 변수 대입으로 인식            | `redirect_to "/posts/..."`                   |
| `@posts.errors`                | 목록 배열에는 errors 없음                | `@post.errors` 단수                          |
| `order(category: :desc)`       | category는 정렬 기준이 아님              | `order(created_at: :desc)`                   |
| `Post.category.ruby.count`     | category는 메서드 아님                   | `Post.ruby.count`                            |

> **오늘의 핵심 기억:**
>
> 1. `enum` 선언 → `Post.ruby.count` 처럼 카테고리 이름이 바로 스코프가 됨
> 2. `increment!` 는 DB 값이 `nil` 이면 에러 — 기본값 `0` 필수
> 3. `render :new` 다음 줄은 실행되지 않음 — 에러 메시지는 뷰에서 출력
> 4. gem 추가 후 반드시 서버 재시작!

---

## 🔑 한눈에 보는 키워드

| 키워드                          | 설명                                     |
| :------------------------------ | :--------------------------------------- |
| `link_to "텍스트", "경로"`      | HTML `<a>` 태그 대신 사용하는 Rails 헬퍼 |
| `enum :필드, { 이름: 숫자 }`    | 숫자를 이름으로 관리                     |
| `Post.ruby`                     | enum으로 자동 생성되는 카테고리 스코프   |
| `scope :이름, -> { 쿼리 }`      | 자주 쓰는 쿼리에 이름 붙이기             |
| `gem 'kaminari'`                | 페이지네이션 gem (Gemfile에 추가)        |
| `.page(params[:page]).per(N)`   | 현재 페이지 / 한 페이지당 개수           |
| `<%= paginate @posts %>`        | 페이지 번호 자동 출력                    |
| `@post.increment!(:view_count)` | DB 컬럼 값을 1 증가                      |
| `where.not(id: @post.id)`       | 특정 id 제외하고 조회                    |
| `@post.errors.full_messages`    | 저장 실패 시 에러 메시지 배열            |
| `change_column_default`         | 마이그레이션에서 기본값 변경             |

---

[← 목차로 돌아가기 / 目次(もくじ)に戻(もど)る](../README.md)
