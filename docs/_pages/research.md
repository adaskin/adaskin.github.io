---
layout: page
title: Research
permalink: /research/
id: research
---

<link rel="stylesheet" href="{{ '/assets/css/styles.css' | relative_url }}">

Main research interest and themes:
* Quantum compilers and circuit design
* Quantum algorithms
* Quantum optimization
* Quantum machine learning and data analysis
* Matrix computations
* Parallel and distributed computation
  

**Tutorials** 
- [A brief tutorial on grouping set elements on quantum computers](https://adaskin.github.io/quantum-grouping-set-elements)  
  - *it generates groups as quantum states in O(1) time*

**Latest Research Papers**   
<div class="research-papers">
  {% for paper in site.research_papers %}
    <div class="paper-card">
      {% if paper.image %}
      <img src="{{ paper.image }}" alt="{{ paper.title }}" class="paper-image">
      {% endif %}
      <div class="paper-content">
        <h3>{{ paper.title }}</h3>
        <p class="meta">By {{ paper.author }} • {{ paper.date | date: "%B %Y" }}</p>
        <p class="abstract">{{ paper.abstract }}</p>
        {% if paper.link %}
        <a href="{{ paper.link }}">Read full paper</a>
        {% endif %}
        {% if paper.cite %}
        <p class="meta">{{ paper.cite }}</p>
        {% endif %}
      </div>
    </div>
  {% endfor %}
</div>

