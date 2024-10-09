---
theme: seriph
background: https://source.unsplash.com/collection/94734566/1920x1080
class: 'text-center'
highlighter: shiki
lineNumbers: false
info: |
  ## Clean Architecture
  Presentation on Clean Architecture principles and implementation.
drawings:
  persist: false
css: unocss
---

# Clean Architecture

A comprehensive guide to building maintainable and scalable software

<div class="pt-12">
  <span @click="$slidev.nav.next" class="px-2 py-1 rounded cursor-pointer" hover="bg-white bg-opacity-10">
    Press Space for next page <carbon:arrow-right class="inline"/>
  </span>
</div>

<div class="abs-br m-6 flex gap-2">
  <a href="https://github.com/topics/clean-architecture" target="_blank" alt="GitHub"
    class="text-xl icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>

---

# What is Clean Architecture?

Clean Architecture is a software design philosophy that separates the elements of a design into ring levels. The main rule of clean architecture is that source code dependencies can only point inwards.

- 🎯 **Separation of Concerns** - Clear boundaries between different parts of your application
- 🔄 **Dependency Rule** - Dependencies point inward, toward higher-level policies
- 🔌 **Pluggable** - The core business logic is independent of UI, database, frameworks, and external agencies
- 📊 **Testable** - Business rules can be tested without UI, database, web server, or any external element

---

# The Clean Architecture Layers

<div class="flex justify-center">
  <img src="https://blog.cleancoder.com/uncle-bob/images/2012-08-13-the-clean-architecture/CleanArchitecture.jpg" class="h-80 rounded shadow" />
</div>

1. **Entities**: Enterprise-wide business rules
2. **Use Cases**: Application-specific business rules
3. **Interface Adapters**: Converts data between use cases and external agencies
4. **Frameworks and Drivers**: External frameworks, tools, and delivery mechanisms

---

# Benefits of Clean Architecture

- 📦 **Independent of Frameworks**: The architecture doesn't depend on the existence of some library of feature-laden software
- 🧪 **Testable**: The business rules can be tested without the UI, database, web server, or any external element
- 🖥️ **Independent of UI**: The UI can change easily, without changing the rest of the system
- 💾 **Independent of Database**: You can swap out Oracle or SQL Server for Mongo, BigTable, CouchDB, or something else
- 🔄 **Independent of any external agency**: Your business rules don't know anything about the outside world

---

# Implementing Clean Architecture

1. **Start with the core**: Define your entities and use cases
2. **Build outward**: Implement interface adapters and frameworks last
3. **Use dependency inversion**: High-level modules should not depend on low-level modules
4. **Create boundaries**: Use interfaces to define clear boundaries between layers
5. **Follow SOLID principles**: Single Responsibility, Open-Closed, Liskov Substitution, Interface Segregation, and Dependency Inversion

---

# Code Example: Entity

<div class="max-h-[400px] overflow-y-auto">

```typescript
class User {
  constructor(
    public id: string,
    public name: string,
    public email: string
  ) {}

  validateEmail(): boolean {
    // Email validation logic
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(this.email);
  }
}
```

</div>

---

# Code Example: Use Case

<div class="max-h-[400px] overflow-y-auto">

```typescript
interface UserRepository {
  save(user: User): Promise<void>;
  findById(id: string): Promise<User | null>;
}

class CreateUserUseCase {
  constructor(private userRepository: UserRepository) {}

  async execute(userData: { name: string; email: string }): Promise<User> {
    const user = new User(generateId(), userData.name, userData.email);
    
    if (!user.validateEmail()) {
      throw new Error('Invalid email');
    }

    await this.userRepository.save(user);
    return user;
  }
}
```

</div>

---

# Code Example: Interface Adapter

<div class="max-h-[400px] overflow-y-auto">

```typescript
class UserController {
  constructor(private createUserUseCase: CreateUserUseCase) {}

  async createUser(req: Request, res: Response) {
    try {
      const user = await this.createUserUseCase.execute(req.body);
      res.status(201).json(user);
    } catch (error) {
      res.status(400).json({ error: error.message });
    }
  }
}
```

</div>

---

# Code Example: Frameworks and Drivers

<div class="max-h-[400px] overflow-y-auto">

```typescript
// src/infrastructure/database/MongoUserRepository.ts
import { MongoClient } from 'mongodb';
import { UserRepository } from '../../domain/repositories/UserRepository';
import { User } from '../../domain/entities/User';

export class MongoUserRepository implements UserRepository {
  private client: MongoClient;
  private db: any;

  constructor(connectionString: string) {
    this.client = new MongoClient(connectionString);
    this.db = this.client.db('users');
  }

  async save(user: User): Promise<void> {
    await this.db.collection('users').insertOne(user);
  }

  async findById(id: string): Promise<User | null> {
    const userData = await this.db.collection('users').findOne({ id });
    return userData ? new User(userData.id, userData.name, userData.email) : null;
  }
}
```

</div>

---

# Putting It All Together: App

<div class="max-h-[400px] overflow-y-auto">

```typescript
// src/main.ts
import express from 'express';
import { MongoUserRepository } from './infrastructure/database/MongoUserRepository';
import { CreateUserUseCase } from './domain/usecases/CreateUserUseCase';
import { UserController } from './interfaces/controllers/UserController';

const app = express();
const mongoRepo = new MongoUserRepository('mongodb://localhost:27017');
const createUserUseCase = new CreateUserUseCase(mongoRepo);
const userController = new UserController(createUserUseCase);

app.post('/users', (req, res) => userController.createUser(req, res));

app.listen(3000, () => console.log('Server running on port 3000'));
```

</div>

---

# Clean Architecture in Front-end: Chat App Example

Let's apply Clean Architecture principles to a simple chat web app using React.

1. Entities
2. Use Cases
3. Interface Adapters
4. Frameworks & Drivers (React Components)

---

# Entities: Message

<div class="max-h-[400px] overflow-y-auto">

```typescript
// src/domain/entities/Message.ts
export class Message {
  constructor(
    public id: string,
    public content: string,
    public sender: string,
    public timestamp: Date
  ) {}

  isValid(): boolean {
    return this.content.trim().length > 0 && this.sender.trim().length > 0;
  }
}
```

</div>

---

# Use Cases: SendMessage and GetMessages

<div class="max-h-[400px] overflow-y-auto">

```typescript
// src/domain/usecases/SendMessageUseCase.ts
import { Message } from '../entities/Message';

export interface MessageRepository {
  saveMessage(message: Message): Promise<void>;
  getMessages(): Promise<Message[]>;
}

export class SendMessageUseCase {
  constructor(private messageRepository: MessageRepository) {}

  async execute(content: string, sender: string): Promise<Message> {
    const message = new Message(
      Date.now().toString(),
      content,
      sender,
      new Date()
    );

    if (!message.isValid()) {
      throw new Error('Invalid message');
    }

    await this.messageRepository.saveMessage(message);
    return message;
  }
}

// src/domain/usecases/GetMessagesUseCase.ts
export class GetMessagesUseCase {
  constructor(private messageRepository: MessageRepository) {}

  async execute(): Promise<Message[]> {
    return await this.messageRepository.getMessages();
  }
}
```

</div>

---

# Interface Adapters: MessagePresenter

<div class="max-h-[400px] overflow-y-auto">

```typescript
// src/presentation/presenters/MessagePresenter.ts
import { Message } from '../../domain/entities/Message';

export interface MessageViewModel {
  id: string;
  content: string;
  sender: string;
  time: string;
}

export class MessagePresenter {
  static toViewModel(message: Message): MessageViewModel {
    return {
      id: message.id,
      content: message.content,
      sender: message.sender,
      time: message.timestamp.toLocaleTimeString(),
    };
  }
}
```

</div>

---

# Frameworks & Drivers: React Components

<div class="max-h-[400px] overflow-y-auto">

```tsx
// src/presentation/components/ChatBox.tsx
import React, { useState, useEffect } from 'react';
import { SendMessageUseCase } from '../../domain/usecases/SendMessageUseCase';
import { GetMessagesUseCase } from '../../domain/usecases/GetMessagesUseCase';
import { MessagePresenter, MessageViewModel } from '../presenters/MessagePresenter';

interface ChatBoxProps {
  sendMessageUseCase: SendMessageUseCase;
  getMessagesUseCase: GetMessagesUseCase;
}

export const ChatBox: React.FC<ChatBoxProps> = ({ sendMessageUseCase, getMessagesUseCase }) => {
  const [messages, setMessages] = useState<MessageViewModel[]>([]);
  const [input, setInput] = useState('');

  useEffect(() => {
    const fetchMessages = async () => {
      const fetchedMessages = await getMessagesUseCase.execute();
      setMessages(fetchedMessages.map(MessagePresenter.toViewModel));
    };
    fetchMessages();
  }, [getMessagesUseCase]);

  const handleSend = async () => {
    try {
      const message = await sendMessageUseCase.execute(input, 'User');
      setMessages([...messages, MessagePresenter.toViewModel(message)]);
      setInput('');
    } catch (error) {
      console.error('Failed to send message:', error);
    }
  };

  return (
    <div>
      <div>
        {messages.map((msg) => (
          <div key={msg.id}>
            <strong>{msg.sender}</strong>: {msg.content} ({msg.time})
          </div>
        ))}
      </div>
      <input
        value={input}
        onChange={(e) => setInput(e.target.value)}
        placeholder="Type a message..."
      />
      <button onClick={handleSend}>Send</button>
    </div>
  );
};
```

</div>

---

# Putting It All Together: App Component

<div class="max-h-[400px] overflow-y-auto">

```tsx
// src/App.tsx
import React from 'react';
import { ChatBox } from './presentation/components/ChatBox';
import { SendMessageUseCase } from './domain/usecases/SendMessageUseCase';
import { GetMessagesUseCase } from './domain/usecases/GetMessagesUseCase';
import { MessageRepository } from './domain/usecases/SendMessageUseCase';
import { Message } from './domain/entities/Message';

// Mock implementation of MessageRepository
class MockMessageRepository implements MessageRepository {
  private messages: Message[] = [];

  async saveMessage(message: Message): Promise<void> {
    this.messages.push(message);
    console.log('Message saved:', message);
  }

  async getMessages(): Promise<Message[]> {
    return this.messages;
  }
}

const App: React.FC = () => {
  const messageRepository = new MockMessageRepository();
  const sendMessageUseCase = new SendMessageUseCase(messageRepository);
  const getMessagesUseCase = new GetMessagesUseCase(messageRepository);

  return (
    <div>
      <h1>Clean Architecture Chat App</h1>
      <ChatBox 
        sendMessageUseCase={sendMessageUseCase}
        getMessagesUseCase={getMessagesUseCase}
      />
    </div>
  );
};

export default App;
```

</div>

---

# Benefits of Clean Architecture in Front-end

1. **Separation of Concerns**: UI logic is separate from business logic
2. **Testability**: Easy to unit test business logic without UI dependencies
3. **Flexibility**: Can easily swap out UI framework or data sources
4. **Maintainability**: Changes in one layer don't affect others
5. **Scalability**: Easy to add new features or modify existing ones

---

# Conclusion

Clean Architecture provides a robust framework for building scalable, maintainable, and testable applications, both in back-end and front-end development.

- 🏗️ Separates concerns into distinct layers
- 🔄 Enforces the dependency rule
- 🧪 Facilitates testing and maintenance
- 🔌 Allows for easy integration of new features and technologies

Remember: The goal is to create an architecture that tells readers about the system, not about the frameworks you used to build it.

---
layout: center
class: text-center
---

# Thank You!

[Learn more about Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)