<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>OpsBank – Interactive Exam</title>
  <style>
    :root { --bg:#0b1220; --card:#111a2e; --text:#e9eefb; --muted:#b8c3e6; --line:#223055; }
    body{margin:0;font-family:-apple-system,BlinkMacSystemFont,Segoe UI,Roboto,Helvetica,Arial,sans-serif;background:var(--bg);color:var(--text);}
    header{padding:18px 16px 10px; border-bottom:1px solid var(--line);}
    h1{margin:0 0 6px;font-size:20px}
    p{margin:6px 0;color:var(--muted);line-height:1.35}
    .wrap{max-width:980px;margin:0 auto;padding:0 12px 40px}
    .toolbar{display:flex;flex-wrap:wrap;gap:8px;padding:12px 0}
    button{background:#162244;color:var(--text);border:1px solid var(--line);border-radius:10px;padding:10px 12px;font-size:14px;cursor:pointer}
    button:hover{filter:brightness(1.08)}
    button.active{outline:2px solid #6ea8ff}
    .meta{font-size:13px;color:var(--muted);padding:6px 0 12px}
    .qcard{background:var(--card);border:1px solid var(--line);border-radius:14px;padding:14px 14px 10px;margin:10px 0}
    .qnum{font-weight:700;margin:0 0 6px}
    .topic{font-size:13px;color:var(--muted);margin:0 0 10px}
    .stem{margin:0 0 10px;white-space:pre-wrap;line-height:1.4}
    .qimg{max-width:100%;border-radius:12px;border:1px solid var(--line);margin:8px 0 10px}
    .opts{display:grid;gap:8px;margin:0 0 8px}
    .opt{background:#0f1730;border:1px solid var(--line);border-radius:12px;padding:10px 12px;cursor:pointer;white-space:pre-wrap;line-height:1.35}
    .opt.correct{border-color:#1f8f3a;background:rgba(31,143,58,.18)}
    .opt.wrong{border-color:#b02a37;background:rgba(176,42,55,.18)}
    .reveal{margin:8px 0 10px}
    details{border-top:1px dashed var(--line);padding-top:8px}
    summary{cursor:pointer;color:#d6defa}
    .answer{margin:8px 0 0;color:#d6defa}
    .small{font-size:12px;color:var(--muted)}
    @media (max-width:520px){ button{flex:1} }
  </style>
</head>
<body>
  <header>
    <div class="wrap">
      <h1>OPS – Full Interactive Exam</h1>
      <p>Tap a section or “Full Exam” and click on answers to see if you’re correct.</p>
      <p class="small">Correct answers turn green. Wrong answers turn red and the correct one is highlighted.</p>
      <div class="toolbar" id="toolbar"></div>
      <div class="meta" id="meta"></div>
    </div>
  </header>

  <main class="wrap" id="list"></main>

  <script>
    // =========================
    // 1) DATA (replace this)
    // =========================
    // Global rules supported:
    // - punctuation preserved
    // - multi-line options preserved (single selectable)
    // - image between stem and options (img)
    const QUESTION_BANK = [
      {
        id: "fire-1",
        section: "Fire & Smoke (Part 1)",
        topic: "Fire and Smoke",
        stem: "Sample question stem with punctuation exactly as provided.",
        img: null, // e.g. "img/fire_q12.png"
        options: [
          "A) Option A",
          "B) Option B (multi-line)\ncontinued line",
          "C) Option C",
          "D) Option D"
        ],
        correctIndex: 1, // 0-based
        explanation: "Correct: B) Option B (multi-line) continued line."
      },
      {
        id: "sec-1",
        section: "Security – Unlawful Interference",
        topic: "Security",
        stem: "Another sample stem.",
        img: null,
        options: ["A) ...","B) ...","C) ...","D) ..."],
        correctIndex: 2,
        explanation: "Correct: C) ..."
      }
    ];

    // =========================
    // 2) STATE
    // =========================
    const state = {
      activeSection: "FULL",
      mode: "NORMAL", // NORMAL | RANDOM50 | REVIEW_WRONG
      wrongSet: new Set(),    // question ids answered wrong at least once
      answered: new Map(),    // id -> {pickedIndex, isCorrect}
    };

    // =========================
    // 3) HELPERS
    // =========================
    const uniqSections = () => Array.from(new Set(QUESTION_BANK.map(q => q.section)));
    const bySection = (sec) => QUESTION_BANK.filter(q => q.section === sec);
    const shuffle = (arr) => {
      const a = arr.slice();
      for (let i=a.length-1;i>0;i--){
        const j=Math.floor(Math.random()*(i+1));
        [a[i],a[j]]=[a[j],a[i]];
      }
      return a;
    };

    function getActiveQuestions(){
      let pool = [];
      if (state.activeSection === "FULL") pool = QUESTION_BANK.slice();
      else pool = bySection(state.activeSection);

      if (state.mode === "REVIEW_WRONG"){
        pool = pool.filter(q => state.wrongSet.has(q.id));
      } else if (state.mode === "RANDOM50"){
        pool = shuffle(pool).slice(0, Math.min(50, pool.length));
      }
      return pool;
    }

    // =========================
    // 4) RENDER TOOLBAR
    // =========================
    function renderToolbar(){
      const tb = document.getElementById("toolbar");
      tb.innerHTML = "";

      const makeBtn = (label, onClick, isActive=false) => {
        const b = document.createElement("button");
        b.textContent = label;
        if (isActive) b.classList.add("active");
        b.onclick = onClick;
        tb.appendChild(b);
      };

      // Sections
      makeBtn("Full Exam (All Questions)", () => { state.activeSection="FULL"; state.mode="NORMAL"; renderAll(); }, state.activeSection==="FULL" && state.mode==="NORMAL");

      uniqSections().forEach(sec => {
        makeBtn(sec, () => { state.activeSection=sec; state.mode="NORMAL"; renderAll(); }, state.activeSection===sec && state.mode==="NORMAL");
      });

      // Modes
      makeBtn("Random 50 Questions", () => { state.mode="RANDOM50"; renderAll(); }, state.mode==="RANDOM50");
      makeBtn("Review Wrong Answers", () => { state.mode="REVIEW_WRONG"; renderAll(); }, state.mode==="REVIEW_WRONG");
      makeBtn("Reset Wrong / Clear", () => {
        state.wrongSet.clear();
        state.answered.clear();
        state.mode="NORMAL";
        renderAll();
      });
    }

    // =========================
    // 5) RENDER QUESTIONS
    // =========================
    function renderAll(){
      renderToolbar();
      const list = document.getElementById("list");
      const meta = document.getElementById("meta");

      const active = getActiveQuestions();
      const total = QUESTION_BANK.length;

      const secLabel = state.activeSection === "FULL" ? "Full Exam" : state.activeSection;
      const modeLabel =
        state.mode === "NORMAL" ? "" :
        state.mode === "RANDOM50" ? " • Random 50" :
        " • Review Wrong";

      meta.textContent = `${secLabel}${modeLabel} — showing ${active.length} of ${total} questions. Wrong saved: ${state.wrongSet.size}.`;

      list.innerHTML = "";
      active.forEach((q, idx) => list.appendChild(renderQCard(q, idx+1)));
    }

    function renderQCard(q, displayNumber){
      const card = document.createElement("div");
      card.className = "qcard";

      const qnum = document.createElement("div");
      qnum.className = "qnum";
      qnum.textContent = `Q${displayNumber}`;
      card.appendChild(qnum);

      const topic = document.createElement("div");
      topic.className = "topic";
      topic.textContent = q.topic || "";
      card.appendChild(topic);

      const stem = document.createElement("div");
      stem.className = "stem";
      stem.textContent = q.stem;
      card.appendChild(stem);

      if (q.img){
        const img = document.createElement("img");
        img.className = "qimg";
        img.src = q.img;
        img.alt = "Question image";
        card.appendChild(img);
      }

      const opts = document.createElement("div");
      opts.className = "opts";

      const prev = state.answered.get(q.id);

      q.options.forEach((txt, i) => {
        const o = document.createElement("div");
        o.className = "opt";
        o.textContent = txt;

        // apply previous state
        if (prev){
          if (i === q.correctIndex) o.classList.add("correct");
          if (prev.pickedIndex === i && !prev.isCorrect) o.classList.add("wrong");
        }

        o.onclick = () => {
          // lock-in first click per question (AvExam-like); remove this guard if you want re-tries
          if (state.answered.has(q.id)) return;

          const isCorrect = (i === q.correctIndex);
          state.answered.set(q.id, { pickedIndex:i, isCorrect });

          if (!isCorrect) state.wrongSet.add(q.id);

          // re-render just this card for clean highlighting
          renderAll();
        };

        opts.appendChild(o);
      });

      card.appendChild(opts);

      const reveal = document.createElement("div");
      reveal.className = "reveal";
      const det = document.createElement("details");
      const sum = document.createElement("summary");
      sum.textContent = "Show answer";
      det.appendChild(sum);

      const ans = document.createElement("div");
      ans.className = "answer";
      ans.textContent = q.explanation || `Correct: ${q.options[q.correctIndex]}`;
      det.appendChild(ans);

      reveal.appendChild(det);
      card.appendChild(reveal);

      return card;
    }

    // init
    renderAll();
  </script>
</body>
</html>
