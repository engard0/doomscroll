# Doomscroll widget (Hermes desktop)

When the user wants the real scroll experience (or after 3+ markdown pages), build the widget instead of another list.

1. Write `doomscroll.html` in the session workspace with the template below.
2. Open it with a line containing only `::preview{file="doomscroll.html"}`.
3. Update the same file on follow-ups; never spawn copies.

Template: self-fetching infinite scroll over HN Algolia (CORS-enabled, no key). Do not set your own background or font — use the injected theme vars. Reddit JSON has no stable CORS from the widget; that is what the agent button is for.

```html
<!doctype html>
<html>
<head><meta charset="utf-8"><style>
html,body{margin:0;font:14px/1.5 ui-sans-serif,system-ui;color:var(--foreground)}
a{color:var(--accent);text-decoration:none}a:hover{text-decoration:underline}
.item{padding:8px 0;border-bottom:1px solid var(--border)}
.meta{color:var(--muted-foreground);font-size:12px}
#sentinel{padding:14px;text-align:center;color:var(--muted-foreground)}
button{background:var(--accent);color:var(--card);border:0;border-radius:8px;padding:6px 12px;margin:10px 0}
</style></head>
<body>
<div id="feed"></div>
<div id="sentinel">cargando…</div>
<script>
const feed=document.getElementById('feed'),sentinel=document.getElementById('sentinel');
let page=0,seen=new Set(),busy=false;
async function load(){
  if(busy)return;busy=true;sentinel.textContent='cargando…';
  const r=await fetch(`https://hn.algolia.com/api/v1/search_by_date?tags=story&hitsPerPage=30&page=${page}`);
  const d=await r.json();
  for(const h of d.hits){
    if(!h.title||seen.has(h.objectID))continue;seen.add(h.objectID);
    const el=document.createElement('div');el.className='item';
    el.innerHTML=`<a href="${h.url||'https://news.ycombinator.com/item?id='+h.objectID}" target="_blank">${h.title}</a>`+
      `<div class="meta">HN · ▲${h.points||0} · 💬${h.num_comments||0}</div>`;
    feed.appendChild(el);
  }
  page++;busy=false;sentinel.textContent=d.hits.length?'↓':'fin del feed';
}
new IntersectionObserver(e=>{if(e[0].isIntersecting)load()}).observe(sentinel);
</script>
<button data-hermes-send="doomscroll, trae la siguiente tanda con Reddit y noticias">pedir más al agente</button>
</body></html>
```

The `data-hermes-send` button sends its text as a user turn — answer it by refreshing the widget file, not with prose.
