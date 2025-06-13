# GitHub Copilot Instructions

This file provides guidance to GitHub Copilot when working with code in this repository.

## Project Context

This is a learning project for React and Rails beginners:

### Target Audience
- Developers familiar with traditional Rails but new to API-only mode
- React beginners learning TypeScript alongside React
- Following structured tutorial materials

### Technology Stack
- **Backend**: Ruby on Rails 8.0 (API-only mode)
- **Frontend**: React 19 with TypeScript and Vite
- **Database**: PostgreSQL 16
- **Containerization**: Docker & Docker Compose

## Special Instructions for Questions and Learning Support

**IMPORTANT**: This project is being developed by React and Rails beginners. When users ask questions or when you help with implementations:

### 1. Document All Questions
Create files in the `questions/` directory to record:
- What questions were asked
- How you answered them
- Any code examples or explanations provided

### 2. File Naming Convention
Use descriptive names like:
- `questions/react-component-basics.md`
- `questions/rails-api-setup.md`
- `questions/typescript-interface-design.md`

### 3. Markdown Format
Each question file should include:

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

### 4. Learning Context
Always consider that users are:
- New to React concepts (components, hooks, state management)
- Familiar with Rails but new to API-only mode
- Learning TypeScript alongside React
- Following the tutorial materials in the `tutorial/` directory

### 5. Reference Tutorial Materials
When answering, reference relevant sections from:
- `tutorial/01-rails-api-basics/` - Rails API mode fundamentals
- `tutorial/02-react-basics/` - React components and state management  
- `tutorial/03-api-react-integration/` - API communication patterns
- `tutorial/04-practical-todo-app/` - Practical implementation examples

### 6. Code Suggestions Guidelines
When providing code suggestions:
- Use TypeScript interfaces for type safety
- Follow React functional component patterns
- Use Rails API conventions (JSON responses, RESTful routes)
- Include proper error handling
- Add comments explaining concepts for beginners
- Reference existing patterns in the codebase

### 7. Common Topics to Document
- React component design patterns
- TypeScript interface definitions
- Rails API controller implementations
- HTTP client configuration (axios)
- State management with useState
- API integration patterns
- Error handling strategies
- Docker environment issues

## Development Commands

### Environment Setup
```bash
docker compose up -d        # Start all services
docker compose logs -f      # View logs
```

### Rails Commands
```bash
bundle exec rails s -p 3000 -b '0.0.0.0'  # Start server
bundle exec rails db:migrate               # Run migrations
bundle exec rspec                          # Run tests
bundle exec rubocop                        # Code linting
```

### Frontend Commands
```bash
pnpm install    # Install dependencies
pnpm dev        # Start dev server
pnpm build      # Build for production
pnpm lint       # Run ESLint
```

Remember: The goal is to help beginners learn effectively while building a practical application. Always prioritize clear explanations and proper documentation of the learning process.