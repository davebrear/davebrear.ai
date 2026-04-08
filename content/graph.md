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
#graph-canvas {
  width: 100%;
  height: 750px;
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
  width: 220px;
  max-height: 700px;
  overflow-y: auto;
}
#controls-panel h4 {
  margin: 0 0 8px 0;
  color: #64ffda;
  font-size: 0.95em;
  cursor: pointer;
}
#controls-panel label {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin: 6px 0;
}
#controls-panel input[type="range"] {
  width: 100px;
  accent-color: #64ffda;
}
#controls-panel select {
  width: 100%;
  padding: 3px 6px;
  background: #2a2a3e;
  color: #e0e0e0;
  border: 1px solid #555;
  border-radius: 4px;
  margin: 4px 0;
}
.control-section {
  border-top: 1px solid #333;
  padding-top: 8px;
  margin-top: 8px;
}
.control-section:first-child {
  border-top: none;
  padding-top: 0;
  margin-top: 0;
}
#note-popup {
  display: none;
  position: absolute;
  top: 10px;
  right: 10px;
  width: 350px;
  max-height: 500px;
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
#note-popup .close-btn { float: right; cursor: pointer; font-size: 1.2em; color: #999; }
#note-popup .close-btn:hover { color: #fff; }
#note-popup .note-type { font-size: 0.75em; text-transform: uppercase; color: #888; margin-bottom: 0.5em; }
#note-popup pre { white-space: pre-wrap; word-wrap: break-word; font-size: 0.85em; background: #0a0a1a; padding: 0.8em; border-radius: 4px; max-height: 300px; overflow-y: auto; }
.legend {
  display: flex; gap: 1rem; flex-wrap: wrap; font-size: 0.8em; margin-bottom: 0.5em;
}
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
    <div class="control-section">
      <h4>Filters</h4>
      <label>Center on: <span></span></label>
      <select id="center-select"><option value="">-- all --</option></select>
      <label>Type: <span></span></label>
      <select id="type-filter">
        <option value="all">All</option>
        <option value="atom">Atoms</option>
        <option value="article">Articles</option>
        <option value="moc">MOCs</option>
        <option value="source">Sources</option>
      </select>
      <label>Depth <span id="depth-val">All</span><input type="range" id="depth-slider" min="1" max="5" value="5"></label>
    </div>
    <div class="control-section">
      <h4>Display</h4>
      <label>Node size <span id="nsize-val">4</span><input type="range" id="node-size" min="1" max="10" value="4"></label>
      <label>Link width <span id="lwidth-val">1</span><input type="range" id="link-width" min="1" max="5" value="1" step="0.5"></label>
      <label>Label size <span id="lsize-val">10</span><input type="range" id="label-size" min="0" max="16" value="10"></label>
      <label>Label fade <span id="lfade-val">1.2</span><input type="range" id="label-fade" min="0.3" max="3" value="1.2" step="0.1"></label>
    </div>
    <div class="control-section">
      <h4>Forces</h4>
      <label>Centre <span id="center-val">0.01</span><input type="range" id="center-force" min="0" max="0.05" value="0.01" step="0.005"></label>
      <label>Repel <span id="repel-val">200</span><input type="range" id="repel-force" min="50" max="600" value="200" step="10"></label>
      <label>Link force <span id="linkf-val">0.3</span><input type="range" id="link-force" min="0" max="1" value="0.3" step="0.05"></label>
      <label>Link dist <span id="linkd-val">80</span><input type="range" id="link-dist" min="20" max="200" value="80" step="5"></label>
    </div>
  </div>
  <div id="note-popup">
    <span class="close-btn" onclick="document.getElementById('note-popup').style.display='none'">&times;</span>
    <div class="note-type" id="popup-type"></div>
    <h3 id="popup-title"></h3>
    <pre id="popup-content"></pre>
  </div>
  <canvas id="canvas"></canvas>
</div>

<script src="https://unpkg.com/d3@7/dist/d3.min.js"></script>
<script>
(async function() {
  const res = await fetch('/graph.json');
  const data = await res.json();
  
  const typeColors = { atom: '#64ffda', article: '#ff6b6b', moc: '#ffd93d', source: '#6bcb77', system: '#9b59b6' };
  
  const container = document.getElementById('graph-canvas');
  const canvas = document.getElementById('canvas');
  const ctx = canvas.getContext('2d');
  
  let width = container.clientWidth;
  let height = container.clientHeight;
  canvas.width = width * devicePixelRatio;
  canvas.height = height * devicePixelRatio;
  canvas.style.width = width + 'px';
  canvas.style.height = height + 'px';
  ctx.scale(devicePixelRatio, devicePixelRatio);
  
  // Build nodes and links
  const nodeMap = {};
  const nodes = data.nodes.map(n => {
    const node = { id: n.id, label: n.id.replace(/-/g, ' '), type: n.type, path: n.path, visible: true };
    nodeMap[n.id] = node;
    return node;
  });
  const links = data.edges.map(e => ({ source: e.source, target: e.target }));
  
  // Populate dropdown
  const select = document.getElementById('center-select');
  nodes.slice().sort((a,b) => a.label.localeCompare(b.label)).forEach(n => {
    const opt = document.createElement('option');
    opt.value = n.id;
    opt.textContent = n.label;
    select.appendChild(opt);
  });
  
  // Settings
  let settings = {
    nodeSize: 4, linkWidth: 1, labelSize: 10, labelFade: 1.2,
    centerForce: 0.01, repelForce: 200, linkForce: 0.3, linkDist: 80
  };
  
  // D3 force simulation
  const simulation = d3.forceSimulation(nodes)
    .force('link', d3.forceLink(links).id(d => d.id).distance(settings.linkDist).strength(settings.linkForce))
    .force('charge', d3.forceManyBody().strength(-settings.repelForce))
    .force('center', d3.forceCenter(width / 2, height / 2).strength(settings.centerForce))
    .force('collision', d3.forceCollide().radius(d => settings.nodeSize * (1 + d.degree * 0.3) + 2))
    .alphaDecay(0.01)
    .on('tick', draw);
  
  // Compute degrees
  nodes.forEach(n => n.degree = 0);
  links.forEach(l => {
    const s = typeof l.source === 'string' ? nodeMap[l.source] : l.source;
    const t = typeof l.target === 'string' ? nodeMap[l.target] : l.target;
    if (s) s.degree = (s.degree || 0) + 1;
    if (t) t.degree = (t.degree || 0) + 1;
  });
  
  // Transform state
  let transform = d3.zoomIdentity;
  
  // Hover / selection state
  let hoveredNode = null;
  let focusedNode = null;
  let visibleNodes = null;
  
  function getNodeRadius(n) {
    return settings.nodeSize * (1 + (n.degree || 0) * 0.25);
  }
  
  function getNeighbors(nodeId, depth) {
    const visited = new Set([nodeId]);
    let frontier = new Set([nodeId]);
    for (let i = 0; i < depth; i++) {
      const next = new Set();
      links.forEach(l => {
        const sId = typeof l.source === 'string' ? l.source : l.source.id;
        const tId = typeof l.target === 'string' ? l.target : l.target.id;
        if (frontier.has(sId) && !visited.has(tId)) { next.add(tId); visited.add(tId); }
        if (frontier.has(tId) && !visited.has(sId)) { next.add(sId); visited.add(sId); }
      });
      frontier = next;
    }
    return visited;
  }
  
  function draw() {
    ctx.save();
    ctx.clearRect(0, 0, width, height);
    ctx.translate(transform.x, transform.y);
    ctx.scale(transform.k, transform.k);
    
    const depth = parseInt(document.getElementById('depth-slider').value);
    const centerNodeId = select.value;
    const typeFilter = document.getElementById('type-filter').value;
    
    // Determine visible nodes
    let activeNodes;
    if (centerNodeId && depth < 5) {
      activeNodes = getNeighbors(centerNodeId, depth);
    } else {
      activeNodes = null; // all visible
    }
    
    function isNodeVisible(n) {
      if (typeFilter !== 'all' && n.type !== typeFilter) return false;
      if (activeNodes && !activeNodes.has(n.id)) return false;
      return true;
    }
    
    function isHoverHighlighted(n) {
      if (!hoveredNode) return false;
      if (n.id === hoveredNode.id) return true;
      return links.some(l => {
        const sId = typeof l.source === 'string' ? l.source : l.source.id;
        const tId = typeof l.target === 'string' ? l.target : l.target.id;
        return (sId === hoveredNode.id && tId === n.id) || (tId === hoveredNode.id && sId === n.id);
      });
    }
    
    // Draw links
    links.forEach(l => {
      const s = typeof l.source === 'string' ? nodeMap[l.source] : l.source;
      const t = typeof l.target === 'string' ? nodeMap[l.target] : l.target;
      if (!s || !t) return;
      if (!isNodeVisible(s) || !isNodeVisible(t)) return;
      
      let alpha = 0.15;
      let lw = settings.linkWidth;
      let color = '#334';
      
      if (hoveredNode) {
        const sId = s.id, tId = t.id;
        if (sId === hoveredNode.id || tId === hoveredNode.id) {
          alpha = 0.8; lw = settings.linkWidth * 2; color = '#64ffda';
        } else {
          alpha = 0.04;
        }
      }
      
      if (centerNodeId && s.id === centerNodeId || t.id === centerNodeId) {
        alpha = Math.max(alpha, 0.5);
        color = '#64ffda';
        lw = Math.max(lw, settings.linkWidth * 1.5);
      }
      
      ctx.beginPath();
      ctx.moveTo(s.x, s.y);
      ctx.lineTo(t.x, t.y);
      ctx.strokeStyle = color;
      ctx.globalAlpha = alpha;
      ctx.lineWidth = lw;
      ctx.stroke();
    });
    
    // Draw nodes
    nodes.forEach(n => {
      if (!isNodeVisible(n)) return;
      
      const r = getNodeRadius(n);
      let alpha = 0.85;
      
      if (hoveredNode && !isHoverHighlighted(n)) {
        alpha = 0.1;
      }
      
      ctx.beginPath();
      ctx.arc(n.x, n.y, r, 0, Math.PI * 2);
      ctx.fillStyle = typeColors[n.type] || '#999';
      ctx.globalAlpha = alpha;
      ctx.fill();
      
      // Border for center/hovered node
      if (n.id === centerNodeId || n === hoveredNode) {
        ctx.strokeStyle = '#fff';
        ctx.lineWidth = 2;
        ctx.globalAlpha = 1;
        ctx.stroke();
      }
    });
    
    // Draw labels
    const labelSize = settings.labelSize * transform.k;
    if (labelSize > 0) {
      const fadeThreshold = settings.labelFade;
      const showAll = transform.k > fadeThreshold;
      
      nodes.forEach(n => {
        if (!isNodeVisible(n)) return;
        
        let shouldShow = showAll || n === hoveredNode || n.id === centerNodeId || isHoverHighlighted(n);
        if (!shouldShow) return;
        
        let alpha = 1;
        if (hoveredNode && !isHoverHighlighted(n)) alpha = 0.08;
        if (!showAll && isHoverHighlighted(n) && n !== hoveredNode) alpha = 0.9;
        
        const fontSize = n === hoveredNode || n.id === centerNodeId ? settings.labelSize + 2 : settings.labelSize;
        ctx.font = `${fontSize}px sans-serif`;
        ctx.fillStyle = n === hoveredNode || n.id === centerNodeId ? '#fff' : '#ccc';
        ctx.globalAlpha = alpha;
        ctx.textAlign = 'center';
        ctx.fillText(n.label, n.x, n.y + getNodeRadius(n) + fontSize + 2);
      });
    }
    
    ctx.restore();
    ctx.globalAlpha = 1;
  }
  
  // Zoom and pan
  const zoom = d3.zoom()
    .scaleExtent([0.1, 6])
    .on('zoom', (event) => {
      transform = event.transform;
      draw();
    });
  
  d3.select(canvas).call(zoom);
  
  // Mouse interaction
  function getNodeAt(mx, my) {
    const x = (mx - transform.x) / transform.k;
    const y = (my - transform.y) / transform.k;
    for (let i = nodes.length - 1; i >= 0; i--) {
      const n = nodes[i];
      const r = getNodeRadius(n) + 3;
      if ((n.x - x) ** 2 + (n.y - y) ** 2 < r ** 2) return n;
    }
    return null;
  }
  
  canvas.addEventListener('mousemove', (e) => {
    const rect = canvas.getBoundingClientRect();
    const node = getNodeAt(e.clientX - rect.left, e.clientY - rect.top);
    if (node !== hoveredNode) {
      hoveredNode = node;
      canvas.style.cursor = node ? 'pointer' : 'default';
      draw();
    }
  });
  
  canvas.addEventListener('mouseleave', () => {
    hoveredNode = null;
    draw();
  });
  
  // Click to show popup
  canvas.addEventListener('click', async (e) => {
    const rect = canvas.getBoundingClientRect();
    const node = getNodeAt(e.clientX - rect.left, e.clientY - rect.top);
    
    if (!node) {
      document.getElementById('note-popup').style.display = 'none';
      return;
    }
    
    const popup = document.getElementById('note-popup');
    document.getElementById('popup-title').textContent = node.label;
    document.getElementById('popup-type').textContent = node.type;
    document.getElementById('popup-content').textContent = 'Loading...';
    popup.style.display = 'block';
    
    // Also center on this node
    select.value = node.id;
    draw();
    
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
      let content = noteData.result.content[0].text;
      content = content.replace(/^---[\s\S]*?---\n*/, '');
      document.getElementById('popup-content').textContent = content.trim();
    } catch(err) {
      document.getElementById('popup-content').textContent = 'Could not load note content.';
    }
  });
  
  // Drag nodes
  let dragNode = null;
  canvas.addEventListener('mousedown', (e) => {
    const rect = canvas.getBoundingClientRect();
    dragNode = getNodeAt(e.clientX - rect.left, e.clientY - rect.top);
    if (dragNode) {
      simulation.alphaTarget(0.3).restart();
      dragNode.fx = dragNode.x;
      dragNode.fy = dragNode.y;
    }
  });
  
  canvas.addEventListener('mousemove', (e) => {
    if (!dragNode) return;
    const rect = canvas.getBoundingClientRect();
    dragNode.fx = (e.clientX - rect.left - transform.x) / transform.k;
    dragNode.fy = (e.clientY - rect.top - transform.y) / transform.k;
  });
  
  canvas.addEventListener('mouseup', () => {
    if (dragNode) {
      simulation.alphaTarget(0);
      dragNode.fx = null;
      dragNode.fy = null;
      dragNode = null;
    }
  });
  
  // Control bindings
  function bindSlider(id, valId, key, updateFn) {
    const el = document.getElementById(id);
    const val = document.getElementById(valId);
    el.addEventListener('input', () => {
      const v = parseFloat(el.value);
      val.textContent = v;
      settings[key] = v;
      if (updateFn) updateFn(v);
      draw();
    });
  }
  
  bindSlider('node-size', 'nsize-val', 'nodeSize');
  bindSlider('link-width', 'lwidth-val', 'linkWidth');
  bindSlider('label-size', 'lsize-val', 'labelSize');
  bindSlider('label-fade', 'lfade-val', 'labelFade');
  
  bindSlider('center-force', 'center-val', 'centerForce', v => {
    simulation.force('center').strength(v);
    simulation.alpha(0.3).restart();
  });
  bindSlider('repel-force', 'repel-val', 'repelForce', v => {
    simulation.force('charge').strength(-v);
    simulation.alpha(0.3).restart();
  });
  bindSlider('link-force', 'linkf-val', 'linkForce', v => {
    simulation.force('link').strength(v);
    simulation.alpha(0.3).restart();
  });
  bindSlider('link-dist', 'linkd-val', 'linkDist', v => {
    simulation.force('link').distance(v);
    simulation.alpha(0.3).restart();
  });
  
  // Center-on and depth
  select.addEventListener('change', () => { simulation.alpha(0.1).restart(); draw(); });
  document.getElementById('depth-slider').addEventListener('input', function() {
    document.getElementById('depth-val').textContent = this.value == 5 ? 'All' : this.value;
    draw();
  });
  document.getElementById('type-filter').addEventListener('change', () => draw());
  
  // Handle resize
  window.addEventListener('resize', () => {
    width = container.clientWidth;
    height = container.clientHeight;
    canvas.width = width * devicePixelRatio;
    canvas.height = height * devicePixelRatio;
    canvas.style.width = width + 'px';
    canvas.style.height = height + 'px';
    ctx.setTransform(devicePixelRatio, 0, 0, devicePixelRatio, 0, 0);
    simulation.force('center', d3.forceCenter(width / 2, height / 2).strength(settings.centerForce));
    simulation.alpha(0.3).restart();
  });
})();
</script>
