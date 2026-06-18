---
layout: default
title: About
---
Hi! I'm Diego. You might've seen me around as *Dhanton*. I'm a Physicist and Software Engineer from Spain, with over 5 years of professional C++ experience.

I specialize in game programming, with a particular interest in gameplay, multiplayer, and engine programming. Lately, I've been learning [Odin](https://odin-lang.org){:target="_blank" rel="noopener"}, and have rediscovered my love for writing small and simple computer programs.

<section class="projects">
  <h1 class="page-title">Projects</h1>
  <hr>
  <div class="project-grid">
    {% assign sorted_projects = site.projects | sort: "date" | reverse %}
    {% for project in sorted_projects %}
    <a href="{{ project.url | relative_url }}" class="project-card">
      <div class="card-media"
        {% if project.gif %}data-gif="{{ project.gif | relative_url }}"{% endif %}
        style="background-image: url('{{ project.image | relative_url }}');
        {% if project.gif %}--gif-url: url('{{ project.gif | relative_url }}'){% endif %}">
        {% if project.gif %}
          <img src="{{ project.gif | relative_url }}" alt="" class="preload-hint" loading="eager">
        {% endif %}
      </div>
      <div class="card-body">
        <h3>{{ project.title }} ({{ project.date | date: "%Y"}})</h3>
        <p>{{ project.subtitle }}</p>
      </div>
    </a>
    {% endfor %}
  </div>
</section>

<section class="projects">
  <h1 class="page-title">Game Credits</h1>
  <hr>
  <div class="project-grid">
    {% assign sorted_games = site.game_credits | sort: "date" | reverse %}
    {% for game in sorted_games %}
    <a href="{{ game.game_url }}" class="project-card" target="_blank" rel="noopener">
      <div class="card-media"
        style="background-image: url('{{ game.image | relative_url }}');">
      </div>
      <div class="card-body">
        <h3>{{ game.title }}</h3>
        <p>{{ game.subtitle }}</p>
      </div>
    </a>
    {% endfor %}
  </div>
</section>
