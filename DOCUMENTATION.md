# MARK XXX - Technical Documentation

## Overview

**MARK XXX** is a real-time, voice-driven AI assistant for Windows that enables hands-free computer control through natural language. Inspired by J.A.R.V.I.S. from the Iron Man movies, it provides a conversational interface that can hear, see, understand, and control the user's computer.

- **Author**: FatihMakes
- **License**: Creative Commons BY-NC 4.0 (Personal and non-commercial use only)
- **Platform**: Windows 10/11
- **Language**: Python 3.10+

---

## Architecture

The application follows a modular architecture with clear separation of concerns:

```
Mark-XXX/
├── main.py              # Entry point and main event loop
├── ui.py                # Tkinter-based GUI with animated visuals
├── setup.py             # Installation script
├── requirements.txt     # Python dependencies
├── core/
│   └── prompt.txt      # System prompt for JARVIS personality
├── config/
│   ├── __init__.py
│   └── api_keys.json   # Gemini API key storage
├── memory/
│   ├── __init__.py
│   ├── memory_manager.py    # Long-term memory persistence
│   └── config_manager.py    # Configuration management
├── agent/
│   ├── task_queue.py        # Multi-step task queuing system
│   ├── executor.py          # Task execution engine
│   ├── planner.py           # AI-powered task planning
│   └── error_handler.py     # Error recovery and replanning
└── actions/              # Individual capability modules
    ├── open_app.py
    ├── web_search.py
    ├── browser_control.py
    ├── file_controller.py
    ├── computer_control.py
    ├── computer_settings.py
    ├── screen_processor.py
    ├── code_helper.py
    ├── dev_agent.py
    ├── cmd_control.py
    ├── desktop.py
    ├── send_message.py
    ├── reminder.py
    ├── weather_report.py
    ├── youtube_video.py
    ├── flight_finder.py
    └── send_message.py
```

---

## Core Components

### 1. Main Controller (`main.py`)

The central orchestrator that:
- Establishes a live audio connection with Google's Gemini API (Live API)
- Manages audio I/O streams (microphone input, speaker output)
- Handles tool calling and execution
- Manages memory updates asynchronously
- Implements automatic reconnection logic

**Key Classes:**
- `JarvisLive`: Main controller class managing the live session

**Key Features:**
- Real-time audio streaming with 16kHz input / 24kHz output
- Session resumption for maintaining context
- Asynchronous memory processing every 5 turns
- Thread-safe speech output from any thread

### 2. User Interface (`ui.py`)

A custom Tkinter-based GUI featuring:
- Animated circular avatar with dynamic scaling and halo effects
- Rotating arc segments simulating JARVIS-style visualization
- Audio visualizer bars that react to speech
- Chat log with typewriter effect
- API key initialization dialog
- Status indicators (INITIALISING, ONLINE, PROCESSING, SPEAKING)

**Key Visual Elements:**
- Pulsing rings with variable opacity
- Scanning arc animations
- Tick marks around the central orb
- Crosshair targeting lines
- Blinking status indicator

### 3. Memory System (`memory/`)

**Long-term Memory (`memory_manager.py`):**
- Persistent JSON storage of user facts and preferences
- Thread-safe read/write operations
- Structured memory format:
  ```json
  {
    "identity": { "name", "age", "city", "birthday" },
    "preferences": { "hobbies", "interests" },
    "relationships": { "family", "contacts" },
    "notes": { "miscellaneous facts" }
  }
  ```
- Automatic extraction using Gemini AI (every 5 conversation turns)
- Two-stage extraction: YES/NO check followed by full extraction
- Prevents redundant API calls (~80% reduction)

### 4. Agent System (`agent/`)

**Task Queue (`task_queue.py`):**
- Priority-based task scheduling (HIGH, NORMAL, LOW)
- Asynchronous task execution in background threads
- Task status tracking (PENDING, RUNNING, COMPLETED, FAILED, CANCELLED)
- Cancellation support via threading events

**Planner (`planner.py`):**
- Breaks complex goals into executable steps
- Uses Gemini Flash Lite for fast planning
- Supports 15+ tool types
- Automatic fallback to web search if planning fails
- Replans on failure with context of completed steps

**Executor (`executor.py`):**
- Executes planned steps sequentially
- Error recovery with retry logic (3 attempts per step)
- Context injection for multi-step workflows
- Language detection and translation support
- Generates dynamic code for unsupported tasks
- Summarizes completed tasks using AI

**Error Handler (`error_handler.py`):**
- AI-powered error analysis
- Four recovery strategies: RETRY, SKIP, REPLAN, ABORT
- Automatic fix generation using alternative approaches
- Critical step detection (cannot skip critical steps)

---

## Action Modules

The application includes 15+ specialized action modules:

### Application Control
- **`open_app.py`**: Launches applications by name using Windows registry lookups
- **`computer_settings.py`**: Controls system settings (volume, brightness, WiFi, etc.)

### File Management
- **`file_controller.py`**: Complete file operations (CRUD, move, copy, find, disk usage)
- **`desktop.py`**: Desktop organization, wallpaper changes, cleaning

### Web & Browser
- **`browser_control.py`**: Playwright-based browser automation
  - Auto-detects default browser (Chrome, Firefox, Edge, Opera, Brave, Vivaldi)
  - Supports navigation, clicking, typing, scrolling, form filling
- **`web_search.py`**: DuckDuckGo-based web search
- **`youtube_video.py`**: YouTube control, video summarization, trending videos

### Communication
- **`send_message.py`**: Messaging via WhatsApp Web, Telegram
- **`reminder.py`**: Windows Task Scheduler integration for reminders

### Development
- **`code_helper.py`**: AI-powered code assistance
  - Write, edit, explain, run, build, optimize code
  - Multi-language support (Python, JS, Java, C++, etc.)
  - Self-correcting build loop (max 3 attempts)
- **`dev_agent.py`**: Full project scaffolding
  - Creates multi-file projects
  - Installs dependencies
  - Opens VS Code
  - Runs and validates the project

### System Control
- **`computer_control.py`**: Direct computer control via PyAutoGUI
  - Mouse: click, double-click, right-click, scroll, move
  - Keyboard: type, hotkeys, key presses
  - Window: focus, minimize, maximize, close
  - Screen: screenshots, element detection
  - Data generation: random names, emails, passwords
  - Form filling: user profile data from memory
- **`cmd_control.py`**: Natural language to command translation

### Information
- **`weather_report.py`**: Weather information retrieval
- **`flight_finder.py`**: Google Flights integration
- **`screen_processor.py`**: Visual analysis using Gemini Vision
  - Screen capture and analysis
  - Webcam image understanding
  - Object detection and recognition

---

## Technical Specifications

### Audio Configuration
```python
FORMAT              = pyaudio.paInt16
CHANNELS            = 1
SEND_SAMPLE_RATE    = 16000    # Microphone input
RECEIVE_SAMPLE_RATE = 24000    # Speaker output
CHUNK_SIZE          = 1024
```

### AI Models Used

| Component | Model | Purpose |
|-----------|-------|---------|
| Live Voice | `gemini-2.5-flash-native-audio-preview-12-2025` | Real-time voice conversation |
| Planning | `gemini-2.5-flash-lite` | Fast task planning |
| Error Analysis | `gemini-2.5-flash-lite` | Error recovery decisions |
| Memory Extraction | `gemini-2.5-flash-lite` | Fact extraction |
| Code Generation | `gemini-2.5-flash` | Code writing and fixing |
| Translation | `gemini-2.5-flash` | Multi-language support |
| Vision | `gemini-2.5-flash` | Screen/webcam analysis |
| Summarization | `gemini-2.5-flash-lite` | Task completion summaries |

### Dependencies

**Core:**
- `pyaudio` - Audio I/O
- `google-genai`, `google-generativeai` - Gemini API
- `pillow` - Image processing

**Browser Automation:**
- `playwright` - Browser control

**System Control:**
- `pyautogui` - Mouse/keyboard control
- `pyperclip` - Clipboard operations
- `opencv-python` - Computer vision
- `mss` - Screen capture
- `psutil` - System utilities
- `pycaw`, `comtypes` - Windows audio control
- `win10toast` - Windows notifications

**Utilities:**
- `requests`, `beautifulsoup4` - Web scraping
- `duckduckgo-search` - Search functionality
- `youtube-transcript-api` - YouTube transcript extraction
- `send2trash` - Safe file deletion

---

## Security Considerations

1. **API Key Storage**: Stored in local JSON file (`config/api_keys.json`)
2. **Execution Scope**: Can execute arbitrary code generated by AI
3. **System Access**: Full control over Windows via PyAutoGUI
4. **Browser Access**: Can automate browser actions including authentication
5. **Memory Persistence**: Stores user facts locally in JSON format

---

## Workflow Examples

### Simple Command Flow
```
User Voice Input
       ↓
Gemini Live API (streaming audio)
       ↓
Intent Recognition → Tool Selection
       ↓
Tool Execution (e.g., open_app)
       ↓
Audio Response (Charon voice)
```

### Complex Task Flow
```
User: "Research Bitcoin and save to a file"
       ↓
agent_task tool triggered
       ↓
Planner: Creates 3-step plan
  1. web_search (Bitcoin info)
  2. file_controller (write to file)
  3. cmd_control (open file)
       ↓
Executor runs each step
       ↓
Error handler manages any failures
       ↓
Summarizer creates completion message
       ↓
Voice response to user
```

---

## Configuration

### Initial Setup
1. Run `python setup.py` to install dependencies and Playwright browsers
2. Run `python main.py`
3. Enter Gemini API key in the initialization dialog
4. System boots and displays animated JARVIS interface

### File Locations
- **API Keys**: `config/api_keys.json`
- **Memory**: `memory/long_term.json`
- **System Prompt**: `core/prompt.txt`

---

## Extensibility

The modular architecture allows easy addition of new capabilities:

1. Create new action module in `actions/`
2. Define tool declaration in `main.py` (`TOOL_DECLARATIONS`)
3. Add execution handler in `JarvisLive._execute_tool()`
4. Update planner prompt in `agent/planner.py`
5. Add tool mapping in `agent/executor.py` `_call_tool()`

---

## Performance Optimizations

- **Async Memory Processing**: Memory extraction happens in background threads
- **Two-Stage Memory Extraction**: Prevents unnecessary API calls
- **Task Queue**: Non-blocking execution of complex tasks
- **Audio Streaming**: Real-time without buffering delays
- **Session Resumption**: Maintains context across reconnections
- **Translation Caching**: Goal language detection cached per task

---

## Limitations

- **Windows Only**: Many features rely on Windows-specific APIs
- **Gemini Dependency**: Requires Google Gemini API key
- **English Focus**: Tool parameters extracted in English, though responses can be multilingual
- **Security**: Full system access could be misused if compromised
- **Browser Detection**: Relies on Windows registry for default browser detection

---

## Future Enhancements

Potential areas for expansion:
- Cross-platform support (macOS, Linux)
- Plugin system for third-party extensions
- Voice training for personalized speech synthesis
- Offline mode with local LLM fallback
- Cloud synchronization of memories
- Mobile companion app

---

## Summary

MARK XXX represents a sophisticated personal AI assistant that combines real-time voice interaction with comprehensive system control. Its modular architecture, error recovery systems, and persistent memory create an experience that closely mirrors fictional AI assistants while maintaining practical utility for everyday computer tasks.

The application's strength lies in its ability to handle both simple commands ("open Chrome") and complex multi-step workflows ("research a topic, compile findings, and save to a document") through its agent system, making it a capable automation tool for knowledge workers.
