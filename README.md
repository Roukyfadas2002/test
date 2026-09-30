<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Order Platform — Architecture hexagonale</title>
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
  <link href="https://fonts.googleapis.com/css2?family=DM+Sans:ital,opsz,wght@0,9..40,400;0,9..40,500;0,9..40,600;0,9..40,700;1,9..40,400&family=Fraunces:opsz,wght@9..144,500;9..144,600;9..144,700&display=swap" rel="stylesheet" />
  <style>
    :root {
      --bg: #f3f6f4;
      --bg-soft: #e7eee9;
      --surface: #ffffff;
      --ink: #14201a;
      --muted: #5a6b62;
      --line: #c9d5ce;
      --accent: #0f6b4c;
      --accent-soft: #d8efe4;
      --inbound: #1a5f7a;
      --inbound-soft: #d7ebf3;
      --outbound: #8a4b12;
      --outbound-soft: #f3e4d4;
      --domain: #0f6b4c;
      --domain-soft: #d8efe4;
      --warn: #9a3412;
      --ok: #166534;
      --shadow: 0 1px 0 rgba(20, 32, 26, 0.04), 0 12px 32px rgba(20, 32, 26, 0.06);
      --radius: 18px;
      --mono: ui-monospace, "Cascadia Code", "SF Mono", Menlo, monospace;
    }

    * { box-sizing: border-box; }
    html { scroll-behavior: smooth; }
    body {
      margin: 0;
      color: var(--ink);
      background:
        radial-gradient(1200px 600px at 10% -10%, #d9ebe2 0%, transparent 55%),
        radial-gradient(900px 500px at 100% 0%, #d6e6ef 0%, transparent 50%),
        var(--bg);
      font-family: "DM Sans", system-ui, sans-serif;
      line-height: 1.55;
      font-size: 16px;
    }

    .wrap {
      max-width: 1120px;
      margin: 0 auto;
      padding: 40px 24px 96px;
    }

    /* Hero */
    header.hero {
      display: grid;
      gap: 28px;
      margin-bottom: 48px;
      padding: 36px 0 8px;
    }
    .eyebrow {
      display: inline-flex;
      align-items: center;
      gap: 8px;
      color: var(--accent);
      font-weight: 600;
      font-size: 0.85rem;
      letter-spacing: 0.04em;
      text-transform: uppercase;
    }
    .eyebrow::before {
      content: "";
      width: 10px;
      height: 10px;
      border-radius: 50%;
      background: var(--accent);
    }
    h1 {
      font-family: Fraunces, Georgia, serif;
      font-weight: 600;
      font-size: clamp(2.2rem, 4vw, 3.4rem);
      line-height: 1.1;
      margin: 0;
      letter-spacing: -0.02em;
      max-width: 16ch;
    }
    .lede {
      font-size: 1.15rem;
      color: var(--muted);
      max-width: 48ch;
      margin: 0;
    }
    .meta {
      display: flex;
      flex-wrap: wrap;
      gap: 10px;
    }
    .chip {
      background: var(--surface);
      border: 1px solid var(--line);
      border-radius: 999px;
      padding: 8px 14px;
      font-size: 0.88rem;
      color: var(--muted);
    }
    .chip strong { color: var(--ink); font-weight: 600; }

    /* Nav */
    nav.toc {
      position: sticky;
      top: 0;
      z-index: 20;
      backdrop-filter: blur(10px);
      background: rgba(243, 246, 244, 0.88);
      border: 1px solid var(--line);
      border-radius: 14px;
      padding: 12px 14px;
      margin-bottom: 40px;
      display: flex;
      flex-wrap: wrap;
      gap: 8px;
    }
    nav.toc a {
      text-decoration: none;
      color: var(--muted);
      font-size: 0.88rem;
      font-weight: 500;
      padding: 7px 12px;
      border-radius: 999px;
    }
    nav.toc a:hover {
      background: var(--accent-soft);
      color: var(--accent);
    }

    section {
      margin-bottom: 56px;
    }
    h2 {
      font-family: Fraunces, Georgia, serif;
      font-size: 1.85rem;
      font-weight: 600;
      margin: 0 0 10px;
      letter-spacing: -0.02em;
    }
    h3 {
      font-family: Fraunces, Georgia, serif;
      font-size: 1.25rem;
      font-weight: 600;
      margin: 28px 0 12px;
    }
    .section-intro {
      color: var(--muted);
      max-width: 62ch;
      margin: 0 0 24px;
    }

    .card {
      background: var(--surface);
      border: 1px solid var(--line);
      border-radius: var(--radius);
      box-shadow: var(--shadow);
      padding: 24px;
    }
    .grid-2 {
      display: grid;
      grid-template-columns: repeat(2, minmax(0, 1fr));
      gap: 16px;
    }
    .grid-3 {
      display: grid;
      grid-template-columns: repeat(3, minmax(0, 1fr));
      gap: 16px;
    }
    @media (max-width: 860px) {
      .grid-2, .grid-3 { grid-template-columns: 1fr; }
      nav.toc { position: static; }
    }

    /* Hexagon visual */
    .hex-stage {
      display: grid;
      place-items: center;
      padding: 20px 8px 8px;
      overflow-x: auto;
    }
    .hex {
      position: relative;
      width: min(720px, 100%);
      aspect-ratio: 1.15;
    }
    .hex-ring {
      position: absolute;
      inset: 8%;
      border: 2px dashed var(--line);
      border-radius: 50%;
    }
    .hex-core {
      position: absolute;
      left: 50%;
      top: 50%;
      transform: translate(-50%, -50%);
      width: 34%;
      aspect-ratio: 1;
      background: var(--domain-soft);
      border: 2px solid var(--domain);
      clip-path: polygon(50% 0%, 100% 25%, 100% 75%, 50% 100%, 0% 75%, 0% 25%);
      display: grid;
      place-items: center;
      text-align: center;
      padding: 12px;
    }
    .hex-core strong {
      display: block;
      font-family: Fraunces, serif;
      color: var(--domain);
      font-size: 1.05rem;
    }
    .hex-core span {
      display: block;
      font-size: 0.78rem;
      color: var(--muted);
      margin-top: 4px;
    }
    .hex-node {
      position: absolute;
      width: 150px;
      padding: 12px 14px;
      border-radius: 14px;
      border: 1px solid var(--line);
      background: var(--surface);
      box-shadow: var(--shadow);
      font-size: 0.82rem;
    }
    .hex-node b {
      display: block;
      font-size: 0.92rem;
      margin-bottom: 2px;
    }
    .hex-node.inbound { border-color: #9dc4d5; background: var(--inbound-soft); }
    .hex-node.inbound b { color: var(--inbound); }
    .hex-node.outbound { border-color: #e0c3a3; background: var(--outbound-soft); }
    .hex-node.outbound b { color: var(--outbound); }
    .hex-node.app { border-color: #b7cfc3; background: #eef6f1; }
    .n1 { left: 50%; top: 2%; transform: translateX(-50%); }
    .n2 { right: 0; top: 22%; }
    .n3 { right: 0; bottom: 22%; }
    .n4 { left: 50%; bottom: 2%; transform: translateX(-50%); }
    .n5 { left: 0; bottom: 22%; }
    .n6 { left: 0; top: 22%; }

    /* Flow */
    .flow {
      display: flex;
      flex-direction: column;
      gap: 0;
    }
    .flow-step {
      display: grid;
      grid-template-columns: 56px 1fr;
      gap: 14px;
      align-items: start;
    }
    .flow-rail {
      display: flex;
      flex-direction: column;
      align-items: center;
      min-height: 100%;
    }
    .flow-dot {
      width: 18px;
      height: 18px;
      border-radius: 50%;
      background: var(--accent);
      border: 3px solid var(--accent-soft);
      flex: 0 0 auto;
      margin-top: 14px;
    }
    .flow-line {
      width: 2px;
      flex: 1;
      background: var(--line);
      min-height: 28px;
    }
    .flow-body {
      background: var(--surface);
      border: 1px solid var(--line);
      border-radius: 14px;
      padding: 14px 16px;
      margin-bottom: 10px;
    }
    .flow-body .jar {
      font-family: var(--mono);
      font-size: 0.75rem;
      color: var(--accent);
      font-weight: 600;
    }
    .flow-body .title {
      font-weight: 650;
      margin: 2px 0 4px;
    }
    .flow-body p {
      margin: 0;
      color: var(--muted);
      font-size: 0.92rem;
    }

    /* Compare */
    .compare {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 14px;
    }
    @media (max-width: 700px) { .compare { grid-template-columns: 1fr; } }
    .bad, .good {
      border-radius: 14px;
      padding: 16px;
      border: 1px solid var(--line);
    }
    .bad { background: #fff1ee; border-color: #f0c7bb; }
    .good { background: var(--accent-soft); border-color: #a8d4c0; }
    .bad h4, .good h4 {
      margin: 0 0 10px;
      font-size: 0.95rem;
      letter-spacing: 0.04em;
      text-transform: uppercase;
    }
    .bad h4 { color: var(--warn); }
    .good h4 { color: var(--ok); }
    pre.tree {
      margin: 0;
      font-family: var(--mono);
      font-size: 0.8rem;
      line-height: 1.45;
      white-space: pre;
      overflow-x: auto;
    }

    /* Tables */
    .table-wrap {
      overflow-x: auto;
      border: 1px solid var(--line);
      border-radius: 14px;
      background: var(--surface);
      box-shadow: var(--shadow);
    }
    table {
      width: 100%;
      border-collapse: collapse;
      font-size: 0.9rem;
    }
    th, td {
      text-align: left;
      padding: 12px 14px;
      border-bottom: 1px solid var(--line);
      vertical-align: top;
    }
    th {
      background: var(--bg-soft);
      font-weight: 600;
      font-size: 0.8rem;
      text-transform: uppercase;
      letter-spacing: 0.04em;
      color: var(--muted);
    }
    tr:last-child td { border-bottom: none; }
    code, .mono {
      font-family: var(--mono);
      font-size: 0.84em;
      background: var(--bg-soft);
      padding: 1px 6px;
      border-radius: 6px;
    }
    .tag {
      display: inline-block;
      font-size: 0.75rem;
      font-weight: 650;
      padding: 3px 8px;
      border-radius: 999px;
    }
    .tag.in { background: var(--inbound-soft); color: var(--inbound); }
    .tag.out { background: var(--outbound-soft); color: var(--outbound); }
    .tag.dom { background: var(--domain-soft); color: var(--domain); }

    /* Maven graph */
    .maven {
      display: grid;
      gap: 10px;
      font-family: var(--mono);
      font-size: 0.82rem;
    }
    .maven .row {
      display: grid;
      grid-template-columns: 220px 28px 1fr;
      gap: 8px;
      align-items: center;
    }
    @media (max-width: 700px) {
      .maven .row { grid-template-columns: 1fr; }
    }
    .maven .from, .maven .to {
      background: var(--bg-soft);
      border: 1px solid var(--line);
      border-radius: 10px;
      padding: 10px 12px;
    }
    .maven .arrow {
      text-align: center;
      color: var(--accent);
      font-weight: 700;
    }

    /* Layers stack */
    .stack {
      display: grid;
      gap: 8px;
    }
    .layer {
      border-radius: 14px;
      padding: 14px 16px;
      border: 1px solid var(--line);
      display: grid;
      grid-template-columns: 140px 1fr;
      gap: 12px;
      align-items: center;
    }
    @media (max-width: 700px) {
      .layer { grid-template-columns: 1fr; }
    }
    .layer .name {
      font-weight: 700;
      font-size: 0.95rem;
    }
    .layer .desc { color: var(--muted); font-size: 0.9rem; }
    .l-rest { background: var(--inbound-soft); }
    .l-api { background: #e8f2f7; }
    .l-core { background: #eef6f1; }
    .l-domain { background: var(--domain-soft); border-width: 2px; border-color: var(--domain); }
    .l-spi { background: #f7efe6; }
    .l-infra { background: var(--outbound-soft); }

    /* Mapping strip */
    .map-strip {
      display: grid;
      grid-template-columns: repeat(7, minmax(0, 1fr));
      gap: 6px;
      align-items: center;
      text-align: center;
      font-size: 0.82rem;
    }
    @media (max-width: 900px) {
      .map-strip { grid-template-columns: 1fr; }
    }
    .map-box {
      background: var(--surface);
      border: 1px solid var(--line);
      border-radius: 12px;
      padding: 12px 8px;
      min-height: 74px;
      display: grid;
      place-items: center;
    }
    .map-box strong { display: block; }
    .map-arrow {
      color: var(--accent);
      font-weight: 700;
      font-size: 1.2rem;
    }

    .callout {
      background: var(--surface);
      border-left: 4px solid var(--accent);
      border-radius: 0 14px 14px 0;
      padding: 14px 16px;
      box-shadow: var(--shadow);
      margin: 18px 0;
    }
    .callout strong { color: var(--accent); }

    footer {
      margin-top: 64px;
      padding-top: 24px;
      border-top: 1px solid var(--line);
      color: var(--muted);
      font-size: 0.9rem;
    }
    .legend {
      display: flex;
      flex-wrap: wrap;
      gap: 12px;
      margin: 12px 0 20px;
      font-size: 0.85rem;
    }
    .legend span {
      display: inline-flex;
      align-items: center;
      gap: 8px;
    }
    .swatch {
      width: 12px;
      height: 12px;
      border-radius: 3px;
    }

    /* Connection diagrams */
    .wire-legend {
      display: flex;
      flex-wrap: wrap;
      gap: 14px;
      margin: 0 0 18px;
      font-size: 0.85rem;
      color: var(--muted);
    }
    .wire-legend .l {
      display: inline-flex;
      align-items: center;
      gap: 8px;
    }
    .wire-legend i {
      width: 28px;
      height: 3px;
      border-radius: 2px;
      display: inline-block;
    }
    .wire-legend .uses { background: var(--inbound); }
    .wire-legend .impl { background: var(--accent); border-top: 2px dashed transparent; background-image: repeating-linear-gradient(90deg, var(--accent) 0 6px, transparent 6px 10px); height: 0; border-top: 3px dashed var(--accent); width: 28px; }
    .wire-legend .calls { background: var(--outbound); }

    .iface {
      border: 2px dashed var(--inbound);
      background: var(--inbound-soft);
      border-radius: 12px;
      padding: 10px 12px;
      font-family: var(--mono);
      font-size: 0.8rem;
    }
    .iface .kind { display:block; font-family:"DM Sans",sans-serif; font-size:0.7rem; font-weight:700; letter-spacing:0.04em; text-transform:uppercase; color:var(--inbound); margin-bottom:4px; }
    .impl-box {
      border: 2px solid var(--accent);
      background: var(--domain-soft);
      border-radius: 12px;
      padding: 10px 12px;
      font-family: var(--mono);
      font-size: 0.8rem;
    }
    .impl-box .kind { display:block; font-family:"DM Sans",sans-serif; font-size:0.7rem; font-weight:700; letter-spacing:0.04em; text-transform:uppercase; color:var(--accent); margin-bottom:4px; }
    .rest-box {
      border: 2px solid var(--inbound);
      background: #eef5f9;
      border-radius: 12px;
      padding: 10px 12px;
      font-family: var(--mono);
      font-size: 0.8rem;
    }
    .rest-box .kind { display:block; font-family:"DM Sans",sans-serif; font-size:0.7rem; font-weight:700; letter-spacing:0.04em; text-transform:uppercase; color:var(--inbound); margin-bottom:4px; }
    .out-box {
      border: 2px solid var(--outbound);
      background: var(--outbound-soft);
      border-radius: 12px;
      padding: 10px 12px;
      font-family: var(--mono);
      font-size: 0.8rem;
    }
    .out-box .kind { display:block; font-family:"DM Sans",sans-serif; font-size:0.7rem; font-weight:700; letter-spacing:0.04em; text-transform:uppercase; color:var(--outbound); margin-bottom:4px; }
    .dom-box {
      border: 2px solid var(--domain);
      background: #e9f6ef;
      border-radius: 12px;
      padding: 10px 12px;
      font-family: var(--mono);
      font-size: 0.8rem;
    }
    .dom-box .kind { display:block; font-family:"DM Sans",sans-serif; font-size:0.7rem; font-weight:700; letter-spacing:0.04em; text-transform:uppercase; color:var(--domain); margin-bottom:4px; }

    .conn-grid {
      display: grid;
      gap: 10px;
    }
    .conn-row {
      display: grid;
      grid-template-columns: 1.1fr 70px 1.1fr 70px 1.1fr;
      gap: 8px;
      align-items: center;
    }
    @media (max-width: 900px) {
      .conn-row {
        grid-template-columns: 1fr;
      }
      .conn-arrow { transform: rotate(90deg); justify-self: center; }
    }
    .conn-arrow {
      text-align: center;
      font-size: 0.72rem;
      font-weight: 700;
      line-height: 1.25;
      color: var(--muted);
    }
    .conn-arrow .sym {
      display: block;
      font-size: 1.35rem;
      color: var(--accent);
      line-height: 1;
    }
    .conn-arrow.uses .sym { color: var(--inbound); }
    .conn-arrow.impls .sym { color: var(--accent); }
    .conn-arrow.calls .sym { color: var(--outbound); }

    .seq {
      display: grid;
      gap: 0;
      font-size: 0.88rem;
    }
    .seq-item {
      display: grid;
      grid-template-columns: 42px 1fr;
      gap: 12px;
    }
    .seq-num {
      width: 32px;
      height: 32px;
      border-radius: 50%;
      background: var(--accent);
      color: white;
      display: grid;
      place-items: center;
      font-weight: 700;
      font-size: 0.85rem;
      margin-top: 10px;
    }
    .seq-card {
      background: var(--surface);
      border: 1px solid var(--line);
      border-radius: 14px;
      padding: 12px 14px;
      margin-bottom: 10px;
    }
    .seq-card .who {
      font-family: var(--mono);
      font-size: 0.75rem;
      font-weight: 650;
      color: var(--accent);
    }
    .seq-card .act {
      font-weight: 650;
      margin: 2px 0 4px;
    }
    .seq-card .why {
      color: var(--muted);
      font-size: 0.88rem;
      margin: 0;
    }
    .seq-card code {
      font-size: 0.8em;
    }
    .mini-jar {
      display: inline-block;
      font-size: 0.7rem;
      font-weight: 700;
      padding: 2px 7px;
      border-radius: 999px;
      background: var(--bg-soft);
      color: var(--muted);
      margin-left: 6px;
      vertical-align: middle;
    }

    .ctor {
      overflow-x: auto;
    }
    .ctor pre {
      margin: 0;
      font-family: var(--mono);
      font-size: 0.78rem;
      line-height: 1.5;
      background: #122018;
      color: #d7ebe2;
      border-radius: 14px;
      padding: 18px 20px;
    }
    .ctor .c-iface { color: #7dd3fc; }
    .ctor .c-impl { color: #86efac; }
    .ctor .c-comment { color: #8fa398; }
    .ctor .c-kw { color: #fbbf24; }

    details.wire-detail {
      background: var(--surface);
      border: 1px solid var(--line);
      border-radius: 14px;
      padding: 0;
      margin: 12px 0;
      box-shadow: var(--shadow);
      overflow: hidden;
    }
    details.wire-detail summary {
      cursor: pointer;
      padding: 14px 16px;
      font-weight: 650;
      list-style: none;
      display: flex;
      justify-content: space-between;
      align-items: center;
      gap: 12px;
    }
    details.wire-detail summary::-webkit-details-marker { display: none; }
    details.wire-detail summary::after {
      content: "+";
      font-size: 1.2rem;
      color: var(--accent);
      font-weight: 700;
    }
    details.wire-detail[open] summary::after { content: "−"; }
    details.wire-detail .detail-body {
      padding: 0 16px 18px;
      border-top: 1px solid var(--line);
    }

    .matrix {
      display: grid;
      grid-template-columns: 180px repeat(6, minmax(70px, 1fr));
      gap: 4px;
      font-size: 0.72rem;
      overflow-x: auto;
    }
    .matrix .mh, .matrix .ml {
      background: var(--bg-soft);
      border-radius: 8px;
      padding: 8px 6px;
      text-align: center;
      font-weight: 650;
      color: var(--muted);
    }
    .matrix .ml { text-align: left; font-family: var(--mono); font-size: 0.7rem; color: var(--ink); }
    .matrix .cell {
      border-radius: 8px;
      padding: 8px 4px;
      text-align: center;
      background: #f8faf9;
      border: 1px solid var(--line);
    }
    .matrix .cell.yes {
      background: var(--accent-soft);
      border-color: #a8d4c0;
      color: var(--ok);
      font-weight: 700;
    }

    .tabs {
      display: flex;
      flex-wrap: wrap;
      gap: 8px;
      margin-bottom: 14px;
    }
    .tabs button {
      border: 1px solid var(--line);
      background: var(--surface);
      border-radius: 999px;
      padding: 8px 14px;
      font: inherit;
      font-size: 0.88rem;
      font-weight: 550;
      color: var(--muted);
      cursor: pointer;
    }
    .tabs button.active {
      background: var(--accent);
      border-color: var(--accent);
      color: white;
    }
    .tab-panel { display: none; }
    .tab-panel.active { display: block; }
    .copy-box {
      position: relative;
      background: #122018;
      color: #d7ebe2;
      border-radius: 14px;
      padding: 16px 18px;
      font-family: var(--mono);
      font-size: 0.78rem;
      line-height: 1.5;
      overflow-x: auto;
      white-space: pre-wrap;
      word-break: break-word;
      margin: 0;
    }
    .copy-wrap {
      position: relative;
    }
    .copy-btn {
      position: absolute;
      top: 10px;
      right: 10px;
      z-index: 2;
      border: 1px solid #2a4a3a;
      background: #1a3026;
      color: #a7f3d0;
      border-radius: 999px;
      padding: 6px 12px;
      font: inherit;
      font-size: 0.78rem;
      font-weight: 650;
      cursor: pointer;
    }
    .copy-btn:hover { background: #234836; }
    .skill-block h4 {
      margin: 20px 0 8px;
      font-family: Fraunces, Georgia, serif;
      font-size: 1.05rem;
    }
    .checklist {
      list-style: none;
      padding: 0;
      margin: 0;
      display: grid;
      gap: 8px;
    }
    .checklist li {
      background: var(--surface);
      border: 1px solid var(--line);
      border-radius: 12px;
      padding: 10px 14px;
      display: grid;
      grid-template-columns: 28px 1fr;
      gap: 10px;
      align-items: start;
      font-size: 0.92rem;
    }
    .checklist .n {
      width: 22px;
      height: 22px;
      border-radius: 50%;
      background: var(--accent-soft);
      color: var(--accent);
      display: grid;
      place-items: center;
      font-size: 0.75rem;
      font-weight: 700;
      margin-top: 1px;
    }
  </style>
</head>
<body>
  <div class="wrap">
    <header class="hero">
      <div class="eyebrow">Projet de référence</div>
      <h1>Architecture hexagonale</h1>
      <p class="lede">
        Schéma pédagogique du projet <strong>Order Platform</strong> :
        Ports &amp; Adapters, inversion de dépendances, multi-modules Maven,
        Java&nbsp;25 et Quarkus&nbsp;3.33 LTS.
      </p>
      <div class="meta">
        <span class="chip"><strong>Java</strong> 25</span>
        <span class="chip"><strong>Quarkus</strong> 3.33.3 LTS</span>
        <span class="chip"><strong>8</strong> modules JAR</span>
        <span class="chip"><strong>Métier</strong> commandes · stock · paiement</span>
      </div>
    </header>

    <nav class="toc" aria-label="Sommaire">
      <a href="#hexagone">Hexagone</a>
      <a href="#modules">Modules</a>
      <a href="#maven">Maven</a>
      <a href="#connexions">Connexions</a>
      <a href="#pay-detail">Pay détaillé</a>
      <a href="#tous-usecases">Tous Use Cases</a>
      <a href="#ports">Ports</a>
      <a href="#dip">Inversion</a>
      <a href="#mapping">4 modèles</a>
      <a href="#flux">Flux paiement</a>
      <a href="#cdi">CDI Quarkus</a>
      <a href="#tests">Tests</a>
      <a href="#evoluer">Évoluer</a>
      <a href="#skill">Skill &amp; Prompt</a>
    </nav>

    <!-- HEXAGONE -->
    <section id="hexagone">
      <h2>L’idée en une image</h2>
      <p class="section-intro">
        Le <strong>domaine</strong> est au centre. Autour : les ports (contrats).
        Sur le bord : les adaptateurs (technologies). Les flèches pointent
        <em>toujours vers l’intérieur</em>.
      </p>

      <div class="legend">
        <span><i class="swatch" style="background:#1a5f7a"></i> Inbound (entrée)</span>
        <span><i class="swatch" style="background:#0f6b4c"></i> Domaine / application</span>
        <span><i class="swatch" style="background:#8a4b12"></i> Outbound (sortie)</span>
      </div>

      <div class="card hex-stage">
        <div class="hex" aria-hidden="false">
          <div class="hex-ring"></div>
          <div class="hex-core">
            <div>
              <strong>domain</strong>
              <span>règles métier<br/>sans framework</span>
            </div>
          </div>

          <div class="hex-node inbound n1">
            <b>adapter-rest</b>
            HTTP · Jackson · OpenAPI
          </div>
          <div class="hex-node inbound n2">
            <b>application-api</b>
            Inbound Ports<br/>Use Cases
          </div>
          <div class="hex-node app n3">
            <b>application-core</b>
            Orchestration<br/>des Use Cases
          </div>
          <div class="hex-node outbound n4">
            <b>infra-payment</b>
            FakePaymentGateway
          </div>
          <div class="hex-node outbound n5">
            <b>infra-persistence</b>
            JPA · PostgreSQL · Flyway
          </div>
          <div class="hex-node outbound n6">
            <b>application-spi</b>
            Outbound Ports<br/>Repos / gateways
          </div>
        </div>
      </div>

      <div class="callout">
        <strong>À retenir :</strong> REST, PostgreSQL et le faux PSP sont des
        <em>détails</em>. On peut les remplacer sans toucher au JAR <code>domain</code>.
      </div>
    </section>

    <!-- MODULES -->
    <section id="modules">
      <h2>Couches &amp; responsabilités</h2>
      <p class="section-intro">
        Du monde extérieur vers le coeur métier. Chaque couche a un job unique.
      </p>

      <div class="stack">
        <div class="layer l-rest">
          <div class="name">adapter-rest</div>
          <div class="desc">Endpoints HTTP. Traduit JSON ↔ Commands / Views. <strong>Zéro règle métier.</strong></div>
        </div>
        <div class="layer l-api">
          <div class="name">application-api</div>
          <div class="desc">Interfaces des Use Cases (<code>PayOrderUseCase</code>…) + Commands + Views.</div>
        </div>
        <div class="layer l-core">
          <div class="name">application-core</div>
          <div class="desc">Implémentations. Orchestre domaine + ports sortants. Ne connaît ni SQL ni HTTP.</div>
        </div>
        <div class="layer l-domain">
          <div class="name">domain</div>
          <div class="desc">Entities, Value Objects, DiscountPolicy, transitions d’état, événements.</div>
        </div>
        <div class="layer l-spi">
          <div class="name">application-spi</div>
          <div class="desc">Ports sortants : <code>OrderRepository</code>, <code>PaymentGateway</code>, <code>ClockProvider</code>…</div>
        </div>
        <div class="layer l-infra">
          <div class="name">infrastructure-*</div>
          <div class="desc">Réalisations techniques : JPA/PostgreSQL et FakePaymentGateway.</div>
        </div>
      </div>

      <h3>Pourquoi <code>application-spi</code> en plus de <code>application-api</code> ?</h3>
      <div class="grid-2">
        <div class="card">
          <span class="tag in">Inbound</span>
          <h3 style="margin-top:10px">application-api</h3>
          <p style="color:var(--muted);margin:0">
            Ce que le <strong>monde appelle</strong>.<br/>
            REST → <code>PayOrderUseCase</code>
          </p>
        </div>
        <div class="card">
          <span class="tag out">Outbound</span>
          <h3 style="margin-top:10px">application-spi</h3>
          <p style="color:var(--muted);margin:0">
            Ce que l’<strong>application appelle</strong>.<br/>
            Use Case → <code>PaymentGateway</code>
          </p>
        </div>
      </div>
    </section>

    <!-- MAVEN -->
    <section id="maven">
      <h2>Graphe Maven</h2>
      <p class="section-intro">
        Une dépendance Maven = une flèche autorisée. Si A ne dépend pas de B dans le POM,
        le code de A <strong>ne peut pas</strong> importer B. C’est la police de l’architecture.
      </p>

      <div class="card maven">
        <div class="row">
          <div class="from">quarkus-app</div>
          <div class="arrow">→</div>
          <div class="to">tous les modules (composition root)</div>
        </div>
        <div class="row">
          <div class="from">adapter-rest</div>
          <div class="arrow">→</div>
          <div class="to">application-api → domain</div>
        </div>
        <div class="row">
          <div class="from">application-core</div>
          <div class="arrow">→</div>
          <div class="to">application-api + application-spi + domain</div>
        </div>
        <div class="row">
          <div class="from">infrastructure-persistence</div>
          <div class="arrow">→</div>
          <div class="to">application-spi → domain</div>
        </div>
        <div class="row">
          <div class="from">infrastructure-payment</div>
          <div class="arrow">→</div>
          <div class="to">application-spi → domain</div>
        </div>
      </div>

      <div class="callout">
        <strong>Interdit volontairement :</strong>
        <code>adapter-rest</code> ne dépend <em>pas</em> de
        <code>infrastructure-persistence</code>.
        Le controller ne doit jamais parler SQL.
      </div>
    </section>

    <!-- CONNEXIONS -->
    <section id="connexions">
      <h2>Schéma des connexions : interface ↔ implémentation</h2>
      <p class="section-intro">
        Voici la règle exacte à retenir. Une <strong>interface</strong> (port) est un contrat.
        Une classe <strong>utilise</strong> le contrat (dépendance). Une autre classe
        <strong>implémente</strong> le contrat (réalisation). Quarkus relie les deux au démarrage.
      </p>

      <div class="wire-legend">
        <span class="l"><i class="uses"></i> utilise (injecte / appelle)</span>
        <span class="l"><i class="impl"></i> implémente (<code>implements</code>)</span>
        <span class="l"><i class="calls"></i> appelle une méthode technique</span>
      </div>

      <h3>1. Lien inbound (entrée HTTP → Use Case)</h3>
      <div class="card conn-grid">
        <div class="conn-row">
          <div class="rest-box">
            <span class="kind">Classe REST · adapter-rest</span>
            <strong>OrderResource</strong><br/>
            champ : <code>PayOrderUseCase</code>
          </div>
          <div class="conn-arrow uses"><span class="sym">⟶</span>utilise<br/>(injecté)</div>
          <div class="iface">
            <span class="kind">Interface inbound · application-api</span>
            <strong>PayOrderUseCase</strong><br/>
            <code>OrderView execute(PayOrderCommand)</code>
          </div>
          <div class="conn-arrow impls"><span class="sym">⟸</span>implémenté<br/>par</div>
          <div class="impl-box">
            <span class="kind">Classe service · application-core</span>
            <strong>PayOrderService</strong><br/>
            <code>implements PayOrderUseCase</code>
          </div>
        </div>
      </div>
      <div class="callout">
        <strong>Pourquoi ?</strong> <code>OrderResource</code> ne connaît <em>pas</em>
        <code>PayOrderService</code>. Il ne connaît que l’interface. Demain tu peux changer
        l’implémentation : le controller REST reste identique.
      </div>

      <h3>2. Lien outbound (Use Case → infrastructure)</h3>
      <div class="card conn-grid">
        <div class="conn-row">
          <div class="impl-box">
            <span class="kind">Service · application-core</span>
            <strong>PayOrderService</strong><br/>
            champ : <code>PaymentGateway</code>
          </div>
          <div class="conn-arrow uses"><span class="sym">⟶</span>utilise<br/>(injecté)</div>
          <div class="iface" style="border-color:var(--outbound);background:var(--outbound-soft)">
            <span class="kind" style="color:var(--outbound)">Interface outbound · application-spi</span>
            <strong>PaymentGateway</strong><br/>
            <code>PaymentResult charge(OrderId, Money)</code>
          </div>
          <div class="conn-arrow impls"><span class="sym">⟸</span>implémenté<br/>par</div>
          <div class="out-box">
            <span class="kind">Adaptateur · infrastructure-payment</span>
            <strong>FakePaymentGateway</strong><br/>
            <code>implements PaymentGateway</code>
          </div>
        </div>
      </div>
      <div class="callout">
        <strong>Même principe, autre sens :</strong> le service applicatif exige un
        « moyen de payer ». L’infra fournit Fake (ou Stripe). Le métier ne change pas.
      </div>

      <h3>3. Injection constructeur vue depuis le code</h3>
      <div class="card ctor">
        <pre><span class="c-comment">// OrderResource (adapter-rest) — dépend d'INTERFACES inbound</span>
<span class="c-kw">public</span> OrderResource(
    <span class="c-iface">CreateOrderUseCase</span> createOrderUseCase,      <span class="c-comment">// ← CreateOrderService</span>
    <span class="c-iface">AddProductToOrderUseCase</span> add…,             <span class="c-comment">// ← AddProductToOrderService</span>
    <span class="c-iface">ValidateOrderUseCase</span> validate…,            <span class="c-comment">// ← ValidateOrderService</span>
    <span class="c-iface">PayOrderUseCase</span> payOrderUseCase,           <span class="c-comment">// ← PayOrderService</span>
    <span class="c-iface">CancelOrderUseCase</span> cancel…,               <span class="c-comment">// ← CancelOrderService</span>
    <span class="c-iface">GetOrderUseCase</span> getOrderUseCase            <span class="c-comment">// ← GetOrderService</span>
) { … }

<span class="c-comment">// PayOrderService (application-core) — dépend d'INTERFACES outbound</span>
<span class="c-kw">public</span> PayOrderService(
    <span class="c-iface">OrderRepository</span> orderRepository,           <span class="c-comment">// ← JpaOrderRepository</span>
    <span class="c-iface">PaymentRepository</span> paymentRepository,       <span class="c-comment">// ← JpaPaymentRepository</span>
    <span class="c-iface">PaymentGateway</span> paymentGateway,             <span class="c-comment">// ← FakePaymentGateway</span>
    <span class="c-iface">StockRepository</span> stockRepository,           <span class="c-comment">// ← JpaStockRepository</span>
    <span class="c-iface">EventPublisher</span> eventPublisher,             <span class="c-comment">// ← LoggingEventPublisher</span>
    <span class="c-iface">IdGenerator</span> idGenerator,                   <span class="c-comment">// ← UuidIdGenerator</span>
    <span class="c-iface">ClockProvider</span> clockProvider                <span class="c-comment">// ← SystemClockProvider</span>
) { … }

<span class="c-comment">// Quarkus Arc résout chaque paramètre d'interface vers LA classe @ApplicationScoped
// qui implements cette interface (une seule candidate = injection automatique)</span></pre>
      </div>
    </section>

    <!-- PAY DETAIL -->
    <section id="pay-detail">
      <h2>Zoom chirurgical : <code>POST /orders/{id}/pay</code></h2>
      <p class="section-intro">
        Chaque flèche = un appel de méthode réel dans le code. À chaque étape :
        <em>quelle classe</em>, <em>dans quel JAR</em>, <em>via quelle interface</em>.
      </p>

      <div class="seq">
        <div class="seq-item">
          <div class="seq-num">1</div>
          <div class="seq-card">
            <div class="who">Client HTTP</div>
            <div class="act">POST /orders/{id}/pay</div>
            <p class="why">Corps vide. Quarkus route vers <code>OrderResource.pay(UUID id)</code>.</p>
          </div>
        </div>
        <div class="seq-item">
          <div class="seq-num">2</div>
          <div class="seq-card">
            <div class="who">OrderResource.pay() <span class="mini-jar">adapter-rest</span></div>
            <div class="act">new PayOrderCommand(id) → payOrderUseCase.execute(command)</div>
            <p class="why">
              Type du champ injecté : <code>PayOrderUseCase</code> (interface).<br/>
              Objet réel au runtime : <code>PayOrderService</code>.<br/>
              Mapping : UUID path → Command applicative (pas le domaine encore).
            </p>
          </div>
        </div>
        <div class="seq-item">
          <div class="seq-num">3</div>
          <div class="seq-card">
            <div class="who">PayOrderService.execute() <span class="mini-jar">application-core</span></div>
            <div class="act">OrderId.of(command.orderId())</div>
            <p class="why">Passe du type technique UUID au Value Object métier <code>OrderId</code> (<span class="mini-jar">domain</span>).</p>
          </div>
        </div>
        <div class="seq-item">
          <div class="seq-num">4</div>
          <div class="seq-card">
            <div class="who">orderRepository.findById(orderId)</div>
            <div class="act">Interface <code>OrderRepository</code> → <code>JpaOrderRepository</code></div>
            <p class="why">
              <code>JpaOrderRepository</code> charge <code>OrderJpaEntity</code>,
              puis <code>PersistenceMapper.toDomain()</code> → agrégat <code>Order</code>.
              Le service ne voit que <code>Optional&lt;Order&gt;</code>.
            </p>
          </div>
        </div>
        <div class="seq-item">
          <div class="seq-num">5</div>
          <div class="seq-card">
            <div class="who">Order (domaine) <span class="mini-jar">domain</span></div>
            <div class="act">order.markPaymentPending()</div>
            <p class="why">
              Transition d’état VALIDATED → PAYMENT_PENDING.
              Si statut CANCELLED ou PAID : <code>DomainException</code> (refus métier).
            </p>
          </div>
        </div>
        <div class="seq-item">
          <div class="seq-num">6</div>
          <div class="seq-card">
            <div class="who">eventPublisher.publish(OrderStatusChangedEvent)</div>
            <div class="act">Interface <code>EventPublisher</code> → <code>LoggingEventPublisher</code></div>
            <p class="why">Aujourd’hui : log. Demain : Kafka. Même appel dans le service.</p>
          </div>
        </div>
        <div class="seq-item">
          <div class="seq-num">7</div>
          <div class="seq-card">
            <div class="who">new Payment(…) + paymentGateway.charge(…)</div>
            <div class="act">Interface <code>PaymentGateway</code> → <code>FakePaymentGateway.charge()</code></div>
            <p class="why">
              Retourne <code>PaymentResult.Success</code> ou <code>.Failure</code> (sealed interface).
              Montant 13.37 EUR = échec simulé volontaire.
            </p>
          </div>
        </div>
        <div class="seq-item">
          <div class="seq-num">8</div>
          <div class="seq-card">
            <div class="who">Si Success</div>
            <div class="act">payment.markSucceeded() → paymentRepository.save()</div>
            <p class="why">
              <code>PaymentRepository</code> → <code>JpaPaymentRepository</code> → table <code>payments</code>.
              Puis <code>order.markPaid()</code> (domaine) + <code>orderRepository.save()</code>.
            </p>
          </div>
        </div>
        <div class="seq-item">
          <div class="seq-num">9</div>
          <div class="seq-card">
            <div class="who">Pour chaque OrderLine</div>
            <div class="act">stock.confirmReservation(qty) via StockRepository</div>
            <p class="why">
              <code>StockRepository</code> → <code>JpaStockRepository</code>.
              La réservation devient consommation définitive (reserved ↓).
            </p>
          </div>
        </div>
        <div class="seq-item">
          <div class="seq-num">10</div>
          <div class="seq-card">
            <div class="who">Retour vers REST</div>
            <div class="act">ApplicationMapper.toOrderView(order) → RestMapper.toResponse(view)</div>
            <p class="why">
              Domain → OrderView (application-api) → OrderResponse (DTO JSON).
              Quatre formes, une seule vérité métier.
            </p>
          </div>
        </div>
      </div>

      <h3>Carte des objets touchés pendant ce flux</h3>
      <div class="card conn-grid" style="gap:14px">
        <div class="conn-row">
          <div class="rest-box"><span class="kind">REST</span>OrderResource<br/>PayOrderCommand<br/>OrderResponse</div>
          <div class="conn-arrow uses"><span class="sym">⟶</span></div>
          <div class="iface"><span class="kind">API</span>PayOrderUseCase<br/>OrderView</div>
          <div class="conn-arrow impls"><span class="sym">⟸</span></div>
          <div class="impl-box"><span class="kind">CORE</span>PayOrderService<br/>ApplicationMapper</div>
        </div>
        <div class="conn-row">
          <div class="impl-box"><span class="kind">CORE</span>PayOrderService</div>
          <div class="conn-arrow uses"><span class="sym">⟶</span></div>
          <div class="dom-box"><span class="kind">DOMAIN</span>Order · Payment<br/>OrderStatus · Money</div>
          <div class="conn-arrow uses"><span class="sym">⟶</span></div>
          <div class="iface" style="border-color:var(--outbound);background:var(--outbound-soft)">
            <span class="kind" style="color:var(--outbound)">SPI</span>
            OrderRepository<br/>PaymentGateway<br/>StockRepository…
          </div>
        </div>
        <div class="conn-row">
          <div class="iface" style="border-color:var(--outbound);background:var(--outbound-soft)">
            <span class="kind" style="color:var(--outbound)">SPI ports</span>
            (interfaces)
          </div>
          <div class="conn-arrow impls"><span class="sym">⟸</span></div>
          <div class="out-box"><span class="kind">PERSISTENCE</span>Jpa*Repository<br/>PersistenceMapper<br/>*JpaEntity</div>
          <div class="conn-arrow calls"><span class="sym">⟶</span></div>
          <div class="out-box"><span class="kind">DB / PSP</span>PostgreSQL<br/>FakePaymentGateway</div>
        </div>
      </div>
    </section>

    <!-- TOUS USE CASES -->
    <section id="tous-usecases">
      <h2>Tous les Use Cases : qui appelle quoi</h2>
      <p class="section-intro">
        Clique sur un scénario pour voir la chaîne complète
        Resource → Interface → Service → Ports → Implémentations.
      </p>

      <div class="tabs" id="uc-tabs" role="tablist">
        <button type="button" class="active" data-tab="uc-pay">Payer</button>
        <button type="button" data-tab="uc-validate">Valider</button>
        <button type="button" data-tab="uc-create-order">Créer commande</button>
        <button type="button" data-tab="uc-add-line">Ajouter ligne</button>
        <button type="button" data-tab="uc-cancel">Annuler</button>
        <button type="button" data-tab="uc-product">Créer produit</button>
        <button type="button" data-tab="uc-matrix">Matrice globale</button>
      </div>

      <div id="uc-pay" class="tab-panel active">
        <div class="card">
          <pre class="tree">POST /orders/{id}/pay
  OrderResource.pay()
       │  utilise
       ▼
  PayOrderUseCase  ◄── implements ── PayOrderService
                                         │
                    ┌────────────────────┼────────────────────┬──────────────┐
                    ▼                    ▼                    ▼              ▼
             OrderRepository      PaymentGateway      StockRepository   EventPublisher
                    ▲                    ▲                    ▲              ▲
                    │ implements         │                    │              │
             JpaOrderRepository   FakePaymentGateway  JpaStockRepository LoggingEventPublisher
                    │                                         │
                    ▼                                         ▼
               PostgreSQL                                PostgreSQL

  + PaymentRepository ← JpaPaymentRepository
  + IdGenerator ← UuidIdGenerator
  + ClockProvider ← SystemClockProvider
  + Domain : Order.markPaymentPending() / markPaid()
             Payment.markSucceeded()
             Stock.confirmReservation()</pre>
        </div>
      </div>

      <div id="uc-validate" class="tab-panel">
        <div class="card">
          <pre class="tree">POST /orders/{id}/validate
  OrderResource.validate()
       │
       ▼
  ValidateOrderUseCase  ◄── ValidateOrderService
                                │
               ┌────────────────┼────────────────┐
               ▼                ▼                ▼
        OrderRepository   StockRepository   EventPublisher
               ▲                ▲                ▲
        JpaOrderRepository JpaStockRepository LoggingEventPublisher

  Domain appelé dans le service :
    • stock.assertAvailable(qty)     → refuse si stock insuffisant
    • OrderPricingService.price()    → DiscountPolicy (remise)
    • order.validate()               → DRAFT → VALIDATED + stockReserved=true
    • stock.reserve(qty)             → available↓ reserved↑</pre>
        </div>
      </div>

      <div id="uc-create-order" class="tab-panel">
        <div class="card">
          <pre class="tree">POST /orders  { "customerId": "…" }
  OrderResource.create()
       │  CreateOrderRequest → CreateOrderCommand
       ▼
  CreateOrderUseCase  ◄── CreateOrderService
                               │
          ┌────────────────────┼─────────────────┬──────────────┐
          ▼                    ▼                 ▼              ▼
   CustomerRepository   OrderRepository   IdGenerator   ClockProvider
          ▲                    ▲                 ▲              ▲
   JpaCustomerRepository JpaOrderRepository UuidIdGenerator SystemClockProvider

  Étapes :
    1. Vérifier que le customer existe (sinon EntityNotFoundException)
    2. IdGenerator.nextUuid() → OrderId
    3. new Order(id, customerId, clock.now())  statut = DRAFT
    4. orderRepository.save(order)</pre>
        </div>
      </div>

      <div id="uc-add-line" class="tab-panel">
        <div class="card">
          <pre class="tree">POST /orders/{id}/lines  { productId, quantity }
  OrderResource.addLine()
       │  AddOrderLineRequest → AddProductToOrderCommand
       ▼
  AddProductToOrderUseCase  ◄── AddProductToOrderService
                                     │
                ┌────────────────────┼────────────────┐
                ▼                    ▼                ▼
         OrderRepository      ProductRepository  StockRepository
                ▲                    ▲                ▲
         JpaOrderRepository   JpaProductRepository JpaStockRepository

  Domain :
    • Quantity.of(n)           → refuse si ≤ 0
    • product.assertOrderable()→ refuse si inactif
    • stock.assertAvailable()  → refuse si pas assez de stock
    • order.addProduct()       → refuse si statut ≠ DRAFT
    • snapshot du prix unitaire dans OrderLine</pre>
        </div>
      </div>

      <div id="uc-cancel" class="tab-panel">
        <div class="card">
          <pre class="tree">POST /orders/{id}/cancel
  OrderResource.cancel()
       │
       ▼
  CancelOrderUseCase  ◄── CancelOrderService
                               │
          ┌────────────────────┼────────────────┐
          ▼                    ▼                ▼
   OrderRepository      StockRepository   EventPublisher
          ▲                    ▲                ▲
   JpaOrderRepository   JpaStockRepository LoggingEventPublisher

  Domain :
    • order.cancel()              → transition autorisée ?
    • si stock était réservé :
        stock.release(qty)        → reserved↓ available↑
        order.markStockReleased()
  Interdit : annuler une commande déjà PAID</pre>
        </div>
      </div>

      <div id="uc-product" class="tab-panel">
        <div class="card">
          <pre class="tree">POST /products  { name, description, unitPriceEuros, initialStock }
  ProductResource.create()
       │  CreateProductRequest → CreateProductCommand
       ▼
  CreateProductUseCase  ◄── CreateProductService
                                 │
            ┌────────────────────┼────────────────┐
            ▼                    ▼                ▼
     ProductRepository    StockRepository   IdGenerator
            ▲                    ▲                ▲
     JpaProductRepository JpaStockRepository UuidIdGenerator

  Domain :
    • new Product(id, name, desc, Money.euros(price))
    • new Stock(productId, initialStock)   → refuse si stock &lt; 0

  Lecture ensuite :
  GET /products/{id}
    GetProductUseCase ← GetProductService
      → ProductRepository + StockRepository
      → ProductView (prix + availableStock)</pre>
        </div>
      </div>

      <div id="uc-matrix" class="tab-panel">
        <p class="section-intro" style="margin-bottom:12px">
          Croisement <strong>Service applicatif × Port sortant</strong>.
          Une case verte = ce service injecte et appelle ce port.
        </p>
        <div class="card">
          <div class="matrix">
            <div class="mh">Service ↓ / Port →</div>
            <div class="mh">Order<br/>Repo</div>
            <div class="mh">Product<br/>Repo</div>
            <div class="mh">Customer<br/>Repo</div>
            <div class="mh">Stock<br/>Repo</div>
            <div class="mh">Payment<br/>Repo/GW</div>
            <div class="mh">Clock / Id<br/>/ Events</div>

            <div class="ml">CreateProductService</div>
            <div class="cell"></div><div class="cell yes">●</div><div class="cell"></div>
            <div class="cell yes">●</div><div class="cell"></div><div class="cell yes">Id</div>

            <div class="ml">GetProductService</div>
            <div class="cell"></div><div class="cell yes">●</div><div class="cell"></div>
            <div class="cell yes">●</div><div class="cell"></div><div class="cell"></div>

            <div class="ml">AdjustStockService</div>
            <div class="cell"></div><div class="cell"></div><div class="cell"></div>
            <div class="cell yes">●</div><div class="cell"></div><div class="cell"></div>

            <div class="ml">CreateCustomerService</div>
            <div class="cell"></div><div class="cell"></div><div class="cell yes">●</div>
            <div class="cell"></div><div class="cell"></div><div class="cell yes">Id</div>

            <div class="ml">CreateOrderService</div>
            <div class="cell yes">●</div><div class="cell"></div><div class="cell yes">●</div>
            <div class="cell"></div><div class="cell"></div><div class="cell yes">Id+Clock</div>

            <div class="ml">AddProductToOrderService</div>
            <div class="cell yes">●</div><div class="cell yes">●</div><div class="cell"></div>
            <div class="cell yes">●</div><div class="cell"></div><div class="cell"></div>

            <div class="ml">ValidateOrderService</div>
            <div class="cell yes">●</div><div class="cell"></div><div class="cell"></div>
            <div class="cell yes">●</div><div class="cell"></div><div class="cell yes">Evt+Clock</div>

            <div class="ml">PayOrderService</div>
            <div class="cell yes">●</div><div class="cell"></div><div class="cell"></div>
            <div class="cell yes">●</div><div class="cell yes">●</div><div class="cell yes">tous</div>

            <div class="ml">CancelOrderService</div>
            <div class="cell yes">●</div><div class="cell"></div><div class="cell"></div>
            <div class="cell yes">●</div><div class="cell"></div><div class="cell yes">Evt+Clock</div>

            <div class="ml">GetOrderService</div>
            <div class="cell yes">●</div><div class="cell"></div><div class="cell"></div>
            <div class="cell"></div><div class="cell"></div><div class="cell"></div>
          </div>
        </div>
      </div>

      <details class="wire-detail">
        <summary>Tableau complet Resource → UseCase → Service → Impl</summary>
        <div class="detail-body">
          <div class="table-wrap" style="margin-top:14px;box-shadow:none">
            <table>
              <thead>
                <tr>
                  <th>HTTP</th>
                  <th>Resource.méthode</th>
                  <th>Interface inbound</th>
                  <th>Implémentation</th>
                  <th>Ports sortants utilisés</th>
                </tr>
              </thead>
              <tbody>
                <tr>
                  <td><code>POST /products</code></td>
                  <td>ProductResource.create</td>
                  <td>CreateProductUseCase</td>
                  <td>CreateProductService</td>
                  <td>ProductRepo, StockRepo, IdGenerator</td>
                </tr>
                <tr>
                  <td><code>GET /products/{id}</code></td>
                  <td>ProductResource.get</td>
                  <td>GetProductUseCase</td>
                  <td>GetProductService</td>
                  <td>ProductRepo, StockRepo</td>
                </tr>
                <tr>
                  <td><code>POST /products/{id}/stock</code></td>
                  <td>ProductResource.adjustStock</td>
                  <td>AdjustStockUseCase</td>
                  <td>AdjustStockService</td>
                  <td>StockRepo</td>
                </tr>
                <tr>
                  <td><code>POST /customers</code></td>
                  <td>CustomerResource.create</td>
                  <td>CreateCustomerUseCase</td>
                  <td>CreateCustomerService</td>
                  <td>CustomerRepo, IdGenerator</td>
                </tr>
                <tr>
                  <td><code>POST /orders</code></td>
                  <td>OrderResource.create</td>
                  <td>CreateOrderUseCase</td>
                  <td>CreateOrderService</td>
                  <td>OrderRepo, CustomerRepo, Id, Clock</td>
                </tr>
                <tr>
                  <td><code>POST /orders/{id}/lines</code></td>
                  <td>OrderResource.addLine</td>
                  <td>AddProductToOrderUseCase</td>
                  <td>AddProductToOrderService</td>
                  <td>OrderRepo, ProductRepo, StockRepo</td>
                </tr>
                <tr>
                  <td><code>POST /orders/{id}/validate</code></td>
                  <td>OrderResource.validate</td>
                  <td>ValidateOrderUseCase</td>
                  <td>ValidateOrderService</td>
                  <td>OrderRepo, StockRepo, Events, Clock</td>
                </tr>
                <tr>
                  <td><code>POST /orders/{id}/pay</code></td>
                  <td>OrderResource.pay</td>
                  <td>PayOrderUseCase</td>
                  <td>PayOrderService</td>
                  <td>Order/Payment/Stock repos, Gateway, Events, Id, Clock</td>
                </tr>
                <tr>
                  <td><code>POST /orders/{id}/cancel</code></td>
                  <td>OrderResource.cancel</td>
                  <td>CancelOrderUseCase</td>
                  <td>CancelOrderService</td>
                  <td>OrderRepo, StockRepo, Events, Clock</td>
                </tr>
                <tr>
                  <td><code>GET /orders/{id}</code></td>
                  <td>OrderResource.get</td>
                  <td>GetOrderUseCase</td>
                  <td>GetOrderService</td>
                  <td>OrderRepo</td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </details>

      <details class="wire-detail">
        <summary>Comment lire une flèche « utilise » vs « implémente »</summary>
        <div class="detail-body">
          <div class="compare" style="margin-top:14px">
            <div class="good">
              <h4>utilise (dépendance)</h4>
              <pre class="tree">class OrderResource {
  private final PayOrderUseCase uc;
  // OrderResource CONNAÎT le type interface
  // Il APPELLE uc.execute(...)
}

Sens Maven :
  adapter-rest → application-api</pre>
            </div>
            <div class="good">
              <h4>implémente (réalisation)</h4>
              <pre class="tree">class PayOrderService
  implements PayOrderUseCase {
  // PayOrderService Fournit le comportement
  // Quarkus l'injecte là où PayOrderUseCase
  // est demandé
}

Sens Maven :
  application-core → application-api
  (core dépend du contrat qu'il réalise)</pre>
            </div>
          </div>
          <div class="callout">
            <strong>Piège classique :</strong> croire que « qui implémente dépend de qui utilise ».
            Non : <em>les deux</em> dépendent du <strong>contrat</strong> (l’interface).
            L’utilisateur et l’implémenteur regardent le même port, depuis deux côtés.
          </div>
        </div>
      </details>
    </section>

    <!-- PORTS -->
    <section id="ports">
      <h2>Ports : qui définit, qui utilise, qui implémente</h2>
      <p class="section-intro">
        Une interface n’est pas “juste du découplage”. Elle fixe
        <strong>où vit le contrat</strong> et <strong>dans quel sens</strong> va la dépendance.
      </p>

      <div class="table-wrap">
        <table>
          <thead>
            <tr>
              <th>Interface</th>
              <th>Type</th>
              <th>Définie dans</th>
              <th>Utilisée par</th>
              <th>Implémentée par</th>
            </tr>
          </thead>
          <tbody>
            <tr>
              <td><code>PayOrderUseCase</code></td>
              <td><span class="tag in">Inbound</span></td>
              <td>application-api</td>
              <td>OrderResource</td>
              <td>PayOrderService</td>
            </tr>
            <tr>
              <td><code>CreateOrderUseCase</code></td>
              <td><span class="tag in">Inbound</span></td>
              <td>application-api</td>
              <td>OrderResource</td>
              <td>CreateOrderService</td>
            </tr>
            <tr>
              <td><code>ValidateOrderUseCase</code></td>
              <td><span class="tag in">Inbound</span></td>
              <td>application-api</td>
              <td>OrderResource</td>
              <td>ValidateOrderService</td>
            </tr>
            <tr>
              <td><code>OrderRepository</code></td>
              <td><span class="tag out">Outbound</span></td>
              <td>application-spi</td>
              <td>services core</td>
              <td>JpaOrderRepository</td>
            </tr>
            <tr>
              <td><code>PaymentGateway</code></td>
              <td><span class="tag out">Outbound</span></td>
              <td>application-spi</td>
              <td>PayOrderService</td>
              <td>FakePaymentGateway</td>
            </tr>
            <tr>
              <td><code>StockRepository</code></td>
              <td><span class="tag out">Outbound</span></td>
              <td>application-spi</td>
              <td>Validate / Pay / Cancel</td>
              <td>JpaStockRepository</td>
            </tr>
            <tr>
              <td><code>EventPublisher</code></td>
              <td><span class="tag out">Outbound</span></td>
              <td>application-spi</td>
              <td>Validate / Pay / Cancel</td>
              <td>LoggingEventPublisher</td>
            </tr>
          </tbody>
        </table>
      </div>

      <h3>Exemple détaillé : <code>PaymentGateway</code></h3>
      <div class="card">
        <pre class="tree">Défini par     :  application-spi
Utilisé par    :  PayOrderService          (application-core)
Implémenté par :  FakePaymentGateway       (infrastructure-payment)
Remplaçable par:  StripePaymentGateway     (sans toucher au métier)

Raison :
  le métier dit  « j’ai besoin d’encaisser »
  l’infra dit    « voici comment (fake / Stripe / Adyen) »</pre>
      </div>
    </section>

    <!-- DIP -->
    <section id="dip">
      <h2>Inversion de dépendances</h2>
      <p class="section-intro">
        Sans ports, le métier dépend des technologies. Avec ports, les technologies
        dépendent des besoins du métier.
      </p>

      <div class="compare">
        <div class="bad">
          <h4>Mauvais</h4>
          <pre class="tree">PayOrderService
    ↓
JpaOrderRepository
    ↓
PostgreSQL

PayOrderService
    ↓
StripeClient</pre>
        </div>
        <div class="good">
          <h4>Bon</h4>
          <pre class="tree">PayOrderService
    ↓
OrderRepository  ←── JpaOrderRepository

PayOrderService
    ↓
PaymentGateway   ←── FakePaymentGateway
                 ←── StripePaymentGateway</pre>
        </div>
      </div>
    </section>

    <!-- MAPPING -->
    <section id="mapping">
      <h2>Quatre modèles, quatre raisons</h2>
      <p class="section-intro">
        Ce ne sont pas les mêmes objets. Chacun évolue pour sa frontière.
      </p>

      <div class="card">
        <div class="map-strip">
          <div class="map-box">
            <strong>REST DTO</strong>
            <span style="color:var(--muted)">JSON API</span>
          </div>
          <div class="map-arrow">→</div>
          <div class="map-box">
            <strong>Command / View</strong>
            <span style="color:var(--muted)">application-api</span>
          </div>
          <div class="map-arrow">→</div>
          <div class="map-box">
            <strong>Domain</strong>
            <span style="color:var(--muted)">Order, Money…</span>
          </div>
          <div class="map-arrow">→</div>
          <div class="map-box">
            <strong>JPA Entity</strong>
            <span style="color:var(--muted)">OrderJpaEntity</span>
          </div>
        </div>
      </div>

      <div class="grid-2" style="margin-top:16px">
        <div class="card">
          <h3 style="margin-top:0">Pourquoi pas d’annotations JPA sur le domaine&nbsp;?</h3>
          <p style="color:var(--muted);margin:0">
            Hibernate impose constructeur vide, lazy-loading, noms de colonnes…
            Le JAR <code>domain</code> resterait prisonnier de JPA et inutilisable
            hors Quarkus.
          </p>
        </div>
        <div class="card">
          <h3 style="margin-top:0">Mapping</h3>
          <pre class="tree">Domain Order
      ↓
PersistenceMapper
      ↓
OrderJpaEntity
      ↓
PostgreSQL</pre>
        </div>
      </div>
    </section>

    <!-- FLUX -->
    <section id="flux">
      <h2>Flux complet : <code>POST /orders/{id}/pay</code></h2>
      <p class="section-intro">
        Suivre un appel de bout en bout pour voir chaque JAR intervenir.
      </p>

      <div class="flow">
        <div class="flow-step">
          <div class="flow-rail"><div class="flow-dot"></div><div class="flow-line"></div></div>
          <div class="flow-body">
            <div class="jar">HTTP</div>
            <div class="title">Requête client</div>
            <p><code>POST /orders/{id}/pay</code></p>
          </div>
        </div>
        <div class="flow-step">
          <div class="flow-rail"><div class="flow-dot"></div><div class="flow-line"></div></div>
          <div class="flow-body">
            <div class="jar">adapter-rest</div>
            <div class="title">OrderResource.pay()</div>
            <p>Pas de métier ici. Appelle uniquement l’interface <code>PayOrderUseCase</code>.</p>
          </div>
        </div>
        <div class="flow-step">
          <div class="flow-rail"><div class="flow-dot"></div><div class="flow-line"></div></div>
          <div class="flow-body">
            <div class="jar">application-api</div>
            <div class="title">PayOrderUseCase</div>
            <p>Contrat inbound. Le REST dépend de ça, pas de <code>PayOrderService</code>.</p>
          </div>
        </div>
        <div class="flow-step">
          <div class="flow-rail"><div class="flow-dot"></div><div class="flow-line"></div></div>
          <div class="flow-body">
            <div class="jar">application-core</div>
            <div class="title">PayOrderService</div>
            <p>Charge la commande, demande le paiement, confirme le stock, publie des events.</p>
          </div>
        </div>
        <div class="flow-step">
          <div class="flow-rail"><div class="flow-dot"></div><div class="flow-line"></div></div>
          <div class="flow-body">
            <div class="jar">application-spi → infrastructure-persistence</div>
            <div class="title">OrderRepository → JpaOrderRepository</div>
            <p>Lecture / écriture PostgreSQL via mapper Domain ↔ JPA.</p>
          </div>
        </div>
        <div class="flow-step">
          <div class="flow-rail"><div class="flow-dot"></div><div class="flow-line"></div></div>
          <div class="flow-body">
            <div class="jar">application-spi → infrastructure-payment</div>
            <div class="title">PaymentGateway → FakePaymentGateway</div>
            <p>Simulation PSP. Remplaçable par Stripe sans changer <code>PayOrderService</code>.</p>
          </div>
        </div>
        <div class="flow-step">
          <div class="flow-rail"><div class="flow-dot"></div></div>
          <div class="flow-body">
            <div class="jar">adapter-rest</div>
            <div class="title">OrderResponse JSON</div>
            <p><code>OrderView</code> → <code>RestMapper</code> → DTO HTTP.</p>
          </div>
        </div>
      </div>
    </section>

    <!-- CDI -->
    <section id="cdi">
      <h2>Comment Quarkus branche les JAR</h2>
      <p class="section-intro">
        <code>quarkus-app</code> est le <strong>composition root</strong> : il met tous les JAR
        sur le classpath. Jandex indexe les beans CDI de chaque module.
      </p>

      <div class="grid-2">
        <div class="card">
          <h3 style="margin-top:0">Injection par type</h3>
          <pre class="tree">PayOrderUseCase
   └── une seule @ApplicationScoped
       = PayOrderService

OrderRepository
   └── une seule @ApplicationScoped
       = JpaOrderRepository

PaymentGateway
   └── une seule @ApplicationScoped
       = FakePaymentGateway</pre>
        </div>
        <div class="card">
          <h3 style="margin-top:0">Pourquoi ça marche</h3>
          <ol style="margin:0;padding-left:18px;color:var(--muted)">
            <li>Chaque module beans a le plugin <strong>Jandex</strong></li>
            <li><code>quarkus-app</code> dépend de tous les JAR</li>
            <li>Arc voit 1 implémentation par interface</li>
            <li>Injection constructeur → câblage automatique</li>
          </ol>
        </div>
      </div>
    </section>

    <!-- TESTS -->
    <section id="tests">
      <h2>Pyramide de tests</h2>
      <div class="table-wrap">
        <table>
          <thead>
            <tr>
              <th>Niveau</th>
              <th>Où</th>
              <th>Quoi prouver</th>
              <th>Techno</th>
            </tr>
          </thead>
          <tbody>
            <tr>
              <td><strong>Domaine</strong></td>
              <td><code>domain/src/test</code></td>
              <td>Invariants, remises, stock, statuts</td>
              <td>JUnit seul</td>
            </tr>
            <tr>
              <td><strong>Application</strong></td>
              <td><code>application-core/src/test</code></td>
              <td>Orchestration Use Cases</td>
              <td>Fakes + Mockito</td>
            </tr>
            <tr>
              <td><strong>Intégration</strong></td>
              <td><code>quarkus-app/src/test</code></td>
              <td>Câblage CDI + HTTP bout-en-bout</td>
              <td>Quarkus + H2 + REST Assured</td>
            </tr>
          </tbody>
        </table>
      </div>
      <div class="callout">
        <strong>Intérêt de l’hexagone :</strong> tester <code>PayOrderService</code>
        sans PostgreSQL et sans vraie API de paiement.
      </div>
    </section>

    <!-- EVOLUER -->
    <section id="evoluer">
      <h2>Comment faire évoluer sans casser le métier</h2>
      <div class="grid-3">
        <div class="card">
          <h3 style="margin-top:0">MongoDB à la place de PG</h3>
          <p style="color:var(--muted);margin:0">
            Nouveau module qui implémente les ports spi.
            Changer la dépendance dans <code>quarkus-app</code> seulement.
          </p>
        </div>
        <div class="card">
          <h3 style="margin-top:0">Stripe à la place du Fake</h3>
          <p style="color:var(--muted);margin:0">
            <code>StripePaymentGateway implements PaymentGateway</code>.
            Zéro changement dans <code>PayOrderService</code>.
          </p>
        </div>
        <div class="card">
          <h3 style="margin-top:0">CLI ou Kafka</h3>
          <p style="color:var(--muted);margin:0">
            Nouvel adaptateur inbound qui dépend de <code>application-api</code>.
            Mêmes Use Cases, autre porte d’entrée.
          </p>
        </div>
      </div>
    </section>

    <!-- SYNTHÈSE MODULES -->
    <section id="synthese">
      <h2>Tableau modules</h2>
      <div class="table-wrap">
        <table>
          <thead>
            <tr>
              <th>Module</th>
              <th>Type</th>
              <th>Dépend de</th>
              <th>Ne doit pas connaître</th>
            </tr>
          </thead>
          <tbody>
            <tr>
              <td><code>domain</code></td>
              <td><span class="tag dom">Coeur</span></td>
              <td>—</td>
              <td>Quarkus, JPA, REST, SQL</td>
            </tr>
            <tr>
              <td><code>application-api</code></td>
              <td><span class="tag in">Inbound</span></td>
              <td>domain</td>
              <td>JPA, REST, Quarkus</td>
            </tr>
            <tr>
              <td><code>application-spi</code></td>
              <td><span class="tag out">Outbound</span></td>
              <td>domain</td>
              <td>JPA, Stripe, HTTP</td>
            </tr>
            <tr>
              <td><code>application-core</code></td>
              <td>Application</td>
              <td>api + spi + domain</td>
              <td>PostgreSQL, REST, Stripe</td>
            </tr>
            <tr>
              <td><code>infrastructure-persistence</code></td>
              <td>Adapter out</td>
              <td>spi + domain</td>
              <td>REST, Use Cases</td>
            </tr>
            <tr>
              <td><code>infrastructure-payment</code></td>
              <td>Adapter out</td>
              <td>spi + domain</td>
              <td>REST</td>
            </tr>
            <tr>
              <td><code>adapter-rest</code></td>
              <td>Adapter in</td>
              <td>api + domain</td>
              <td>JPA, PaymentGateway</td>
            </tr>
            <tr>
              <td><code>quarkus-app</code></td>
              <td>Bootstrap</td>
              <td><strong>tous</strong></td>
              <td>Logique métier</td>
            </tr>
          </tbody>
        </table>
      </div>
    </section>

    <!-- SKILL & PROMPT -->
    <section id="skill">
      <h2>Skill Cursor + Prompt réutilisable</h2>
      <p class="section-intro">
        Tout est ici : le contenu du skill Agent, la checklist de transformation,
        et les prompts à copier-coller pour créer ou transformer un projet basique
        en architecture hexagonale avec interfaces.
      </p>

      <div class="tabs" id="skill-tabs" role="tablist">
        <button type="button" class="active" data-tab="sk-overview">Vue d’ensemble</button>
        <button type="button" data-tab="sk-skill">Contenu du Skill</button>
        <button type="button" data-tab="sk-prompt">Prompt complet</button>
        <button type="button" data-tab="sk-short">Prompt court</button>
        <button type="button" data-tab="sk-checklist">Checklist transform</button>
      </div>

      <!-- OVERVIEW -->
      <div id="sk-overview" class="tab-panel active skill-block">
        <div class="grid-2">
          <div class="card">
            <h3 style="margin-top:0">Qu’est-ce qu’un Skill&nbsp;?</h3>
            <p style="color:var(--muted);margin:0">
              Un fichier d’instructions que Cursor lit automatiquement quand tu parles
              d’architecture hexagonale, Ports &amp; Adapters, multi-modules, etc.
              Il force l’agent à respecter les règles de ce projet de référence.
            </p>
          </div>
          <div class="card">
            <h3 style="margin-top:0">Où il vit dans ce repo</h3>
            <pre class="tree" style="margin:0">order-platform/
└── .cursor/skills/
    └── hexagonal-java-quarkus/
        ├── SKILL.md
        ├── PROMPT.md
        └── reference.md

(équivalent aussi intégré ci-dessous)</pre>
          </div>
        </div>

        <h4>Comment l’utiliser</h4>
        <ul class="checklist">
          <li><span class="n">1</span><span>Ouvre un chat <strong>Agent</strong> dans Cursor sur le projet à créer ou transformer.</span></li>
          <li><span class="n">2</span><span>Dis par exemple : « utilise le skill hexagonal » ou colle un des prompts ci-dessous.</span></li>
          <li><span class="n">3</span><span>Remplis le bloc <code>CONTEXTE</code> (nom, stack, métier, contraintes).</span></li>
          <li><span class="n">4</span><span>L’agent doit produire modules Maven, interfaces, adaptateurs, tests et graphes.</span></li>
        </ul>

        <div class="callout" style="margin-top:18px">
          <strong>Astuce :</strong> tu peux aussi mentionner ce fichier HTML comme référence visuelle
          (<code>docs/architecture.html</code> pour que l’agent aligne le même découpage de JAR.
        </div>
      </div>

      <!-- SKILL CONTENT -->
      <div id="sk-skill" class="tab-panel skill-block">
        <div class="card">
          <h3 style="margin-top:0">Skill <code>hexagonal-java-quarkus</code></h3>
          <p style="color:var(--muted)">
            Description (triggers) : designs or transforms Java/Quarkus projects into hexagonal
            architecture with Maven multi-modules, inbound/outbound ports as interfaces,
            domain free of frameworks. Use when the user asks for architecture hexagonale,
            Ports &amp; Adapters, Clean Architecture, inversion de dépendances, multi-modules,
            refactor d’un UserService/Repository, etc.
          </p>

          <h4>Layout cible</h4>
          <pre class="tree">project/
├── domain/
├── application-api/             # Inbound Ports (UseCase) + commands/views
├── application-spi/             # Outbound Ports (Repository, Gateway, Clock…)
├── application-core/            # implémentations UseCase
├── infrastructure-persistence/
├── infrastructure-*/            # payment, mail…
├── adapter-rest/
└── quarkus-app/                 # composition root</pre>

          <h4>Règles dures</h4>
          <ol style="color:var(--muted);padding-left:18px;margin:0">
            <li>Adapters → ports → domain. Jamais domain → Quarkus/JPA/REST.</li>
            <li>Inbound ports dans <code>application-api</code>. REST injecte des interfaces.</li>
            <li>Outbound ports dans <code>application-spi</code>. Use cases → interfaces seulement.</li>
            <li>REST DTO ≠ Command/View ≠ Domain ≠ JpaEntity.</li>
            <li>Pas d’annotations JPA sur le domaine. Mapper explicite.</li>
            <li>CDI + Jandex ; une impl @ApplicationScoped par port.</li>
            <li><code>adapter-rest</code> ne dépend pas de <code>infrastructure-*</code>.</li>
            <li>Pas de sur-engineering : chaque interface a une justification.</li>
          </ol>

          <h4>Mapping before → after</h4>
          <div class="table-wrap" style="margin-top:12px;box-shadow:none">
            <table>
              <thead>
                <tr><th>Avant (basique)</th><th>Après (hexagonal)</th></tr>
              </thead>
              <tbody>
                <tr>
                  <td><code>@Entity class User</code> partout</td>
                  <td>Domain <code>User</code> + <code>UserJpaEntity</code> + mapper</td>
                </tr>
                <tr>
                  <td>Resource → Service → PanacheRepo</td>
                  <td>Resource → UseCase → Service → Port → JpaAdapter</td>
                </tr>
                <tr>
                  <td>Règles dans service/controller</td>
                  <td>Règles dans entité / domain service</td>
                </tr>
                <tr>
                  <td>Stripe appelé depuis le service</td>
                  <td><code>PaymentGateway</code> + adaptateur Stripe/Fake</td>
                </tr>
              </tbody>
            </table>
          </div>

          <h4>Anti-patterns à rejeter</h4>
          <ul style="color:var(--muted);margin:0">
            <li>Controllers qui appellent Panache / EntityManager</li>
            <li>Domaine qui dépend de <code>jakarta.persistence</code> ou Quarkus</li>
            <li>Retourner des entités JPA en JSON</li>
            <li>Mixer inbound et outbound dans le même JAR</li>
            <li><code>adapter-rest</code> → <code>infrastructure-persistence</code></li>
          </ul>
        </div>
      </div>

      <!-- PROMPT FULL -->
      <div id="sk-prompt" class="tab-panel skill-block">
        <p class="section-intro">
          Prompt long — à coller dans un chat Agent. Remplace les <code>&lt;…&gt;</code>.
        </p>
        <div class="copy-wrap">
          <button type="button" class="copy-btn" data-copy="prompt-full">Copier</button>
          <pre id="prompt-full" class="copy-box">Agis comme un architecte logiciel Java senior (Java 25, Quarkus, architecture hexagonale,
Ports &amp; Adapters, Clean Architecture, DDD, Maven multi-modules).

OBJECTIF
Transforme (ou crée) mon projet pour qu’il utilise des interfaces entre les couches,
avec inversion de dépendances, sans polluer le domaine métier avec Quarkus / JPA / REST.

CONTEXTE DU PROJET
- Nom : &lt;NOM&gt;
- Stack actuelle : &lt;ex. Quarkus monolithique, packages controller/service/repository&gt;
- Domaine métier : &lt;ex. commandes, stock, paiement / ou décrire&gt;
- Ce que je veux conserver : &lt;API existante, schéma DB, …&gt;
- Contraintes : &lt;Java 25 + Quarkus LTS si possible ; sinon expliquer et proposer le plus proche&gt;

ARCHITECTURE CIBLE (obligatoire)
  domain
  application-api          (Inbound Ports = interfaces UseCase + commands/views)
  application-spi          (Outbound Ports = Repository, Gateway, Clock, IdGenerator…)
  application-core         (implémentations UseCase ; orchestration seulement)
  infrastructure-persistence
  infrastructure-&lt;autre&gt;   (ex. payment, mail…)
  adapter-rest
  quarkus-app              (composition root uniquement)

RÈGLES DURES
1. domain : zéro dépendance Quarkus / REST / Hibernate / Panache / SQL.
2. Les controllers REST dépendent des interfaces UseCase, JAMAIS des classes *Service concrètes.
3. Les Use Cases dépendent des ports sortants (interfaces), JAMAIS de JpaRepository / clients HTTP concrets.
4. Distinguer : REST DTO ≠ Command/View ≠ Domain Object ≠ JpaEntity (mappers explicites).
5. PAS d’annotations JPA sur les objets métier.
6. CDI : une impl @ApplicationScoped par port ; jandex sur les modules à beans.
7. adapter-rest ne doit PAS dépendre de infrastructure-persistence.
8. Éviter le sur-engineering : chaque interface doit avoir une justification claire.

LIVRABLES
1. Arborescence complète des modules + rôle de chaque package
2. pom.xml parent + modules + graphe des dépendances Maven
3. Code domaine riche (VOs, règles, transitions interdites)
4. Interfaces inbound + outbound commentées (qui définit / utilise / implémente / pourquoi)
5. Adaptateurs (persistence + au moins un autre sortant, ex. FakePaymentGateway)
6. REST thin + ExceptionMapper domaine → HTTP
7. Bootstrap Quarkus (assemblage JAR + application.properties)
8. Tests : domaine sans Quarkus ; use cases avec fakes/mocks ; IT REST Assured
9. Un flux bout-en-bout documenté avec classe + JAR à chaque étape
10. Tableaux de synthèse modules + interfaces
11. README : Build, Run, swap PostgreSQL / FakeGateway / ajouter Kafka

ORDRE DE TRAVAIL
1) Architecture + graphe  2) Maven  3) Domaine  4) application-api
5) application-spi  6) application-core  7) infrastructure-*
8) adapter-rest  9) quarkus-app  10) Tests  11) Documentation

Avant de coder : vérifier la compatibilité réelle Quarkus ↔ version Java demandée.
Ne simplifie pas l’architecture juste pour réduire le code.
Ne sur-engineere pas : chaque abstraction doit être justifiée.</pre>
        </div>
      </div>

      <!-- PROMPT SHORT -->
      <div id="sk-short" class="tab-panel skill-block">
        <p class="section-intro">
          Variante courte — idéal si le repo existe déjà et que tu veux un refactor rapide.
        </p>
        <div class="copy-wrap">
          <button type="button" class="copy-btn" data-copy="prompt-short">Copier</button>
          <pre id="prompt-short" class="copy-box">Refactor hexagonal de ce repo Quarkus/Java.

Extrais un vrai `domain` pur, crée `application-api` (UseCases) et `application-spi`
(Repositories/Gateways), déplace l’orchestration dans `application-core`,
implémente JPA dans `infrastructure-persistence` avec Domain ≠ JpaEntity + mapper,
garde `adapter-rest` dépendant uniquement des interfaces UseCase,
assemble dans `quarkus-app`.

Montre le graphe Maven, le tableau des interfaces (définie/utilisée/implémentée),
un flux HTTP bout-en-bout, et des tests domaine + use case sans Quarkus.

Interdits : annotations JPA sur le domaine ; REST → infrastructure ; logique métier dans les controllers.

Référence visuelle / pédagogique si besoin :
order-platform/docs/architecture.html</pre>
        </div>
      </div>

      <!-- CHECKLIST -->
      <div id="sk-checklist" class="tab-panel skill-block">
        <p class="section-intro">
          Checklist que l’agent (ou toi) doit suivre pour transformer un projet basique.
        </p>
        <ul class="checklist">
          <li><span class="n">1</span><span>Inventorier packages actuels (controllers, services, entities, repos)</span></li>
          <li><span class="n">2</span><span>Extraire le domaine pur (entities / VOs / règles) → module <code>domain</code></span></li>
          <li><span class="n">3</span><span>Définir les UseCase + Commands/Views → <code>application-api</code></span></li>
          <li><span class="n">4</span><span>Définir les ports sortants → <code>application-spi</code></span></li>
          <li><span class="n">5</span><span>Déplacer l’orchestration → <code>application-core</code> (implements UseCases)</span></li>
          <li><span class="n">6</span><span>Créer JPA entities + mappers + adapters → <code>infrastructure-*</code></span></li>
          <li><span class="n">7</span><span>REST mince dépendant uniquement de <code>application-api</code></span></li>
          <li><span class="n">8</span><span>Assembler dans <code>quarkus-app</code> (POMs + config + beans techniques)</span></li>
          <li><span class="n">9</span><span>Tests domaine + application sans Quarkus</span></li>
          <li><span class="n">10</span><span>Vérifier : aucune dépendance Maven illégale ; <code>mvn clean verify</code></span></li>
        </ul>

        <h4>Livrables attendus à la fin</h4>
        <div class="grid-2">
          <div class="card">
            <ul style="margin:0;padding-left:18px;color:var(--muted)">
              <li>Arborescence + rôle de chaque JAR</li>
              <li>Graphe Maven</li>
              <li>Tableau des interfaces</li>
              <li>Un flux HTTP bout-en-bout</li>
            </ul>
          </div>
          <div class="card">
            <ul style="margin:0;padding-left:18px;color:var(--muted)">
              <li>Comment swap PostgreSQL → Mongo</li>
              <li>Comment swap Fake → Stripe</li>
              <li>Comment ajouter CLI / Kafka</li>
              <li>Comment tester sans lancer Quarkus</li>
            </ul>
          </div>
        </div>
      </div>
    </section>

    <footer>
      Fichier généré pour le projet <strong>order-platform</strong> —
      ouvre cette page dans ton navigateur :
      <code>docs/architecture.html</code>.
      Pour le détail textuel, voir aussi <code>README.md</code>.
    </footer>
  </div>
  <script>
    (function () {
      function wireTabs(containerSelector) {
        const root = document.querySelector(containerSelector);
        if (!root) return;
        const tabs = root.querySelectorAll('button[data-tab]');
        tabs.forEach((btn) => {
          btn.addEventListener('click', () => {
            const parentSection = root.closest('section') || document;
            tabs.forEach((b) => b.classList.remove('active'));
            btn.classList.add('active');
            // Hide only panels that belong to this tab group (siblings under same section after the tabs)
            const panelIds = Array.from(tabs).map((b) => b.dataset.tab);
            panelIds.forEach((id) => {
              const panel = document.getElementById(id);
              if (panel) panel.classList.remove('active');
            });
            const panel = document.getElementById(btn.dataset.tab);
            if (panel) panel.classList.add('active');
          });
        });
      }

      wireTabs('#uc-tabs');
      wireTabs('#skill-tabs');

      document.querySelectorAll('.copy-btn').forEach((btn) => {
        btn.addEventListener('click', async () => {
          const id = btn.getAttribute('data-copy');
          const el = document.getElementById(id);
          if (!el) return;
          const text = el.innerText || el.textContent;
          try {
            await navigator.clipboard.writeText(text);
            const old = btn.textContent;
            btn.textContent = 'Copié ✓';
            setTimeout(() => { btn.textContent = old; }, 1600);
          } catch (e) {
            btn.textContent = 'Sélectionne le texte';
          }
        });
      });
    })();
  </script>
</body>
</html>
