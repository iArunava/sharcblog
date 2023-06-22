---
title: Thoughts
description: |
    This section lists down some of my thoughts.
---

<ul>
    {% for language in site.thoughts %}
        <li>
            <h2>
                <a href="{{ language.url | relative_url }}">
                    {{ language.title }}
                </a>
            </h2>

            <p>
                <i>{{ language.description }}</i>
            </p>
            <p>{{ language.excerpt }}</p>
        </li>
    {% endfor %}
</ul>
