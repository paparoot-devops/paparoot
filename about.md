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

.about-note {
  margin: 1rem 0 0;
  padding: 0.85rem 0.95rem;
  border: 1px solid #2f4f66;
  border-radius: 10px;
  background: #193448;
  color: #d9ecff;
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

      <p>Мені нецікаво переказувати документацію своїми словами. Набагато цікавіше розкласти по поличках, де саме ламається `deployment`, чому черговий `pipeline` починає заважати команді, або як дрібна помилка в конфігу вилізає в інцидент через два тижні.</p>

      <h2>Що тут буде</h2>
      <p>Основний фокус блогу це короткі й середні за розміром технічні статті з живим контекстом. Без вигаданих “ідеальних” платформ, зате з нормальною інженерною логікою: яка була стартова точка, що саме не влаштовувало, чому обрано саме таке рішення і як я перевіряю, що воно не зробило систему гіршою.</p>

      <p>Окремим блоком хочу вести прості статті на базові теми. За останні пів року співбесід дуже добре видно, де в людей є реальне розуміння, а де лише знайомі слова на кшталт `Kubernetes`, `Terraform` чи `observability`. Ці нотатки теж підуть сюди.</p>

      <h2>Для кого цей блог</h2>
      <ul class="about-list">
        <li>Для DevOps і platform engineers, яким ближчий прагматичний підхід, а не “магія з презентації”.</li>
        <li>Для тих, хто росте з `junior` або `middle` рівня і хоче зрозуміти не тільки “як зробити”, а й “чому саме так”.</li>
        <li>Для команд, яким потрібні короткі технічні розбори без води і без штучного героїзму.</li>
      </ul>

      <h2>Про що пишу найчастіше</h2>
      <ul class="about-list">
        <li>`Kubernetes`, Helm, ingress, rollout-и і дрібні нюанси експлуатації.</li>
        <li>`Terraform`, модулі, state, drift і все, що починає боліти після красивого демо.</li>
        <li>`CI/CD`, delivery-процеси, release hygiene і технічний борг у pipeline-ах.</li>
        <li>`Observability`, алерти, Grafana, Prometheus і якість сигналу замість шуму.</li>
        <li>Linux, networking, troubleshooting і супутні речі, без яких DevOps швидко перетворюється на набір кнопок.</li>
      </ul>

      <h2>Як я подаю матеріал</h2>
      <p>Майже кожен текст я хочу будувати від конкретної проблеми. Спочатку контекст, потім рішення, далі перевірка результату, ризики, помилки і висновки. Якщо тема проста, стаття теж буде простою. Якщо тема потребує деталей, я не буду різати її до рівня треду в соцмережі.</p>

      <div class="about-note">
        Усі приклади тут подаються у знеособленому вигляді: без секретів, реальних доменів, внутрішніх ідентифікаторів та іншого сміття, яке не має потрапляти в публічний текст.
      </div>
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
          <li>`Kubernetes`</li>
          <li>`Terraform`</li>
          <li>`CI/CD`</li>
          <li>`Observability`</li>
          <li>Linux і networking</li>
        </ul>
      </div>
    </section>

    <section class="about-side-box">
      <h2 class="about-side-title">Формат</h2>
      <div class="about-side-inner">
        <p>Короткі кейси, розбори рішень, базові пояснення і серія постів на основі реальних співбесід та щоденної інженерної практики.</p>
      </div>
    </section>
  </aside>
</div>
