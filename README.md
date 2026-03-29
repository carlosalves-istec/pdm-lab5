# pdm-lab5

---
# EX1 — Lista de Tarefas _(Guiado)_

Neste exercício vais construir uma app de gestão de tarefas. O utilizador pode adicionar tarefas, marcá-las como concluídas, deslizar para removê-las e filtrar por estado.

---
## Passo 1 — Criar o projeto

No terminal, cria o projeto e abre-o no VSCode:

```bash
flutter create lab_ex1
cd lab_ex1
```

> Alternativamente, com o VSCode aberto, podes criar o projeto ao clicar simultaneamente nas teclas `CTRL` + `SHIFT` + `P` e selecionar a opção: `Flutter: New Project` e selecionar o template `Empty Application`. 

Cria a estrutura de ficheiros:

```
lib/
├── main.dart
├── models/
│   └── todo.dart
└── todo_screen.dart
```

---

## Passo 2 — Criar o modelo de dados

Abre `lib/models/todo.dart` e adiciona o seguinte código. O modelo `Todo` é uma classe Dart simples e não é um widget. Apenas servirá para guardar os dados de uma tarefa.

```dart
class Todo {
  final int id;
  final String title;
  bool completed;

  Todo({
    required this.id,
    required this.title,
    this.completed = false,
  });
}

// Lista de tarefas de exemplo
List<Todo> exemplo() => [
      Todo(id: 1, title: 'Estudar Flutter', completed: true),
      Todo(id: 2, title: 'Fazer o lab3'),
      Todo(id: 3, title: 'Fazer o lab4'),
      Todo(id: 4, title: 'Testar o emulador'),
    ];
```

> **Nota:** `completed` não é do tipo `final` uma vez que é o único campo que pode mudar depois de criado (quando o utilizador marca a tarefa).

---

## Passo 3 — Configurar o `main.dart`

Substitui todo o conteúdo de `lib/main.dart` pelo seguinte. 
`TodoApp` é um **`StatelessWidget`** - não guarda o estado, apenas configura a app e define o tema:

```dart
import 'package:flutter/material.dart';
import 'todo_screen.dart';

void main() => runApp(const TodoApp()); // ponto de entrada da app

class TodoApp extends StatelessWidget {
  const TodoApp({super.key});

  @override
  Widget build(BuildContext context) { // build descreve o que este widget apresenta no ecrã
    return MaterialApp(
      title: 'Lista de Tarefas',
      theme: ThemeData(
        useMaterial3: true,
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.teal),
      ),
      home: const TodoScreen(), // página inicial da aplicação
    );
  }
}
```

> **`StatelessWidget`:** utilizado para widgets que não precisam de guardar um estado. `TodoApp` apenas descreve a configuração da app, a qual  nunca muda depois de ser criada. Repara que não tem `setState`, nem variáveis que mudem.


---

## Passo 4 — Criar o `TodoScreen` como `StatefulWidget`

Abre `lib/todo_screen.dart`:

```dart
import 'package:flutter/material.dart';
import 'models/todo.dart';

// Enumeração dos três filtros para as tarefas (todas, ativas e concluídas)
enum TodoFilter { all, active, completed }

// TodoScreen é StatefulWidget porque a lista e os filtros podem mudar durante a execução
class TodoScreen extends StatefulWidget {
  const TodoScreen({super.key});

  @override
  State<TodoScreen> createState() => _TodoScreenState(); // cria o objeto de estado associado a este widget
}

// _TodoScreenState guarda e gere todo o estado deste ecrã
class _TodoScreenState extends State<TodoScreen> {
  final List<Todo> _todos = exemplo(); // lista de tarefas; começa com os dados de exemplo
  TodoFilter _filter = TodoFilter.all;   // filtro default = "Todas"
```

> **Relativamente ao estado:**  `_TodoScreenState` guarda o estado mutável: a lista `_todos` e o filtro `_filter`. Sempre que o estado muda com `setState`, o Flutter chama novamente `build` e atualiza a interface.

---

## Passo 5 — Adicionar o getter de filtragem

Ainda dentro de `_TodoScreenState`:

```dart
  // Getter que devolve a lista de tarefas conforme o filtro selecionado
  List<Todo> get _filteredTodos {
    switch (_filter) {
      case TodoFilter.all:       // devolve a lista completa sem filtrar
        return _todos;
      case TodoFilter.active:    // devolve as tarefas que ainda não foram concluídas
        return [for (final t in _todos) if (!t.completed) t];
      case TodoFilter.completed: // só devolve as tarefas que foram concluídas
        return [for (final t in _todos) if (t.completed) t];
    }
  }
```

---

## Passo 6 — Componente para adicionar uma tarefa

```dart
  // Controlador do campo de texto
  final _ctrl = TextEditingController();

  // Adiciona uma nova tarefa à lista se o campo não estiver vazio
  void _addTodo() {
    if (_ctrl.text.trim().isEmpty) return; // ignora se o utilizador não escreveu nada
    
    setState(() => _todos.add(Todo(
          id: (_todos.isEmpty ? 0 : _todos.last.id) + 1, // id = último id da lista + 1
          title: _ctrl.text.trim(),                  // título da tarefa introduzido pelo utilizador
        )));
    _ctrl.clear(); // limpa o campo após adicionar a tarefa
  }
```

---

## Passo 7 — Métodos para completar e eliminar tarefas

```dart
  // Alterna o estado da tarefa entre completada e por completar
  void _toggleCompleted(Todo todo) =>
      setState(() => todo.completed = !todo.completed);

  // Elimina a tarefa da lista
  void _deleteTodo(Todo todo) =>
      setState(() => _todos.remove(todo));
```

> **Nota:** qualquer alteração a variáveis de estado tem de ser feita dentro de `setState`. Caso contrário o widget não será atualizado.

---

## Passo 8 — `build` do `TodoScreen`

```dart
  @override
  Widget build(BuildContext context) { // chamado pelo Flutter sempre que o estado muda
    return Scaffold( // estrutura base do ecrã
      appBar: AppBar(
        title: const Text('Lista de Tarefas'), // título da barra superior
        centerTitle: true,
        actions: [
          IconButton(
            icon: const Icon(Icons.delete_sweep), // ícone para eliminar tarefas concluídas
            tooltip: 'Eliminar tarefas concluídas', // este texto aparece ao pressionar o ícone
            onPressed: () =>
                setState(() => _todos.removeWhere((t) => t.completed)), // remove todas as tarefas concluídas
          ),
        ],
      ),
      body: Column( // organiza os filhos verticalmente
        children: [

          // Barra de filtros
          Padding(
            padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
            child: SegmentedButton<TodoFilter>( // botão para escolher entre filtros (cada filtro é um segmento)
              segments: const [
                ButtonSegment(
                    value: TodoFilter.all,
                    label: Text('Todas'),
                    icon: Icon(Icons.list)),
                ButtonSegment(
                    value: TodoFilter.active,
                    label: Text('Ativas'),
                    icon: Icon(Icons.radio_button_unchecked)),
                ButtonSegment(
                    value: TodoFilter.completed,
                    label: Text('Concluídas'),
                    icon: Icon(Icons.check_circle_outline)),
              ],
              selected: {_filter}, // segmento ativo
              onSelectionChanged: (s) => setState(() => _filter = s.first), // atualiza o filtro
            ),
          ),

          // Contador de tarefas pendentes
          Padding(
            padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 4),
            child: Align(
              child: Text(
                '${_todos.where((t) => !t.completed).length} tarefa(s) por concluir',
                style: const TextStyle(fontSize: 13, color: Colors.grey),
              ),
            ),
          ),

          // Lista de tarefas
          Expanded( // ocupa todo o espaço vertical disponível
            child: _filteredTodos.isEmpty
            // condição ternária: se ? (condição verdadeira) : (senão)
                ? const Center(
                    child: Column(
                      mainAxisSize: MainAxisSize.min,
                      children: [
                        Icon(Icons.check_circle_outline, size: 64, color: Colors.grey),
                        SizedBox(height: 12),
                        Text('Sem tarefas a apresentar', style: TextStyle(color: Colors.grey)),
                      ],
                    ),
                  )
                : ListView.builder(
                    itemCount: _filteredTodos.length,
                    itemBuilder: (context, index) {
                      final todo = _filteredTodos[index];
                      return Dismissible(              // permite remover ao deslizar para o lado
                        key: ValueKey(todo.id),        // chave única obrigatória
                        direction: DismissDirection.startToEnd, // desliza da esquerda para a direita
                        onDismissed: (_) => _deleteTodo(todo),
                        background: Container(         // fundo vermelho visível ao deslizar
                          color: Colors.red,
                          alignment: Alignment.centerLeft,
                          padding: const EdgeInsets.only(left: 20),
                          child: const Icon(Icons.delete, color: Colors.white),
                        ),
                        child: _TodoCard(
                          todo: todo,
                          onToggle: () => _toggleCompleted(todo),
                        ),
                      );
                    },
                  ),
          ),

          // Campo para adicionar nova tarefa
          Padding(
            padding: const EdgeInsets.all(12),
            child: Row(
              children: [
                Expanded(
                  child: TextField(
                    controller: _ctrl,
                    decoration: const InputDecoration(hintText: 'Nova tarefa...'),
                    onSubmitted: (_) => _addTodo(), // adiciona a tarefa à lista
                  ),
                ),
                const SizedBox(width: 8),
                IconButton.filled(
                  icon: const Icon(Icons.add),
                  onPressed: _addTodo,
                ),
              ],
            ),
          ),

        ],
      ),
    );
  }
}
```

> **`Expanded`:** numa `Column`, o `Expanded` diz ao widget filho para ocupar todo o espaço vertical restante depois dos outros filhos.

> `Dismissible`:  envolve qualquer widget e permite removê-lo da lista arrastando horizontalmente. O `key` é obrigatório e deve ser único por item, daí usar o id.

> `ListView.builder`: constrói os itens apenas quando ficam visíveis no ecrã, sendo mais eficiente do que uma lista estática de widgets.

---

## Passo 9 — Criar o widget `_TodoCard` (`StatelessWidget`)

Adiciona este widget fora de `_TodoScreenState`, no mesmo ficheiro:

```dart
// _TodoCard é um StatelessWidget que apresenta os dados de uma tarefa
class _TodoCard extends StatelessWidget {
  final Todo todo; // tarefa a apresentar — recebida por parâmetro
  final VoidCallback onToggle; // função a ser chamada quando o utilizador clica na checkbox

  const _TodoCard({required this.todo, required this.onToggle}); // construtor com parâmetros posicionais obrigatórios

  @override
  Widget build(BuildContext context) {
    return Card(
      margin: const EdgeInsets.symmetric(horizontal: 12, vertical: 4), // margem à volta do card
      child: CheckboxListTile( // ListTile com checkbox integrada
        value: todo.completed, // estado atual da checkbox
        onChanged: (_) => onToggle(),
        title: Text(
          todo.title,
          style: TextStyle(
            decoration: todo.completed ? TextDecoration.lineThrough : null, // risca o texto se a tarefa estiver concluída
            color: todo.completed ? Colors.grey : null,// cor da tarefa aparece a cinza se concluída,caso contrário usa a cor normal
          ),
        ),
      ),
    );
  }
}
```

>  `_TodoCard`:  A maneira de apresentar uma tarefa é sempre a mesma. Por isso agregamos os componentes envolvidos num só widget.
> Sempre que o mesmo padrão visual se repete devemos extrair partes para widgets próprios.

> O `StatelessWidget` `_TodoCard` não sabe como atualizar o estado, apenas recebe `onToggle` e chama-o. Quem atualiza o estado é sempre o `_TodoScreenState` via `setState`. Este padrão:  filho notifica o pai, é fundamental em Flutter.

---

# EX2 — Lista de Compras _(Autónomo)_

Neste exercício vais criar uma **Lista de Compras** aplicando os mesmos conceitos do EX1.

A diferença em relação ao EX1 é que a lista de compras tem apenas **dois estados** (por comprar e no carrinho) em vez de três.

---

## Requisitos

A app deve incluir:

- Um **modelo de dados** `ShoppingItem` com os campos `id`, `name` e `inCart` (equivalente ao `completed` do EX1)
- Uma função `exemplo()` com pelo menos itens de demonstração (alguns já no carrinho)

- Um **`StatelessWidget`** `ShoppingApp` para configurar a app (tema à escolha)

- Um **`StatefulWidget`** `ShoppingScreen` que gere o estado da lista e deve ter:
	- `_items` — lista de itens inicializada com `exemplo()`
	- `_filter` — filtro ativo (enum com três valores: todos / por comprar / no carrinho)
	- `_ctrl` — `TextEditingController` para o campo de adição
	- Getter `_filteredItems` — devolve a lista conforme `_filter`
	- Método `_addItem` — para adicionar um novo item à lista
	- Método `_toggleInCart` — para alterar o estado entre por comprar e no carrinho/comprado.
	- Método `_deleteItem` — remove o item da lista.
	
- **`build`** — `Scaffold` com `AppBar` e `body`
	- `AppBar` com o título da aplicação  e um botão para remover todos os itens do carrinho
	- `body`: barra de filtros, lista de itens e input para adicionar itens à lista
	- A lista deve usar `ListView.builder` com `Dismissible` de maneira a deslizar para eliminar itens

**`_ShoppingCard`** (`StatelessWidget`) — card de cada item com `CheckboxListTile` semelhante ao EX1.

---
