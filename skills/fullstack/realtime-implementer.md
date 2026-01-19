---
name: realtime-implementer
description: |
  Implement real-time features using WebSockets, Server-Sent Events, or polling.
  Creates live updates, notifications, and collaborative features.
---

# Realtime Implementer Skill

## Purpose
Add real-time capabilities to web applications for live updates and collaboration.

## Technology Options

| Technology | Use Case | Pros | Cons |
|------------|----------|------|------|
| WebSocket | Bidirectional, chat, games | Full duplex, low latency | Connection management |
| SSE | Server push, notifications | Simple, auto-reconnect | One-way only |
| Polling | Simple updates | Easy to implement | Not real-time, wasteful |
| Supabase Realtime | Database changes | Easy setup, built-in | Tied to Supabase |

## WebSocket Implementation

### WebSocket Client
```typescript
// lib/websocket.ts
type MessageType = 'message' | 'notification' | 'presence' | 'typing';

interface WSMessage<T = unknown> {
  type: MessageType;
  payload: T;
  timestamp: string;
}

type MessageHandler<T = unknown> = (payload: T) => void;

class WebSocketClient {
  private ws: WebSocket | null = null;
  private handlers = new Map<MessageType, Set<MessageHandler>>();
  private reconnectAttempts = 0;
  private maxReconnectAttempts = 5;
  private reconnectDelay = 1000;
  private url: string = '';

  connect(url: string, token?: string) {
    this.url = token ? `${url}?token=${token}` : url;
    this.establishConnection();
  }

  private establishConnection() {
    this.ws = new WebSocket(this.url);

    this.ws.onopen = () => {
      console.log('WebSocket connected');
      this.reconnectAttempts = 0;
    };

    this.ws.onmessage = (event) => {
      try {
        const message: WSMessage = JSON.parse(event.data);
        const handlers = this.handlers.get(message.type);
        handlers?.forEach((handler) => handler(message.payload));
      } catch (error) {
        console.error('Failed to parse WebSocket message:', error);
      }
    };

    this.ws.onclose = (event) => {
      console.log('WebSocket closed:', event.code, event.reason);
      this.attemptReconnect();
    };

    this.ws.onerror = (error) => {
      console.error('WebSocket error:', error);
    };
  }

  private attemptReconnect() {
    if (this.reconnectAttempts >= this.maxReconnectAttempts) {
      console.error('Max reconnection attempts reached');
      return;
    }

    this.reconnectAttempts++;
    const delay = this.reconnectDelay * Math.pow(2, this.reconnectAttempts - 1);

    console.log(`Reconnecting in ${delay}ms (attempt ${this.reconnectAttempts})`);

    setTimeout(() => {
      this.establishConnection();
    }, delay);
  }

  subscribe<T>(type: MessageType, handler: MessageHandler<T>) {
    if (!this.handlers.has(type)) {
      this.handlers.set(type, new Set());
    }
    this.handlers.get(type)!.add(handler as MessageHandler);

    // Return unsubscribe function
    return () => {
      this.handlers.get(type)?.delete(handler as MessageHandler);
    };
  }

  send<T>(type: MessageType, payload: T) {
    if (this.ws?.readyState === WebSocket.OPEN) {
      const message: WSMessage<T> = {
        type,
        payload,
        timestamp: new Date().toISOString(),
      };
      this.ws.send(JSON.stringify(message));
    } else {
      console.warn('WebSocket not connected');
    }
  }

  disconnect() {
    this.maxReconnectAttempts = 0; // Prevent reconnection
    this.ws?.close();
  }
}

export const wsClient = new WebSocketClient();
```

### React Hook for WebSocket
```typescript
// hooks/use-websocket.ts
import { useEffect, useCallback, useState } from 'react';
import { wsClient } from '@/lib/websocket';

interface UseWebSocketOptions {
  url: string;
  token?: string;
  autoConnect?: boolean;
}

export function useWebSocket<T>({
  url,
  token,
  autoConnect = true,
}: UseWebSocketOptions) {
  const [isConnected, setIsConnected] = useState(false);
  const [lastMessage, setLastMessage] = useState<T | null>(null);

  useEffect(() => {
    if (autoConnect) {
      wsClient.connect(url, token);
    }

    return () => {
      wsClient.disconnect();
    };
  }, [url, token, autoConnect]);

  const subscribe = useCallback(
    <P>(type: string, handler: (payload: P) => void) => {
      return wsClient.subscribe(type as any, handler);
    },
    []
  );

  const send = useCallback(<P>(type: string, payload: P) => {
    wsClient.send(type as any, payload);
  }, []);

  return {
    isConnected,
    lastMessage,
    subscribe,
    send,
  };
}
```

### Chat Feature Example
```tsx
// hooks/use-chat.ts
import { useState, useEffect, useCallback } from 'react';
import { useWebSocket } from './use-websocket';

interface Message {
  id: string;
  content: string;
  userId: string;
  userName: string;
  createdAt: string;
}

export function useChat(roomId: string) {
  const [messages, setMessages] = useState<Message[]>([]);
  const [typingUsers, setTypingUsers] = useState<string[]>([]);
  const { subscribe, send, isConnected } = useWebSocket({
    url: `${process.env.NEXT_PUBLIC_WS_URL}/chat/${roomId}`,
  });

  useEffect(() => {
    // Subscribe to new messages
    const unsubMessage = subscribe<Message>('message', (message) => {
      setMessages((prev) => [...prev, message]);
    });

    // Subscribe to typing indicators
    const unsubTyping = subscribe<{ userId: string; isTyping: boolean }>(
      'typing',
      ({ userId, isTyping }) => {
        setTypingUsers((prev) =>
          isTyping
            ? [...prev.filter((id) => id !== userId), userId]
            : prev.filter((id) => id !== userId)
        );
      }
    );

    return () => {
      unsubMessage();
      unsubTyping();
    };
  }, [subscribe]);

  const sendMessage = useCallback(
    (content: string) => {
      send('message', { content, roomId });
    },
    [send, roomId]
  );

  const sendTyping = useCallback(
    (isTyping: boolean) => {
      send('typing', { isTyping, roomId });
    },
    [send, roomId]
  );

  return {
    messages,
    typingUsers,
    sendMessage,
    sendTyping,
    isConnected,
  };
}

// components/chat.tsx
export function Chat({ roomId }: { roomId: string }) {
  const { messages, typingUsers, sendMessage, sendTyping, isConnected } = useChat(roomId);
  const [input, setInput] = useState('');

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    if (input.trim()) {
      sendMessage(input);
      setInput('');
    }
  };

  return (
    <div className="flex flex-col h-full">
      <div className="flex-1 overflow-y-auto p-4 space-y-4">
        {messages.map((msg) => (
          <div key={msg.id} className="flex gap-2">
            <span className="font-medium">{msg.userName}:</span>
            <span>{msg.content}</span>
          </div>
        ))}
      </div>

      {typingUsers.length > 0 && (
        <div className="px-4 text-sm text-muted-foreground">
          {typingUsers.join(', ')} {typingUsers.length === 1 ? 'is' : 'are'} typing...
        </div>
      )}

      <form onSubmit={handleSubmit} className="p-4 border-t">
        <div className="flex gap-2">
          <input
            value={input}
            onChange={(e) => {
              setInput(e.target.value);
              sendTyping(e.target.value.length > 0);
            }}
            placeholder="Type a message..."
            className="flex-1 px-3 py-2 border rounded"
            disabled={!isConnected}
          />
          <button
            type="submit"
            disabled={!isConnected || !input.trim()}
            className="px-4 py-2 bg-primary text-white rounded"
          >
            Send
          </button>
        </div>
      </form>
    </div>
  );
}
```

## Server-Sent Events (SSE)

```typescript
// hooks/use-sse.ts
import { useEffect, useState } from 'react';

export function useSSE<T>(url: string) {
  const [data, setData] = useState<T | null>(null);
  const [error, setError] = useState<Error | null>(null);
  const [isConnected, setIsConnected] = useState(false);

  useEffect(() => {
    const eventSource = new EventSource(url);

    eventSource.onopen = () => {
      setIsConnected(true);
      setError(null);
    };

    eventSource.onmessage = (event) => {
      try {
        const parsed = JSON.parse(event.data);
        setData(parsed);
      } catch (e) {
        setError(new Error('Failed to parse SSE data'));
      }
    };

    eventSource.onerror = () => {
      setIsConnected(false);
      setError(new Error('SSE connection failed'));
    };

    return () => {
      eventSource.close();
    };
  }, [url]);

  return { data, error, isConnected };
}

// Usage: Notifications
function Notifications() {
  const { data: notification } = useSSE<Notification>('/api/notifications/stream');

  useEffect(() => {
    if (notification) {
      toast(notification.message);
    }
  }, [notification]);

  return null;
}
```

## Supabase Realtime

```typescript
// hooks/use-realtime-posts.ts
import { useEffect } from 'react';
import { useQueryClient } from '@tanstack/react-query';
import { supabase } from '@/lib/supabase';

export function useRealtimePosts() {
  const queryClient = useQueryClient();

  useEffect(() => {
    const channel = supabase
      .channel('posts-changes')
      .on(
        'postgres_changes',
        { event: '*', schema: 'public', table: 'posts' },
        (payload) => {
          // Invalidate queries to refetch
          queryClient.invalidateQueries({ queryKey: ['posts'] });

          // Or update cache directly
          if (payload.eventType === 'INSERT') {
            queryClient.setQueryData(['posts'], (old: Post[]) => [
              payload.new as Post,
              ...old,
            ]);
          }
        }
      )
      .subscribe();

    return () => {
      supabase.removeChannel(channel);
    };
  }, [queryClient]);
}
```

## Quality Checklist

- [ ] Connection state managed
- [ ] Auto-reconnection implemented
- [ ] Message types typed
- [ ] Cleanup on unmount
- [ ] Error handling for disconnects
- [ ] Optimistic UI updates
- [ ] Fallback for no WebSocket support
- [ ] Rate limiting considered
