---
track: B
notion: https://app.notion.com/p/415178c4afac81b69614de665add7ed7
---

# requestAnimationFrame


<callout icon="📍" color="gray_bg">
**학습 범위**: Track B · 1–3주 · Web Platform · requestAnimationFrame

브라우저의 렌더링 기회와 시간 계산을 연결하여 일정한 의도의 움직임과 수명을 설계한다.
</callout>

<table-of-contents />

## 1. 학습 목표

이 페이지를 마치면 다음을 할 수 있어야 한다.

- 프레임·주사율·타임스탬프·예약 ID의 역할을 구별할 수 있다.
- 일회성 예약을 반복하고 취소·정지·재시작을 관리할 수 있다.
- 진행률과 시간 차 모델로 호출 횟수에 독립적인 이동량을 계산할 수 있다.
- 숨겨진 시간과 동작 감소 설정의 처리 정책을 설명할 수 있다.
- 콜백·측정·스타일 변경과 실제 레이아웃·페인트 비용을 검증할 수 있다.

---

## 2. 이 주제를 왜 배우는가

화면 애니메이션을 일정한 횟수의 타이머 호출로만 만들면 디스플레이 주사율과 실제 호출 지연에 따라 속도가 달라질 수 있다. requestAnimationFrame은 브라우저의 렌더링 기회에 맞춰 시각 갱신 작업을 예약하는 접점이다. 중요한 것은 프레임마다 같은 거리를 움직이는 것이 아니라 시간에 따라 같은 속도로 움직이는 것이다.

---

## 3. 관련 계층과 역할 구분

requestAnimationFrame은 브라우저가 시각 작업을 예약하도록 제공하는 Web API다. JavaScript 콜백은 계산과 DOM·스타일 입력을 바꾸고 브라우저는 이후 필요한 렌더링을 진행한다. API 호출이 계산을 Worker로 옮기거나 페인트 완료를 증명하는 것은 아니다.

CSS가 선언적으로 표현할 수 있는 효과와 매 프레임 코드 계산이 필요한 효과를 구별한다. 상세한 스케줄링 순서는 7–9주 Browser Scheduling에서 확장한다.

---

## 4. 예약·시간·화면 갱신의 관계

`requestAnimationFrame(callback)`은 다음 적절한 렌더링 기회에 콜백을 실행하도록 요청하고 취소용 식별자를 반환한다. 한 번의 요청은 한 번의 콜백이다. 연속 동작을 원하면 콜백에서 다시 예약해야 한다. 이 기능은 자동 반복 타이머가 아니다.

콜백은 고해상도 타임스탬프를 받는다. 이를 이전 호출 시간이나 시작 시간과 비교해 진행량을 계산한다. 호출 빈도는 환경에 따라 달라질 수 있고 숨겨진 탭에서는 일반적으로 중단·제한된다. 정확히 60Hz나 16.7ms 간격이라고 보장하지 않는다. [HTML 애니메이션 프레임 모델](https://html.spec.whatwg.org/multipage/imagebitmap-and-animations.html#animation-frames)을 기준으로 이해한다.

---

## 5. 프레임·시간·예약의 기본 용어

프레임은 화면 갱신의 한 결과를 생각하는 단위이고 주사율은 디스플레이가 갱신할 수 있는 빈도다. JavaScript 콜백의 실행 횟수가 디스플레이 주사율과 언제나 같은 것은 아니다. 부하·탭 상태·전력 정책에 따라 갱신 기회와 실제 실행이 달라질 수 있다.

60Hz의 한 주기 약 16.7ms는 JavaScript가 전부 사용할 수 있는 확정 예산이 아니다. 브라우저는 입력·스타일·레이아웃·페인트 등 다른 작업도 해야 한다. 120Hz라면 주기 자체가 더 짧아질 수 있다. 짧은 루프처럼 보인다는 인상보다 실제 장치의 프레임 시간을 관찰해야 한다.

### rAF는 다음 콜백 예약이지 완료된 페인트의 통지가 아니다

콜백에서 스타일을 변경하면 브라우저가 그 렌더링 기회에 새 표현을 계산할 수 있다. 그러나 콜백에 들어왔다는 사실만으로 이전의 특정 변경이 사용자에게 반드시 표시되었다고 증명할 수는 없다. 두 번 rAF를 중첩하는 패턴도 모든 환경에서 정확한 픽셀 표시 완료 확인 API처럼 사용하지 않는다.

### 반환 ID는 현재 예약을 취소하기 위한 값이다

`cancelAnimationFrame(id)`는 해당 예약을 취소한다. 이미 실행 중인 함수의 계산을 중간에 강제 종료하지는 않는다. 예약 ID를 시간 값이나 영원히 증가하는 업무 식별자로 사용하지 않고, “예약 없음”은 null 같은 별도 상태로 표현한다.

---

## 6. 시간 기반 움직임의 두 가지 모델

### 시작 시점으로부터 진행률 계산

`progress = Math.min((timestamp - start) / duration, 1)`처럼 계산하면 현재 시각의 위치를 직접 구할 수 있다. 프레임을 건너뛰어도 실제 경과 시간에 대응하는 위치가 나온다. 정해진 시간 안에 끝나는 전환에 어울린다.

진행률을 위치에 그대로 곱하면 선형 이동이다. 다른 속도 변화를 원하면 진행률을 변환하는 함수를 적용할 수 있다. 애니메이션 곡선 자체의 설계는 별도 주제이며 이 페이지에서는 시간과 위치 계산의 관계를 먼저 익힌다.

### 이전 시점과의 차이로 속도 누적

```javascript
let previousTime = null;
let distance = 0;
function advance(timestamp) {
  if (previousTime !== null) {
    const seconds = (timestamp - previousTime) / 1000;
    distance += 80 * seconds; // 초당 80px
  }
  previousTime = timestamp;
  return distance;
}
```

이 함수는 시간 차를 계산하는 순수한 일부 흐름이며 스스로 애니메이션을 예약하지 않는다. 위치에 속도×시간을 더하는 모델은 입력이나 시뮬레이션에 따라 속도를 바꾸는 데 쓸 수 있다. 긴 중단 뒤 큰 delta를 허용할지 제한할지는 목적에 따라 정한다.

프레임마다 1을 더하는 카운터와 달리 두 모델은 시간의 단위를 코드에 명시한다. timestamp는 밀리초이므로 초당 속도를 사용하면 1000으로 나누는 등의 단위 변환도 확인한다.

---

## 7. 예제: 프레임 횟수가 아니라 경과 시간으로 이동

```html
<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8"><title>시간 기반 이동</title>
  <style>
    .track { width: 240px; max-width: 100%; border: 1px solid; }
    #box { width: 32px; height: 32px; background: #1457a6; }
  </style>
</head>
<body>
  <h1>시간 기반 이동</h1>
  <button id="start" type="button">처음부터 실행</button>
  <button id="stop" type="button">정지</button>
  <div class="track"><div id="box"></div></div>
  <p id="status" role="status"></p>
  <script>
    const box = document.querySelector('#box');
    const status = document.querySelector('#status');
    const reducedMotion = matchMedia('(prefers-reduced-motion: reduce)');
    let frameId = null;
    let startTime = null;

    function stop() {
      if (frameId !== null) cancelAnimationFrame(frameId);
      frameId = null;
    }

    function tick(timestamp) {
      frameId = null;
      if (startTime === null) startTime = timestamp;
      const elapsed = timestamp - startTime;
      const progress = Math.min(elapsed / 2000, 1);
      const distance = Math.max(0, box.parentElement.clientWidth - box.offsetWidth);
      box.style.transform = `translateX(${distance * progress}px)`;
      if (progress < 1) frameId = requestAnimationFrame(tick);
      else status.textContent = '이동 완료';
    }

    document.querySelector('#start').addEventListener('click', () => {
      stop();
      startTime = null;
      box.style.transform = 'translateX(0)';
      if (reducedMotion.matches) {
        status.textContent = '동작 감소 설정에 따라 이동 효과를 생략했습니다.';
        return;
      }
      status.textContent = '이동 중';
      frameId = requestAnimationFrame(tick);
    });
    document.querySelector('#stop').addEventListener('click', () => {
      stop();
      status.textContent = '정지';
    });
    reducedMotion.addEventListener('change', () => {
      if (reducedMotion.matches) {
        stop();
        status.textContent = '동작 감소 설정에 따라 정지했습니다.';
      }
    });
  </script>
</body>
</html>
```

한 번 실행하면 약 2초의 경과 시간에 따라 오른쪽 끝으로 이동한다. 프레임이 적게 호출되면 중간 위치를 덜 보여줄 수 있지만 매번 1px씩 움직이는 방식처럼 총 이동 시간이 호출 횟수에 종속되지는 않는다. 정지는 다음 예약을 취소하며 현재 위치를 유지한다. 다시 실행은 시작 시간을 초기화한다. 여러 번 실행을 눌러도 이전 예약을 취소하여 중복 루프가 쌓이지 않게 했다.

예제는 매 콜백에서 크기를 읽는다. 작은 실험에서는 이해하기 쉽지만 실무에서는 크기 변경이 없을 때 미리 측정하거나 별도로 갱신하여 반복 측정 비용을 줄일 수 있다. 읽기와 스타일 쓰기를 무분별하게 번갈아 수행하지 않는다.

---

## 8. 숨겨진 시간과 긴 프레임을 어떻게 다룰까

이 예제는 시작 이후 경과 시간을 기준으로 하므로 숨겨진 탭에서 돌아오면 끝 위치로 건너뛸 수 있다. 실제 시간을 따를지, 보이지 않는 동안 진행을 멈출지 목적에 따라 결정한다. 속도 누적 방식에서 큰 시간 차를 제한하면 급격한 점프는 줄지만 실제 시간과의 관계가 달라진다. 어느 방식도 모든 애니메이션의 정답은 아니다.

requestAnimationFrame 안에서 긴 계산을 해도 메인 스레드 비용은 그대로 있다. 예약 API가 계산을 별도 스레드로 옮기거나 실행 시간을 줄여주는 것은 아니다. DOM은 콜백에서 바뀔 수 있지만 그 순간 사용자가 픽셀을 이미 보았다고 보장할 수는 없다.

---

## 9. 애니메이션 수명과 동작 감소 요구

실행·정지·재시작의 상태 전이를 정한다. 정지 후 재개가 같은 위치에서 계속할지 처음부터 시작할지에 따라 시작 시간과 누적 진행량의 관리가 다르다. 버튼을 누를 때마다 새 루프만 시작하면 여러 루프가 같은 스타일을 갱신하여 속도와 비용이 달라질 수 있다.

문서·컴포넌트가 더 이상 효과를 소유하지 않을 때 예약을 취소한다. 화면에서 사라졌다는 것과 JavaScript 참조가 사라졌다는 것도 다르므로 수명과 정리를 명시한다. 반복 효과에는 사용자가 멈출 수 있는 경로와 prefers-reduced-motion에 맞는 대체 표현을 검토한다.

### 성능을 읽기·계산·쓰기의 비용으로 나눈다

위치 측정 API를 호출하고 스타일을 쓰는 일을 계속 교차하면 브라우저가 필요한 레이아웃 계산을 반복할 수 있다. 크기가 고정된 동안에는 측정을 줄이고 여러 읽기·쓰기를 목적에 맞게 묶는다. rAF 안에 넣었다고 모든 강제 레이아웃 문제가 없어지는 것은 아니다.

transform·opacity는 특정 조건에서 비교적 유리할 수 있지만 항상 별도 합성으로만 끝나는 보장은 없다. 레이어를 과도하게 늘리면 메모리 비용도 생긴다. Performance에서 긴 콜백과 레이아웃·페인트를 구별하여 병목이 줄었는지 확인한다.

### 검증 조건을 바꿔 본다

낮은 성능 조건, 다른 주사율, 탭 비활성화, 빠른 반복 클릭, 크기 변경, 동작 감소 설정을 각각 실험한다. 평균 FPS 하나만 보면 간헐적인 긴 프레임과 입력 지연을 놓칠 수 있으므로 긴 구간과 최악 사례도 확인한다.

---

## 10. 시간·프레임·콜백을 같은 숫자로 취급하지 않기

### 화면 주기와 JavaScript 예산

60Hz에서 한 주기가 약 16.7ms라고 해도 그 전부가 사용자 JavaScript의 실행 예산은 아니다. 브라우저는 입력·스타일·레이아웃·그리기 등의 작업도 수행한다. 높은 주사율·장치 부하·백그라운드 정책에 따라 실제 콜백 간격과 표시 결과는 달라질 수 있다. 코드에서 항상 16.7ms를 더하는 모델은 이 차이를 무시한다.

### 콜백 타임스탬프와 Date.now

rAF의 timestamp는 관련 시간 원점을 기준으로 한 고해상도 시간 값이다. 달력상의 현재 시각을 나타내는 Date.now와 바로 빼서 경과 시간을 계산하지 않는다. 같은 종류의 시간 기준을 사용하고 밀리초와 초 단위도 맞춘다. [requestAnimationFrame의 timestamp 정의](https://developer.mozilla.org/en-US/docs/Web/API/Window/requestAnimationFrame)를 기준으로 구현한다.

### 같은 프레임의 여러 콜백

같은 렌더링 기회에서 여러 rAF 콜백을 실행할 때 같은 타임스탬프가 전달될 수 있다. 앞 콜백의 계산 때문에 실제 실행 시각이 늦어졌다고 timestamp가 매번 그만큼 증가하는 것은 아니다. 따라서 콜백 자체의 실행 비용을 재려면 performance.now 같은 측정값을 별도로 사용하고, 애니메이션 진행 시간과 구별한다.

---

## 11. 시간 기반 이동을 수치로 계산하기

### 실험 1. 고정 픽셀 증가와 초당 속도

```javascript
{
  const speed = 80;
  const intervals = [1000 / 60, 1000 / 120];
  for (const milliseconds of intervals) {
    console.log('시간 차:', milliseconds,
      '시간 기반 거리:', speed * milliseconds / 1000);
  }
}
```

60Hz 한 주기의 예에서는 약 1.33px, 120Hz 예에서는 약 0.67px을 이동한다. 둘 다 시간이 같은 1초 동안 진행된다면 약 80px의 속도를 표현한다. 매 콜백마다 1px을 더하면 콜백 횟수에 따라 총 이동량이 달라질 수 있다. 이 계산은 이상적인 간격의 비교이며 실제 rAF 호출 간격이 고정된다는 증거는 아니다.

### 실험 2. 진행률 모델의 경계

```javascript
function positionAt(elapsed, duration, distance) {
  const progress = Math.min(Math.max(elapsed / duration, 0), 1);
  return { progress, x: distance * progress };
}
console.log(positionAt(0, 2000, 240));
console.log(positionAt(1000, 2000, 240));
console.log(positionAt(3000, 2000, 240));
```

각 위치는 시작 0, 절반 120, 완료 240이다. 상한을 두어 마지막 콜백이 예정 종료보다 늦어도 목표를 넘어 계속 이동하지 않게 했다. duration은 양수라는 계약을 전제로 하며 0이나 잘못된 입력을 허용해야 한다면 별도로 처리한다.

### delta 누적과 시작 시간 기준의 차이

시작 시간 기준은 실제 경과 시각에서 현재 위치를 바로 구하기 쉽다. delta 누적은 정지 구간을 제외하거나 속도를 바꾸는 상태를 표현할 때 사용할 수 있다. 긴 시간 차를 임의로 잘라 쓰면 급격한 이동은 줄어도 실제 시간과 진행 시간의 관계가 달라진다. 어떤 시간을 표현할지 요구를 먼저 정한다.

---

## 12. 실행·정지·재개·재시작의 상태를 설계하기

### 상태 전이와 관리할 값

| 상태·사건 | 처리 |
| --- | --- |
| 대기에서 실행 | 이전 timestamp를 비우고 한 번 예약 |
| 실행 중 또 실행 | 중복 루프를 만들지 않음 |
| 실행 중 정지 | 다음 예약을 취소하고 진행량 보관 |
| 정지 후 재개 | 보관한 진행량에서 새 시간 차 누적 |
| 완료 뒤 실행 | 정책에 따라 처음부터 다시 시작 |
| 화면 이탈·동작 감소 | 예약과 효과를 요구에 맞게 정리 |

frameId는 예약의 식별자, elapsed는 보관한 진행 시간, last는 실행 중 이전 콜백 시간이다. 이 셋을 하나의 변수로 표현하려 하면 재시작·취소에서 의미가 섞인다. 예약 취소는 이미 실행 중인 함수의 계산을 강제 중단하지 않는다는 점도 구별한다.

### 실험 3. 정지·재개와 숨겨진 탭 처리

```html
<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8"><title>시간을 보존하는 이동</title>
  <style>
    .track { width: 320px; max-width: 100%; border: 1px solid; }
    #box { width: 32px; height: 32px; background: #1859a8; }
  </style>
</head>
<body>
  <button id="play" type="button">실행·재개</button>
  <button id="pause" type="button">정지</button>
  <button id="reset" type="button">처음으로</button>
  <div class="track"><div id="box"></div></div>
  <p id="status" role="status">대기</p>
  <script>
    const box = document.querySelector('#box');
    const status = document.querySelector('#status');
    const reduced = matchMedia('(prefers-reduced-motion: reduce)');
    const duration = 3000;
    let distance = Math.max(0, box.parentElement.clientWidth - box.offsetWidth);
    let frameId = null;
    let elapsed = 0;
    let last = null;
    let running = false;
    let resumeWhenVisible = false;

    function render() {
      const progress = Math.min(elapsed / duration, 1);
      box.style.transform = `translateX(${distance * progress}px)`;
    }
    function stop(message) {
      running = false;
      if (frameId !== null) cancelAnimationFrame(frameId);
      frameId = null;
      last = null;
      status.textContent = message;
    }
    function tick(timestamp) {
      frameId = null;
      if (!running) return;
      if (last !== null) elapsed += timestamp - last;
      last = timestamp;
      render();
      if (elapsed >= duration) { stop('완료'); return; }
      frameId = requestAnimationFrame(tick);
    }
    function play() {
      if (running) return;
      if (elapsed >= duration) elapsed = 0;
      if (reduced.matches) {
        elapsed = duration;
        render();
        stop('동작 감소 설정에 따라 효과를 생략했습니다.');
        return;
      }
      running = true;
      last = null;
      status.textContent = '이동 중';
      frameId = requestAnimationFrame(tick);
    }
    document.querySelector('#play').addEventListener('click', play);
    document.querySelector('#pause').addEventListener('click', () => {
      resumeWhenVisible = false;
      stop('정지');
    });
    document.querySelector('#reset').addEventListener('click', () => {
      resumeWhenVisible = false;
      stop('대기');
      elapsed = 0;
      distance = Math.max(0, box.parentElement.clientWidth - box.offsetWidth);
      render();
    });
    document.addEventListener('visibilitychange', () => {
      if (document.hidden) {
        resumeWhenVisible = running;
        if (running) stop('탭이 숨겨져 정지했습니다.');
      } else if (resumeWhenVisible) {
        resumeWhenVisible = false;
        play();
      }
    });
    reduced.addEventListener('change', () => {
      if (!reduced.matches) return;
      elapsed = duration;
      render();
      stop('동작 감소 설정으로 효과를 종료했습니다.');
    });
  </script>
</body>
</html>
```

### 시간 변화의 해설

처음 실행한 콜백은 last를 정하고, 다음 콜백부터 실제 timestamp 차이를 elapsed에 더한다. 정지하면 last를 비워 정지 구간의 시간이 다음 실행에 합산되지 않는다. 버튼을 여러 번 눌러도 running 검사로 루프를 추가하지 않는다. 완료 뒤 실행은 처음부터 시작하는 정책을 사용했다.

탭이 숨겨지면 실행 중이었는지 기억하고 정지한다. 다시 보이면 그 경우에만 재개한다. 이는 브라우저가 백그라운드 콜백을 어떻게 제한하든 “보이지 않은 시간은 이 효과의 진행에 포함하지 않는다”는 앱 정책을 분명하게 표현한 것이다. [Page Visibility 문서](https://developer.mozilla.org/en-US/docs/Web/API/Page_Visibility_API)와 예약 API의 수명은 각각 다른 계약이다.

### 적용 범위와 남은 요구

예제는 처음과 reset 시 이동 폭을 측정한다. 실행 중 컨테이너 크기가 바뀌어도 즉시 목표 폭을 다시 계산하는 완성형 반응형 엔진은 아니다. 크기 변화가 요구라면 어느 시점에 새 목표와 현재 진행량을 결합할지 정한다. 또한 컴포넌트 단위로 이 로직을 옮기면 그 수명이 끝날 때 예약·리스너를 정리해야 한다.

---

## 13. 콜백 예약이 의미하는 것과 보장하지 않는 것

### 실험 4. 같은 갱신 기회의 시간 값 비교

```javascript
let firstTime;
requestAnimationFrame((time) => {
  firstTime = time;
  console.log('첫 콜백 시간:', time);
});
requestAnimationFrame((time) => {
  console.log('둘째 콜백 시간:', time);
  console.log('같은 시간 값:', time === firstTime);
});
```

같은 프레임 콜백 묶음으로 처리되면 같은 timestamp를 관찰할 수 있다. 이 코드는 두 함수의 CPU 실행 시간이 0이라는 뜻이 아니다. 애니메이션의 시간 기준과 콜백의 실행 비용이 다르다는 사실을 확인하는 실험이다.

### 콜백 안에서 다시 예약한 작업

현재 콜백에서 requestAnimationFrame을 다시 호출하면 반복을 이어갈 수 있지만 같은 콜백이 재귀 함수 호출처럼 즉시 실행되는 것은 아니다. 브라우저의 다음 처리 기회를 기다린다. [HTML animation frame 알고리즘](https://html.spec.whatwg.org/multipage/imagebitmap-and-animations.html#animation-frames)을 예약·콜백 목록 처리로 읽으면 일회성 API라는 점이 분명해진다.

### rAF를 기다렸다는 것이 페인트 완료 증거는 아니다

rAF 콜백은 화면 갱신에 맞춘 작업 기회이지 특정 픽셀이 사용자 눈에 표시되었다는 확정 통지가 아니다. Promise로 rAF를 감싼 뒤 await했다고 이 계약이 달라지지 않는다. 스크린샷·렌더링 완료·사용자에게 보였다는 지표는 각각 필요한 관찰 기준을 별도로 정의한다.

---

## 14. 성능과 접근성을 검증하는 순서

### 콜백 횟수보다 프레임 시간과 긴 구간

평균적으로 부드러워 보여도 간헐적인 긴 콜백이 입력을 늦출 수 있다. Performance에서 계산·스타일·레이아웃·페인트 비용을 구별하고 낮은 성능 조건에서도 시험한다. rAF 안에 긴 작업을 넣는 것만으로 비용이 분산되거나 Worker로 이동하지 않는다.

### 읽기와 쓰기의 관계

매 프레임 스타일을 쓰고 기하 정보를 다시 읽으면 브라우저가 필요한 배치를 반복 계산할 수 있다. 값이 변하지 않는 동안 측정을 줄이고, 같은 요구를 만족하는 계산을 묶는 방법을 검토한다. transform을 사용해도 모든 장치에서 합성만 일어나고 비용이 항상 작다는 보장은 없으므로 실제 기록으로 확인한다.

### 동작 감소·정지 경로·안내

prefers-reduced-motion을 확인하고 기능의 의미를 유지하는 대체 표현을 정한다. 반복 효과에는 사용자가 중단할 방법을 제공하고 매 프레임 위치를 live 영역에 출력해 안내를 과도하게 만들지 않는다. 시작·정지·완료처럼 사용자에게 의미 있는 상태를 전달한다. [사용자 동작 설정](https://www.w3.org/TR/mediaqueries-5/#prefers-reduced-motion)을 기능 요구로 반영하는 과정이다.

---

## 15. 적용 기준과 디버깅

단순한 선언형 전환은 CSS 애니메이션·트랜지션이 더 적합할 수 있다. JavaScript에서 매 프레임 값을 계산해야 하는 경우 rAF를 검토한다. transform을 바꿨다고 모든 환경에서 합성만 수행된다는 보장도 없으므로 Performance에서 프레임 시간과 레이아웃·페인트 비용을 확인한다.

정지·재시작·화면 이탈 시 정리를 구현하고, 사용자가 동작 감소를 요청하면 효과를 줄이거나 생략한다. 매 프레임 위치를 aria-live로 읽게 하면 안내가 과도해지므로 사용자에게 의미 있는 시작·완료 정도를 전달한다.

---

## 16. 핵심 정리

<callout icon="✅" color="green_bg">
requestAnimationFrame은 렌더링 기회에 맞춘 일회성 예약이다. 타임스탬프로 진행량을 계산하고 중복 예약·중단·탭 비활성화·사용자 동작 설정을 함께 설계한다.
</callout>

---

## 출처

- [WHATWG — Animation Frames](https://html.spec.whatwg.org/multipage/imagebitmap-and-animations.html#animation-frames)
- [MDN — requestAnimationFrame](https://developer.mozilla.org/en-US/docs/Web/API/Window/requestAnimationFrame)
- [W3C — prefers-reduced-motion](https://www.w3.org/TR/mediaqueries-5/#prefers-reduced-motion)

### 이번 설명의 근거와 적용 범위

- [WHATWG — Animation Frames](https://html.spec.whatwg.org/multipage/imagebitmap-and-animations.html#animation-frames): 예약·취소·콜백 목록 처리의 플랫폼 규칙.
- [MDN — requestAnimationFrame](https://developer.mozilla.org/en-US/docs/Web/API/Window/requestAnimationFrame): timestamp·일회성 예약·백그라운드 동작 범위.
- [MDN — cancelAnimationFrame](https://developer.mozilla.org/en-US/docs/Web/API/Window/cancelAnimationFrame): 취소 식별자와 예약 취소의 의미.
- [MDN — Page Visibility](https://developer.mozilla.org/en-US/docs/Web/API/Page_Visibility_API): 문서가 숨겨진 상태와 알림.
- [W3C — prefers-reduced-motion](https://www.w3.org/TR/mediaqueries-5/#prefers-reduced-motion): 동작 감소 선호를 확인하는 표준 조건.

---

## 17. 이해도 확인 질문

<callout icon="📝" color="purple_bg">
코드와 상황의 결과를 먼저 예측하고, 근거와 검증 방법을 함께 설명한다. 답과 피드백은 대화에서만 다룬다.
</callout>

1. 콜백마다 2px 이동하는 코드가 60Hz와 120Hz에서 다르게 움직일 수 있는 이유와 수정 기준을 설명하라.
2. requestAnimationFrame을 한 번 호출했는데 애니메이션이 계속되지 않는다. 반복은 누가 예약해야 하는가?
3. 실행 버튼을 반복해서 누를 때 속도가 빨라진다. 어떤 예약·정리 상태를 점검하겠는가?
4. 숨긴 탭에서 돌아왔을 때 위치가 크게 바뀐다. 실제 시간과 활성 시간 중 무엇을 기준으로 설계할지 설명하라.
