const CATALOG={
"دایره‌ای":[["circle-center","دایره مرکز","circle"],["circle-corner","دایره گوشه","corner"],["circle-edge","دایره لبه","edge"],["circle-two","دو دایره","two"],["circle-out","دایره بیرون","out"],["iris","ایریس","iris"],["radial-lines","خطوط شعاعی","radial"],["spiral","مارپیچ","spiral"],["pulse-circle","ضربانی","pulse"],["ring-expand","حلقه","ring"]],
"خطی":[["bar-horizontal","افقی","scaleX"],["bar-vertical","عمودی","scaleY"],["bar-diagonal","مورب","45°"],["split-halves","دو نیمه","halves"],["split-sides","دو طرف","sides"],["stripes","نوار افقی","stripes"],["stripes-v","نوار عمودی","v-stripes"],["blinds","کرکره","blinds"],["wipe-left","از چپ","wipe"],["wipe-right","از راست","wipe"]],
"هندسی":[["triangle","مثلث","polygon"],["triangle-center","مثلث مرکز","center"],["diamond","الماس","diamond"],["hexagon","شش‌ضلعی","hex"],["checkerboard","شطرنجی","grid"],["grid-fill","شبکه","grid"],["pixelate","پیکسل","pixels"],["cross","صلیب","cross"],["zigzag","زیگزاگ","zig"],["mosaic","موزاییک","mosaic"]],
"حرکتی":[["rotate","چرخش","rotate"],["zoom-in","زوم داخل","zoom"],["zoom-out","زوم بیرون","zoom"],["slide-in","اسلاید","slide"],["flip","سه‌بعدی","flip"],["glitch","گلیچ","glitch"],["whip-pan","ویپ","whip"],["blur-in","بلور","blur"],["scale-flash","فلاش","flash"],["reveal-mask","ماسک","mask"]]
};
const SOUNDS=[["silent","بی‌صدا","none"],["whoosh-soft","ووش نرم","whoosh"],["whoosh-hard","ووش سخت","whoosh"],["impact","ضربه","hit"],["sub-boom","بیس","hit"],["punch","پانچ","hit"],["pop","پاپ","hit"],["snap","اسنپ","hit"],["click","کلیک","hit"],["tick","تیک","hit"],["riser","رایزر","sweep"],["upsweep","سویپ بالا","sweep"],["downsweep","سویپ پایین","sweep"],["reverse","ریورس","sweep"],["air","هوا","tex"],["noise-hit","نویز","tex"],["shimmer","شیمر","tex"],["beep","بیپ","dig"],["glitch","گلیچ","dig"],["cinematic","سینمایی","hit"],["bass-drop","دراپ","hit"]];
const TEXT_STYLES=[["word-slam","اسلم کلمه","کل عبارت با مقیاس و چرخش"],["phrase-fade","محو عبارت","شفافیت + مقیاس کم"],["line-reveal","نمایان خط","از راست برای RTL"],["mask-wipe","ماسک متن","ماسک افقی"],["stagger-words","کلمات جدا","فقط فاصله؛ نه حروف"],["blur-sharp","بلور به وضوح","کل عبارت"],["slide-bounce","اسلاید فنری","از پایین"],["opacity-y","بالا آمدن نرم","ساده و خوانا"]];
const FONTS=[["vazirmatn","Vazirmatn","Vazirmatn-Bold.woff2"],["estedad","Estedad","Estedad-Bold.woff2"],["iransans","IRANSansX","IRANSansX-Bold.woff2"],["shabnam","Shabnam","Shabnam-Bold.woff2"],["sahel","Sahel","Sahel-Bold.woff2"],["inter","Inter","Inter-Bold.woff2"]];
const PLUGINS=[["ScrollTrigger","اتصال اسکرول به تایم‌لاین"],["Flip","جابه‌جایی بین چیدمان‌ها"],["Draggable","کشیدن عنصر"],["MotionPathPlugin","مسیر SVG"],["CustomEase","منحنی سفارشی"],["SplitText","فقط words/lines — هرگز chars برای فارسی"]];

const state={step:0,texts:"سلام\nدنیا",order:[],soundId:"whoosh-soft",bgOn:false,bgFile:"bg.mp3",textStyle:"word-slam",fontId:"vazirmatn",plugins:[],sceneDur:2,wipeDur:1.2,wipeEase:"power2.inOut",wordRatio:.5,wordIn:.35,wordOut:.25,wordEase:"power4.out",wordScale:1.45,wordRotation:6,aiBase:"https://api.openai.com/v1",aiModel:"gpt-4o-mini",aiKey:"",saveKey:false,optimized:""};
try{const s=JSON.parse(localStorage.getItem("tw_state")||"{}");Object.assign(state,s);if(!Array.isArray(state.order))state.order=[];if(!Array.isArray(state.plugins))state.plugins=[];}catch{}
if(localStorage.getItem("tw_ai_key"))state.aiKey=localStorage.getItem("tw_ai_key");

function save(){const{aiKey,optimized,...r}=state;localStorage.setItem("tw_state",JSON.stringify(r));if(state.saveKey&&aiKey)localStorage.setItem("tw_ai_key",aiKey);else if(!state.saveKey)localStorage.removeItem("tw_ai_key");}
function lines(){return state.texts.split(/\n/).map(s=>s.trim()).filter(Boolean);}
function settings(){return{sceneDur:+state.sceneDur,wipeDur:+state.wipeDur,wipeEase:state.wipeEase,wordRatio:+state.wordRatio,wordIn:+state.wordIn,wordOut:+state.wordOut,wordEase:state.wordEase,wordScale:+state.wordScale,wordRotation:+state.wordRotation};}
function fontMeta(){return FONTS.find(f=>f[0]===state.fontId)||FONTS[0];}
function plan(){const t=lines();const texts=t.length?t:state.order.map(()=>"SLAM");const ids=state.order.length?state.order:["circle-center"];return texts.map((text,i)=>({text,transitionId:ids[i%ids.length]}));}
function lookup(id){for(const [cat,items] of Object.entries(CATALOG)){const hit=items.find(r=>r[0]===id);if(hit)return{name:hit[1],tech:hit[2],cat};}return{name:id,tech:id,cat:""};}

function validate(){
  const e=[],w=[],s=settings();
  if(!(s.sceneDur>0))e.push("مدت صحنه باید > ۰ باشد.");
  if(!(s.wipeDur>0))e.push("مدت پرده باید > ۰ باشد.");
  if(s.wipeDur>s.sceneDur)e.push("مدت پرده نباید از صحنه بیشتر باشد.");
  if(s.wipeDur*s.wordRatio+s.wordIn+s.wordOut>s.sceneDur+.001)e.push("ورود/خروج کلمه از طول صحنه بیرون می‌زند.");
  if(!state.order.length)e.push("حداقل یک ترانزیشن انتخاب کنید.");
  if(!lines().length)w.push("متن خالی است → از SLAM استفاده می‌شود.");
  if(state.plugins.includes("SplitText"))w.push("SplitText فقط روی کلمه/خط؛ حروف فارسی را جدا نکنید.");
  if(state.bgOn&&!state.bgFile.trim())e.push("نام فایل صدای زمینه خالی است.");
  return{errors:e,warnings:w};
}

function buildPrompt(){
  const s=settings(),p=plan(),font=fontMeta(),sound=SOUNDS.find(x=>x[0]===state.soundId);
  const style=TEXT_STYLES.find(t=>t[0]===state.textStyle);
  const L=[];
  L.push("# Kinetic slam motion — implementation brief");
  L.push("","Build one self-contained index.html (RTL, Persian-safe). Use GSAP 3.12 CDN. No build step.");
  L.push("","## Hard rules");
  L.push("- Never animate Persian/Arabic text per character. No SplitText chars.");
  L.push("- Deterministic timeline. No Date.now / Math.random in tweens.");
  L.push("- wipeDur <= sceneDur. Word in/out must fit inside the scene.");
  L.push("","## Timing");
  L.push("```js\nconst SETTINGS = "+JSON.stringify(s,null,2)+"\n```");
  L.push(`- Scenes: ${p.length} · Total: ${(p.length*s.sceneDur).toFixed(2)}s`);
  L.push("- Color cycle: #22D3EE, #0A0A0A, #F5F5F5, #0E7490");
  L.push("","## Copy (keep exactly)");
  p.forEach((sc,i)=>{const m=lookup(sc.transitionId);L.push(`${i+1}. t0=${(i*s.sceneDur).toFixed(2)}s | \`${sc.transitionId}\` (${m.tech}) | ${sc.text}`);});
  L.push("","## Text style",`- ${state.textStyle}: ${style?style[2]:""}`,"- Split on spaces only for stagger-words.");
  L.push("","## Font",`- ${font[1]} → ./fonts/${font[2]}`,"- @font-face relative; fallback Tahoma.");
  L.push("","## Sound",`- Hit: ${sound?sound[0]:"silent"} via Web Audio or Howler, onStart of each scene.`);
  if(state.bgOn)L.push(`- Background: <audio src="./${state.bgFile.trim()}" loop> volume ~0.35. File sits beside HTML.`);
  else L.push("- No background bed.");
  L.push("","## Plugins");
  if(!state.plugins.length)L.push("- Core GSAP only.");
  else state.plugins.forEach(n=>{const note=PLUGINS.find(p=>p[0]===n);L.push(`- ${n}: ${note?note[1]:""}`);});
  L.push("","## Deliverable","- index.html + font file + optional bg audio beside it.");
  return L.join("\n");
}

function kineticRuntime(stage,config){
  const COLORS=["#22d3ee","#0a1628","#f4f4f5","#0e7490","#67e8f9","#082f49"];
  const s=config.settings;
  const reduce=window.matchMedia&&window.matchMedia("(prefers-reduced-motion: reduce)").matches;
  stage.innerHTML="";stage.style.cssText="position:relative;overflow:hidden;background:#060b14;width:100%;height:100%";
  const base=document.createElement("div");base.style.cssText="position:absolute;inset:0;background:#0a1628";
  const word=document.createElement("div");word.style.cssText="position:absolute;inset:0;display:flex;align-items:center;justify-content:center;z-index:5;font-weight:800;font-size:clamp(28px,6vw,72px);color:#fff;text-align:center;padding:24px;pointer-events:none;font-family:"+(config.fontFamily||"Tahoma")+",Tahoma,sans-serif";
  const label=document.createElement("div");label.style.cssText="position:absolute;left:10px;bottom:8px;z-index:6;font:12px ui-monospace,monospace;color:#94a3b8";
  stage.append(base,word,label);
  function layer(extra=""){const d=document.createElement("div");d.style.cssText="position:absolute;inset:0;will-change:transform,clip-path,opacity,filter;"+extra;stage.appendChild(d);return d;}
  function paint(id,color){
    const tl=gsap.timeline(),dur=reduce?0.01:s.wipeDur,ease=s.wipeEase==="none"?"none":s.wipeEase;
    const fill=c=>"background:"+c+";";
    if(id==="circle-center"||id==="iris"){const w=layer(fill(color)+"clip-path:circle(0% at 50% 50%)");tl.to(w,{clipPath:"circle(150% at 50% 50%)",duration:dur,ease},0);}
    else if(id==="circle-corner"){const w=layer(fill(color)+"clip-path:circle(0% at 0% 0%)");tl.to(w,{clipPath:"circle(180% at 0% 0%)",duration:dur,ease},0);}
    else if(id==="circle-edge"){const w=layer(fill(color)+"clip-path:circle(0% at 50% 0%)");tl.to(w,{clipPath:"circle(160% at 50% 0%)",duration:dur,ease},0);}
    else if(id==="circle-two"){const a=layer(fill(color)+"clip-path:circle(0% at 0% 50%)"),b=layer(fill(color)+"clip-path:circle(0% at 100% 50%);opacity:.85");tl.to(a,{clipPath:"circle(120% at 0% 50%)",duration:dur,ease},0);tl.to(b,{clipPath:"circle(120% at 100% 50%)",duration:dur,ease},0);}
    else if(id==="circle-out"){const w=layer(fill(color)+"clip-path:circle(140% at 50% 50%)");tl.to(w,{clipPath:"circle(0% at 50% 50%)",duration:dur,ease},0);}
    else if(id==="bar-horizontal"){const w=layer(fill(color)+"transform-origin:left center");gsap.set(w,{scaleX:0});tl.to(w,{scaleX:1,duration:dur,ease},0);}
    else if(id==="bar-vertical"){const w=layer(fill(color)+"transform-origin:center top");gsap.set(w,{scaleY:0});tl.to(w,{scaleY:1,duration:dur,ease},0);}
    else if(id==="bar-diagonal"){const w=layer(fill(color)+"width:200%;height:200%;left:-50%;top:-50%");gsap.set(w,{rotate:45,scaleX:0});tl.to(w,{scaleX:1.4,duration:dur,ease},0);}
    else if(id==="split-halves"){const t=layer(fill(color)+"height:50%;bottom:auto"),b=layer(fill(color)+"height:50%;top:auto");gsap.set(t,{yPercent:-100});gsap.set(b,{yPercent:100});tl.to([t,b],{yPercent:0,duration:dur,ease},0);}
    else if(id==="split-sides"){const l=layer(fill(color)+"width:50%;right:auto"),r=layer(fill(color)+"width:50%;left:auto");gsap.set(l,{xPercent:-100});gsap.set(r,{xPercent:100});tl.to([l,r],{xPercent:0,duration:dur,ease},0);}
    else if(id==="stripes"||id==="blinds"||id==="zigzag"){for(let i=0;i<8;i++){const row=layer(fill(color)+"height:12.5%;top:"+(i*12.5)+"%;bottom:auto");if(id==="blinds"){gsap.set(row,{scaleY:0,transformOrigin:"top"});tl.to(row,{scaleY:1,duration:dur*.6,ease:"power2.out"},i*.03);}else{gsap.set(row,{xPercent:id==="zigzag"&&i%2?100:-100});tl.to(row,{xPercent:0,duration:dur*.7,ease:"power2.out"},i*.03);}}}
    else if(id==="stripes-v"){for(let i=0;i<10;i++){const c=layer(fill(color)+"width:10%;left:"+(i*10)+"%;right:auto");gsap.set(c,{yPercent:-100});tl.to(c,{yPercent:0,duration:dur*.7,ease:"power2.out"},i*.025);}}
    else if(id==="wipe-left"){const w=layer(fill(color)+"clip-path:inset(0 100% 0 0)");tl.to(w,{clipPath:"inset(0 0% 0 0)",duration:dur,ease},0);}
    else if(id==="wipe-right"){const w=layer(fill(color)+"clip-path:inset(0 0 0 100%)");tl.to(w,{clipPath:"inset(0 0 0 0%)",duration:dur,ease},0);}
    else if(id==="triangle"){const w=layer(fill(color)+"clip-path:polygon(0 0,0 0,0 0)");tl.to(w,{clipPath:"polygon(0 0,200% 0,0 200%)",duration:dur,ease},0);}
    else if(id==="diamond"){const w=layer(fill(color)+"clip-path:polygon(50% 50%,50% 50%,50% 50%,50% 50%)");tl.to(w,{clipPath:"polygon(50% -30%,130% 50%,50% 130%,-30% 50%)",duration:dur,ease},0);}
    else if(id==="checkerboard"||id==="grid-fill"||id==="pixelate"||id==="mosaic"){const cols=id==="pixelate"?10:6,rows=id==="pixelate"?6:4,cells=[];for(let r=0;r<rows;r++)for(let c=0;c<cols;c++){const cell=layer(fill(color)+"width:"+(100/cols)+"%;height:"+(100/rows)+"%;left:"+(c*100/cols)+"%;top:"+(r*100/rows)+"%;right:auto;bottom:auto");gsap.set(cell,{scale:0});cells.push(cell);}tl.to(cells,{scale:1,duration:Math.min(.45,dur),stagger:{each:.02,from:id==="pixelate"?"center":"start"},ease:"power2.out"},0);}
    else if(id==="zoom-in"){const w=layer(fill(color));gsap.set(w,{scale:0});tl.to(w,{scale:1,duration:dur,ease:"power3.out"},0);}
    else if(id==="zoom-out"){const w=layer(fill(color));gsap.set(w,{scale:2.4,opacity:0});tl.to(w,{scale:1,opacity:1,duration:dur,ease:"power2.out"},0);}
    else if(id==="rotate"){const w=layer(fill(color));gsap.set(w,{rotate:90,scale:1.8});tl.to(w,{rotate:0,scale:1,duration:dur,ease},0);}
    else if(id==="slide-in"){const w=layer(fill(color));gsap.set(w,{xPercent:-100});tl.to(w,{xPercent:0,duration:dur,ease},0);}
    else if(id==="flip"){stage.style.perspective="800px";const w=layer(fill(color));gsap.set(w,{rotateY:90});tl.to(w,{rotateY:0,duration:dur,ease},0);}
    else if(id==="glitch"){const w=layer(fill(color));gsap.set(w,{opacity:0});tl.to(w,{opacity:1,duration:.05},0).to(w,{x:-10,duration:.05}).to(w,{x:8,duration:.05}).to(w,{x:0,duration:.08});}
    else if(id==="whip-pan"){const w=layer(fill(color));gsap.set(w,{xPercent:-120,filter:"blur(10px)"});tl.to(w,{xPercent:0,filter:"blur(0px)",duration:dur*.6,ease:"power3.out"},0);}
    else if(id==="blur-in"){const w=layer(fill(color));gsap.set(w,{opacity:0,filter:"blur(16px)"});tl.to(w,{opacity:1,filter:"blur(0px)",duration:dur,ease:"power2.out"},0);}
    else if(id==="scale-flash"){const w=layer(fill(color));gsap.set(w,{scale:.4,opacity:0});tl.to(w,{scale:1.08,opacity:1,duration:dur*.45,ease:"power3.out"},0).to(w,{scale:1,duration:dur*.25});}
    else if(id==="reveal-mask"||id==="triangle-center"||id==="hexagon"||id==="cross"||id==="radial-lines"||id==="spiral"||id==="pulse-circle"||id==="ring-expand"){const w=layer(fill(color)+"clip-path:inset(0 100% 0 0)");tl.to(w,{clipPath:"inset(0 0% 0 0)",duration:dur,ease},0);}
    else{const w=layer(fill(color));gsap.set(w,{opacity:0});tl.to(w,{opacity:1,duration:dur*.6},0);}
    return tl;
  }
  function showWords(text){word.innerHTML="";if(config.textStyle==="stagger-words"){text.split(/\s+/).filter(Boolean).forEach(part=>{const span=document.createElement("span");span.textContent=part;span.style.cssText="display:inline-block;margin:0 .25em";word.appendChild(span);});return word.querySelectorAll("span");}word.textContent=text;return word;}
  function wordTween(target){
    const style=config.textStyle||"word-slam",durIn=reduce?0.01:s.wordIn;
    if(style==="phrase-fade")return gsap.fromTo(target,{opacity:0,scale:1.04},{opacity:1,scale:1,duration:durIn,ease:s.wordEase});
    if(style==="line-reveal"||style==="mask-wipe"){gsap.set(target,{opacity:1,clipPath:"inset(0 0 0 100%)"});return gsap.to(target,{clipPath:"inset(0 0 0 0%)",duration:durIn,ease:s.wordEase});}
    if(style==="blur-sharp")return gsap.fromTo(target,{opacity:0,filter:"blur(12px)"},{opacity:1,filter:"blur(0px)",duration:durIn,ease:"power2.out"});
    if(style==="slide-bounce")return gsap.fromTo(target,{opacity:0,y:48},{opacity:1,y:0,duration:durIn,ease:"back.out(1.6)"});
    if(style==="opacity-y")return gsap.fromTo(target,{opacity:0,y:24},{opacity:1,y:0,duration:durIn,ease:"power2.out"});
    if(style==="stagger-words")return gsap.fromTo(target,{opacity:0,y:18},{opacity:1,y:0,duration:durIn,stagger:.06,ease:s.wordEase});
    return gsap.fromTo(target,{opacity:0,scale:s.wordScale,rotation:s.wordRotation},{opacity:1,scale:1,rotation:0,duration:durIn,ease:s.wordEase});
  }
  let audioCtx=null;
  function hit(id){
    if(!id||id==="silent"||reduce)return;
    if(!audioCtx)audioCtx=new(window.AudioContext||window.webkitAudioContext)();
    if(audioCtx.state==="suspended")audioCtx.resume();
    const ac=audioCtx,t0=ac.currentTime,osc=ac.createOscillator(),gain=ac.createGain(),filter=ac.createBiquadFilter();
    osc.connect(filter);filter.connect(gain);gain.connect(ac.destination);gain.gain.setValueAtTime(0.0001,t0);
    const noise=(sec,type,freq,peak)=>{const len=Math.floor(ac.sampleRate*sec);const buf=ac.createBuffer(1,len,ac.sampleRate);const d=buf.getChannelData(0);let x=12345;for(let i=0;i<len;i++){x=(x*16807)%2147483647;d[i]=(x/2147483647)*2-1;}const src=ac.createBufferSource();src.buffer=buf;const g=ac.createGain(),f=ac.createBiquadFilter();f.type=type;f.frequency.value=freq;src.connect(f);f.connect(g);g.connect(ac.destination);g.gain.setValueAtTime(peak,t0);g.gain.exponentialRampToValueAtTime(0.0001,t0+sec);src.start(t0);src.stop(t0+sec);};
    if(id.includes("whoosh")||id==="air"){noise(.4,"bandpass",1200,.28);return;}
    if(id==="noise-hit"){noise(.18,"lowpass",2000,.45);return;}
    if(id==="impact"||id==="punch"||id==="cinematic"){osc.type="sine";osc.frequency.setValueAtTime(180,t0);osc.frequency.exponentialRampToValueAtTime(45,t0+.25);gain.gain.setValueAtTime(.5,t0);gain.gain.exponentialRampToValueAtTime(0.0001,t0+.35);filter.type="lowpass";filter.frequency.value=800;osc.start(t0);osc.stop(t0+.36);noise(.12,"highpass",1000,.2);return;}
    if(id==="sub-boom"||id==="bass-drop"){osc.type="sine";osc.frequency.setValueAtTime(id==="bass-drop"?140:90,t0);osc.frequency.exponentialRampToValueAtTime(32,t0+.45);gain.gain.setValueAtTime(.55,t0);gain.gain.exponentialRampToValueAtTime(0.0001,t0+.5);filter.type="lowpass";filter.frequency.value=300;osc.start(t0);osc.stop(t0+.52);return;}
    if(["pop","snap","click","tick","beep"].includes(id)){osc.type=id==="beep"?"square":"triangle";osc.frequency.setValueAtTime(id==="tick"?1400:880,t0);gain.gain.setValueAtTime(.2,t0);gain.gain.exponentialRampToValueAtTime(0.0001,t0+.1);filter.type="highpass";filter.frequency.value=400;osc.start(t0);osc.stop(t0+.12);return;}
    if(id==="riser"||id==="upsweep"||id==="reverse"){osc.type="sawtooth";osc.frequency.setValueAtTime(180,t0);osc.frequency.exponentialRampToValueAtTime(1600,t0+.45);gain.gain.linearRampToValueAtTime(.18,t0+.4);gain.gain.exponentialRampToValueAtTime(0.0001,t0+.5);filter.type="lowpass";filter.frequency.setValueAtTime(400,t0);filter.frequency.linearRampToValueAtTime(4000,t0+.45);osc.start(t0);osc.stop(t0+.52);return;}
    if(id==="downsweep"){osc.type="sawtooth";osc.frequency.setValueAtTime(1400,t0);osc.frequency.exponentialRampToValueAtTime(120,t0+.4);gain.gain.setValueAtTime(.2,t0);gain.gain.exponentialRampToValueAtTime(0.0001,t0+.42);filter.type="lowpass";filter.frequency.value=2500;osc.start(t0);osc.stop(t0+.44);return;}
    if(id==="shimmer"){osc.type="sine";osc.frequency.setValueAtTime(880,t0);gain.gain.setValueAtTime(.12,t0);gain.gain.exponentialRampToValueAtTime(0.0001,t0+.4);osc.start(t0);osc.stop(t0+.42);return;}
    if(id==="glitch"){noise(.08,"bandpass",2400,.3);osc.type="square";osc.frequency.setValueAtTime(300,t0);gain.gain.setValueAtTime(.15,t0);gain.gain.exponentialRampToValueAtTime(0.0001,t0+.09);osc.start(t0);osc.stop(t0+.1);return;}
    osc.type="sine";osc.frequency.value=220;gain.gain.setValueAtTime(.2,t0);gain.gain.exponentialRampToValueAtTime(0.0001,t0+.2);osc.start(t0);osc.stop(t0+.22);
  }
  const master=gsap.timeline({repeat:config.loop?-1:0});
  (config.scenes||[]).forEach((scene,i)=>{
    const t0=i*s.sceneDur,color=COLORS[i%COLORS.length];
    master.add(()=>{label.textContent=scene.transitionId;},t0);
    master.add(paint(scene.transitionId,color),t0);
    master.to(base,{background:color,duration:.01},t0+s.wipeDur*.45);
    master.add(()=>{wordTween(showWords(scene.text));},t0+s.wipeDur*s.wordRatio);
    master.to(word,{opacity:0,duration:reduce?.01:s.wordOut},t0+s.sceneDur-s.wordOut);
    master.add(()=>{if(config.soundId)hit(config.soundId);},t0);
  });
  if(config.bgFile){const bg=new Audio(config.bgFile);bg.loop=true;bg.volume=.35;master.eventCallback("onStart",()=>bg.play().catch(()=>{}));}
  master.play();return master;
}

function buildExportHtml(){
  const font=fontMeta(),p=plan();
  const config={settings:settings(),scenes:p,textStyle:state.textStyle,fontFamily:font[1],soundId:state.soundId,bgFile:state.bgOn?"./"+state.bgFile.trim():"",loop:true};
  const plugs=state.plugins.map(n=>`<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/${n}.min.js"><\/script>`).join("\n");
  return `<!DOCTYPE html><html lang="fa" dir="rtl"><head><meta charset="UTF-8"><meta name="viewport" content="width=device-width,initial-scale=1"><title>Kinetic Slam</title>
<style>@font-face{font-family:"${font[1]}";src:url("./fonts/${font[2]}") format("woff2");font-weight:700;font-display:swap}
html,body{margin:0;height:100%;background:#000}#stage{position:fixed;inset:0}</style></head><body><div id="stage"></div>
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/gsap.min.js"><\/script>${plugs}
<script>const CONFIG=${JSON.stringify(config)};\n${kineticRuntime.toString()}\nkineticRuntime(document.getElementById("stage"),CONFIG);<\/script></body></html>`;
}

let previewTl=null;
function playCard(id){
  const stage=document.getElementById("previewStage");if(!stage||typeof gsap==="undefined")return;
  if(previewTl)previewTl.kill();
  previewTl=kineticRuntime(stage,{settings:{...settings(),sceneDur:1.6,wipeDur:Math.min(1.1,+state.wipeDur||1.1)},scenes:[{text:lines()[0]||"سلام",transitionId:id}],textStyle:state.textStyle,fontFamily:fontMeta()[1],soundId:"silent",loop:false});
  document.getElementById("previewLabel").textContent=lookup(id).name;
}
function playSound(id){kineticRuntime(document.getElementById("soundSink"),{settings:{sceneDur:.5,wipeDur:.2,wipeEase:"none",wordRatio:.9,wordIn:.05,wordOut:.05,wordEase:"none",wordScale:1,wordRotation:0},scenes:[{text:"",transitionId:"wipe-left"}],textStyle:"opacity-y",soundId:id,loop:false});}

function showIssues(){
  const{errors,warnings}=validate();
  const html=[...errors.map(e=>`<p class="bad">${e}</p>`),...warnings.map(e=>`<p class="warn">${e}</p>`)];
  if(!html.length)html.push(`<p class="ok">زمان‌بندی معتبر است.</p>`);
  const h=html.join("");
  const a=document.getElementById("issues"),b=document.getElementById("issuesFinal");
  if(a)a.innerHTML=h;if(b)b.innerHTML=h;return errors;
}

function render(){
  const names=["متن","ترانزیشن","صدا","سبک و فونت","زمان","پرامپت"];
  const nav=document.getElementById("steps");nav.innerHTML="";
  names.forEach((n,i)=>{const b=document.createElement("button");b.type="button";b.className="step"+(i===state.step?" on":"");b.textContent=i+" "+n;b.onclick=()=>{state.step=i;save();render();};nav.appendChild(b);});
  document.querySelectorAll(".panel").forEach(p=>p.classList.toggle("active",+p.dataset.step===state.step));
  document.getElementById("prev").disabled=state.step===0;
  document.getElementById("next").textContent=state.step===5?"به‌روزرسانی پرامپت":"بعدی";

  if(state.step===1){
    const host=document.getElementById("catalog");host.innerHTML="";
    Object.entries(CATALOG).forEach(([cat,items])=>{
      const box=document.createElement("section");box.className="card";
      box.innerHTML=`<h2>${cat}</h2>`;const grid=document.createElement("div");grid.className="grid";
      items.forEach(([id,name,tech])=>{
        const el=document.createElement("button");el.type="button";el.className="item"+(state.order.includes(id)?" selected":"");
        const idx=state.order.indexOf(id);
        el.innerHTML=`<div class="name">${idx>=0?(idx+1)+". ":""}${name}</div><div class="tech">${tech}</div>`;
        el.onclick=()=>{playCard(id);if(state.order.includes(id))state.order=state.order.filter(x=>x!==id);else state.order.push(id);save();render();playCard(id);};
        grid.appendChild(el);
      });
      box.appendChild(grid);host.appendChild(box);
    });
  }
  if(state.step===2){
    const host=document.getElementById("soundList");host.innerHTML="";
    SOUNDS.forEach(([id,name,group])=>{
      const row=document.createElement("div");row.className="sound"+(state.soundId===id?" selected":"");
      row.innerHTML=`<label><input type="radio" name="sound" ${state.soundId===id?"checked":""}> <b>${name}</b> <span class="tech">${group}</span></label>`;
      const play=document.createElement("button");play.type="button";play.className="ghost";play.textContent="پخش";
      play.onclick=e=>{e.stopPropagation();playSound(id);};
      row.querySelector("input").onchange=()=>{state.soundId=id;save();render();};
      row.appendChild(play);host.appendChild(row);
    });
    document.getElementById("bgOn").checked=state.bgOn;
    document.getElementById("bgFile").value=state.bgFile;
  }
  if(state.step===3){
    const styles=document.getElementById("styleList");styles.innerHTML="";
    TEXT_STYLES.forEach(([id,name,blurb])=>{const b=document.createElement("button");b.type="button";b.className="item"+(state.textStyle===id?" selected":"");b.innerHTML=`<div class="name">${name}</div><div class="tech">${blurb}</div>`;b.onclick=()=>{state.textStyle=id;save();render();};styles.appendChild(b);});
    const fonts=document.getElementById("fontList");fonts.innerHTML="";
    FONTS.forEach(([id,name,file])=>{const b=document.createElement("button");b.type="button";b.className="item"+(state.fontId===id?" selected":"");b.innerHTML=`<div class="name">${name}</div><div class="tech">fonts/${file}</div>`;b.onclick=()=>{state.fontId=id;save();render();};fonts.appendChild(b);});
  }
  if(state.step===4){
    ["sceneDur","wipeDur","wordRatio","wordIn","wordOut","wordScale","wordRotation"].forEach(id=>{const el=document.getElementById(id);if(el)el.value=state[id];});
    document.getElementById("wipeEase").value=state.wipeEase;
    document.getElementById("wordEase").value=state.wordEase;
    const host=document.getElementById("pluginList");host.innerHTML="";
    PLUGINS.forEach(([id,blurb])=>{const row=document.createElement("label");row.className="sound";row.innerHTML=`<span><input type="checkbox" ${state.plugins.includes(id)?"checked":""}> <b>${id}</b><div class="tech">${blurb}</div></span>`;row.querySelector("input").onchange=e=>{if(e.target.checked)state.plugins.push(id);else state.plugins=state.plugins.filter(x=>x!==id);save();};host.appendChild(row);});
    showIssues();
  }
  if(state.step===5){
    const errors=showIssues();
    document.getElementById("promptOut").textContent=state.optimized||buildPrompt();
    document.getElementById("openViewer").disabled=errors.length>0;
    document.getElementById("dlHtml").disabled=errors.length>0;
    document.getElementById("aiBase").value=state.aiBase;
    document.getElementById("aiModel").value=state.aiModel;
    document.getElementById("aiKey").value=state.aiKey;
    document.getElementById("saveKey").checked=state.saveKey;
  }
}

document.getElementById("texts").value=state.texts;
document.getElementById("texts").oninput=e=>{state.texts=e.target.value;save();};
document.getElementById("prev").onclick=()=>{state.step=Math.max(0,state.step-1);save();render();};
document.getElementById("next").onclick=()=>{if(state.step<5)state.step++;state.optimized="";save();render();};
document.getElementById("bgOn").onchange=e=>{state.bgOn=e.target.checked;save();};
document.getElementById("bgFile").oninput=e=>{state.bgFile=e.target.value;save();};
["sceneDur","wipeDur","wordRatio","wordIn","wordOut","wordScale","wordRotation"].forEach(id=>{
  document.getElementById(id).addEventListener("input",e=>{state[id]=+e.target.value;save();showIssues();});
});
document.getElementById("wipeEase").onchange=e=>{state.wipeEase=e.target.value;save();};
document.getElementById("wordEase").onchange=e=>{state.wordEase=e.target.value;save();};

function downloadText(name, text, type) {
  const a = document.createElement("a");
  a.href = URL.createObjectURL(new Blob([text], { type: type || "text/plain" }));
  a.download = name;
  a.click();
  URL.revokeObjectURL(a.href);
}

function exportProject() {
  const { aiKey, optimized, ...rest } = state;
  const project = {
    version: 1,
    exportedAt: new Date().toISOString(),
    state: rest,
    prompt: optimized || buildPrompt(),
  };
  downloadText("kinetic-project.json", JSON.stringify(project, null, 2), "application/json");
}

function importProject(file) {
  const reader = new FileReader();
  reader.onload = () => {
    try {
      const data = JSON.parse(reader.result);
      const incoming = data.state || data;
      Object.assign(state, incoming);
      if (!Array.isArray(state.order)) state.order = [];
      if (!Array.isArray(state.plugins)) state.plugins = [];
      state.optimized = "";
      save();
      const t = document.getElementById("texts");
      if (t) t.value = state.texts || "";
      render();
      alert("پروژه بارگذاری شد.");
    } catch (err) {
      alert("فایل نامعتبر است: " + (err.message || err));
    }
  };
  reader.readAsText(file);
}

document.getElementById("copyPrompt").onclick = async () => {
  try {
    await navigator.clipboard.writeText(document.getElementById("promptOut").textContent);
  } catch {}
  const b = document.getElementById("copyPrompt");
  b.textContent = "کپی شد";
  setTimeout(() => (b.textContent = "کپی پرامپت"), 1200);
};

function packBundle() {
  const { errors } = validate();
  if (errors.length) return null;
  const prompt = state.optimized || buildPrompt();
  const html = buildExportHtml();
  localStorage.setItem(
    "tw_bundle",
    JSON.stringify({
      html,
      prompt,
      bgFile: state.bgOn ? state.bgFile.trim() : "",
    })
  );
  return { html, prompt };
}

document.getElementById("openViewer").onclick = () => {
  if (!packBundle()) return;
  window.open("viewer.html", "_blank");
};

document.getElementById("dlHtml").onclick = () => {
  const b = packBundle();
  if (!b) return;
  downloadText("index.html", b.html, "text/html");
};

document.getElementById("optimize").onclick = async () => {
  const status = document.getElementById("aiStatus");
  if (!state.aiKey.trim()) {
    status.textContent = "کلید API را وارد کنید.";
    return;
  }
  status.textContent = "در حال ارسال…";
  try {
    const res = await fetch(state.aiBase.replace(/\/$/, "") + "/chat/completions", {
      method: "POST",
      headers: {
        Authorization: "Bearer " + state.aiKey.trim(),
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        model: state.aiModel.trim() || "gpt-4o-mini",
        temperature: 0.3,
        messages: [
          {
            role: "system",
            content:
              "Rewrite the motion-design brief for a coding model. Return only the improved English prompt. Keep every user-facing text line exactly. Never suggest per-character animation for Persian/Arabic.",
          },
          { role: "user", content: buildPrompt() },
        ],
      }),
    });
    const data = await res.json();
    if (!res.ok) throw new Error(data.error?.message || res.statusText);
    state.optimized = data.choices?.[0]?.message?.content || "";
    status.textContent = "پرامپت بهینه شد.";
    render();
  } catch (err) {
    status.textContent = "خطا: " + (err.message || err);
  }
};

["aiBase", "aiModel", "aiKey"].forEach((id) => {
  document.getElementById(id).oninput = (e) => {
    state[id === "aiBase" ? "aiBase" : id === "aiModel" ? "aiModel" : "aiKey"] = e.target.value;
    save();
  };
});
document.getElementById("saveKey").onchange = (e) => {
  state.saveKey = e.target.checked;
  save();
};

document.getElementById("helpBtn").onclick = () => document.getElementById("modal").classList.add("on");
document.getElementById("modalClose").onclick = () => document.getElementById("modal").classList.remove("on");
document.getElementById("modal").onclick = (e) => {
  if (e.target.id === "modal") e.currentTarget.classList.remove("on");
};
document.getElementById("helpBody").innerHTML =
  PLUGINS.map(([id, b]) => `<details><summary>${id}</summary><p>${b}</p></details>`).join("") +
  `<details><summary>زمان‌بندی</summary><p>wipeDur ≤ sceneDur. ورود+خروج کلمه باید داخل صحنه بماند.</p></details>` +
  `<details><summary>فارسی</summary><p>حرف‌به‌حرف ممنوع. فقط عبارت، خط، یا کلمه با فاصله.</p></details>` +
  `<details><summary>صدا</summary><p>ضربه با Web Audio. زمینه فایل کنار index.html است.</p></details>`;

const expBtn = document.getElementById("exportProject");
if (expBtn) expBtn.onclick = exportProject;
const impBtn = document.getElementById("importProject");
const impFile = document.getElementById("importFile");
if (impBtn && impFile) {
  impBtn.onclick = () => impFile.click();
  impFile.onchange = () => {
    if (impFile.files && impFile.files[0]) importProject(impFile.files[0]);
    impFile.value = "";
  };
}
const resetBtn = document.getElementById("resetProject");
if (resetBtn)
  resetBtn.onclick = () => {
    if (!confirm("همه تنظیمات پاک شود؟")) return;
    localStorage.removeItem("tw_state");
    location.reload();
  };

render();
