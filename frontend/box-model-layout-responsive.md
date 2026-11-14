---
track: B
notion: https://app.notion.com/p/415178c4afac814cbe3efc1cc7a38908
---

# Box Model / Flex / Grid / Position / Responsive Design


<callout icon="📍" color="gray_bg">
**학습 범위**: Track B · 1–3주 · Web Platform · Box Model / Flex / Grid / Position / Responsive Design

박스의 크기에서 시작하여 한 축·행과 열·위치 기준·환경 변화로 설명 범위를 넓힌다.
</callout>

<table-of-contents />

## 1. 학습 목표

이 페이지를 마치면 다음을 할 수 있어야 한다.

- box-sizing과 여백을 기준으로 테두리 상자의 크기를 계산할 수 있다.
- Flex의 공간 분배와 Grid의 트랙 배치를 목적에 맞게 선택할 수 있다.
- absolute·fixed·sticky의 위치 기준과 흐름 참여를 설명할 수 있다.
- 반응형 분기·유연한 크기·확대·읽기 순서를 함께 설계할 수 있다.
- DevTools로 넘침·최소 크기·겹침·실제 렌더링 비용을 검증할 수 있다.

---

## 2. 이 주제를 왜 배우는가

레이아웃 문제는 하나의 속성만으로 생기지 않는다. 카드의 폭은 박스 모델에, 카드들의 관계는 배치 방식에, 배지 위치는 기준 박스에, 좁은 화면의 변화는 반응형 규칙에 영향을 받는다. 먼저 상자의 크기를 정하고 상자 사이의 관계를 정해야 예측 가능한 화면을 만들 수 있다.

---

## 3. 관련 계층과 역할 구분

HTML은 읽기와 조작의 순서를 정하고 DOM은 그 관계를 보관한다. CSS는 캐스케이드로 정한 스타일을 바탕으로 박스 크기와 배치 문맥을 만든다. 브라우저의 레이아웃은 위치·크기를 계산하고 페인트는 필요한 표현을 그린다.

같은 DOM에 다른 CSS를 적용하여 열 수를 바꿀 수 있지만 의미와 기본 키보드 순서를 자동으로 함께 재설계해 주지는 않는다. JavaScript로 폭을 읽어 조건을 관리하기 전에 CSS 자체로 표현 가능한 관계인지 확인한다. 렌더링 파이프라인의 세부 구조는 4–6주 Browser Internals에서 확장한다.

---

## 4. 크기와 배치의 기본 단위

CSS 박스는 안쪽부터 content, padding, border, margin으로 이해한다. 기본 `content-box`에서 `width: 200px; padding: 20px; border: 2px solid`이면 가로 테두리 상자 크기는 244px이다. `border-box`에서는 지정한 200px 안에 content·padding·border가 들어간다. margin은 두 경우 모두 바깥에 있다. [박스 모델 표준](https://www.w3.org/TR/css-box-3/)은 이 영역들의 관계를 정의한다.

블록과 인라인은 기본 배치 흐름을 이해하는 출발점이다. 모든 요소가 언제나 단순한 직사각형 하나만 만드는 것은 아니며, 줄바꿈된 인라인은 여러 조각으로 나뉠 수 있다. 일반 블록 흐름에서는 일부 세로 margin이 겹쳐지는 margin collapse가 있지만, Flex·Grid 아이템 사이의 margin을 같은 방식으로 생각하지 않는다.

---

## 5. Box Model: 지정한 폭과 화면에서 차지하는 폭

### content·padding·border·margin을 따로 계산한다

가로 content 폭이 200px, 좌우 padding이 각각 16px, border가 각각 2px이면 content-box의 테두리 상자 폭은 236px이다. 좌우 margin 12px까지 배치 여유에 고려하면 바깥 간격을 포함한 합은 260px이 된다. 다만 실제 공간 분배는 배치 방식과 auto margin 등에 따라 달라지므로 단순 덧셈을 모든 레이아웃의 크기 결정 알고리즘으로 일반화하지 않는다.

border-box에서는 width에 padding·border가 포함되어 같은 width 200px에서 내용 영역이 줄어든다. margin까지 width에 포함되는 것은 아니다. border-box를 전역으로 지정하면 계산을 단순하게 만들 수 있지만 내용의 최소 크기·줄바꿈·오버플로를 해결하는 만능 설정은 아니다.

### width auto와 width 100%의 차이

일반 블록의 auto 폭은 가용 공간과 margin·padding·border를 고려하여 정해진다. content-box에서 width 100%에 padding을 더하면 부모보다 테두리 상자가 커질 수 있다. “부모 폭을 채운다”는 요구를 구현할 때 auto·100%·box-sizing을 함께 확인한다.

### margin collapse가 간격 덧셈을 바꾸는 경우

일반 블록 흐름의 인접 세로 margin은 조건에 따라 겹쳐질 수 있다. 양수 20px과 30px이면 간격이 50px이 아니라 30px처럼 보이는 것이 대표적인 경우다. 부모와 첫 자식 사이에서도 조건에 따라 겹침이 일어나 부모 밖으로 간격이 나온 것처럼 보일 수 있다.

Flex·Grid 아이템 사이에는 같은 margin collapse 규칙을 그대로 적용하지 않는다. 반복 항목 사이의 일관된 간격은 해당 배치 방식의 gap을 검토하면 관계가 분명해진다. 특정 현상을 없애기 위해 overflow를 넣기 전에 그 속성이 잘림·스크롤·sticky에 미치는 영향도 확인한다.

---

## 6. 상자를 배치하는 방법을 고른다

| 방법 | 적합한 관계 | 주의할 점 |
| --- | --- | --- |
| 일반 흐름 | 문단·제목 등 문서 순서 | 우선 기본 흐름으로 해결 가능한지 확인 |
| Flex | 한 축의 항목 정렬·공간 분배 | 주축은 항상 가로가 아니라 `flex-direction`에 따름 |
| Grid | 행과 열을 함께 맞추는 배치 | 트랙 크기와 최소 크기 제약 확인 |
| Position | 배지·고정 영역 등 별도 위치 관계 | 기준 박스와 흐름 이탈 여부 확인 |

Flex의 주축(main axis)은 `flex-direction`이 정하고 교차축(cross axis)은 그에 수직이다. `justify-content`는 주축의 남는 공간 분배, `align-items`는 교차축 정렬에 관여한다. `flex-grow`, `flex-shrink`, `flex-basis`는 여유·부족 공간을 나누는 기준이다. `flex: 1`만으로 긴 콘텐츠의 최소 크기 제약이 항상 사라지지는 않는다.

Grid의 행·열은 트랙(track)이다. `fr`은 남는 공간의 비율이며 무조건 컨테이너 전체 폭을 나눈 값이 아니다. `gap`, 고정 트랙, 아이템의 최소 크기도 함께 관여한다. `minmax(0, 1fr)`는 자동 최소 크기 때문에 열이 과도하게 늘어나는 상황을 다룰 때 유용하다. [Flex](https://www.w3.org/TR/css-flexbox-1/)와 [Grid](https://www.w3.org/TR/css-grid-1/)는 서로 대체하는 경쟁 도구가 아니라 배치 관계에 따라 조합하는 도구다.

### Position은 기준을 먼저 찾는다

`relative`는 일반 흐름의 자리를 유지하면서 표시 위치를 이동한다. `absolute`는 일반 흐름에서 빠져 기준이 되는 containing block에 따라 배치된다. 단순한 예제에서는 가까운 positioned 조상이 기준이지만 transform 등 다른 조건도 기준을 만들 수 있다. `fixed`도 보통 viewport를 기준으로 하지만 조상의 transform 등의 영향을 받을 수 있다.

`sticky`는 흐름에 참여하면서 스크롤 조건과 지정한 inset에 따라 달라붙는다. `top` 같은 기준, 조상의 overflow, 스크롤 영역과 부모의 높이를 확인한다. `z-index`는 문서 전체의 공통 점수가 아니라 stacking context라는 겹침 그룹 안에서 비교된다. 자세한 합성 과정은 후속 브라우저 학습에서 다룬다.

---

## 7. Flex: 한 축의 공간을 분배한다

### 주축은 flex-direction이 정한다

row에서는 보통 가로가 주축이고 column에서는 세로가 주축이다. 문서의 쓰기 방향도 고려해야 하므로 justify-content를 “가로 정렬”로만 외우지 않는다. align-items는 각 아이템의 교차축 정렬을 정하고, align-content는 여러 줄이 있는 컨테이너에서 줄들의 남는 공간 분배와 연결된다.

```css
.toolbar {
  display: flex;
  align-items: center;
  gap: .5rem;
}
.toolbar .title { min-width: 0; overflow-wrap: anywhere; }
.toolbar .save { margin-inline-start: auto; }
```

저장 버튼의 auto margin이 주축의 여유 공간을 흡수하여 반대편으로 밀 수 있다. 긴 제목은 최소 크기와 줄바꿈 정책을 정하여 버튼을 밀어내지 않게 한다. CSS의 시각 순서를 바꾸지 않고 DOM에서 제목 다음에 버튼을 두면 읽기와 키보드 순서를 유지하기 쉽다.

### basis·grow·shrink는 출발점과 여유·부족분을 다룬다

flex-basis는 공간 분배를 시작할 기준 크기다. grow는 여유 공간, shrink는 부족 공간을 나누는 데 관여한다. shrink는 단순히 숫자 비율만으로 최종 크기를 나누는 규칙이 아니며 기준 크기와 최소 크기 제약도 영향을 준다.

`flex: 1`로 같은 비율의 여유 공간을 주어도 콘텐츠·min-width·padding 조건에 따라 눈에 보이는 폭이 항상 같지는 않다. 무엇을 같게 만들려는지와 최소 크기 제약을 함께 확인한다.

### wrap은 2차원 행·열 동기화를 보장하지 않는다

flex-wrap으로 여러 줄을 만들면 각 줄은 자체 공간 분배를 한다. 두 번째 줄의 열이 첫 줄과 정확히 맞아야 한다면 Grid가 더 직접적인 모델일 수 있다. 단순히 항목 수가 많다는 이유가 아니라 행·열 사이의 정렬 요구로 선택한다.

---

## 8. Grid: 행과 열의 관계를 함께 표현한다

### 트랙·라인·셀을 구별한다

트랙은 행 또는 열의 폭·높이를 가진 공간이고 라인은 그 경계다. 셀은 행과 열이 만나는 한 칸이며, 영역은 여러 셀을 묶을 수 있다. grid-template-columns로 열의 구조를 선언하면 자식들을 그 구조에 배치할 수 있다.

```css
.layout {
  display: grid;
  grid-template-columns: minmax(0, 1fr) 16rem;
  gap: 1.5rem;
}
```

16rem 열과 gap을 고려한 뒤 남는 공간을 첫 열이 받는다. 1fr을 컨테이너 전체 폭이라고 생각하면 이 계산을 놓친다. minmax(0, 1fr)은 첫 열의 자동 최소 크기를 완화하지만 내용이 넘칠 가능성은 여전히 줄바꿈·스크롤 정책으로 다루어야 한다.

### 내용의 최소 폭과 열 개수를 함께 정하기

```css
.cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(min(100%, 16rem), 1fr));
  gap: 1rem;
}
```

카드가 최소 16rem을 기준으로 배치되되 컨테이너 자체가 더 좁으면 100%를 사용할 수 있도록 했다. auto-fit은 빈 트랙을 접는 동작과 연결되고 auto-fill은 빈 트랙을 유지할 수 있다는 차이가 있다. 카드 개수와 남는 열을 실제로 바꿔 보며 원하는 결과를 확인한다.

명시적으로 선언한 트랙 밖에 항목이 배치되면 암시적 트랙이 생길 수 있다. 예상하지 못한 새 행·열이 보일 때 아이템의 위치 지정과 grid-auto-rows·grid-auto-columns도 확인한다.

---

## 9. Position: 무엇을 기준으로 어디에 남는가

| 값 | 일반 흐름의 자리 | 위치 판단의 핵심 |
| --- | --- | --- |
| static | 참여 | 보통 inset으로 이동하지 않음 |
| relative | 원래 자리 유지 | 자기 원래 위치를 기준으로 시각 이동 |
| absolute | 흐름에서 빠짐 | containing block과 inset |
| fixed | 흐름에서 빠짐 | 보통 viewport, 기준을 바꾸는 조상 조건 확인 |
| sticky | 흐름에 참여 | 스크롤 기준과 inset·부모 영역의 제한 |

relative로 요소를 이동해도 다른 형제는 이동한 새 위치를 기준으로 다시 공간을 비워주지 않는다. absolute 배지는 본문 높이에 자동으로 공간을 예약하지 않으므로 내용과 겹칠 수 있다. 고정 헤더는 작은 화면과 확대 상태에서 핵심 콘텐츠를 가리지 않는지도 점검한다.

### sticky가 작동하지 않을 때

top 또는 논리 inset 같은 기준이 있는지, 실제 스크롤하는 조상이 누구인지, 부모 안에서 움직일 공간이 있는지 확인한다. overflow를 가진 조상 때문에 예상과 다른 스크롤 영역을 기준으로 동작할 수 있다. 숫자를 더 크게 바꾸기 전에 기준을 찾는 것이 우선이다.

### z-index가 커도 앞에 안 나오는 이유

stacking context는 서로 안에서 겹침 순서를 정하는 그룹이다. 자식의 큰 z-index가 부모 그룹 자체의 순서를 넘어 다른 그룹의 모든 요소보다 앞으로 나오는 것은 아니다. position·z-index뿐 아니라 transform·opacity 등의 조건도 그룹을 만들 수 있다. DevTools로 조상들의 그룹 경계를 확인한다.

---

## 10. 예제: 하나의 카드 목록을 화면 크기에 맞추기

```html
<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>학습 카드</title>
  <style>
    *, *::before, *::after { box-sizing: border-box; }
    body { margin: 0; font-family: sans-serif; }
    main { max-width: 70rem; margin-inline: auto; padding: 1rem; }
    .cards { display: grid; grid-template-columns: 1fr; gap: 1rem;
             padding: 0; list-style: none; }
    .card { position: relative; padding: 1.25rem; border: 1px solid #777; }
    .card h2 { padding-inline-end: 3rem; overflow-wrap: anywhere; }
    .badge { position: absolute; inset-block-start: .5rem; inset-inline-end: .5rem; }
    .actions { display: flex; flex-wrap: wrap; gap: .5rem; }
    .actions a { padding: .5rem; }
    a:focus-visible { outline: 3px solid #1259b0; outline-offset: 3px; }
    @media (min-width: 48rem) {
      .cards { grid-template-columns: repeat(2, minmax(0, 1fr)); }
    }
  </style>
</head>
<body>
  <main>
    <h1>학습 카드</h1>
    <ul class="cards" role="list">
      <li class="card">
        <span class="badge">기초</span>
        <h2>문서와 의미</h2>
        <p>작은 화면에서도 내용이 흐르도록 구성합니다.</p>
        <div class="actions"><a href="#html">읽기</a><a href="#practice">실습</a></div>
      </li>
      <li class="card"><h2>스타일과 배치</h2><p>행과 열을 맞춥니다.</p></li>
    </ul>
  </main>
</body>
</html>
```

좁은 화면에서는 한 열, 48rem 이상에서는 두 열이 된다. Grid가 카드 사이의 관계를 정하고 Flex는 카드 안의 링크를 정렬한다. 배지는 `.card`의 위치 기준을 사용하며 흐름 밖에 있으므로 제목과 겹치지 않을 공간을 따로 확보했다. 내용이 길어져도 카드 높이는 콘텐츠에 따라 늘어난다.

목록 표시를 제거할 때 일부 브라우저·보조 기술 조합의 목록 의미를 보완하기 위해 예제에는 `role="list"`를 명시했다. 실제 타깃 환경에서도 의미가 전달되는지 확인한다. HTML의 순서는 CSS 배치 이후에도 키보드·보조 기술 탐색의 중요한 기준으로 남는다.

---

## 11. 반응형 설계는 기기 이름보다 콘텐츠의 한계를 본다

미디어 쿼리는 viewport 폭 같은 환경 조건에 따라 적용할 스타일을 고른다. 처음부터 고정된 데스크톱 폭을 만들고 축소하기보다 좁은 화면의 읽기 흐름을 먼저 잡고, 충분한 공간이 생기면 열을 늘릴 수 있다. 분기점은 특정 기종 이름보다 콘텐츠가 더 이상 편하게 읽히지 않는 지점에서 정한다.

`max-width`, 유연한 트랙, 줄바꿈을 조합하고 이미지에는 필요에 따라 `max-width: 100%; height: auto`를 적용한다. `rem`을 썼다는 사실만으로 확대 접근성이 자동 보장되지는 않는다. 글자 확대, 긴 번역문, 시스템 글꼴 차이에서 잘리거나 겹치지 않는지 실제로 확인한다.

---

## 12. Responsive Design: 바뀌는 환경에서 관계를 보존한다

### viewport·내용 폭·단위를 구별한다

viewport 메타데이터는 모바일의 레이아웃 viewport 설정과 관련된다. 반응형 CSS는 그 환경에서 폭을 유연하게 배분하고 필요한 시점에 구조를 바꾼다. viewport 설정 한 줄만으로 모든 요소가 반응형이 되는 것은 아니다.

px은 화면 장치의 물리 픽셀과 항상 일대일인 단위가 아니며 rem은 루트 글꼴 크기에 의존한다. 백분율은 속성과 containing block에 따라 기준이 다르다. 단위 이름을 정답처럼 고르기보다 무엇에 비례해야 하는지 결정한다.

### 확대·긴 글·입력 장치를 포함한다

카드 제목 한 줄을 전제로 고정 높이를 정하면 번역이나 확대에서 잘릴 수 있다. 마우스 hover에만 정보를 숨겨두면 터치·키보드에서는 접근이 어려울 수 있다. 반응형 설계에는 폭뿐 아니라 입력 방식과 사용자 설정도 포함된다.

레이아웃을 바꿔도 콘텐츠의 의미와 DOM 읽기 순서는 유지한다. 모바일에서 중요한 정보를 display none으로 없앨 때는 같은 기능을 사용할 다른 경로가 있는지 확인한다.

### 성능 개선의 판단 기준

뷰포트 변경·스크롤·입력 시 실제 레이아웃과 페인트 비용을 측정한다. 고정 위치 요소와 큰 그림자·필터 같은 표현의 비용은 구현과 장치에 따라 다르다. “Flex보다 Grid가 빠르다” 같은 전역 결론보다 동일한 요구를 구현한 화면의 병목을 비교한다.

개선 뒤에는 화면 폭뿐 아니라 문장 길이, 최소 글꼴 설정, 확대, 키보드 포커스를 다시 검사한다. 빠르게 그려도 정보가 잘리면 요구를 만족하지 못한다.

---

## 13. 크기 결정을 값과 제약의 문제로 풀기

### width 하나만으로 최종 크기가 결정되지 않는다

width는 특정 축의 선호 크기 지정에 관여한다. min-width·max-width, 내용의 고유 크기, box-sizing, 사용 중인 배치 알고리즘이 함께 결과를 제한한다. 이미지·긴 단어·입력 요소가 들어 있는 상자가 예상보다 줄어들지 않는다면 숫자를 더 작게 쓰기 전에 어떤 최소 크기 제약이 남아 있는지 확인한다.

HTML의 의미와 CSS의 박스 종류도 별개다. article이라고 반드시 특별한 크기 계산을 하는 것은 아니며 div에도 Grid나 Flex를 적용할 수 있다. 적절한 HTML로 읽기·조작 구조를 정하고 그 위에 필요한 배치 문맥을 선택한다. [CSS Sizing](https://www.w3.org/TR/css-sizing-3/)은 선호 크기와 최소·최대·고유 크기를 구별하는 기준이다.

### 실험 1. 테두리 상자의 폭을 직접 비교하기

```html
<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8"><title>박스 크기 비교</title>
  <style>
    .box { width: 200px; padding: 20px; border: 5px solid; margin: 12px; }
    .content { box-sizing: content-box; }
    .border { box-sizing: border-box; }
  </style>
</head>
<body>
  <div class="box content">content-box</div>
  <div class="box border">border-box</div>
  <script>
    for (const box of document.querySelectorAll('.box')) {
      console.log(box.className, box.getBoundingClientRect().width);
    }
  </script>
</body>
</html>
```

첫 테두리 상자의 폭은 250px, 둘째는 200px이다. 좌우 margin은 두 측정값에 들어가지 않는다. 첫 요소는 내용 200+padding 40+border 10이고, 둘째는 전체 200 안에서 내용 폭이 150이 된다. 이 실험에는 transform을 사용하지 않았으므로 시각적인 테두리 사각형을 단순하게 비교할 수 있다.

측정 API를 배우기 위한 예제는 아니다. 콘솔 값은 CSS 계산 결과를 관찰하는 보조 수단이며, DevTools Box Model에서도 같은 영역을 나누어 확인한다. 이후 transform을 추가하면 시각 사각형과 레이아웃 크기의 관계가 달라질 수 있으므로 같은 측정을 무조건 모든 크기의 정의로 삼지 않는다.

### 콘텐츠가 지정 크기를 넘는 경우

상자 폭을 작게 정해도 긴 콘텐츠가 사라지는 것은 아니다. overflow는 넘친 내용을 어떻게 표현·스크롤할지 정하고, 줄바꿈은 문장을 어디서 나눌지 정한다. 상자를 줄이는 일과 내용의 표시 정책은 별개의 결정이다. overflow hidden으로 잘라서 문제를 감추면 링크·입력·포커스 표시까지 잘릴 수 있다.

---

## 14. Flex의 공간 분배를 수치로 추적하기

### grow는 여유 공간을 나누는 비율이다

container가 500px이고 두 아이템의 basis가 각각 100px, grow가 1과 2이며 다른 제약이 없다고 하자. 먼저 기준 크기 합 200px을 제외한 300px이 여유다. grow 합 3으로 나누어 첫째는 100px, 둘째는 200px을 추가로 받아 최종 200px과 300px이 된다.

이것은 최종 폭을 무조건 1:2로 나눈 결과가 아니다. 기준 크기 위에 남는 공간을 나누었기 때문이다. padding·border·gap·min/max 제약이 있으면 계산에 영향을 주므로 다음 실험은 이를 단순화한다. [Flex 길이 해석 알고리즘](https://www.w3.org/TR/css-flexbox-1/#resolve-flexible-lengths)의 의미를 작은 숫자로 읽는 예다.

### 실험 2. grow와 shrink를 분리해서 보기

```html
<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8"><title>Flex 공간 분배</title>
  <style>
    .row { display: flex; margin-block: 16px; background: #eee; }
    .item { min-width: 0; height: 48px; outline: 1px solid; }
    .grow { width: 500px; }
    .grow .a { flex: 1 1 100px; }
    .grow .b { flex: 2 1 100px; }
    .shrink { width: 300px; }
    .shrink .a { flex: 0 1 200px; }
    .shrink .b { flex: 0 3 200px; }
  </style>
</head>
<body>
  <div class="row grow"><div class="item a">A</div><div class="item b">B</div></div>
  <div class="row shrink"><div class="item a">A</div><div class="item b">B</div></div>
  <script>
    for (const row of document.querySelectorAll('.row')) {
      console.log(row.className,
        [...row.children].map((item) => item.getBoundingClientRect().width));
    }
  </script>
</body>
</html>
```

grow 행은 200·300px이다. shrink 행은 기준 합 400px에서 100px이 부족하다. shrink의 분배에는 기준 크기를 곱한 축소 가중치가 관여한다. 이 경우 두 basis가 같아 가중치가 1:3이므로 각각 25·75px이 줄어 최종 175·125px이다. 기준 크기가 다르면 shrink 숫자 비율만 보고 같은 결론을 내릴 수 없다.

### 최소·최대 제약이 걸리면 다시 나눈다

앞 실험의 a에 min-width를 추가하면 그 한계 아래로 축소하지 못하고 다른 항목이 더 많은 부족분을 처리할 수 있다. 알고리즘은 단순한 한 번의 나눗셈이 아니라 제한에 걸린 아이템을 확정하고 남은 공간을 처리하는 과정을 포함한다. 복잡한 결과를 외우기보다 Flex 오버레이와 각 아이템의 min/max·basis를 확인한다.

### 기본 auto 최소 크기가 긴 글과 만나는 경우

Flex 아이템의 min-width auto는 내용 기반 최소 크기가 관여할 수 있다. 긴 URL이나 줄바꿈 없는 이름이 있으면 flex-shrink가 있어도 부모를 넘길 수 있다. 필요한 아이템에 min-width 0을 주는 것과 내용에 줄바꿈·스크롤을 주는 것은 함께 판단해야 한다. 하나는 아이템 크기 제한을 바꾸고 다른 하나는 남은 내용의 표현을 결정한다.

### 여러 줄의 Flex와 Grid 선택

Flex의 wrap은 아이템을 여러 줄로 나누지만 각 줄의 공간 분배는 독립적이다. 두 번째 줄의 두 칸을 첫 번째 줄의 세 칸과 같은 열 경계로 맞추려면 추가 제약이 필요하다. 행과 열을 공유하는 것이 핵심 요구라면 Grid로 트랙을 정의하는 편이 모델 자체에 맞는다.

---

## 15. Grid의 최소 크기와 남는 공간을 나누기

### fr 앞에 이미 배정된 공간을 먼저 뺀다

600px 컨테이너에 gap 20px과 동일한 두 유연한 열이 있으면, 단순한 조건에서 열에 나눌 공간은 580px이다. 따라서 각 열은 290px이 된다. 열 폭 50% 두 개에 gap을 추가하면 총합이 부모를 넘을 수 있는데, 비율 폭과 남는 공간 비율이 다른 이유다.

### 실험 3. 긴 단어와 minmax

```html
<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8"><title>Grid 최소 크기</title>
  <style>
    .grid { width: 360px; display: grid; grid-template-columns: 1fr 1fr;
            gap: 16px; border: 2px solid; margin-block: 20px; }
    .grid > div { background: #e3eef8; }
    .zero { grid-template-columns: repeat(2, minmax(0, 1fr)); }
    .wrap > div { overflow-wrap: anywhere; }
  </style>
</head>
<body>
  <div class="grid"><div>ABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZ</div><div>설명</div></div>
  <div class="grid zero"><div>ABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZ</div><div>설명</div></div>
  <div class="grid zero wrap"><div>ABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZ</div><div>설명</div></div>
</body>
</html>
```

첫 예제에서는 자동 최소 크기와 내용의 고유 크기 때문에 기대한 동일한 좁은 두 열을 얻지 못할 수 있다. 두 번째는 열의 최소를 0으로 두어 트랙을 줄이지만 글자는 여전히 넘칠 수 있다. 세 번째는 긴 문자열을 나눌 수 있게 하여 내용의 표현까지 해결한다. 정확한 넘침 정도는 글꼴과 장치에 따라 달라도, 트랙 크기와 콘텐츠 넘침의 구분은 유지된다.

Grid 아이템의 자동 최소 크기에는 overflow·트랙의 최소 함수·걸쳐 있는 트랙 등의 조건이 있다. 따라서 모든 Grid 아이템에 “항상 내용의 min-content가 강제된다”고 일반화하지 않는다. [자동 최소 크기의 조건](https://www.w3.org/TR/css-grid-1/#min-size-auto)을 확인하고 실제 아이템이 어떤 조건인지 살핀다.

### 명시 트랙과 암시 트랙

선언한 열·행은 명시 그리드의 일부다. 더 많은 아이템을 배치하거나 범위를 벗어난 라인을 지정하면 암시 트랙이 생길 수 있다. 예상보다 행이 늘어날 때는 항목 수만 보지 말고 grid-row·grid-column과 자동 배치 규칙을 확인한다. 임의의 고정 높이로 덮으면 마지막 행의 내용이 잘릴 수 있다.

### auto-fit과 auto-fill의 비교 기준

둘 다 컨테이너 공간에 맞는 반복 트랙 수를 만드는 데 사용할 수 있다. 차이는 빈 반복 트랙을 접는지 등에 있다. 항목이 적을 때 빈 열을 남길지, 남은 항목이 여유 공간을 채울지가 선택의 기준이다. 예제가 꽉 찬 상태에서만 보면 차이를 놓치므로 항목 수를 1·2·여러 개로 바꾸어 본다.

---

## 16. Position과 겹침을 별개의 기준으로 추적하기

### absolute는 가까운 태그 이름이 아니라 containing block을 찾는다

단순한 경우 positioned 조상을 기준으로 배치하지만 transform 등도 기준을 만들 수 있다. 부모라는 이유만으로 언제나 그 요소가 기준인 것은 아니다. inset의 숫자가 틀린 것처럼 보이면 어느 사각형을 기준으로 해석했는지부터 확인한다. [Position 표준](https://www.w3.org/TR/css-position-3/#def-cb)은 위치 지정의 기준 박스와 흐름을 정의한다.

### 실험 4. 부모 안의 배지와 본문 공간

```html
<article class="card">
  <span class="badge">새 글</span>
  <h2>길이가 달라질 수 있는 제목</h2>
  <p>배지의 위치와 본문이 차지하는 공간을 따로 봅니다.</p>
</article>
<style>
  .card { position: relative; max-width: 25rem; padding: 1rem; border: 1px solid; }
  .badge { position: absolute; top: .5rem; right: .5rem; background: #ffe084; }
  .card h2 { padding-inline-end: 4rem; }
</style>
```

배지는 흐름 밖에 있어 본문이 자동으로 배지를 피해 배치되지 않는다. 제목에 확보한 공간은 이 요구를 명시한 것이다. 글자 확대·다른 언어·배지 문구 변경으로 겹칠 수 있으므로 고정 4rem이 모든 콘텐츠에서 충분하다고 가정하지 않는다. 배지가 본문 흐름에 들어가도 되는 디자인이라면 Flex나 Grid로 관계를 표현하는 방법도 비교한다.

### 실험 5. 자식의 큰 z-index가 다른 그룹을 못 넘는 경우

```html
<div class="one"><span class="child">큰 z-index의 자식</span></div>
<div class="two">다른 그룹</div>
<style>
  .one { position: relative; z-index: 0; height: 50px; }
  .child { position: absolute; z-index: 9999; top: 30px; background: #fcc; }
  .two { position: relative; z-index: 1; background: #cdf; }
</style>
```

one과 two는 서로 다른 겹침 그룹을 만든다. child의 큰 값은 one 내부에서의 순서를 정하고, 부모 그룹 one 자체는 two보다 뒤에 놓인다. 겹친 영역에서는 two가 앞에 올 수 있다. 원인은 숫자가 충분히 크지 않아서가 아니라 비교하는 그룹의 경계다.

### sticky의 세 가지 검사

붙을 기준 inset이 있는가, 어떤 조상이 스크롤 기준을 만드는가, 부모 영역 안에 이동할 공간이 있는가를 순서대로 본다. 조상에 무심코 넣은 overflow가 기대와 다른 스크롤 기준을 만들 수 있다. 부모가 아이템과 같은 높이이면 달라붙는 움직임을 볼 공간이 없을 수도 있다. 레이아웃 오버레이와 조상들의 overflow를 함께 확인한다.

---

## 17. Responsive Design을 콘텐츠의 제약으로 설계하기

### 분기점은 콘텐츠가 깨지는 지점에서 찾는다

카드의 최소 읽기 폭, 버튼 라벨 길이, 주 메뉴의 항목 수를 실제로 넣어 보고 더 이상 한 줄로 쓸 수 없는 지점을 찾는다. 특정 기기의 이름으로만 분기점을 정하면 새 기기나 확대 환경에서 같은 문제가 반복된다. 기본 흐름을 좁은 화면에서 사용할 수 있게 한 뒤 여유가 생길 때 열을 늘리는 접근이 유용하다.

### 실험 6. 확대에도 읽기 순서를 유지하는 구조

```css
.page { max-width: 70rem; margin-inline: auto; padding: 1rem; }
.content { display: grid; grid-template-columns: minmax(0, 1fr); gap: 1.5rem; }
.tools { display: flex; flex-wrap: wrap; gap: .5rem; }
@media (min-width: 55rem) {
  .content { grid-template-columns: minmax(0, 2fr) minmax(14rem, 1fr); }
}
```

HTML에서는 주요 내용 뒤에 보조 정보를 두고 위 CSS로 넓을 때 두 열로 표현할 수 있다. 좁거나 확대된 환경에서 한 열로 돌아가도 DOM의 읽기·키보드 순서는 자연스럽게 이어진다. 중요한 기능을 order나 시각 좌표로만 앞세워 보이는 순서와 탐색 순서를 충돌시키지 않는다.

### 가로 넘침을 무조건 숨기지 않는다

표처럼 관계를 유지해야 하는 콘텐츠에는 의도한 스크롤 영역이 필요할 수 있고, 일반 본문에는 유연한 줄바꿈이 맞을 수 있다. body 전체에 overflow-x hidden을 붙여 원인을 감추면 포커스·그림자·실제 필요한 콘텐츠가 사라질 수 있다. 어떤 요소가 너비를 밀었는지 찾아 해결한다.

### 검증 결과를 남기는 방법

최소·최대 폭만 보지 말고 그 사이에서 제목이 가장 길어지는 상황, 버튼이 줄바꿈되는 순간, 이미지가 없는 상태, 브라우저 확대, 키보드 포커스를 시험한다. 느린 장치에서는 Performance에서 변경 구간의 레이아웃·페인트 비용을 확인한다. 수정 전후를 같은 콘텐츠·동작으로 비교해야 개선의 효과와 새 비용을 판단할 수 있다.

---

## 18. 흔한 문제와 검증 순서

너비가 넘치면 Box Model 패널에서 content·padding·border를 확인하고, Flex·Grid 오버레이에서 트랙과 최소 크기를 확인한다. 긴 단어 때문에 줄어들지 않는 Flex 아이템은 필요한 요소의 `min-width: 0`과 줄바꿈 정책을 검토한다. absolute 요소가 엉뚱한 곳에 있으면 offset 숫자보다 기준 박스부터 찾는다.

`order`나 Grid 배치로 시각 순서를 바꾸어 키보드 순서와 다르게 만들지 않는다. 좁은 viewport, 브라우저 확대, 긴 콘텐츠, 키보드 포커스를 함께 점검한다. 레이아웃 성능은 실제 변경 빈도와 문서 크기에 따라 달라지므로 Performance에서 비용을 확인한다. 반복적인 기하 측정과 스타일 변경의 세부 비용은 이후 성능 학습에서 다룬다.

---

## 19. 핵심 정리

<callout icon="✅" color="green_bg">
박스 모델은 상자의 크기, Flex와 Grid는 상자 사이의 관계, Position은 별도의 위치 기준, 반응형 규칙은 환경에 따른 배치 변화를 다룬다. 읽기와 조작 순서를 보존하면서 이들을 조합한다.
</callout>

---

## 출처

- [W3C — CSS Box Model](https://www.w3.org/TR/css-box-3/)
- [W3C — Flexbox](https://www.w3.org/TR/css-flexbox-1/)
- [W3C — Grid](https://www.w3.org/TR/css-grid-1/)
- [W3C — Positioned Layout](https://www.w3.org/TR/css-position-3/)
- [W3C — Media Queries](https://www.w3.org/TR/mediaqueries-4/)
- [W3C — CSS Sizing](https://www.w3.org/TR/css-sizing-3/)
- [W3C — CSS Overflow](https://www.w3.org/TR/css-overflow-3/)

### 이번 설명의 근거와 적용 범위

- [W3C — CSS Sizing](https://www.w3.org/TR/css-sizing-3/): 선호 크기·최소/최대 크기·고유 크기를 구별하는 근거.
- [W3C — Flexible Length Resolution](https://www.w3.org/TR/css-flexbox-1/#resolve-flexible-lengths): grow·shrink와 기준 크기·제약에 따른 분배.
- [W3C — Grid Automatic Minimum Size](https://www.w3.org/TR/css-grid-1/#min-size-auto): 긴 콘텐츠가 트랙 크기에 영향을 주는 조건.
- [W3C — Containing Blocks](https://www.w3.org/TR/css-position-3/#def-cb): absolute·fixed·sticky의 기준과 흐름.
- [MDN — Stacking Context](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Positioned_layout/Stacking_context): 겹침 그룹과 자식 z-index의 비교 범위.
- [W3C WAI — Reflow 이해](https://www.w3.org/WAI/WCAG22/Understanding/reflow.html): 확대·좁은 폭에서 정보와 기능을 유지하는 접근성 기준의 설명.

---

## 20. 이해도 확인 질문

<callout icon="📝" color="purple_bg">
코드와 상황의 결과를 먼저 예측하고, 근거와 검증 방법을 함께 설명한다. 답과 피드백은 대화에서만 다룬다.
</callout>

1. width 240px, padding 16px, border 2px일 때 content-box와 border-box의 테두리 상자 폭을 계산하라.
2. 카드 목록과 카드 내부 도구 모음에 각각 Grid와 Flex를 고르는 근거를 설명하라.
3. 배지가 페이지 모서리로 이동했거나 sticky가 작동하지 않는다. 각각 어떤 기준과 조상을 확인하겠는가?
4. 화면에서 보기 좋게 순서를 바꾸었지만 Tab 이동이 어색하다. CSS와 DOM의 관계로 원인을 설명하라.
