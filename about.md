---
layout: default
title: "post"
permalink: /about/
---

# Showerbugs는? 
* IT 기술을 공부하고 기술 서적을 읽고 정리하고, 작은 프로젝트를 만들고 있는 모임입니다.
* python, javascript, go, android 등 언어와 플랫폼에 제한을 두지 않고 다양한 분야를 공부하려 합니다.
* 끊임없이 공부하는 개발자가 되려 하고 있습니다.
* 매주 하나의 포스팅을 목표로 하고 있습니다.

## Git
[github.com/showerbugs](https://github.com/showerbugs)

## Member
<div class="about">
  {% for author in site.authors %}
    <div class="about-author">
        <div class="about-author-left-box">
            <img class="about-author-profile" src="/{{author[1].image}}" width="200px" height="200px"/>
        </div>
        <div class="about-author-right-box">
            <h1 class="about-author-nickname">{{author[1].nickname}}
              <span class="about-author-name">{{author[1].name}} </span>
              </h1>
            <div class="about-author-bar"></div>
            <div>{{author[1].bio}}</div>
            <span class="about-author-email">{{author[1].email}}</span>
        <div><a class="about-author-blog-link" href="{{author[1].url}}" target="_blank">
            Blog
         </a></div>
        </div>
    </div>
    {% endfor %}  
</div>

<!-- {% include discuss.html %} -->