---
title: Cases & Enclosures
description: Complete catalog of BC-250 cases, enclosures, and 3D-printable designs — searchable and filterable by PSU type, availability, and platform.
tags:
  - cases
  - enclosures
  - 3D printing
  - PSU
hide:
  - toc
---

# Cases & Enclosures

Community-built and 3D-printable enclosures for the AMD BC-250. **145 designs** documented from Discord, Reddit, Printables, MakerWorld, Thingiverse, and more.

<div class="cases-app" markdown="0">

<div class="cases-toolbar" id="cases-toolbar">
  <div class="cases-search-wrap">
    <span class="cases-search-icon">&#x2315;</span>
    <input type="text" id="cases-search" placeholder="Search by name, author, PSU…" autocomplete="off" spellcheck="false">
    <button class="cases-clear-btn" id="cases-clear-btn" title="Clear search">&#x2715;</button>
  </div>
  <div class="cases-view-toggle" id="cases-view-toggle">
    <button class="view-btn active" data-view="grid" title="Grid view">Grid</button>
    <button class="view-btn" data-view="list" title="List view">List</button>
  </div>
</div>

<div class="cases-filters" id="cases-filters">
  <div class="filter-group filter-group-scroll">
    <span class="filter-label">PSU</span>
    <div class="filter-chips filter-chips-scroll" id="filter-psu">
      <button class="chip active" data-filter="psu" data-value="all">All</button>
      <button class="chip chip-lop" data-filter="psu" data-value="lop">MeanWell LOP</button>
      <button class="chip chip-lrs" data-filter="psu" data-value="lrs">LRS / UHP</button>
      <button class="chip chip-flexatx" data-filter="psu" data-value="flexatx">FlexATX</button>
      <button class="chip chip-atx" data-filter="psu" data-value="atx">Full ATX</button>
      <button class="chip chip-tfx" data-filter="psu" data-value="tfx">TFX</button>
      <button class="chip chip-hp" data-filter="psu" data-value="hp-server">HP Server</button>
      <button class="chip chip-sfx" data-filter="psu" data-value="sfx">SFX</button>
      <button class="chip chip-other" data-filter="psu" data-value="other">Other</button>
    </div>
  </div>
  <div class="filter-group">
    <span class="filter-label">Availability</span>
    <div class="filter-chips" id="filter-avail">
      <button class="chip active" data-filter="avail" data-value="all">All</button>
      <button class="chip chip-public" data-filter="avail" data-value="public">Free</button>
      <button class="chip chip-discord" data-filter="avail" data-value="discord">Discord</button>
      <button class="chip chip-wip" data-filter="avail" data-value="wip">WIP</button>
    </div>
  </div>
  <div class="filter-group">
    <span class="filter-label">Sort</span>
    <div class="filter-chips" id="filter-sort">
      <button class="chip active" data-filter="sort" data-value="default">Category</button>
      <button class="chip" data-filter="sort" data-value="alpha">A–Z</button>
      <button class="chip" data-filter="sort" data-value="platform">Platform</button>
    </div>
  </div>
</div>
<div class="cases-stats" id="cases-stats">Loading…</div>

<div class="cases-grid" id="cases-grid">
  <div class="cases-loading">Loading cases…</div>
</div>

<div class="cases-empty" id="cases-empty" style="display:none">
  <div class="empty-icon">&#x2315;</div>
  <div class="empty-text">No cases match your filters</div>
  <div class="empty-active" id="empty-active"></div>
  <button class="empty-reset" id="empty-reset">Reset all filters</button>
</div>

</div>

<style>
/* ── Cases App ── */
.cases-app { margin-top: 1.5rem; width: 100%; }

/* Sticky header wrapper — top set via JS from actual header height */
.cases-toolbar, .cases-filters {
  position: sticky; z-index: 10;
  background: var(--md-default-bg-color);
}
.cases-toolbar {
  top: var(--cases-toolbar-top, 3.5rem);
  padding: 0.5rem 0;
}
.cases-filters {
  top: var(--cases-filters-top, 6.9rem);
  padding-bottom: 0.5rem;
}

/* Toolbar */
.cases-toolbar {
  display: flex; align-items: center; gap: 0.75rem;
  flex-wrap: wrap;
}
.cases-search-wrap {
  position: relative; flex: 1 1 220px; min-width: 180px;
}
.cases-search-icon {
  position: absolute; left: 0.65rem; top: 50%;
  transform: translateY(-50%); font-size: 0.85rem;
  pointer-events: none; opacity: 0.5;
}
#cases-search {
  width: 100%; padding: 0.45rem 2.2rem 0.45rem 2.1rem;
  border: 1.5px solid var(--md-default-fg-color--lightest);
  border-radius: 6px; font-size: 0.9rem;
  background: var(--md-default-bg-color);
  color: var(--md-default-fg-color);
  transition: border-color 0.2s;
}
#cases-search:focus {
  outline: none;
  border-color: var(--md-accent-fg-color);
}
.cases-clear-btn {
  position: absolute; right: 0.5rem; top: 50%;
  transform: translateY(-50%); background: none;
  border: none; cursor: pointer; font-size: 1rem;
  color: var(--md-default-fg-color--light);
  display: none; padding: 0.2rem;
}

/* View toggle */
.cases-view-toggle {
  display: flex; gap: 0; border: 1.5px solid var(--md-default-fg-color--lightest);
  border-radius: 6px; overflow: hidden; flex-shrink: 0;
}
.view-btn {
  padding: 0.32rem 0.65rem; border: none; background: transparent;
  cursor: pointer; font-size: 0.78rem; color: var(--md-default-fg-color--light);
  white-space: nowrap; transition: all 0.15s;
}
.view-btn:not(:last-child) { border-right: 1.5px solid var(--md-default-fg-color--lightest); }
.view-btn:hover { color: var(--md-accent-fg-color); }
.view-btn.active {
  background: var(--md-accent-fg-color); color: #fff;
}

/* Stats */
.cases-stats {
  font-size: 0.78rem; color: var(--md-default-fg-color--light);
  padding: 0.2rem 0 0.6rem; white-space: nowrap;
}

/* Filters */
.cases-filters { margin-bottom: 0; }
.filter-group {
  display: flex; align-items: center; gap: 0.4rem;
  margin-bottom: 0.4rem; flex-wrap: wrap;
}
.filter-group-scroll {
  flex-wrap: nowrap;
}
.filter-chips-scroll {
  overflow-x: auto; white-space: nowrap; flex-wrap: nowrap;
  -webkit-overflow-scrolling: touch; scrollbar-width: none;
  display: flex; gap: 0.3rem;
}
.filter-chips-scroll::-webkit-scrollbar { display: none; }
.filter-label {
  font-size: 0.72rem; font-weight: 600;
  text-transform: uppercase; letter-spacing: 0.04em;
  color: var(--md-default-fg-color--light);
  min-width: 5rem; flex-shrink: 0;
}
.chip {
  padding: 0.2rem 0.55rem; border-radius: 999px;
  border: 1.5px solid var(--md-default-fg-color--lightest);
  background: transparent; cursor: pointer;
  font-size: 0.75rem; color: var(--md-default-fg-color--light);
  transition: all 0.15s; white-space: nowrap; flex-shrink: 0;
}
.chip:hover { border-color: var(--md-accent-fg-color); color: var(--md-accent-fg-color); }
.chip.active {
  background: var(--md-accent-fg-color);
  border-color: var(--md-accent-fg-color);
  color: #fff;
}

/* ── Grid view ── */
.cases-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(160px, 1fr));
  gap: 0.7rem;
  width: 100%;
}
.cases-loading {
  grid-column: 1 / -1; text-align: center;
  padding: 3rem; color: var(--md-default-fg-color--light);
}

/* Section Headers */
.cases-section-header {
  grid-column: 1 / -1;
  font-size: 0.9rem; font-weight: 700;
  padding: 0.5rem 0 0.25rem;
  border-bottom: 2px solid var(--md-accent-fg-color);
  margin-top: 0.3rem;
  color: var(--md-default-fg-color);
}
.cases-section-header:first-child { margin-top: 0; }

/* Card (grid) */
.case-card {
  border: 1px solid var(--md-default-fg-color--lightest);
  border-radius: 6px; overflow: hidden;
  background: var(--md-default-bg-color);
  transition: box-shadow 0.2s, border-color 0.2s;
  display: flex; flex-direction: column;
  position: relative;
}
.case-card-slot.hovered .case-card {
  overflow: visible;
}

/* Thumbnail (grid) */
.case-thumb {
  height: 125px; min-height: 125px; max-height: 125px;
  overflow: hidden; position: relative;
  background: var(--md-default-fg-color--lightest);
  border-radius: 6px 6px 0 0;
  flex-shrink: 0;
}
.case-thumb img {
  position: absolute; top: 0; left: 0;
  width: 100%; height: 100%; object-fit: cover;
  opacity: 0; transition: opacity 0.3s;
}
.case-thumb img.loaded { opacity: 1; }
.case-thumb-placeholder {
  position: absolute; top: 0; left: 0;
  width: 100%; height: 100%;
  display: flex; align-items: center; justify-content: center;
  font-size: 1.6rem; font-weight: 700; color: #fff;
  letter-spacing: 0.05em;
}

/* Shimmer skeleton */
.case-thumb-shimmer {
  position: absolute; inset: 0;
  background: linear-gradient(90deg,
    var(--md-default-fg-color--lightest) 25%,
    var(--md-default-bg-color) 50%,
    var(--md-default-fg-color--lightest) 75%);
  background-size: 200% 100%;
  animation: shimmer 1.5s infinite;
}
@keyframes shimmer {
  0% { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}

/* Card body (grid) */
.case-body { padding: 0.4rem 0.5rem 0.45rem; flex: 1; display: flex; flex-direction: column; }
.case-name {
  font-weight: 700; font-size: 0.82rem; line-height: 1.25;
  display: -webkit-box; -webkit-line-clamp: 1;
  -webkit-box-orient: vertical; overflow: hidden;
  margin-bottom: 0.1rem;
}
.case-name mark {
  background: rgba(255, 213, 79, 0.35); padding: 0 1px;
  border-radius: 2px;
}
.case-author {
  font-size: 0.7rem; color: var(--md-default-fg-color--light);
  margin-bottom: 0.25rem;
}
.case-badges { display: flex; gap: 0.25rem; flex-wrap: wrap; margin-bottom: 0.25rem; }
.badge {
  font-size: 0.6rem; padding: 0.08rem 0.35rem;
  border-radius: 999px; font-weight: 600;
  white-space: nowrap;
}
.badge-psu { background: var(--md-default-fg-color--lightest); color: var(--md-default-fg-color--light); }
.badge-avail-public { background: #e8f5e9; color: #2e7d32; }
.badge-avail-discord { background: #ede7f6; color: #5e35b1; }
.badge-avail-commercial { background: #fff3e0; color: #e65100; }
.badge-avail-wip { background: #fff8e1; color: #f9a825; }
.badge-avail-request { background: #fce4ec; color: #c62828; }
.badge-avail-unpublished { background: #eceff1; color: #546e7a; }
.badge-avail-unknown { background: #eceff1; color: #78909c; }

[data-md-color-scheme="slate"] .badge-avail-public { background: #1b5e20; color: #a5d6a7; }
[data-md-color-scheme="slate"] .badge-avail-discord { background: #311b92; color: #b39ddb; }
[data-md-color-scheme="slate"] .badge-avail-commercial { background: #bf360c; color: #ffcc80; }
[data-md-color-scheme="slate"] .badge-avail-wip { background: #f57f17; color: #fff9c4; }
[data-md-color-scheme="slate"] .badge-avail-request { background: #b71c1c; color: #ef9a9a; }
[data-md-color-scheme="slate"] .badge-avail-unpublished { background: #37474f; color: #b0bec5; }
[data-md-color-scheme="slate"] .badge-avail-unknown { background: #37474f; color: #90a4ae; }

/* ── Remix badge ── */
.case-remix-badge {
  display: inline-block;
  background: transparent;
  color: var(--md-accent-fg-color);
  border: 1px solid currentColor;
  font-size: 0.58rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.06em;
  padding: 0.1rem 0.35rem;
  border-radius: 3px;
  margin-left: 0.35rem;
  vertical-align: middle;
  opacity: 0.8;
}
.badge-remix {
  background: linear-gradient(135deg, #ff6b35, #f7c59f);
  color: #fff;
}

/* ── Card slot wrapper — reserves grid space ── */
.case-card-slot {
  position: relative;
  height: 320px; /* fixed height — all cards identical */
}
.case-card-slot > .case-card {
  height: 100%;
}

/* ── Hover: card floats above grid ── */
.case-card-slot.hovered .case-card {
  position: absolute;
  top: 0; left: 0; width: 100%;
  height: auto; /* allow expansion */
  z-index: 20;
  box-shadow: 0 8px 28px rgba(0,0,0,0.18);
  border-color: var(--md-accent-fg-color);
}
[data-md-color-scheme="slate"] .case-card-slot.hovered .case-card {
  box-shadow: 0 8px 28px rgba(0,0,0,0.4);
}

/* Description */
.case-desc-wrap {
  font-size: 0.7rem; line-height: 1.35;
  color: var(--md-default-fg-color--light);
  margin-bottom: 0.25rem;
}
.case-desc-text {
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
  transition: all 0.15s ease;
}
/* Unclamp description when card is elevated */
.case-card-slot.hovered .case-desc-text {
  display: block;
  -webkit-line-clamp: unset;
  overflow: visible;
}

.case-links { display: flex; gap: 0.25rem; flex-wrap: wrap; }
.case-link {
  font-size: 0.63rem; padding: 0.1rem 0.3rem;
  border-radius: 4px; text-decoration: none;
  border: 1px solid var(--md-default-fg-color--lightest);
  color: var(--md-default-fg-color--light);
  transition: all 0.15s; white-space: nowrap;
}
.case-link:hover {
  border-color: var(--md-accent-fg-color);
  color: var(--md-accent-fg-color);
}
.link-printables { color: #fa6831; border-color: #fa683166; }
.link-printables:hover { background: #fa683115; }
.link-makerworld { color: #1a73e8; border-color: #1a73e866; }
.link-makerworld:hover { background: #1a73e815; }
.link-thingiverse { color: #248bfb; border-color: #248bfb66; }
.link-thingiverse:hover { background: #248bfb15; }
.link-github { color: #6e5494; border-color: #6e549466; }
.link-github:hover { background: #6e549415; }
.link-reddit { color: #ff4500; border-color: #ff450066; }
.link-reddit:hover { background: #ff450015; }
.link-discord { color: #5865f2; border-color: #5865f266; }
.link-discord:hover { background: #5865f215; }
.discord-only-label {
  font-size: 0.7rem;
  color: var(--md-default-fg-color--light, #888);
  font-style: italic;
  padding: 0.1rem 0;
  cursor: default;
  user-select: none;
}
.cases-discord-divider {
  grid-column: 1 / -1;
  display: flex;
  flex-direction: column;
  gap: 0.2rem;
  padding: 0.75rem 0 0.5rem;
  margin-top: 0.5rem;
  border-top: 1px dashed var(--md-default-fg-color--lighter, #ccc);
}
.cases-discord-divider span {
  font-size: 0.8rem;
  font-weight: 600;
  color: var(--md-default-fg-color--light, #888);
  text-transform: uppercase;
  letter-spacing: 0.05em;
}
.cases-discord-divider small {
  font-size: 0.72rem;
  color: var(--md-default-fg-color--lighter, #aaa);
}
.card-discord-only {
  opacity: 0.7;
}
.card-discord-only:hover {
  opacity: 1;
}
.link-ebay { color: #e53238; border-color: #e5323866; }
.link-ebay:hover { background: #e5323815; }
.link-etsy { color: #f1641e; border-color: #f1641e66; }
.link-etsy:hover { background: #f1641e15; }
.link-cults3d { color: #9c27b0; border-color: #9c27b066; }
.link-cults3d:hover { background: #9c27b015; }
.link-gdrive { color: #34a853; border-color: #34a85366; }
.link-gdrive:hover { background: #34a85315; }

/* ── List view ── */
.cases-grid.list-view {
  display: flex; flex-direction: column; gap: 0;
}
.cases-grid.list-view .cases-section-header {
  padding: 0.5rem 0.4rem 0.2rem; margin-top: 0.4rem;
}
.cases-grid.list-view .case-card {
  flex-direction: row; border-radius: 0;
  border: none; border-bottom: 1px solid var(--md-default-fg-color--lightest);
  min-height: 56px; align-items: center;
}
.cases-grid.list-view .case-card-slot:nth-child(even) .case-card {
  background: color-mix(in srgb, var(--md-default-fg-color--lightest) 20%, transparent);
}
.cases-grid.list-view .case-card:hover {
  box-shadow: none;
  background: color-mix(in srgb, var(--md-accent-fg-color) 8%, transparent);
}
/* Disable float-above hover in list view */
.cases-grid.list-view .case-card-slot.hovered .case-card {
  position: relative;
  transform: none;
  box-shadow: none;
}
.cases-grid.list-view .case-thumb {
  width: 80px; min-width: 80px; height: 56px;
  border-radius: 4px; margin: 0; flex-shrink: 0;
}
.cases-grid.list-view .case-thumb-placeholder { font-size: 1rem; }
.cases-grid.list-view .case-body {
  flex-direction: row; align-items: center; gap: 0.6rem;
  padding: 0.3rem 0.6rem; flex-wrap: wrap;
}
.cases-grid.list-view .case-name {
  flex: 1 1 140px; min-width: 100px; margin-bottom: 0;
  -webkit-line-clamp: 1; font-size: 0.82rem;
}
.cases-grid.list-view .case-author {
  flex: 0 0 auto; margin-bottom: 0; font-size: 0.7rem;
  min-width: 70px;
}
.cases-grid.list-view .case-badges {
  flex: 0 0 auto; margin-bottom: 0; gap: 0.2rem;
}
.cases-grid.list-view .case-links {
  flex: 0 0 auto; gap: 0.2rem;
}
.cases-grid.list-view .case-desc-wrap { display: none; }

/* Empty state */
.cases-empty {
  text-align: center; padding: 3rem 1rem;
}
.empty-icon { font-size: 3rem; margin-bottom: 0.5rem; opacity: 0.4; }
.empty-text { font-size: 1.1rem; font-weight: 600; margin-bottom: 0.4rem; }
.empty-active {
  font-size: 0.85rem; color: var(--md-default-fg-color--light);
  margin-bottom: 1rem;
}
.empty-reset {
  padding: 0.4rem 1.2rem; border-radius: 6px;
  border: 1.5px solid var(--md-accent-fg-color);
  background: transparent; cursor: pointer;
  color: var(--md-accent-fg-color); font-size: 0.85rem;
}
.empty-reset:hover { background: var(--md-accent-fg-color); color: #fff; }

/* PSU color backgrounds for placeholders */
.psu-bg-lop { background: #e65100; }
.psu-bg-lrs { background: #1565c0; }
.psu-bg-flexatx { background: #2e7d32; }
.psu-bg-atx { background: #6a1b9a; }
.psu-bg-tfx { background: #00838f; }
.psu-bg-hp-server { background: #283593; }
.psu-bg-sfx { background: #4e342e; }
.psu-bg-server-1f { background: #37474f; }
.psu-bg-led-driver { background: #f9a825; }
.psu-bg-ps3 { background: #1a237e; }
.psu-bg-other { background: #546e7a; }

@media (max-width: 600px) {
  .cases-grid { grid-template-columns: 1fr 1fr; gap: 0.5rem; }
  .case-thumb { height: 100px; min-height: 100px; max-height: 100px; }
  .case-body { padding: 0.3rem 0.4rem 0.4rem; }
  .case-name { font-size: 0.75rem; }
  .filter-label { min-width: auto; }
  .cases-grid.list-view .case-thumb { width: 60px; min-width: 60px; height: 44px; }
  .cases-grid.list-view .case-body { gap: 0.3rem; padding: 0.2rem 0.4rem; }
}
@media (max-width: 400px) {
  .cases-grid { grid-template-columns: 1fr; }
}
</style>

<script>
(function() {
  "use strict";

  var CASES = [];

  var PSU_LABELS = {
    lop: "MeanWell LOP", lrs: "LRS/UHP", flexatx: "FlexATX",
    atx: "Full ATX", tfx: "TFX", "hp-server": "HP Server",
    sfx: "SFX", "server-1f": "Server 1F", "led-driver": "LED Driver",
    ps3: "PS3 PSU", other: "Other"
  };
  var AVAIL_LABELS = {
    public: "Free", discord: "Discord", commercial: "Paid",
    wip: "WIP", request: "On request", unpublished: "Unpublished", unknown: "Unknown"
  };
  var LINK_META = {
    printables:  { label: "Printables",  cls: "link-printables",  icon: "" },
    makerworld:  { label: "MakerWorld",  cls: "link-makerworld",  icon: "" },
    thingiverse: { label: "Thingiverse", cls: "link-thingiverse", icon: "" },
    github:      { label: "GitHub",      cls: "link-github",      icon: "" },
    reddit:      { label: "Reddit",      cls: "link-reddit",      icon: "" },
    ebay:        { label: "eBay",        cls: "link-ebay",        icon: "" },
    etsy:        { label: "Etsy",        cls: "link-etsy",        icon: "" },
    cults3d:     { label: "Cults3D",     cls: "link-cults3d",     icon: "" },
    gdrive:      { label: "Drive",       cls: "link-gdrive",      icon: "" },
    discord:     { label: "Discord",     cls: "link-discord",     icon: "" }
  };

  /* ── State ── */
  var filterPsu = "all";
  var filterAvail = "all";
  var filterSort = "default";
  var searchQuery = "";

  /* ── DOM refs ── */
  var grid = document.getElementById("cases-grid");
  var stats = document.getElementById("cases-stats");
  var searchInput = document.getElementById("cases-search");
  var clearBtn = document.getElementById("cases-clear-btn");
  var emptyEl = document.getElementById("cases-empty");
  var emptyActive = document.getElementById("empty-active");
  var emptyReset = document.getElementById("empty-reset");
  var viewToggle = document.getElementById("cases-view-toggle");
  var currentView = localStorage.getItem("bc250-view") || "grid";

  /* ── Search scoring ── */
  function scoreCase(c, q) {
    if (!q) return 0;
    var ql = q.toLowerCase();
    var score = 0;
    var nl = c.name.toLowerCase();
    if (nl === ql) score += 100;
    else if (nl.indexOf(ql) === 0) score += 60;
    else if (nl.indexOf(ql) > -1) score += 40;
    if (c.author.toLowerCase().indexOf(ql) > -1) score += 30;
    if (c.psu_label.toLowerCase().indexOf(ql) > -1) score += 25;
    if (c.section.toLowerCase().indexOf(ql) > -1) score += 20;
    if (c.description.toLowerCase().indexOf(ql) > -1) score += 10;
    return score;
  }

  function highlightName(name, q) {
    if (!q) return escHtml(name);
    var idx = name.toLowerCase().indexOf(q.toLowerCase());
    if (idx === -1) return escHtml(name);
    var before = name.slice(0, idx);
    var match = name.slice(idx, idx + q.length);
    var after = name.slice(idx + q.length);
    return escHtml(before) + "<mark>" + escHtml(match) + "</mark>" + escHtml(after);
  }

  function escHtml(s) {
    var d = document.createElement("div");
    d.textContent = s;
    return d.innerHTML;
  }

  /* ── Initials for placeholder ── */
  function getInitials(name) {
    var words = name.replace(/[^a-zA-Z0-9\s]/g, "").split(/\s+/).filter(Boolean);
    if (words.length === 0) return "?";
    if (words.length === 1) return words[0].slice(0, 2).toUpperCase();
    return (words[0][0] + words[1][0]).toUpperCase();
  }

  /* ── Filter + sort ── */
  function getFiltered() {
    var results = [];
    for (var i = 0; i < CASES.length; i++) {
      var c = CASES[i];
      if (c.availability === "commercial") continue;
      if (filterPsu !== "all" && c.psu_family !== filterPsu) continue;
      if (filterAvail !== "all") {
        if (filterAvail === "public" && c.availability !== "public") continue;
        if (filterAvail === "discord" && c.availability !== "discord") continue;
        if (filterAvail === "wip" && c.availability !== "wip") continue;
      }
      var score = searchQuery ? scoreCase(c, searchQuery) : 1;
      if (searchQuery && score === 0) continue;
      results.push({ c: c, score: score });
    }

    if (searchQuery) {
      results.sort(function(a, b) { return b.score - a.score; });
    } else if (filterSort === "alpha") {
      results.sort(function(a, b) { return a.c.name.localeCompare(b.c.name); });
    } else if (filterSort === "platform") {
      results.sort(function(a, b) {
        var pa = Object.keys(a.c.links).length;
        var pb = Object.keys(b.c.links).length;
        return pb - pa;
      });
    }
    return results;
  }

  /* ── Render ── */
  function renderCard(c) {
    var card = document.createElement("div");
    card.className = "case-card";

    // Thumbnail
    var thumbDiv = document.createElement("div");
    thumbDiv.className = "case-thumb";
    if (c.thumbnail) {
      var shimmer = document.createElement("div");
      shimmer.className = "case-thumb-shimmer";
      thumbDiv.appendChild(shimmer);
      var img = document.createElement("img");
      img.dataset.src = c.thumbnail;
      img.alt = c.name;
      img.onload = function() {
        img.classList.add("loaded");
        if (shimmer.parentNode) shimmer.parentNode.removeChild(shimmer);
      };
      img.onerror = function() {
        if (shimmer.parentNode) shimmer.parentNode.removeChild(shimmer);
        thumbDiv.innerHTML = "";
        var ph = document.createElement("div");
        ph.className = "case-thumb-placeholder psu-bg-" + c.psu_family;
        ph.textContent = getInitials(c.name);
        thumbDiv.appendChild(ph);
      };
      thumbDiv.appendChild(img);
    } else {
      var ph = document.createElement("div");
      ph.className = "case-thumb-placeholder psu-bg-" + c.psu_family;
      ph.textContent = getInitials(c.name);
      thumbDiv.appendChild(ph);
    }
    card.appendChild(thumbDiv);

    var body = document.createElement("div");
    body.className = "case-body";

    // Name line with remix badge
    var nameEl = document.createElement("div");
    nameEl.className = "case-name";
    nameEl.innerHTML = highlightName(c.name, searchQuery);
    if (c.is_remix) {
      var remixInline = document.createElement("span");
      remixInline.className = "case-remix-badge";
      remixInline.textContent = "REMIX";
      nameEl.appendChild(remixInline);
    }
    body.appendChild(nameEl);

    var author = document.createElement("div");
    author.className = "case-author";
    author.textContent = c.author;
    body.appendChild(author);

    // Description — 2-line clamp + hover tooltip for full text
    if (c.description) {
      var descWrap = document.createElement("div");
      descWrap.className = "case-desc-wrap";
      var descText = document.createElement("div");
      descText.className = "case-desc-text";
      descText.textContent = c.description;
      descWrap.appendChild(descText);
      body.appendChild(descWrap);
    }

    var badges = document.createElement("div");
    badges.className = "case-badges";
    var psuBadge = document.createElement("span");
    psuBadge.className = "badge badge-psu";
    psuBadge.textContent = c.psu_label || PSU_LABELS[c.psu_family] || c.psu_family;
    badges.appendChild(psuBadge);
    var availBadge = document.createElement("span");
    availBadge.className = "badge badge-avail-" + c.availability;
    availBadge.textContent = AVAIL_LABELS[c.availability] || c.availability;
    badges.appendChild(availBadge);
    if (c.is_remix) {
      var remixBadge = document.createElement("span");
      remixBadge.className = "badge badge-remix";
      remixBadge.textContent = "Remix";
      badges.appendChild(remixBadge);
    }
    body.appendChild(badges);

    var links = document.createElement("div");
    links.className = "case-links";
    var platforms = ["printables","makerworld","thingiverse","github","reddit","ebay","etsy","cults3d","gdrive","discord"];
    for (var p = 0; p < platforms.length; p++) {
      var key = platforms[p];
      if (c.links && c.links[key]) {
        var a = document.createElement("a");
        a.className = "case-link " + (LINK_META[key] ? LINK_META[key].cls : "");
        a.href = c.links[key];
        a.target = "_blank";
        a.rel = "noopener";
        a.textContent = (LINK_META[key] ? LINK_META[key].label : key);
        links.appendChild(a);
      }
    }
    if (links.children.length === 0) {
      var span = document.createElement("span");
      span.className = "discord-only-label";
      span.textContent = "Discord only";
      links.appendChild(span);
    }
    body.appendChild(links);
    card.appendChild(body);

    // Wrap in slot div to reserve grid space on hover
    var slot = document.createElement("div");
    slot.className = "case-card-slot";
    slot.appendChild(card);

    // Freeze slot height on hover so grid doesn't reflow
    slot.addEventListener("mouseenter", function() {
      if (grid.classList.contains("list-view")) return;
      slot.style.minHeight = slot.offsetHeight + "px";
      slot.classList.add("hovered");
    });
    slot.addEventListener("mouseleave", function() {
      slot.classList.remove("hovered");
      slot.style.minHeight = "";
    });

    return slot;
  }

  function render() {
    var results = getFiltered();
    grid.innerHTML = "";

    if (results.length === 0) {
      grid.style.display = "none";
      emptyEl.style.display = "";
      var parts = [];
      if (filterPsu !== "all") parts.push("PSU: " + (PSU_LABELS[filterPsu] || filterPsu));
      if (filterAvail !== "all") parts.push("Availability: " + (AVAIL_LABELS[filterAvail] || filterAvail));
      if (searchQuery) parts.push('Search: "' + searchQuery + '"');
      emptyActive.textContent = parts.length ? "Active filters: " + parts.join(" + ") : "";
      return;
    }

    grid.style.display = "";
    emptyEl.style.display = "none";

    // Separate Discord-only from public designs
    function isDiscordOnly(c) {
      var links = c.links || {};
      var keys = Object.keys(links);
      return keys.length === 0 || (keys.length === 1 && keys[0] === 'discord');
    }

    var publicResults = results.filter(function(r) { return !isDiscordOnly(r.c); });
    var discordResults = results.filter(function(r) { return isDiscordOnly(r.c); });

    function renderGroup(items, grouped) {
      if (grouped && filterSort === "default" && !searchQuery) {
        var sections = [];
        var sectionMap = {};
        for (var i = 0; i < items.length; i++) {
          var sec = items[i].c ? items[i].c.section : items[i].section;
          var card = items[i].c || items[i];
          if (!sectionMap[sec]) { sectionMap[sec] = []; sections.push(sec); }
          sectionMap[sec].push(card);
        }
        for (var s = 0; s < sections.length; s++) {
          var header = document.createElement("div");
          header.className = "cases-section-header";
          header.textContent = sections[s];
          grid.appendChild(header);
          var group = sectionMap[sections[s]];
          for (var j = 0; j < group.length; j++) grid.appendChild(renderCard(group[j]));
        }
      } else {
        for (var i = 0; i < items.length; i++) {
          var card = items[i].c || items[i];
          grid.appendChild(renderCard(card));
        }
      }
    }

    renderGroup(publicResults, true);

    if (discordResults.length > 0) {
      var divider = document.createElement("div");
      divider.className = "cases-discord-divider";
      divider.innerHTML = '<span>Discord community projects</span><small>These designs are shared on Discord — join the server to find them</small>';
      grid.appendChild(divider);
      for (var i = 0; i < discordResults.length; i++) {
        var slot = renderCard(discordResults[i].c || discordResults[i]);
        slot.querySelector('.case-card').classList.add('card-discord-only');
        grid.appendChild(slot);
      }
    }

    // Update stats
    var total = CASES.length;
    var shown = results.length;
    var freeCount = 0;
    var discordCount = 0;
    var photoCount = 0;
    var remixCount = 0;
    for (var i = 0; i < results.length; i++) {
      if (results[i].c.availability === "public") freeCount++;
      if (results[i].c.availability === "discord") discordCount++;
      if (results[i].c.thumbnail) photoCount++;
      if (results[i].c.is_remix) remixCount++;
    }

    var statsText = shown === total
      ? total + " cases"
      : shown + " of " + total + " cases";
    statsText += " \u00B7 " + photoCount + " with photos \u00B7 " + freeCount + " free \u00B7 " + discordCount + " Discord";
    if (remixCount > 0) statsText += " \u00B7 " + remixCount + " remixes";
    stats.textContent = statsText;

    // Lazy load images
    lazyLoad();
  }

  /* ── Lazy loading with IntersectionObserver ── */
  var observer = null;
  function lazyLoad() {
    if (observer) observer.disconnect();
    var images = grid.querySelectorAll("img[data-src]");
    if (!("IntersectionObserver" in window)) {
      for (var i = 0; i < images.length; i++) {
        images[i].src = images[i].dataset.src;
        delete images[i].dataset.src;
      }
      return;
    }
    observer = new IntersectionObserver(function(entries) {
      for (var i = 0; i < entries.length; i++) {
        if (entries[i].isIntersecting) {
          var img = entries[i].target;
          img.src = img.dataset.src;
          delete img.dataset.src;
          observer.unobserve(img);
        }
      }
    }, { rootMargin: "200px" });
    for (var i = 0; i < images.length; i++) {
      observer.observe(images[i]);
    }
  }

  /* ── Event handlers ── */
  document.querySelectorAll(".chip[data-filter]").forEach(function(chip) {
    chip.addEventListener("click", function() {
      var filterType = chip.dataset.filter;
      var value = chip.dataset.value;
      var parent = chip.parentNode;
      parent.querySelectorAll(".chip").forEach(function(c) { c.classList.remove("active"); });
      chip.classList.add("active");
      if (filterType === "psu") filterPsu = value;
      else if (filterType === "avail") filterAvail = value;
      else if (filterType === "sort") filterSort = value;
      render();
    });
  });

  var searchTimeout;
  searchInput.addEventListener("input", function() {
    clearTimeout(searchTimeout);
    searchTimeout = setTimeout(function() {
      searchQuery = searchInput.value.trim();
      clearBtn.style.display = searchQuery ? "" : "none";
      render();
    }, 150);
  });

  clearBtn.addEventListener("click", function() {
    searchInput.value = "";
    searchQuery = "";
    clearBtn.style.display = "none";
    render();
    searchInput.focus();
  });

  emptyReset.addEventListener("click", function() {
    filterPsu = "all";
    filterAvail = "all";
    searchQuery = "";
    searchInput.value = "";
    clearBtn.style.display = "none";
    document.querySelectorAll(".chip[data-filter]").forEach(function(c) {
      c.classList.remove("active");
      if (c.dataset.value === "all" || (c.dataset.filter === "sort" && c.dataset.value === "default")) {
        c.classList.add("active");
      }
    });
    render();
  });

  /* ── View toggle ── */
  function applyView(mode) {
    currentView = mode;
    localStorage.setItem("bc250-view", mode);
    if (mode === "list") {
      grid.classList.add("list-view");
    } else {
      grid.classList.remove("list-view");
    }
    viewToggle.querySelectorAll(".view-btn").forEach(function(btn) {
      btn.classList.toggle("active", btn.dataset.view === mode);
    });
    lazyLoad();
  }

  viewToggle.querySelectorAll(".view-btn").forEach(function(btn) {
    btn.addEventListener("click", function() {
      applyView(btn.dataset.view);
    });
  });

  /* ── Sticky offset: measure actual MkDocs header height ── */
  function updateStickyOffsets() {
    var header = document.querySelector(".md-header");
    if (!header) return;
    var hh = header.getBoundingClientRect().height;
    var toolbar = document.getElementById("cases-toolbar");
    var toolbarH = toolbar ? toolbar.getBoundingClientRect().height : 48;
    document.documentElement.style.setProperty("--cases-toolbar-top", hh + "px");
    document.documentElement.style.setProperty("--cases-filters-top", (hh + toolbarH) + "px");
  }

  /* ── Load data and init ── */
  var jsonUrl = "/amd-bc250-docs/community/cases-data.json";
  fetch(jsonUrl)
    .then(function(r) {
      if (!r.ok) throw new Error("Failed to load cases data: " + r.status);
      return r.json();
    })
    .then(function(data) {
      CASES = data;
      applyView(currentView);
      render();
      updateStickyOffsets();
      window.addEventListener("resize", updateStickyOffsets);
      setTimeout(updateStickyOffsets, 300);
    })
    .catch(function(err) {
      grid.innerHTML = '<div class="cases-loading" style="color:#c62828">Failed to load cases data. ' + escHtml(err.message) + '</div>';
      console.error(err);
    });
})();
</script>
