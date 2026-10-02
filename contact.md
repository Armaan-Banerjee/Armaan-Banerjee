---
layout: page
title: Contact
permalink: /contact/
---

{% comment %} Set `email:` in _config.yml and it appears here. {% endcomment %}
{% if site.email != nil and site.email != '' %}
Email: [{{ site.email }}](mailto:{{ site.email }})
{% else %}
Add your address to `email:` in `_config.yml` and it will show up here.
{% endif %}

{% if site.social.github != nil and site.social.github != '' %}
GitHub: [{{ site.social.github }}](https://github.com/{{ site.social.github }})
{% endif %}
