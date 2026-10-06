<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Netflix Clone DevSecOps Architecture</title>
<style>
  body { font-family: -apple-system, "Segoe UI", Roboto, Helvetica, Arial, sans-serif; background: #ffffff; color: #0f172a; margin: 0; padding: 24px; }
  .wrap { max-width: 940px; margin: 0 auto; }
  h1 { font-size: 24px; margin: 0 0 4px; }
  p.sub { margin: 0 0 16px; color: #475569; font-size: 14px; }
  svg { width: 100%; height: auto; display: block; border: 1px solid #e2e8f0; border-radius: 12px; background: #ffffff; }
  .t  { font-size: 14px; font-weight: 600; text-anchor: middle; dominant-baseline: central; }
  .s  { font-size: 12px; text-anchor: middle; dominant-baseline: central; }
  .b  { font-size: 11px; font-weight: 700; text-anchor: middle; dominant-baseline: central; fill: #ffffff; }
  .lb { font-size: 12px; fill: #475569; dominant-baseline: central; }
  .ln { stroke: #475569; stroke-width: 2; fill: none; marker-end: url(#ah); }
  .dl { stroke: #475569; stroke-width: 2; fill: none; stroke-dasharray: 6 4; marker-end: url(#ah); }
  .dl2 { stroke: #475569; stroke-width: 2; fill: none; stroke-dasharray: 6 4; marker-end: url(#ah); marker-start: url(#ah); }
  .git { fill: #dbeafe; stroke: #1d4ed8; stroke-width: 2; } .git-t { fill: #0b2a5b; }
  .scan { fill: #dcfce7; stroke: #15803d; stroke-width: 2; } .scan-t { fill: #052e16; }
  .build { fill: #fef9c3; stroke: #ca8a04; stroke-width: 2; } .build-t { fill: #422006; }
  .reg { fill: #f3e8ff; stroke: #7e22ce; stroke-width: 2; } .reg-t { fill: #3b0764; }
  .cd { fill: #fee2e2; stroke: #dc2626; stroke-width: 2; } .cd-t { fill: #450a0a; }
  .app { fill: #cffafe; stroke: #0e7490; stroke-width: 2; } .app-t { fill: #083344; }
  .usr { fill: #f3f4f6; stroke: #4b5563; stroke-width: 2; } .usr-t { fill: #111827; }
  .g-local { fill: #f8fafc; stroke: #475569; stroke-width: 2; }
  .g-sq { fill: #f0fdf4; stroke: #15803d; stroke-width: 2; }
  .g-jk { fill: #fffbeb; stroke: #ca8a04; stroke-width: 2; }
  .g-eks { fill: #ecfeff; stroke: #0e7490; stroke-width: 2; }
  .bd { fill: #1e293b; }
  .legend { display: grid; grid-template-columns: repeat(auto-fit, minmax(260px, 1fr)); gap: 8px 24px; margin: 16px 0 0; padding: 0; list-style: none; font-size: 14px; }
  .legend li { display: flex; gap: 8px; align-items: baseline; }
  .legend b { display: inline-flex; width: 20px; height: 20px; border-radius: 50%; background: #1e293b; color: #fff; font-size: 11px; align-items: center; justify-content: center; flex: none; }
  .key { display: flex; flex-wrap: wrap; gap: 8px 18px; margin: 16px 0 0; font-size: 13px; color: #334155; }
  .key span::before { content: ""; display: inline-block; width: 12px; height: 12px; border-radius: 3px; margin-right: 6px; vertical-align: -1px; border: 2px solid; }
  .k1::before { background: #dbeafe; border-color: #1d4ed8; } .k2::before { background: #dcfce7; border-color: #15803d; }
  .k3::before { background: #fef9c3; border-color: #ca8a04; } .k4::before { background: #f3e8ff; border-color: #7e22ce; }
  .k5::before { background: #fee2e2; border-color: #dc2626; } .k6::before { background: #cffafe; border-color: #0e7490; }
  @media print { body { padding: 0; } }
</style>
</head>
<body>
<div class="wrap">
  <h1>Netflix Clone: DevSecOps Architecture</h1>
  <p class="sub">CI on Docker Desktop (local machine), GitOps deployment with Argo CD on AWS EKS.</p>

  <svg viewBox="0 0 900 800" role="img" aria-label="DevSecOps architecture: GitHub to Jenkins and SonarQube containers on a local machine, then Docker Hub, then Argo CD and the Netflix app on AWS EKS">
    <defs>
      <marker id="ah" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
        <path d="M2 1L8 5L2 9" fill="none" stroke="#475569" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"/>
      </marker>
    </defs>

    <rect class="g-local" x="250" y="20" width="620" height="374" rx="16"/>
    <text class="t" x="560" y="42" fill="#0f172a">Local machine</text>
    <text class="s" x="560" y="58" fill="#475569">Docker Desktop containers</text>

    <rect class="g-sq" x="270" y="72" width="180" height="92" rx="12"/>
    <text class="t" x="360" y="90" fill="#052e16">SonarQube container</text>
    <rect class="scan" x="285" y="108" width="150" height="44" rx="8"/>
    <text class="t scan-t" x="360" y="126">Quality gate</text>
    <text class="s scan-t" x="360" y="142">Defined in SonarQube</text>

    <rect class="g-jk" x="270" y="184" width="580" height="188" rx="12"/>
    <text class="t" x="560" y="202" fill="#422006">Jenkins container</text>
    <text class="s" x="560" y="217" fill="#713f12">Trivy installed, OWASP plugin</text>

    <rect class="scan" x="280" y="232" width="125" height="44" rx="8"/>
    <text class="t scan-t" x="342" y="251">1. Sonar scan</text><text class="s scan-t" x="342" y="267">Quality gate</text>
    <rect class="scan" x="425" y="232" width="125" height="44" rx="8"/>
    <text class="t scan-t" x="487" y="251">2. OWASP</text><text class="s scan-t" x="487" y="267">Dependency check</text>
    <rect class="scan" x="570" y="232" width="125" height="44" rx="8"/>
    <text class="t scan-t" x="632" y="251">3. Trivy fs</text><text class="s scan-t" x="632" y="267">Files scan</text>
    <rect class="build" x="715" y="232" width="125" height="44" rx="8"/>
    <text class="t build-t" x="777" y="251">4. Docker build</text><text class="s build-t" x="777" y="267">Tagged image</text>

    <rect class="scan" x="715" y="310" width="125" height="44" rx="8"/>
    <text class="t scan-t" x="777" y="329">5. Trivy image</text><text class="s scan-t" x="777" y="345">Blocks HIGH</text>
    <rect class="build" x="570" y="310" width="125" height="44" rx="8"/>
    <text class="t build-t" x="632" y="329">6. Docker push</text><text class="s build-t" x="632" y="345">To Docker Hub</text>
    <rect class="build" x="425" y="310" width="125" height="44" rx="8"/>
    <text class="t build-t" x="487" y="329">7. Update Helm</text><text class="s build-t" x="487" y="345">values.yaml tag</text>

    <line class="ln" x1="405" y1="254" x2="423" y2="254"/>
    <line class="ln" x1="550" y1="254" x2="568" y2="254"/>
    <line class="ln" x1="695" y1="254" x2="713" y2="254"/>
    <line class="ln" x1="777" y1="276" x2="777" y2="308"/>
    <line class="ln" x1="715" y1="332" x2="697" y2="332"/>
    <line class="ln" x1="570" y1="332" x2="552" y2="332"/>
    <line class="dl2" x1="342" y1="166" x2="342" y2="230"/>

    <rect class="git" x="30" y="200" width="180" height="100" rx="10"/>
    <text class="t git-t" x="120" y="222">GitHub</text>
    <text class="s git-t" x="120" y="244">App code</text>
    <text class="s git-t" x="120" y="262">Dockerfile</text>
    <text class="s git-t" x="120" y="280">Helm chart</text>

    <line class="ln" x1="210" y1="240" x2="268" y2="240"/>
    <circle class="bd" cx="239" cy="240" r="10"/><text class="b" x="239" y="240">A</text>

    <path class="ln" d="M423 332 L120 332 L120 302"/>
    <circle class="bd" cx="300" cy="332" r="10"/><text class="b" x="300" y="332">C</text>

    <rect class="reg" x="560" y="440" width="145" height="50" rx="10"/>
    <text class="t reg-t" x="632" y="459">Docker Hub</text><text class="s reg-t" x="632" y="476">Image registry</text>
    <line class="ln" x1="632" y1="356" x2="632" y2="438"/>
    <circle class="bd" cx="632" cy="416" r="10"/><text class="b" x="632" y="416">B</text>

    <rect class="cd" x="720" y="440" width="130" height="50" rx="10"/>
    <text class="t cd-t" x="785" y="459">Helm</text><text class="s cd-t" x="785" y="476">Installed locally</text>
    <line class="dl" x1="785" y1="492" x2="785" y2="538"/>
    <text class="lb" x="775" y="516" text-anchor="end">manual helm commands</text>

    <rect class="g-eks" x="250" y="540" width="620" height="150" rx="16"/>
    <text class="t" x="560" y="562" fill="#083344">AWS EKS cluster</text>
    <text class="s" x="560" y="578" fill="#155e75">Created from CLI</text>

    <rect class="cd" x="270" y="600" width="170" height="56" rx="10"/>
    <text class="t cd-t" x="355" y="620">Argo CD</text><text class="s cd-t" x="355" y="638">GitOps sync</text>
    <rect class="app" x="480" y="600" width="170" height="56" rx="10"/>
    <text class="t app-t" x="565" y="620">Netflix app</text><text class="s app-t" x="565" y="638">Helm release</text>
    <rect class="app" x="690" y="600" width="160" height="56" rx="10"/>
    <text class="t app-t" x="770" y="620">NGINX ingress</text><text class="s app-t" x="770" y="638">Public entry point</text>

    <line class="ln" x1="440" y1="628" x2="478" y2="628"/>
    <circle class="bd" cx="459" cy="628" r="10"/><text class="b" x="459" y="628">E</text>
    <line class="ln" x1="690" y1="628" x2="652" y2="628"/>

    <path class="ln" d="M60 302 L60 628 L268 628"/>
    <circle class="bd" cx="60" cy="470" r="10"/><text class="b" x="60" y="470">D</text>

    <line class="ln" x1="600" y1="492" x2="600" y2="598"/>
    <circle class="bd" cx="600" cy="530" r="10"/><text class="b" x="600" y="530">F</text>

    <rect class="usr" x="700" y="724" width="140" height="50" rx="10"/>
    <text class="t usr-t" x="770" y="743">Users</text><text class="s usr-t" x="770" y="760">Web browser</text>
    <line class="ln" x1="770" y1="722" x2="770" y2="658"/>
    <circle class="bd" cx="770" cy="706" r="10"/><text class="b" x="770" y="706">G</text>
  </svg>

  <ul class="legend">
    <li><b>A</b> Jenkins checks out the main branch from GitHub</li>
    <li><b>B</b> Docker push sends the image to Docker Hub</li>
    <li><b>C</b> New image tag is committed to values.yaml in GitHub</li>
    <li><b>D</b> Argo CD watches the Helm chart in GitHub</li>
    <li><b>E</b> Argo CD deploys the Netflix app</li>
    <li><b>F</b> The cluster pulls the image from Docker Hub</li>
    <li><b>G</b> Users open the app through the NGINX ingress</li>
  </ul>

  <div class="key">
    <span class="k1">Source code</span><span class="k2">Security scans</span><span class="k3">Build and push</span>
    <span class="k4">Registry</span><span class="k5">Deployment tools</span><span class="k6">Running app</span>
  </div>
</div>
</body>
</html>
