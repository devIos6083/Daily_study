# 📅 Day 24 — 나만의 Rails 프로젝트 시작 (Dev Portfolio)

**날짜 / 日付(ひづけ):** 2026.03.15  
**학습 시간 / 学習時間(がくしゅうじかん):** 자율 / 自律(じりつ)  
**한 줄 목표:** 부트캠프에서 배운 Rails MVC를 활용해 개인 포트폴리오 앱의 라우팅·컨트롤러·뷰 기초를 직접 설계한다.  
**一言目標(ひとこともくひょう):** ブートキャンプで学(まな)んだRails MVCを活(い)かして、個人(こじん)ポートフォリオアプリのルーティング・コントローラー・ビューの基礎(きそ)を自力(じりき)で設計(せっけい)する。

---

## 프로젝트 개요 / プロジェクト概要(がいよう)

| 항목       | 내용                                              |
| :--------- | :------------------------------------------------ |
| 프로젝트명 | **Dev Portfolio**                                 |
| 목적       | 엔지니어의 포트폴리오를 공개·관리하는 웹 앱       |
| 현재 구현  | 홈·About·포트폴리오 페이지 + Projects CRUD 라우팅 |
| GitHub     | `devIos6083`                                      |

---

## 목차 / 目次(もくじ)

1. [라우팅 설계](#1-라우팅-설계)
2. [레이아웃 — application.html.erb](#2-레이아웃--applicationhtmlerb)
3. [HomeController — 루트 페이지](#3-homecontroller--루트-페이지)
4. [PagesController — About 페이지](#4-pagescontroller--about-페이지)
5. [PortfoliosController — 포트폴리오 페이지](#5-portfolioscontroller--포트폴리오-페이지)
6. [현재 앱 구조 정리](#6-현재-앱-구조-정리)

---

## 1. 라우팅 설계

**URL과 컨트롤러#액션을 연결하는 지도 역할**.  
`config/routes.rb` 에 정의하며, 요청이 들어오면 Rails가 이 파일을 보고  
"어떤 컨트롤러의 어떤 메서드를 실행할지" 결정한다.

**URLとコントローラー#アクションを結(むす)ぶ地図(ちず)の役割(やくわり)**。

```ruby
# config/routes.rb
Rails.application.routes.draw do

  # root: 최상위 경로 (/)  → HomeController의 index 액션
  root "home#index"

  # get: GET 요청을 특정 경로에 연결
  # "/about" → PagesController의 about 액션
  # → about_path 헬퍼가 자동 생성됨
  get "/about", to: "pages#about"

  # :username — URL 파라미터 (동적 세그먼트)
  # /portfolio/devIos6083 → params[:username] = "devIos6083"
  get "/portfolio/:username", to: "portfolios#show"

  # resources: CRUD 7개 라우트를 한 번에 생성
  # GET    /projects          → projects#index
  # GET    /projects/new      → projects#new
  # POST   /projects          → projects#create
  # GET    /projects/:id      → projects#show
  # GET    /projects/:id/edit → projects#edit
  # PATCH  /projects/:id      → projects#update
  # DELETE /projects/:id      → projects#destroy
  resources :projects

end
```

### 생성된 URL 헬퍼

| 라우팅                | 헬퍼 메서드                   | URL                  |
| :-------------------- | :---------------------------- | :------------------- |
| `root`                | `root_path`                   | `/`                  |
| `get "/about"`        | `about_path`                  | `/about`             |
| `resources :projects` | `projects_path`               | `/projects`          |
|                       | `new_project_path`            | `/projects/new`      |
|                       | `project_path(@project)`      | `/projects/:id`      |
|                       | `edit_project_path(@project)` | `/projects/:id/edit` |

---

## 2. 레이아웃 — application.html.erb

**모든 페이지에 공통으로 적용되는 외곽 HTML 구조**.  
`yield` 가 있는 곳에 각 페이지의 내용이 삽입된다.

**全(すべ)てのページに共通(きょうつう)で適用(てきよう)される外枠(そとわく)のHTML構造(こうぞう)**。  
`yield`のある場所(ばしょ)に各(かく)ページの内容(ないよう)が挿入(そうにゅう)される。

```erb
<%# app/views/layouts/application.html.erb %>
<!DOCTYPE html>
<html>
  <head>
    <%# content_for(:title)が設定されていればそれを、なければデフォルト値 %>
    <title><%= content_for(:title) || "Dev Portfolio" %></title>

    <%= csrf_meta_tags %>   <%# CSRF攻撃(こうげき)防止(ぼうし)のトークン %>
    <%= csp_meta_tag %>     <%# コンテンツセキュリティポリシー %>

    <%= stylesheet_link_tag :app, "data-turbo-track": "reload" %>
    <%= javascript_importmap_tags %>
  </head>
  <body>

    <%# 全ページ共通(きょうつう)ナビゲーション %>
    <nav>
      <%= link_to "Home", root_path %>
      <%= link_to "About", about_path %>
      <%# 自分(じぶん)のポートフォリオへの直接(ちょくせつ)リンク %>
      <%= link_to "マイポートフォリオ", "/portfolio/devIos6083" %>
    </nav>

    <%# ここに各(かく)ページのビューが挿入(そうにゅう)される %>
    <%= yield %>

  </body>
</html>
```

### 주요 헬퍼 태그

| 태그                        | 역할                         |
| :-------------------------- | :--------------------------- |
| `csrf_meta_tags`            | CSRF 공격 방지 토큰 생성     |
| `csp_meta_tag`              | 콘텐츠 보안 정책 메타 태그   |
| `stylesheet_link_tag`       | CSS 파일 읽기                |
| `javascript_importmap_tags` | JS importmap 설정            |
| `link_to "텍스트", 경로`    | `<a>` 태그 생성              |
| `yield`                     | 각 뷰의 내용이 들어오는 자리 |

---

## 3. HomeController — 루트 페이지

**앱의 첫 화면. `/` 로 접속하면 표시된다.**

```ruby
# app/controllers/home_controller.rb
class HomeController < ApplicationController
  def index
    # 현재는 인스턴스 변수 없음 — 뷰에 하드코딩
    # 나중에 @projects = Project.recent(5) 등을 추가할 예정
  end
end
```

```erb
<%# app/views/home/index.html.erb %>
<h1>Dev_Portfolio</h1>
<p>エンジニアのポートフォリオを公開(こうかい)・管理(かんり)するアプリです</p>
<%= link_to "アプリについて", about_path %>
```

---

## 4. PagesController — About 페이지

**앱 소개 페이지. `/about` 으로 접속하면 표시된다.**  
인스턴스 변수 `@title`, `@description` 을 컨트롤러에서 설정해서 뷰에 전달한다.

```ruby
# app/controllers/pages_controller.rb
class PagesController < ApplicationController
  def about
    # @변수: 컨트롤러 → 뷰로 데이터를 전달하는 인스턴스 변수
    @title       = "Dev_Portfolio"
    @description = "#{@title}はエンジニアのポートフォリオを管理(かんり)・公開(こうかい)するアプリです"
  end
end
```

```erb
<%# app/views/pages/about.html.erb %>
<h1><%= @title %></h1>
<p><%= @description %></p>
<%= link_to "ホームへ", root_path %>
```

---

## 5. PortfoliosController — 포트폴리오 페이지

**URL에서 `:username` 파라미터를 받아서 해당 유저의 포트폴리오를 표시.**  
현재는 준비중 페이지이며, 나중에 DB 연동 예정.

```ruby
# app/controllers/portfolios_controller.rb
class PortfoliosController < ApplicationController
  def show
    # params[:username]: URL의 :username 부분을 꺼냄
    # /portfolio/devIos6083 → params[:username] = "devIos6083"
    # .to_s: nil 방지 (파라미터가 없을 때 빈 문자열로)
    @username = params[:username].to_s
  end
end
```

```erb
<%# app/views/portfolios/show.html.erb %>
<h1><%= @username %>さんのポートフォリオ</h1>
<p>プロジェクトは準備中(じゅんびちゅう)です</p>
<%= link_to "ホームへ", root_path %>
```

---

## 6. 현재 앱 구조 정리

```
dev_portfolio/
├── config/
│   └── routes.rb               ← 라우팅 정의
│
├── app/
│   ├── controllers/
│   │   ├── application_controller.rb  ← 공통 부모
│   │   ├── home_controller.rb         ← / (루트)
│   │   ├── pages_controller.rb        ← /about
│   │   └── portfolios_controller.rb   ← /portfolio/:username
│   │
│   └── views/
│       ├── layouts/
│       │   └── application.html.erb   ← 공통 레이아웃 (nav 포함)
│       ├── home/
│       │   └── index.html.erb
│       ├── pages/
│       │   └── about.html.erb
│       └── portfolios/
│           └── show.html.erb
```

### 요청 → 응답 흐름

```
브라우저: GET /about
    ↓
routes.rb: get "/about" → pages#about
    ↓
PagesController#about 실행
  @title = "Dev_Portfolio"
  @description = "..."
    ↓
app/views/pages/about.html.erb 렌더링
  (application.html.erb 의 yield 안에 삽입)
    ↓
브라우저에 HTML 응답
```

---

### 🔥 다음 구현 목표

```
Phase 1 — 현재 완료 ✅
  ✅ 기본 라우팅 (root, about, portfolio/:username)
  ✅ resources :projects (CRUD 라우팅)
  ✅ 공통 레이아웃 (nav 포함)
  ✅ 각 페이지 컨트롤러 & 뷰

Phase 2 — 다음 목표 🔜
  □ Project 모델 생성 (title, description, github_url, demo_url)
  □ projects#index — 프로젝트 목록 페이지
  □ projects#show  — 프로젝트 상세 페이지
  □ projects#new / create — 프로젝트 추가 폼
  □ portfolios#show 에 실제 프로젝트 목록 연동

Phase 3 — 발전 목표 💡
  □ CSS 스타일링 (Tailwind or Bootstrap)
  □ 이미지 업로드 (Active Storage)
  □ 사용자 인증 (Devise)
  □ 배포 (Render / Fly.io)
```

---

## 🔑 한눈에 보는 키워드

| 키워드                            | 설명                                    |
| :-------------------------------- | :-------------------------------------- |
| `root "컨트롤러#액션"`            | 루트 경로(/) 설정                       |
| `get "경로", to: "컨트롤러#액션"` | GET 요청 라우팅                         |
| `get "경로/:파라미터"`            | URL 동적 파라미터                       |
| `resources :모델`                 | CRUD 7개 라우팅 한 번에 생성            |
| `params[:파라미터]`               | URL 파라미터 값 꺼내기                  |
| `params[:파라미터].to_s`          | nil 방지 문자열 변환                    |
| `root_path` / `about_path`        | 라우팅에서 자동 생성되는 헬퍼           |
| `<%= yield %>`                    | 레이아웃에서 각 뷰 내용이 삽입되는 자리 |
| `content_for(:title)`             | 페이지별로 title을 동적으로 설정        |
| `link_to "텍스트", 경로`          | `<a href="">` 태그 생성                 |
| `csrf_meta_tags`                  | CSRF 토큰 메타 태그                     |
| `@변수`                           | 컨트롤러 → 뷰로 데이터 전달             |

---

[← 목차로 돌아가기 / 目次(もくじ)に戻(もど)る](../README.md)
