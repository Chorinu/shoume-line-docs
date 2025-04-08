# shoUme LINE Messaging Specification

## Overview
This document outlines the technical specifications for the LINE messaging functionality in the shoUme application. It covers message formats, API endpoints, authentication, and user interaction flows.

## Message Types

### Text Messages
- **Format**: Plain text
- **Character Limit**: 5,000 characters
- **Supported Features**: Emoji, line breaks, URLs

### Rich Messages
- **Card Format**: Title, subtitle, image, action buttons
- **Carousel Format**: Multiple cards in horizontal scrollable view
- **Quick Reply Buttons**: Up to 13 quick reply options

### Media Messages
- **Image**: JPEG, PNG (Max size: 10MB)
- **Video**: MP4 (Max size: 200MB, Max duration: 1 minute)
- **Audio**: M4A (Max size: 10MB, Max duration: 5 minutes)
- **Files**: PDF, ZIP, etc. (Max size: 300MB)

## API Integration

### Authentication
- **Channel Access Token**: Required for all API calls
- **Token Expiration**: 30 days
- **Refresh Process**: Automatic renewal via webhook

### Webhook Configuration
- **Endpoint URL**: https://api.shoume.com/line/webhook
- **Verification**: HMAC-SHA256 signature validation
- **Events**: message, follow, unfollow, join, leave, postback

### Rate Limits
- **Standard Plan**: 1000 requests/minute
- **Business Plan**: 3000 requests/minute
- **Enterprise Plan**: Custom limits

## User Interaction Flow

### Initial Contact
1. User adds shoUme as a friend on LINE
2. Welcome message sent automatically
3. User profile information captured (with consent)

### Command Processing
- **Format**: Keyword prefixed with "/"
- **Example**: "/help", "/status", "/settings"
- **Fallback**: Natural language processing for non-command messages

### Notification System
- **Types**: Transaction confirmations, reminders, updates
- **Frequency Control**: User-configurable settings
- **Opt-out**: Available via settings menu

## Security Considerations

### Data Protection
- **Message Encryption**: TLS 1.3 for transmission
- **Storage**: End-to-end encryption for sensitive content
- **Retention Policy**: Messages purged after 90 days

### Privacy Controls
- **User Data**: Stored in compliance with GDPR and APPI
- **Consent Management**: Explicit opt-in for data usage
- **Access Control**: Role-based permissions for admin panel

## Implementation Guidelines

### Development Environment
- **SDK**: LINE Messaging API SDK for Node.js
- **Testing Tools**: LINE Bot Designer, Messaging API Simulator
- **Logging**: Structured JSON logs with request IDs

### Deployment
- **Environment**: Containerized microservice
- **Scaling**: Horizontal auto-scaling based on message volume
- **Monitoring**: Prometheus metrics for API calls and response times

## Error Handling

### Error Codes
- **400**: Invalid message format
- **401**: Authentication failure
- **429**: Rate limit exceeded
- **500**: Server error

### Recovery Strategies
- **Retry Logic**: Exponential backoff for transient errors
- **Fallback Channels**: Email notification for critical failures
- **User Communication**: Friendly error messages with support contact

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0   | 2025-04-08 | Initial specification |