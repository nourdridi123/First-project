# First-project
A code html 
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Calculator</title>
<style>
  :root{
    --bg:#0f1720;
    --panel:#0b1220;
    --accent:#06b6d4;
    --muted:#9aa7b2;
    --btn:#111827;
    --glass: rgba(255,255,255,0.03);
    font-family: Inter, ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
  }
  html,body{height:100%}
  body{
    margin:0;
    display:flex;
    align-items:center;
    justify-content:center;
    background: linear-gradient(180deg, #071226 0%, #071a2a 100%);
    padding:20px;
    color:#e6eef3;
  }

  .calc {
    width: 320px;
    max-width: 96vw;
    border-radius:14px;
    background: linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));
    box-shadow: 0 10px 30px rgba(2,6,23,0.6);
    padding:18px;
    display:flex;
    flex-direction:column;
    gap:12px;
  }

  .display {
    background:var(--glass);
    border-radius:10px;
    padding:12px;
    min-height:64px;
    display:flex;
    flex-direction:column;
    justify-content:center;
    box-shadow: inset 0 -1px 0 rgba(255,255,255,0.02);
  }

  .display .expr {
    font-size:13px;
    color:var(--muted);
    min-height:16px;
    word-break:break-all;
  }
  .display .out {
    font-size:28px;
    font-weight:600;
    text-align:right;
    letter-spacing:0.5px;
    margin-top:4px;
    word-break:break-all;
  }

  .keys {
    display:grid;
    grid-template-columns: repeat(4, 1fr);
    gap:10px;
  }

  button.key {
    background: linear-gradient(180deg, rgba(255,255,255,0.015), rgba(255,255,255,0.01));
    border:1px solid rgba(255,255,255,0.03);
    padding:14px;
    border-radius:10px;
    font-size:16px;
    color:inherit;
    cursor:pointer;
    user-select:none;
    transition: transform .06s ease, box-shadow .06s;
    box-shadow: 0 4px 10px rgba(2,6,23,0.45);
  }
  button.key:active{ transform: translateY(1px) scale(.998); }
  button.op { color: var(--accent); font-weight:600; }
  button.func { background: rgba(255,255,255,0.02); color:var(--muted); }
  button.equal {
    grid-column: span 2;
    background: linear-gradient(180deg, #05a6bd, #038b9d);
    color:#071826;
    font-weight:700;
    box-shadow: 0 10px 20px rgba(5,166,189,0.16);
  }

  /* responsive smaller buttons on mobile */
  @media (max-width:400px){
    .display .out{ font-size:22px; }
    button.key{ padding:12px; font-size:15px; }
  }

  .hint{
    font-size:12px;
    color:var(--muted);
    text-align:center;
    margin-top:6px;
  }
</style>
</head>
<body>
  <main class="calc" role="application" aria-label="Calculator">
    <div class="display" aria-live="polite">
      <div id="expr" class="expr" aria-hidden="true"></div>
      <div id="out" class="out">0</div>
    </div>

    <div class="keys" role="group" aria-label="Calculator keys">
      <button class="key func" data-action="clear" title="Clear (Esc)">C</button>
      <button class="key func" data-action="back" title="Backspace (Backspace)">⌫</button>
      <button class="key func" data-value="(">(</button>
      <button class="key func" data-value=")">)</button>

      <button class="key" data-value="7">7</button>
      <button class="key" data-value="8">8</button>
      <button class="key" data-value="9">9</button>
      <button class="key op" data-value="/">÷</button>

      <button class="key" data-value="4">4</button>
      <button class="key" data-value="5">5</button>
      <button class="key" data-value="6">6</button>
      <button class="key op" data-value="*">×</button>

      <button class="key" data-value="1">1</button>
      <button class="key" data-value="2">2</button>
      <button class="key" data-value="3">3</button>
      <button class="key op" data-value="-">−</button>

      <button class="key" data-value="0" style="grid-column: span 2;">0</button>
      <button class="key" data-value=".">.</button>
      <button class="key op" data-value="+">+</button>

      <button class="key func" data-action="percent">%</button>
      <button class="key func" data-action="neg">±</button>
      <button class="key equal" data-action="equals" title="Equals (Enter)">=</button>
    </div>

    <div class="hint">Keyboard: numbers, + - * / ( ) . Enter =, Backspace, Esc</div>
  </main>

<script>
(function(){
  const exprEl = document.getElementById('expr');
  const outEl  = document.getElementById('out');
  const keys = document.querySelectorAll('button.key');

  let expression = ''; // raw expression used for eval (uses *,/ etc)
  let lastResult = null;

  function render(){
    exprEl.textContent = expression || '';
    outEl.textContent = expression ? expression : '0';
  }

  // sanitize before evaluating: only allow digits, operators, parentheses, dot, spaces
  function isSafeExpression(s){
    return /^[0-9+\-*/().\s%]*$/.test(s);
  }

  function calculate(expr){
    // handle percent by transforming "number%" into "(number/100)"
    // Replace occurrences like 50% -> (50/100)
    try {
      let prepared = expr.replace(/(\d+(\.\d+)?)%/g, '($1/100)');
      if(!isSafeExpression(prepared)) throw new Error('Invalid characters');
      // Use Function constructor instead of eval to be slightly cleaner
      // It's still JS evaluation but input is validated above.
      const fn = new Function('return (' + prepared + ')');
      const res = fn();
      if (!isFinite(res)) throw new Error('Result is not finite');
      return +parseFloat(res.toFixed(12)); // trim floating noise
    } catch (e) {
      return 'Error';
    }
  }

  function onKeyClick(btn){
    const val = btn.getAttribute('data-value');
    const action = btn.getAttribute('data-action');

    if(action === 'clear'){
      expression = '';
      lastResult = null;
      render();
      return;
    }

    if(action === 'back'){
      expression = expression.slice(0, -1);
      render();
      return;
    }

    if(action === 'percent'){
      // append percent sign to the last number (simple approach)
      expression += '%';
      render();
      return;
    }

    if(action === 'neg'){
      // toggle negative for the last number: find last number and invert
      const m = expression.match(/(.*?)(-?\d+(\.\d+)?)(\D*)$/);
      if(m){
        const before = m[1] || '';
        const num = m[2] || '';
        const after = m[4] || '';
        if(num.startsWith('-')) {
          expression = before + num.slice(1) + after;
        } else {
          expression = before + '-' + num + after;
        }
      } else {
        // if empty just add minus
        expression = expression ? expression : '-';
      }
      render();
      return;
    }

    if(action === 'equals'){
      if(!expression) return;
      const result = calculate(expression);
      if(result === 'Error'){
        outEl.textContent = 'Error';
      } else {
        outEl.textContent = result;
        expression = String(result);
        lastResult = result;
      }
      return;
    }

    // default: append value (digit/operator/paren/dot)
    if(val){
      // prevent multiple dots in the same number segment
      if(val === '.'){
        // get last number
        const lastNumMatch = expression.match(/(\d+\.\d*|\d+|\.\d*)$/);
        if(lastNumMatch && lastNumMatch[0].includes('.')) {
          return; // already has dot
        }
      }
      // Prevent two operators in a row (except minus as unary)
      const operators = ['+','-','*','/'];
      const lastChar = expression.slice(-1);
      if(operators.includes(val) && operators.includes(lastChar)){
        // allow if lastChar is '(' or nothing? just replace the operator
        expression = expression.slice(0, -1) + val;
        render();
        return;
      }

      expression += val;
      render();
    }
  }

  keys.forEach(k => k.addEventListener('click', () => onKeyClick(k)));

  // Keyboard support
  window.addEventListener('keydown', (e) => {
    const key = e.key;
    if(/\d/.test(key)) {
      onKeyClick({ getAttribute: (n)=> n === 'data-value'? key : null });
      e.preventDefault();
      return;
    }

    if(key === 'Enter' || key === '='){
      onKeyClick({ getAttribute: (n)=> n === 'data-action'? 'equals' : null });
      e.preventDefault();
      return;
    }
    if(key === 'Backspace'){
      onKeyClick({ getAttribute: (n)=> n === 'data-action'? 'back' : null });
      e.preventDefault();
      return;
    }
    if(key === 'Escape'){
      onKeyClick({ getAttribute: (n)=> n === 'data-action'? 'clear' : null });
      e.preventDefault();
      return;
    }
    if(key === '.' || key === '(' || key === ')'){
      onKeyClick({ getAttribute: (n)=> n === 'data-value'? key : null });
      e.preventDefault();
      return;
    }
    if(key === '+' || key === '-' || key === '*' || key === '/'){
      onKeyClick({ getAttribute: (n)=> n === 'data-value'? key : null });
      e.preventDefault();
      return;
    }
    if(key === '%'){
      onKeyClick({ getAttribute: (n)=> n === 'data-action'? 'percent' : null });
      e.preventDefault();
      return;
    }
  });

  // initialize
  render();
})();
</script>
</body>
</html>
