# JARDesigner - Complete Architecture Documentation

## Executive Summary

**JARDesigner** is a web-based GUI for building and simulating multiscale neural models using MOOSE (Multiscale Object-Oriented Simulation Environment). It enables users to design neurons with electrical signaling, chemical signaling (reaction-diffusion), and their bidirectional coupling through a visual interface with real-time 3D visualization.

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Technology Stack](#technology-stack)
3. [Architecture Overview](#architecture-overview)
4. [Directory Structure](#directory-structure)
5. [Backend Architecture](#backend-architecture)
6. [Frontend Architecture](#frontend-architecture)
7. [Core Python Package](#core-python-package)
8. [Communication Flow](#communication-flow)
9. [Data Models](#data-models)
10. [3D Visualization System](#3d-visualization-system)
11. [Simulation Workflow](#simulation-workflow)
12. [Key Features](#key-features)
13. [Deployment Modes](#deployment-modes)

---

## 1. Project Overview

### What is JARDesigner?

JARDesigner is a successor to RDesigner (the original MOOSE GUI) that provides:
- **Web-based interface** for building multiscale neuronal models
- **3D visualization** of neuronal morphology and simulation results
- **Real-time simulation** with live data streaming
- **Interactive model building** with graphical configuration panels
- **JSON-based model definitions** following a structured schema

### Primary Use Cases

1. **Neuronal Morphology Design**: Build neurons from scratch or load from SWC/NeuroML files
2. **Electrical Properties**: Add ion channels, passive properties, and stimulation
3. **Chemical Signaling**: Integrate reaction-diffusion models for molecular pathways
4. **Multiscale Coupling**: Link electrical and chemical domains via adaptors
5. **Simulation & Visualization**: Run simulations with real-time 3D visualization and plotting

---

## 2. Technology Stack

### Frontend Technologies

| Technology | Version | Purpose |
|------------|---------|---------|
| **React** | 18.0.0 | UI framework |
| **Vite** | 6.3.5 | Build tool and dev server |
| **Material-UI (MUI)** | 6.4.1 | Component library |
| **Three.js** | 0.178.0 | 3D graphics rendering |
| **Socket.IO Client** | 4.8.1 | WebSocket communication |
| **Lodash** | 4.17.21 | Utility functions |
| **AJV** | 8.17.1 | JSON schema validation |
| **React Markdown** | 10.1.0 | Markdown rendering for docs |
| **UUID** | 11.1.0 | Unique ID generation |

### Backend Technologies

| Technology | Version | Purpose |
|------------|---------|---------|
| **Flask** | 3.1.1 | Web framework |
| **Flask-SocketIO** | 5.5.1 | WebSocket support |
| **Flask-CORS** | 5.0.1 | Cross-origin resource sharing |
| **MOOSE** | 0.9.8 | Neural simulation engine |
| **NumPy** | 1.21.5 | Numerical computing |
| **Matplotlib** | 3.10.5 | Plotting |
| **lxml** | 4.8.0 | XML parsing (NeuroML/SBML) |
| **jsonschema** | 3.2.0 | JSON validation |
| **Requests** | 2.32.4 | HTTP client |

### Core Languages

- **Python 3.x** - Backend, simulation engine, MOOSE integration
- **JavaScript (ES6+)** - Frontend application logic
- **JSX** - React components
- **JSON** - Data interchange and model definitions
- **CSS** - Styling

---

## 3. Architecture Overview

### High-Level Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                         USER BROWSER                            │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                   REACT FRONTEND                           │ │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐    │ │
│  │  │  Menu    │  │   3D     │  │  Graph   │  │   JSON   │    │ │
│  │  │  Boxes   │  │ Viewer   │  │  Viewer  │  │  Editor  │    │ │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘    │ │
│  │                                                            │ │
│  │  ┌────────────────────────────────────────────────────┐    │ │
│  │  │           App Logic (State Management)             │    │ │
│  │  └────────────────────────────────────────────────────┘    │ │
│  └────────────────────────────────────────────────────────────┘ │
│                            ▲  │                                 │
│                            │  │ HTTP/WebSocket                  │
│                            │  ▼                                 │
└────────────────────────────┼──┼──────────────────────────────────
                             │  │
                    ┌────────┴──┴───────┐
                    │   FLASK SERVER    │
                    │   (Port 5000)     │
                    └────────┬──┬───────┘
                             │  │
         ┌───────────────────┘  └──────────────────┐
         │                                          │
         ▼                                          ▼
┌─────────────────┐                    ┌──────────────────────┐
│  FILE STORAGE   │                    │  SUBPROCESS MANAGER  │
│  - Temp configs │                    │  (MOOSE Simulations) │
│  - User uploads │                    └──────────┬───────────┘
│  - Session data │                               │
└─────────────────┘                               ▼
                                         ┌──────────────────┐
                                         │  MOOSE PYTHON    │
                                         │  - jardesigner.py│
                                         │  - jarmoogli.py  │
                                         │  - Simulation    │
                                         └──────────────────┘
```

### Three-Tier Architecture

1. **Presentation Tier** (Frontend)
   - React-based single-page application
   - Runs in user's browser
   - Handles UI rendering and user interactions

2. **Application Tier** (Backend)
   - Flask server managing HTTP and WebSocket connections
   - Orchestrates simulation processes
   - Routes data between frontend and simulation engine

3. **Computation Tier** (MOOSE Engine)
   - Python subprocess running MOOSE simulations
   - Generates 3D scene data and simulation frames
   - Sends data back to Flask via HTTP POST

---

## 4. Directory Structure

```
jardesigner-main/
├── backend/                      # Flask server
│   ├── server.py                 # Main Flask application
│   ├── requirements.txt          # Python dependencies
│   ├── jardesignerSchema.json    # JSON schema for validation
│   ├── jardesignerDataFrameSchema.json
│   ├── jardesignerSceneGraphSchema.json
│   ├── CELL_MODELS/              # Morphology files (SWC)
│   ├── CHEM_MODELS/              # Chemical models (.g files)
│   ├── temp_configs/             # Temporary JSON configs (runtime)
│   ├── user_uploads/             # Per-session user files (runtime)
│   └── simulation_plots/         # Generated SVG plots (runtime)
│
├── frontend/                     # React application
│   ├── src/
│   │   ├── App.jsx               # Root component
│   │   ├── AppLayout.jsx         # Main layout with menu bar
│   │   ├── appLogic.js           # Core state management hook
│   │   ├── replayLogic.js        # Simulation replay logic
│   │   ├── main.jsx              # Entry point
│   │   ├── standalone.jsx        # Standalone build entry
│   │   ├── components/
│   │   │   ├── DisplayWindow.jsx    # Main display area
│   │   │   ├── GraphWindow.jsx      # SVG plot viewer
│   │   │   ├── ThreeDViewer.jsx     # 3D visualization
│   │   │   ├── ThreeDManager.js     # Three.js scene manager
│   │   │   ├── ShapeFactory.js      # 3D shape creation
│   │   │   ├── ReplayContext.js     # Replay state context
│   │   │   ├── colormap.js          # Color mapping utilities
│   │   │   ├── JsonText.jsx         # JSON editor
│   │   │   ├── MarkdownText.jsx     # Help documentation viewer
│   │   │   └── MenuBoxes/           # Configuration panels
│   │   │       ├── FileMenuBox.jsx
│   │   │       ├── RunMenuBox.jsx
│   │   │       ├── MorphoMenuBox.jsx
│   │   │       ├── SpineMenuBox.jsx
│   │   │       ├── ElecMenuBox.jsx
│   │   │       ├── PassiveMenuBox.jsx
│   │   │       ├── ChemMenuBox.jsx
│   │   │       ├── AdaptorsMenuBox.jsx
│   │   │       ├── StimMenuBox.jsx
│   │   │       ├── PlotMenuBox.jsx
│   │   │       ├── ThreeDMenuBox.jsx
│   │   │       └── SimOutputMenuBox.jsx
│   │   ├── assets/               # Icons and images
│   │   └── utils/                # Utility functions
│   ├── public/                   # Static assets
│   ├── package.json              # NPM dependencies
│   ├── vite.config.js            # Vite build config
│   ├── vite.config.standalone.js # Standalone build config
│   └── standalone.html           # Standalone HTML template
│
├── jardesigner/                  # Core Python package
│   ├── __init__.py
│   ├── jardesigner.py            # Main MOOSE integration (2225 lines)
│   ├── jardesignerProtos.py      # Prototype builders
│   ├── jarmoogli.py              # 3D visualization data generation
│   ├── fixXreacs.py              # Chemical reaction fixes
│   ├── context.py                # Context management
│   ├── jardesignerSchema.json    # JSON schema
│   └── CHEM_MODELS/              # Built-in chemical models
│
├── SERVER_CONFIG/                # Deployment configuration
│   ├── jardesigner               # Launch script
│   ├── index.html                # Entry HTML
│   └── manifest.txt              # File manifest
│
└── run_jardesigner.py            # Python entry point
```

---

## 5. Backend Architecture

### Flask Server (`backend/server.py`)

The Flask server is the central orchestrator of the application. It handles:

#### Key Components

**1. Flask App with CORS and SocketIO**
```python
app = Flask(__name__)
CORS(app)  # Allow cross-origin requests
socketio = SocketIO(app, cors_allowed_origins="*")
```

**2. Process Management**
```python
running_processes = {}      # PID -> process info
client_sim_map = {}         # client_id -> PID
sid_clientid_map = {}       # socket_id -> client_id
```

**3. Directory Structure**
```python
TEMP_CONFIG_DIR     # Temporary JSON configs
USER_UPLOADS_DIR    # Per-session user uploads
```

#### HTTP Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/` | GET | Health check |
| `/upload_file` | POST | Upload morphology/model files |
| `/launch_simulation` | POST | Start MOOSE simulation subprocess |
| `/simulation_status/<pid>` | GET | Check simulation status |
| `/session_file/<client_id>/<filename>` | GET | Serve session files (SVG plots) |
| `/reset_simulation` | POST | Terminate simulation process |
| `/internal/push_data` | POST | Receive data from MOOSE subprocess |

#### WebSocket Events

| Event | Direction | Purpose |
|-------|-----------|---------|
| `connect` | Client → Server | WebSocket connection established |
| `register_client` | Client → Server | Register client ID to socket |
| `join_sim_channel` | Client → Server | Join room for data streaming |
| `sim_command` | Client → Server | Send command to MOOSE process |
| `simulation_data` | Server → Client | Stream simulation frames |
| `disconnect` | Client → Server | Cleanup on disconnect |

### Simulation Launch Process

1. **Client sends JSON config** via `/launch_simulation`
2. **Server creates temp config file** with unique UUID
3. **Server spawns Python subprocess**:
   ```bash
   python -u -m jardesigner.jardesigner \
       <config_file> \
       --plotFile <svg_path> \
       --data-channel-id <uuid> \
       --session-path <session_dir>
   ```
4. **Server monitors subprocess** stdout/stderr with stream printers
5. **MOOSE process sends data** back via HTTP POST to `/internal/push_data`
6. **Server broadcasts data** to client via WebSocket room

### Session Management

- Each client gets a **unique UUID** (`clientId`)
- Session directory: `user_uploads/<clientId>/`
- Files uploaded by client stored in session directory
- On disconnect, session directory is **deleted automatically**
- Only one simulation per client at a time

---

## 6. Frontend Architecture

### Component Hierarchy

```
App (App.jsx)
└── AppLayout (AppLayout.jsx)
    ├── AppBar with Menu Buttons
    │   ├── File
    │   ├── SimOutput
    │   ├── Run
    │   ├── Morphology
    │   ├── Spines
    │   ├── Channels
    │   ├── Passive
    │   ├── Signaling
    │   ├── Adaptors
    │   ├── Stimuli
    │   ├── Plots
    │   └── 3D
    │
    └── DisplayWindow (DisplayWindow.jsx)
        ├── Menu Panels (conditionally rendered)
        │   └── [Corresponding MenuBox component]
        │
        └── Main Display Area
            ├── ThreeDViewer (Setup view)
            ├── ThreeDViewer (Run view)
            └── GraphWindow (SVG plots)
```

### State Management (`appLogic.js`)

JARDesigner uses a **custom hook pattern** for centralized state management (no Redux/MobX).

#### Core State Structure

```javascript
const useAppLogic = () => {
    // UI State
    const [activeMenu, setActiveMenu] = useState(null);
    
    // Model Data (JSON config)
    const [jsonData, setJsonData] = useState(initialJsonData);
    const [jsonContent, setJsonContent] = useState('...');
    
    // Simulation State
    const [isSimulating, setIsSimulating] = useState(false);
    const [activeSim, setActiveSim] = useState({
        pid: null,
        data_channel_id: null,
        svg_filename: null
    });
    
    // Session Management
    const [clientId] = useState(() => uuidv4());
    const socketRef = useRef(null);
    
    // Visualization State (keyed by viewId)
    const VIEW_IDS = { SETUP: 'setup', RUN: 'run' };
    const [threeDConfigs, setThreeDConfigs] = useState({
        [VIEW_IDS.SETUP]: null,
        [VIEW_IDS.RUN]: null
    });
    const [simulationFrames, setSimulationFrames] = useState({
        [VIEW_IDS.SETUP]: [],
        [VIEW_IDS.RUN]: []
    });
    const [clickSelected, setClickSelected] = useState({
        [VIEW_IDS.SETUP]: [],
        [VIEW_IDS.RUN]: []
    });
    
    // Plot State
    const [svgPlotFilename, setSvgPlotFilename] = useState(null);
    const [isPlotReady, setIsPlotReady] = useState(false);
    
    // Replay State
    const [replayTime, setReplayTime] = useState(0);
    const [isReplaying, setIsReplaying] = useState(false);
    
    // ... handlers and effects ...
    
    return { /* all state and handlers */ };
};
```

### Key State Management Patterns

**1. JSON Data Management**
- Central `jsonData` object holds entire model configuration
- Follows `jardesignerSchema.json` structure
- Changes propagated via `updateJsonData()` callback
- Compacted before saving (removes default values)

**2. Two-View System**
- **SETUP view**: Shows initial model structure
- **RUN view**: Shows live/replayed simulation
- Separate state for each view (3D config, frames, selections)

**3. WebSocket Communication**
```javascript
useEffect(() => {
    const socket = io(API_BASE_URL);
    socketRef.current = socket;
    
    socket.on('connect', () => {
        socket.emit('register_client', { clientId });
    });
    
    socket.on('simulation_data', (data) => {
        if (data.type === 'scene_graph') {
            // Initial 3D scene structure
            setThreeDConfigs(prev => ({
                ...prev,
                [VIEW_IDS.RUN]: data.sceneGraph
            }));
        } else if (data.type === 'data_frame') {
            // Simulation frame data
            frameQueueRef.current.push(data);
        }
    });
    
    return () => socket.disconnect();
}, []);
```

### React Component Patterns

**1. Menu Box Components**
- Each configures a specific aspect of the model
- Receives `currentConfig` prop with relevant JSON data
- Calls `onConfigurationChange(updates)` to modify model
- Includes built-in help documentation in JSON format

**2. Three.js Integration**
- `ThreeDViewer.jsx`: React wrapper and UI controls
- `ThreeDManager.js`: Pure Three.js scene management class
- `ShapeFactory.js`: Creates 3D geometries (cylinders, spheres, meshes)
- Imperative control via ref to manager instance

**3. Replay System**
- Separate hook: `useReplayLogic()`
- Manages playback of recorded simulation frames
- Interpolates between frames for smooth animation
- Syncs with plot display

---

## 7. Core Python Package

### `jardesigner.py` - Main Simulation Engine (2225 lines)

This is the heart of MOOSE integration. Key responsibilities:

#### Class Structure

```python
class Jardesigner:
    def __init__(self):
        self.model = moose.Neutral('/model')
        self.elecid = moose.Neutral('/model/elec')
        self.chemid = moose.Neutral('/model/chem')
        self.libraryId = moose.Neutral('/library')
        # ... many more MOOSE paths ...
    
    def buildModel(self, configDict):
        """Main entry point - builds entire model from JSON"""
        self._loadMorphology()
        self._buildPassiveDistrib()
        self._buildChanDistrib()
        self._buildChemProto()
        self._buildSpineProto()
        self._buildAdaptors()
        self._buildStims()
        self._buildPlots()
        # ... etc ...
    
    def run(self, runtime):
        """Execute simulation"""
        moose.reinit()
        moose.start(runtime)
```

#### Key Functions

**Morphology Loading**
```python
def _loadMorphology(self):
    """Load cell from SWC, NeuroML, or programmatic definition"""
    if cellType == 'file':
        if fname.endswith('.swc'):
            cell = moose.loadModel(fname, cellPath)
        elif fname.endswith('.nml'):
            cell = NeuroML().readNeuroMLFromFile(fname)
    elif cellType == 'soma':
        # Create single compartment
    elif cellType == 'ballAndStick':
        # Create soma + dendrite
```

**Chemical System Integration**
```python
def _buildChemProto(self):
    """Load and distribute chemical signaling models"""
    for proto in chemProtos:
        self._loadChem(proto['fname'], proto['name'])
    for distrib in chemDistribs:
        self._installChemProto(meshName, chemPath)
```

**Adaptor System** (Elec ↔ Chem Coupling)
```python
def _buildAdaptor(self, meshName, elecRelPath, elecField,
                  chemRelPath, chemField, isElecToChem, offset, scale):
    """
    Creates bidirectional coupling between electrical and chemical domains
    Examples:
    - Ca current → Ca concentration
    - Molecule concentration → channel conductance
    """
```

### `jarmoogli.py` - 3D Visualization Data Generator (727 lines)

Converts MOOSE simulation data into JSON structures for Three.js rendering.

#### Key Classes

**`FieldInfo`** - Defines displayable fields
```python
class FieldInfo:
    def __init__(self, relpath, field, title, colormap):
        self.relpath = relpath      # Path in MOOSE tree
        self.field = field          # Field name (Vm, conc, etc.)
        self.title = title          # Display title
        self.colormap = colormap    # Color mapping
        self.vmin, self.vmax = ...  # Value range
```

**`Drawable`** - Renderable entity
```python
class Drawable:
    def __init__(self, entityName, meshType, drawOrder):
        self.entityName = entityName
        self.meshType = meshType    # 'cylinder', 'sphere', 'mesh'
        self.objList = []           # MOOSE objects
        self.coords = []            # 3D coordinates
        self.values = []            # Field values
    
    def getDataFrame(self, timestamp):
        """Returns current values for coloring"""
        return [obj.getField(self.field) for obj in self.objList]
```

#### Data Streaming

**Scene Graph** (sent once at start):
```json
{
    "type": "scene_graph",
    "sceneGraph": {
        "drawables": [
            {
                "entityName": "elec",
                "meshType": "cylinder",
                "numElements": 156,
                "coords": [[x0,y0,z0,x1,y1,z1,dia], ...],
                "paths": ["soma", "dend[0]", ...],
                "drawOrder": 0,
                "fieldInfo": {
                    "field": "Vm",
                    "dataUnits": "mV",
                    "vmin": -80.0,
                    "vmax": 40.0
                }
            }
        ],
        "bbox": {"xmin": ..., "xmax": ..., ...}
    }
}
```

**Data Frames** (sent periodically during simulation):
```json
{
    "type": "data_frame",
    "timestamp": 0.05,
    "data": {
        "elec": [v0, v1, v2, ...],
        "Ca_pool": [c0, c1, c2, ...]
    }
}
```

### Data Push Mechanism

```python
def pushFrameToFlask(data_channel_id, payload):
    """Send data to Flask server, which broadcasts via WebSocket"""
    requestBody = {
        'data_channel_id': data_channel_id,
        'payload': payload
    }
    http_session.post(FLASK_SERVER_URL, json=requestBody, timeout=0.5)
```

---

## 8. Communication Flow

### Complete Request-Response Cycle

#### A. Model Building (Setup Phase)

```
USER                    FRONTEND              FLASK             MOOSE
 │                         │                   │                 │
 │ 1. Configure Model      │                   │                 │
 │ ──────────────────────> │                   │                 │
 │                         │                   │                 │
 │                         │ 2. Update State   │                 │
 │                         │ ─────────────────>│                 │
 │                         │                   │                 │
 │ 3. Load Morphology      │                   │                 │
 │ ──────────────────────> │                   │                 │
 │                         │                   │                 │
 │                         │ 4. Upload File    │                 │
 │                         │ ────────(POST)───>│                 │
 │                         │                   │ Store in        │
 │                         │    <──(200 OK)────│ user_uploads/   │
 │                         │                   │                 │
 │ 5. Click "Build Model"  │                   │                 │
 │ ──────────────────────> │                   │                 │
 │                         │                   │                 │
 │                         │ 6. Launch Build   │                 │
 │                         │ ────(POST JSON)──>│                 │
 │                         │                   │                 │
 │                         │                   │ 7. Spawn Process│
 │                         │                   │ ───────────────>│
 │                         │                   │                 │
 │                         │   <─(PID+channel)─│   python -m     │
 │                         │                   │   jardesigner   │
 │                         │ 8. Join Channel   │                 │
 │                         │ ──(WebSocket)────>│                 │
 │                         │                   │                 │
 │                         │                   │   <─(builds)────│
 │                         │                   │                 │
 │                         │                   │ 9. Scene Graph  │
 │                         │                   │ <───(HTTP POST)─│
 │                         │                   │                 │
 │                         │ 10. Scene Data    │                 │
 │                         │ <──(WebSocket)────│                 │
 │                         │                   │                 │
 │ 11. Render 3D View      │                   │                 │
 │ <───────────────────────│                   │                 │
```

#### B. Simulation Execution

```
USER                    FRONTEND              FLASK             MOOSE
 │                         │                   │                 │
 │ 1. Click "Run"          │                   │                 │
 │ ──────────────────────> │                   │                 │
 │                         │                   │                 │
 │                         │ 2. Send Command   │                 │
 │                         │ ──(WebSocket)────>│                 │
 │                         │                   │                 │
 │                         │                   │ 3. Forward Cmd  │
 │                         │                   │ ──(stdin)──────>│
 │                         │                   │                 │
 │                         │                   │   {"command":   │
 │                         │                   │    "run"}       │
 │                         │                   │                 │
 │                         │                   │                 │
 │                         │                   │   <─(runs sim)──│
 │                         │                   │                 │
 │                         │                   │   Periodically: │
 │                         │                   │ 4. Data Frames  │
 │                         │                   │ <───(HTTP POST)─│
 │                         │                   │                 │
 │                         │ 5. Broadcast      │                 │
 │                         │ <──(WebSocket)────│                 │
 │                         │                   │                 │
 │ 6. Update 3D Colors     │                   │                 │
 │ <───────────────────────│                   │                 │
 │                         │                   │                 │
 │                         │                   │   <─(completes)─│
 │                         │                   │                 │
 │                         │                   │ 7. SVG Plot     │
 │                         │                   │ <───(saves file)│
 │                         │                   │                 │
 │ 8. Request Plot         │                   │                 │
 │ ──────────────────────> │                   │                 │
 │                         │                   │                 │
 │                         │ 9. Get SVG        │                 │
 │                         │ ────(HTTP GET)───>│                 │
 │                         │                   │                 │
 │                         │   <─(SVG file)────│                 │
 │                         │                   │                 │
 │ 10. Display Plot        │                   │                 │
 │ <───────────────────────│                   │                 │
```

### WebSocket Rooms

JARDesigner uses **WebSocket rooms** for targeted broadcasting:

1. Client connects and gets a **socket ID** (SID)
2. Client emits `register_client` with its **client ID** (UUID)
3. Server maps `SID → client_id`
4. On simulation launch, server creates **data channel ID**
5. Client emits `join_sim_channel` with data channel ID
6. Client joins that **room**
7. MOOSE sends data to Flask with channel ID
8. Flask broadcasts to **room**, reaching only that client

This allows multiple clients to run independent simulations simultaneously.

---

## 9. Data Models

### JSON Configuration Schema

The model is defined by a JSON object following `jardesignerSchema.json`:

```json
{
  "filetype": "jardesigner",
  "version": "1.0",
  "fileinfo": {
    "creator": "User Name",
    "modelNotes": "Description",
    "licence": "CC BY"
  },
  "modelPath": "/model",
  "diffusionLength": 2e-6,
  "turnOffElec": false,
  "useGssa": false,
  "odeMethod": "lsoda",
  "runtime": 0.3,
  "elecDt": 50e-6,
  "chemDt": 0.1,
  "temperature": 32,
  "cellProto": {
    "type": "file",
    "source": "h10.CNG.swc"
  },
  "passiveDistrib": [
    {
      "path": "/##",
      "RM": 1.0,
      "CM": 0.01,
      "RA": 1.0,
      "Em": -0.065,
      "initVm": -0.065
    }
  ],
  "chanProto": [
    {
      "source": "Ca.xml",
      "name": "Ca",
      "type": "channel_file"
    }
  ],
  "chanDistrib": [
    {
      "path": "/##",
      "chan": "Ca",
      "Gbar": 1e-6
    }
  ],
  "spineProto": [
    {
      "name": "spine1",
      "shaftLen": 1e-6,
      "shaftDia": 0.2e-6,
      "headLen": 0.5e-6,
      "headDia": 0.5e-6
    }
  ],
  "spineDistrib": [
    {
      "path": "/##",
      "proto": "spine1",
      "spacing": 2e-6
    }
  ],
  "chemProto": [
    {
      "fname": "chanPhosphByCaMKII.g",
      "name": "chem"
    }
  ],
  "chemDistrib": [
    {
      "path": "/##",
      "proto": "chem"
    }
  ],
  "adaptors": [
    {
      "elecpath": ".",
      "elecfield": "Ik",
      "chempath": "Ca",
      "chemfield": "conc",
      "offset": 0,
      "scale": 5e-9
    }
  ],
  "stims": [
    {
      "path": "/soma",
      "geom": "soma",
      "field": "inject",
      "value": 1e-9,
      "onset": 0.1,
      "duration": 0.1
    }
  ],
  "plots": [
    {
      "path": "/soma",
      "geom": "soma",
      "field": "Vm",
      "title": "Soma Vm"
    }
  ],
  "moogli": [
    {
      "relpath": ".",
      "field": "Vm",
      "title": "Membrane Potential"
    }
  ]
}
```

### Schema Validation

- Uses **JSONSchema Draft 07**
- Validated client-side with **AJV**
- Validated server-side with **jsonschema** Python library
- Provides default values for missing fields
- Supports `oneOf` for polymorphic types (e.g., cell prototypes)

### Key Schema Sections

| Section | Purpose |
|---------|---------|
| `cellProto` | Defines neuronal morphology |
| `passiveDistrib` | Sets passive electrical properties |
| `chanProto` + `chanDistrib` | Adds ion channels |
| `spineProto` + `spineDistrib` | Adds dendritic spines |
| `chemProto` + `chemDistrib` | Integrates chemical models |
| `adaptors` | Links electrical ↔ chemical domains |
| `stims` | Defines stimulation protocols |
| `plots` | Configures data recording |
| `moogli` | Specifies 3D visualization fields |

---

## 10. 3D Visualization System

### Three.js Integration

#### Architecture

```
React Component (ThreeDViewer.jsx)
         │
         │ ref
         ▼
  ThreeDManager (class)
         │
         ├── Scene
         ├── Camera (OrbitControls)
         ├── Renderer
         └── Drawables
              │
              └── ShapeFactory
                   │
                   ├── createCylinder()
                   ├── createSphere()
                   └── createMesh()
```

### ThreeDManager Class

```javascript
class ThreeDManager {
    constructor(containerElement, viewId) {
        this.scene = new THREE.Scene();
        this.camera = new THREE.PerspectiveCamera(45, ...);
        this.renderer = new THREE.WebGLRenderer();
        this.controls = new OrbitControls(this.camera, ...);
        this.drawables = {};  // entityName -> THREE.Object3D
    }
    
    initializeScene(sceneGraph) {
        // Convert JSON scene graph to Three.js objects
        for (const drawable of sceneGraph.drawables) {
            const group = new THREE.Group();
            
            if (drawable.meshType === 'cylinder') {
                // Create cylinder for each coordinate pair
                drawable.coords.forEach((coord, i) => {
                    const cyl = createCylinder(coord, color);
                    group.add(cyl);
                });
            }
            
            this.scene.add(group);
            this.drawables[drawable.entityName] = {
                group, coords, fieldInfo
            };
        }
    }
    
    updateFrame(frameData) {
        // Update colors based on field values
        for (const [entityName, values] of Object.entries(frameData)) {
            const drawable = this.drawables[entityName];
            const { vmin, vmax } = drawable.fieldInfo;
            
            drawable.group.children.forEach((mesh, i) => {
                const value = values[i];
                const normalized = (value - vmin) / (vmax - vmin);
                const color = getColor(normalized, colormap);
                mesh.material.color.set(color);
            });
        }
        this.renderer.render(this.scene, this.camera);
    }
}
```

### Mesh Types

**1. Cylinder Meshes** (Neurons)
- Represents compartments as cylinders
- Coordinates: `[x0, y0, z0, x1, y1, z1, diameter]`
- Used for soma, dendrites, axons

**2. Sphere Meshes** (Spines, Molecules)
- Point objects
- Coordinates: `[x, y, z, diameter]`
- Used for spine heads, molecular pools

**3. Indexed Meshes** (Complex Geometries)
- Arbitrary triangular meshes
- Vertices + face indices
- Used for PSDs, endoplasmic reticulum

### Color Mapping

```javascript
function getColor(normalizedValue, colormapName) {
    // Maps 0.0-1.0 to RGB color
    // Supports: 'jet', 'hot', 'cool', 'viridis', etc.
    const colormap = colormaps[colormapName];
    const index = Math.floor(normalizedValue * (colormap.length - 1));
    return colormap[index];
}
```

### Interactive Features

- **Click Selection**: Raycasting to select objects
- **Hover Tooltips**: Show object path and current value
- **Camera Controls**: Orbit, pan, zoom with OrbitControls
- **Explode View**: Separate components along axes
- **Visibility Toggle**: Show/hide individual drawables
- **Color Range Adjustment**: Interactive vmin/vmax tuning

---

## 11. Simulation Workflow

### Step-by-Step Execution

#### Phase 1: Model Building (SETUP View)

```
1. User configures model via menu boxes
   └─> Updates jsonData state

2. User clicks "Build Model" button
   └─> Triggers buildModel() in RunMenuBox

3. Frontend sends POST to /launch_simulation
   ├─> JSON config
   ├─> clientId
   └─> subprocess=False, run=False

4. Flask spawns MOOSE process:
   python -m jardesigner.jardesigner config.json \
       --plotFile plot.svg \
       --data-channel-id <uuid> \
       --session-path <dir>

5. MOOSE jardesigner.py:
   ├─> Loads morphology
   ├─> Builds electrical system
   ├─> Builds chemical system
   ├─> Creates adaptors
   └─> Generates scene graph JSON

6. MOOSE sends scene graph to Flask via POST /internal/push_data

7. Flask broadcasts scene graph to client via WebSocket

8. Frontend renders 3D model in SETUP view
   └─> User can inspect structure, check connectivity
```

#### Phase 2: Simulation Execution (RUN View)

```
9. User clicks "Run Simulation" button
   └─> Triggers handleStartRun() in appLogic

10. Frontend sends WebSocket event: sim_command
    ├─> pid: <process_id>
    └─> command: "run"

11. Flask forwards command to MOOSE stdin as JSON

12. MOOSE jardesigner.py receives command:
    ├─> Calls moose.reinit()
    ├─> Calls moose.start(runtime)
    └─> Periodically sends data frames

13. During simulation, MOOSE sends frames:
    Every chemPlotDt or elecPlotDt:
    ├─> Collects current field values
    ├─> Posts to Flask /internal/push_data
    └─> Flask broadcasts via WebSocket

14. Frontend receives frames:
    ├─> Queues frames in frameQueueRef
    ├─> Processes queue with requestAnimationFrame
    └─> Updates 3D colors in RUN view

15. On completion, MOOSE:
    ├─> Generates SVG plot with matplotlib
    ├─> Saves to session directory
    └─> Exits process

16. Frontend polls simulation_status endpoint

17. When completed, frontend fetches SVG plot

18. User can replay simulation with controls
```

### Command Protocol

MOOSE process listens on stdin for JSON commands:

```json
{"command": "run", "params": {}}
{"command": "pause", "params": {}}
{"command": "reset", "params": {}}
```

Flask forwards these from WebSocket `sim_command` events.

---

## 12. Key Features

### 1. Real-Time Streaming Simulation

- Data streamed via WebSocket during simulation
- Low-latency visualization updates
- Framerates up to 60fps for electrical signals
- Buffered delivery for chemical signals

### 2. Replay System

After simulation completes:
- All frames stored in memory
- **Play/Pause/Rewind** controls
- **Seek bar** for random access
- **Speed control** for replay rate
- Synchronized with plot display

### 3. Multi-View Architecture

**SETUP View**
- Shows initial model structure
- Static geometry
- Used for model validation

**RUN View**
- Shows live or replayed simulation
- Dynamic coloring based on field values
- Time-synchronized with plots

### 4. File Handling

**Supported Morphology Formats**
- SWC (standard neuron morphology)
- NeuroML (XML-based)
- Programmatic (soma, ball-and-stick, Y-branch)

**Supported Chemical Formats**
- GENESIS .g files
- SBML (Systems Biology Markup Language)

**Supported Channel Formats**
- ChannelML (XML)
- GENESIS .p files

### 5. Dual-Mode Deployment

**Server Mode** (Default)
- Frontend and backend separate
- WebSocket communication
- Multi-user support
- Development friendly

**Standalone Mode**
- Single HTML file with embedded data
- No backend required
- Pre-simulated results
- Shareable, portable

### 6. Help System

Each menu box includes contextual help:
- Stored in `.Help.json` files
- Markdown formatted
- Rendered with `react-markdown`
- Explains parameters and usage

---

## 13. Deployment Modes

### Development Mode

**Frontend:**
```bash
cd frontend
npm install
npm run dev
# Runs on http://localhost:5173
```

**Backend:**
```bash
cd backend
pip install -r requirements.txt
python server.py
# Runs on http://localhost:5000
```

### Production Build

**Frontend:**
```bash
cd frontend
npm run build
# Outputs to frontend/dist/
```

Serve with Flask:
```python
@app.route('/jardesigner')
def serve_frontend():
    return send_from_directory('frontend/dist', 'index.html')
```

### Standalone Build

```bash
cd frontend
npm run build:standalone
# Creates single-file HTML in dist/standalone.html
```

Embed scene and frame data:
```javascript
window.__JARDESIGNER_SCENE_CONFIG__ = { /* scene graph */ };
window.__JARDESIGNER_SIMULATION_FRAMES__ = [ /* frames */ ];
```

### Package Installation Mode

JARDesigner can be pip-installed:

```bash
pip install jardesigner
jardesigner  # Launches browser with local server
```

This uses:
- `SERVER_CONFIG/jardesigner` script
- `run_jardesigner.py` as entry point
- Bundled frontend in package data

---

## Summary

**JARDesigner** is a sophisticated web application for multiscale neural simulation with:

 **Modern Web Stack**: React + Vite + Three.js frontend  
 **Powerful Backend**: Flask + SocketIO + MOOSE engine  
 **Real-Time Visualization**: WebSocket streaming + 3D rendering  
 **Flexible Architecture**: Supports standalone and server modes  
 **Comprehensive Features**: Build → Simulate → Visualize → Analyze  
 **Scientific Rigor**: Schema-validated, MOOSE-powered, research-grade  

The architecture separates concerns cleanly:
- **UI** (React) handles user interaction and rendering
- **Backend** (Flask) orchestrates processes and routes data
- **Engine** (MOOSE) performs heavy numerical computation

Communication flows seamlessly via HTTP + WebSocket, with JSON as the universal data interchange format.

This design makes JARDesigner both user-friendly for neuroscientists and maintainable for developers.
