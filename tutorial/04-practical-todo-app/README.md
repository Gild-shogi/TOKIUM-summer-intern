# 実践編：ToDoアプリ作成

これまでに学んだ知識を活用して、実際のToDoアプリケーションを作成します。Rails APIとReactを使った本格的なCRUDアプリケーションの開発を体験しましょう。

> **💡 学習のコツ**
> 
> この章は総仕上げです！これまでの学習内容を組み合わせて実践的なアプリを作成します。
> - 分からない部分は以前の章や補足資料を参照しましょう
> - 一度に全部を理解しようとせず、段階的に進めましょう
> - エラーが出ても焦らず、メンターやLLMに相談してください
> - 完成したときの達成感を楽しみにして頑張りましょう！

## 参考資料・復習用リンク

実装中に困ったら、これまでの章を振り返ってみてください：

### Rails API関連
- 📖 **[JSONとは何か？](../01-rails-api-basics/what-is-json.md)** - API レスポンスの形式について
- 🔒 **[CORSとは何か？](../01-rails-api-basics/what-is-cors.md)** - フロントエンドとバックエンド間の通信について
- 🏛️ **[StatelessとStateful](../01-rails-api-basics/stateless-vs-stateful.md)** - API設計の考え方について

### React関連
- 🧩 **[コンポーネントとは何か？](../02-react-basics/what-is-component.md)** - UIコンポーネントの作り方
- 🔄 **[useStateの内部動作](../02-react-basics/how-usestate-works.md)** - 状態管理の仕組み
- 🪝 **[React Hooks 一覧](../02-react-basics/react-hooks-overview.md)** - Hooksの使い方

### API連携関連
- 🤝 **[Promiseとは何か？](../03-api-react-integration/understanding-promises.md)** - 非同期処理の基本概念
- ⚡ **[同期処理と非同期処理の違い](../03-api-react-integration/sync-vs-async.md)** - async/awaitの使い方

## 学習目標

- 実際のアプリケーション開発フローを体験する
- 企画から実装まで一通りの流れを理解する
- よく使われるUI/UXパターンを学ぶ
- 実践的なエラーハンドリングを実装する
- テストコードの基本を理解する

## 1. アプリケーション企画

### 作成するToDoアプリの仕様

**基本機能**
- ✅ ToDoアイテムの一覧表示
- ✅ 新しいToDoアイテムの追加
- ✅ ToDoアイテムの完了/未完了切り替え
- ✅ ToDoアイテムの編集
- ✅ ToDoアイテムの削除
- ✅ フィルター機能（全て/未完了/完了済み）

**データ構造**
```
Todo:
- id: number (自動生成)
- title: string (必須)
- description: string (任意)
- completed: boolean (デフォルト: false)
- created_at: datetime
- updated_at: datetime
```

### UI/UXの設計

```
┌─────────────────────────────────────┐
│ 📝 My Todo App                      │
├─────────────────────────────────────┤
│ 新規追加フォーム                     │
│ [タイトル入力] [追加ボタン]          │
├─────────────────────────────────────┤
│ フィルター: [全て] [未完了] [完了済み] │
├─────────────────────────────────────┤
│ ☐ Todo 1 [編集] [削除]              │
│ ☑ Todo 2 [編集] [削除]              │
│ ☐ Todo 3 [編集] [削除]              │
└─────────────────────────────────────┘
```

## 2. Rails API側の実装

> **💡 実装のヒント**
> 
> Rails APIの実装について詳しく知りたい場合は、以下を参照してください：
> - **[Rails APIモード基礎編](../01-rails-api-basics/README.md)** - API設計の基本
> - **[JSONとは何か？](../01-rails-api-basics/what-is-json.md)** - APIレスポンスの形式

### Step 1: Todoモデルの作成

```bash
# Rails コンテナ内でモデル生成
docker compose exec app bundle exec rails generate model Todo title:string description:text completed:boolean

# マイグレーション実行
docker compose exec app bundle exec rails db:migrate
```

### Step 2: Todoモデルの実装

`app/models/todo.rb`:

```ruby
class Todo < ApplicationRecord
  validates :title, presence: true, length: { maximum: 100 }
  validates :description, length: { maximum: 500 }
  
  scope :completed, -> { where(completed: true) }
  scope :pending, -> { where(completed: false) }
  scope :recent, -> { order(created_at: :desc) }
  
  def toggle_completed!
    update!(completed: !completed)
  end
end
```

### Step 3: Todos APIコントローラの実装

`app/controllers/api/v1/todos_controller.rb`:

```ruby
class Api::V1::TodosController < ApplicationController
  before_action :set_todo, only: [:show, :update, :destroy]
  
  def index
    @todos = Todo.recent
    
    # フィルター処理
    case params[:filter]
    when 'completed'
      @todos = @todos.completed
    when 'pending'
      @todos = @todos.pending
    end
    
    render json: {
      todos: @todos.map(&method(:todo_json)),
      meta: {
        total: @todos.count,
        completed: @todos.completed.count,
        pending: @todos.pending.count
      }
    }
  end
  
  def show
    render json: { todo: todo_json(@todo) }
  end
  
  def create
    @todo = Todo.new(todo_params)
    
    if @todo.save
      render json: { 
        todo: todo_json(@todo), 
        message: 'ToDoが作成されました' 
      }, status: :created
    else
      render json: { 
        errors: @todo.errors.full_messages 
      }, status: :unprocessable_entity
    end
  end
  
  def update
    if @todo.update(todo_params)
      render json: { 
        todo: todo_json(@todo), 
        message: 'ToDoが更新されました' 
      }
    else
      render json: { 
        errors: @todo.errors.full_messages 
      }, status: :unprocessable_entity
    end
  end
  
  def destroy
    @todo.destroy!
    render json: { message: 'ToDoが削除されました' }
  end
  
  # 完了状態をトグル
  def toggle
    @todo = Todo.find(params[:id])
    @todo.toggle_completed!
    render json: { 
      todo: todo_json(@todo), 
      message: @todo.completed? ? 'ToDoを完了しました' : 'ToDoを未完了に戻しました'
    }
  end
  
  private
  
  def set_todo
    @todo = Todo.find(params[:id])
  rescue ActiveRecord::RecordNotFound
    render json: { error: 'ToDoが見つかりません' }, status: :not_found
  end
  
  def todo_params
    params.require(:todo).permit(:title, :description, :completed)
  end
  
  def todo_json(todo)
    {
      id: todo.id,
      title: todo.title,
      description: todo.description,
      completed: todo.completed,
      created_at: todo.created_at.strftime('%Y-%m-%d %H:%M'),
      updated_at: todo.updated_at.strftime('%Y-%m-%d %H:%M')
    }
  end
end
```

### Step 4: ルーティングの追加

`config/routes.rb`:

```ruby
Rails.application.routes.draw do
  namespace :api do
    namespace :v1 do
      get '/hello', to: 'hello#index'
      resources :users
      resources :todos do
        member do
          patch :toggle
        end
      end
    end
  end
end
```

### Step 5: サンプルデータの作成

`db/seeds.rb`:

```ruby
# 既存のToDoを削除
Todo.destroy_all

# サンプルデータを作成
todos = [
  {
    title: "Reactの学習",
    description: "チュートリアルを完了する",
    completed: false
  },
  {
    title: "Rails APIの理解",
    description: "RESTful APIの設計について学ぶ",
    completed: true
  },
  {
    title: "ToDoアプリの完成",
    description: "実践的なアプリケーションを作成する",
    completed: false
  }
]

todos.each do |todo_data|
  Todo.create!(todo_data)
end

puts "#{Todo.count}件のToDoが作成されました"
```

```bash
# サンプルデータを投入
docker compose exec app bundle exec rails db:seed
```

## 3. React側の実装

> **💡 実装のヒント**
> 
> Reactの実装について詳しく知りたい場合は、以下を参照してください：
> - **[React基礎編](../02-react-basics/README.md)** - コンポーネントとstate管理
> - **[コンポーネントとは何か？](../02-react-basics/what-is-component.md)** - UIコンポーネントの設計
> - **[useStateの内部動作](../02-react-basics/how-usestate-works.md)** - 状態管理の詳細

### Step 1: Todo関連の型定義

`front/src/types/todo.ts`:

```typescript
export interface Todo {
  id: number;
  title: string;
  description: string;
  completed: boolean;
  created_at: string;
  updated_at: string;
}

export interface CreateTodoData {
  title: string;
  description?: string;
}

export interface UpdateTodoData {
  title?: string;
  description?: string;
  completed?: boolean;
}

export type FilterType = 'all' | 'pending' | 'completed';

export interface TodosResponse {
  todos: Todo[];
  meta: {
    total: number;
    completed: number;
    pending: number;
  };
}
```

### Step 2: Todo APIサービス

> **💡 API通信のヒント**
> 
> API通信について詳しく知りたい場合は、以下を参照してください：
> - **[Rails API + React連携編](../03-api-react-integration/README.md)** - API通信の基本
> - **[Promiseとは何か？](../03-api-react-integration/understanding-promises.md)** - 非同期処理の基本
> - **[同期処理と非同期処理の違い](../03-api-react-integration/sync-vs-async.md)** - async/awaitの使い方

`front/src/services/todoService.ts`:

```typescript
import { api } from '../lib/api';
import { Todo, CreateTodoData, UpdateTodoData, TodosResponse, FilterType } from '../types/todo';

export const todoService = {
  // Todo一覧取得
  async getTodos(filter: FilterType = 'all'): Promise<TodosResponse> {
    const params = filter !== 'all' ? { filter } : {};
    const response = await api.get('/todos', { params });
    return response.data;
  },

  // 単一Todo取得
  async getTodo(id: number): Promise<Todo> {
    const response = await api.get(`/todos/${id}`);
    return response.data.todo;
  },

  // Todo作成
  async createTodo(todoData: CreateTodoData): Promise<Todo> {
    const response = await api.post('/todos', { todo: todoData });
    return response.data.todo;
  },

  // Todo更新
  async updateTodo(id: number, todoData: UpdateTodoData): Promise<Todo> {
    const response = await api.put(`/todos/${id}`, { todo: todoData });
    return response.data.todo;
  },

  // Todo削除
  async deleteTodo(id: number): Promise<void> {
    await api.delete(`/todos/${id}`);
  },

  // 完了状態をトグル
  async toggleTodo(id: number): Promise<Todo> {
    const response = await api.patch(`/todos/${id}/toggle`);
    return response.data.todo;
  }
};
```

### Step 3: Todoアイテムコンポーネント

> **💡 コンポーネント設計のヒント**
> 
> コンポーネントの設計について詳しく知りたい場合は、以下を参照してください：
> - **[コンポーネントとは何か？](../02-react-basics/what-is-component.md)** - 再利用可能なUIコンポーネントの作り方
> - **[React Hooks 一覧](../02-react-basics/react-hooks-overview.md)** - useStateやuseCallbackの使い方

`front/src/components/TodoItem.tsx`:

```typescript
import { useState } from 'react';
import { Todo } from '../types/todo';

interface TodoItemProps {
  todo: Todo;
  onToggle: (id: number) => void;
  onUpdate: (id: number, data: { title: string; description: string }) => void;
  onDelete: (id: number) => void;
}

function TodoItem({ todo, onToggle, onUpdate, onDelete }: TodoItemProps) {
  const [isEditing, setIsEditing] = useState(false);
  const [editTitle, setEditTitle] = useState(todo.title);
  const [editDescription, setEditDescription] = useState(todo.description);

  const handleSave = () => {
    if (editTitle.trim()) {
      onUpdate(todo.id, { title: editTitle, description: editDescription });
      setIsEditing(false);
    }
  };

  const handleCancel = () => {
    setEditTitle(todo.title);
    setEditDescription(todo.description);
    setIsEditing(false);
  };

  if (isEditing) {
    return (
      <div style={{
        border: '1px solid #ddd',
        borderRadius: '8px',
        padding: '16px',
        margin: '8px 0',
        backgroundColor: '#f8f9fa'
      }}>
        <div style={{ marginBottom: '12px' }}>
          <input
            type="text"
            value={editTitle}
            onChange={(e) => setEditTitle(e.target.value)}
            style={{
              width: '100%',
              padding: '8px',
              border: '1px solid #ccc',
              borderRadius: '4px',
              fontSize: '16px'
            }}
            placeholder="タイトル"
          />
        </div>
        <div style={{ marginBottom: '12px' }}>
          <textarea
            value={editDescription}
            onChange={(e) => setEditDescription(e.target.value)}
            style={{
              width: '100%',
              padding: '8px',
              border: '1px solid #ccc',
              borderRadius: '4px',
              minHeight: '60px',
              resize: 'vertical'
            }}
            placeholder="説明（任意）"
          />
        </div>
        <div>
          <button 
            onClick={handleSave}
            style={{
              backgroundColor: '#28a745',
              color: 'white',
              border: 'none',
              padding: '8px 16px',
              borderRadius: '4px',
              marginRight: '8px',
              cursor: 'pointer'
            }}
          >
            保存
          </button>
          <button 
            onClick={handleCancel}
            style={{
              backgroundColor: '#6c757d',
              color: 'white',
              border: 'none',
              padding: '8px 16px',
              borderRadius: '4px',
              cursor: 'pointer'
            }}
          >
            キャンセル
          </button>
        </div>
      </div>
    );
  }

  return (
    <div style={{
      border: '1px solid #ddd',
      borderRadius: '8px',
      padding: '16px',
      margin: '8px 0',
      backgroundColor: todo.completed ? '#f0f0f0' : 'white',
      opacity: todo.completed ? 0.7 : 1
    }}>
      <div style={{ display: 'flex', alignItems: 'center', marginBottom: '8px' }}>
        <input
          type="checkbox"
          checked={todo.completed}
          onChange={() => onToggle(todo.id)}
          style={{ marginRight: '12px', transform: 'scale(1.2)' }}
        />
        <h3 style={{
          margin: 0,
          textDecoration: todo.completed ? 'line-through' : 'none',
          flex: 1
        }}>
          {todo.title}
        </h3>
      </div>
      
      {todo.description && (
        <p style={{
          margin: '8px 0',
          color: '#666',
          fontStyle: todo.description ? 'normal' : 'italic'
        }}>
          {todo.description || '説明なし'}
        </p>
      )}
      
      <div style={{ 
        display: 'flex', 
        justifyContent: 'space-between', 
        alignItems: 'center',
        marginTop: '12px'
      }}>
        <small style={{ color: '#999' }}>
          作成: {todo.created_at}
        </small>
        <div>
          <button 
            onClick={() => setIsEditing(true)}
            style={{
              backgroundColor: '#007bff',
              color: 'white',
              border: 'none',
              padding: '6px 12px',
              borderRadius: '4px',
              marginRight: '8px',
              cursor: 'pointer'
            }}
          >
            編集
          </button>
          <button 
            onClick={() => onDelete(todo.id)}
            style={{
              backgroundColor: '#dc3545',
              color: 'white',
              border: 'none',
              padding: '6px 12px',
              borderRadius: '4px',
              cursor: 'pointer'
            }}
          >
            削除
          </button>
        </div>
      </div>
    </div>
  );
}

export default TodoItem;
```

### Step 4: Todo追加フォーム

> **💡 フォーム処理のヒント**
> 
> フォームの実装について詳しく知りたい場合は、以下を参照してください：
> - **[useStateの内部動作](../02-react-basics/how-usestate-works.md)** - フォームの状態管理
> - **[React基礎編](../02-react-basics/README.md#6-イベントハンドリング)** - イベントハンドリングの基本

`front/src/components/TodoForm.tsx`:

```typescript
import { useState } from 'react';
import { CreateTodoData } from '../types/todo';

interface TodoFormProps {
  onAdd: (todoData: CreateTodoData) => void;
  loading?: boolean;
}

function TodoForm({ onAdd, loading = false }: TodoFormProps) {
  const [title, setTitle] = useState('');
  const [description, setDescription] = useState('');

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    
    if (!title.trim()) return;

    onAdd({
      title: title.trim(),
      description: description.trim() || undefined
    });

    setTitle('');
    setDescription('');
  };

  return (
    <div style={{
      border: '1px solid #ddd',
      borderRadius: '8px',
      padding: '20px',
      marginBottom: '20px',
      backgroundColor: '#f8f9fa'
    }}>
      <h2 style={{ marginTop: 0 }}>新しいToDoを追加</h2>
      
      <form onSubmit={handleSubmit}>
        <div style={{ marginBottom: '16px' }}>
          <input
            type="text"
            value={title}
            onChange={(e) => setTitle(e.target.value)}
            placeholder="何をしますか？"
            style={{
              width: '100%',
              padding: '12px',
              border: '1px solid #ccc',
              borderRadius: '4px',
              fontSize: '16px'
            }}
            disabled={loading}
          />
        </div>
        
        <div style={{ marginBottom: '16px' }}>
          <textarea
            value={description}
            onChange={(e) => setDescription(e.target.value)}
            placeholder="詳細（任意）"
            style={{
              width: '100%',
              padding: '12px',
              border: '1px solid #ccc',
              borderRadius: '4px',
              minHeight: '80px',
              resize: 'vertical'
            }}
            disabled={loading}
          />
        </div>
        
        <button 
          type="submit"
          disabled={!title.trim() || loading}
          style={{
            backgroundColor: '#28a745',
            color: 'white',
            border: 'none',
            padding: '12px 24px',
            borderRadius: '4px',
            cursor: !title.trim() || loading ? 'not-allowed' : 'pointer',
            opacity: !title.trim() || loading ? 0.6 : 1,
            fontSize: '16px'
          }}
        >
          {loading ? '追加中...' : '追加'}
        </button>
      </form>
    </div>
  );
}

export default TodoForm;
```

### Step 5: フィルターコンポーネント

> **💡 フィルター機能のヒント**
> 
> フィルターコンポーネントの実装について詳しく知りたい場合は、以下を参照してください：
> - **[コンポーネントとは何か？](../02-react-basics/what-is-component.md)** - 再利用可能なUIコンポーネントの設計思想
> - **[useStateの内部動作](../02-react-basics/how-usestate-works.md)** - フィルター状態の管理方法

`front/src/components/TodoFilter.tsx`:

```typescript
import { FilterType } from '../types/todo';

interface TodoFilterProps {
  currentFilter: FilterType;
  onFilterChange: (filter: FilterType) => void;
  counts: {
    total: number;
    pending: number;
    completed: number;
  };
}

function TodoFilter({ currentFilter, onFilterChange, counts }: TodoFilterProps) {
  const filters: { key: FilterType; label: string; count: number }[] = [
    { key: 'all', label: '全て', count: counts.total },
    { key: 'pending', label: '未完了', count: counts.pending },
    { key: 'completed', label: '完了済み', count: counts.completed }
  ];

  return (
    <div style={{
      display: 'flex',
      gap: '8px',
      marginBottom: '20px',
      padding: '16px',
      backgroundColor: '#f8f9fa',
      borderRadius: '8px'
    }}>
      <span style={{ marginRight: '12px', fontWeight: 'bold' }}>表示:</span>
      {filters.map(filter => (
        <button
          key={filter.key}
          onClick={() => onFilterChange(filter.key)}
          style={{
            padding: '8px 16px',
            border: '1px solid #007bff',
            borderRadius: '4px',
            backgroundColor: currentFilter === filter.key ? '#007bff' : 'white',
            color: currentFilter === filter.key ? 'white' : '#007bff',
            cursor: 'pointer',
            fontSize: '14px'
          }}
        >
          {filter.label} ({filter.count})
        </button>
      ))}
    </div>
  );
}

export default TodoFilter;
```

### Step 6: メインのTodoアプリコンポーネント

> **💡 統合コンポーネントのヒント**
> 
> メインアプリケーションの実装について詳しく知りたい場合は、以下を参照してください：
> - **[useStateの内部動作](../02-react-basics/how-usestate-works.md)** - 複数の状態管理の方法
> - **[同期処理と非同期処理の違い](../03-api-react-integration/sync-vs-async.md)** - API呼び出しのエラーハンドリング
> - **[React Hooks 一覧](../02-react-basics/react-hooks-overview.md)** - useEffect の適切な使用場面

`front/src/components/TodoApp.tsx`:

```typescript
import { useState, useEffect } from 'react';
import { todoService } from '../services/todoService';
import { Todo, FilterType, TodosResponse, CreateTodoData } from '../types/todo';
import TodoForm from './TodoForm';
import TodoFilter from './TodoFilter';
import TodoItem from './TodoItem';

function TodoApp() {
  const [todos, setTodos] = useState<Todo[]>([]);
  const [filter, setFilter] = useState<FilterType>('all');
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  const [counts, setCounts] = useState({ total: 0, pending: 0, completed: 0 });

  // Todo一覧を取得
  const fetchTodos = async () => {
    try {
      setLoading(true);
      setError(null);
      const response: TodosResponse = await todoService.getTodos(filter);
      setTodos(response.todos);
      setCounts(response.meta);
    } catch (err) {
      setError('Todoの取得に失敗しました');
      console.error('Fetch todos error:', err);
    } finally {
      setLoading(false);
    }
  };

  // Todo追加
  const handleAddTodo = async (todoData: CreateTodoData) => {
    try {
      await todoService.createTodo(todoData);
      await fetchTodos(); // リストを再取得
    } catch (err) {
      setError('Todoの追加に失敗しました');
      console.error('Add todo error:', err);
    }
  };

  // Todo更新
  const handleUpdateTodo = async (id: number, data: { title: string; description: string }) => {
    try {
      await todoService.updateTodo(id, data);
      await fetchTodos(); // リストを再取得
    } catch (err) {
      setError('Todoの更新に失敗しました');
      console.error('Update todo error:', err);
    }
  };

  // Todo削除
  const handleDeleteTodo = async (id: number) => {
    if (!confirm('本当に削除しますか？')) return;

    try {
      await todoService.deleteTodo(id);
      await fetchTodos(); // リストを再取得
    } catch (err) {
      setError('Todoの削除に失敗しました');
      console.error('Delete todo error:', err);
    }
  };

  // 完了状態をトグル
  const handleToggleTodo = async (id: number) => {
    try {
      await todoService.toggleTodo(id);
      await fetchTodos(); // リストを再取得
    } catch (err) {
      setError('Todoの更新に失敗しました');
      console.error('Toggle todo error:', err);
    }
  };

  // フィルター変更
  const handleFilterChange = (newFilter: FilterType) => {
    setFilter(newFilter);
  };

  // 初回とフィルター変更時にデータを取得
  useEffect(() => {
    fetchTodos();
  }, [filter]);

  return (
    <div style={{ maxWidth: '800px', margin: '0 auto', padding: '20px' }}>
      <header style={{ textAlign: 'center', marginBottom: '30px' }}>
        <h1>📝 My Todo App</h1>
      </header>

      {error && (
        <div style={{
          backgroundColor: '#f8d7da',
          color: '#721c24',
          padding: '12px',
          borderRadius: '4px',
          marginBottom: '20px',
          border: '1px solid #f5c6cb'
        }}>
          {error}
          <button 
            onClick={() => setError(null)}
            style={{
              float: 'right',
              background: 'none',
              border: 'none',
              fontSize: '16px',
              cursor: 'pointer'
            }}
          >
            ×
          </button>
        </div>
      )}

      <TodoForm onAdd={handleAddTodo} loading={loading} />
      
      <TodoFilter 
        currentFilter={filter}
        onFilterChange={handleFilterChange}
        counts={counts}
      />

      {loading ? (
        <div style={{ textAlign: 'center', padding: '40px' }}>
          読み込み中...
        </div>
      ) : todos.length === 0 ? (
        <div style={{ 
          textAlign: 'center', 
          padding: '40px',
          color: '#666',
          fontStyle: 'italic'
        }}>
          {filter === 'all' ? 'Todoがありません' : 
           filter === 'pending' ? '未完了のTodoがありません' : 
           '完了済みのTodoがありません'}
        </div>
      ) : (
        <div>
          {todos.map(todo => (
            <TodoItem
              key={todo.id}
              todo={todo}
              onToggle={handleToggleTodo}
              onUpdate={handleUpdateTodo}
              onDelete={handleDeleteTodo}
            />
          ))}
        </div>
      )}

      <footer style={{ 
        textAlign: 'center', 
        marginTop: '40px', 
        color: '#666',
        fontSize: '14px'
      }}>
        <p>Rails API + React で作成したTodoアプリ</p>
      </footer>
    </div>
  );
}

export default TodoApp;
```

### Step 7: App.tsx を更新

`front/src/App.tsx`:

```typescript
import TodoApp from './components/TodoApp';
import './App.css';

function App() {
  return <TodoApp />;
}

export default App;
```

## 4. 動作確認とテスト

> **💡 テストのヒント**
> 
> アプリケーションのテストについて詳しく知りたい場合は、以下を参照してください：
> - **[Rails API基礎編](../01-rails-api-basics/README.md)** - APIエンドポイントの確認方法
> - **[JSONとは何か？](../01-rails-api-basics/what-is-json.md)** - APIレスポンスの形式確認
> - **[CORSとは何か？](../01-rails-api-basics/what-is-cors.md)** - フロントエンド・バックエンド間通信のトラブルシューティング

### Step 1: アプリケーションの起動

```bash
# Docker環境起動
docker compose up -d

# ログ確認
docker compose logs -f

# ブラウザで http://localhost:8080 を開く
```

### Step 2: 機能テスト

1. **Todo追加**: フォームからTodoを追加できることを確認
2. **Todo一覧**: 追加したTodoが表示されることを確認
3. **完了切り替え**: チェックボックスで完了状態を切り替えられることを確認
4. **フィルター**: 全て/未完了/完了済みのフィルターが動作することを確認
5. **編集**: Todoの編集ができることを確認
6. **削除**: Todoの削除ができることを確認

### Step 3: APIテスト（curl）

```bash
# Todo一覧取得
curl http://localhost:3000/api/v1/todos

# Todo作成
curl -X POST http://localhost:3000/api/v1/todos \
  -H "Content-Type: application/json" \
  -d '{"todo":{"title":"新しいTodo","description":"テスト"}}'

# Todo更新
curl -X PUT http://localhost:3000/api/v1/todos/1 \
  -H "Content-Type: application/json" \
  -d '{"todo":{"title":"更新されたTodo"}}'

# 完了状態をトグル
curl -X PATCH http://localhost:3000/api/v1/todos/1/toggle

# Todo削除
curl -X DELETE http://localhost:3000/api/v1/todos/1
```

## 5. 改善とベストプラクティス

> **💡 最適化のヒント**
> 
> アプリケーションの改善について詳しく知りたい場合は、以下を参照してください：
> - **[React Hooks 一覧](../02-react-basics/react-hooks-overview.md)** - useCallback、useMemoの活用方法
> - **[同期処理と非同期処理の違い](../03-api-react-integration/sync-vs-async.md)** - エラーハンドリングの改善手法
> - **[Promiseとは何か？](../03-api-react-integration/understanding-promises.md)** - リトライ機能の実装方法

### パフォーマンス最適化

1. **React.memo でコンポーネントの再レンダリングを最適化**
2. **useCallback で関数の再作成を防ぐ**
3. **楽観的アップデート（Optimistic Update）の実装**

### エラーハンドリングの改善

1. **より詳細なエラーメッセージ**
2. **リトライ機能**
3. **オフライン対応**

### UI/UXの改善

1. **ローディングスピナー**
2. **アニメーション効果**
3. **キーボードショートカット**
4. **ドラッグ&ドロップでの並び替え**

## 6. まとめ

### 学習した内容

- ✅ 実際のアプリケーション開発フロー
- ✅ Rails APIとReactの本格的な連携
- ✅ CRUD操作の完全な実装
- ✅ 状態管理とコンポーネント設計
- ✅ エラーハンドリングとユーザーフィードバック
- ✅ フィルター機能の実装

### 達成したスキル

| スキル | 習得レベル |
|--------|------------|
| **Rails API開発** | ⭐⭐⭐⭐ |
| **React基礎** | ⭐⭐⭐⭐ |
| **TypeScript** | ⭐⭐⭐ |
| **HTTP通信** | ⭐⭐⭐⭐ |
| **状態管理** | ⭐⭐⭐ |
| **UI/UX設計** | ⭐⭐ |

### 振り返り・復習のススメ

このプロジェクトを完成させた今、もう一度これまでの学習を振り返ってみましょう：

- **理解が浅かった部分**: [補足資料](#参考資料復習用リンク)を再読して理解を深める
- **忘れてしまった概念**: 以前の章を復習して知識を定着させる
- **応用したい機能**: 次のステップの内容にチャレンジしてみる

### 次のステップ

より高度な機能を学習したい場合（参考資料も活用してください）：

1. **認証・認可**: JWTトークンを使ったユーザー認証
   - **[StatelessとStateful](../01-rails-api-basics/stateless-vs-stateful.md)** を参考にAPI設計を理解
2. **リアルタイム機能**: WebSocketを使った即座の更新
   - **[同期処理と非同期処理の違い](../03-api-react-integration/sync-vs-async.md)** で非同期処理を復習
3. **テスト**: RSpec（Rails）とJest（React）によるテスト
   - **[React基礎編](../02-react-basics/README.md)** でコンポーネントの構造を再確認
4. **デプロイ**: HerokuやVercelへのデプロイ
   - **[Rails API基礎編](../01-rails-api-basics/README.md)** で環境設定を再確認
5. **状態管理ライブラリ**: Redux ToolkitやZustandの導入
   - **[useStateの内部動作](../02-react-basics/how-usestate-works.md)** で状態管理の基礎を復習

### 学習を続けるために

- **実際のプロジェクト**: 自分のアイデアでアプリを作ってみる
- **コミュニティ参加**: 勉強会やオンラインコミュニティに参加
- **継続的学習**: 新しい技術や手法を定期的に学ぶ

### 参考資料

- [Rails Testing Guide](https://guides.rubyonrails.org/testing.html)
- [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/)
- [TypeScript Deep Dive](https://basarat.gitbook.io/typescript/)

**🎉 お疲れさまでした！**

実践的なToDoアプリケーションの開発を通じて、Rails APIとReactの連携について深く学ぶことができました。ここで身につけたスキルを活かして、さらに高度なアプリケーション開発にチャレンジしてください！