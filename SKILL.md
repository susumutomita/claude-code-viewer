---
name: codewiki
description: Generate a DeepWiki-style interactive code documentation site for the current project. Scans the entire codebase, analyzes every major component, and generates a self-contained HTML wiki with live Claude Code chat integration (right-click → ask, chat widget, file reader). Every run is a full rebuild — no stale docs.
---

You are generating a comprehensive, interactive code documentation wiki for the current project.

## Process

Follow these steps exactly:

### Step 1: Scan the project

Use Glob and Read tools to understand the project:

1. List the top-level directory structure
2. Identify the tech stack (package.json, requirements.txt, go.mod, cdk.json, Dockerfile, etc.)
3. Find all major directories containing source code
4. Read README, CLAUDE.md, and key config files

Collect this information before proceeding.

### Step 2: Analyze each component

For each major directory/component you found, read the key files and write a thorough analysis. Cover:

- **Overview**: Purpose and architecture of the entire project (2-3 paragraphs)
- **Per-directory analysis**: For each major directory, explain its role, key files, important functions/classes, and how it connects to other parts
- **Design patterns**: Architecture patterns, security patterns, error handling strategies used
- **Dependencies & API**: External libraries and their purpose, internal module relationships, public APIs/endpoints
- **Developer guide**: Setup, build, test, deploy instructions based on actual scripts/configs

Be thorough. Read actual source files, not just directory listings. Quote specific code when it helps explain concepts.

### Step 3: Generate the static site

Create the file `.codewiki/index.html` using the Write tool. The HTML must:

1. Include ALL analysis content rendered as styled HTML
2. Include the interactive features (see HTML template below)
3. Be completely self-contained (inline CSS/JS, no external dependencies)

### Step 4: Start the server

Run this command to start the server and open the browser:

```bash
python3 -c "
import http.server, json, subprocess, os, sys
from pathlib import Path
from urllib.parse import urlparse, parse_qs

ROOT = os.getcwd()
WIKI = os.path.join(ROOT, '.codewiki')

class H(http.server.SimpleHTTPRequestHandler):
    def __init__(s, *a, **k): super().__init__(*a, directory=WIKI, **k)
    def do_POST(s):
        if s.path=='/api/ask':
            d=json.loads(s.rfile.read(int(s.headers.get('Content-Length',0))))
            q=d.get('question','')
            try:
                r=subprocess.run(['claude','-p','--output-format','text',q],capture_output=True,text=True,timeout=120,cwd=ROOT)
                s._j(200,{'answer':r.stdout.strip() or r.stderr.strip() or '(empty)'})
            except: s._j(500,{'error':'claude error'})
        else: s._j(404,{})
    def do_GET(s):
        if s.path.startswith('/api/file'):
            qs=parse_qs(urlparse(s.path).query);rel=qs.get('path',[''])[0]
            t=Path(ROOT,rel).resolve()
            if not str(t).startswith(ROOT): return s._j(403,{'error':'denied'})
            if not t.is_file(): return s._j(404,{'error':'not found'})
            try:
                c=t.read_text(errors='replace');lines=c.split(chr(10))
                if len(lines)>500: c=chr(10).join(lines[:500])
                s._j(200,{'content':c,'path':rel})
            except Exception as e: s._j(500,{'error':str(e)})
        else: super().do_GET()
    def _j(s,st,d):
        b=json.dumps(d,ensure_ascii=False).encode()
        s.send_response(st);s.send_header('Content-Type','application/json');s.send_header('Content-Length',str(len(b)));s.end_headers();s.wfile.write(b)
    def log_message(s,f,*a):
        if '/api/' in (a[0] if a else ''): super().log_message(f,*a)

print('Code Wiki: http://localhost:8080');
import webbrowser; webbrowser.open('http://localhost:8080')
http.server.HTTPServer(('',8080),H).serve_forever()
" &
```

Tell the user the wiki is ready at http://localhost:8080.

## HTML Template

When generating `.codewiki/index.html`, use this structure. Replace `{{CONTENT}}` placeholders with the actual analysis content you wrote:

```html
<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>PROJECT_NAME — Code Wiki</title>
<style>
:root{--bg:#0d1117;--bg-card:#161b22;--bg-code:#1c2128;--border:#30363d;--text:#e6edf3;--text-muted:#8b949e;--accent:#58a6ff;--accent-green:#3fb950;--accent-orange:#d29922;--accent-red:#f85149;--accent-purple:#bc8cff;--claude-bg:#1a1520;--claude-border:#6e40aa;--sidebar-w:260px}
*{margin:0;padding:0;box-sizing:border-box}
html{scroll-behavior:smooth}
body{font-family:-apple-system,BlinkMacSystemFont,'Segoe UI',Helvetica,Arial,sans-serif;background:var(--bg);color:var(--text);line-height:1.7;font-size:14px}
.sidebar{position:fixed;top:0;left:0;bottom:0;width:var(--sidebar-w);background:var(--bg-card);border-right:1px solid var(--border);overflow-y:auto;z-index:100;padding:20px 0}
.sidebar-logo{padding:0 20px 16px;border-bottom:1px solid var(--border);margin-bottom:12px}
.sidebar-logo h2{font-size:14px;color:var(--accent);font-weight:600}
.sidebar-logo p{font-size:11px;color:var(--text-muted);margin-top:2px}
.nav-section{padding:12px 20px 4px;font-size:10px;font-weight:700;color:var(--text-muted);text-transform:uppercase;letter-spacing:.8px}
.nav-link{display:block;padding:5px 20px 5px 28px;font-size:13px;color:var(--text-muted);text-decoration:none;border-left:2px solid transparent;transition:all .15s}
.nav-link:hover,.nav-link.active{color:var(--text);background:rgba(88,166,255,.06);border-left-color:var(--accent)}
.main{margin-left:var(--sidebar-w);padding:40px 60px 100px;max-width:920px}
h1{font-size:28px;margin-bottom:6px}h1 small{font-size:13px;color:var(--text-muted);font-weight:400;display:block;margin-top:4px}
h2{font-size:22px;margin:48px 0 16px;padding-top:24px;border-top:1px solid var(--border)}
h3{font-size:17px;margin:28px 0 10px;color:var(--accent)}
h4{font-size:14px;margin:20px 0 8px;color:var(--accent-purple)}
p{margin-bottom:10px}li{margin:3px 0 3px 20px}
.card{background:var(--bg-card);border:1px solid var(--border);border-radius:8px;padding:20px;margin:16px 0}
pre{background:var(--bg-code);border:1px solid var(--border);border-radius:6px;padding:16px;overflow-x:auto;font-size:12.5px;line-height:1.5;margin:12px 0}
code{font-family:'SF Mono','Fira Code',monospace;font-size:12.5px}
.inline-code{background:var(--bg-code);border:1px solid var(--border);border-radius:4px;padding:2px 6px;font-size:12px}
.badge{display:inline-block;padding:2px 8px;border-radius:12px;font-size:11px;font-weight:600;margin:2px}
.badge-blue{background:rgba(88,166,255,.15);color:var(--accent)}
.badge-green{background:rgba(63,185,80,.15);color:var(--accent-green)}
.badge-orange{background:rgba(210,153,34,.15);color:var(--accent-orange)}
.badge-purple{background:rgba(188,140,255,.15);color:var(--accent-purple)}
table{width:100%;border-collapse:collapse;margin:12px 0;font-size:13px}
th,td{padding:8px 12px;border:1px solid var(--border);text-align:left}
th{background:var(--bg-card);font-weight:600;color:var(--accent);font-size:12px}
.section-content{background:var(--bg-card);border:1px solid var(--border);border-radius:8px;padding:24px;margin:12px 0}
.file-tree{font-family:'SF Mono',monospace;font-size:12.5px;line-height:1.8}
.file-tree .dir{color:var(--accent);font-weight:600}.file-tree .file{color:var(--text);cursor:pointer}
.file-tree .file:hover{color:var(--accent-purple);text-decoration:underline}
.file-tree li{list-style:none;padding-left:20px;border-left:1px solid var(--border)}.file-tree>li{border-left:none;padding-left:0}
.claude-box{background:var(--claude-bg);border:1px solid var(--claude-border);border-radius:8px;padding:12px 16px;margin:12px 0}
.claude-prompt{background:#0d1117;border:1px solid var(--claude-border);border-radius:6px;padding:10px 100px 10px 12px;font-family:'SF Mono',monospace;font-size:12.5px;color:#e6edf3;display:block;width:100%;position:relative;cursor:pointer;white-space:pre-wrap;word-break:break-word}
.claude-prompt:hover{border-color:#a371f7}
.copy-btn,.ask-btn{position:absolute;top:50%;transform:translateY(-50%);border:none;color:white;border-radius:4px;padding:4px 8px;font-size:10px;cursor:pointer;opacity:.7;transition:opacity .15s}
.copy-btn{right:8px;background:var(--border)}.ask-btn{right:55px;background:var(--claude-border)}
.copy-btn:hover,.ask-btn:hover{opacity:1}
.ctx-item{padding:8px 14px;font-size:13px;cursor:pointer;border-radius:4px;color:var(--text);display:flex;align-items:center}
.ctx-item:hover{background:rgba(110,64,170,.2)}
.chat-msg{margin-bottom:12px;display:flex}.chat-msg.user{justify-content:flex-end}.chat-msg.bot{justify-content:flex-start}
.chat-bubble{max-width:90%;padding:10px 14px;border-radius:12px;font-size:13px;line-height:1.6;word-break:break-word}
.chat-msg.user .chat-bubble{background:var(--claude-border);color:white;border-bottom-right-radius:4px}
.chat-msg.bot .chat-bubble{background:var(--bg-code);color:var(--text);border:1px solid var(--border);border-bottom-left-radius:4px}
.chat-bubble pre{margin:8px 0;padding:10px;border-radius:6px;background:var(--bg);border:1px solid var(--border);overflow-x:auto;font-size:12px}
.typing-indicator{display:inline-flex;gap:4px;padding:4px 0}
.typing-indicator span{width:6px;height:6px;border-radius:50%;background:var(--text-muted);animation:typing 1.2s infinite}
.typing-indicator span:nth-child(2){animation-delay:.2s}.typing-indicator span:nth-child(3){animation-delay:.4s}
@keyframes typing{0%,60%,100%{opacity:.3;transform:translateY(0)}30%{opacity:1;transform:translateY(-4px)}}
.toast{position:fixed;bottom:24px;left:50%;transform:translateX(-50%) translateY(80px);background:var(--accent-green);color:#000;padding:10px 20px;border-radius:8px;font-size:13px;font-weight:600;opacity:0;transition:all .3s;z-index:1000}
.toast.show{transform:translateX(-50%) translateY(0);opacity:1}
.file-reader-panel{background:var(--bg-card);border:1px solid var(--border);border-radius:8px;margin:16px 0;overflow:hidden}
.file-reader-header{padding:10px 16px;background:var(--bg-code);border-bottom:1px solid var(--border);display:flex;gap:8px}
.file-reader-header input{flex:1;background:var(--bg);border:1px solid var(--border);border-radius:6px;padding:6px 10px;color:var(--text);font-size:12px;font-family:'SF Mono',monospace;outline:none}
.file-reader-header input:focus{border-color:var(--accent)}
.file-reader-header button{background:var(--accent);border:none;color:#000;border-radius:6px;padding:6px 14px;font-size:12px;font-weight:600;cursor:pointer}
.file-reader-content{max-height:500px;overflow:auto}
.file-reader-content pre{margin:0;border:none;border-radius:0}
#chatInput:focus{border-color:var(--claude-border)}
@media(max-width:900px){.sidebar{display:none}.main{margin-left:0;padding:20px}}
::-webkit-scrollbar{width:8px}::-webkit-scrollbar-track{background:var(--bg)}::-webkit-scrollbar-thumb{background:var(--border);border-radius:4px}
strong{color:var(--text)}
</style>
</head>
<body>

<!-- SIDEBAR: Generate nav links for each section you create -->
<nav class="sidebar">
  <div class="sidebar-logo"><h2>PROJECT_NAME</h2><p>Code Wiki</p></div>
  <!-- Add nav-link entries for each h2 section -->
</nav>

<div class="main">
<h1>PROJECT_NAME<small>Code Wiki — Generated by Claude Code</small></h1>

<div class="card" style="border-color:var(--accent-green)">
  <div style="font-size:12px;font-weight:600;color:var(--accent-green);margin-bottom:6px">Interactive Features</div>
  <ul style="font-size:13px;color:var(--text-muted);margin-left:16px">
    <li><strong>Chat</strong> — right bottom button to ask anything</li>
    <li><strong>Right-click</strong> — select text, right-click → Ask Claude Code</li>
    <li><strong>Ask</strong> — click Ask button on suggested prompts</li>
    <li><strong>File Reader</strong> — browse source, select & right-click to ask</li>
  </ul>
</div>

<!-- YOUR ANALYSIS CONTENT HERE -->
<!-- Use h2 for major sections, h3 for subsections -->
<!-- Wrap analysis text in <div class="section-content"> -->
<!-- Add claude-box with suggested prompts after each section -->
<!-- Include a File Tree section and File Reader section at the end -->

</div>

<!-- Context Menu (copy exactly) -->
<div id="contextMenu" style="display:none;position:fixed;z-index:9999;background:var(--bg-card);border:1px solid var(--claude-border);border-radius:8px;padding:4px;min-width:240px;box-shadow:0 8px 30px rgba(0,0,0,.5)">
<div id="ctxAsk" class="ctx-item">&#x1F916;&nbsp; Claude Code に聞く</div>
<div id="ctxExplain" class="ctx-item">&#x1F4A1;&nbsp; この部分を解説して</div>
<div id="ctxDeep" class="ctx-item">&#x1F50D;&nbsp; コードを深掘りして</div>
<div style="border-top:1px solid var(--border);margin:4px 0"></div>
<div id="ctxCopy" class="ctx-item" style="color:var(--text-muted)">&#x1F4CB;&nbsp; コピー</div>
</div>

<!-- Chat Widget (copy exactly) -->
<div id="chatWidget" style="position:fixed;bottom:24px;right:24px;z-index:1000;display:flex;flex-direction:column;align-items:flex-end;gap:12px">
<div id="chatPanel" style="display:none;width:480px;max-height:70vh;background:var(--bg-card);border:1px solid var(--claude-border);border-radius:12px;overflow:hidden;box-shadow:0 12px 40px rgba(110,64,170,.3);flex-direction:column">
<div style="padding:14px 16px;background:linear-gradient(135deg,#1a1520,#2d1b4e);border-bottom:1px solid var(--claude-border);display:flex;align-items:center;justify-content:space-between">
<div style="display:flex;align-items:center;gap:8px"><div style="width:10px;height:10px;border-radius:50%;background:var(--accent-green)"></div><span style="font-size:14px;font-weight:600">Claude Code</span><span style="font-size:11px;color:var(--text-muted)">connected</span></div>
<button onclick="toggleChat()" style="background:none;border:none;color:var(--text-muted);cursor:pointer;font-size:18px">&times;</button>
</div>
<div id="chatMessages" style="flex:1;overflow-y:auto;padding:16px;max-height:calc(70vh - 130px);min-height:200px">
<div class="chat-msg bot"><div class="chat-bubble">このプロジェクトについて何でも聞いてください。</div></div>
</div>
<div style="padding:12px;border-top:1px solid var(--border);display:flex;gap:8px">
<input id="chatInput" type="text" placeholder="質問を入力..." style="flex:1;background:var(--bg);border:1px solid var(--border);border-radius:8px;padding:10px 14px;color:var(--text);font-size:13px;outline:none" onkeydown="if(event.key==='Enter')sendChat()">
<button onclick="sendChat()" style="background:var(--claude-border);border:none;color:white;border-radius:8px;padding:10px 16px;cursor:pointer;font-size:13px;font-weight:600">送信</button>
</div></div>
<button id="chatFab" onclick="toggleChat()" style="width:56px;height:56px;border-radius:50%;background:linear-gradient(135deg,#6e40aa,#9b59b6);border:none;color:white;font-size:24px;cursor:pointer;box-shadow:0 4px 20px rgba(110,64,170,.5);transition:transform .2s;display:flex;align-items:center;justify-content:center" onmouseover="this.style.transform='scale(1.1)'" onmouseout="this.style.transform='scale(1)'">&#x1F916;</button>
</div>
<div class="toast" id="toast"></div>

<script>
let chatOpen=false;
function toggleChat(){chatOpen=!chatOpen;const p=document.getElementById('chatPanel'),f=document.getElementById('chatFab');p.style.display=chatOpen?'flex':'none';f.innerHTML=chatOpen?'&times;':'&#x1F916;';if(chatOpen){document.getElementById('chatInput').focus();scrollChat()}}
function scrollChat(){const m=document.getElementById('chatMessages');m.scrollTop=m.scrollHeight}
function escH(s){const d=document.createElement('div');d.textContent=s;return d.innerHTML}
function renderMd(t){return t.replace(/```(\w*)\n([\s\S]*?)```/g,'<pre><code>$2</code></pre>').replace(/`([^`]+)`/g,'<code class="inline-code">$1</code>').replace(/\*\*([^*]+)\*\*/g,'<strong>$1</strong>').replace(/\n/g,'<br>')}
function addMsg(text,role){const m=document.getElementById('chatMessages'),d=document.createElement('div');d.className='chat-msg '+role;const b=document.createElement('div');b.className='chat-bubble';b.innerHTML=role==='bot'?renderMd(text):escH(text);d.appendChild(b);m.appendChild(d);scrollChat()}
function addTyping(){const m=document.getElementById('chatMessages'),d=document.createElement('div');d.className='chat-msg bot';d.id='typingMsg';d.innerHTML='<div class="chat-bubble"><div class="typing-indicator"><span></span><span></span><span></span></div></div>';m.appendChild(d);scrollChat()}
function rmTyping(){const e=document.getElementById('typingMsg');if(e)e.remove()}
async function sendChat(prefill){const input=document.getElementById('chatInput'),q=prefill||input.value.trim();if(!q)return;input.value='';if(!chatOpen)toggleChat();addMsg(q,'user');addTyping();try{const r=await fetch('/api/ask',{method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify({question:q})});rmTyping();if(r.ok){const d=await r.json();addMsg(d.answer||'(empty)','bot')}else{const e=await r.json().catch(()=>({}));addMsg('Error: '+(e.error||r.statusText),'bot')}}catch(e){rmTyping();addMsg('Server not reachable.','bot')}}
let selectedText='';
document.addEventListener('contextmenu',e=>{const sel=window.getSelection().toString().trim();if(!sel)return;e.preventDefault();selectedText=sel;const menu=document.getElementById('contextMenu');menu.style.display='block';let x=e.clientX,y=e.clientY;if(x+menu.offsetWidth>window.innerWidth)x=window.innerWidth-menu.offsetWidth-8;if(y+menu.offsetHeight>window.innerHeight)y=window.innerHeight-menu.offsetHeight-8;menu.style.left=x+'px';menu.style.top=y+'px'});
document.addEventListener('click',()=>{document.getElementById('contextMenu').style.display='none'});
document.getElementById('ctxAsk').onclick=()=>sendChat('"'+selectedText+'"について詳しく教えて');
document.getElementById('ctxExplain').onclick=()=>sendChat('以下を解説して:\n'+selectedText);
document.getElementById('ctxDeep').onclick=()=>sendChat('"'+selectedText+'"に関連するソースコードを読んで実装詳細を教えて');
document.getElementById('ctxCopy').onclick=()=>{navigator.clipboard.writeText(selectedText);showToast('Copied!')};
document.querySelectorAll('.claude-prompt').forEach(el=>{const a=document.createElement('button');a.className='ask-btn';a.textContent='Ask';a.onclick=e=>{e.stopPropagation();sendChat(el.textContent.replace('Copy','').replace('Ask','').trim())};el.appendChild(a)});
async function loadFile(){const p=document.getElementById('filePathInput').value.trim(),c=document.getElementById('fileContent');if(!p)return;c.innerHTML='<pre style="color:var(--text-muted);padding:20px">Loading...</pre>';try{const r=await fetch('/api/file?path='+encodeURIComponent(p));if(r.ok){const d=await r.json();c.innerHTML='<pre><code>'+escH(d.content)+'</code></pre>'}else{const e=await r.json().catch(()=>({}));c.innerHTML='<pre style="color:var(--accent-red);padding:20px">'+escH(e.error||'Not found')+'</pre>'}}catch(e){c.innerHTML='<pre style="color:var(--accent-red);padding:20px">Cannot connect</pre>'}}
document.querySelectorAll('.file-tree .file').forEach(el=>{el.addEventListener('click',()=>{let parts=[el.textContent],node=el.closest('li');while(node.parentElement&&node.parentElement.closest('li')){node=node.parentElement.closest('li');const d=node.querySelector(':scope > .dir');if(d)parts.unshift(d.textContent.replace('/',''))}document.getElementById('filePathInput').value=parts.join('/');loadFile();document.getElementById('file-reader').scrollIntoView({behavior:'smooth'})})});
function copyPrompt(el){navigator.clipboard.writeText(el.textContent.replace('Copy','').replace('Ask','').trim()).then(()=>showToast('Copied!'))}
function showToast(msg){const t=document.getElementById('toast');t.textContent=msg;t.classList.add('show');setTimeout(()=>t.classList.remove('show'),1500)}
const navLinks=document.querySelectorAll('.nav-link');
const obs=new IntersectionObserver(entries=>{entries.forEach(e=>{if(e.isIntersecting){navLinks.forEach(l=>l.classList.remove('active'));const l=document.querySelector('.nav-link[href="#'+e.target.id+'"]');if(l)l.classList.add('active')}})},{rootMargin:'-20% 0px -70% 0px'});
document.querySelectorAll('h2[id],h1[id]').forEach(s=>obs.observe(s));
</script>
</body>
</html>
```

## Important guidelines

- Generate THOROUGH analysis. Read actual source files. Don't just list file names.
- Each section should have specific code quotes and explanations.
- Add `<div class="claude-box">` blocks with suggested follow-up prompts after each section.
- Include a File Tree section at the end with clickable file names.
- Include a File Reader section at the end.
- The HTML MUST be self-contained — everything in one file, no external resources.
- Do NOT do incremental updates. Always generate the complete HTML from scratch.
- Add SVG architecture diagrams where they help explain component relationships.
