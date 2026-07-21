<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>패킷 트레이서 기초 — 조경익</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;500;600;700&family=Noto+Sans+KR:wght@400;500;600;700;900&display=swap" rel="stylesheet">
<style>
  :root{
    --bg:#ffffff; --bg-alt:#f5f6f8; --panel:#f7f8fa; --panel-border:#e2e5ea;
    --text:#1a1f24; --text-dim:#4b5560; --text-faint:#7c8894; --accent:#0f9d8f; --line:#e5e8ec;
    --c-device:#2563eb;
  }
  *{box-sizing:border-box; margin:0; padding:0;}
  body{
    background:radial-gradient(ellipse at top left, rgba(94,234,212,0.06), transparent 45%), var(--bg);
    color:var(--text); font-family:'Noto Sans KR', sans-serif; line-height:1.7; -webkit-font-smoothing:antialiased;
  }
  .mono{font-family:'JetBrains Mono', monospace;}
  a{color:inherit; text-decoration:none;}
  .wrap{max-width:760px; margin:0 auto; padding:0 24px;}

  .topbar{border-bottom:1px solid var(--line); background:rgba(255,255,255,0.9); backdrop-filter:blur(6px);}
  .topbar-inner{display:flex; align-items:center; gap:16px; padding:14px 24px; max-width:760px; margin:0 auto;
    font-family:'JetBrains Mono',monospace; font-size:12.5px; color:var(--text-dim);}
  .topbar-inner a:hover{color:var(--accent);}
  .sep{color:var(--text-faint);}

  .hero{padding:44px 0 28px; border-bottom:1px solid var(--line); margin-bottom:32px;}
  .eyebrow{font-family:'JetBrains Mono',monospace; font-size:12px; color:var(--c-device); letter-spacing:.08em; margin-bottom:14px;}
  .hero h1{font-size:clamp(24px,4vw,32px); font-weight:900; margin-bottom:10px;}
  .hero p{color:var(--text-dim); font-size:14px;}

  .content{padding-bottom:70px;}
  .content h2{font-size:19px; font-weight:700; margin:38px 0 14px; padding-bottom:8px; border-bottom:1px solid var(--line);}
  .content h3{font-size:15px; font-weight:700; margin:22px 0 8px; color:var(--c-device);}
  .content p{font-size:14px; color:var(--text-dim); margin-bottom:10px;}
  .content ul{margin:0 0 14px 20px; font-size:14px; color:var(--text-dim);}
  .content li{margin-bottom:4px;}
  .content li b{color:var(--text);}

  table{width:100%; border-collapse:collapse; margin:14px 0 20px; font-size:13px;}
  th, td{border:1px solid var(--panel-border); padding:8px 12px; text-align:left;}
  th{background:var(--panel); color:var(--text); font-family:'JetBrains Mono',monospace; font-size:12px;}
  td{color:var(--text-dim);}

  pre{background:var(--panel); border:1px solid var(--panel-border); border-radius:8px; padding:14px 16px;
    overflow-x:auto; margin:12px 0; font-family:'JetBrains Mono',monospace; font-size:12.5px; color:var(--accent); line-height:1.6;}

  .img-placeholder{
    display:flex; align-items:center; gap:8px; background:var(--panel); border:1px dashed var(--panel-border);
    border-radius:8px; padding:12px 14px; margin:10px 0; font-size:12px; color:var(--text-faint); font-family:'JetBrains Mono',monospace;
  }

  .note{margin:14px 0; padding:10px 14px; background:var(--panel); border-left:3px solid var(--c-device); border-radius:6px; font-size:13px; color:var(--text-dim);}

  footer{border-top:1px solid var(--line); padding:24px 0 50px; text-align:center; font-family:'JetBrains Mono',monospace; font-size:12px; color:var(--text-faint);}
</style>
</head>
<body>

<div class="topbar">
  <div class="topbar-inner">
    <a href="../../index.html">메인</a><span class="sep">/</span>
    <a href="../../network.html">서버·네트워크 기초</a><span class="sep">/</span>
    <span>패킷 트레이서 기초</span>
  </div>
</div>

<div class="wrap">

  <section class="hero">
    <div class="eyebrow mono">01 · 네트워크 장비 설정</div>
    <h1>패킷 트레이서 기초</h1>
    <p>OSI 3/4계층 개념, 사설 IP 대역, IPv4/IPv6/MAC 주소 비교와 Cisco 장비 기본 명령어 정리</p>
  </section>

  <div class="content">

    <h2>1. 네트워크 기초</h2>

    <h3>OSI 3계층 vs 4계층</h3>
    <ul>
      <li><b>3계층</b> — 어디로 보낼지 결정 (경로 선택)</li>
      <li><b>4계층</b> — 어떻게 보낼지 결정 (전송 방식)</li>
    </ul>
    <table>
      <tr><th>구분</th><th>3계층 (네트워크)</th><th>4계층 (전송)</th></tr>
      <tr><td>역할</td><td>경로 선택</td><td>전송 방식</td></tr>
      <tr><td>주소</td><td>IP 주소</td><td>포트 번호</td></tr>
      <tr><td>장비</td><td>라우터</td><td>없음 (논리적)</td></tr>
      <tr><td>프로토콜</td><td>IP</td><td>TCP / UDP</td></tr>
    </table>

    <h3>사설 IP 대역</h3>
    <ul>
      <li>10.0.0.0 ~ 10.255.255.255</li>
      <li>172.16.0.0 ~ 172.31.255.255</li>
      <li>192.168.0.0 ~ 192.168.255.255</li>
    </ul>
    <p>인터넷 충돌을 막기 위해 따로 빼둔 내부 전용 주소. 192.168 대역은 범위가 상대적으로 작아 실습에서 자주 사용.</p>

    <h3>IPv4 / IPv6 / MAC 주소 비교</h3>
    <table>
      <tr><th>구분</th><th>IPv4</th><th>IPv6</th><th>MAC</th></tr>
      <tr><td>비트</td><td>32bit</td><td>128bit</td><td>48bit</td></tr>
      <tr><td>묶음</td><td>4개</td><td>8개</td><td>6개</td></tr>
      <tr><td>형식</td><td>10진수(1~3자리 × 4묶음)</td><td>16진수(4자리 × 8묶음)</td><td>16진수(2자리 × 6묶음)</td></tr>
      <tr><td>구분자</td><td>.</td><td>:</td><td>: / -</td></tr>
    </table>

    <h2>2. 패킷 트레이서</h2>
    <p>Cisco 장비를 기반으로 한 가상 환경에서 네트워크를 구성하고 실습할 수 있는 시뮬레이션 소프트웨어.</p>

    <h3>장비 이름 변경</h3>
    <div class="img-placeholder">🖼 스크린샷 자리 — 장비 이름 변경 화면</div>
    <pre>en
conf t
hostname R-1</pre>
    <p><code class="mono">en → conf t</code>로 설정 모드 진입 후, <code class="mono">hostname [이름]</code>으로 Router → R-1로 변경.</p>

    <h3>모드 이동</h3>
    <div class="img-placeholder">🖼 스크린샷 자리 — 모드 이동 화면</div>
    <ul>
      <li><b>end</b> — 인터페이스 모드에서 바로 관리자 모드로 이동</li>
      <li><b>exit</b> — 한 단계 전 모드로 이동</li>
    </ul>

    <h3>비밀번호 설정 및 암호화</h3>
    <div class="img-placeholder">🖼 스크린샷 자리 — 비밀번호 설정 결과</div>
    <pre>enable password 1234
service password-encryption</pre>
    <p>관리자 모드 비밀번호를 1234로 설정. <code class="mono">show run</code>으로 확인하면 처음엔 비밀번호가 평문으로 노출되는데, <code class="mono">service password-encryption</code> 입력 후 다시 확인하면 암호화되어 저장됨.</p>

  </div>

  <footer>© 2026 조경익 · security learning log</footer>
</div>
</body>
</html>
