<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>OPS PT2 – Sample QDB</title>
<style>
body {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Arial;
  background: #f5f7fa;
  margin: 0;
  padding: 20px;
}
.question {
  background: #fff;
  border-radius: 12px;
  padding: 18px;
  margin-bottom: 20px;
  box-shadow: 0 4px 12px rgba(0,0,0,.08);
}
.q-number {
  font-weight: 700;
  margin-bottom: 10px;
}
.options {
  list-style: none;
  padding: 0;
  margin-top: 12px;
}
.options li {
  padding: 12px;
  border: 1px solid #ddd;
  border-radius: 8px;
  margin-bottom: 8px;
  cursor: pointer;
}
.options li.correct {
  background: #d1fae5;
  border-color: #10b981;
}
.options li.wrong {
  background: #fee2e2;
  border-color: #ef4444;
}
.question.locked .options li {
  cursor: default;
}
</style>
</head>

<body>

<div class="question" data-correct="0">
  <div class="q-number">
    1) Before take-off on an MNPS flight, one of your Long Range Navigation System fails.
    What is your action in case you have one system left?
  </div>
  <ul class="options">
    <li data-idx="0">Fly Special routes or fly above or below the MNPS.</li>
    <li data-idx="1">Fly 15 NM offset to the normal tracks.</li>
    <li data-idx="2">The co-pilot has to exchange the two black boxes.</li>
    <li data-idx="3">Continue, because only one system must be operative.</li>
  </ul>
</div>

<div class="question" data-correct="1">
  <div class="q-number">
    2) What is the NAT Track Message?
  </div>
  <ul class="options">
    <li data-idx="0">The complete MNPS flight plan.</li>
    <li data-idx="1">The publication of the Organized Track System (OTS).</li>
    <li data-idx="2">The publication of the Preferred Route Message (PRM).</li>
    <li data-idx="3">The ATC clearance given before the boundary window.</li>
  </ul>
</div>

<div class="question" data-correct="3">
  <div class="q-number">
    3) Minimum time route is a route calculated for:
  </div>
  <ul class="options">
    <li data-idx="0">a period of 12 hours.</li>
    <li data-idx="1">aircraft flying in MNPS airspace.</li>
    <li data-idx="2">a period of 24 hours.</li>
    <li data-idx="3">a given flight from departure to destination.</li>
  </ul>
</div>

<script>
document.querySelectorAll(".question").forEach(q => {
  const correct = parseInt(q.dataset.correct);
  const options = q.querySelectorAll(".options li");

  options.forEach(opt => {
    opt.addEventListener("click", () => {
      if (q.classList.contains("locked")) return;
      q.classList.add("locked");

      const idx = parseInt(opt.dataset.idx);
      options[correct].classList.add("correct");
      if (idx !== correct) opt.classList.add("wrong");
    });
  });
});
</script>

</body>
</html>
