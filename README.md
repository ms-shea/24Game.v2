# 24Game.v2
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Make 24!</title>
  <style>
    :root {
      --bg: #0f172a;
      --card-bg: #1e293b;
      --accent: #38bdf8;
      --accent-hover: #0284c7;
      --text: #f8fafc;
      --muted: #94a3b8;
      --success: #4ade80;
      --error: #f87171;
    }

    * { box-sizing: border-box; margin: 0; padding: 0; font-family: system-ui, -apple-system, sans-serif; }
    
    body {
      background-color: var(--bg);
      color: var(--text);
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
      padding: 1rem;
    }

    .container {
      background-color: var(--card-bg);
      padding: 2rem;
      border-radius: 1rem;
      box-shadow: 0 10px 25px rgba(0, 0, 0, 0.5);
      width: 100%;
      max-width: 480px;
      text-align: center;
    }

    h1 { margin-bottom: 0.5rem; color: var(--accent); }
    p.desc { color: var(--muted); margin-bottom: 1.5rem; font-size: 0.95rem; }

    .score-board {
      display: flex;
      justify-content: space-around;
      margin-bottom: 1.5rem;
      font-weight: bold;
      color: var(--muted);
    }
    .score-board span { color: var(--accent); }

    .numbers-grid {
      display: grid;
      grid-template-columns: repeat(4, 1fr);
      gap: 0.75rem;
      margin-bottom: 1.5rem;
    }

    .number-btn {
      background: #334155;
      color: var(--text);
      border: 2px solid transparent;
      padding: 1rem;
      font-size: 1.5rem;
      font-weight: bold;
      border-radius: 0.5rem;
      cursor: pointer;
      transition: all 0.2s;
    }

    .number-btn:hover:not(:disabled) { background: #475569; }
    .number-btn:disabled { opacity: 0.3; cursor: not-allowed; }

    .input-display {
      background: #0f172a;
      border: 2px solid #334155;
      border-radius: 0.5rem;
      padding: 0.75rem;
      font-size: 1.25rem;
      min-height: 52px;
      margin-bottom: 1rem;
      display: flex;
      align-items: center;
      justify-content: center;
      letter-spacing: 2px;
    }

    .controls {
      display: grid;
      grid-template-columns: repeat(4, 1fr);
      gap: 0.5rem;
      margin-bottom: 1rem;
    }

    .op-btn {
      background: #0284c7;
      color: white;
      border: none;
      padding: 0.75rem;
      font-size: 1.25rem;
      font-weight: bold;
      border-radius: 0.5rem;
      cursor: pointer;
      transition: background 0.2s;
    }
    .op-btn:hover { background: #0369a1; }

    .action-btns {
      display: flex;
      gap: 0.5rem;
      margin-bottom: 1rem;
    }

    .action-btn {
      flex: 1;
      padding: 0.75rem;
      border: none;
      border-radius: 0.5rem;
      font-weight: bold;
      cursor: pointer;
      transition: background 0.2s;
    }

    .btn-clear { background: #ef4444; color: white; }
    .btn-clear:hover { background: #dc2626; }

    .btn-submit { background: var(--success); color: #0f172a; }
    .btn-submit:hover { background: #22c55e; }

    .btn-next {
      width: 100%;
      background: #64748b;
      color: white;
      padding: 0.75rem;
      border: none;
      border-radius: 0.5rem;
      font-weight: bold;
      cursor: pointer;
      margin-top: 0.5rem;
    }
    .btn-next:hover { background: #475569; }

    .message {
      min-height: 24px;
      font-weight: bold;
      margin-top: 0.5rem;
    }
    .message.success { color: var(--success); }
    .message.error { color: var(--error); }
  </style>
</head>
<body>

<div class="container">
  <h1>Make 24</h1>
  <p class="desc">Use all 4 numbers once with +, -, *, / to get 24.</p>

  <div class="score-board">
    <div>Solved: <span id="score">0</span></div>
  </div>

  <div class="numbers-grid" id="numbersGrid"></div>

  <div class="input-display" id="display"></div>

  <div class="controls">
    <button class="op-btn" onclick="appendOperator('+')">+</button>
    <button class="op-btn" onclick="appendOperator('-')">-</button>
    <button class="op-btn" onclick="appendOperator('*')">&times;</button>
    <button class="op-btn" onclick="appendOperator('/')">&divide;</button>
    <button class="op-btn" onclick="appendOperator('(')">(</button>
    <button class="op-btn" onclick="appendOperator(')')">)</button>
    <button class="op-btn" style="grid-column: span 2" onclick="backspace()">&#9003;</button>
  </div>

  <div class="action-btns">
    <button class="action-btn btn-clear" onclick="clearInput()">Clear</button>
    <button class="action-btn btn-submit" onclick="checkSolution()">Submit</button>
  </div>

  <button class="btn-next" onclick="generateNewPuzzle()">Skip / New Hand</button>

  <div class="message" id="message"></div>
</div>

<script>
  let targetNumbers = [];
  let availableNumbers = [];
  let expressionTokens = [];
  let score = 0;

  function generateNewPuzzle() {
    targetNumbers = Array.from({ length: 4 }, () => Math.floor(Math.random() * 9) + 1);
    availableNumbers = [...targetNumbers];
    clearInput();
    renderNumbers();
    setMessage('');
  }

  function renderNumbers() {
    const grid = document.getElementById('numbersGrid');
    grid.innerHTML = '';
    
    // Tracks count of available numbers to disable used ones
    const counts = {};
    availableNumbers.forEach(n => counts[n] = (counts[n] || 0) + 1);

    targetNumbers.forEach((num, index) => {
      const btn = document.createElement('button');
      btn.className = 'number-btn';
      btn.textContent = num;
      
      if (counts[num] > 0) {
        counts[num]--;
        btn.onclick = () => selectNumber(num, index);
      } else {
        btn.disabled = true;
      }
      grid.appendChild(btn);
    });
  }

  function selectNumber(num, index) {
    expressionTokens.push({ type: 'number', value: num, index: index });
    const availIndex = availableNumbers.indexOf(num);
    if (availIndex > -1) availableNumbers.splice(availIndex, 1);
    
    updateDisplay();
    renderNumbers();
  }

  function appendOperator(op) {
    expressionTokens.push({ type: 'operator', value: op });
    updateDisplay();
  }

  function backspace() {
    const removed = expressionTokens.pop();
    if (removed && removed.type === 'number') {
      availableNumbers.push(removed.value);
    }
    updateDisplay();
    renderNumbers();
  }

  function clearInput() {
    expressionTokens = [];
    availableNumbers = [...targetNumbers];
    updateDisplay();
    renderNumbers();
    setMessage('');
  }

  function updateDisplay() {
    const displayStr = expressionTokens.map(t => {
      if (t.value === '*') return '×';
      if (t.value === '/') return '÷';
      return t.value;
    }).join(' ');
    
    document.getElementById('display').textContent = displayStr;
  }

  function setMessage(msg, type = '') {
    const msgEl = document.getElementById('message');
    msgEl.textContent = msg;
    msgEl.className = 'message ' + type;
  }

  function checkSolution() {
    if (availableNumbers.length > 0) {
      setMessage('You must use all 4 numbers!', 'error');
      return;
    }

    const rawExpression = expressionTokens.map(t => t.value).join('');
    
    try {
      // Evaluate expression safely using Function constructor
      const result = new Function(`return ${rawExpression}`)();

      // Check floating point equality for 24
      if (Math.abs(result - 24) < 0.00001) {
        setMessage('Correct! 🎉', 'success');
        score++;
        document.getElementById('score').textContent = score;
        setTimeout(generateNewPuzzle, 1500);
      } else {
        setMessage(`Equals ${Math.round(result * 100) / 100}, not 24. Try again!`, 'error');
      }
    } catch (e) {
      setMessage('Invalid expression syntax!', 'error');
    }
  }

  // Start initial game
  generateNewPuzzle();
</script>

</body>
</html>
