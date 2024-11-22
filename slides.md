---
theme: seriph
background: https://picsum.photos/seed/sum/1920/1080
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

1. **Entities**: Enterprise-wide business rules
2. **Use Cases**: Application-specific business rules
3. **Interface Adapters**: Converts data between use cases and external agencies
4. **Frameworks and Drivers**: External frameworks, tools, and delivery mechanisms

<div class="flex justify-center">
  <img src="https://blog.cleancoder.com/uncle-bob/images/2012-08-13-the-clean-architecture/CleanArchitecture.jpg" class="h-80 rounded shadow" />
</div>

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
layout: center
class: text-center
---

# Code Example for Node.js Application

An `express.js` application with create user API.

---

# Code Example: Project Structure

```sh
src
├── domain
│   ├── entities
│   │   └── user.ts
│   ├── repositories
│   │   └── user-repository.ts
│   └── usecases
│       └── create-user-usecase.ts
├── infrastructure
│   └── database
│       └── mongo-user-repository.ts
├── interfaces
│   └── controller
│       └── user-controller.ts
└── main.ts
```

---

# Code Example: Entity

<div class="max-h-[400px] overflow-y-auto">

```typescript
// src/domain/entities/user.ts
export class User {
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

# Code Example: Repository

<div class="max-h-[400px] overflow-y-auto">

```typescript
// src/domain/repositories/user-repository.ts
export interface UserRepository {
  save(user: User): Promise<void>;
  findById(id: string): Promise<User | null>;
}
```

</div>

---

# Code Example: Use Case

<div class="max-h-[400px] overflow-y-auto">

```typescript
// src/domain/usecases/create-user-usecase.ts
import { UserRepository } from '../repositories/user-repository'
import { User } from '../entities/user'

export class CreateUserUseCase {
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

# Code Example: Interface Adapter - Controller

<div class="max-h-[400px] overflow-y-auto">

```typescript
// src/interfaces/controller/user-controller.ts
export class UserController {
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

<div class="max-h-[500px] overflow-y-auto">

```typescript
// src/infrastructure/database/mongo-user-repository.ts
import { MongoClient } from 'mongodb';
import { UserRepository } from '../../domain/repositories/user-repository';
import { User } from '../../domain/entities/user';

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

# Code Example: Putting It All Together in App

<div class="max-h-[400px] overflow-y-auto">

```typescript
// src/main.ts
import express from 'express';
import { MongoUserRepository } from './infrastructure/database/mongo-user-repository';
import { CreateUserUseCase } from './domain/usecases/create-user-usecase';
import { UserController } from './interfaces/controllers/user-controller';

const app = express();
const mongoRepo = new MongoUserRepository('mongodb://localhost:27017');
const createUserUseCase = new CreateUserUseCase(mongoRepo);
const userController = new UserController(createUserUseCase);

app.post('/users', (req, res) => userController.createUser(req, res));

app.listen(3000, () => console.log('Server running on port 3000'));
```

</div>

---
layout: center
class: text-center
---

# Code Example for Front-end Application

A `React` application with chat room.

---

# Code Example: Project Structure

```sh
src
├── domain
│   ├── entities
│   │   └── message.ts
│   ├── repositories
│   │   └── message-repository.ts
│   └── usecases
│       ├── send-message-usecase.ts
│       └── get-message-usecase.ts
├── infrastructure
│   └── repositories
│       └── mock-message-repository.ts
├── presentation
│   └── message-presenter.ts
├── ui
│   └── components
│       └── chat-box
│           ├── chat-box.tsx
│           └── use-chat-box.ts
└── app.tsx
```

---

# Code Example: Entity

<div class="max-h-[400px] overflow-y-auto">

```typescript
// src/domain/entities/message-data.ts
export interface MessageData {
  id: string;
  content: string;
  sender: string;
  timestamp: Date;
}
```

</div>

---

# Code Example: Entity

<div class="max-h-[500px] overflow-y-auto">

```typescript
// src/domain/entities/message.ts
import { MessageData } from './message-data';

export class Message {
  constructor(
    public id: string,
    public content: string,
    public sender: string
  ) {}

  isValid(): boolean {
    return this.content.trim().length > 0 && this.sender.trim().length > 0;
  }

  toData(): MessageData {
    return {
      id: this.id,
      content: this.content,
      sender: this.sender
    }
  }: 
}
```

</div>

---

# Code Example: Repository

<div class="max-h-[400px] overflow-y-auto">

```typescript
// src/domain/repositories/message-repository.ts
import { Message } from '../entities/message';

export interface MessageRepository {
  saveMessage(message: Message): Promise<void>;
  getMessages(): Promise<Message[]>;
}
```

</div>

---

# Code Example: Use Case - SendMessageUseCase

<div class="max-h-[500px] overflow-y-auto">

```typescript
// src/domain/usecases/send-message-usecase.ts
import { Message } from '../entities/message';
import { MessageRepository } from '../repositories/message-repository';

export class SendMessageUseCase {
  constructor(private messageRepository: MessageRepository) {}

  async execute(content: string, sender: string): Promise<Message> {
    const message = new Message(
      Date.now().toString(),
      content,
      sender
    );

    if (!message.isValid()) {
      throw new Error('Invalid message');
    }

    await this.messageRepository.saveMessage(message);
    return message;
  }
}
```

</div>

---

# Code Example: Use Case - GetMessageUseCase

<div class="max-h-[400px] overflow-y-auto">

```typescript
// src/domain/usecases/get-message-usecase.ts
import { Message } from '../entities/message';
import { MessageRepository } from '../repositories/message-repository';

export class GetMessagesUseCase {
  constructor(private messageRepository: MessageRepository) {}

  async execute(): Promise<Message[]> {
    return await this.messageRepository.getMessages();
  }
}
```

</div>

---

# Code Example: Interfaces Adapeter - Presenter

<div class="max-h-[400px] overflow-y-auto">

```typescript
// src/presentation/message-presenter.ts
import { Message } from '../domain/entities/message';
import { MessageData } from '../domain/entities/message-data';
import { SendMessageUseCase } from '../domain/usecases/send-message-usecase.ts';
import { GetMessagesUseCase } from '../domain/usecases/get-messages-usecase.ts';

export class MessagePresenter {
  constructor(private sendMessageUseCase: SendMessageUseCase, private getMessagesUseCase: GetMessagesUseCase) {}

  async getMessages(): Promise<MessageData[]> {
    return this.getMessagesUseCase.execute();
  }

  async sendMessage(content: string, sender: string): Promise<Message> {
    return this.sendMessageUseCase.execute(content, sender);
  }
}
```

</div>

---

# Code Example: Frameworks and Drivers

<div class="max-h-[400px] overflow-y-auto">

```typescript
// src/infrastructure/repositories/mock-message-repository.ts
import { Message } from '../../domain/entities/message';
import { MessageRepository } from '../../domain/repositories/message-repository';

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
```

</div>

---

# Code Example: Frameworks and Drivers - UI

<div class="max-h-[500px] overflow-y-auto">

```tsx
// src/ui/components/chat-box/chat-box.tsx
import React from 'react';
import { useChatBox } from './use-chat-box';

interface ChatBoxProps {
  messagePresenter: MessagePresenter
}

export const ChatBox: React.FC<ChatBoxProps> = ({ messagePresenter }) => {
  const { messages, input, setInput, handleSend } = useChatBox(messagePresenter);

  return (
    <div>
      <div>
        {messages.map((msg) => (<div key={msg.id}><strong>{msg.sender}</strong>: {msg.content}</div>))}
      </div>
      <input value={input} onChange={(e) => setInput(e.target.value)} placeholder="Type a message..." />
      <button onClick={handleSend}>Send</button>
    </div>
  );
};
```

</div>

---

# Code Example: Frameworks and Drivers - UI

<div class="max-h-[500px] overflow-y-auto">

```tsx
// src/ui/components/chat-box/hooks/use-chat-box.ts
import React, { useState, useEffect } from 'react';
import { MessageData } from '../../../domain/entities/message-data';
import { MessagePresenter } from '../../../interfaces/presenters/message-presenter';

export const useChatBox = (messagePresenter: MessagePresenter) => {
  const [messages, setMessages] = useState<MessageData[]>([]);
  const [input, setInput] = useState('');

  useEffect(() => {
    const fetchMessages = async () => {
      const fetchedMessages = await messagePresenter.getMessages();
      setMessages(fetchedMessages.map(message => message.toData()));
    };
    fetchMessages();
  }, [messagePresenter]);

  const handleSend = async () => {
    const message = await messagePresenter.sendMessage(input, 'User');
    setMessages([...messages, message.toData()]);
    setInput('');
  };
  return { messages, input, setInput, handleSend }
};
```

</div>

---

# Code Example: Putting It All Together - App

<div class="max-h-[500px] overflow-y-auto">

```tsx
// src/app.tsx
import React from 'react';
import { ChatBox } from './ui/components/chat-box/chat-box';
import { SendMessageUseCase } from './domain/usecases/send-message-usecase';
import { GetMessagesUseCase } from './domain/usecases/get-messages-usecase';
import { MockMessageRepository } from './infrastructure/repositories/mock-message-repository';
import { MessagePresenter } from './interfaces/presenters/message-presenter';

const App: React.FC = () => {
  const messageRepository = new MockMessageRepository();
  const sendMessageUseCase = new SendMessageUseCase(messageRepository);
  const getMessagesUseCase = new GetMessagesUseCase(messageRepository);
  const messagePresenter = new MessagePresenter(sendMessageUseCase, getMessagesUseCase);

  return (
    <div>
      <h1>Clean Architecture Chat App</h1>
      <ChatBox messagePresenter={messagePresenter} />
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