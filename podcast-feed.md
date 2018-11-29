---
layout: raw-html
---
{::nomarkdown} 
<?xml version="1.0" encoding="UTF-8"?>
<rss xmlns:itunes="http://www.itunes.com/dtds/podcast-1.0.dtd" version="2.0">
    <channel>
        <title>CVA Boys Basketball</title>
        <link>https://cva.origas.org</link>
        <language></language>
        <copyright></copyright>
        <itunes:subtitle></itunes:subtitle>
        <itunes:author>Ben Origas</itunes:author>
        <itunes:summary>CVA Boys Basketball Team Podcast</itunes:summary>  
        <itunes:owner>
            <itunes:name>Ben Origas</itunes:name>
            <itunes:email>cvapodcast@benorigas.com</itunes:email>
        </itunes:owner>
        <itunes:image href="https://cva.origas.org/logo.jpg" />
        <itunes:category text="Sports &amp; Recreation"/>
        <itunes:explicit>no</itunes:explicit>
        <image>
            <title>CVA Basketball</title>
            <url>https://cva.origas.org/logo.jpg</url>
            <link>https://cva.origas.org</link>
	    </image>
{:/nomarkdown}
        
{% for podcast in site.data.podcasts %}
        <item>
            <title>{{ podcast.title }}</title>
            <itunes:author>{{ podcast.author }}</itunes:author>
            <itunes:subtitle></itunes:subtitle>
            <itunes:summary>{{ podcast.summary }}</itunes:summary>
            <itunes:image href="" />
            <enclosure url="{{ podcast.audio_file }}" length="{{ podcast.audio_file_length }}" type="{{ podcast.audio_file_duration }}"/>
            <itunes:duration>{{ podcast.audio_file_duration }}</itunes:duration>
            <guid>{{ podcast.original_source }}</guid>
            <pubDate>{{ podcast.publish_date }}</pubDate>
        </item>
{% endfor %}

    </channel>
</rss>