---
# NOTE: This index.md renders the root README.md on GitHub Pages
layout: null
title: Home
---

<link rel="stylesheet" href="assets/site.css" />

<div class="layout">
  <aside class="sidebar">
    <div class="brand">API Docs</div>
    <nav class="nav">
      <a class="active" href="./">All APIs</a>
      <a href="./upscale">Upscale Quickstart</a>
      <a href="./sound-to-video-quickstart">Sound To Video Quickstart</a>
      <a href="./video-add-sound-quickstart">Video Add Sound Quickstart</a>
      <a href="./text-to-speech-quickstart">Text To Speech Quickstart</a>
      <a href="./docs/virtual-influencer">Virtual Influencer Quickstart</a>
      <a href="./docs/image-edit-quickstart">Image Edit Quickstart</a>
    </nav>
  </aside>
  <main markdown="1" class="content">
  {% include_relative docs/Video-Generation.md %}
  {% include_relative docs/virtual-influencer.md %}

  </main>
</div>
