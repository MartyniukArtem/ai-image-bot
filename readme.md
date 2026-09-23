View Demo video and project presentation on behance: https://www.behance.net/gallery/256126063/Telegram-AI-Bot-with-Image-Generation-via-n8n-ComfyUI

# 🤖 AI Image Bot — Telegram + ComfyUI + n8n
> Telegram bot for AI image generation powered by ComfyUI, orchestrated through n8n workflows with conversational AI capabilities.
![Project Status](https://img.shields.io/badge/status-completed-success)
![License](https://img.shields.io/badge/license-MIT-blue)
---
## 🎯 Features
- 🎨 **Text-to-Image Generation** — ComfyUI with Z-Image Turbo model
- 🤖 **Conversational AI** — Claude API integration for natural language interaction
- ⚡ **Automated Workflows** — n8n orchestration for seamless integration
- 🛡️ **Robust Error Handling** — Multi-level checks and user-friendly feedback
- 📊 **Real-time Status Updates** — Live generation progress messages
- 💬 **Hybrid Interface** — Image generation + AI chat in one bot
---
## 🛠️ Tech Stack
| Component | Technology |
|-----------|-----------|
| **Workflow Automation** | n8n |
| **Image Generation** | ComfyUI (Z-Image Turbo) |
| **Conversational AI** | Claude API (Anthropic) |
| **User Interface** | Telegram Bot API |
| **GPU Infrastructure** | RunPod |
| **Containerization** | Docker |
| **Tunneling** | ngrok |
---
## 📋 Bot Commands
| Command | Description | Response Time |
|---------|-------------|---------------|
| `/image <prompt>` | Generate AI image from text | 20-30 sec |
| `/help` | Show available commands | Instant |
| `/reset` | Clear conversation history | Instant |
| Regular message | Chat with Claude AI | 2-5 sec |
**Example:**
/image beautiful sunset over mountains, vibrant colors, 4k

TEXT
---
## 🏗️ Architecture
### System Overview
![Workflow Architecture](screenshots/workflow.png)
### Data Flow
┌─────────────┐
│ Telegram │
│ User │
└──────┬──────┘
│ /image
▼
┌─────────────────────────────────┐
│ n8n Workflow │
│ ┌──────────────────────────┐ │
│ │ 1. Extract Prompt │ │
│ └────────┬─────────────────┘ │
│ │ │
│ ┌────────▼─────────────────┐ │
│ │ 2. Build Workflow JSON │ │
│ │ (dynamic prompt + │ │
│ │ random seed) │ │
│ └────────┬─────────────────┘ │
│ │ │
│ ┌────────▼─────────────────┐ │
│ │ 3. Send "Generating..." │ │
│ └────────┬─────────────────┘ │
└───────────┼─────────────────────┘
│ HTTP POST /prompt
▼
┌─────────────────────────────────┐
│ ComfyUI API (RunPod GPU) │
│ ┌──────────────────────────┐ │
│ │ Z-Image Turbo Model │ │
│ │ (15-30 sec processing) │ │
│ └────────┬─────────────────┘ │
└───────────┼─────────────────────┘
│ prompt_id
▼
┌─────────────────────────────────┐
│ n8n Workflow │
│ ┌──────────────────────────┐ │
│ │ 4. Wait 15 seconds │ │
│ └────────┬─────────────────┘ │
│ │ │
│ ┌────────▼─────────────────┐ │
│ │ 5. GET /history │ │
│ │ Check status │ │
│ └────────┬─────────────────┘ │
│ │ │
│ ┌────────▼─────────────────┐ │
│ │ 6. GET /view (image) │ │
│ └────────┬─────────────────┘ │
└───────────┼─────────────────────┘
│ Binary image data
▼
┌─────────────┐
│ Telegram │
│ Send Photo │
└─────────────┘

TEXT
### Error Handling Strategy
```javascript
// Level 1: Service Availability
IF (ComfyUI responds) 
  → Continue
ELSE 
  → "⚠️ Image service temporarily unavailable"
// Level 2: Generation Success
IF (image generated successfully)
  → Send photo
ELSE 
  → "❌ Generation failed, please try again"
// Level 3: Request-level
All HTTP nodes: "Continue On Fail" = true
🎬 Demo
Bot in Action
[Image blocked: Help Command]
Available commands overview

[Image blocked: Generation Process]
Real-time status feedback

[Image blocked: Generated Result]
Final AI-generated image

🚀 How to Run
Prerequisites
✅ Docker installed
✅ ngrok account (free tier)
✅ RunPod account with credits
✅ Telegram Bot Token (create via @BotFather)
✅ Claude API key (get from Anthropic)
Step 1: Start n8n
BASH
docker run -d \
  --name n8n \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n
Access n8n at: http://localhost:5678

Step 2: Expose n8n with ngrok
BASH
ngrok http 5678
Copy the HTTPS URL (e.g., https://abc123.ngrok.io)

Step 3: Set Telegram Webhook
BASH
curl -X POST "https://api.telegram.org/bot<YOUR_BOT_TOKEN>/setWebhook?url=<NGROK_URL>/webhook/telegram"
Example:

BASH
curl -X POST "https://api.telegram.org/bot123456:ABC-DEF/setWebhook?url=https://abc123.ngrok.io/webhook/telegram"
Step 4: Start RunPod Pod
Go to RunPod Console
Deploy ComfyUI template
Select GPU (RTX 3090 recommended)
Wait for pod to start
Copy API endpoint URL (e.g., https://xyz-pod.runpod.net)
Step 5: Import & Configure n8n Workflow
Import workflow:
Open n8n
Settings → Import from File
Select workflow.json
Update 3 HTTP Request nodes:
"Send Prompt to ComfyUI": Update base URL
"Check Generation Status": Update base URL
"Get Generated Image": Update base URL
Replace YOUR_RUNPOD_URL with actual RunPod endpoint
Add Telegram credentials:
Telegram Trigger node → Credentials
Add Bot Token
Add Claude API key:
HTTP Request node (Claude) → Authentication
Add API key in header: x-api-key
Activate workflow ✅
Step 6: Test!
Open Telegram and send:

TEXT
/help
Then try:

TEXT
/image a cyberpunk city at night, neon lights, rain
📊 Performance Metrics
Metric	Value
Average generation time	20-30 seconds
Success rate	~95% (with error handling)
Concurrent users	Limited by RunPod GPU availability
Image resolution	512x512 (Z-Image Turbo default)
💡 Technical Highlights
1. Dynamic Workflow Generation
Instead of static JSON templates, workflows are built dynamically:

JAVASCRIPT
// Code node: Build ComfyUI Workflow
const prompt = $input.first().json.message.text.replace('/image ', '');
const workflow = {
  "67": {
    "class_type": "CLIPTextEncode",
    "inputs": {
      "text": prompt,  // ← Dynamic user input
      "clip": ["68", 0]
    }
  },
  "70": {
    "class_type": "KSampler",
    "inputs": {
      "seed": Math.floor(Math.random() * 999999999999),  // ← Random seed each time
      "steps": 4,
      "cfg": 1.0,
      // ...
    }
  }
}
return { json: { prompt: workflow } };
Benefits:

✅ Unique generation each time
✅ Any text prompt supported
✅ Easy to extend (add parameters)
2. Multi-Level Error Handling
Problem: ComfyUI might be offline, or generation might fail

Solution: 3-layer error handling

TEXT
Layer 1: Service Check
  ↓
Layer 2: Generation Validation
  ↓
Layer 3: Request-Level Fallbacks
Implementation:

IF nodes check responses before proceeding
"Continue On Fail" on all HTTP requests
User-friendly error messages
Automatic fallback to error notifications
3. Stateful Conversation Memory
JAVASCRIPT
// Store conversation history globally
let conversationHistory = [];
// On new message
conversationHistory.push({
  role: "user",
  content: userMessage
});
// On /reset
conversationHistory = [];
Result: Claude remembers context across messages

4. Asynchronous Job Handling
Challenge: Image generation takes 15-30 seconds

Approach:

Submit job → Get prompt_id
Wait 15 seconds (Wait node)
Poll /history endpoint
Extract image from response
Fetch binary data via /view
Why not polling loop?

Fixed wait is simpler for demo
Z-Image Turbo is consistently fast (~20 sec)
Production version would use polling
🔮 Future Improvements
Planned features:

 Multi-model support
Flux, SDXL, Stable Diffusion 3
Model selection via inline keyboard
 Advanced parameters
Image size (512x512, 1024x1024, etc.)
Negative prompts
CFG scale adjustment
 User features
/gallery — last 5 generated images
Image upscaling (4x)
Style presets (anime, realistic, cartoon)
 Infrastructure
Rate limiting (3 images/hour per user)
Queue system for multiple users
Cost tracking per user
 Analytics
Google Sheets logging
Usage statistics dashboard
Most popular prompts
📚 What I Learned
Technical Skills
✅ n8n workflow automation — visual programming, node orchestration

✅ ComfyUI API integration — asynchronous job submission, binary data handling

✅ Claude API — conversational AI, context management

✅ Telegram Bot API — webhooks, message formatting, media sending

✅ RunPod GPU infrastructure — serverless GPU deployment

✅ Docker — containerized application deployment

Architecture & Design
✅ Error handling patterns — graceful degradation, user feedback

✅ Asynchronous processing — job queues, polling vs webhooks

✅ API orchestration — chaining multiple services

✅ UX considerations — loading states, clear error messages

DevOps
✅ Webhook setup — ngrok tunneling, endpoint configuration

✅ Environment management — credentials, API keys

✅ Debugging distributed systems — logs, request inspection

🎓 Skills Demonstrated
Category	Skills
Integration	Multi-API orchestration, webhook handling, authentication
Automation	Workflow design, error handling, state management
AI/ML	LLM integration, image generation models, prompt engineering
Backend	Asynchronous processing, binary data, HTTP protocols
Infrastructure	Docker, GPU servers, tunneling, cloud services
UX	User feedback, error messaging, command design
📂 Project Structure
TEXT
ai-image-bot/
├── README.md                 # This file
├── workflow.json             # n8n workflow export
├── screenshots/              # UI screenshots
│   ├── workflow.png         # n8n workflow overview
│   ├── help.png             # /help command
│   ├── generation.png       # Generation in progress
│   └── result.png           # Final generated image
├── demo.mp4                 # Video demonstration (optional)
└── .gitignore               # Git ignore file
🤝 Contributing
This is a portfolio project, but suggestions are welcome!

Ideas for improvement?

Open an issue
Describe your idea
I'll consider it for v2.0
📝 License
MIT License — feel free to learn from this project!

👤 Author
[Martyniuk Artem]

LinkedIn: [https://www.linkedin.com/in/martyniuk-artem/]
GitHub: [@MartyniukArtem]
🙏 Acknowledgments
n8n — for the amazing workflow automation platform
ComfyUI — for the powerful image generation API
Anthropic — for Claude API
RunPod — for affordable GPU infrastructure
⭐ If this project helped you learn something new, consider starring it!

Built as a portfolio project to demonstrate AI automation & integration skills.

Technologies: n8n · ComfyUI · Claude API · Telegram Bot · RunPod · Docker
