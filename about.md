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

.about-side-box {
  background: #2d3949;
  border: 1px solid var(--line);
  border-radius: 10px;
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
  padding-left: 1rem;
}

@media (max-width: 900px) {
  .about-grid { grid-template-columns: 1fr; }
}
</style>

<div class="about-grid">
  <section>
    <article class="article-card">
      <h1>Про блог</h1>
      <p>Цей блог про практичні DevOps-кейси з реальної експлуатації систем.</p>
      <p>Формат матеріалів: короткі інженерні розбори з контекстом, рішенням, перевіркою результату та висновками.</p>
      <h2>Теми</h2>
      <ul>
        <li>Kubernetes</li>
        <li>Terraform</li>
        <li>CI/CD</li>
        <li>Observability</li>
        <li>Security tooling</li>
      </ul>
      <h2>Принцип публікацій</h2>
      <p>Усі приклади подаються у знеособленому вигляді: без секретів, реальних доменів і внутрішніх ідентифікаторів.</p>
    </article>
  </section>
</div>
