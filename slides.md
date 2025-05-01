---
theme: seriph
background: https://picsum.photos/seed/sum/1920/1080
class: 'text-center'
highlighter: shiki
lineNumbers: false
info: |
  ## 整洁架构
  关于整洁架构原则和实现的演示。
drawings:
  persist: false
css: unocss
---

# 整洁架构

构建可维护和可扩展软件的综合指南

<div class="pt-12">
  <span @click="$slidev.nav.next" class="px-2 py-1 rounded cursor-pointer" hover="bg-white bg-opacity-10">
    按空格键进入下一页 <carbon:arrow-right class="inline"/>
  </span>
</div>

<div class="abs-br m-6 flex gap-2">
  <a href="https://github.com/topics/clean-architecture" target="_blank" alt="GitHub"
    class="text-xl icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>

---

# 什么是整洁架构？

整洁架构是一种软件设计哲学，它将设计元素分离成不同的环层。整洁架构的主要规则是源代码依赖只能指向内部。

- 🎯 **关注点分离** - 应用程序不同部分之间的明确边界
- 🔄 **依赖规则** - 依赖指向内部，朝向高级策略，单向依赖
- 🔌 **可插拔** - 核心业务逻辑独立于UI、数据库、框架和外部代理
- 📊 **可测试** - 业务规则可以在没有UI、数据库、Web服务器或任何外部元素的情况下进行测试

---

# 整洁架构的层次

1. **实体（Entities）**：核心的业务实体
2. **用例（Use Cases）**：应用程序特定的业务规则
3. **接口适配器（Interface Adapters）**：在用例和外部代理之间转换数据
4. **框架和驱动（Frameworks and Drivers）**：外部框架、工具和驱动

<div class="flex justify-center">
  <img src="https://blog.cleancoder.com/uncle-bob/images/2012-08-13-the-clean-architecture/CleanArchitecture.jpg" class="h-80 rounded shadow" />
</div>

---

# 整洁架构的好处

- 📦 **独立于框架**：架构不依赖于某些功能丰富的软件库的存在
- 🧪 **可测试**：业务规则可以在没有UI、数据库、Web服务器或任何外部元素的情况下进行测试
- 🖥️ **独立于UI**：UI可以轻松更改，而不改变系统的其余部分
- 💾 **独立于数据库**：你可以将Oracle或SQL Server替换为Mongo、BigTable、CouchDB或其他数据库
- 🔄 **独立于任何外部代理**：你的业务规则不了解外部世界的任何信息

---

# 实现整洁架构

1. **从核心开始**：定义你的实体和用例
2. **向外构建**：最后实现接口适配器和框架
3. **使用依赖倒置**：高级模块不应该依赖于低级模块
4. **创建边界**：使用接口定义层之间的明确边界
5. **遵循SOLID原则**：单一职责、开放封闭、里氏替换、接口隔离和依赖倒置

---
layout: center
class: text-center
---

# Node.js应用程序的代码示例

一个带有创建用户API的`express.js`应用程序。

---

# 代码示例：项目结构

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

# 代码示例：实体

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
    // 电子邮件验证逻辑
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(this.email);
  }
}
```

</div>

---

# 代码示例：数据仓库

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

# 代码示例：用例

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
      throw new Error('无效的电子邮箱');
    }

    await this.userRepository.save(user);
    return user;
  }
}
```

</div>

---

# 代码示例：接口适配器 - 控制器

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

# 代码示例：框架和驱动

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

# 代码示例：将所有内容放在一起

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

# 前端应用程序的代码示例

一个带有聊天室的`React`应用程序。

---

# 代码示例：项目结构

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

# 代码示例：实体

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

# 代码示例：实体

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

# 代码示例：数据仓库

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

# 代码示例：用例 - SendMessageUseCase

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
      throw new Error('无效的消息');
    }

    await this.messageRepository.saveMessage(message);
    return message;
  }
}
```

</div>

---

# 代码示例：用例 - GetMessageUseCase

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

# 代码示例：接口适配器 - Presenter

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

# 代码示例：框架和驱动

<div class="max-h-[400px] overflow-y-auto">

```typescript
// src/infrastructure/repositories/mock-message-repository.ts
import { Message } from '../../domain/entities/message';
import { MessageRepository } from '../../domain/repositories/message-repository';

// Mock实现MessageRepository
class MockMessageRepository implements MessageRepository {
  private messages: Message[] = [];

  async saveMessage(message: Message): Promise<void> {
    this.messages.push(message);
    console.log('消息保存:', message);
  }

  async getMessages(): Promise<Message[]> {
    return this.messages;
  }
}
```

</div>

---

# 代码示例：框架和驱动 - UI

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
      <input value={input} onChange={(e) => setInput(e.target.value)} placeholder="输入消息..." />
      <button onClick={handleSend}>发送</button>
    </div>
  );
};
```

</div>

---

# 代码示例：框架和驱动 - UI

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
    const message = await messagePresenter.sendMessage(input, '用户');
    setMessages([...messages, message.toData()]);
    setInput('');
  };
  return { messages, input, setInput, handleSend }
};
```

</div>

---

# 代码示例：将所有内容放在一起 - App

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
      <h1>整洁架构聊天应用</h1>
      <ChatBox messagePresenter={messagePresenter} />
    </div>
  );
};

export default App;
```

</div>

---

# 整洁架构在前端的好处

1. 📦 **关注点分离**：UI逻辑与业务逻辑分离
2. 🧪 **可测试**：可以在没有UI依赖的情况下测试业务逻辑
3. 🖥️ **灵活性**：可以轻松替换UI框架或数据源
4. 💾 **可维护性**：修改一个层不会影响其他层
5. 🔄 **可扩展性**：可以轻松添加新功能或修改现有功能

---

# 结论

整洁架构为构建可扩展、可维护和可测试的应用程序提供了一个强大的框架，无论是在后端还是前端开发中。

- 🏗️ **将关注点分离成不同的层**
- 🔄 **强制依赖规则**
- 🧪 **促进测试和维护**
- 🔌 **允许轻松集成新功能和技术**

记住：目标是创建一个架构，让读者了解系统，而不是了解你用来构建它的框架。

---
layout: center
class: text-center
---

# 谢谢！

[了解更多关于整洁架构](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)