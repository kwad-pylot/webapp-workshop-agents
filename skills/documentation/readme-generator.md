---
name: readme-generator
description: |
  Generate comprehensive README files for projects.
  Creates documentation for setup, usage, and contribution.
---

# README Generator Skill

## Purpose
Create professional, comprehensive README files that help users understand and use the project.

## README Template

```markdown
# Project Name

[![CI](https://github.com/username/repo/actions/workflows/ci.yml/badge.svg)](https://github.com/username/repo/actions)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.0-blue.svg)](https://www.typescriptlang.org/)

Brief description of what this project does and who it's for.

## Features

- Feature 1 - Brief description
- Feature 2 - Brief description
- Feature 3 - Brief description

## Demo

[Live Demo](https://demo.example.com) | [Screenshots](#screenshots)

## Tech Stack

| Category | Technologies |
|----------|-------------|
| Frontend | React, TypeScript, Tailwind CSS |
| Backend | Node.js, Express, Prisma |
| Database | PostgreSQL |
| Auth | JWT, bcrypt |
| Deployment | Docker, GitHub Actions |

## Prerequisites

Before you begin, ensure you have the following installed:

- [Node.js](https://nodejs.org/) (v20 or higher)
- [npm](https://www.npmjs.com/) or [pnpm](https://pnpm.io/)
- [PostgreSQL](https://www.postgresql.org/) (v15 or higher)
- [Docker](https://www.docker.com/) (optional, for containerized development)

## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/username/repo.git
cd repo
```

### 2. Install dependencies

```bash
npm install
```

### 3. Set up environment variables

```bash
cp .env.example .env
```

Edit `.env` with your configuration:

```env
DATABASE_URL=postgresql://user:password@localhost:5432/dbname
JWT_SECRET=your-secret-key
```

### 4. Set up the database

```bash
# Run migrations
npm run db:migrate

# Seed with sample data (optional)
npm run db:seed
```

### 5. Start the development server

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) in your browser.

## Usage

### Basic Example

```typescript
import { createClient } from '@myapp/client';

const client = createClient({
  apiKey: 'your-api-key',
});

const users = await client.users.list();
```

### Configuration Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `apiKey` | string | - | Your API key (required) |
| `baseUrl` | string | `https://api.example.com` | API base URL |
| `timeout` | number | `30000` | Request timeout in ms |

## Scripts

| Script | Description |
|--------|-------------|
| `npm run dev` | Start development server |
| `npm run build` | Build for production |
| `npm start` | Start production server |
| `npm test` | Run tests |
| `npm run lint` | Run linter |
| `npm run type-check` | Run TypeScript checks |
| `npm run db:migrate` | Run database migrations |
| `npm run db:seed` | Seed database |
| `npm run db:studio` | Open Prisma Studio |

## Project Structure

```
├── src/
│   ├── app/              # Next.js app router
│   │   ├── (auth)/       # Auth routes (login, register)
│   │   ├── (dashboard)/  # Dashboard routes
│   │   └── api/          # API routes
│   ├── components/       # React components
│   │   ├── ui/           # Base UI components
│   │   └── features/     # Feature-specific components
│   ├── lib/              # Utility functions
│   ├── hooks/            # Custom React hooks
│   ├── types/            # TypeScript types
│   └── styles/           # Global styles
├── prisma/
│   ├── schema.prisma     # Database schema
│   ├── migrations/       # Database migrations
│   └── seed.ts           # Seed data
├── public/               # Static assets
├── tests/                # Test files
└── docs/                 # Documentation
```

## API Reference

See [API Documentation](./docs/api.md) for detailed endpoint documentation.

### Quick Reference

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/auth/login` | User login |
| POST | `/api/auth/register` | User registration |
| GET | `/api/users` | List users |
| GET | `/api/users/:id` | Get user by ID |

## Configuration

### Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `DATABASE_URL` | Yes | - | PostgreSQL connection string |
| `JWT_SECRET` | Yes | - | Secret for JWT signing |
| `PORT` | No | `3000` | Server port |
| `NODE_ENV` | No | `development` | Environment mode |

## Testing

```bash
# Run all tests
npm test

# Run with coverage
npm run test:coverage

# Run specific test file
npm test -- path/to/test.ts

# Run in watch mode
npm run test:watch
```

## Deployment

### Docker

```bash
# Build image
docker build -t myapp .

# Run container
docker run -p 3000:3000 --env-file .env myapp
```

### Vercel

```bash
vercel deploy --prod
```

### Manual Deployment

```bash
npm run build
npm start
```

## Contributing

Contributions are welcome! Please read our [Contributing Guide](CONTRIBUTING.md) first.

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## Troubleshooting

### Common Issues

**Database connection failed**
- Ensure PostgreSQL is running
- Check `DATABASE_URL` in `.env`
- Verify database exists

**Build fails**
- Clear `node_modules` and reinstall
- Check Node.js version (v20+ required)

## Roadmap

- [ ] Feature 1
- [ ] Feature 2
- [ ] Feature 3

See [open issues](https://github.com/username/repo/issues) for more.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- [Dependency 1](https://link.com) - Description
- [Dependency 2](https://link.com) - Description

## Contact

Your Name - [@twitter](https://twitter.com/username) - email@example.com

Project Link: [https://github.com/username/repo](https://github.com/username/repo)
```

## Section Guidelines

### Essential Sections
1. Title and description
2. Installation/setup
3. Basic usage
4. Configuration

### Recommended Sections
- Features list
- Tech stack
- Project structure
- API reference
- Contributing guide

### Optional Sections
- Demo/screenshots
- Roadmap
- Troubleshooting
- Acknowledgments

## Quality Checklist

- [ ] Clear project description
- [ ] Prerequisites listed
- [ ] Step-by-step setup
- [ ] All scripts documented
- [ ] Environment variables explained
- [ ] Project structure shown
- [ ] License included
- [ ] Contact information present
