<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>StudyPulse — Quiz & Study Hub</title>
  <style>
    :root {
      --bg: #0f172a;
      --card-bg: #1e293b;
      --card-border: #334155;
      --text: #f8fafc;
      --text-muted: #94a3b8;
      --accent: #6366f1;
      --accent-hover: #4f46e5;
      --success: #10b981;
      --danger: #ef4444;
      --radius: 12px;
    }
    * { box-sizing: border-box; margin: 0; padding: 0; font-family: system-ui, sans-serif; }
    body { background-color: var(--bg); color: var(--text); min-height: 100vh; display: flex; flex-direction: column; align-items: center; padding: 2rem 1rem; }
    .container { width: 100%; max-width: 680px; }
    header { text-align: center; margin-bottom: 2rem; }
    header h1 { font-size: 2rem; margin-bottom: 0.5rem; }
    header p { color: var(--text-muted); font-size: 0.95rem; }
    .card { background: var(--card-bg); border: 1px solid var(--card-border); border-radius: var(--radius); padding: 1.5rem; margin-bottom: 1.5rem; }
    .btn-row { display: flex; gap: 0.75rem; flex-wrap: wrap; margin-top: 1rem; }
    button { background: var(--accent); color: #fff; border: none; padding: 0.75rem 1.25rem; border-radius: 8px; font-weight: 600; cursor: pointer; }
    button:hover { background: var(--accent-hover); }
    button.secondary { background: #334155; }
    .progress-bar { width: 100%; height: 8px; background: var(--card-border); border-radius: 999px; overflow: hidden; margin-bottom: 1.5rem; }
    .progress-fill { height: 100%; background: var(--accent); width: 0%; transition: width 0.3s ease; }
    .quiz-meta { display: flex; justify-content: space-between; color: var(--text-muted); font-size: 0.85rem; margin-bottom: 0.75rem; }
    .question-title { font-size: 1.25rem; line-height: 1.4; margin-bottom: 1.25rem; }
    .options-grid { display: flex; flex-direction: column; gap: 0.75rem; margin-bottom: 1.25rem; }
    .option-btn { background: #0f172a; border: 1px solid var(--card-border); color: var(--text); text-align: left; padding: 1rem; border-radius: 8px; }
    .option-btn.correct { background: rgba(16, 185, 129, 0.2) !important; border-color: var(--success) !important; color: #6ee7b7 !important; }
    .option-btn.incorrect { background: rgba(239, 68, 68, 0.2) !important; border-color: var(--danger) !important; color: #fca5a5 !important; }
    .explanation-box { background: #0f172a; border-left: 4px solid var(--accent); padding: 0.75rem 1rem; margin-bottom: 1.25rem; font-size: 0.9rem; color: var(--text-muted); }
    input, textarea { width: 100%; background: #0f172a; border: 1px solid var(--card-border); border-radius: 6px; color: var(--text); padding: 0.75rem; margin-bottom: 1rem; }
    label { display: block; font-size: 0.85rem; font-weight: 600; color: var(--text-muted); margin-bottom: 0.25rem; }
    .hidden { display: none !important; }
    .share-banner { background: #1e1b4b; border: 1px solid #4338ca; padding: 1rem; border-radius: 8px; margin-top: 1rem; }
  </style>
</head>
<body>
  <div class="container">
    <header>
      <h1>StudyPulse</h1>
      <p>Interactive study hub — practice, build quizzes, and challenge friends</p>
    </header>

    <div id="view-home" class="card">
      <h2 style="margin-bottom: 1rem;">Available Quizzes</h2>
      <div id="quiz-list" style="display: flex; flex-direction: column; gap: 0.75rem;"></div>
      <div class="btn-row" style="margin-top: 1.5rem;">
        <button onclick="showView('create')">+ Create Your Own Quiz</button>
      </div>
    </div>

    <div id="view-runner" class="card hidden">
      <div class="quiz-meta">
        <span id="quiz-badge">Question 1/3</span>
        <span id="score-counter">Score: 0</span>
      </div>
      <div class="progress-bar"><div id="progress-fill" class="progress-fill"></div></div>
      <h3 id="question-text" class="question-title"></h3>
      <div id="options-container" class="options-grid"></div>
      <div id="explanation" class="explanation-box hidden"></div>
      <button id="next-btn" class="hidden" onclick="nextQuestion()">Next Question →</button>
    </div>

    <div id="view-results" class="card hidden" style="text-align: center;">
      <h2>Quiz Completed!</h2>
      <p id="result-summary" style="font-size: 1.5rem; margin: 1rem 0; font-weight: bold;"></p>
      <p id="result-message" style="color: var(--text-muted); margin-bottom: 1.5rem;"></p>
      <div class="share-banner">
        <p style="font-size: 0.9rem; margin-bottom: 0.5rem;">Share this quiz with your friends:</p>
        <button class="secondary" onclick="copyCurrentQuizLink()">🔗 Copy Share Link</button>
      </div>
      <div class="btn-row" style="justify-content: center; margin-top: 1.5rem;">
        <button onclick="restartQuiz()">Try Again</button>
        <button class="secondary" onclick="showView('home')">All Quizzes</button>
      </div>
    </div>

    <div id="view-create" class="card hidden">
      <h2>Build a Custom Quiz</h2>
      <label>Quiz Title</label>
      <input type="text" id="custom-title" placeholder="e.g., Biology Review" />
      <div id="creator-questions"></div>
      <div class="btn-row">
        <button type="button" class="secondary" onclick="addQuestionField()">+ Add Question</button>
        <button type="button" onclick="saveCustomQuiz()">Generate & Save Quiz</button>
        <button type="button" class="secondary" onclick="showView('home')">Cancel</button>
      </div>
    </div>
  </div>

  <script>
    const defaultQuizzes = [
      {
        id: "web-dev",
        title: "General Knowledge & Science",
        questions: [
          {
            question: "What gas do plants primarily absorb during daylight photosynthesis?",
            options: ["Oxygen", "Carbon Dioxide", "Nitrogen", "Argon"],
            answer: 1,
            explanation: "Plants absorb carbon dioxide (CO2) and produce oxygen."
          },
          {
            question: "What is the hardest naturally occurring mineral on Earth?",
            options: ["Quartz", "Granite", "Diamond", "Corundum"],
            answer: 2,
            explanation: "Diamond is ranked 10 on the Mohs mineral hardness scale."
          }
        ]
      }
    ];

    let quizzes = [...defaultQuizzes];
    let currentQuiz = null;
    let currentQuestionIdx = 0;
    let score = 0;

    function showView(view) {
      document.getElementById("view-home").classList.add("hidden");
      document.getElementById("view-runner").classList.add("hidden");
      document.getElementById("view-results").classList.add("hidden");
      document.getElementById("view-create").classList.add("hidden");
      document.getElementById("view-" + view).classList.remove("hidden");
      if (view === "home") renderQuizList();
    }

    function renderQuizList() {
      const list = document.getElementById("quiz-list");
      list.innerHTML = "";
      quizzes.forEach((quiz, index) => {
        const item = document.createElement("div");
        item.style.display = "flex";
        item.style.justifyContent = "space-between";
        item.style.alignItems = "center";
        item.style.padding = "0.75rem";
        item.style.background = "#0f172a";
        item.style.borderRadius = "8px";
        item.innerHTML = `
          <div>
            <strong>${quiz.title}</strong>
            <p style="font-size: 0.8rem; color: var(--text-muted);">${quiz.questions.length} Questions</p>
          </div>
          <button onclick="startQuiz(${index})">Start Quiz</button>
        `;
        list.appendChild(item);
      });
    }

    function startQuiz(index) {
      currentQuiz = quizzes[index];
      currentQuestionIdx = 0;
      score = 0;
      showView("runner");
      renderQuestion();
    }

    function renderQuestion() {
      const q = currentQuiz.questions[currentQuestionIdx];
      document.getElementById("quiz-badge").innerText = `Question ${currentQuestionIdx + 1} of ${currentQuiz.questions.length}`;
      document.getElementById("score-counter").innerText = `Score: ${score}`;
      document.getElementById("question-text").innerText = q.question;
      document.getElementById("progress-fill").style.width = `${((currentQuestionIdx) / currentQuiz.questions.length) * 100}%`;
      const container = document.getElementById("options-container");
      container.innerHTML = "";
      document.getElementById("explanation").classList.add("hidden");
      document.getElementById("next-btn").classList.add("hidden");

      q.options.forEach((opt, idx) => {
        const btn = document.createElement("button");
        btn.className = "option-btn";
        btn.innerText = opt;
        btn.onclick = () => selectOption(idx);
        container.appendChild(btn);
      });
    }

    function selectOption(selected) {
      const q = currentQuiz.questions[currentQuestionIdx];
      const btns = document.querySelectorAll(".option-btn");
      btns.forEach(b => b.disabled = true);
      if (selected === q.answer) {
        btns[selected].classList.add("correct");
        score++;
      } else {
        btns[selected].classList.add("incorrect");
        btns[q.answer].classList.add("correct");
      }
      document.getElementById("score-counter").innerText = `Score: ${score}`;
      if (q.explanation) {
        const exp = document.getElementById("explanation");
        exp.innerText = q.explanation;
        exp.classList.remove("hidden");
      }
      document.getElementById("next-btn").classList.remove("hidden");
    }

    function nextQuestion() {
      currentQuestionIdx++;
      if (currentQuestionIdx < currentQuiz.questions.length) {
        renderQuestion();
      } else {
        showView("results");
        document.getElementById("result-summary").innerText = `${score} / ${currentQuiz.questions.length}`;
      }
    }

    function restartQuiz() {
      currentQuestionIdx = 0;
      score = 0;
      showView("runner");
      renderQuestion();
    }

    function addQuestionField() {
      const wrap = document.createElement("div");
      const idx = document.getElementById("creator-questions").children.length;
      wrap.className = "card";
      wrap.style.background = "#0f172a";
      wrap.style.marginTop = "1rem";
      wrap.innerHTML = `
        <label>Question ${idx + 1}</label>
        <input type="text" class="cq-title" placeholder="Enter question" required />
        <label>Options (Select radio for correct answer)</label>
        ${[0, 1, 2, 3].map(i => `
          <div style="display:flex; gap:0.5rem; align-items:center; margin-bottom:0.5rem;">
            <input type="radio" name="ans-${idx}" value="${i}" ${i === 0 ? "checked" : ""} style="width:auto; margin:0;" />
            <input type="text" class="cq-opt-${idx}" placeholder="Option ${i + 1}" required style="margin:0;" />
          </div>
        `).join("")}
      `;
      document.getElementById("creator-questions").appendChild(wrap);
    }

    function saveCustomQuiz() {
      const title = document.getElementById("custom-title").value.trim() || "My Custom Quiz";
      const qEls = document.getElementById("creator-questions").children;
      const questions = [];
      for (let i = 0; i < qEls.length; i++) {
        const qTitle = qEls[i].querySelector(".cq-title").value.trim();
        const opts = Array.from(qEls[i].querySelectorAll(`.cq-opt-${i}`)).map(input => input.value.trim());
        const ans = parseInt(qEls[i].querySelector(`input[name="ans-${i}"]:checked`).value, 10);
        if (qTitle && opts.every(o => o)) questions.push({ question: qTitle, options: opts, answer: ans });
      }
      if (questions.length === 0) { alert("Please add at least one complete question."); return; }
      const customQuiz = { id: "c-" + Date.now(), title, questions };
      quizzes.push(customQuiz);
      currentQuiz = customQuiz;
      copyCurrentQuizLink();
      showView("home");
    }

    function copyCurrentQuizLink() {
      const json = JSON.stringify(currentQuiz);
      const encoded = encodeURIComponent(btoa(unescape(encodeURIComponent(json))));
      const shareUrl = `${window.location.origin}${window.location.pathname}#quiz=${encoded}`;
      navigator.clipboard.writeText(shareUrl).then(() => alert("Share link copied! Send it to your friends.")).catch(() => prompt("Copy link:", shareUrl));
    }

    window.addEventListener("DOMContentLoaded", () => {
      addQuestionField();
      if (window.location.hash.startsWith("#quiz=")) {
        try {
          const raw = window.location.hash.replace("#quiz=", "");
          const decoded = decodeURIComponent(escape(atob(decodeURIComponent(raw))));
          quizzes.unshift(JSON.parse(decoded));
          startQuiz(0);
          return;
        } catch (e) {}
      }
      renderQuizList();
    });
  </script>
</body>
</html>
