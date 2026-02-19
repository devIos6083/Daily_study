# ============================================
# Day 1: 클래스(Class) & 모듈(Module)
# ============================================


# ──────────────────────────────────────────
# 1. 클래스 (Class)
# ──────────────────────────────────────────
# - 데이터(변수)와 기능(메서드)을 하나로 묶는 설계도
# - User.new("강씨", 26) 으로 설계도를 바탕으로 실제 객체를 만든다
# - 만들어진 객체를 "인스턴스"라고 부른다

class User
  # attr_accessor: @name, @age의 getter/setter를 자동으로 만들어줌
  # 없으면 외부에서 user.name 으로 읽거나 쓰는 게 불가능
  attr_accessor :name, :age

  # initialize: new() 호출 시 자동으로 실행되는 메서드
  # 반드시 이름이 initialize 여야 함 (initializer ❌)
  def initialize(name, age)
    @name = name   # @변수: 인스턴스 변수. 이 객체 안 어디서든 사용 가능
    @age  = age
  end

  def introduce
    "안녕하세요, #{@name}이고 #{@age}살입니다"
  end
end

user = User.new("강씨", 26)
puts user.introduce   # → 안녕하세요, 강씨이고 26살입니다
puts user.name        # → 강씨      (attr_accessor 덕분에 읽기 가능)
user.name = "Charles" # attr_accessor 덕분에 쓰기도 가능
puts user.name        # → Charles


# ──────────────────────────────────────────
# 2. 인스턴스 변수 @변수 vs 로컬 변수
# ──────────────────────────────────────────

class Example
  def set_value
    @saved = "나는 살아남는다"  # @변수: 메서드가 끝나도 인스턴스에 저장됨
    temp   = "나는 사라진다"    # 로컬 변수: 이 메서드 안에서만 존재
  end

  def get_value
    puts @saved   # ✅ 다른 메서드에서도 접근 가능
    puts temp     # ❌ 에러! 로컬 변수는 메서드 밖에서 접근 불가
  end
end


# ──────────────────────────────────────────
# 3. 모듈 (Module)
# ──────────────────────────────────────────
# - 메서드 묶음. 클래스가 아니라서 new()로 객체를 만들 수 없음
# - include로 클래스에 장착하면, 그 클래스의 메서드처럼 사용 가능
# - 핵심 장점: 하나의 모듈을 수백 개의 클래스에 재사용 가능
#
# 비유:
# 클래스 = 사람
# 모듈   = 자격증 (수영, 운전, 요리...)
# 자격증 하나를 여러 사람이 취득(include)해서 그 기능을 쓸 수 있다

module Greetable
  def greet
    # @name은 이 모듈을 include한 클래스의 인스턴스 변수를 가리킴
    # include되는 순간 같은 인스턴스 안으로 합쳐지기 때문에 접근 가능
    "#{@name}가 인사합니다!"
  end
end

module Farewell
  def bye
    "#{@name}가 작별 인사합니다!"
  end
end

class User
  include Greetable  # 모듈 장착. 여러 개 include 가능
  include Farewell

  def initialize(name)
    @name = name
  end
end

user = User.new("강씨")
puts user.greet   # → 강씨가 인사합니다!
puts user.bye     # → 강씨가 작별 인사합니다!


# ──────────────────────────────────────────
# 4. Ruby다운 습관 (오늘 배운 것)
# ──────────────────────────────────────────

# ✅ ?로 끝나는 메서드는 true/false를 반환하는 관례
# ✅ 비교식 자체가 이미 true/false이므로 if/else 불필요
def long_book?
  @page_count >= 300  # 300 이상(>=) vs 300 초과(>) 주의!
end

# ✅ 할인 계산처럼 값을 반환만 하면 되는 경우
# 변수에 재할당할 필요 없이 계산식을 바로 반환
def discounted_price(original_price)
  if @page_count >= 300
    original_price * 0.8   # 마지막 줄의 값이 자동으로 반환됨
  else
    original_price * 0.9
  end
end


# ──────────────────────────────────────────
# 5. 한눈에 보는 키워드 정리
# ──────────────────────────────────────────
#
# class 이름        → 클래스 정의 시작
# end               → 클래스/메서드/모듈 종료
# def initialize    → new() 호출 시 자동 실행 (생성자)
# @변수             → 인스턴스 변수. 같은 객체 안 어디서든 접근 가능
# attr_accessor :x  → @x의 읽기(getter) + 쓰기(setter) 자동 생성
# attr_reader :x    → 읽기만
# attr_writer :x    → 쓰기만
# module 이름       → 모듈 정의
# include 모듈이름  → 클래스에 모듈 장착
# 클래스.new(값)    → 인스턴스(객체) 생성
