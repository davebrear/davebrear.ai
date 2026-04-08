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
  height: 600px;
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

<div class="legend">
  <div class="legend-item"><span class="legend-dot" style="background:#64ffda"></span> Atomic Note</div>
  <div class="legend-item"><span class="legend-dot" style="background:#ff6b6b"></span> Article</div>
  <div class="legend-item"><span class="legend-dot" style="background:#ffd93d"></span> Map of Content</div>
  <div class="legend-item"><span class="legend-dot" style="background:#6bcb77"></span> Source</div>
  <div class="legend-item"><span class="legend-dot" style="background:#9b59b6"></span> System</div>
</div>

<div id="graph-controls">
  <label>Center on: <select id="center-select"><option value="">-- select note --</option></select></label>
  <label>Depth: <input type="range" id="depth-slider" min="1" max="4" value="4" style="width:100px"><span id="depth-label">All</span></label>
  <label>Type: <select id="type-filter">
    <option value="all">All</option>
    <option value="atom">Atoms</option>
    <option value="article">Articles</option>
    <option value="moc">MOCs</option>
    <option value="source">Sources</option>
  </select></label>
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
          'label': 'data(label)',
          'font-size': '8px',
          'color': '#ccc',
          'text-valign': 'bottom',
          'text-margin-y': '5px',
          'width': function(ele) {
            return Math.max(15, Math.min(40, 10 + ele.degree() * 2));
          },
          'height': function(ele) {
            return Math.max(15, Math.min(40, 10 + ele.degree() * 2));
          },
          'background-color': function(ele) {
            return typeColors[ele.data('type')] || '#999';
          },
          'border-width': 0
        }
      },
      {
        selector: 'node:selected',
        style: {
          'border-width': 3,
          'border-color': '#fff'
        }
      },
      {
        selector: 'edge',
        style: {
          'width': 1,
          'line-color': '#334',
          'curve-style': 'bezier',
          'opacity': 0.4
        }
      },
      {
        selector: '.highlighted',
        style: {
          'border-width': 3,
          'border-color': '#fff',
          'opacity': 1
        }
      },
      {
        selector: '.highlighted-edge',
        style: {
          'line-color': '#64ffda',
          'opacity': 0.8,
          'width': 2
        }
      },
      {
        selector: '.dimmed',
        style: {
          'opacity': 0.1
        }
      }
    ],
    layout: {
      name: 'cose',
      idealEdgeLength: 80,
      nodeRepulsion: 8000,
      gravity: 0.3,
      animate: false,
      randomize: true
    },
    minZoom: 0.3,
    maxZoom: 3
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
  
  // Center on note with depth limit
  function centerOn(nodeId, maxDepth) {
    cy.elements().removeClass('highlighted highlighted-edge dimmed');
    
    if (!nodeId) {
      cy.elements().style('opacity', 1);
      return;
    }
    
    const root = cy.getElementById(nodeId);
    if (!root.length) return;
    
    const neighborhood = root.closedNeighborhood();
    let visible = neighborhood;
    
    for (let i = 1; i < maxDepth; i++) {
      visible = visible.closedNeighborhood();
    }
    
    cy.elements().addClass('dimmed');
    visible.removeClass('dimmed').addClass('highlighted');
    visible.edges().removeClass('dimmed').addClass('highlighted-edge');
    root.style('border-width', 4).style('border-color', '#fff');
    
    cy.animate({ fit: { eles: visible, padding: 40 } }, { duration: 500 });
  }
  
  select.addEventListener('change', function() {
    const depth = parseInt(document.getElementById('depth-slider').value);
    centerOn(this.value, depth);
  });
  
  const depthSlider = document.getElementById('depth-slider');
  const depthLabel = document.getElementById('depth-label');
  depthSlider.addEventListener('input', function() {
    depthLabel.textContent = this.value == 4 ? 'All' : this.value;
    const nodeId = select.value;
    if (nodeId) centerOn(nodeId, parseInt(this.value));
  });
  
  // Type filter
  document.getElementById('type-filter').addEventListener('change', function() {
    const type = this.value;
    if (type === 'all') {
      cy.elements().style('display', 'element');
    } else {
      cy.nodes().forEach(n => {
        n.style('display', n.data('type') === type ? 'element' : 'none');
      });
      cy.edges().forEach(e => {
        const s = e.source().data('type');
        const t = e.target().data('type');
        e.style('display', (s === type && t === type) ? 'element' : 'none');
      });
    }
  });
  
  // Click to show note content
  cy.on('tap', 'node', async function(evt) {
    const node = evt.target;
    const popup = document.getElementById('note-popup');
    const path = node.data('path');
    
    document.getElementById('popup-title').textContent = node.data('label');
    document.getElementById('popup-type').textContent = node.data('type');
    document.getElementById('popup-content').textContent = 'Loading...';
    popup.style.display = 'block';
    
    // Fetch from MCP
    try {
      // Initialize session
      const initRes = await fetch('/mcp', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json', 'Accept': 'application/json, text/event-stream' },
        body: JSON.stringify({jsonrpc:'2.0',method:'initialize',params:{protocolVersion:'2025-03-26',capabilities:{},clientInfo:{name:'graph',version:'1.0'}},id:1})
      });
      const initText = await initRes.text();
      const sessionId = initRes.headers.get('mcp-session-id');
      
      // Get note
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
      
      // Strip frontmatter
      content = content.replace(/^---[\s\S]*?---\n*/, '');
      
      document.getElementById('popup-content').textContent = content.trim();
    } catch(e) {
      document.getElementById('popup-content').textContent = 'Could not load note content.';
    }
  });
  
  // Click background to dismiss popup
  cy.on('tap', function(evt) {
    if (evt.target === cy) {
      document.getElementById('note-popup').style.display = 'none';
      cy.elements().removeClass('highlighted highlighted-edge dimmed');
    }
  });
})();
</script>
