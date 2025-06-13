# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Development Environment Setup
- `docker compose build` - Build Docker containers
- `docker compose up -d` - Start all services in detached mode
- Frontend runs on http://localhost:8080
- Backend API runs on http://localhost:3000

### Rails (Backend) Commands
- `bundle exec rails db:create` - Create database
- `bundle exec rails db:migrate` - Run migrations
- `bundle exec rails s -p 3000 -b '0.0.0.0'` - Start Rails server
- `bundle exec rspec` - Run tests

### Frontend Commands (in front/ directory)
- `pnpm install` - Install dependencies
- `pnpm dev` - Start development server
- `pnpm build` - Build for production
- `pnpm lint` - Run ESLint

### Code Quality
- `bundle exec rubocop` - Run Ruby linter
- `pnpm lint` - Run JavaScript/TypeScript linter

## Architecture Overview

This is a full-stack web application with:

### Backend (Ruby on Rails 8.0)
- **API-only Rails application** serving JSON responses
- **PostgreSQL database** with separate instances for development and testing
- **API versioning** with namespace `api/v1/`
- **CORS enabled** for cross-origin requests from frontend
- **Docker containerized** with health checks for database

### Frontend (React + TypeScript + Vite)
- **React 19** with TypeScript
- **Vite** as build tool and dev server
- **ESLint** for code quality
- **Axios** for API communication
- **Docker containerized** with Node.js 20

### Project Structure
- `app/` - Rails application code
  - `controllers/api/v1/` - API controllers (versioned)
  - `models/` - ActiveRecord models
- `front/` - React frontend application
  - `src/` - Source code
  - `package.json` - Node.js dependencies
- `spec/` - RSpec tests
- `docker/` - Docker configuration files
- `compose.yml` - Docker Compose configuration

### Database Configuration
- Development: PostgreSQL running in Docker container
- Test: Separate PostgreSQL instance for testing
- Environment variables for database connection (DATABASE_HOST, DATABASE_USER, DATABASE_PASSWORD)

### Key Technologies
- **Ruby 3.3.4** with Rails 8.0
- **React 19** with TypeScript
- **PostgreSQL 16** as database
- **pnpm** as Node.js package manager
- **Docker & Docker Compose** for containerization

## Special Instructions for Questions and Learning Support

**IMPORTANT**: This project is being developed by React and Rails beginners. When users ask questions or when you help with implementations:

1. **Document All Questions**: Create files in the `questions/` directory to record:
   - What questions were asked
   - How you answered them
   - Any code examples or explanations provided

2. **File Naming Convention**: Use descriptive names like `questions/react-component-basics.md` or `questions/rails-api-setup.md`

3. **Markdown Format**: Each question file should include:
   ```markdown
   # [Topic Title]
   
   ## 質問 (Question)
   [User's original question in Japanese]
   
   ## 回答 (Answer)
   [Your detailed explanation in Japanese]
   
   ## コード例 (Code Examples)
   [Any code examples you provided]
   
   ## 関連資料 (Related Resources)
   [Links to tutorial sections or external resources]
   
   ## 日時 (Date)
   [Date of the question]
   ```

4. **Learning Context**: Always consider that users are:
   - New to React concepts
   - Familiar with Rails but new to API-only mode
   - Learning TypeScript alongside React
   - Following the tutorial materials in the `tutorial/` directory

5. **Reference Tutorial Materials**: When answering, reference relevant sections from:
   - `tutorial/01-rails-api-basics/`
   - `tutorial/02-react-basics/`
   - `tutorial/03-api-react-integration/`
   - `tutorial/04-practical-todo-app/`