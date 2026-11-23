---
permalink: /bdb/
title: "BDB"
excerpt: "A browser and viewer for game files"
layout: splash
author_profile: false
share: false
comments: false
related: false
read_time: false

# Unlisted: not in the navigation, the sitemap or the site search, and search engines are asked not to index it
sitemap: false
search: false
noindex: true

# ---- Change these when there is a new build ----
download_url: "https://drive.google.com/file/d/1rOddmg_wjlrrxXWJoqqUXKnQsTytS7fe/view?usp=sharing"
version: "5.1.2"
updated: 2026-11-23
size: ""            # e.g. "180 MB", shown under the button when set
screenshot: ""      # e.g. /assets/images/bdb/screenshot.webp, shown under the header when set

# Newest first; every entry shows up in the changelog
changelog:
  - version: "5.1.2"
    date: 2026-11-23
    notes:
      - "First build on this page"

# The feature cards (icons: https://fontawesome.com/v5/search?m=free)
features:
  - icon: "fas fa-box-open"
    title: "Archives"
    text: "Point a profile at a game's folder - or any folder on disk - browse it like a folder tree and export what you need."
  - icon: "fas fa-cube"
    title: "Models & animation"
    text: "Meshes, skeletons, animations and collision meshes in a live 3D viewport."
  - icon: "fas fa-magic"
    title: "Effects"
    text: "Particle and mesh effects played back in the viewport, with an importer for Blender."
  - icon: "fas fa-tree"
    title: "World"
    text: "Map data, world sectors with their placed objects, and SpeedTree vegetation."
  - icon: "fas fa-image"
    title: "Textures"
    text: "Textures and UI images, viewable and exportable."
  - icon: "fas fa-volume-up"
    title: "Sound & video"
    text: "Wwise sound banks and videos (Bink 2, WebM, MP4), played right in the app."
  - icon: "fas fa-camera-retro"
    title: "Path tracer"
    text: "Render the viewport with the built-in path tracer and AI denoising."
  - icon: "fas fa-search"
    title: "Search & diff"
    text: "Find text across every text file, and see what a game update added, changed or removed."
  - icon: "fab fa-python"
    title: "Python"
    text: "Script the whole thing for batch exports and your own tools; examples included."
---

<style>
  /* Colours are the dark skin's own: #00adb5 primary, #51555d border, #1a1d24 footer background */
  .bdb-top { margin: 1.5em 0 1em; padding-bottom: 1.5em; border-bottom: 1px solid #51555d; }
  .bdb-top h1 { margin: 0 0 0.4em; font-size: 2.4em; line-height: 0; }
  .bdb-top h1 img { display: block; width: 100%; max-width: 480px; height: auto; }
  .bdb-top .bdb-tagline { margin: 0 0 1.2em; color: #bdbfc3; }
  .bdb-top .btn i { margin-right: 0.4em; }
  .bdb-meta { margin: 1.6em 0 0; font-size: 0.8em; color: #9a9ca1; }
  .bdb-meta span + span::before { content: "|"; margin: 0 0.6em; color: #51555d; }

  .bdb-shot { margin: 1.5em 0; }
  .bdb-shot img { border: 1px solid #51555d; border-radius: 4px; }

  .bdb h2 { padding-bottom: 0.5em; border-bottom: 1px solid #51555d; }

  .bdb-grid { display: grid; grid-template-columns: 1fr; gap: 1em; }
  @media (min-width: 560px) { .bdb-grid { grid-template-columns: repeat(2, 1fr); } }
  @media (min-width: 900px) { .bdb-grid { grid-template-columns: repeat(3, 1fr); } }
  .bdb-card { padding: 1em 1.1em; border: 1px solid #51555d; border-radius: 4px; background: #1a1d24; }
  .bdb-card h3 { margin: 0 0 0.4em; font-size: 1em; }
  .bdb-card h3 i { width: 1.4em; color: #00adb5; }
  .bdb-card p { margin: 0; font-size: 0.85em; color: #bdbfc3; }

  .bdb-release { margin-bottom: 1.2em; }
  .bdb-release h3 { margin: 0 0 0.3em; font-size: 1em; }
  .bdb-release h3 small { margin-left: 0.6em; font-weight: normal; color: #9a9ca1; }
  .bdb-release ul { margin: 0; }

  .bdb-note { margin-top: 2.5em; font-size: 0.8em; color: #9a9ca1; }
  .bdb-disclaimer { margin-top: 0.5em; font-size: 0.75em; font-style: italic; color: #7d8086; }
</style>

<div class="bdb">

  <div class="bdb-top">
    <h1><img src="{{ '/assets/images/bdb/bdb_logo.webp' | relative_url }}" alt="BDB" width="715" height="205"></h1>
    <p class="bdb-tagline">{{ page.excerpt }}</p>
    <a class="btn btn--primary btn--large" href="{{ page.download_url }}" target="_blank" rel="noopener"><i class="fas fa-download"></i>Download {{ page.version }}</a>
    <p class="bdb-meta">
      <span>Version {{ page.version }}</span>
      <span>Updated {{ page.updated | date: "%B %-d, %Y" }}</span>
      {% if page.size != "" %}<span>{{ page.size }}</span>{% endif %}
      <span>Windows 64-bit</span>
    </p>
  </div>

  {% if page.screenshot != "" %}
  <div class="bdb-shot"><img src="{{ page.screenshot | relative_url }}" alt="BDB screenshot"></div>
  {% endif %}

  <h2>What's in it</h2>
  <div class="bdb-grid">
    {% for feature in page.features %}
    <div class="bdb-card">
      <h3><i class="{{ feature.icon }}"></i>{{ feature.title }}</h3>
      <p>{{ feature.text }}</p>
    </div>
    {% endfor %}
  </div>

  <h2>Getting started</h2>
  <ol>
    <li>Download the zip with the button above and extract it into its own folder.</li>
    <li>Start <code>BDBrowser.exe</code>.</li>
    <li>Create a profile for your game's install folder and start browsing.</li>
  </ol>
  <p><code>BDBrowser_Documentation.pdf</code> in the download covers the rest.</p>

  <h2 id="changelog">Changelog</h2>
  {% for release in page.changelog %}
  <div class="bdb-release">
    <h3>{{ release.version }}<small>{{ release.date | date: "%B %-d, %Y" }}</small></h3>
    <ul>
      {% for note in release.notes %}<li>{{ note }}</li>{% endfor %}
    </ul>
  </div>
  {% endfor %}

  <p class="bdb-note">Please don't reupload BDB elsewhere. Send people this page instead, so they always get the latest build.</p>
  <p class="bdb-disclaimer">Disclaimer: the page is meant to be crap.</p>

</div>
