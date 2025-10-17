# GitHub MCP Workflow - Полная инструкция

## 🔑 Аутентификация и настройка

### 1. Personal Access Token (PAT)
- Создать токен на GitHub: Settings → Developer settings → Personal access tokens → Tokens (classic)
- **Обязательные права:**
  - `repo` - полный доступ к репозиториям
  - `workflow` - управление GitHub Actions
  - `write:packages` - загрузка пакетов
  - `delete:packages` - удаление пакетов
  - `admin:org` - администрирование организаций
  - `gist` - управление gist'ами
  - `notifications` - управление уведомлениями
  - `user` - доступ к профилю пользователя
  - `delete_repo` - удаление репозиториев

### 2. Настройка переменной окружения
```bash
export GITHUB_TOKEN="your_personal_access_token_here"
```

## 🛠️ Основные MCP GitHub инструменты

### Информация о пользователе
```javascript
mcp_github_get_me()
// Получить информацию о текущем пользователе GitHub
```

### Управление репозиториями
```javascript
// Создание репозитория
mcp_github_create_repository({
  name: "repository-name",
  description: "Описание репозитория",
  private: false,
  autoInit: false
})

// Получение содержимого файла
mcp_github_get_file_contents({
  owner: "username",
  repo: "repository-name",
  path: "path/to/file"
})
```

### Работа с файлами
```javascript
// Создание/обновление одного файла
mcp_github_create_or_update_file({
  owner: "username",
  repo: "repository-name",
  path: "path/to/file",
  content: "содержимое файла",
  message: "commit message",
  branch: "main"
})

// Загрузка нескольких файлов
mcp_github_push_files({
  owner: "username",
  repo: "repository-name",
  branch: "main",
  files: [
    {path: "file1.txt", content: "content1"},
    {path: "file2.txt", content: "content2"}
  ],
  message: "Add multiple files"
})
```

### Управление Issues и Pull Requests
```javascript
// Создание issue
mcp_github_create_issue({
  owner: "username",
  repo: "repository-name",
  title: "Issue title",
  body: "Issue description"
})

// Создание Pull Request
mcp_github_create_pull_request({
  owner: "username",
  repo: "repository-name",
  title: "PR title",
  head: "feature-branch",
  base: "main",
  body: "PR description"
})
```

## 📋 Стандартный workflow

### 1. Инициализация проекта
```javascript
// Шаг 1: Получить информацию о пользователе
const user = await mcp_github_get_me()

// Шаг 2: Создать репозиторий
const repo = await mcp_github_create_repository({
  name: "project-name",
  description: "Project description",
  private: false
})
```

### 2. Загрузка файлов
```javascript
// Для пустого репозитория - сначала создать README
await mcp_github_create_or_update_file({
  owner: user.login,
  repo: "project-name",
  path: "README.md",
  content: "# Project Name\n\nDescription...",
  message: "Initial commit: Add README",
  branch: "main"
})

// Затем загрузить остальные файлы
await mcp_github_push_files({
  owner: user.login,
  repo: "project-name",
  branch: "main",
  files: [
    {path: "src/main.cpp", content: "// main.cpp content"},
    {path: "CMakeLists.txt", content: "cmake_minimum_required..."}
  ],
  message: "Add source files and build configuration"
})
```

### 3. Обновление существующих файлов
```javascript
// Получить SHA существующего файла
const fileInfo = await mcp_github_get_file_contents({
  owner: "username",
  repo: "repository-name",
  path: "existing-file.txt"
})

// Обновить файл с указанием SHA
await mcp_github_create_or_update_file({
  owner: "username",
  repo: "repository-name",
  path: "existing-file.txt",
  content: "updated content",
  message: "Update existing file",
  branch: "main",
  sha: fileInfo.sha  // Важно для обновления!
})
```

## ⚠️ Важные моменты

### 1. Работа с пустыми репозиториями
- **Проблема:** `push_files` не работает с пустыми репозиториями
- **Решение:** Сначала создать хотя бы один файл через `create_or_update_file`

### 2. Обновление существующих файлов
- **Обязательно** указывать параметр `sha` при обновлении
- Получить SHA через `get_file_contents`

### 3. Обработка ошибок
```javascript
try {
  await mcp_github_create_or_update_file({...})
} catch (error) {
  if (error.message.includes("Git Repository is empty")) {
    // Сначала создать файл через create_or_update_file
  }
}
```

### 4. Размер файлов
- MCP GitHub имеет ограничения на размер файлов
- Для больших файлов использовать Git LFS или разбивать на части

## 🎯 Практические примеры

### Создание проекта с нуля
```javascript
// 1. Получить информацию о пользователе
const user = await mcp_github_get_me()
console.log(`Working with user: ${user.login}`)

// 2. Создать репозиторий
const repo = await mcp_github_create_repository({
  name: "my-new-project",
  description: "My awesome project",
  private: false
})

// 3. Создать README
await mcp_github_create_or_update_file({
  owner: user.login,
  repo: "my-new-project",
  path: "README.md",
  content: `# My New Project\n\nThis is my awesome project!`,
  message: "Initial commit: Add README",
  branch: "main"
})

// 4. Загрузить исходный код
await mcp_github_push_files({
  owner: user.login,
  repo: "my-new-project",
  branch: "main",
  files: [
    {path: "src/main.cpp", content: "#include <iostream>\n\nint main() {\n    std::cout << \"Hello World!\" << std::endl;\n    return 0;\n}"},
    {path: "CMakeLists.txt", content: "cmake_minimum_required(VERSION 3.10)\nproject(MyProject)\nadd_executable(main src/main.cpp)"}
  ],
  message: "Add source code and build files"
})
```

### Обновление документации
```javascript
// Получить текущий README
const readme = await mcp_github_get_file_contents({
  owner: "username",
  repo: "repository-name",
  path: "README.md"
})

// Обновить README
const updatedContent = readme.content + "\n\n## New Section\n\nAdded new features!"

await mcp_github_create_or_update_file({
  owner: "username",
  repo: "repository-name",
  path: "README.md",
  content: updatedContent,
  message: "Update README with new features",
  branch: "main",
  sha: readme.sha
})
```

## 🔧 Troubleshooting

### Частые ошибки и решения

1. **"Git Repository is empty"**
   - Решение: Сначала создать файл через `create_or_update_file`

2. **"Resource not accessible by integration"**
   - Решение: Проверить права токена, добавить недостающие права

3. **"Not Found"**
   - Решение: Проверить правильность owner/repo параметров

4. **"Validation Failed"**
   - Решение: Проверить формат данных, обязательные поля

## 📚 Дополнительные ресурсы

- [GitHub API Documentation](https://docs.github.com/en/rest)
- [Personal Access Tokens](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)
- [MCP GitHub Tools](https://github.com/modelcontextprotocol/servers/tree/main/src/github)

---

**Создано:** 2025-10-17  
**Автор:** AI Assistant  
**Версия:** 1.0  
**Репозиторий:** [CursorExpert](https://github.com/AlexLan73/CursorExpert)