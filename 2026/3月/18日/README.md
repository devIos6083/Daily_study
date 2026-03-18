# 📅 Day 25 — Dev Portfolio : 모델 설계 & CRUD (Create / Read)

**날짜 / 日付(ひづけ):** 2026.03.18  
**소요 시간 / 所要時間(しょようじかん):** 約(やく) 2時間(じかん)30分(ふん) × 2세션  
**한 줄 목표:** Project 모델 + DB 테이블을 설계하고, 목록·상세·등록 페이지를 실제로 동작하게 구현한다.  
**一言目標(ひとこともくひょう):** Projectモデル・DBテーブルを設計(せっけい)し、一覧(いちらん)・詳細(しょうさい)・登録(とうろく)ページを実際(じっさい)に動作(どうさ)するよう実装(じっそう)する。

---

## 목차 / 目次(もくじ)

1. [모델 & 마이그레이션](#1-모델--마이그레이션)
2. [ActiveRecord — 모델 클래스](#2-activerecord--모델-클래스)
3. [CRUD — Create / Read 구현](#3-crud--create--read-구현)
4. [뷰 설계](#4-뷰-설계)
5. [오늘의 실수 & 해결](#5-오늘의-실수--해결)

---

## 1. 모델 & 마이그레이션

### 모델 · 마이그레이션 · ActiveRecord의 관계

```
모델 (app/models/project.rb)
  → Ruby 클래스. DB 테이블과 1:1 대응. 유효성 검사·스코프 정의.

마이그레이션 (db/migrate/xxxx_create_projects.rb)
  → DB 테이블을 만들거나 변경하는 Ruby 스크립트.
    rails db:migrate 로 실행해야 실제 DB에 반영됨.

ActiveRecord
  → Rails가 제공하는 ORM. 모델과 DB를 연결해서
    Project.find(1) 같은 Ruby 코드로 SQL을 자동 생성.
```

### 모델 생성 명령어

```bash
# rails generate model 모델명 컬럼명:타입 ...
# → app/models/project.rb 와 db/migrate/xxxx_create_projects.rb 동시 생성
rails generate model Project title:string description:text \
  github_url:string demo_url:string thumbnail:string
```

### 최초 마이그레이션 파일

```ruby
# db/migrate/xxxx_create_projects.rb
class CreateProjects < ActiveRecord::Migration[8.1]
  def change
    create_table :projects do |t|
      t.string  :title                      # 짧은 텍스트 (255자)
      t.text    :description                # 긴 텍스트 (제한 없음)
      t.string  :github_url
      t.string  :demo_url
      t.string  :thumbnail
      t.integer :view_count,   default: 0   # 정수, 기본값 0
      t.integer :likes_count,  default: 0
      t.timestamps                          # created_at / updated_at 자동 추가
    end
  end
end
```

### 컬럼 타입 정리

| 타입         | Ruby       | DB                      | 사용 예         |
| :----------- | :--------- | :---------------------- | :-------------- |
| `string`     | String     | VARCHAR(255)            | 제목, URL, 이름 |
| `text`       | String     | TEXT                    | 본문, 설명      |
| `integer`    | Integer    | INT                     | 카운터, 나이    |
| `boolean`    | true/false | BOOLEAN                 | 공개 여부       |
| `datetime`   | Time       | DATETIME                | 날짜·시각       |
| `timestamps` | —          | created_at + updated_at | 자동 관리       |

### 마이그레이션 실행

```bash
rails db:migrate           # 미실행 마이그레이션 모두 실행
rails db:migrate:status    # 실행 상태 확인 (up / down)
cat db/schema.rb           # 현재 DB 상태 확인 ← 자주 확인하는 습관!
```

---

### 기존 테이블에 컬럼 추가 — add_column

```bash
# 마이그레이션 이름 규칙: Add[컬럼명]To[테이블명]
# → Rails가 자동으로 add_column을 생성해줌
rails generate migration AddViewCountToProjects view_count:integer
rails generate migration AddLikesCountToProjects likes_count:integer
```

```ruby
class AddViewCountToProjects < ActiveRecord::Migration[8.1]
  def change
    add_column :projects, :view_count, :integer
    # 주의: 타입과 컬럼명 순서 → (테이블명, 컬럼명, 타입)
  end
end
```

### 기본값 변경 — change_column_default

```ruby
class AddDefaultToProjectCount < ActiveRecord::Migration[8.1]
  def change
    # from: nil → to: 0 으로 기본값 변경
    # 기존 데이터는 변경되지 않고 새로 추가되는 레코드부터 적용
    change_column_default :projects, :view_count,  from: nil, to: 0
    change_column_default :projects, :likes_count, from: nil, to: 0
  end
end
```

> **💡 처음부터 `default: 0` 을 쓰면 이 과정이 필요 없다.**  
> `create_table` 안에서 `t.integer :view_count, default: 0` 으로 처음부터 정의하는 것이 깔끔하다.

---

## 2. ActiveRecord — 모델 클래스

### 유효성 검사 (Validation)

```ruby
# app/models/project.rb
class Project < ApplicationRecord
  # title이 비어있으면 저장 불가
  validates :title, presence: true

  # ...
end
```

### 스코프 (Scope)

```ruby
class Project < ApplicationRecord
  validates :title, presence: true

  # scope: 자주 쓰는 쿼리에 이름을 붙여 재사용
  # -> { } : 람다(lambda) — 호출 시점에 평가됨 (항상 최신 시각 기준)
  scope :recent,      -> { order(created_at: :desc) }   # 최신순
  scope :popular,     -> { order(likes_count: :desc) }  # 좋아요 많은 순
  scope :most_viewed, -> { order(view_count: :desc) }   # 조회수 많은 순
end
```

### Rails 콘솔로 DB 직접 조작

```bash
rails console   # 또는 rails c
```

```ruby
# 레코드 생성
Project.create!(title: "Dev Portfolio", description: "ポートフォリオアプリ")

# 전체 조회
Project.all

# 스코프 사용
Project.recent
Project.popular

# 특정 레코드 찾기
Project.find(1)           # id=1, 없으면 에러
Project.find_by(title: "Dev Portfolio")  # 없으면 nil

# 종료
exit
```

---

## 3. CRUD — Create / Read 구현

### ProjectsController

```ruby
# app/controllers/projects_controller.rb
class ProjectsController < ApplicationController

  # GET /projects — 프로젝트 목록
  def index
    # scope :recent 사용 → 최신순으로 전체 목록
    @projects = Project.recent
  end

  # GET /projects/:id — 프로젝트 상세
  def show
    # params[:id]: URL의 :id 부분 (예: /projects/3 → params[:id] = "3")
    # find: id가 없으면 ActiveRecord::RecordNotFound 에러 발생
    @project = Project.find(params[:id])
  end

  # GET /projects/new — 등록 폼 표시
  def new
    # 빈 Project 객체를 뷰에 전달 (form_with에서 새 레코드임을 판단)
    @project = Project.new
  end

  # POST /projects — 실제 저장
  def create
    @project = Project.new(project_params)

    if @project.save           # 유효성 검사 통과 + DB 저장 성공
      redirect_to @project     # → /projects/:id 로 리다이렉트
    else
      # 저장 실패 → 폼 다시 표시 (에러 메시지 포함)
      render :new, status: :unprocessable_entity
    end
  end

  private

  # Strong Parameters: 허용된 파라미터만 받아서 보안 강화
  # params.require(:project).permit(...) → :project 키 안의 지정 컬럼만 허용
  def project_params
    params.require(:project).permit(:title, :description, :github_url, :demo_url, :thumbnail)
  end
end
```

### create 흐름 상세

```
POST /projects
    ↓
ProjectsController#create
    ↓
Project.new(project_params)  ← Strong Parameters로 안전하게 받기
    ↓
@project.save
    ↓ 성공                    ↓ 실패 (validates 위반 등)
redirect_to @project         render :new, status: :unprocessable_entity
    ↓                             ↓
/projects/:id 페이지          새 글 작성 폼 다시 표시
                               (에러 메시지가 @project.errors 에 담겨있음)
```

---

## 4. 뷰 설계

### 공유 폼 파셜 — `_form.html.erb`

```erb
<%# app/views/projects/_form.html.erb %>
<%# form_with model: @project — @project가 새 객체면 POST, 기존이면 PATCH %>
<%= form_with model: @project do |f| %>
  <%= f.label :title, "タイトル" %>
  <%= f.text_field :title %>

  <%= f.label :description, "説明(せつめい)" %>
  <%= f.text_area :description %>

  <%= f.label :github_url, "GitHub URL" %>
  <%= f.url_field :github_url %>

  <%= f.label :demo_url, "デモ URL" %>
  <%= f.url_field :demo_url %>

  <%= f.label :thumbnail, "サムネイル URL" %>
  <%= f.url_field :thumbnail %>

  <%= f.submit "保存(ほぞん)する" %>
<% end %>
```

### 목록 페이지 — `index.html.erb`

```erb
<%# app/views/projects/index.html.erb %>
<h1>プロジェクト一覧(いちらん)</h1>
<%= link_to "新規(しんき)プロジェクト登録(とうろく)", new_project_path %>

<% @projects.each do |project| %>
  <div>
    <%# link_to "텍스트", project → /projects/:id へのリンク %>
    <%= link_to project.title, project %>
    <p><%= project.description %></p>
  </div>
<% end %>
```

### 등록 페이지 — `new.html.erb`

```erb
<%# app/views/projects/new.html.erb %>
<h1>新規(しんき)プロジェクト登録(とうろく)</h1>

<%# 파셜 렌더링 — _form.html.erb 를 project 변수와 함께 %>
<%= render "form", project: @project %>

<%= link_to "一覧(いちらん)に戻(もど)る", projects_path %>
```

### 상세 페이지 — `show.html.erb`

```erb
<%# app/views/projects/show.html.erb %>
<h1><%= @project.title %></h1>
<p><%= @project.description %></p>

<%# present?: nil / 빈 문자열이 아닐 때만 true → URL이 있을 때만 링크 표시 %>
<%= link_to "GitHubを見(み)る", @project.github_url,
    target: "_blank",
    rel: "noopener noreferrer" if @project.github_url.present? %>

<%= link_to "デモを見(み)る", @project.demo_url,
    target: "_blank",
    rel: "noopener noreferrer" if @project.demo_url.present? %>

<%= link_to "一覧(いちらん)に戻(もど)る", projects_path %>
```

> **`target: "_blank"` 를 쓸 때는 반드시 `rel: "noopener noreferrer"` 함께 쓸 것!**  
> 이유: `_blank` 로 열린 페이지에서 `window.opener` 를 통해 원래 페이지를 조작하는 보안 취약점 방지.

---

## 5. 오늘의 실수 & 해결

| 실수                                  | 원인                                   | 해결                                   |
| :------------------------------------ | :------------------------------------- | :------------------------------------- |
| `view_count`, `likes_count` 컬럼 누락 | 마이그레이션 실행 전에 파일 수정 안 함 | 새 마이그레이션으로 `add_column` 추가  |
| `t.view_count :integer`               | 타입/컬럼명 순서 혼동                  | `t.integer :view_count` 가 올바른 순서 |
| `scope :recent ->`                    | 쉼표 누락                              | `scope :recent, ->` (쉼표 필수!)       |
| `order(created_at: desc)`             | 심볼 표기 오류                         | `order(created_at: :desc)` (`:` 필요)  |
| `AddViewCountPosts`                   | 마이그레이션 네이밍 오류               | `Add[컬럼]To[테이블]` 규칙 준수        |

### 마이그레이션 네이밍 규칙

```bash
# ✅ 올바른 예
rails generate migration AddViewCountToProjects view_count:integer
#                         ↑컬럼명         ↑테이블명

# ❌ 잘못된 예
rails generate migration AddViewCountPosts   # 테이블명 앞에 To 없음
rails generate migration AddViewCount        # 테이블명 없음
```

---

## 현재 구현 상태

```
✅ GET  /projects          → 목록 (index)
✅ GET  /projects/new      → 등록 폼 (new)
✅ POST /projects          → 저장 (create)
✅ GET  /projects/:id      → 상세 (show)

🔜 GET  /projects/:id/edit → 수정 폼 (edit)
🔜 PATCH /projects/:id     → 수정 저장 (update)
🔜 DELETE /projects/:id    → 삭제 (destroy)
```

---

## 🔑 한눈에 보는 키워드

| 키워드                                            | 설명                              |
| :------------------------------------------------ | :-------------------------------- | --- | ------------------------ |
| `rails g model 이름 컬럼:타입`                    | 모델 + 마이그레이션 동시 생성     |
| `rails db:migrate`                                | 마이그레이션 실행 → DB에 반영     |
| `create_table :테이블 do                          | t                                 | `   | 테이블 생성 마이그레이션 |
| `t.string` / `t.text` / `t.integer`               | 컬럼 타입 지정                    |
| `default: 0`                                      | 기본값 설정                       |
| `t.timestamps`                                    | created_at / updated_at 자동 추가 |
| `add_column :테이블, :컬럼, :타입`                | 기존 테이블에 컬럼 추가           |
| `change_column_default`                           | 기본값 변경                       |
| `cat db/schema.rb`                                | 현재 DB 상태 확인                 |
| `validates :title, presence: true`                | 제목 필수 유효성 검사             |
| `scope :이름, -> { }`                             | 자주 쓰는 쿼리 재사용             |
| `Project.find(params[:id])`                       | id로 레코드 조회                  |
| `Project.new` / `.save`                           | 새 객체 생성 / DB 저장            |
| `redirect_to @project`                            | 저장 후 상세 페이지로 이동        |
| `render :new, status: :unprocessable_entity`      | 저장 실패 시 폼 재표시            |
| `params.require(:모델).permit(...)`               | Strong Parameters                 |
| `form_with model: @project`                       | 모델 기반 폼 생성                 |
| `render "form", project: @project`                | 파셜 렌더링                       |
| `.present?`                                       | nil / 빈 문자열이 아닐 때 true    |
| `target: "_blank"` + `rel: "noopener noreferrer"` | 외부 링크 보안 설정               |

---

[← 목차로 돌아가기 / 目次(もくじ)に戻(もど)る](../README.md)
