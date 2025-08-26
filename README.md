<!doctype html>

<html lang="ko">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>바이러스 이펙트 페이지(데모)</title>
  <style>
    html,body{height:100%;margin:0;background:#041006;overflow:hidden;font-family:monospace}
    .overlay{position:fixed;inset:0;pointer-events:none}/* 위로 올라가는 초록 텍스트 행 */
.ticker{position:fixed;left:0;right:0;bottom:-10%;height:100%;overflow:visible}
.line{position:absolute;left:50%;transform:translateX(-50%);white-space:nowrap;font-size:18px;color:#8ff28f;text-shadow:0 0 8px rgba(0,255,120,.18);opacity:.95}
@keyframes rise{
  0%{transform:translate(-50%,20vh) scale(1);opacity:0}
  10%{opacity:1}
  100%{transform:translate(-50%,-120vh) scale(.9);opacity:0}
}

/* 깜빡이는 이모지 */
.emoji{position:fixed;font-size:36px;will-change:transform,opacity;pointer-events:none;mix-blend-mode:screen}

/* 화면 중앙 큰 경고 느낌 텍스트 */
.center-msg{position:fixed;inset:0;display:flex;align-items:center;justify-content:center;flex-direction:column;gap:12px;pointer-events:none}
.center-msg .big{font-size:48px;color:#9aff9a;text-shadow:0 6px 30px rgba(0,255,150,.12);opacity:.14}
.center-msg .small{font-size:14px;color:#9aff9a;opacity:.12}

/* 조금의 노이즈 효과 */
.grain{position:fixed;inset:0;opacity:.06;mix-blend-mode:overlay;background-image:url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="200" height="200"><filter id="n"><feTurbulence baseFrequency="0.9" numOctaves="1"/></filter><rect width="100%" height="100%" filter="url(%23n)"/></svg>')}

  </style>
</head>
<body>
  <div class="overlay">
    <div class="ticker" id="ticker"></div>
    <div class="center-msg">
      <div class="big">SYSTEM: COMPROMISED</div>
      <div class="small">(데모 애니메이션 — 실제 바이러스 아님)</div>
    </div>
    <div id="emojiLayer"></div>
    <div class="grain"></div>
  </div>  <script>
    // 설정값
    const LINE_COUNT = 18;           // 동시에 떠있는 위로가는 텍스트 줄 수
    const MIN_INTERVAL = 400;       // 새 라인 생성 최소 간격(ms)
    const MAX_INTERVAL = 1400;      // 최대 간격(ms)
    const LINE_DURATION = 4000;     // 한 줄이 위로 이동하는 시간(ms)

    const ticker = document.getElementById('ticker');
    const emojiLayer = document.getElementById('emojiLayer');

    // 랜덤 텍스트 생성 (초록 글자)
    const CHARS = 'abcdefghijklmnopqrstuvwxyz0123456789!@#$%^&*()[]{}<>/\\|;:\"\' + "'" + ' ';
    function randText(len){
      let s = '';
      for(let i=0;i<len;i++) s += CHARS.charAt(Math.floor(Math.random()*CHARS.length));
      return s;
    }

    // 라인 생성
    function spawnLine(){
      const line = document.createElement('div');
      line.className = 'line';
      const fontSize = 12 + Math.floor(Math.random()*22); // 12~33px
      line.style.fontSize = fontSize + 'px';
      line.style.left = (20 + Math.random()*60) + '%';
      const txtLen = 20 + Math.floor(Math.random()*60);
      line.textContent = randText(txtLen);

      // 애니메이션 타이밍을 랜덤화
      const dur = LINE_DURATION + Math.floor(Math.random()*2000);
      line.style.animation = `rise ${dur}ms linear forwards`;
      // 시작 위치 높낮이 약간씩
      line.style.bottom = (-10 + Math.random()*40) + 'vh';
      ticker.appendChild(line);

      // 제거
      setTimeout(()=>{
        line.remove();
      }, dur + 100);
    }

    // 이모지/문구 순간 표시 (화면에 2초 미만)
    const EMOJIS = ['lol','👾','😈'];
    function spawnEmoji(){
      const span = document.createElement('div');
      span.className = 'emoji';
      // 랜덤 선택: 텍스트(lol) 혹은 이모지
      const pick = EMOJIS[Math.floor(Math.random()*EMOJIS.length)];
      span.textContent = pick;
      // 위치와 크기 랜덤
      const x = Math.random()*100;
      const y = Math.random()*80 + 5;
      span.style.left = x + '%';
      span.style.top = y + '%';
      const size = 20 + Math.floor(Math.random()*72);
      span.style.fontSize = size + 'px';
      span.style.opacity = '0';
      // 애니메이션 (빠르게 나타났다 사라짐, 400~1800ms)
      const life = 500 + Math.random()*1400; // 500ms ~ 1900ms
      span.animate([
        {transform: 'scale(.6) translateY(0px)', opacity: 0},
        {transform: 'scale(1.05) translateY(-6px)', opacity: 1, offset: 0.2},
        {transform: 'scale(.9) translateY(-14px)', opacity: 0}
      ], {duration: life, easing: 'cubic-bezier(.2,.8,.2,1)'});

      emojiLayer.appendChild(span);
      setTimeout(()=> span.remove(), life + 20);
    }

    // 반복 스폰 관리
    let running = true;
    function startSpawning(){
      // 라인 스폰 주기
      (function loopLines(){
        if(!running) return;
        // 유지되는 라인 수를 체크해 새로 추가 여부 결정
        const currentLines = ticker.querySelectorAll('.line').length;
        if(currentLines < LINE_COUNT) spawnLine();
        const interval = MIN_INTERVAL + Math.random()*(MAX_INTERVAL-MIN_INTERVAL);
        setTimeout(loopLines, interval);
      })();

      // 이모지 스폰 주기 (0.5~1.2초 대)
      (function loopEmoji(){
        if(!running) return;
        spawnEmoji();
        const next = 300 + Math.random()*900;
        setTimeout(loopEmoji, next);
      })();
    }

    // 시작
    startSpawning();

    // 키보드로 정지(디버그용) — Escape 누르면 정지
    window.addEventListener('keydown', (e)=>{
      if(e.key === 'Escape'){ running = false; }
    });

    // 모바일/저성능 장치에서 프레임 부담 줄이기: 탭 하면 멈추게
    window.addEventListener('touchstart', ()=>{ running = !running; if(running) startSpawning(); });

    // 페이지 포커스/블러에 따라 자동 정지 — 백그라운드에서 계속 실행되는 걸 방지
    document.addEventListener('visibilitychange', ()=>{
      running = !document.hidden;
      if(running) startSpawning();
    });
  </script></body>
</html>
