---
title:
layout: default
permalink: /projects/
published: true
---


<div class="ProjectContainer">

	<div class="gallery">


  {% for project in site.projects %}

  {% if project.redirect %}
  <div class="projectTile">
          <span>
          <a href="https://github.com/northboys/cloudflare-grafana" target="_blank">
          <img src="https://github-readme-stats.vercel.app/api/pin/?username=northboys&amp;repo=cloudflare-grafana">
          </a>
          </span>
          <!-- <a href="{{ project.redirect }}" target="_blank">
          <span>
              <h2>{{ project.title }}</h2>
              <br/>
              <p>{{ project.description }}</p>
          </span>
          </a> -->
  </div>

  <!-- {% else %}

  <div class="projectTile">
          <a href="{{ project.url | prepend: site.baseurl | prepend: site.url }}">
          <span>
              <h2>{{ project.title }}</h2>
              <br/>
              <p>{{ project.description }}</p>
          </span>
          </a>
  </div> -->

  {% endif %}

  {% endfor %}

	</div>

</div>
