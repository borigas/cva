
{% for podcast in site.data.podcasts %}
## [{{ podcast.title }}]({{ podcast.original_source }})
### From {{ podcast.author }}
{{ podcast.summary }}

{% endfor %}