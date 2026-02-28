# 📅 Day 8 — 파셜(Partial) & N+1 문제 해결 & 헬퍼 메서드

**날짜 / 日付(ひづけ):** 2026.02.28  
**학습 시간 / 学習時間(がくしゅうじかん):** 2시간 / 2時間(じかん)  
**한 줄 목표:** 파셜로 뷰 코드를 재사용하고, N+1 문제를 해결하고, 헬퍼 메서드로 로직을 분리한다.  
**一言目標(ひとこともくひょう):** パーシャルでビューを再利用(さいりよう)し、N+1問題(もんだい)を解決(かいけつ)し、ヘルパーメソッドでロジックを分離(ぶんり)する。

---

## 목차 / 目次(もくじ)

1. [파셜(Partial)이란?](#1-파셜partial이란)
2. [N+1 문제와 includes](#2-n1-문제와-includes)
3. [헬퍼 메서드](#3-헬퍼-메서드)
4. [전체 코드](#4-전체-코드)
5. [오늘의 실수 & 배운 점](#5-오늘의-실수--배운-점)

---

## 1. 파셜(Partial)이란?

반복되는 뷰 코드를 별도 파일로 분리해서 재사용하는 기능입니다.  
파셜 파일은 반드시 `_` (언더바)로 시작합니다.

繰(く)り返(かえ)されるビューコードを別(べつ)ファイルに分(わ)けて再利用(さいりよう)する機能(きのう)です。  
パーシャルファイルは必(かなら)ず`_`(アンダーバー)で始(はじ)まります。

```
app/views/posts/
├── _form.html.erb     ← 게시글 폼 파셜
├── _post.html.erb     ← 게시글 카드 파셜
├── index.html.erb
├── new.html.erb
├── edit.html.erb
└── show.html.erb

app/views/comments/
└── _comment.html.erb  ← 댓글 파셜

app/views/shared/
└── _errors.html.erb   ← 에러 메시지 파셜 (어디서든 재사용!)
```

### 파셜 호출 방법

```erb
<%# 단순 호출 %>
<%= render "form" %>

<%# locals로 변수 전달 %>
<%= render "form", post: @post, url: "/posts", method: :post, submit_text: "저장" %>

<%# collection 파셜 — 배열을 자동으로 순회 %>
<%= render partial: "comments/comment", collection: @post.comments, as: :comment %>
<%= render partial: "posts/post", collection: @posts, as: :post %>
```

> **collection 파셜의 장점:**  
> `each` 루프 없이 한 줄로 배열 전체를 렌더링할 수 있어요.  
> `コレクションパーシャル`の利点(りてん)：`each`ループなしで一行(いちぎょう)で配列全体(はいれつぜんたい)をレンダリングできます。

---

### 게시글 폼 파셜 — `_form.html.erb`

```erb
<%# app/views/posts/_form.html.erb %>

<%# 에러 메시지 파셜 호출 — shared/_errors 재사용 %>
<%= render "shared/errors", object: post %>

<%= form_with model: post, url: url, method: method do |f| %>
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
  <%= f.submit submit_text %>
<% end %>
```

### new / edit 파셜로 교체

```erb
<%# new.html.erb %>
<h1>새 게시글 작성</h1>
<%= render "form", post: @post, url: "/posts", method: :post, submit_text: "저장" %>
<a href="/posts">← 목록으로</a>

<%# edit.html.erb %>
<h1>게시글 수정</h1>
<%= render "form", post: @post, url: "/posts/#{@post.id}", method: :patch, submit_text: "수정하기" %>
<a href="/posts/<%= @post.id %>">← 취소</a>
```

### 에러 메시지 파셜 — `shared/_errors.html.erb`

```erb
<%# app/views/shared/_errors.html.erb %>
<%# object는 밖에서 전달받아서 어디서든 재사용 가능 %>
<% if object.errors.full_messages.any? %>
  <% object.errors.full_messages.each do |message| %>
    <p style="color: red"><%= message %></p>
  <% end %>
<% end %>
```

### 댓글 파셜 — `comments/_comment.html.erb`

```erb
<%# app/views/comments/_comment.html.erb %>
<div>
  <p>작성자: <%= comment.author %></p>
  <p><%= comment.body %></p>
  <small><%= comment.created_at.strftime("%Y.%m.%d %H:%M") %></small>
  <%= button_to "삭제",
      "/posts/#{comment.post_id}/comments/#{comment.id}",
      method: :delete,
      data: { turbo_confirm: "댓글을 삭제할까요?" } %>
</div>
<hr>
```

```erb
<%# show.html.erb — each 루프 대신 collection 파셜 한 줄로! %>
<%= render partial: "comments/comment", collection: @post.comments, as: :comment %>
```

### 게시글 카드 파셜 — `posts/_post.html.erb`

```erb
<%# app/views/posts/_post.html.erb %>
<div>
  <span><%= category_label(post.category) %></span>
  <%= link_to post.title, "/posts/#{post.id}" %>
  <small>(댓글 <%= post.comments.count %>개)</small>
  <small><%= formatted_date(post.created_at) %></small>
</div>
```

```erb
<%# index.html.erb — each 루프를 파셜 한 줄로 교체 %>
<%= render partial: "posts/post", collection: @posts, as: :post %>
```

---

## 2. N+1 문제와 includes

게시글 목록에서 댓글 수를 출력할 때 N+1 문제가 발생합니다.

```ruby
# ❌ N+1 문제 — 게시글 10개면 쿼리 11번 실행
@posts = Post.all
# SELECT * FROM posts                          → 1번
# SELECT COUNT(*) FROM comments WHERE post_id = 1  → 1번
# SELECT COUNT(*) FROM comments WHERE post_id = 2  → 1번
# ...                                          → N번
# 총 N+1번!

# ✅ includes로 해결 — 쿼리 2번으로 끝!
@posts = Post.includes(:comments).latest
# SELECT * FROM posts                          → 1번
# SELECT * FROM comments WHERE post_id IN (1,2,3...) → 1번
# 총 2번!
```

```ruby
# posts_controller.rb
def index
  @posts = Post.includes(:comments)  # ← includes 추가!
               .latest
               .by_category(params[:category])

  if params[:search].present?
    @posts = @posts.where("title LIKE ?", "%#{params[:search]}%")
  end

  @posts = @posts.page(params[:page]).per(3)
end
```

---

## 3. 헬퍼 메서드

뷰의 복잡한 로직을 헬퍼 메서드로 분리합니다.  
`app/helpers/` 폴더의 파일은 뷰에서 자동으로 사용 가능합니다.

ビューの複雑(ふくざつ)なロジックをヘルパーメソッドに分離(ぶんり)します。

```ruby
# app/helpers/posts_helper.rb
module PostsHelper

  # 카테고리를 이모지 + 한글로 변환
  def category_label(category)
    case category
    when "ruby"  then "🔴 Ruby"
    when "rails" then "🟣 Rails"
    when "other" then "⚪ 기타"
    else "❓ 미분류"
    end
  end

  # 조회수를 이모지 + 단위 붙여서 출력
  def view_count_label(count)
    "👁 #{count}회"
  end

  # 날짜를 한국어 형식으로 변환
  # 기대 출력: "2026년 02월 28일"
  def formatted_date(datetime)
    datetime.strftime("%Y년 %m월 %d일")
  end

end
```

```erb
<%# 뷰에서 사용 %>
<span><%= category_label(post.category) %></span>
<p><%= view_count_label(@post.view_count) %></p>
<small><%= formatted_date(post.created_at) %></small>
```

---

## 4. 전체 코드 구조

```
app/
├── controllers/
│   └── posts_controller.rb   ← includes 추가
├── helpers/
│   └── posts_helper.rb       ← 헬퍼 메서드 3개
└── views/
    ├── posts/
    │   ├── _form.html.erb    ← 폼 파셜
    │   ├── _post.html.erb    ← 게시글 카드 파셜
    │   ├── index.html.erb    ← collection 파셜 사용
    │   ├── new.html.erb      ← render "form" 한 줄
    │   ├── edit.html.erb     ← render "form" 한 줄
    │   └── show.html.erb     ← collection 파셜 사용
    ├── comments/
    │   └── _comment.html.erb ← 댓글 파셜
    └── shared/
        └── _errors.html.erb  ← 에러 메시지 파셜
```

---

## 5. 오늘의 실수 & 배운 점

今日(きょう)つまずいた部分(ぶぶん)を記録(きろく)します。

| 실수                                   | 원인                                  | 올바른 방법                            |
| :------------------------------------- | :------------------------------------ | :------------------------------------- |
| `view_count(...)`                      | 헬퍼 이름 틀림                        | `view_count_label(...)`                |
| each 루프 안에 collection 파셜         | 댓글이 중복 출력됨                    | each 루프 제거 후 파셜만 사용          |
| `as :comment`                          | 콜론 위치 오류                        | `as: :comment`                         |
| `created_at.strftime(...)`             | 파라미터 대신 존재하지 않는 변수 사용 | `datetime.strftime(...)`               |
| `formated_date`                        | 오타 (t 하나)                         | `formatted_date` (t 두 개)             |
| `_errors.html.erb`에서 `@post` 고정    | 재사용 불가                           | `object`로 받아서 범용적으로 사용      |
| `comments/_comment.html.erb` 파일 없음 | 파일을 만들지 않음                    | `app/views/comments/` 폴더 + 파일 생성 |

> **오늘의 핵심 기억:**
>
> 1. 파셜 파일은 반드시 `_` 로 시작, `render "파셜명"` 으로 호출
> 2. collection 파셜 — `each` 루프 없이 배열 전체 렌더링 가능
> 3. `includes(:comments)` 한 줄로 N+1 문제 해결
> 4. 헬퍼 메서드는 파라미터를 받아야 재사용 가능

---

## 🔑 한눈에 보는 키워드

| 키워드                                                 | 설명                                  |
| :----------------------------------------------------- | :------------------------------------ |
| `_파일명.html.erb`                                     | 파셜 파일 (언더바로 시작)             |
| `render "파셜명"`                                      | 파셜 호출                             |
| `render "파셜명", 변수: 값`                            | locals로 변수 전달                    |
| `render partial: "...", collection: 배열, as: :변수명` | 배열 전체를 파셜로 렌더링             |
| `Post.includes(:comments)`                             | N+1 문제 해결 — 연관 데이터 미리 로드 |
| `app/helpers/posts_helper.rb`                          | 뷰 로직을 분리하는 헬퍼 파일          |
| `datetime.strftime("형식")`                            | 날짜 형식 변환                        |

---

[← 목차로 돌아가기 / 目次(もくじ)に戻(もど)る](../README.md)
