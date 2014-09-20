---
layout: default
---

<div class="home">
  <section id="tag-cloud">
    <p class="title">博客归档<p>
    {{ site | tag_cloud }}
  </section>

  <section class="latest-posts">
    <p class="title" >最新博客<p>
    <ul>
      {% for post in site.categories["archive"] limit:6  %}
        <li>
          <a class="post-link" href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a>
          <span class="post-date">{{ post.date | date: "%b %-d, %Y" }}</span>
        </li>
      {% endfor %}
    </ul>
  </section>

  <section class="latest-posts books">
    <p class="title" >在线电子书<p>
    <ul>
      <li><a href="http://www.redisbook.com/en/latest/" target="_blank">Redis 设计与实现</a> </li>

      <li><a href="http://vbird.dic.ksu.edu.tw/" target="_blank">鸟哥私房菜</a></li>
      <li><a href="http://billie66.github.io/TLCL/" target="_blank">TLCL</a></li>

      <li><a href="https://gettingreal.37signals.com/GR_chn.php" target="_blank">Getting Real</a></li>
    </ul>
  </section>
</div>