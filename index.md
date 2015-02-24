---
layout: default
---

<div class="home">
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

  <section class="latest-posts reprinted">
    <p class="title" >转载文章<p>
    <ul>
      {% for post in site.categories["reprinted"] limit:6  %}
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
      <li><a href="https://github.com/justjavac/free-programming-books-zh_CN" target="_blank">免费的编程中文书籍索引</a> </li>
      <li><a href="http://redisbook.com" target="_blank">Redis 设计与实现</a> </li>

      <li><a href="http://vbird.dic.ksu.edu.tw/" target="_blank">鸟哥私房菜</a></li>
      <li><a href="http://billie66.github.io/TLCL/" target="_blank">TLCL</a></li>

      <li><a href="https://gettingreal.37signals.com/GR_chn.php" target="_blank">Getting Real</a></li>
      <li><a href="http://www.tldp.org/LDP/Bash-Beginners-Guide/html" target="_blank">Bash Guide for Beginners</a></li>

      <li><a href="http://www.beiww.com/doc/oss/smart-questions.html" target="_blank">提问的智慧</a></li>
      <li><a href="http://blog.xdite.net/posts/2012/03/17/website-online-todo-1" target="_blank">网站程式上线前需要准备的事</a><span>（需跳墙）</span></li>
    </ul>
  </section>

  <section class="other-sites">
    <p class="title" >友情链接<p>
    <p>
      <a href="http://blog.xdite.net/" target="_blank">xdite</a>
    </p>
  </section>
</div>
