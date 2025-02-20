# **Step-by-Step Guide to Using Node.js with NestJS**
NestJS is a progressive Node.js framework for building efficient, reliable, and scalable server-side applications using TypeScript.

---

### **Step 1: Install Node.js and npm**
Ensure you have **Node.js** installed on your machine. You can check your version by running:

```sh
node -v
npm -v
```

If you don't have Node.js installed, download it from [Node.js official site](https://nodejs.org/) and install it.

---

### **Step 2: Install the NestJS CLI**
NestJS provides a CLI tool to help bootstrap projects. Install it globally using:

```sh
npm install -g @nestjs/cli
```

Verify installation:

```sh
nest --version
```

---

### **Step 3: Create a New NestJS Project**
Run the following command to create a new project:

```sh
nest new my-nest-app
```

- The CLI will prompt you to choose a package manager (npm, yarn, or pnpm). Select one and wait for the dependencies to install.

Navigate to the project directory:

```sh
cd my-nest-app
```

---

### **Step 4: Run the Application**
Start the NestJS application in development mode:

```sh
npm run start
```

Or with auto-restart (watch mode):

```sh
npm run start:dev
```

Your app should now be running on **http://localhost:3000**.

---

### **Step 5: Understanding the Folder Structure**
- `src/`
  - `app.controller.ts` – Handles incoming requests and responses.
  - `app.service.ts` – Contains business logic.
  - `app.module.ts` – The root module.
  - `main.ts` – The entry point (bootstraps the application).
- `test/` – Test files.
- `package.json` – Dependencies and scripts.

---

### **Step 6: Create a New Module, Controller, and Service**
NestJS follows a modular architecture. Let's create a new module called `users`:

```sh
nest generate module users
```

Next, create a controller for handling routes:

```sh
nest generate controller users
```

And a service for business logic:

```sh
nest generate service users
```

This will generate:
```
src/users/
  ├── users.controller.ts
  ├── users.service.ts
  ├── users.module.ts
```

---

### **Step 7: Define a Basic Controller**
Edit `src/users/users.controller.ts`:

```ts
import { Controller, Get } from '@nestjs/common';

@Controller('users')
export class UsersController {
  @Get()
  getAllUsers() {
    return [{ id: 1, name: 'John Doe' }];
  }
}
```

Now, if you go to **http://localhost:3000/users**, you should see:

```json
[{"id":1,"name":"John Doe"}]
```

---

### **Step 8: Implement Dependency Injection**
Edit `src/users/users.service.ts`:

```ts
import { Injectable } from '@nestjs/common';

@Injectable()
export class UsersService {
  private users = [{ id: 1, name: 'John Doe' }];

  getAllUsers() {
    return this.users;
  }
}
```

Modify `users.controller.ts` to use the service:

```ts
import { Controller, Get } from '@nestjs/common';
import { UsersService } from './users.service';

@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Get()
  getAllUsers() {
    return this.usersService.getAllUsers();
  }
}
```

Now the data comes from `UsersService`, following the NestJS architecture.

---

### **Step 9: Use DTOs and Validation**
Create a Data Transfer Object (DTO) for handling user input:

```sh
mkdir src/users/dto
touch src/users/dto/create-user.dto.ts
```

Edit `src/users/dto/create-user.dto.ts`:

```ts
export class CreateUserDto {
  name: string;
}
```

Modify `users.controller.ts` to accept a POST request:

```ts
import { Controller, Get, Post, Body } from '@nestjs/common';
import { UsersService } from './users.service';
import { CreateUserDto } from './dto/create-user.dto';

@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Get()
  getAllUsers() {
    return this.usersService.getAllUsers();
  }

  @Post()
  createUser(@Body() createUserDto: CreateUserDto) {
    return this.usersService.createUser(createUserDto);
  }
}
```

Modify `users.service.ts` to handle user creation:

```ts
import { Injectable } from '@nestjs/common';
import { CreateUserDto } from './dto/create-user.dto';

@Injectable()
export class UsersService {
  private users = [{ id: 1, name: 'John Doe' }];

  getAllUsers() {
    return this.users;
  }

  createUser(user: CreateUserDto) {
    const newUser = { id: this.users.length + 1, ...user };
    this.users.push(newUser);
    return newUser;
  }
}
```

Now you can send a `POST` request to `http://localhost:3000/users` with:

```json
{
  "name": "Jane Doe"
}
```

---

### **Step 10: Connect to a Database (PostgreSQL with TypeORM)**
1. Install TypeORM and PostgreSQL driver:

```sh
npm install @nestjs/typeorm typeorm pg
```

2. Configure database connection in `app.module.ts`:

```ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'postgres',
      host: 'localhost',
      port: 5432,
      username: 'your_db_user',
      password: 'your_db_password',
      database: 'your_db_name',
      autoLoadEntities: true,
      synchronize: true,
    }),
  ],
})
export class AppModule {}
```

3. Define a User entity (`src/users/user.entity.ts`):

```ts
import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;
}
```

4. Register the entity in `users.module.ts`:

```ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { UsersService } from './users.service';
import { UsersController } from './users.controller';
import { User } from './user.entity';

@Module({
  imports: [TypeOrmModule.forFeature([User])],
  controllers: [UsersController],
  providers: [UsersService],
})
export class UsersModule {}
```

5. Modify `users.service.ts` to use TypeORM:

```ts
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from './user.entity';
import { CreateUserDto } from './dto/create-user.dto';

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private usersRepository: Repository<User>,
  ) {}

  getAllUsers() {
    return this.usersRepository.find();
  }

  createUser(user: CreateUserDto) {
    return this.usersRepository.save(user);
  }
}
```

Now, your app is connected to PostgreSQL, and users are saved in the database.

---

### **Conclusion**
You now have a NestJS app that:
✅ Handles GET and POST requests  
✅ Uses Dependency Injection  
✅ Validates input with DTOs  
✅ Connects to PostgreSQL with TypeORM  

To explore more, check out the [NestJS documentation](https://docs.nestjs.com/). 🚀
