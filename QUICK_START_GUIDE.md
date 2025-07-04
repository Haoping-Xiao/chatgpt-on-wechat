# Quick Start Guide

## Overview

This guide will help you get ChatGPT-on-WeChat up and running in minutes. Follow these steps to deploy your own AI-powered chatbot across multiple platforms.

## Prerequisites

- Python 3.7.1 - 3.9.x (recommended: Python 3.8)
- Git
- OpenAI API key (or other supported AI service credentials)
- Platform-specific credentials (WeChat, DingTalk, etc.)

## Installation

### Method 1: Quick Install Script (Recommended)

```bash
# One-line installation (Linux/macOS)
bash <(curl -sS https://cdn.link-ai.tech/code/cow/install.sh)
```

### Method 2: Manual Installation

```bash
# 1. Clone the repository
git clone https://github.com/zhayujie/chatgpt-on-wechat.git
cd chatgpt-on-wechat

# 2. Install dependencies
pip3 install -r requirements.txt

# 3. Install optional dependencies (recommended)
pip3 install -r requirements-optional.txt

# 4. Copy configuration template
cp config-template.json config.json
```

## Basic Configuration

### 1. Edit Configuration File

Open `config.json` and configure the essentials:

```json
{
    "model": "gpt-3.5-turbo",
    "open_ai_api_key": "sk-your-openai-api-key-here",
    "channel_type": "terminal",
    "single_chat_prefix": ["bot", "@bot"],
    "group_chat_prefix": ["@bot"],
    "character_desc": "You are a helpful AI assistant."
}
```

### 2. Environment Variables (Alternative)

You can also use environment variables:

```bash
export OPEN_AI_API_KEY="sk-your-openai-api-key-here"
export MODEL="gpt-3.5-turbo"
export CHANNEL_TYPE="terminal"
```

## Running Your First Bot

### Terminal Mode (Testing)

```bash
# Start in terminal mode for testing
python3 app.py --cmd

# Or specify terminal channel in config
python3 app.py
```

**Example Terminal Interaction:**
```
User: bot hello
Bot: Hello! How can I help you today?

User: bot tell me a joke
Bot: Why don't scientists trust atoms? Because they make up everything!
```

## Platform Integration

### WeChat Personal Account

1. **Configure WeChat channel:**
```json
{
    "channel_type": "wx",
    "hot_reload": true
}
```

2. **Run and scan QR code:**
```bash
python3 app.py
```

3. **Scan the QR code** displayed in terminal with WeChat

### WeChat Official Account

1. **Configure WeChat MP:**
```json
{
    "channel_type": "wechatmp",
    "wechatmp_app_id": "your-app-id",
    "wechatmp_app_secret": "your-app-secret",
    "wechatmp_token": "your-token",
    "wechatmp_port": 8080
}
```

2. **Set webhook URL** in WeChat MP admin: `http://your-domain.com:8080/wx`

### DingTalk

1. **Configure DingTalk:**
```json
{
    "channel_type": "dingtalk",
    "dingtalk_client_id": "your-client-id",
    "dingtalk_client_secret": "your-client-secret"
}
```

### Feishu/Lark

1. **Configure Feishu:**
```json
{
    "channel_type": "feishu",
    "feishu_app_id": "your-app-id",
    "feishu_app_secret": "your-app-secret",
    "feishu_token": "your-verification-token",
    "feishu_port": 80
}
```

## Popular AI Models

### OpenAI Models
```json
{
    "model": "gpt-4o",
    "open_ai_api_key": "sk-your-key"
}
```

### Claude
```json
{
    "model": "claude-3-5-sonnet-latest",
    "claude_api_key": "your-claude-key"
}
```

### Chinese Models

**Baidu Wenxin:**
```json
{
    "model": "wenxin",
    "baidu_wenxin_api_key": "your-api-key",
    "baidu_wenxin_secret_key": "your-secret-key"
}
```

**Xunfei Spark:**
```json
{
    "model": "xunfei",
    "xunfei_app_id": "your-app-id",
    "xunfei_api_key": "your-api-key",
    "xunfei_api_secret": "your-api-secret"
}
```

### LinkAI (Recommended for China)
```json
{
    "use_linkai": true,
    "linkai_api_key": "your-linkai-key",
    "linkai_app_code": "your-app-code"
}
```

## Essential Features

### 1. Voice Processing

Enable voice recognition and synthesis:

```json
{
    "speech_recognition": true,
    "voice_reply_voice": true,
    "voice_to_text": "openai",
    "text_to_voice": "openai"
}
```

### 2. Image Generation

Enable image creation with DALL-E:

```json
{
    "image_create_prefix": ["画", "draw", "生成"],
    "text_to_image": "dall-e-3"
}
```

**Usage:**
- "画一只可爱的猫" → Generates cat image
- "draw a sunset" → Creates sunset image

### 3. Group Chat

Configure group chat settings:

```json
{
    "group_name_white_list": ["AI助手群", "工作群"],
    "group_chat_in_one_session": ["AI助手群"],
    "group_chat_keyword": ["帮助", "AI"]
}
```

## Plugin System

### Enable Popular Plugins

1. **Create plugin config:**
```bash
cp plugins/config.json.template plugins/config.json
```

2. **Configure plugins:**
```json
{
    "plugins": [
        {
            "name": "hello",
            "enabled": true
        },
        {
            "name": "role",
            "enabled": true
        }
    ]
}
```

### Quick Plugin Examples

**Role Playing Plugin:**
```
User: $角色 你是一个专业的Python程序员
Bot: 好的，我现在是一个专业的Python程序员，可以帮您解决编程问题。

User: 如何实现单例模式？
Bot: 在Python中实现单例模式有几种方式...
```

**Godcmd Plugin (Admin Commands):**
```
Admin: #help
Bot: 管理员指令列表:
- #reset: 重置会话
- #debug: 开启调试模式
- #reload: 重载插件
```

## Docker Deployment

### Quick Docker Setup

1. **Download docker-compose:**
```bash
wget https://open-1317903499.cos.ap-guangzhou.myqcloud.com/docker-compose.yml
```

2. **Edit environment variables:**
```yaml
environment:
  OPEN_AI_API_KEY: "sk-your-openai-api-key"
  MODEL: "gpt-3.5-turbo"
  CHANNEL_TYPE: "wx"
```

3. **Start container:**
```bash
docker compose up -d
```

4. **View logs and scan QR:**
```bash
docker logs -f chatgpt-on-wechat
```

## Production Deployment

### Server Deployment

1. **Run in background:**
```bash
nohup python3 app.py > output.log 2>&1 &
```

2. **View logs:**
```bash
tail -f output.log
```

3. **Process management:**
```bash
# Check running process
ps -ef | grep app.py

# Kill process
kill <process-id>
```

### Using PM2 (Recommended)

1. **Install PM2:**
```bash
npm install -g pm2
```

2. **Create ecosystem file:**
```javascript
// ecosystem.config.js
module.exports = {
  apps: [{
    name: 'chatgpt-on-wechat',
    script: 'python3',
    args: 'app.py',
    cwd: '/path/to/chatgpt-on-wechat',
    instances: 1,
    autorestart: true,
    watch: false,
    max_memory_restart: '1G',
    env: {
      NODE_ENV: 'production'
    }
  }]
}
```

3. **Start with PM2:**
```bash
pm2 start ecosystem.config.js
pm2 save
pm2 startup
```

## Common Configurations

### Complete Basic Config

```json
{
    "model": "gpt-3.5-turbo",
    "open_ai_api_key": "sk-your-openai-api-key",
    "channel_type": "wx",
    "proxy": "",
    
    "single_chat_prefix": ["bot", "@bot"],
    "single_chat_reply_prefix": "[bot] ",
    "group_chat_prefix": ["@bot"],
    "group_name_white_list": ["ChatGPT测试群"],
    
    "character_desc": "你是ChatGPT，一个由OpenAI训练的大型语言模型，你旨在回答并解决人们的任何问题。",
    "conversation_max_tokens": 1000,
    "expires_in_seconds": 3600,
    
    "speech_recognition": true,
    "voice_reply_voice": false,
    "image_create_prefix": ["画", "看", "找"],
    
    "hot_reload": true,
    "debug": false
}
```

### Enterprise WeChat Config

```json
{
    "channel_type": "wechatcom_app",
    "wechatcom_corp_id": "your-corp-id",
    "wechatcomapp_agent_id": "your-agent-id",
    "wechatcomapp_secret": "your-app-secret",
    "wechatcomapp_token": "your-token",
    "wechatcomapp_aes_key": "your-aes-key",
    "wechatcomapp_port": 9898
}
```

### Multi-Channel Config

```json
{
    "channel_type": "wx",
    "backup_channels": ["terminal", "web"],
    "channel_routing": {
        "admin_users": ["admin123"],
        "vip_groups": ["VIP群"]
    }
}
```

## Troubleshooting

### Common Issues

1. **Import Error:**
```bash
# Install missing dependencies
pip3 install -r requirements-optional.txt
```

2. **WeChat Login Issues:**
```bash
# Clear cache and retry
rm -rf ./tmp/
python3 app.py
```

3. **API Rate Limits:**
```json
{
    "rate_limit_chatgpt": 10,
    "request_timeout": 120,
    "conversation_max_tokens": 500
}
```

4. **Voice Issues:**
```bash
# Install voice dependencies
pip3 install pydub SpeechRecognition
```

### Debug Mode

Enable detailed logging:

```json
{
    "debug": true,
    "log_level": "DEBUG"
}
```

### Health Check

```bash
# Test bot response
curl -X POST http://localhost:8080/health

# Check logs
tail -f nohup.out | grep ERROR
```

## Next Steps

### Advanced Features

1. **Knowledge Base Integration**
   - Configure LinkAI for custom knowledge bases
   - Upload documents for domain-specific responses

2. **Custom Plugins**
   - Follow the [Plugin Development Guide](PLUGIN_DEVELOPMENT_GUIDE.md)
   - Create business-specific functionality

3. **Multi-Model Setup**
   - Use different models for different tasks
   - Implement model switching based on context

4. **Monitoring & Analytics**
   - Set up logging and monitoring
   - Track usage patterns and performance

### Scaling

1. **Load Balancing**
   - Deploy multiple instances
   - Use Redis for session sharing

2. **Database Integration**
   - Add persistent storage
   - Implement user profiles

3. **API Gateway**
   - Expose REST API endpoints
   - Implement authentication

## Resources

- **Official Documentation:** [API Documentation](API_DOCUMENTATION.md)
- **Plugin Development:** [Plugin Guide](PLUGIN_DEVELOPMENT_GUIDE.md)
- **Community:** GitHub Issues and Discussions
- **Support:** WeChat Group (see README)

## Quick Command Reference

```bash
# Start bot
python3 app.py

# Terminal mode
python3 app.py --cmd

# Background mode
nohup python3 app.py &

# Docker mode
docker compose up -d

# View logs
tail -f nohup.out
docker logs -f chatgpt-on-wechat

# Reload config (send to bot)
#reload

# Clear memory (send to bot)
#reset
```

This guide should get you started with ChatGPT-on-WeChat quickly. For detailed API documentation and advanced features, refer to the comprehensive [API Documentation](API_DOCUMENTATION.md).