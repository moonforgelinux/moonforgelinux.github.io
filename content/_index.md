---
title: 'Moonforge'
description: Create Linux-based operating systems
outputs:
  - HTML
  - RSS
---
<link rel="stylesheet" href="css/home.css" />

<!-- Hero Section: Light Wisteria / Black Aurora-->
<div class="hero-wrap">
  <div class="hero">
    <div class="hero-eyebrow">Built by Igalia <i class="fa-solid fa-diamond"></i> Fully Open Source</div>
    <img class="display-dark" src="img/logos/logo-horizontal-color-dark.svg" />
    <img class="display-light" src="img/logos/logo-horizontal-color-light.svg" />
    <h1>Build Your Own<br/><span class="grad">Embedded Linux</span></h1>
    <p class="hero-sub">Moonforge is a curated collection of Yocto layers and build tooling that gives you a production-ready foundation for custom embedded and device operating systems without starting from scratch.</p>
    <div class="hero-actions">
      <a class="button btn-outline-dark" href="https://moonforgelinux.org/goals/" target="_blank"><i class="fa-solid fa-bullseye"></i>Goals</a>
      <a class="button btn-outline-dark" href="https://github.com/moonforgelinux" target="_blank"><i class="fab fa-github ms-2 "></i>Download</a>
      <a class="button btn-indigo" href="https://moonforgelinux.org/docs/tutorials/" target="_blank"><i class="fas fa-arrow-alt-circle-right ms-2"></i>Get Started</a>
    </div>
  </div>
</div>

<!-- What it is Section: Indigo -->
<div class="clarify-wrap" id="what">
  <section class="section">
    <p class="section-label">What makes Moonforge special</p>
    <h2>Key characteristics </h2>
    <div class="clarify-columns">
      <div class="clarify-grid">
        <div class="clarify-card">
          <h3>A framework of well-maintained Yocto layers</h3>
          <p>Moonforge's composable layers let you assemble your own image-based operating system.</p>
        </div>
        <div class="clarify-card">
          <h3>An opinionated, reusable starting point</h3>
          <p>Sensible defaults, CI/CD pipelines baked in, clear separation between upstream and downstream.</p>
        </div>
        <div class="clarify-card">
          <h3>Focus on what matters to you</h3>
          <p>We take care of security, updates, and features, so you can concentrate on developing, deploying, and managing your own application or appliance.</p>
        </div>
        <div class="clarify-card">
          <h3>Designed with customization in mind</h3>
          <p>Each solution is different. Build your own image using Moonforge as the foundation.</p>
        </div>
      </div>
      <div class="clarify-stats">
        <div class="clarify-stat"><h3>Immutable</h3><span>Read-only root FS</span></div>
        <div class="clarify-stat"><h3>OTA updates</h3><span>Built-in updates</span></div>
        <div class="clarify-stat"><h3>CVE & SBOM</h3><span>Compliance reports</span></div>
        <div class="clarify-stat"><h3>Yocto</h3><span>Industry standard</span></div>
      </div>
    </div>
  </section>
</div>

<!-- Principles Section: Wisteria-25 -->
<div class="principles-wrap" id="principles">
  <section class="section">
    <p class="section-label">Design goals</p>
    <h2>Three core principles</h2>
    <div class="principle-grid">
      <div class="principle-card">
        <img src="img/principle-balance.svg" alt="Balance" />
        <h3>Balance</h3>
        <p>A good balance between turn-key solutions and the flexibility to extend beyond them, supporting a wide range of products without reinventing common pieces.</p>
      </div>
      <div class="principle-card">
        <img src="img/principle-separation.svg" alt="Separation" />
        <h3>Separation</h3>
        <p>Clear separation between upstream and downstream components makes it straightforward to build derivative products with predictable processes and releases.</p>
      </div>
      <div class="principle-card">
        <img src="img/principle-best-practices.svg" alt="Best Practices" />
        <h3>Best Practices</h3>
        <p>Modern tooling baked in: BitBake for image creation, kas for declarative build config, and CI/CD pipelines that automatically produce images, OTA bundles, CVE reports, and SBOM metadata.</p>
      </div>
    </div>
  </section>
</div>

<!-- Layers Section: White / Ink -->
<div class="layers-wrap" id="layers">
  <section class="section">
    <p class="section-label">Available layers</p>
    <div class="layers-intro">
      <h2>Modular layers.<br />Compose your OS.</h2>
      <p>Every Moonforge layer provides a single, isolated feature and ships with its own kas configuration fragment — so each layer declares exactly what it needs. Layers can be reasoned about independently and combined freely into a final custom product image.</p>
    </div>
    <div class="layers-grid">
      <div class="layer-tag"><div class="layer-dot"></div><span>Immutable root filesystem</span></div>
      <div class="layer-tag"><div class="layer-dot"></div><span>QEMU x86_64 target</span></div>
      <div class="layer-tag"><div class="layer-dot"></div><span>Raspberry Pi target</span></div>
      <div class="layer-tag"><div class="layer-dot"></div><span>Containerisation with Docker</span></div>
      <div class="layer-tag"><div class="layer-dot"></div><span>Containerisation with Podman</span></div>
      <div class="layer-tag"><div class="layer-dot"></div><span>A/B updates via RAUC</span></div>
      <div class="layer-tag"><div class="layer-dot"></div><span>A/B updates via systemd</span></div>
      <div class="layer-tag"><div class="layer-dot"></div><span>Weston graphical session</span></div>
      <div class="layer-tag"><div class="layer-dot"></div><span>WPE WebKit environment</span></div>
      <div class="layer-tag"><div class="layer-dot"></div><span>+ More on GitHub<i class="fab fa-github ms-2 "></i></span></div>
    </div>
  </section>
</div>
<!-- How it works section: Indigo -->
<div class="how-wrap" id="how">
  <section class="section">
    <p class="section-label">How it works</p>
    <h2>From individual layers<br/>to a running OS image</h2>
    <div class="how-grid">
      <div class="how-step">
        <div class="step-num">Step 01</div>
        <h3>Choose your layers</h3>
        <p>Browse the available Moonforge layers for the features, architectures, and hardware targets you need: containerisation, OTA updates, a graphical session, and more.</p>
      </div>
      <div class="how-step">
        <div class="step-num">Step 02</div>
        <h3>Compose with kas</h3>
        <p>Use kas YAML configuration fragments to declare which layers are included, override configurations, and manage dependencies. Use your downstream layer for product-specific customisations.</p>
      </div>
      <div class="how-step">
        <div class="step-num">Step 03</div>
        <h3>Build inside a container</h3>
        <p>kas spins up the entire build environment inside a container, keeping your host clean and making it trivial to run on any CI/CD platform (GitLab, GitHub, Bitbucket, or your own infrastructure).</p>
      </div>
      <div class="how-step">
        <div class="step-num">Step 04</div>
        <h3>Ship your product</h3>
        <p>The pipeline automatically produces your OS image, OTA update bundles, CVE security reports, and SBOM metadata. All reproducibly, no shell-script glue required.</p>
      </div>
    </div>
  </section>
</div>

<!-- Quotes Section: Wisteria bg -->
<div class="quotes-wrap" id="press">
  <section class="section">
    <p class="section-label">From the press</p>
    <h2>What people are saying</h2>
    <div class="quotes-grid">
      <div class="quote-card">
        <span class="quote-mark">"</span>
        <p class="quote-text">
          Moonforge lets you create a new OS in minutes, selecting a series of features you care about from various available layers. Since every layer is isolated, we can reason about their dependencies and interactions, and combine them into a final, custom product.
        </p>
        <div class="quote-source">
          <div class="quote-avatar">EB</div>
          <div class="quote-meta">
            <strong>Emmanuele Bassi</strong>
            <a href="https://www.bassi.io/articles/2026/03/17/lets-talk-about-moonforge/" target="_blank">halting problem · March 2026</a>
          </div>
        </div>
      </div>
      <div class="quote-card">
        <span class="quote-mark">"</span>
        <p class="quote-text">
          Moonforge focuses on extensibility, flexibility, and long-term maintainability, enabling developers and system integrators to create custom operating system images while relying on well-established industry tooling and best practices.
        </p>
        <div class="quote-source">
          <div class="quote-avatar">IG</div>
          <div class="quote-meta">
            <strong>Igalia</strong>
            <a href="https://www.igalia.com/2026/03/09/Introducing-Moonforge-A-Yocto-Based-Linux-OS.html" target="_blank">Announcement · March 2026</a>
          </div>
        </div>
      </div>
    </div>
  </section>
</div>
<div class="cta-wrap">
  <section class="section">
    <div class="cta-inner">
      <h2>Start building your OS <span>today.</span></h2>
      <div class="cta-buttons">
        <a class="button btn-indigo" href="https://moonforgelinux.org/docs/tutorials/" target="_blank">Read the Tutorials →</a>
        <a class="button btn-outline-dark" href="https://github.com/moonforgelinux/meta-derivative/" target="_blank">derivative example →</a>
      </div>
    </div>
  </section>
</div>

<!-- Footer: ink -->
<footer>
  <div class="footer-inner">
    <div class="footer-brand">
      <img src="img/logos/logo-horizontal-color-dark.svg" alt="Moonforge" />
      An open-source project by Igalia S.L.</a>
    </div>
    <ul class="footer-links">
      <li><a href="https://moonforgelinux.org/docs/" target="_blank">Docs</a></li>
      <li><a href="https://moonforgelinux.org/docs/tutorials/" target="_blank">Tutorials</a></li>
      <li><a href="https://github.com/moonforgelinux" target="_blank">GitHub</a></li>
      <li><a href="https://www.igalia.com/" target="_blank">Igalia</a></li>
    </ul>
  </div>
</footer>
