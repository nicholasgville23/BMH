# Getting Started with BMH

This guide covers how to build and run BMH in Java with Python GUI support, and integrate with C# Visual Studio 2026 (Visual Studio Insiders).

## Table of Contents

- [Prerequisites](#prerequisites)
- [Java Setup & Build](#java-setup--build)
- [Python GUI Setup](#python-gui-setup)
- [C# Visual Studio 2026 Integration](#c-visual-studio-2026-integration)
- [Running BMH](#running-bmh)
- [Development Workflow](#development-workflow)

---

## Prerequisites

### System Requirements

- **OS**: Linux (CentOS/RHEL recommended), macOS, or Windows
- **Disk Space**: ~5-10 GB for full build + dependencies
- **RAM**: 8 GB minimum, 16 GB recommended

### Common Requirements

| Component | Version | Purpose |
|-----------|---------|---------|
| Git | 2.40+ | Version control |
| Docker | 20.10+ | Optional: containerized deployment |

---

## Java Setup & Build

### 1. Install Java Development Kit (JDK)

**Option A: JDK 11 (Recommended for BMH)**

```bash
# Ubuntu/Debian
sudo apt-get update
sudo apt-get install openjdk-11-jdk openjdk-11-jdk-headless

# macOS (Homebrew)
brew install openjdk@11

# CentOS/RHEL
sudo yum install java-11-openjdk java-11-openjdk-devel

# Windows (via Chocolatey)
choco install openjdk11
```

**Verify installation:**

```bash
java -version
javac -version
```

### 2. Install Maven

Maven is used for building BMH Java components.

```bash
# Ubuntu/Debian
sudo apt-get install maven

# macOS
brew install maven

# CentOS/RHEL
sudo yum install maven

# Windows (via Chocolatey)
choco install maven

# Or download from: https://maven.apache.org/download.cgi
```

**Verify:**

```bash
mvn --version
```

### 3. Install Eclipse/IDE (Optional but Recommended)

For GUI development, Eclipse is commonly used:

```bash
# Download Eclipse IDE for RCP/Plug-in Developers
# https://www.eclipse.org/downloads/packages/

# Or via package manager
# macOS
brew install eclipse-java

# Linux: Download and extract manually
wget https://www.eclipse.org/downloads/download.php?file=/eclipse/downloads/drops4/R-4.26.0-202303010400/eclipse-java-2023-03-R-linux-gtk-x86_64.tar.gz
tar -xzf eclipse-*.tar.gz
```

### 4. Clone and Build BMH

```bash
# Clone the repository
git clone https://github.com/nicholasgville23/BMH.git
cd BMH

# Build all modules
mvn clean install

# Build specific module (e.g., common)
mvn clean install -pl common/com.raytheon.uf.common.bmh

# Skip tests for faster builds
mvn clean install -DskipTests
```

**Build Output:**
- JAR files: `target/` directories in each module
- Artifacts: `~/.m2/repository/com/raytheon/uf/` (local Maven cache)

### 5. Database Setup (EDEX/Server)

If running the full server stack:

```bash
# Install PostgreSQL
sudo apt-get install postgresql postgresql-contrib

# Start PostgreSQL service
sudo systemctl start postgresql

# Create BMH databases
sudo -u postgres psql < edex/deploy.edex-BMH/opt/db/ddl/bmh/createBMHDB.sql
sudo -u postgres psql < edex/deploy.edex-BMH/opt/db/ddl/bmh/createBMHPracticeDB.sql
```

---

## Python GUI Setup

### 1. Install Python & Dependencies

```bash
# Install Python 3.9+ (recommended)
# Ubuntu/Debian
sudo apt-get install python3.9 python3.9-venv python3-pip

# macOS
brew install python@3.9

# Windows
# Download from https://www.python.org/downloads/
# Ensure "Add Python to PATH" is checked during installation
```

### 2. Create Virtual Environment

```bash
# Create venv
python3 -m venv ~/bmh-env
source ~/bmh-env/bin/activate  # Linux/macOS
# OR
~/bmh-env\Scripts\activate  # Windows

# Upgrade pip
pip install --upgrade pip setuptools wheel
```

### 3. Install Python GUI Dependencies

```bash
# PyQt5 or PySimpleGUI for GUI
pip install PyQt5>=5.15.0 PySimpleGUI>=4.60.0

# Additional scientific/data libraries
pip install numpy scipy pandas matplotlib

# Server/API interaction
pip install requests flask fastapi uvicorn

# Database
pip install psycopg2-binary sqlalchemy

# Utilities
pip install python-dotenv pyyaml logging-loki
```

### 4. Create Python GUI Application Structure

**Example: `python/bmh_gui/main.py`**

```python
#!/usr/bin/env python3
"""
BMH Python GUI Application
Provides interface to interact with BMH services
"""

import sys
import logging
from pathlib import Path

# PyQt5 example
from PyQt5.QtWidgets import (
    QApplication, QMainWindow, QWidget, QVBoxLayout, QHBoxLayout,
    QLabel, QPushButton, QTableWidget, QTableWidgetItem, QLineEdit
)
from PyQt5.QtCore import Qt, QThread, pyqtSignal

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class BMHMainWindow(QMainWindow):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("BMH - Broadcast Message Handler")
        self.setGeometry(100, 100, 1024, 768)
        self.init_ui()
    
    def init_ui(self):
        """Initialize UI components"""
        central_widget = QWidget()
        self.setCentralWidget(central_widget)
        
        layout = QVBoxLayout()
        
        # Title
        title = QLabel("BMH Control Panel")
        layout.addWidget(title)
        
        # Message table
        self.table = QTableWidget()
        self.table.setColumnCount(4)
        self.table.setHorizontalHeaderLabels([
            "Message ID", "Status", "Type", "Transmitters"
        ])
        layout.addWidget(self.table)
        
        # Control buttons
        button_layout = QHBoxLayout()
        btn_refresh = QPushButton("Refresh")
        btn_new = QPushButton("New Message")
        btn_delete = QPushButton("Delete")
        
        btn_refresh.clicked.connect(self.refresh_messages)
        btn_new.clicked.connect(self.new_message)
        
        button_layout.addWidget(btn_refresh)
        button_layout.addWidget(btn_new)
        button_layout.addWidget(btn_delete)
        layout.addLayout(button_layout)
        
        central_widget.setLayout(layout)
    
    def refresh_messages(self):
        """Fetch and display messages from server"""
        logger.info("Refreshing messages...")
        # TODO: Connect to BMH REST API
    
    def new_message(self):
        """Open dialog for new message creation"""
        logger.info("Creating new message...")
        # TODO: Open message creation dialog

def main():
    app = QApplication(sys.argv)
    window = BMHMainWindow()
    window.show()
    sys.exit(app.exec_())

if __name__ == '__main__':
    main()
```

**Run the GUI:**

```bash
cd python/bmh_gui
python main.py
```

---

## C# Visual Studio 2026 Integration

### 1. Install Visual Studio 2026 Insiders

```bash
# Windows only
# Download from: https://visualstudio.microsoft.com/vs/preview/
# Install Visual Studio 2026 Preview with:
# - .NET Desktop Development
# - C# support
# - Docker tools (optional)
```

### 2. Create C# BMH Client Project

**Visual Studio New Project:**

1. File → New Project
2. Select "C# Console App (.NET 8.0)" or "WPF App"
3. Name: `BMH.Client.CSharp`
4. Solution: `BMH.sln`

### 3. Add NuGet Dependencies

**In Package Manager Console:**

```powershell
Install-Package Newtonsoft.Json -Version 13.0.3
Install-Package RestSharp -Version 107.0.0
Install-Package Serilog -Version 3.1.0
Install-Package Serilog.Sinks.Console -Version 5.0.0
Install-Package EntityFramework -Version 6.4.4
```

### 4. Create C# BMH Client Code

**File: `BMHClient.cs`**

```csharp
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using Newtonsoft.Json;
using RestSharp;
using Serilog;

namespace BMH.Client.CSharp
{
    /// <summary>
    /// BMH REST API Client for C#/.NET applications
    /// </summary>
    public class BMHClient
    {
        private readonly string _baseUrl;
        private readonly RestClient _client;
        private readonly ILogger _logger;

        public BMHClient(string baseUrl = "http://localhost:8080/bmh")
        {
            _baseUrl = baseUrl;
            _client = new RestClient(_baseUrl);
            _logger = new LoggerConfiguration()
                .WriteTo.Console()
                .CreateLogger();
        }

        /// <summary>
        /// Fetch all active messages
        /// </summary>
        public async Task<List<BroadcastMessage>> GetMessagesAsync()
        {
            var request = new RestRequest("/api/messages", Method.Get);
            var response = await _client.ExecuteAsync(request);

            if (response.IsSuccessful)
            {
                _logger.Information("Retrieved messages successfully");
                return JsonConvert.DeserializeObject<List<BroadcastMessage>>(response.Content);
            }

            _logger.Error($"Failed to retrieve messages: {response.StatusCode}");
            return new List<BroadcastMessage>();
        }

        /// <summary>
        /// Create a new broadcast message
        /// </summary>
        public async Task<BroadcastMessage> CreateMessageAsync(BroadcastMessage message)
        {
            var request = new RestRequest("/api/messages", Method.Post);
            request.AddJsonBody(message);
            var response = await _client.ExecuteAsync(request);

            if (response.IsSuccessful)
            {
                _logger.Information("Message created: {MessageId}", message.Id);
                return JsonConvert.DeserializeObject<BroadcastMessage>(response.Content);
            }

            _logger.Error($"Failed to create message: {response.StatusCode}");
            throw new Exception($"API Error: {response.StatusCode}");
        }

        /// <summary>
        /// Get transmitter status
        /// </summary>
        public async Task<List<Transmitter>> GetTransmittersAsync()
        {
            var request = new RestRequest("/api/transmitters", Method.Get);
            var response = await _client.ExecuteAsync(request);

            if (response.IsSuccessful)
            {
                return JsonConvert.DeserializeObject<List<Transmitter>>(response.Content);
            }

            throw new Exception($"Failed to fetch transmitters: {response.StatusCode}");
        }
    }

    /// <summary>
    /// Broadcast Message model
    /// </summary>
    public class BroadcastMessage
    {
        [JsonProperty("id")]
        public int Id { get; set; }

        [JsonProperty("afosid")]
        public string AfosId { get; set; }

        [JsonProperty("message_text")]
        public string MessageText { get; set; }

        [JsonProperty("status")]
        public string Status { get; set; }

        [JsonProperty("effective_time")]
        public DateTime EffectiveTime { get; set; }

        [JsonProperty("expiration_time")]
        public DateTime ExpirationTime { get; set; }

        [JsonProperty("transmitters")]
        public List<int> Transmitters { get; set; }
    }

    /// <summary>
    /// Transmitter model
    /// </summary>
    public class Transmitter
    {
        [JsonProperty("id")]
        public int Id { get; set; }

        [JsonProperty("mnemonic")]
        public string Mnemonic { get; set; }

        [JsonProperty("location")]
        public string Location { get; set; }

        [JsonProperty("status")]
        public string Status { get; set; }

        [JsonProperty("frequency")]
        public double Frequency { get; set; }
    }
}
```

**File: `Program.cs`**

```csharp
using System;
using System.Threading.Tasks;
using Serilog;

namespace BMH.Client.CSharp
{
    class Program
    {
        static async Task Main(string[] args)
        {
            Log.Logger = new Serilog.LoggerConfiguration()
                .MinimumLevel.Information()
                .WriteTo.Console()
                .CreateLogger();

            try
            {
                var client = new BMHClient("http://localhost:8080/bmh");

                // Fetch messages
                Log.Information("Fetching BMH messages...");
                var messages = await client.GetMessagesAsync();
                Log.Information("Retrieved {Count} messages", messages.Count);

                // Fetch transmitters
                Log.Information("Fetching transmitters...");
                var transmitters = await client.GetTransmittersAsync();
                Log.Information("Retrieved {Count} transmitters", transmitters.Count);

                // Create new message
                var newMessage = new BroadcastMessage
                {
                    AfosId = "WCXO53",
                    MessageText = "Severe Weather Alert",
                    Status = "ACTIVE",
                    EffectiveTime = DateTime.UtcNow,
                    ExpirationTime = DateTime.UtcNow.AddHours(2)
                };

                Log.Information("Creating new message...");
                var createdMessage = await client.CreateMessageAsync(newMessage);
                Log.Information("Message created with ID: {Id}", createdMessage.Id);
            }
            catch (Exception ex)
            {
                Log.Fatal(ex, "Application terminated unexpectedly");
            }
            finally
            {
                Log.CloseAndFlush();
            }
        }
    }
}
```

### 5. Build and Run C# Project

```powershell
# In Visual Studio 2026
# Build → Build Solution (Ctrl+Shift+B)
# Run the application (F5)

# Or from command line
cd BMH.Client.CSharp
dotnet build
dotnet run
```

---

## Running BMH

### Java-Based Services

```bash
# Build all
cd BMH
mvn clean install -DskipTests

# Run CAVE client (GUI)
cd cave/com.raytheon.uf.viz.bmh
mvn exec:java -Dexec.mainClass="com.raytheon.uf.viz.bmh.Activator"

# Run EDEX server (requires full EDEX environment)
# See: edex/com.raytheon.uf.edex.bmh/README
```

### Python GUI

```bash
source ~/bmh-env/bin/activate
cd python/bmh_gui
python main.py
```

### C# Client

```powershell
cd BMH.Client.CSharp
dotnet run
```

---

## Development Workflow

### 1. Local Development Environment Setup

```bash
# Clone repo
git clone https://github.com/nicholasgville23/BMH.git
cd BMH

# Create local branches for different language stacks
git checkout -b feature/java-service
git checkout -b feature/python-gui
git checkout -b feature/csharp-client
```

### 2. Java Development

```bash
# Use Eclipse IDE
# File → Open Projects from File System
# Select: BMH/cave, BMH/common, BMH/edex directories

# Or command-line Maven workflow
mvn clean install -pl <module> -am
mvn test -Dtest=<TestClass>
```

### 3. Python Development

```bash
source ~/bmh-env/bin/activate

# Install in development mode
pip install -e python/bmh_gui

# Run with hot-reload
python -m PyQt5.QtCore

# Testing
pytest python/tests/ -v
```

### 4. C# Development

```
# Visual Studio 2026
# - Right-click project → Properties
# - Set Debug → Program arguments
# - Press F5 to run with debugger
# - Use NuGet Package Manager for dependencies
```

### 5. Integration Testing

```bash
# Run full test suite
mvn clean verify

# Python tests
pytest python/tests/ --cov=python/bmh_gui

# C# unit tests
dotnet test BMH.Client.CSharp.Tests
```

---

## Troubleshooting

### Java Issues

| Problem | Solution |
|---------|----------|
| Maven build fails | Run `mvn dependency:resolve` or clear `~/.m2/repository` |
| Eclipse plugin errors | Update Eclipse to latest version |
| Database connection error | Verify PostgreSQL is running: `systemctl status postgresql` |

### Python Issues

| Problem | Solution |
|---------|----------|
| PyQt5 import error | Run: `pip install --upgrade PyQt5` |
| Virtual env issues | Recreate: `rm -rf ~/bmh-env && python3 -m venv ~/bmh-env` |

### C# Issues

| Problem | Solution |
|---------|----------|
| NuGet restore fails | Run: `nuget restore BMH.sln` or use Visual Studio Package Manager |
| .NET version mismatch | Install: `dotnet-sdk-8.0` or newer |

---

## Next Steps

- Review [README.md](README.md) for project overview
- Check individual module documentation in respective directories
- Join discussions or contribute via GitHub Issues/PRs
- Refer to deployment guides in `edex/deploy-BMH/`

---

**Maintained by:** @warrickmoran and @nicholasgville23
