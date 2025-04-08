# shoUme LINE Messaging Implementation Guide

## Introduction

This document provides practical guidance for implementing the LINE messaging functionality in the shoUme application. It complements the [LINE Messaging Specification](../specifications/line-messaging.md) with code examples and implementation best practices.

## Prerequisites

- LINE Developer Account
- LINE Messaging API Channel
- Node.js (v16 or higher)
- Express.js framework

## Setup

### 1. Install Required Packages

```bash
npm install @line/bot-sdk express dotenv
```

### 2. Configure Environment Variables

Create a `.env` file with the following variables:

```
LINE_CHANNEL_ACCESS_TOKEN=your_channel_access_token
LINE_CHANNEL_SECRET=your_channel_secret
PORT=3000
```

### 3. Basic Server Setup

```javascript
// app.js
const express = require('express');
const line = require('@line/bot-sdk');
require('dotenv').config();

const app = express();

// LINE SDK configuration
const lineConfig = {
  channelAccessToken: process.env.LINE_CHANNEL_ACCESS_TOKEN,
  channelSecret: process.env.LINE_CHANNEL_SECRET
};

// Create LINE client
const lineClient = new line.Client(lineConfig);

// Webhook endpoint
app.post('/line/webhook', line.middleware(lineConfig), (req, res) => {
  Promise
    .all(req.body.events.map(handleEvent))
    .then((result) => res.json(result))
    .catch((err) => {
      console.error(err);
      res.status(500).end();
    });
});

// Start server
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

## Message Handling

### Basic Message Handler

```javascript
// Event handler
function handleEvent(event) {
  if (event.type !== 'message' || event.message.type !== 'text') {
    // Ignore non-text messages
    return Promise.resolve(null);
  }

  const userMessage = event.message.text;
  
  // Check if message is a command
  if (userMessage.startsWith('/')) {
    return handleCommand(event);
  }
  
  // Handle regular message
  return lineClient.replyMessage(event.replyToken, {
    type: 'text',
    text: `You said: ${userMessage}`
  });
}
```

### Command Processing

```javascript
function handleCommand(event) {
  const command = event.message.text.toLowerCase();
  
  switch(command) {
    case '/help':
      return lineClient.replyMessage(event.replyToken, {
        type: 'text',
        text: 'Available commands:\n/help - Show this help\n/status - Check system status\n/settings - Manage your settings'
      });
      
    case '/status':
      return lineClient.replyMessage(event.replyToken, {
        type: 'text',
        text: 'All systems operational'
      });
      
    case '/settings':
      return sendSettingsMenu(event.replyToken);
      
    default:
      return lineClient.replyMessage(event.replyToken, {
        type: 'text',
        text: 'Unknown command. Type /help for available commands.'
      });
  }
}
```

## Rich Message Examples

### Card Message

```javascript
function sendCardMessage(replyToken, title, text, imageUrl, actions) {
  return lineClient.replyMessage(replyToken, {
    type: 'template',
    altText: title,
    template: {
      type: 'buttons',
      thumbnailImageUrl: imageUrl,
      title: title,
      text: text,
      actions: actions
    }
  });
}

// Example usage
sendCardMessage(
  replyToken,
  'Product Details',
  'Premium Kampo Medicine',
  'https://example.com/product.jpg',
  [
    {
      type: 'uri',
      label: 'View Details',
      uri: 'https://shoume.com/products/123'
    },
    {
      type: 'postback',
      label: 'Add to Cart',
      data: 'action=addToCart&productId=123'
    }
  ]
);
```

### Carousel Message

```javascript
function sendCarouselMessage(replyToken, items) {
  return lineClient.replyMessage(replyToken, {
    type: 'template',
    altText: 'Carousel',
    template: {
      type: 'carousel',
      columns: items.map(item => ({
        thumbnailImageUrl: item.imageUrl,
        title: item.title,
        text: item.description,
        actions: item.actions
      }))
    }
  });
}
```

### Quick Reply Buttons

```javascript
function sendMessageWithQuickReplies(replyToken, text, quickReplies) {
  return lineClient.replyMessage(replyToken, {
    type: 'text',
    text: text,
    quickReply: {
      items: quickReplies.map(item => ({
        type: 'action',
        action: {
          type: 'message',
          label: item.label,
          text: item.text
        }
      }))
    }
  });
}
```

## Handling Postback Events

```javascript
function handlePostback(event) {
  const data = new URLSearchParams(event.postback.data);
  const action = data.get('action');
  
  switch(action) {
    case 'addToCart':
      const productId = data.get('productId');
      // Process adding to cart
      return lineClient.replyMessage(event.replyToken, {
        type: 'text',
        text: `Added product ${productId} to your cart`
      });
      
    // Handle other actions
    default:
      return Promise.resolve(null);
  }
}
```

## Error Handling

```javascript
// Middleware for error handling
app.use((err, req, res, next) => {
  if (err instanceof line.SignatureValidationFailed) {
    res.status(401).send('Invalid signature');
    return;
  } else if (err instanceof line.JSONParseError) {
    res.status(400).send('Invalid request body');
    return;
  }
  
  console.error(err);
  res.status(500).send('Internal server error');
});
```

## Logging

```javascript
// Request logging middleware
app.use((req, res, next) => {
  const requestId = crypto.randomUUID();
  
  // Attach request ID to request object
  req.requestId = requestId;
  
  // Log request
  console.log(JSON.stringify({
    timestamp: new Date().toISOString(),
    requestId: requestId,
    method: req.method,
    path: req.path,
    ip: req.ip
  }));
  
  next();
});
```

## Testing

### Local Testing with ngrok

1. Install ngrok: `npm install -g ngrok`
2. Start your application: `node app.js`
3. Start ngrok: `ngrok http 3000`
4. Update your webhook URL in the LINE Developer Console with the ngrok URL

### Unit Testing

```javascript
// test/message-handler.test.js
const { handleMessage } = require('../handlers/message-handler');
const { LineClient } = require('@line/bot-sdk');
const sinon = require('sinon');

describe('Message Handler', () => {
  let lineClientStub;
  
  beforeEach(() => {
    lineClientStub = sinon.stub(LineClient.prototype, 'replyMessage');
  });
  
  afterEach(() => {
    lineClientStub.restore();
  });
  
  it('should handle text messages correctly', async () => {
    const event = {
      type: 'message',
      message: {
        type: 'text',
        text: 'hello'
      },
      replyToken: 'test-reply-token'
    };
    
    await handleMessage(event);
    
    sinon.assert.calledWith(
      lineClientStub,
      'test-reply-token',
      sinon.match({ type: 'text' })
    );
  });
});
```

## Deployment

### Docker Configuration

```dockerfile
FROM node:16-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .

EXPOSE 3000

CMD ["node", "app.js"]
```

### Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: shoume-line-bot
spec:
  replicas: 3
  selector:
    matchLabels:
      app: shoume-line-bot
  template:
    metadata:
      labels:
        app: shoume-line-bot
    spec:
      containers:
      - name: shoume-line-bot
        image: shoume/line-bot:latest
        ports:
        - containerPort: 3000
        env:
        - name: LINE_CHANNEL_ACCESS_TOKEN
          valueFrom:
            secretKeyRef:
              name: line-credentials
              key: channel-access-token
        - name: LINE_CHANNEL_SECRET
          valueFrom:
            secretKeyRef:
              name: line-credentials
              key: channel-secret
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
```

## Best Practices

1. **Rate Limiting**: Implement rate limiting to prevent abuse
2. **Validation**: Validate all user inputs before processing
3. **Monitoring**: Set up alerts for error rates and response times
4. **Caching**: Cache frequently used data to improve performance
5. **Graceful Degradation**: Implement fallbacks for when LINE API is unavailable
6. **Security**: Store sensitive credentials in environment variables or secrets management

## Troubleshooting

| Issue | Possible Cause | Solution |
|-------|----------------|----------|
| Webhook verification fails | Invalid channel secret | Double-check the channel secret in your .env file |
| Messages not being received | Incorrect webhook URL | Verify the webhook URL in LINE Developer Console |
| Rate limit exceeded | Too many requests | Implement exponential backoff strategy |
| Message delivery failures | Network issues | Add retry logic with circuit breaker pattern |

## References

- [LINE Messaging API Documentation](https://developers.line.biz/en/docs/messaging-api/)
- [LINE Bot SDK for Node.js](https://github.com/line/line-bot-sdk-nodejs)
- [Express.js Documentation](https://expressjs.com/)