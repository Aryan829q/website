<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Attendance Calculator — Gen Z Dark</title>
  <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-black min-h-screen flex items-center justify-center p-6 text-gray-100 font-sans">

  <div class="w-full max-w-xl bg-white/5 backdrop-blur-lg rounded-2xl shadow-2xl p-6 border border-white/10">
    <h1 class="text-3xl font-extrabold mb-6 text-center bg-gradient-to-r from-fuchsia-400 via-purple-400 to-blue-400 bg-clip-text text-transparent">
      🎧 Attendance Calculator
    </h1>

    <form id="form" class="space-y-6" onsubmit="return false;">
      <!-- Inputs -->
      <div class="grid grid-cols-2 gap-4">
        <label class="flex flex-col">
          <span class="text-sm font-medium text-gray-300">Total classes</span>
          <input id="total" type="number" min="0" step="1" value="10"
                 class="mt-1 p-2 rounded-md bg-black/70 text-gray-100 border border-gray-700 focus:ring-2 focus:ring-fuchsia-400" />
        </label>

        <label class="flex flex-col">
          <span class="text-sm font-medium text-gray-300">Attended</span>
          <input id="attended" type="number" min="0" step="1" value="6"
                 class="mt-1 p-2 rounded-md bg-black/70 text-gray-100 border border-gray-700 focus:ring-2 focus:ring-fuchsia-400" />
        </label>
      </div>

      <!-- Target -->
      <label class="block">
        <span class="text-sm font-medium text-gray-300">Target % 🎯</span>
        <div class="flex items-center gap-3 mt-2">
          <input id="targetRange" type="range" min="0" max="100" value="85" step="1"
                 class="flex-1 accent-fuchsia-400" />
          <input id="targetNumber" type="number" min="0" max="100" value="85" step="1"
                 class="w-20 p-2 rounded-md bg-black/70 text-gray-100 border border-gray-700 focus:ring-2 focus:ring-fuchsia-400" />
        </div>
      </label>

      <!-- Progress -->
      <div>
        <div class="text-sm text-gray-400 mb-2">Your current vibe</div>
        <div class="relative h-6 bg-gray-800 rounded-full overflow-hidden">
          <div id="bar" class="h-full rounded-full transition-all duration-300"></div>
          <div id="marker" class="absolute top-0 h-full w-0.5 bg-white/70"></div>
          <div id="percentLabel" class="absolute inset-0 flex items-center justify-center text-xs font-semibold"></div>
        </div>
      </div>

      <!-- Message -->
      <div id="message" class="mt-4 p-4 rounded-xl text-sm text-center font-medium"></div>
    </form>
  </div>

  <script>
    const totalEl = document.getElementById('total');
    const attendedEl = document.getElementById('attended');
    const targetRange = document.getElementById('targetRange');
    const targetNumber = document.getElementById('targetNumber');
    const bar = document.getElementById('bar');
    const marker = document.getElementById('marker');
    const percentLabel = document.getElementById('percentLabel');
    const message = document.getElementById('message');

    targetRange.addEventListener('input', () => targetNumber.value = targetRange.value);
    targetNumber.addEventListener('input', () => {
      let v = Number(targetNumber.value);
      if (isNaN(v)) v = 0;
      if (v < 0) v = 0;
      if (v > 100) v = 100;
      targetNumber.value = v;
      targetRange.value = v;
      update();
    });

    [totalEl, attendedEl].forEach(el => el.addEventListener('input', update));
    targetRange.addEventListener('input', update);

    function requiredClasses(total, attended, targetPercent) {
      if (attended > total) return { error: "Bro... attended > total? 🧐" };

      const P = targetPercent / 100;
      if (total === 0) return { x: (P === 0 ? 0 : 1), current: 0 };

      const current = attended / total;
      if (current >= P) return { x: 0, current };

      if (P >= 1) return { error: "100% target means zero skips allowed 🚫" };

      const x = Math.ceil((P * total - attended) / (1 - P));
      return { x, current };
    }

    function update() {
      const total = Number(totalEl.value);
      const attended = Number(attendedEl.value);
      const target = Number(targetRange.value);

      const res = requiredClasses(total, attended, target);

      let currentPct = total > 0 ? (attended / total) * 100 : 0;
      const cappedPct = Math.min(100, Math.max(0, currentPct));

      bar.style.width = cappedPct + '%';
      bar.style.background = cappedPct >= target
        ? 'linear-gradient(90deg,#22c55e,#16a34a)'
        : 'linear-gradient(90deg,#0ea5e9,#8b5cf6,#ec4899)';
      marker.style.left = Math.min(100, Math.max(0, target)) + '%';
      percentLabel.textContent = `${cappedPct.toFixed(1)}% 🎧`;

      if (res.error) {
        message.className = 'mt-4 p-4 rounded-xl text-sm text-red-300 bg-red-900/40';
        message.innerHTML = res.error;
        return;
      }

      if (res.x === 0) {
        message.className = 'mt-4 p-4 rounded-xl text-sm bg-green-900/40 text-green-300';
        message.innerHTML = `✅ Chill fam, you're safe at <strong>${currentPct.toFixed(1)}%</strong>`;
        return;
      }

      const futureTotal = total + res.x;
      const futureAttended = attended + res.x;
      const futurePct = (futureAttended / futureTotal) * 100;

      message.className = 'mt-4 p-4 rounded-xl text-sm bg-yellow-900/40 text-yellow-300';
      message.innerHTML = `
        ⚠ You're at <strong>${currentPct.toFixed(1)}%</strong> rn.<br>
        To hit <strong>${target}%</strong>, don’t skip the next <strong>${res.x}</strong> classes.<br>
        After grinding: <strong>${futurePct.toFixed(1)}%</strong> 📈
      `;
    }

    update();
  </script>
</body>
</html>
