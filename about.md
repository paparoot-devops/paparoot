---
layout: default
title: "Про блог"
permalink: /about/
---

<style>
.about-grid {
  display: grid;
  grid-template-columns: minmax(0, 2fr) minmax(260px, 0.9fr);
  gap: 1rem;
  padding: 1rem;
}

.about-grid section,
.about-grid aside {
  min-width: 0;
}

.about-side-box {
  background: #2d3949;
  border: 1px solid var(--line);
  border-radius: 12px;
  margin-bottom: 0.75rem;
  overflow: hidden;
}

.about-side-title {
  margin: 0;
  padding: 0.62rem 0.78rem;
  border-bottom: 1px solid var(--line);
  background: #253142;
  color: #eaf2ff;
  font-size: 1rem;
}

.about-side-inner { padding: 0.72rem 0.78rem; color: #d7e3f5; }

.about-list {
  margin: 0;
  padding-left: 1.1rem;
}

.about-list li {
  margin-bottom: 0.45rem;
}

.about-lead {
  font-size: 1.05rem;
  color: #dce8f8;
}

.about-article h2 {
  margin-top: 1.5rem;
}

.about-article p,
.about-article li {
  color: #d6e3f5;
}

.about-side-inner p {
  margin: 0 0 0.65rem;
}

.about-side-inner p:last-child {
  margin-bottom: 0;
}

@media (max-width: 900px) {
  .about-grid { grid-template-columns: 1fr; }
}
</style>

<div class="about-grid">
  <section>
    <article class="article-card about-article">
      <h1>Про блог</h1>
      <p class="about-lead">Це блог про практичний DevOps без зайвого пафосу. Тут я збираю кейси, спостереження і розбори речей, які або працюють у проді, або боляче не працюють.</p>

      <p>Мені нецікаво переказувати документацію своїми словами. Набагато цікавіше розкласти по поличках, де саме ламається <code>deployment</code>, чому черговий <code>pipeline</code> починає заважати команді, або як дрібна помилка в конфігу вилізає в інцидент через два тижні.</p>

      <h2>Що тут буде</h2>
      <p>Основний фокус блогу це короткі й середні за розміром технічні статті з живим контекстом. Без вигаданих “ідеальних” платформ, зате з нормальною інженерною логікою: яка була стартова точка, що саме не влаштовувало, чому обрано саме таке рішення і як я перевіряю, що воно не зробило систему гіршою.</p>

      <h2>Про що пишу найчастіше</h2>
      <ul class="about-list">
        <li>Kubernetes, Helm, ingress, rollout-и і дрібні нюанси експлуатації.</li>
        <li>Terraform, модулі, state, drift і все, що починає боліти після красивого демо.</li>
        <li>CI/CD, delivery-процеси, release hygiene і технічний борг у pipeline-ах.</li>
        <li>Observability, алерти, Grafana, Prometheus і якість сигналу замість шуму.</li>
        <li>Linux, networking, troubleshooting і супутні речі, без яких DevOps швидко перетворюється на набір кнопок.</li>
      </ul>

    </article>
  </section>

  <aside>
    <section class="about-side-box">
      <h2 class="about-side-title">Коротко</h2>
      <div class="about-side-inner">
        <p>Практичні DevOps-нотатки з акцентом на експлуатацію, зміни в існуючих системах і технічні розбори без прикрас.</p>
      </div>
    </section>

    <section class="about-side-box">
      <h2 class="about-side-title">Основні теми</h2>
      <div class="about-side-inner">
        <ul class="about-list">
          <li>Kubernetes</li>
          <li>Terraform</li>
          <li>CI/CD</li>
          <li>Observability</li>
          <li>Linux і networking</li>
        </ul>
      </div>
    </section>
  </aside>
</div>
