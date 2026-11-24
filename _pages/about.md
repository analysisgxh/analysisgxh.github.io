---
layout: about
title: about
permalink: /
subtitle: <a href='#'>Affiliations</a>. SCUT

profile:
  align: right
  image: prof_pic.jpg
  image_circular: false # crops the image to make it circular
  more_info: >
    <p>SCUT</p>
    <p>SCUT</p>
    <p>SCUT</p>

selected_papers: true # includes a list of papers marked as "selected={true}"
social: true # includes social icons at the bottom of the page

announcements:
  enabled: true # includes a list of news items
  scrollable: true # adds a vertical scroll bar if there are more than 3 news items
  limit: 5 # leave blank to include all the news in the `_news` folder

latest_posts:
  enabled: true
  scrollable: true # adds a vertical scroll bar if there are more than 3 new posts items
  limit: 3 # leave blank to include all the blog posts
---


<style>
  .post-header .post-title,
  .post-header .post-title span {
    font-size: 0 !important;
    line-height: 0 !important;
  }
  .post-header .post-title {
    margin-bottom: 0.6rem;
  }
  .post-header .post-title::after {
    content: "";
    display: inline-block;
    width: min(268px, 80vw);
    aspect-ratio: 362 / 81;
    background: url("{{ '/assets/img/signature.png' | relative_url }}") no-repeat left center / contain;
  }
  html[data-theme="dark"] .post-header .post-title::after {
    filter: invert(1);
  }
</style>

NOTHING

{% comment %}
Write your biography here. Tell the world about yourself. Link to your favorite [subreddit](https://www.reddit.com). You can put a picture in, too. The code is already in, just name your picture `prof_pic.jpg` and put it in the `img/` folder.

Put your address / P.O. box / other info right below your picture. You can also disable any of these elements by editing `profile` property of the YAML header of your `_pages/about.md`. Edit `_bibliography/papers.bib` and Jekyll will render your [publications page](/al-folio/publications/) automatically.

Link to your social media connections, too. This theme is set up to use [Font Awesome icons](https://fontawesome.com/) and [Academicons](https://jpswalsh.github.io/academicons/), like the ones below. Add your Facebook, Twitter, LinkedIn, Google Scholar, or just disable all of them.
{% endcomment %}
