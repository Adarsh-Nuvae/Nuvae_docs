# Prisma A–Z Guide

This README will walk you through **everything about Prisma**, from installing it, adding models, running migrations, seeding the database, querying data, and finally deploying your Prisma project.

---

## 📦 1. Installation

First, set up a Node.js project and install Prisma CLI.

```bash
# Initialize a new Node.js project
npm init -y

# Install Prisma and database client (e.g., PostgreSQL)
npm install prisma @prisma/client

# Install Prisma CLI globally (optional)
npm install -g prisma
```

---

## 📂 2. Initialize Prisma

Create the Prisma setup in your project:

```bash
npx prisma init
```

This command creates:

* A `prisma/` folder containing `schema.prisma`
* A `.env` file for environment variables

Edit the `.env` file to set your database URL:

```env
DATABASE_URL="postgresql://USER:PASSWORD@HOST:PORT/DATABASE?schema=public"
```

---

## 📜 3. Understanding `schema.prisma`

The Prisma schema defines:

* The database provider
* Models (represent tables)
* Relationships between models

Example:

```prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

model User {
  id        Int      @id @default(autoincrement())
  name      String
  email     String   @unique
  posts     Post[]
  createdAt DateTime @default(now())
}

model Post {
  id        Int      @id @default(autoincrement())
  title     String
  content   String?
  published Boolean  @default(false)
  author    User     @relation(fields: [authorId], references: [id])
  authorId  Int
  createdAt DateTime @default(now())
}
```

---

## 📥 4. Apply Migrations

Whenever you modify `schema.prisma`, you must run a migration:

```bash
# Create a new migration
npx prisma migrate dev --name init

# Push schema changes without migration history (for prototyping)
npx prisma db push

# Reset the database (use with caution!)
npx prisma migrate reset
```

---

## 🌱 5. Seeding the Database

Add seed data in `prisma/seed.js`:

```javascript
import { PrismaClient } from '@prisma/client';
const prisma = new PrismaClient();

async function main() {
  await prisma.user.create({
    data: {
      name: 'John Doe',
      email: 'john@example.com',
      posts: {
        create: { title: 'Hello Prisma' }
      }
    },
  });
}

main()
  .catch(e => console.error(e))
  .finally(async () => await prisma.$disconnect());
```

Run the seed script:

```bash
npx prisma db seed
```

Configure seeding in `package.json`:

```json
"prisma": {
  "seed": "node prisma/seed.js"
}
```

---

## 🔎 6. Using Prisma Client

Generate Prisma Client whenever you update your schema:

```bash
npx prisma generate
```

Example usage in your application:

```javascript
import { PrismaClient } from '@prisma/client';
const prisma = new PrismaClient();

async function main() {
  const users = await prisma.user.findMany({ include: { posts: true } });
  console.log(users);
}

main();
```

Common commands:

```javascript
// Create a new user
await prisma.user.create({ data: { name: 'Alice', email: 'alice@example.com' } });

// Find a user by ID
await prisma.user.findUnique({ where: { id: 1 } });

// Update a user
await prisma.user.update({
  where: { id: 1 },
  data: { name: 'Alice Updated' }
});

// Delete a user
await prisma.user.delete({ where: { id: 1 } });

// Fetch all posts
await prisma.post.findMany();
```

---

## 📈 7. Prisma Studio

Prisma Studio provides a GUI to view and edit your database:

```bash
npx prisma studio
```

---

## 🚀 8. Deployment

### 1. Prepare Production Database

Ensure you have a production-ready database (e.g., PostgreSQL on AWS, Railway, Neon, Supabase, etc.).

Update the `DATABASE_URL` in `.env`:

```env
DATABASE_URL="postgresql://USER:PASSWORD@HOST:PORT/PROD_DB"
```

### 2. Run Migrations in Production

```bash
npx prisma migrate deploy
```

### 3. Build and Start the App

```bash
npm run build
npm start
```

Make sure you set environment variables correctly on the production server.

---

## 📋 9. Useful Prisma Commands Cheat Sheet

| Task                        | Command                                |
| --------------------------- | -------------------------------------- |
| Initialize Prisma           | `npx prisma init`                      |
| Generate Prisma Client      | `npx prisma generate`                  |
| Run Development Migration   | `npx prisma migrate dev --name <name>` |
| Push Schema to Database     | `npx prisma db push`                   |
| Deploy Production Migration | `npx prisma migrate deploy`            |
| Reset Database              | `npx prisma migrate reset`             |
| Seed Database               | `npx prisma db seed`                   |
| Open Prisma Studio          | `npx prisma studio`                    |

---

## ✅ 10. Best Practices

* Always keep your `schema.prisma` as the **source of truth**.
* Run `npx prisma generate` after modifying your schema.
* Use `migrate dev` during development and `migrate deploy` in production.
* Keep your `.env` file secure and never commit it to version control.
* Use proper indexing and constraints in models for performance and data integrity.

---

## 📚 References

* [Prisma Docs](https://www.prisma.io/docs/)
* [Prisma Migrate](https://www.prisma.io/docs/concepts/components/prisma-migrate)
* [Prisma Client API Reference](https://www.prisma.io/docs/reference/api-reference/prisma-client-reference)
* [Prisma Studio](https://www.prisma.io/studio)
