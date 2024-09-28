---
layout: archive
title: "CV"
permalink: /cv/
author_profile: true
redirect_from:
  - /resume
---

{% include base_path %}

Education
======
* Ph.D in Computational Material Science, City University of Hong Kong, 2024
* M.S. in Chemical Engineering, East China University of Science and Technology, 2020
* B.S. in Applied Chemistry, Anhui Jianzhu University, 2017

Work experience
======
* 2024: Postdoc
  * Great Bay University
  * Supervisor: XIA Guangjie


Skills
======
* DFT
* Machine learning
* Graph attention learning

Publications
======
  <ul>{% for post in site.publications reversed %}
    {% include archive-single-cv.html %}
  {% endfor %}</ul>

Talks
======
  <ul>{% for post in site.talks reversed %}
    {% include archive-single-talk-cv.html  %}
  {% endfor %}</ul>

Teaching
======
  <ul>{% for post in site.teaching reversed %}
    {% include archive-single-cv.html %}
  {% endfor %}</ul>

Service and leadership
======
* Reviewer for Nanoscale and npj Computational Materials
