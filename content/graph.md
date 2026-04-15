---
title: "Knowledge Graph"
layout: "single"
summary: "Interactive visualization of the note network"
ShowReadingTime: false
ShowShareButtons: false
ShowPostNavLinks: false
ShowBreadCrumbs: true
---

<style>
.post-content { max-width: none !important; }
.main { max-width: none !important; }
.post-single { max-width: none !important; padding: 0 1em !important; }
#graph-canvas {
  width: 100%;
  height: 85vh;
  border: 1px solid #333;
  border-radius: 8px;
  margin: 1em 0;
  background: #1a1a2e;
  position: relative;
  overflow: hidden;
}
#controls-panel {
  position: absolute;
  top: 10px;
  left: 10px;
  background: rgba(22, 33, 62, 0.95);
  border: 1px solid #444;
  border-radius: 8px;
  padding: 12px 16px;
  font-size: 0.8em;
  color: #ccc;
  z-index: 10;
  width: 240px;
  max-height: 80vh;
  overflow-y: auto;
}
#controls-panel.collapsed .control-section { display: none; }
#toggle-btn {
  cursor: pointer;
  user-select: none;
  margin: 0 0 4px 0;
  color: #64ffda;
  font-size: 0.95em;
}
#controls-panel h4 {
  margin: 0 0 6px 0;
  color: #64ffda;
  font-size: 0.9em;
}
#controls-panel label {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin: 5px 0;
  gap: 6px;
}
#controls-panel label span:first-child { white-space: nowrap; min-width: 65px; }
#controls-panel input[type="range"] {
  width: 90px;
  accent-color: #64ffda;
}
#controls-panel select {
  width: 100%;
  padding: 3px 6px;
  background: #2a2a3e;
  color: #e0e0e0;
  border: 1px solid #555;
  border-radius: 4px;
  margin: 3px 0;
}
#controls-panel button {
  width: 100%;
  padding: 5px;
  background: #2a2a3e;
  color: #64ffda;
  border: 1px solid #555;
  border-radius: 4px;
  cursor: pointer;
  margin-top: 6px;
}
#controls-panel button:hover { background: #3a3a5e; }
.control-section {
  border-top: 1px solid #333;
  padding-top: 8px;
  margin-top: 8px;
}
#note-popup {
  display: none;
  position: absolute;
  top: 10px;
  right: 10px;
  width: 350px;
  max-height: 70vh;
  overflow-y: auto;
  background: rgba(22, 33, 62, 0.97);
  border: 1px solid #555;
  border-radius: 8px;
  padding: 1em;
  font-size: 0.85em;
  line-height: 1.5;
  z-index: 10;
  color: #e0e0e0;
}
#note-popup h3 { margin-top: 0; color: #64ffda; }
#popup-actions { display: flex; gap: 8px; margin-top: 8px; }
#popup-actions button {
  flex: 1; padding: 5px; background: #2a2a3e; color: #64ffda;
  border: 1px solid #555; border-radius: 4px; cursor: pointer; font-size: 0.85em;
}
#popup-actions button:hover { background: #3a3a5e; }
.close-btn { float: right; cursor: pointer; font-size: 1.2em; color: #999; }
.close-btn:hover { color: #fff; }
.note-type { font-size: 0.75em; text-transform: uppercase; color: #888; margin-bottom: 0.5em; }
#note-popup pre {
  white-space: pre-wrap; word-wrap: break-word; font-size: 0.85em;
  background: #0a0a1a; padding: 0.8em; border-radius: 4px; max-height: 250px; overflow-y: auto;
}
.legend { display: flex; gap: 1rem; flex-wrap: wrap; font-size: 0.8em; margin-bottom: 0.5em; }
.legend-item { display: flex; align-items: center; gap: 4px; }
.legend-dot { width: 12px; height: 12px; border-radius: 50%; display: inline-block; }
</style>

<div class="legend">
  <div class="legend-item"><span class="legend-dot" style="background:#64ffda"></span> Atomic Note</div>
  <div class="legend-item"><span class="legend-dot" style="background:#ff6b6b"></span> Article</div>
  <div class="legend-item"><span class="legend-dot" style="background:#ffd93d"></span> Map of Content</div>
  <div class="legend-item"><span class="legend-dot" style="background:#6bcb77"></span> Source</div>
  <div class="legend-item"><span class="legend-dot" style="background:#9b59b6"></span> System</div>
</div>

<div id="graph-canvas">
  <div id="controls-panel">
    <h4 id="toggle-btn">Controls ▼</h4>
    <div class="control-section">
      <h4>Filters</h4>
      <div style="display:flex;gap:8px;margin:4px 0 2px 0"><a href="#" id="select-all" style="color:#64ffda;font-size:0.85em">All</a> <a href="#" id="select-none" style="color:#64ffda;font-size:0.85em">None</a></div>
      <div style="display:flex;flex-wrap:wrap;gap:6px;margin:4px 0">
        <label style="display:flex;align-items:center;gap:3px;margin:0"><input type="checkbox" class="type-cb" value="atom" checked> <span class="legend-dot" style="background:#64ffda;width:8px;height:8px"></span> Atoms</label>
        <label style="display:flex;align-items:center;gap:3px;margin:0"><input type="checkbox" class="type-cb" value="article" checked> <span class="legend-dot" style="background:#ff6b6b;width:8px;height:8px"></span> Articles</label>
        <label style="display:flex;align-items:center;gap:3px;margin:0"><input type="checkbox" class="type-cb" value="moc" checked> <span class="legend-dot" style="background:#ffd93d;width:8px;height:8px"></span> MOCs</label>
        <label style="display:flex;align-items:center;gap:3px;margin:0"><input type="checkbox" class="type-cb" value="source" checked> <span class="legend-dot" style="background:#6bcb77;width:8px;height:8px"></span> Sources</label>
        <label style="display:flex;align-items:center;gap:3px;margin:0"><input type="checkbox" class="type-cb" value="system" checked> <span class="legend-dot" style="background:#9b59b6;width:8px;height:8px"></span> System</label>
      </div>
      <label><span>Depth</span> <span id="depth-val">All</span><input type="range" id="depth-slider" min="1" max="5" value="5"></label>
      <button id="reset-btn">Reset view</button>
    </div>
    <div class="control-section">
      <h4>Display</h4>
      <label><span>Labels</span><select id="label-mode" style="width:95px"><option value="always">Always on</option><option value="hover">On hover</option><option value="off">Off</option></select></label>
      <label><span>Node size</span> <span id="nsize-val">3</span><input type="range" id="node-size" min="1" max="8" value="3"></label>
      <label><span>Link width</span> <span id="lwidth-val">1</span><input type="range" id="link-width" min="0.5" max="4" value="1" step="0.5"></label>
      <label><span>Label size</span> <span id="lsize-val">10</span><input type="range" id="label-size" min="0" max="16" value="10"></label>
      <label><span>Label fade</span> <span id="lfade-val">1.2</span><input type="range" id="label-fade" min="0.3" max="3" value="1.2" step="0.1"></label>
      <label><span>Spacing</span> <span id="spacing-val">1</span><input type="range" id="spacing" min="0.5" max="4" value="1" step="0.25"></label>
    </div>
    <div class="control-section">
      <h4>Forces</h4>
      <label><span>Centre</span> <span id="center-val">0.01</span><input type="range" id="center-force" min="0" max="0.05" value="0.01" step="0.005"></label>
      <label><span>Repel</span> <span id="repel-val">200</span><input type="range" id="repel-force" min="50" max="600" value="200" step="10"></label>
      <label><span>Link pull</span> <span id="linkf-val">0.3</span><input type="range" id="link-force" min="0" max="1" value="0.3" step="0.05"></label>
      <label><span>Link dist</span> <span id="linkd-val">80</span><input type="range" id="link-dist" min="20" max="200" value="80" step="5"></label>
    </div>
  </div>
  <div id="note-popup">
    <span class="close-btn" onclick="document.getElementById('note-popup').style.display='none'">&times;</span>
    <div class="note-type" id="popup-type"></div>
    <h3 id="popup-title"></h3>
    <pre id="popup-content"></pre>
    <div id="popup-actions">
      <button id="popup-center">Center on this note</button>
      <button id="popup-close">Close</button>
    </div>
  </div>
  <canvas id="canvas"></canvas>
</div>

<script src="https://unpkg.com/d3@7/dist/d3.min.js"></script>
<script>
(async function() {
  const res = await fetch('/graph.json?v=' + Date.now());
  const data = await res.json();
  
  const typeColors = { atom: '#64ffda', article: '#ff6b6b', moc: '#ffd93d', source: '#6bcb77', system: '#9b59b6' };
  
  const container = document.getElementById('graph-canvas');
  const canvas = document.getElementById('canvas');
  const ctx = canvas.getContext('2d');
  
  let width, height;
  function resizeCanvas() {
    width = container.clientWidth;
    height = container.clientHeight;
    canvas.width = width * devicePixelRatio;
    canvas.height = height * devicePixelRatio;
    canvas.style.width = width + 'px';
    canvas.style.height = height + 'px';
    ctx.setTransform(devicePixelRatio, 0, 0, devicePixelRatio, 0, 0);
  }
  resizeCanvas();
  
  const nodeMap = {};
  const nodes = data.nodes.map(n => {
    const node = { id: n.id, label: n.id.replace(/-/g, ' '), type: n.type, path: n.path };
    nodeMap[n.id] = node;
    return node;
  });
  const links = data.edges.map(e => ({ source: e.source, target: e.target }));
  
  // Compute degrees
  nodes.forEach(n => n.degree = 0);
  data.edges.forEach(e => {
    if (nodeMap[e.source]) nodeMap[e.source].degree++;
    if (nodeMap[e.target]) nodeMap[e.target].degree++;
  });
  
  let settings = {
    nodeSize: 3, linkWidth: 1, labelSize: 10, labelFade: 1.2, labelMode: 'always', spacing: 1,
    centerForce: 0.01, repelForce: 200, linkForce: 0.3, linkDist: 80
  };
  
  let transform = d3.zoomIdentity;
  let hoveredNode = null;
  let centeredNodeId = null;
  
  const simulation = d3.forceSimulation(nodes)
    .force('link', d3.forceLink(links).id(d => d.id).distance(settings.linkDist).strength(settings.linkForce))
    .force('charge', d3.forceManyBody().strength(-settings.repelForce))
    .force('center', d3.forceCenter(width / 2, height / 2).strength(settings.centerForce))
    .force('collision', d3.forceCollide().radius(d => getNodeRadius(d) + 2))
    .alphaDecay(0.01)
    .on('tick', draw);
  
  function fitToView() {
    const drawable = nodes.filter(n => Number.isFinite(n.x) && Number.isFinite(n.y));
    if (drawable.length === 0) return;
    let minX = Infinity, maxX = -Infinity, minY = Infinity, maxY = -Infinity;
    drawable.forEach(n => { minX = Math.min(minX, n.x); maxX = Math.max(maxX, n.x); minY = Math.min(minY, n.y); maxY = Math.max(maxY, n.y); });
    const pad = 60;
    const dx = maxX - minX + pad * 2, dy = maxY - minY + pad * 2;
    const scale = Math.min(width / dx, height / dy, 2);
    const cx = (minX + maxX) / 2, cy = (minY + maxY) / 2;
    transform = d3.zoomIdentity.translate(width / 2 - cx * scale, height / 2 - cy * scale).scale(scale);
    d3.select(canvas).call(d3.zoom().transform, transform);
    draw();
  }
  
  // Auto-fit after the initial simulation settles so any node placed far from
  // the origin by the force layout gets pulled into view on page load.
  setTimeout(fitToView, 2500);
  
  function getNodeRadius(n) {
    return settings.nodeSize * (1 + Math.min((n.degree || 0) * 0.15, 1.5));
  }
  
  function getNeighborsWithDepth(nodeId, maxDepth) {
    const depthMap = new Map([[nodeId, 0]]);
    let frontier = new Set([nodeId]);
    for (let d = 1; d <= maxDepth; d++) {
      const next = new Set();
      links.forEach(l => {
        const sId = l.source.id || l.source;
        const tId = l.target.id || l.target;
        if (frontier.has(sId) && !depthMap.has(tId)) { next.add(tId); depthMap.set(tId, d); }
        if (frontier.has(tId) && !depthMap.has(sId)) { next.add(sId); depthMap.set(sId, d); }
      });
      frontier = next;
    }
    return depthMap;
  }
  
  function isDirectNeighbor(nodeId, targetId) {
    return links.some(l => {
      const sId = l.source.id || l.source;
      const tId = l.target.id || l.target;
      return (sId === nodeId && tId === targetId) || (tId === nodeId && sId === targetId);
    });
  }
  
  function draw() {
    ctx.save();
    ctx.clearRect(0, 0, width, height);
    ctx.translate(transform.x, transform.y);
    ctx.scale(transform.k, transform.k);
    
    const depth = parseInt(document.getElementById('depth-slider').value);
    const activeTypes = new Set([...document.querySelectorAll('.type-cb:checked')].map(cb => cb.value));
    
    let depthMap = null;
    if (centeredNodeId && depth < 5) {
      depthMap = getNeighborsWithDepth(centeredNodeId, depth);
    }
    
    function isVisible(n) {
      // Centered node is always visible regardless of type filter
      if (n.id === centeredNodeId) return true;
      if (!activeTypes.has(n.type)) return false;
      // When centered, only show nodes that are in the neighbourhood
      if (depthMap && !depthMap.has(n.id)) return false;
      return true;
    }
    
    function nodeDepth(n) {
      if (!depthMap) return 0;
      return depthMap.get(n.id) || 0;
    }
    
    const depthLinkColors = ['#64ffda', '#4a9e8e', '#3a6e6e', '#2a4e5e'];
    function linkColorForDepth(d) { return depthLinkColors[Math.min(d, depthLinkColors.length - 1)]; }
    
    function isHoverLit(n) {
      if (!hoveredNode) return false;
      return n.id === hoveredNode.id || isDirectNeighbor(hoveredNode.id, n.id);
    }
    
    // Links
    links.forEach(l => {
      const s = l.source, t = l.target;
      if (!isVisible(s) || !isVisible(t)) return;
      
      let alpha = 0.25;
      let lw = settings.linkWidth;
      let color = '#445';
      
      if (hoveredNode) {
        if (s.id === hoveredNode.id || t.id === hoveredNode.id) {
          alpha = 0.8; lw = settings.linkWidth * 2.5; color = '#64ffda';
        } else { alpha = 0.05; }
      } else if (depthMap) {
        const sD = nodeDepth(s), tD = nodeDepth(t);
        const minD = Math.min(sD, tD);
        color = linkColorForDepth(minD);
        alpha = minD === 0 ? 0.7 : minD === 1 ? 0.5 : 0.35;
        lw = minD === 0 ? settings.linkWidth * 2 : minD === 1 ? settings.linkWidth * 1.5 : settings.linkWidth;
      }
      
      ctx.beginPath();
      ctx.moveTo(s.x, s.y);
      ctx.lineTo(t.x, t.y);
      ctx.strokeStyle = color;
      ctx.globalAlpha = alpha;
      ctx.lineWidth = lw;
      ctx.stroke();
    });
    
    // Nodes
    nodes.forEach(n => {
      if (!isVisible(n)) return;
      const r = getNodeRadius(n);
      let alpha = 0.85;
      if (hoveredNode && !isHoverLit(n)) alpha = 0.1;
      else if (depthMap) {
        const d = nodeDepth(n);
        alpha = d === 0 ? 1 : d === 1 ? 0.9 : 0.6;
      }
      
      ctx.beginPath();
      ctx.arc(n.x, n.y, r, 0, Math.PI * 2);
      ctx.fillStyle = typeColors[n.type] || '#999';
      ctx.globalAlpha = alpha;
      ctx.fill();
      
      if (n.id === centeredNodeId || n === hoveredNode) {
        ctx.strokeStyle = '#fff';
        ctx.lineWidth = 2;
        ctx.globalAlpha = 1;
        ctx.stroke();
      }
    });
    
    // Labels
    if (settings.labelMode !== 'off') {
      const zoomShow = transform.k > settings.labelFade;
      const alwaysOn = settings.labelMode === 'always';
      
      nodes.forEach(n => {
        if (!isVisible(n)) return;
        
        const isCentered = !!centeredNodeId;
        let show = alwaysOn || zoomShow || n === hoveredNode || n.id === centeredNodeId || isHoverLit(n) || (isCentered && isVisible(n));
        if (!show) return;
        
        let alpha = 1;
        if (hoveredNode && !isHoverLit(n)) alpha = 0.06;
        else if (isCentered && !hoveredNode) alpha = 0.85;
        else if (alwaysOn && !hoveredNode && !isCentered) alpha = 0.65;
        
        const fs = (n === hoveredNode || n.id === centeredNodeId) ? settings.labelSize + 2 : settings.labelSize;
        ctx.font = `${fs}px sans-serif`;
        ctx.fillStyle = (n === hoveredNode || n.id === centeredNodeId) ? '#fff' : '#bbb';
        ctx.globalAlpha = alpha;
        ctx.textAlign = 'center';
        ctx.fillText(n.label, n.x, n.y + getNodeRadius(n) + fs + 2);
      });
    }
    
    ctx.restore();
    ctx.globalAlpha = 1;
  }
  
  // Hit test
  function nodeAt(mx, my) {
    const x = (mx - transform.x) / transform.k;
    const y = (my - transform.y) / transform.k;
    for (let i = nodes.length - 1; i >= 0; i--) {
      const n = nodes[i];
      if ((n.x - x) ** 2 + (n.y - y) ** 2 < (getNodeRadius(n) + 4) ** 2) return n;
    }
    return null;
  }
  
  // Drag behaviour (must be set up before zoom so it takes priority on nodes)
  let dragNode = null;
  const drag = d3.drag()
    .container(canvas)
    .subject(e => {
      const n = nodeAt(e.x, e.y);
      if (n) return { x: transform.applyX(n.x), y: transform.applyY(n.y), node: n };
      return null;
    })
    .on('start', e => {
      if (!e.subject) return;
      dragNode = e.subject.node;
      simulation.alphaTarget(0.3).restart();
      dragNode.fx = dragNode.x;
      dragNode.fy = dragNode.y;
    })
    .on('drag', e => {
      if (!dragNode) return;
      dragNode.fx = transform.invertX(e.x);
      dragNode.fy = transform.invertY(e.y);
    })
    .on('end', e => {
      if (!dragNode) return;
      simulation.alphaTarget(0);
      dragNode.fx = null;
      dragNode.fy = null;
      dragNode = null;
    });
  
  // Zoom (pans when not on a node, zooms with scroll)
  const zoom = d3.zoom()
    .scaleExtent([0.1, 6])
    .filter(e => {
      // Allow scroll zoom always. For pointer events, only pan if not on a node.
      if (e.type === 'wheel') return true;
      if (e.type === 'mousedown' || e.type === 'touchstart') {
        const rect = canvas.getBoundingClientRect();
        const mx = (e.touches ? e.touches[0].clientX : e.clientX) - rect.left;
        const my = (e.touches ? e.touches[0].clientY : e.clientY) - rect.top;
        return !nodeAt(mx, my);
      }
      return true;
    })
    .on('zoom', e => { transform = e.transform; draw(); });
  
  const sel = d3.select(canvas);
  sel.call(drag);
  sel.call(zoom);
  
  // Hover
  canvas.addEventListener('mousemove', e => {
    if (dragNode) return; // don't change hover during drag
    const rect = canvas.getBoundingClientRect();
    const n = nodeAt(e.clientX - rect.left, e.clientY - rect.top);
    if (n !== hoveredNode) { hoveredNode = n; canvas.style.cursor = n ? 'grab' : 'default'; draw(); }
  });
  canvas.addEventListener('mouseleave', () => { hoveredNode = null; draw(); });
  
  // Click: show popup (don't auto-center)
  canvas.addEventListener('click', async e => {
    const rect = canvas.getBoundingClientRect();
    const node = nodeAt(e.clientX - rect.left, e.clientY - rect.top);
    
    if (!node) { document.getElementById('note-popup').style.display = 'none'; return; }
    
    const popup = document.getElementById('note-popup');
    document.getElementById('popup-title').textContent = node.label;
    document.getElementById('popup-type').textContent = node.type;
    document.getElementById('popup-content').textContent = 'Loading...';
    popup.style.display = 'block';
    
    // Store clicked node for center button
    popup.dataset.nodeId = node.id;
    
    try {
      const initRes = await fetch('/mcp', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json', 'Accept': 'application/json, text/event-stream' },
        body: JSON.stringify({jsonrpc:'2.0',method:'initialize',params:{protocolVersion:'2025-03-26',capabilities:{},clientInfo:{name:'graph',version:'1.0'}},id:1})
      });
      const sessionId = initRes.headers.get('mcp-session-id');
      const noteRes = await fetch('/mcp', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json', 'Accept': 'application/json, text/event-stream', 'Mcp-Session-Id': sessionId },
        body: JSON.stringify({jsonrpc:'2.0',method:'tools/call',params:{name:'get_note',arguments:{path:node.path}},id:2})
      });
      const noteText = await noteRes.text();
      const noteData = JSON.parse(noteText.split('data: ')[1]);
      let content = noteData.result.content[0].text.replace(/^---[\s\S]*?---\n*/, '');
      document.getElementById('popup-content').textContent = content.trim();
    } catch(err) {
      document.getElementById('popup-content').textContent = 'Could not load note content.';
    }
  });
  
  // Popup: center on this note
  document.getElementById('popup-center').addEventListener('click', () => {
    const nodeId = document.getElementById('note-popup').dataset.nodeId;
    if (nodeId) {
      centeredNodeId = nodeId;
      document.getElementById('depth-slider').value = 2;
      document.getElementById('depth-val').textContent = '2';
      draw();
    }
  });
  
  // Popup: close
  document.getElementById('popup-close').addEventListener('click', () => {
    document.getElementById('note-popup').style.display = 'none';
  });
  
  
  // Slider bindings
  function bind(id, valId, key, fn) {
    document.getElementById(id).addEventListener('input', function() {
      const v = parseFloat(this.value);
      document.getElementById(valId).textContent = v;
      settings[key] = v;
      if (fn) fn(v);
      draw();
    });
  }
  bind('node-size', 'nsize-val', 'nodeSize');
  bind('link-width', 'lwidth-val', 'linkWidth');
  bind('label-size', 'lsize-val', 'labelSize');
  bind('label-fade', 'lfade-val', 'labelFade');
  bind('spacing', 'spacing-val', 'spacing', v => {
    simulation.force('charge').strength(-settings.repelForce * v);
    simulation.force('link').distance(settings.linkDist * v);
    simulation.force('collision').radius(d => (getNodeRadius(d) + 2) * v);
    simulation.alpha(0.5).restart();
    // Fit visible nodes to screen after simulation settles
    setTimeout(() => {
      const depth = parseInt(document.getElementById('depth-slider').value);
      const activeTypes = new Set([...document.querySelectorAll('.type-cb:checked')].map(cb => cb.value));
      const depthMap2 = (centeredNodeId && depth < 5) ? getNeighborsWithDepth(centeredNodeId, depth) : null;
      const visible = nodes.filter(n => {
        if (n.id === centeredNodeId) return true;
        if (!activeTypes.has(n.type)) return false;
        if (depthMap2 && !depthMap2.has(n.id)) return false;
        return true;
      });
      if (visible.length === 0 || visible.length === nodes.length) return;
      let minX = Infinity, maxX = -Infinity, minY = Infinity, maxY = -Infinity;
      visible.forEach(n => { minX = Math.min(minX, n.x); maxX = Math.max(maxX, n.x); minY = Math.min(minY, n.y); maxY = Math.max(maxY, n.y); });
      const pad = 60;
      const dx = maxX - minX + pad * 2, dy = maxY - minY + pad * 2;
      const scale = Math.min(width / dx, height / dy, 2);
      const cx = (minX + maxX) / 2, cy = (minY + maxY) / 2;
      transform = d3.zoomIdentity.translate(width / 2 - cx * scale, height / 2 - cy * scale).scale(scale);
      d3.select(canvas).call(d3.zoom().transform, transform);
      draw();
    }, 800);
  });
  bind('center-force', 'center-val', 'centerForce', v => { simulation.force('center').strength(v); simulation.alpha(0.3).restart(); });
  bind('repel-force', 'repel-val', 'repelForce', v => { simulation.force('charge').strength(-v); simulation.alpha(0.3).restart(); });
  bind('link-force', 'linkf-val', 'linkForce', v => { simulation.force('link').strength(v); simulation.alpha(0.3).restart(); });
  bind('link-dist', 'linkd-val', 'linkDist', v => { simulation.force('link').distance(v); simulation.alpha(0.3).restart(); });
  
  // Toggle panel
  document.getElementById('toggle-btn').addEventListener('click', () => {
    const panel = document.getElementById('controls-panel');
    panel.classList.toggle('collapsed');
    document.getElementById('toggle-btn').textContent = panel.classList.contains('collapsed') ? 'Controls ▶' : 'Controls ▼';
  });
  
  // Label mode
  document.getElementById('label-mode').addEventListener('change', function() { settings.labelMode = this.value; draw(); });
  
  // Depth
  document.getElementById('depth-slider').addEventListener('input', function() {
    document.getElementById('depth-val').textContent = this.value == 5 ? 'All' : this.value;
    draw();
  });
  
  // Type filter checkboxes
  document.querySelectorAll('.type-cb').forEach(cb => cb.addEventListener('change', () => draw()));
  document.getElementById('select-all').addEventListener('click', e => { e.preventDefault(); document.querySelectorAll('.type-cb').forEach(cb => cb.checked = true); draw(); });
  document.getElementById('select-none').addEventListener('click', e => { e.preventDefault(); document.querySelectorAll('.type-cb').forEach(cb => cb.checked = false); draw(); });
  
  // Reset
  document.getElementById('reset-btn').addEventListener('click', () => {
    centeredNodeId = null;
    document.getElementById('depth-slider').value = 5;
    document.getElementById('depth-val').textContent = 'All';
    document.querySelectorAll('.type-cb').forEach(cb => cb.checked = true);
    document.getElementById('note-popup').style.display = 'none';
    draw();
    simulation.alpha(0.1).restart();
  });
  
  // Resize
  window.addEventListener('resize', () => {
    resizeCanvas();
    simulation.force('center', d3.forceCenter(width / 2, height / 2).strength(settings.centerForce));
    simulation.alpha(0.3).restart();
  });
})();
</script>
