# Quick Start: Python HTTP Server (Port 8080) with JavaScript & Python

This guide shows how to set up and run a lightweight Python HTTP server that serves BMH web interfaces built with JavaScript and Python backend integration.

## Table of Contents

- [Quick Start](#quick-start)
- [Directory Structure](#directory-structure)
- [Python HTTP Server Setup](#python-http-server-setup)
- [JavaScript Frontend](#javascript-frontend)
- [Python Backend API](#python-backend-api)
- [Running the Server](#running-the-server)
- [Development Workflow](#development-workflow)

---

## Quick Start

```bash
# 1. Navigate to repository
cd BMH

# 2. Create web directory structure
mkdir -p web/{frontend,backend,static,templates}

# 3. Install Python dependencies (optional)
pip install flask fastapi uvicorn requests

# 4. Start Python HTTP server on port 8080
python3 -m http.server 8080 --directory ./web/static

# 5. Access in browser
# http://localhost:8080
```

---

## Directory Structure

```
BMH/
├── web/
│   ├── frontend/              # JavaScript files
│   │   ├── index.html
│   │   ├── app.js
│   │   ├── components/
│   │   │   ├── messages.js
│   │   │   ├── transmitters.js
│   │   │   └── dashboard.js
│   │   └── css/
│   │       └── style.css
│   ├── backend/               # Python backend
│   │   ├── server.py          # Main Flask/FastAPI server
│   │   ├── api.py             # API endpoints
│   │   ├── models.py          # Data models
│   │   └── handlers.py        # Business logic
│   ├── static/                # Served by http.server
│   │   ├── index.html
│   │   ├── app.js
│   │   └── style.css
│   └── templates/             # HTML templates (optional)
│       └── base.html
```

---

## Python HTTP Server Setup

### 1. Basic HTTP Server (Static Files Only)

The simplest approach using Python's built-in `http.server`:

```bash
# Serve current directory on port 8080
python3 -m http.server 8080

# Serve specific directory
python3 -m http.server 8080 --directory ./web/static

# Serve on different port
python3 -m http.server 9000

# Bind to specific interface
python3 -m http.server 8080 --bind 127.0.0.1
```

**Access via browser:**
```
http://localhost:8080
http://localhost:8080/index.html
http://localhost:8080/app.js
```

### 2. Flask HTTP Server (Dynamic Backend)

More powerful option with Python backend API.

**File: `web/backend/server.py`**

```python
#!/usr/bin/env python3
"""
BMH Web Server - Flask-based HTTP server on port 8080
Serves static frontend files and provides REST API backend
"""

import os
import json
import logging
from pathlib import Path
from datetime import datetime

from flask import Flask, jsonify, request, send_from_directory
from flask_cors import CORS

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# Initialize Flask app
app = Flask(__name__, 
            static_folder='../static',
            template_folder='../templates')

# Enable CORS for cross-origin requests
CORS(app)

# Configuration
app.config['JSON_SORT_KEYS'] = False
WEB_ROOT = Path(__file__).parent.parent

# ============================================================================
# STATIC FILE SERVING
# ============================================================================

@app.route('/')
def index():
    """Serve index.html"""
    return send_from_directory(app.static_folder, 'index.html')

@app.route('/<path:filename>')
def serve_static(filename):
    """Serve static files (JS, CSS, etc.)"""
    return send_from_directory(app.static_folder, filename)

# ============================================================================
# REST API ENDPOINTS
# ============================================================================

@app.route('/api/status', methods=['GET'])
def get_status():
    """Get BMH system status"""
    return jsonify({
        'status': 'online',
        'timestamp': datetime.utcnow().isoformat(),
        'version': '1.0.0',
        'service': 'BMH HTTP Server',
        'port': 8080
    })

@app.route('/api/messages', methods=['GET'])
def get_messages():
    """Get all broadcast messages"""
    messages = [
        {
            'id': 1,
            'afosid': 'WCXO53',
            'text': 'Severe Thunderstorm Warning',
            'status': 'ACTIVE',
            'created': '2026-10-03T10:00:00Z',
            'expires': '2026-10-03T12:00:00Z',
            'transmitters': [101, 102, 103]
        },
        {
            'id': 2,
            'afosid': 'WCXO51',
            'text': 'Winter Weather Advisory',
            'status': 'ACTIVE',
            'created': '2026-10-03T09:30:00Z',
            'expires': '2026-10-03T15:00:00Z',
            'transmitters': [101, 104]
        }
    ]
    logger.info(f"Retrieved {len(messages)} messages")
    return jsonify(messages)

@app.route('/api/messages/<int:msg_id>', methods=['GET'])
def get_message(msg_id):
    """Get specific message by ID"""
    # Mock data
    message = {
        'id': msg_id,
        'afosid': f'WC{msg_id}O53',
        'text': f'Weather Message {msg_id}',
        'status': 'ACTIVE',
        'created': datetime.utcnow().isoformat(),
        'transmitters': [101, 102]
    }
    return jsonify(message)

@app.route('/api/messages', methods=['POST'])
def create_message():
    """Create new broadcast message"""
    data = request.get_json()
    
    # Validate input
    if not data or 'text' not in data:
        return jsonify({'error': 'Missing required field: text'}), 400
    
    new_message = {
        'id': 999,
        'afosid': data.get('afosid', 'NEW'),
        'text': data.get('text'),
        'status': 'PENDING',
        'created': datetime.utcnow().isoformat(),
        'transmitters': data.get('transmitters', [])
    }
    
    logger.info(f"Created message: {new_message['id']}")
    return jsonify(new_message), 201

@app.route('/api/transmitters', methods=['GET'])
def get_transmitters():
    """Get all transmitters"""
    transmitters = [
        {
            'id': 101,
            'mnemonic': 'TXO1',
            'location': 'Chicago, IL',
            'frequency': 162.55,
            'status': 'OPERATIONAL'
        },
        {
            'id': 102,
            'mnemonic': 'TXO2',
            'location': 'Indianapolis, IN',
            'frequency': 162.40,
            'status': 'OPERATIONAL'
        },
        {
            'id': 103,
            'mnemonic': 'TXO3',
            'location': 'Milwaukee, WI',
            'frequency': 162.55,
            'status': 'OFFLINE'
        },
        {
            'id': 104,
            'mnemonic': 'TXO4',
            'location': 'Detroit, MI',
            'frequency': 162.40,
            'status': 'OPERATIONAL'
        }
    ]
    logger.info(f"Retrieved {len(transmitters)} transmitters")
    return jsonify(transmitters)

@app.route('/api/transmitters/<int:tx_id>', methods=['GET'])
def get_transmitter(tx_id):
    """Get specific transmitter"""
    transmitter = {
        'id': tx_id,
        'mnemonic': f'TXO{tx_id}',
        'location': 'City, State',
        'frequency': 162.55,
        'status': 'OPERATIONAL'
    }
    return jsonify(transmitter)

# ============================================================================
# HEALTH CHECK & ERROR HANDLERS
# ============================================================================

@app.route('/health', methods=['GET'])
def health_check():
    """Health check endpoint"""
    return jsonify({'status': 'healthy'}), 200

@app.errorhandler(404)
def not_found(error):
    """Handle 404 errors"""
    return jsonify({'error': 'Not found'}), 404

@app.errorhandler(500)
def server_error(error):
    """Handle 500 errors"""
    logger.error(f"Server error: {error}")
    return jsonify({'error': 'Internal server error'}), 500

# ============================================================================
# MAIN
# ============================================================================

if __name__ == '__main__':
    logger.info("Starting BMH HTTP Server on http://0.0.0.0:8080")
    # Run Flask development server
    app.run(
        host='0.0.0.0',
        port=8080,
        debug=True,
        use_reloader=True
    )
```

**Install Flask:**

```bash
pip install flask flask-cors
```

**Run Flask server:**

```bash
python3 web/backend/server.py
```

### 3. FastAPI HTTP Server (Async Alternative)

Modern async Python framework.

**File: `web/backend/fastapi_server.py`**

```python
#!/usr/bin/env python3
"""
BMH FastAPI Server - Async HTTP server on port 8080
"""

import logging
from datetime import datetime
from typing import List, Optional

from fastapi import FastAPI, HTTPException
from fastapi.staticfiles import StaticFiles
from fastapi.responses import FileResponse
from fastapi.middleware.cors import CORSMiddleware
import uvicorn

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# Initialize FastAPI app
app = FastAPI(
    title="BMH HTTP Server",
    description="BMH Web API and Static File Server",
    version="1.0.0"
)

# Enable CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# ============================================================================
# DATA MODELS
# ============================================================================

class Message:
    def __init__(self, id: int, afosid: str, text: str, status: str):
        self.id = id
        self.afosid = afosid
        self.text = text
        self.status = status
        self.created = datetime.utcnow().isoformat()

class Transmitter:
    def __init__(self, id: int, mnemonic: str, location: str, frequency: float):
        self.id = id
        self.mnemonic = mnemonic
        self.location = location
        self.frequency = frequency
        self.status = "OPERATIONAL"

# ============================================================================
# ENDPOINTS
# ============================================================================

@app.get("/")
async def root():
    """Root endpoint - redirect to index.html"""
    return FileResponse("../static/index.html")

@app.get("/api/status")
async def get_status():
    """Get system status"""
    return {
        "status": "online",
        "timestamp": datetime.utcnow().isoformat(),
        "version": "1.0.0",
        "service": "BMH FastAPI Server",
        "port": 8080
    }

@app.get("/api/messages")
async def list_messages():
    """Get all messages"""
    messages = [
        {
            "id": 1,
            "afosid": "WCXO53",
            "text": "Severe Thunderstorm Warning",
            "status": "ACTIVE"
        },
        {
            "id": 2,
            "afosid": "WCXO51",
            "text": "Winter Weather Advisory",
            "status": "ACTIVE"
        }
    ]
    logger.info(f"Retrieved {len(messages)} messages")
    return messages

@app.get("/api/messages/{msg_id}")
async def get_message(msg_id: int):
    """Get specific message"""
    return {
        "id": msg_id,
        "afosid": f"WC{msg_id}O53",
        "text": f"Weather Message {msg_id}",
        "status": "ACTIVE"
    }

@app.post("/api/messages")
async def create_message(message: dict):
    """Create new message"""
    return {
        "id": 999,
        "afosid": message.get("afosid", "NEW"),
        "text": message.get("text"),
        "status": "PENDING"
    }

@app.get("/api/transmitters")
async def list_transmitters():
    """Get all transmitters"""
    return [
        {"id": 101, "mnemonic": "TXO1", "location": "Chicago, IL", "frequency": 162.55},
        {"id": 102, "mnemonic": "TXO2", "location": "Indianapolis, IN", "frequency": 162.40},
        {"id": 103, "mnemonic": "TXO3", "location": "Milwaukee, WI", "frequency": 162.55}
    ]

@app.get("/health")
async def health():
    """Health check"""
    return {"status": "healthy"}

# ============================================================================
# MAIN
# ============================================================================

if __name__ == "__main__":
    # Mount static files
    app.mount("/static", StaticFiles(directory="../static"), name="static")
    
    logger.info("Starting BMH FastAPI Server on http://0.0.0.0:8080")
    uvicorn.run(
        app,
        host="0.0.0.0",
        port=8080,
        log_level="info"
    )
```

**Install FastAPI & Uvicorn:**

```bash
pip install fastapi uvicorn python-multipart
```

**Run FastAPI server:**

```bash
python3 -m uvicorn web.backend.fastapi_server:app --host 0.0.0.0 --port 8080 --reload
```

---

## JavaScript Frontend

### File: `web/static/index.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>BMH - Broadcast Message Handler</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <div id="app">
        <header class="header">
            <h1>BMH - Broadcast Message Handler</h1>
            <div id="status" class="status">
                <span id="status-indicator" class="indicator online"></span>
                <span id="status-text">Loading...</span>
            </div>
        </header>

        <main class="container">
            <section class="section">
                <h2>Messages</h2>
                <button id="btn-new-message" class="btn btn-primary">New Message</button>
                <div id="messages-list" class="list">
                    <!-- Populated by JavaScript -->
                </div>
            </section>

            <section class="section">
                <h2>Transmitters</h2>
                <div id="transmitters-list" class="list">
                    <!-- Populated by JavaScript -->
                </div>
            </section>
        </main>

        <footer class="footer">
            <p>&copy; 2026 BMH. Maintained by @warrickmoran and @nicholasgville23</p>
        </footer>
    </div>

    <script src="app.js"></script>
</body>
</html>
```

### File: `web/static/app.js`

```javascript
/**
 * BMH Web Application
 * JavaScript frontend for BMH HTTP server
 */

const API_BASE = 'http://localhost:8080/api';

class BMHApp {
    constructor() {
        this.messages = [];
        this.transmitters = [];
        this.init();
    }

    async init() {
        console.log('Initializing BMH App...');
        await this.checkStatus();
        await this.loadMessages();
        await this.loadTransmitters();
        this.attachEventListeners();
    }

    /**
     * Check server status
     */
    async checkStatus() {
        try {
            const response = await fetch(`${API_BASE}/status`);
            const data = await response.json();
            
            document.getElementById('status-text').textContent = 
                `${data.status} - ${data.version}`;
            document.getElementById('status-indicator').classList.add('online');
        } catch (error) {
            console.error('Status check failed:', error);
            document.getElementById('status-text').textContent = 'Offline';
            document.getElementById('status-indicator').classList.remove('online');
            document.getElementById('status-indicator').classList.add('offline');
        }
    }

    /**
     * Load messages from API
     */
    async loadMessages() {
        try {
            const response = await fetch(`${API_BASE}/messages`);
            this.messages = await response.json();
            this.renderMessages();
        } catch (error) {
            console.error('Failed to load messages:', error);
        }
    }

    /**
     * Render messages list
     */
    renderMessages() {
        const container = document.getElementById('messages-list');
        container.innerHTML = '';

        if (this.messages.length === 0) {
            container.innerHTML = '<p>No messages found</p>';
            return;
        }

        this.messages.forEach(msg => {
            const div = document.createElement('div');
            div.className = 'message-card card';
            div.innerHTML = `
                <div class="card-header">
                    <h3>${msg.afosid}</h3>
                    <span class="badge badge-${msg.status.toLowerCase()}">${msg.status}</span>
                </div>
                <div class="card-body">
                    <p>${msg.text}</p>
                    <small>Created: ${new Date(msg.created).toLocaleString()}</small>
                </div>
                <div class="card-footer">
                    <small>Transmitters: ${msg.transmitters.join(', ')}</small>
                </div>
            `;
            container.appendChild(div);
        });
    }

    /**
     * Load transmitters from API
     */
    async loadTransmitters() {
        try {
            const response = await fetch(`${API_BASE}/transmitters`);
            this.transmitters = await response.json();
            this.renderTransmitters();
        } catch (error) {
            console.error('Failed to load transmitters:', error);
        }
    }

    /**
     * Render transmitters list
     */
    renderTransmitters() {
        const container = document.getElementById('transmitters-list');
        container.innerHTML = '';

        if (this.transmitters.length === 0) {
            container.innerHTML = '<p>No transmitters found</p>';
            return;
        }

        this.transmitters.forEach(tx => {
            const div = document.createElement('div');
            div.className = 'transmitter-card card';
            div.innerHTML = `
                <div class="card-header">
                    <h3>${tx.mnemonic}</h3>
                    <span class="badge badge-${tx.status.toLowerCase()}">${tx.status}</span>
                </div>
                <div class="card-body">
                    <p><strong>Location:</strong> ${tx.location}</p>
                    <p><strong>Frequency:</strong> ${tx.frequency} MHz</p>
                </div>
            `;
            container.appendChild(div);
        });
    }

    /**
     * Create new message
     */
    async createMessage() {
        const text = prompt('Enter message text:');
        if (!text) return;

        try {
            const response = await fetch(`${API_BASE}/messages`, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ text, afosid: 'NEW' })
            });
            const newMsg = await response.json();
            this.messages.push(newMsg);
            this.renderMessages();
            alert('Message created successfully!');
        } catch (error) {
            console.error('Failed to create message:', error);
            alert('Error creating message');
        }
    }

    /**
     * Attach event listeners
     */
    attachEventListeners() {
        document.getElementById('btn-new-message').addEventListener('click', 
            () => this.createMessage());
    }
}

// Initialize app when DOM is ready
document.addEventListener('DOMContentLoaded', () => {
    window.app = new BMHApp();
});
```

### File: `web/static/style.css`

```css
/* BMH Web Application Styles */

* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
    background-color: #f5f5f5;
    color: #333;
    line-height: 1.6;
}

.header {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    padding: 20px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}

.header h1 {
    font-size: 1.8em;
    margin: 0;
}

.status {
    display: flex;
    align-items: center;
    gap: 10px;
    font-size: 0.9em;
}

.indicator {
    display: inline-block;
    width: 12px;
    height: 12px;
    border-radius: 50%;
    animation: pulse 2s infinite;
}

.indicator.online {
    background-color: #4caf50;
}

.indicator.offline {
    background-color: #f44336;
    animation: none;
}

@keyframes pulse {
    0%, 100% { opacity: 1; }
    50% { opacity: 0.5; }
}

.container {
    max-width: 1200px;
    margin: 20px auto;
    padding: 0 20px;
}

.section {
    background: white;
    padding: 20px;
    margin-bottom: 20px;
    border-radius: 8px;
    box-shadow: 0 1px 3px rgba(0,0,0,0.1);
}

.section h2 {
    margin-bottom: 15px;
    color: #667eea;
}

.btn {
    padding: 10px 20px;
    border: none;
    border-radius: 4px;
    font-size: 0.95em;
    cursor: pointer;
    transition: background 0.3s;
}

.btn-primary {
    background-color: #667eea;
    color: white;
    margin-bottom: 15px;
}

.btn-primary:hover {
    background-color: #5568d3;
}

.list {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
    gap: 15px;
}

.card {
    background: white;
    border: 1px solid #ddd;
    border-radius: 6px;
    overflow: hidden;
    transition: box-shadow 0.3s;
}

.card:hover {
    box-shadow: 0 4px 12px rgba(0,0,0,0.15);
}

.card-header {
    background-color: #f8f9fa;
    padding: 15px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    border-bottom: 1px solid #ddd;
}

.card-header h3 {
    margin: 0;
    font-size: 1.1em;
}

.card-body {
    padding: 15px;
}

.card-footer {
    padding: 10px 15px;
    background-color: #f8f9fa;
    font-size: 0.9em;
    color: #666;
}

.badge {
    display: inline-block;
    padding: 4px 8px;
    border-radius: 3px;
    font-size: 0.8em;
    font-weight: bold;
}

.badge-active {
    background-color: #4caf50;
    color: white;
}

.badge-pending {
    background-color: #ff9800;
    color: white;
}

.badge-offline {
    background-color: #f44336;
    color: white;
}

.badge-operational {
    background-color: #4caf50;
    color: white;
}

.footer {
    background-color: #f8f9fa;
    color: #666;
    text-align: center;
    padding: 20px;
    margin-top: 40px;
    border-top: 1px solid #ddd;
}

@media (max-width: 768px) {
    .header {
        flex-direction: column;
        gap: 10px;
    }

    .list {
        grid-template-columns: 1fr;
    }
}
```

---

## Python Backend API

### File: `web/backend/models.py`

```python
"""
BMH Data Models
"""

from dataclasses import dataclass, asdict
from datetime import datetime
from typing import List, Optional

@dataclass
class Message:
    """Broadcast message model"""
    id: int
    afosid: str
    text: str
    status: str
    created: str = None
    expires: str = None
    transmitters: List[int] = None

    def __post_init__(self):
        if self.created is None:
            self.created = datetime.utcnow().isoformat()
        if self.transmitters is None:
            self.transmitters = []

    def to_dict(self):
        return asdict(self)

@dataclass
class Transmitter:
    """Transmitter model"""
    id: int
    mnemonic: str
    location: str
    frequency: float
    status: str = "OPERATIONAL"

    def to_dict(self):
        return asdict(self)

@dataclass
class TransmitterGroup:
    """Transmitter group model"""
    id: int
    name: str
    transmitters: List[int]
    enabled: bool = True

    def to_dict(self):
        return asdict(self)
```

### File: `web/backend/handlers.py`

```python
"""
BMH API Handlers - Business logic layer
"""

import logging
from typing import List, Dict, Optional
from .models import Message, Transmitter

logger = logging.getLogger(__name__)

class MessageHandler:
    """Handle message operations"""
    
    def __init__(self):
        self.messages: Dict[int, Message] = {}
        self._load_sample_data()
    
    def _load_sample_data(self):
        """Load sample messages"""
        self.messages[1] = Message(
            id=1,
            afosid="WCXO53",
            text="Severe Thunderstorm Warning",
            status="ACTIVE",
            transmitters=[101, 102, 103]
        )
        self.messages[2] = Message(
            id=2,
            afosid="WCXO51",
            text="Winter Weather Advisory",
            status="ACTIVE",
            transmitters=[101, 104]
        )
    
    def get_all(self) -> List[Message]:
        """Get all messages"""
        return list(self.messages.values())
    
    def get_by_id(self, msg_id: int) -> Optional[Message]:
        """Get message by ID"""
        return self.messages.get(msg_id)
    
    def create(self, message: Message) -> Message:
        """Create new message"""
        msg_id = max(self.messages.keys()) + 1 if self.messages else 1
        message.id = msg_id
        self.messages[msg_id] = message
        logger.info(f"Created message {msg_id}")
        return message
    
    def update(self, msg_id: int, data: dict) -> Optional[Message]:
        """Update message"""
        if msg_id not in self.messages:
            return None
        msg = self.messages[msg_id]
        for key, value in data.items():
            if hasattr(msg, key):
                setattr(msg, key, value)
        logger.info(f"Updated message {msg_id}")
        return msg
    
    def delete(self, msg_id: int) -> bool:
        """Delete message"""
        if msg_id in self.messages:
            del self.messages[msg_id]
            logger.info(f"Deleted message {msg_id}")
            return True
        return False

class TransmitterHandler:
    """Handle transmitter operations"""
    
    def __init__(self):
        self.transmitters: Dict[int, Transmitter] = {}
        self._load_sample_data()
    
    def _load_sample_data(self):
        """Load sample transmitters"""
        self.transmitters[101] = Transmitter(
            id=101, mnemonic="TXO1", location="Chicago, IL", frequency=162.55
        )
        self.transmitters[102] = Transmitter(
            id=102, mnemonic="TXO2", location="Indianapolis, IN", frequency=162.40
        )
        self.transmitters[103] = Transmitter(
            id=103, mnemonic="TXO3", location="Milwaukee, WI", 
            frequency=162.55, status="OFFLINE"
        )
    
    def get_all(self) -> List[Transmitter]:
        """Get all transmitters"""
        return list(self.transmitters.values())
    
    def get_by_id(self, tx_id: int) -> Optional[Transmitter]:
        """Get transmitter by ID"""
        return self.transmitters.get(tx_id)
```

---

## Running the Server

### Option 1: Simple HTTP Server (Static Files Only)

```bash
cd BMH/web/static
python3 -m http.server 8080

# In browser:
# http://localhost:8080
```

### Option 2: Flask Server (Recommended)

```bash
cd BMH
pip install flask flask-cors

python3 web/backend/server.py

# In browser:
# http://localhost:8080
```

### Option 3: FastAPI Server (Async)

```bash
cd BMH
pip install fastapi uvicorn

python3 -m uvicorn web.backend.fastapi_server:app --host 0.0.0.0 --port 8080 --reload

# In browser:
# http://localhost:8080
```

### Verify Server is Running

```bash
# Test health endpoint
curl http://localhost:8080/health

# Get API status
curl http://localhost:8080/api/status

# Get messages
curl http://localhost:8080/api/messages

# Get transmitters
curl http://localhost:8080/api/transmitters
```

---

## Development Workflow

### 1. File Watching & Auto-Reload

**Python backend (Flask with reloader):**
```bash
python3 web/backend/server.py
# Changes to .py files auto-reload
```

**Frontend (with browser refresh):**
```bash
# Edit web/static/app.js or style.css
# Refresh browser (Ctrl+R / Cmd+R)
```

### 2. Testing API Endpoints

**Using curl:**
```bash
# Create message
curl -X POST http://localhost:8080/api/messages \
  -H "Content-Type: application/json" \
  -d '{"afosid":"NEW","text":"Test message"}'

# Get specific message
curl http://localhost:8080/api/messages/1
```

**Using Python requests:**
```python
import requests

# Get messages
resp = requests.get('http://localhost:8080/api/messages')
print(resp.json())

# Create message
resp = requests.post('http://localhost:8080/api/messages', 
                     json={'text': 'Test'})
print(resp.json())
```

### 3. Debugging JavaScript

**Browser DevTools:**
- Open: `F12` or `Ctrl+Shift+I`
- Console: View logs from `console.log()`
- Network: Monitor API calls
- Elements: Inspect HTML/CSS

### 4. Logging & Monitoring

**Python logging:**
```python
import logging
logging.basicConfig(level=logging.DEBUG)
logger = logging.getLogger(__name__)
logger.debug("Debug message")
logger.info("Info message")
logger.error("Error message")
```

**JavaScript console:**
```javascript
console.log('Log message');
console.error('Error message');
console.table(arrayOfObjects);
```

---

## Project Structure Recap

```
web/
├── backend/
│   ├── server.py           # Flask HTTP server
│   ├── fastapi_server.py   # FastAPI alternative
│   ├── api.py              # API endpoints
│   ├── models.py           # Data models
│   └── handlers.py         # Business logic
├── static/
│   ├── index.html          # Main HTML
│   ├── app.js              # JavaScript app
│   └── style.css           # Stylesheets
├── frontend/
│   └── (source files)
└── templates/
    └── (Jinja2 templates, optional)
```

---

## Next Steps

1. **Frontend Enhancement**: Add more components (modals, forms, charts)
2. **Backend Integration**: Connect to actual BMH Java services
3. **Database**: Persist messages/transmitters to PostgreSQL
4. **Authentication**: Add login/user management
5. **Deployment**: Containerize with Docker, deploy to production

---

**Maintained by:** @warrickmoran and @nicholasgville23

Refer to [GETTING_STARTED.md](GETTING_STARTED.md) for Java/C# setup guides.
