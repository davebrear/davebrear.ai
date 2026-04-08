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
#graph-container {
  width: 100%;
  height: 700px;
  border: 1px solid #444;
  border-radius: 8px;
  margin: 1em 0;
  position: relative;
  background: #1a1a2e;
}
#graph-controls {
  display: flex;
  gap: 1rem;
  align-items: center;
  flex-wrap: wrap;
  margin-bottom: 1em;
}
#graph-controls label {
  font-size: 0.9em;
}
#graph-controls select, #graph-controls input {
  padding: 4px 8px;
  border-radius: 4px;
  border: 1px solid #666;
  background: #2a2a3e;
  color: #e0e0e0;
}
#graph-controls button {
  padding: 4px 12px;
  border-radius: 4px;
  border: 1px solid #666;
  background: #2a2a3e;
  color: #e0e0e0;
  cursor: pointer;
}
#graph-controls button:hover {
  background: #3a3a5e;
}
#note-popup {
  display: none;
  position: absolute;
  top: 10px;
  right: 10px;
  width: 350px;
  max-height: 500px;
  overflow-y: auto;
  background: #16213e;
  border: 1px solid #555;
  border-radius: 8px;
  padding: 1em;
  font-size: 0.85em;
  line-height: 1.5;
  z-index: 10;
  color: #e0e0e0;
}
#note-popup h3 {
  margin-top: 0;
  color: #64ffda;
}
#note-popup .close-btn {
  float: right;
  cursor: pointer;
  font-size: 1.2em;
  color: #999;
}
#note-popup .close-btn:hover {
  color: #fff;
}
#note-popup .note-type {
  font-size: 0.75em;
  text-transform: uppercase;
  color: #888;
  margin-bottom: 0.5em;
}
#note-popup pre {
  white-space: pre-wrap;
  word-wrap: break-word;
  font-size: 0.85em;
  background: #0a0a1a;
  padding: 0.8em;
  border-radius: 4px;
  max-height: 300px;
  overflow-y: auto;
}
.legend {
  display: flex;
  gap: 1rem;
  flex-wrap: wrap;
  font-size: 0.8em;
  margin-bottom: 0.5em;
}
.legend-item {
  display: flex;
  align-items: center;
  gap: 4px;
}
.legend-dot {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  display: inline-block;
}
</style>

Hover over a node to see its connections. Click to read the note. Use the controls to explore.

<div class="legend">
  <div class="legend-item"><span class="legend-dot" style="background:#64ffda"></span> Atomic Note</div>
  <div class="legend-item"><span class="legend-dot" style="background:#ff6b6b"></span> Article</div>
  <div class="legend-item"><span class="legend-dot" style="background:#ffd93d"></span> Map of Content</div>
  <div class="legend-item"><span class="legend-dot" style="background:#6bcb77"></span> Source</div>
  <div class="legend-item"><span class="legend-dot" style="background:#9b59b6"></span> System</div>
</div>

<div id="graph-controls">
  <label>Center on: <select id="center-select"><option value="">-- all notes --</option></select></label>
  <label>Depth: <input type="range" id="depth-slider" min="1" max="3" value="3" style="width:100px"><span id="depth-label">3</span></label>
  <label>Type: <select id="type-filter">
    <option value="all">All</option>
    <option value="atom">Atoms</option>
    <option value="article">Articles</option>
    <option value="moc">MOCs</option>
    <option value="source">Sources</option>
  </select></label>
  <button id="reset-btn">Reset</button>
</div>

<div id="graph-container">
  <div id="note-popup">
    <span class="close-btn" onclick="document.getElementById('note-popup').style.display='none'">&times;</span>
    <div class="note-type" id="popup-type"></div>
    <h3 id="popup-title"></h3>
    <pre id="popup-content"></pre>
  </div>
</div>

<script src="https://unpkg.com/cytoscape@3.28.1/dist/cytoscape.min.js"></script>
<script>
(async function() {
  const res = await fetch('/graph.json');
  const data = await res.json();
  
  const typeColors = {
    atom: '#64ffda',
    article: '#ff6b6b',
    moc: '#ffd93d',
    source: '#6bcb77',
    system: '#9b59b6'
  };
  
  const elements = [];
  
  data.nodes.forEach(n => {
    elements.push({
      data: { 
        id: n.id, 
        label: n.id.replace(/-/g, ' '),
        type: n.type,
        path: n.path
      }
    });
  });
  
  data.edges.forEach(e => {
    elements.push({
      data: { source: e.source, target: e.target }
    });
  });
  
  const cy = cytoscape({
    container: document.getElementById('graph-container'),
    elements: elements,
    style: [
      {
        selector: 'node',
        style: {
          'label': '',
          'width': function(ele) {
            return Math.max(12, Math.min(35, 8 + ele.degree() * 1.5));
          },
          'height': function(ele) {
            return Math.max(12, Math.min(35, 8 + ele.degree() * 1.5));
          },
          'background-color': function(ele) {
            return typeColors[ele.data('type')] || '#999';
          },
          'border-width': 0,
          'opacity': 0.85
        }
      },
      {
        selector: 'edge',
        style: {
          'width': 0.75,
          'line-color': '#2a2a4e',
          'curve-style': 'bezier',
          'opacity': 0.3
        }
      }
    ],
    layout: {
      name: 'cose',
      idealEdgeLength: 140,
      nodeRepulsion: 20000,
      nodeOverlap: 30,
      gravity: 0.15,
      numIter: 1000,
      animate: false,
      randomize: true
    },
    minZoom: 0.2,
    maxZoom: 4
  });
  
  // Populate center-on dropdown
  const select = document.getElementById('center-select');
  const sorted = data.nodes
    .map(n => ({id: n.id, label: n.id.replace(/-/g, ' '), type: n.type}))
    .sort((a,b) => a.label.localeCompare(b.label));
  
  sorted.forEach(n => {
    const opt = document.createElement('option');
    opt.value = n.id;
    opt.textContent = n.label;
    select.appendChild(opt);
  });

  // Get nodes within N hops
  function getNeighborhood(root, depth) {
    let visible = root.closedNeighborhood();
    for (let i = 1; i < depth; i++) {
      visible = visible.closedNeighborhood();
    }
    return visible;
  }

  // Apply focus: dim everything, highlight neighborhood
  function applyFocus(nodeId, depth) {
    resetStyles();
    if (!nodeId) return;
    
    const root = cy.getElementById(nodeId);
    if (!root.length) return;
    
    const visible = getNeighborhood(root, depth);
    
    // Dim everything
    cy.elements().style('opacity', 0.06);
    
    // Highlight visible
    visible.nodes().style({
      'opacity': 1,
      'label': 'data(label)',
      'font-size': '9px',
      'color': '#ddd',
      'text-valign': 'bottom',
      'text-margin-y': '6px',
      'text-max-width': '100px',
      'text-wrap': 'ellipsis'
    });
    visible.edges().style({ 'opacity': 0.6, 'line-color': '#445', 'width': 1.5 });
    
    // Highlight direct connections stronger
    const direct = root.closedNeighborhood();
    direct.edges().style({ 'line-color': '#64ffda', 'opacity': 0.8, 'width': 2 });
    
    // Highlight root
    root.style({ 
      'border-width': 3, 
      'border-color': '#fff',
      'label': 'data(label)',
      'font-size': '11px',
      'color': '#fff',
      'text-valign': 'bottom',
      'text-margin-y': '8px'
    });
    
    cy.animate({ fit: { eles: visible, padding: 50 } }, { duration: 400 });
  }
  
  function resetStyles() {
    cy.elements().removeStyle();
    cy.nodes().style('label', '');
    document.getElementById('note-popup').style.display = 'none';
  }
  
  // Center-on dropdown
  select.addEventListener('change', function() {
    const depth = parseInt(document.getElementById('depth-slider').value);
    applyFocus(this.value, depth);
  });
  
  // Depth slider
  const depthSlider = document.getElementById('depth-slider');
  const depthLabel = document.getElementById('depth-label');
  depthSlider.addEventListener('input', function() {
    depthLabel.textContent = this.value;
    const nodeId = select.value;
    if (nodeId) applyFocus(nodeId, parseInt(this.value));
  });
  
  // Type filter
  document.getElementById('type-filter').addEventListener('change', function() {
    const type = this.value;
    resetStyles();
    select.value = '';
    if (type === 'all') {
      cy.elements().style('display', 'element');
    } else {
      cy.nodes().forEach(n => {
        if (n.data('type') === type) {
          n.style({ 'display': 'element', 'label': 'data(label)', 'font-size': '9px', 'color': '#ddd', 'text-valign': 'bottom', 'text-margin-y': '6px' });
        } else {
          n.style('display', 'none');
        }
      });
      cy.edges().forEach(e => {
        const sVis = e.source().style('display') !== 'none';
        const tVis = e.target().style('display') !== 'none';
        e.style('display', (sVis && tVis) ? 'element' : 'none');
      });
    }
  });
  
  // Reset button
  document.getElementById('reset-btn').addEventListener('click', function() {
    select.value = '';
    depthSlider.value = 3;
    depthLabel.textContent = '3';
    document.getElementById('type-filter').value = 'all';
    resetStyles();
    cy.elements().style('display', 'element');
    cy.fit(undefined, 50);
  });
  
  // Hover: show label and highlight neighborhood
  cy.on('mouseover', 'node', function(evt) {
    const node = evt.target;
    // Only show hover labels if we're not in focus mode
    if (!select.value) {
      const neighborhood = node.closedNeighborhood();
      cy.elements().style('opacity', 0.12);
      neighborhood.nodes().style({ 
        'opacity': 1, 
        'label': 'data(label)', 
        'font-size': '9px', 
        'color': '#ddd',
        'text-valign': 'bottom',
        'text-margin-y': '6px'
      });
      neighborhood.edges().style({ 'opacity': 0.7, 'line-color': '#64ffda', 'width': 1.5 });
      node.style({ 'font-size': '11px', 'color': '#fff', 'border-width': 2, 'border-color': '#fff' });
    }
  });
  
  cy.on('mouseout', 'node', function() {
    if (!select.value) {
      resetStyles();
    }
  });
  
  // Click to show note content
  cy.on('tap', 'node', async function(evt) {
    const node = evt.target;
    const popup = document.getElementById('note-popup');
    const path = node.data('path');
    
    // Also focus on this node
    select.value = node.data('id');
    const depth = parseInt(depthSlider.value);
    applyFocus(node.data('id'), depth);
    
    document.getElementById('popup-title').textContent = node.data('label');
    document.getElementById('popup-type').textContent = node.data('type');
    document.getElementById('popup-content').textContent = 'Loading...';
    popup.style.display = 'block';
    
    try {
      const initRes = await fetch('/mcp', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json', 'Accept': 'application/json, text/event-stream' },
        body: JSON.stringify({jsonrpc:'2.0',method:'initialize',params:{protocolVersion:'2025-03-26',capabilities:{},clientInfo:{name:'graph',version:'1.0'}},id:1})
      });
      const sessionId = initRes.headers.get('mcp-session-id');
      
      const noteRes = await fetch('/mcp', {
        method: 'POST',
        headers: { 
          'Content-Type': 'application/json', 
          'Accept': 'application/json, text/event-stream',
          'Mcp-Session-Id': sessionId
        },
        body: JSON.stringify({jsonrpc:'2.0',method:'tools/call',params:{name:'get_note',arguments:{path:path}},id:2})
      });
      const noteText = await noteRes.text();
      const noteData = JSON.parse(noteText.split('data: ')[1]);
      let content = noteData.result.content[0].text;
      content = content.replace(/^---[\s\S]*?---\n*/, '');
      document.getElementById('popup-content').textContent = content.trim();
    } catch(e) {
      document.getElementById('popup-content').textContent = 'Could not load note content.';
    }
  });
  
  // Click background to reset
  cy.on('tap', function(evt) {
    if (evt.target === cy) {
      select.value = '';
      resetStyles();
      document.getElementById('note-popup').style.display = 'none';
    }
  });
})();
</script>
