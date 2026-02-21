# 📅 Day 3 — Rails MVC 흐름 이해

**날짜 / 日付(ひづけ):** 2026.02.21  
**학습 시간 / 学習時間(がくしゅうじかん):** 2시간 / 2時間(じかん)  
**한 줄 목표:** Rails의 MVC 흐름을 실제로 만들면서, 요청이 화면까지 가는 과정을 몸으로 익힌다.  
**一言目標(ひとこともくひょう):** RailsのMVCの流(なが)れを実際(じっさい)に作(つく)りながら、リクエストが画面(がめん)に届(とど)くまでの流(なが)れを体(からだ)で覚(おぼ)える。

---

## 목차 / 目次(もくじ)

1. [MVC 패턴이란?](#1-mvc-패턴이란)
2. [Rails 프로젝트 구조](#2-rails-프로젝트-구조)
3. [요청 한 번의 전체 흐름](#3-요청-한-번의-전체-흐름)
4. [routes.rb — URL 지도](#4-routesrb--url-지도)
5. [Controller — 요청 처리](#5-controller--요청-처리)
6. [View — 화면 출력](#6-view--화면-출력)
7. [params — URL에서 값 꺼내기](#7-params--url에서-값-꺼내기)
8. [오늘의 실수 & 배운 점](#8-오늘의-실수--배운-점)

---

## 1. MVC 패턴이란?

MVC는 코드를 역할별로 나누는 구조입니다. 레스토랑에 비유하면 이해하기 쉽습니다.

MVCはコードを役割(やくわり)ごとに分(わ)ける構造(こうぞう)です。レストランにたとえるとわかりやすいです。

| 레스토랑 |   Rails    | 역할                               |
| :------: | :--------: | :--------------------------------- |
|   손님   |  브라우저  | URL로 요청을 보냄                  |
|  웨이터  | Controller | 요청을 받아서 처리하고 결과를 전달 |
|   주방   |   Model    | DB에서 데이터를 꺼내서 처리        |
| 플레이팅 |    View    | 결과를 HTML로 예쁘게 출력          |

```
브라우저 → routes.rb → Controller → Model → DB
                            ↓
                           View → 브라우저
```

---

## 2. Rails 프로젝트 구조

`rails new` 로 프로젝트를 만들면 아래 구조가 생깁니다.

`rails new`でプロジェクトを作(つく)ると、下(した)の構造(こうぞう)ができます。

```
my_app/
├── app/
│   ├── controllers/   ← 요청을 받아서 처리
│   ├── models/        ← 데이터베이스와 대화
│   └── views/         ← 화면에 보여줄 HTML
├── config/
│   └── routes.rb      ← URL을 어디로 연결할지 지도
├── db/
│   └── schema.rb      ← 데이터베이스 구조
└── Gemfile            ← 사용할 라이브러리 목록
```

---

## 3. 요청 한 번의 전체 흐름

`http://localhost:3000/posts` 에 접속했을 때 일어나는 일입니다.

`http://localhost:3000/posts`にアクセスしたときに起(お)きることです。

```
1. 브라우저가 GET /posts 요청
         ↓
2. routes.rb: "posts#index 로 가라"
         ↓
3. PostsController#index 실행
         ↓
4. @posts 에 데이터를 담아서
         ↓
5. views/posts/index.html.erb 렌더링
         ↓
6. 브라우저에 HTML 반환
```

> **핵심:** 컨트롤러에서 `@변수`에 담은 데이터는 **자동으로 뷰에서 사용 가능**합니다.  
> コントローラで`@変数(へんすう)`に入(い)れたデータは、**自動(じどう)でビューから使(つか)えます**。

---

## 4. routes.rb — URL 지도

모든 URL 요청은 여기서 어느 컨트롤러로 갈지 결정됩니다.

すべてのURLリクエストはここで、どのコントローラに行(い)くか決(き)まります。

```ruby
# config/routes.rb
Rails.application.routes.draw do
  root "home#index"               # GET /        → HomeController#index
  get "/about", to: "home#about"  # GET /about   → HomeController#about
  get "/posts", to: "posts#index" # GET /posts   → PostsController#index
  get "/posts/:id", to: "posts#show" # GET /posts/1 → PostsController#show
end
```

routes 확인 명령어:

```bash
rails routes
```

---

## 5. Controller — 요청 처리

컨트롤러는 데이터를 준비해서 `@변수`에 담고, 뷰에 넘겨줍니다.  
**컨트롤러와 뷰는 1:1 연결** — `PostsController`의 `@변수`는 `posts/` 뷰에서만 사용 가능합니다.

コントローラはデータを準備(じゅんび)して`@変数(へんすう)`に入(い)れ、ビューに渡(わた)します。  
**コントローラとビューは1対1(いちたいいち)の関係(かんけい)** — `PostsController`の`@変数(へんすう)`は`posts/`のビューでしか使(つか)えません。

```ruby
# app/controllers/posts_controller.rb
class PostsController < ApplicationController
  def index
    @posts = [
      { id: 1, title: "Day 1 - 클래스와 모듈",        date: "2026.02.19" },
      { id: 2, title: "Day 2 - 블록과 이터레이터",     date: "2026.02.20" },
      { id: 3, title: "Day 3 - MVC 흐름 이해",        date: "2026.02.21" },
      { id: 4, title: "Day 4 - 영단어 공부하기",       date: "2026.02.25" },
      { id: 5, title: "Day 5 - 일본어 JLPT N2 공부하기", date: "2026.02.27" }
    ]
  end

  def show
    @posts = [
      { id: 1, title: "Day 1 - 클래스와 모듈",        date: "2026.02.19" },
      { id: 2, title: "Day 2 - 블록과 이터레이터",     date: "2026.02.20" },
      { id: 3, title: "Day 3 - MVC 흐름 이해",        date: "2026.02.21" },
      { id: 4, title: "Day 4 - 영단어 공부하기",       date: "2026.02.25" },
      { id: 5, title: "Day 5 - 일본어 JLPT N2 공부하기", date: "2026.02.27" }
    ]
    # params[:id] 로 URL의 값을 꺼내서 숫자로 변환
    id = params[:id].to_i
    # find로 해당 id의 게시글 하나만 찾기
    @post = @posts.find { |post| post[:id] == id }
  end
end
```

> **왜 @posts를 show에서 또 선언하나요?**  
> Rails는 요청마다 컨트롤러 인스턴스를 새로 만들고, 메서드 하나만 실행하고 버립니다.  
> `index`에서 만든 `@posts`는 `show`가 실행될 때 이미 사라져 있어요.  
> → Day 7에서 `before_action`으로 이 중복을 깔끔하게 해결합니다!

---

## 6. View — 화면 출력

ERB(Embedded Ruby) — HTML 안에 Ruby 코드를 쓸 수 있는 템플릿입니다.

ERB(Embedded Ruby) — HTMLの中(なか)にRubyのコードを書(か)けるテンプレートです。

```erb
<%# <% %>  : Ruby 실행만, 화면에 출력 안 함 %>
<%# <%= %> : Ruby 실행 + 결과를 화면에 출력 %>

<!-- app/views/posts/index.html.erb -->
<h1>게시글 목록</h1>

<% @posts.each do |post| %>
  <p>
    [<%= post[:id] %>]
    <a href="/posts/<%= post[:id] %>"><%= post[:title] %></a>
    <small>(<%= post[:date] %>)</small>
  </p>
<% end %>

<a href="/">홈으로</a>
```

```erb
<!-- app/views/posts/show.html.erb -->
<h1><%= @post[:title] %></h1>
<p>날짜: <%= @post[:date] %></p>

<a href="/posts">← 목록으로</a>
```

---

## 7. params — URL에서 값 꺼내기

URL의 `:id` 부분은 `params[:id]`로 꺼낼 수 있습니다.  
단, 항상 **문자열(String)** 로 들어오기 때문에 `.to_i`로 숫자로 변환해야 합니다.

URLの`:id`の部分(ぶぶん)は`params[:id]`で取(と)り出(だ)せます。  
ただし、常(つね)に**文字列(もじれつ)**として入(はい)ってくるので、`.to_i`で数字(すうじ)に変換(へんかん)する必要(ひつよう)があります。

```ruby
# GET /posts/2 로 접속하면
params[:id]        # → "2"  (문자열!)
params[:id].to_i   # → 2    (숫자!)

# 주의: "2" == 2 는 false
# 그래서 .to_i 변환이 반드시 필요
```

---

## 8. 오늘의 실수 & 배운 점

今日(きょう)つまずいた部分(ぶぶん)を記録(きろく)します。

| 실수                                                   | 원인                                          | 올바른 방법                             |
| :----------------------------------------------------- | :-------------------------------------------- | :-------------------------------------- |
| `post[:id] == @posts[:id]`                             | `@posts`는 배열 전체, `[:id]` 없음            | `post[:id] == id` (변수 사용)           |
| `"총 @count 개"`                                       | 문자열 안에 `@변수` 그대로 쓰면 텍스트로 출력 | `"총 #{@count} 개"`                     |
| `HomeController`에서 `PostsController`의 `@count` 사용 | 컨트롤러와 뷰는 1:1 연결                      | `HomeController`에서 직접 `@count` 선언 |
| `<a href="/posts/#{post[:id]}"></a>`                   | 태그 사이에 텍스트 없어서 링크가 안 보임      | 태그 사이에 `<%= post[:title] %>` 추가  |

> **오늘의 핵심 기억:**
>
> 1. 컨트롤러의 `@변수`는 자기 뷰에서만 사용 가능
> 2. `params[:id]`는 항상 문자열 → `.to_i`로 변환 필수
> 3. `<% %>` 실행만 / `<%= %>` 출력까지

---

## 🔑 한눈에 보는 키워드

| 키워드                               | 설명                            |
| :----------------------------------- | :------------------------------ |
| `rails new 앱이름`                   | Rails 프로젝트 생성             |
| `rails generate controller 이름`     | 컨트롤러 파일 자동 생성         |
| `rails server`                       | 로컬 서버 실행 (localhost:3000) |
| `rails routes`                       | 등록된 모든 라우트 확인         |
| `root "컨트롤러#메서드"`             | 루트 경로(`/`) 설정             |
| `get "/경로", to: "컨트롤러#메서드"` | GET 요청 라우트 설정            |
| `params[:id]`                        | URL의 `:id` 값을 꺼냄 (문자열)  |
| `.to_i`                              | 문자열을 정수로 변환            |
| `<% %>`                              | ERB: Ruby 실행만 (출력 없음)    |
| `<%= %>`                             | ERB: Ruby 실행 + 화면 출력      |

---

[← 목차로 돌아가기 / 目次(もくじ)に戻(もど)る](../README.md)
