## [Podcast Feed](https://cva.origas.org/podcast-feed)
Add https://cva.origas.org/podcast-feed by URL to your podcast player

[Instructions for most podcast players](https://medium.com/@joshmuccio/how-to-manually-add-a-rss-feed-to-your-podcast-app-on-desktop-ios-android-478d197a3770)

## Episode List
{% for podcast in site.data.podcasts %}
### [{{ podcast.title }}]({{ podcast.original_source }})
#### From {{ podcast.author }}
{{ podcast.summary }}
{% endfor %}