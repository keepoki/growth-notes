#HTML 

# Semantic Tag

**시맨틱(semantic)** = "의미가 있는"  
즉, 시맨틱 태그는 **"이 태그 안에 담긴 내용이 어떤 성격/역할을 하는지 브라우저와 개발자에게 알려주는 태그"** 예요.

## 기본 개념

예를 들어, 단순히 `<div>`는 "그냥 박스"일 뿐이지만,
- `<header>` → "여기는 머리글(상단 영역)"
- `<main>` → "여기가 문서의 핵심 내용"
- `<article>` → "독립된 글/게시물"
- `<footer>` → "하단 영역"
처럼 **태그 이름만 봐도 무슨 역할을 하는지 알 수 있죠.**

### 시맨틱 태그가 중요한 이유
1. **가독성**
    개발자가 코드를 읽을 때 구조를 바로 파악할 수 있음. (유지보수에 유리)
2. **SEO(검색 최적화)**
    검색 엔진이 페이지 구조를 이해하기 쉬움 → 검색 노출에 도움.
3. **웹 접근성**
    스크린리더 같은 보조 기기가 문서를 읽을 때 의미를 이해 가능.

### 알기 쉬운 비유
시맨틱 태그는 **책의 목차**와 같아요.
- `<header>` → 책의 제목 페이지
- `<nav>` → 목차
- `<main>` → 본문 내용
- `<article>` → 독립적인 챕터(기사, 블로그 글)
- `<section>` → 본문 안의 큰 단락
- `<aside>` → 각주나 참고자료
- `<footer>` → 저자 소개, 출판사 정보
만약 책에 목차나 장 제목 없이 그냥 글자만 줄줄이 있으면 읽기 힘들겠죠?  
→ 시맨틱 태그는 웹페이지를 "잘 정리된 책"처럼 만들어줍니다.


##  핵심 개념

1. **시맨틱의 정의 — 의미를 담는 태그**
   태그가 단지 스타일용(class 이름이 아닌) 의미(의도)를 표현해야 한다는 뜻입니다. 의미가 있으면 브라우저·스크린리더·검색엔진·개발자가 문서 구조를 바로 이해합니다. 예: `article`은 *독립적으로 의미가 유지되는 콘텐츠 단위*.
2. **랜드마크(페이지 골격)**
   * `header`, `nav`, `main`, `aside`, `footer`는 문서의 **주요 역할(랜드마크)** 을 나타냅니다.
   * **원칙**: `main`은 페이지당 하나, `header/footer`는 섹션 수준으로 여러 번 가능. 랜드마크는 탐색·접근성 개선의 핵심입니다.
3. **`section` vs `article` vs `div` (결정 규칙)**
   * `article`: 독립 배포/재사용 가능(블로그 포스트, 뉴스 기사, 댓글 단건).
   * `section`: **주제별 구획**(보통 heading을 포함해야 정당화됨).
   * `div`: 의미 없음 — **레이아웃/그룹** 용도일 때만 사용.
     → 판단: “이 블록을 단독으로 보면 의미가 있나?” → `article` / “제목(주제)이 있는 구획인가?” → `section` / 아니면 `div`.
4. **헤딩 계층 구조(문서의 논리적 목차)**
   * `h1`은 문서의 최상위 제목(보통 1개). 그 아래는 `h2`, `h3` 순으로 **논리적 순서**를 지킵니다. 건너뛰면 스크린리더/SEO/구조 파악에 혼란을 줍니다.
5. **폼 접근성 — 레이블이 핵심**
   * 각 입력은 `label`과 연결(예: `for`/`id`)되어야 클릭 영역과 스크린리더 경험이 정상 작동합니다. 복잡한 그룹엔 `fieldset` + `legend` 사용.
6. **미디어와 캡션**
   * 이미지엔 `alt`(내용 요약) 필수. 설명 필요하면 `figure` + `figcaption`. 날짜엔 `time datetime="..."`로 기계 판독 가능하게.
7. **인라인 의미 태그**
   * `strong`(의미적 중요도), `em`(강조), `code`/`pre`(코드), `abbr`(약어), `mark`(하이라이트) 등을 사용하면 기계와 사람이 더 잘 이해합니다.
8. **ARIA는 보조 수단 (원칙: Native > ARIA)**
   * 가능한 네이티브 시맨틱 태그를 먼저 쓰고, 부족할 때만 `role`/`aria-*`로 보완하세요. ARIA 오용은 더 나쁜 접근성을 만듭니다.
9. **SEO·접근성·유지보수 연결고리**
   * 시맨틱 태그는 검색엔진에게 ‘중요 콘텐츠’임을 알리고, 스크린리더 사용자가 문서 구조를 건너뛸 수 있게 하며, 코드 가독성과 유지보수를 크게 개선합니다.
10. **안티패턴(주의할 것)**
    * 모든 걸 `div`로만 구성(`div soup`)
    * `section`에 제목 없음
    * 버튼 역할을 `div/span`으로 처리(네이티브 `button` 사용 권장)
    * 이미지 alt 비워두기(장식 이미지면 `alt=""`로 명시)

## 예시

### 1) 기본 페이지 골격

```html
<!doctype html>
<html lang="ko">
<head><meta charset="utf-8"><title>예시</title></head>
<body>
  <header>로고 · 검색</header>
  <nav aria-label="주요 메뉴">…</nav>
  <main>
    <article>
      <h1>글 제목</h1>
      <p>본문…</p>
    </article>
  </main>
  <footer>© 2025</footer>
</body>
</html>
```

### 2) article + section + figure 예시

```html
<article class="post">
  <header><h1>무선 헤드폰 리뷰</h1></header>

  <section aria-labelledby="feat">
    <h2 id="feat">특징</h2>
    <ul><li>노이즈 캔슬링</li></ul>
  </section>

  <figure>
    <img src="phone.jpg" alt="헤드폰 정면 사진">
    <figcaption>제품 사진</figcaption>
  </figure>

  <footer><time datetime="2025-09-02">2025.09.02</time></footer>
</article>
```

### 3) 올바른 폼 (접근성)

```html
<form action="/signup" method="post">
  <fieldset>
    <legend>회원가입</legend>

    <label for="email">이메일</label>
    <input id="email" name="email" type="email" required>

    <label for="pw">비밀번호</label>
    <input id="pw" name="pw" type="password" required>

    <button type="submit">가입</button>
  </fieldset>
</form>
```

### 4) dl (사양/용어 설명)

```html
<dl>
  <dt>CPU</dt><dd>Apple M3</dd>
  <dt>RAM</dt><dd>16GB</dd>
</dl>
```

### 5) div-soup → 시맨틱 변환 (간단 비교)

원본(안티패턴):

```html
<div class="wrap">
  <div class="top">My Blog</div>
  <div class="menu">Home | About</div>
  <div class="content"><div class="title">첫 글</div></div>
  <div class="bottom">©</div>
</div>
```

시맨틱 변환:

```html
<body>
  <header>My Blog</header>
  <nav aria-label="메인 네비">Home · About</nav>
  <main>
    <article>
      <h1>첫 글</h1>
      <p>본문…</p>
    </article>
  </main>
  <footer>©</footer>
</body>
```

### 빠른 체크(3초 점검)
* `main` 1개인가? / `section`에 제목 있나? / 폼엔 `label`이 연결됐나? / 이미지 `alt`가 적절한가?
