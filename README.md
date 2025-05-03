

**Описание приложения:**

Приложение будет имитировать систему управления задачами (To-Do List).  В нем можно будет добавлять, просматривать, редактировать и удалять задачи.  Приложение будет построено с использованием принципов SOLID.

**Язык:** C# (как наиболее распространенный для .NET консольных приложений)

**Реализованные принципы SOLID:**

1.  **Single Responsibility Principle (SRP) - Принцип единственной ответственности:**

    *   **Почему выбран:**  Это фундаментальный принцип, который упрощает поддержку и изменение кода.  Классы с единственной ответственностью легче тестировать и понимать.
    *   **Реализация:**
        *   Класс `Task` будет отвечать только за хранение информации о задаче (название, описание, статус, дата выполнения).
        *   Класс `TaskManager` будет отвечать за управление списком задач (добавление, удаление, редактирование, просмотр).
        *   Класс `TaskDisplay` будет отвечать за отображение задач в консоли.
        *   Класс `TaskInput` будет отвечать за получение ввода от пользователя.
2.  **Open/Closed Principle (OCP) - Принцип открытости/закрытости:**

    *   **Почему выбран:** Этот принцип позволяет расширять функциональность приложения без изменения существующего кода.  Это важно для долгосрочной поддержки и развития.
    *   **Реализация:**
        *   Будет создан абстрактный класс `TaskFilter` с методом `IsSatisfiedBy(Task task)`.
        *   Реализованы конкретные классы-фильтры, наследующиеся от `TaskFilter` (например, `CompletedTaskFilter`, `OverdueTaskFilter`).
        *   `TaskManager` будет использовать `TaskFilter` для фильтрации задач, не требуя изменений при добавлении новых фильтров.
3.  **Liskov Substitution Principle (LSP) - Принцип подстановки Лисков:**

    *   **Почему выбран:**  Этот принцип гарантирует, что подклассы могут быть использованы вместо базовых классов без нарушения корректности работы программы.
    *   **Реализация:**
        *   Этот принцип тесно связан с OCP.  `TaskFilter` и его подклассы реализуют этот принцип.  Например, `CompletedTaskFilter` всегда должен возвращать корректный результат при использовании вместо `TaskFilter`.

**Не реализованные принципы SOLID:**

1.  **Interface Segregation Principle (ISP) - Принцип разделения интерфейса:**

    *   **Почему не выбран:**  В данной простой системе не требуется создавать множество мелких интерфейсов, чтобы классы реализовывали только необходимые методы.  Реализация этого принципа не даст значительного преимущества, а только усложнит код.
2.  **Dependency Inversion Principle (DIP) - Принцип инверсии зависимостей:**

    *   **Почему не выбран:**  В данной реализации для простоты используется прямое создание экземпляров классов.  DIP требует использования интерфейсов и Dependency Injection (DI) контейнеров, что усложнило бы код без существенной пользы для данного примера.  В реальных больших проектах DIP крайне важен для тестируемости и гибкости, но здесь это излишне.

** код (C#):**
```
using System;
using System.Collections.Generic;
using System.Linq;

// 1. Single Responsibility Principle (SRP)
// Класс, представляющий задачу
public class Task
{
    public string Title { get; set; }
    public string Description { get; set; }
    public bool IsCompleted { get; set; }
    public DateTime DueDate { get; set; }

    public Task(string title, string description, DateTime dueDate)
    {
        Title = title;
        Description = description;
        DueDate = dueDate;
        IsCompleted = false;
    }
}

// Класс для управления списком задач
public class TaskManager
{
    private List<Task> tasks = new List<Task>();

    public void AddTask(Task task)
    {
        tasks.Add(task);
    }

    public void RemoveTask(Task task)
    {
        tasks.Remove(task);
    }

    public void CompleteTask(Task task)
    {
        task.IsCompleted = true;
    }

    public List<Task> GetTasks(TaskFilter filter = null)
    {
        if (filter == null)
        {
            return tasks;
        }
        else
        {
            return tasks.Where(task => filter.IsSatisfiedBy(task)).ToList();
        }
    }
}

// Класс для отображения задач
public class TaskDisplay
{
    public void DisplayTasks(List<Task> tasks)
    {
        foreach (var task in tasks)
        {
            Console.WriteLine($"Title: {task.Title}");
            Console.WriteLine($"Description: {task.Description}");
            Console.WriteLine($"Due Date: {task.DueDate.ToShortDateString()}");
            Console.WriteLine($"Completed: {task.IsCompleted}");
            Console.WriteLine("---");
        }
    }
}

// Класс для получения ввода от пользователя
public class TaskInput
{
    public string GetInput(string prompt)
    {
        string input;
        do
        {
            Console.WriteLine(prompt);
            input = Console.ReadLine();
        } while (string.IsNullOrWhiteSpace(input));
        return input;
    }

    public DateTime GetDateInput(string prompt)
    {
        DateTime date;
        while (true)
        {
            string input = GetInput(prompt);
            if (DateTime.TryParse(input, out date))
            {
                return date;
            }
            Console.WriteLine("Invalid date format. Please use a valid date format (yyyy-MM-dd).");
        }
    }
}

// 2. Open/Closed Principle (OCP) and 3. Liskov Substitution Principle (LSP)
// Абстрактный класс для фильтров задач
public abstract class TaskFilter
{
    public abstract bool IsSatisfiedBy(Task task);
}

// Фильтр для завершенных задач
public class CompletedTaskFilter : TaskFilter
{
    public override bool IsSatisfiedBy(Task task)
    {
        return task.IsCompleted;
    }
}

// Фильтр для просроченных задач
public class OverdueTaskFilter : TaskFilter
{
    public override bool IsSatisfiedBy(Task task)
    {
        return task.DueDate < DateTime.Now && !task.IsCompleted;
    }
}

// Main class
public class Program
{
    public static void Main(string[] args)
    {
        TaskManager taskManager = new TaskManager();
        TaskDisplay taskDisplay = new TaskDisplay();
        TaskInput taskInput = new TaskInput();

        // Добавление задач
        string title = taskInput.GetInput("Enter task title:");
        string description = taskInput.GetInput("Enter task description:");
        DateTime dueDate = taskInput.GetDateInput("Enter due date (yyyy-MM-dd):");

        Task task1 = new Task(title, description, dueDate);
        taskManager.AddTask(task1);

        title = taskInput.GetInput("Enter another task title:");
        description = taskInput.GetInput("Enter another task description:");
        dueDate = taskInput.GetDateInput("Enter due date (yyyy-MM-dd):");

        Task task2 = new Task(title, description, dueDate.AddDays(-2));  // Task will be overdue
        taskManager.AddTask(task2);

        // Mark task1 as complete
        taskManager.CompleteTask(task1);

        // Отображение всех задач
        Console.WriteLine("\nAll Tasks:");
        taskDisplay.DisplayTasks(taskManager.GetTasks());

        // Отображение завершенных задач
        Console.WriteLine("\nCompleted Tasks:");
        taskDisplay.DisplayTasks(taskManager.GetTasks(new CompletedTaskFilter()));

        // Отображение просроченных задач
        Console.WriteLine("\nOverdue Tasks:");
        taskDisplay.DisplayTasks(taskManager.GetTasks(new OverdueTaskFilter()));

        Console.ReadKey();
    }
}

```

**Объяснение кода:**

*   **`Task`:**  Простой класс для хранения информации о задаче.  Отвечает только за данные.
*   **`TaskManager`:**  Управляет списком задач.  Имеет методы для добавления, удаления, завершения и получения задач.  Использует `TaskFilter` для фильтрации задач (реализация OCP и LSP).
*   **`TaskDisplay`:**  Отображает задачи в консоли.  Отделен от логики управления задачами.
*   **`TaskInput`:**  Получает ввод от пользователя.  Инкапсулирует логику ввода/вывода.
*   **`TaskFilter`, `CompletedTaskFilter`, `OverdueTaskFilter`:**  Реализуют OCP и LSP.  Позволяют добавлять новые фильтры задач без изменения `TaskManager`.
  
*   Single Responsibility Principle (SRP):
Каждый класс отвечает за одну конкретную задачу. Например, Task хранит данные о задаче, TaskManager управляет задачами, TaskDisplay отображает задачи, TaskInput получает ввод от пользователя.
Open/Closed Principle (OCP):
TaskManager использует абстрактный класс TaskFilter для фильтрации задач. Это позволяет добавлять новые фильтры без изменения существующего кода.
Liskov Substitution Principle (LSP):
Конкретные фильтры (CompletedTaskFilter, OverdueTaskFilter) наследуются от абстрактного класса TaskFilter и могут быть использованы вместо него без нарушения корректности работы программы.

Почему выбран SRP:
SRP упрощает поддержку и изменение кода. Каждый класс имеет одну ответственность, что делает его легче тестировать и понимать.
Почему выбран OCP:
OCP позволяет расширять функциональность приложения без изменения существующего кода. Это важно для долгосрочной поддержки и развития.
Почему выбран LSP:
LSP гарантирует, что подклассы могут быть использованы вместо базовых классов без нарушения корректности работы программы. Это важно для гибкости и расширяемости кода.
Почему не выбран ISP:
В данной простой системе не требуется создавать множество мелких интерфейсов. Реализация ISP не даст значительного преимущества, а только усложнит код.
Почему не выбран DIP:
В данной реализации для простоты используется прямое создание экземпляров классов. DIP требует использования интерфейсов и DI контейнеров, что усложнило бы код без существенной пользы для данного примера.

**Как запустить:**

1.  Сохраните код в файл с расширением `.cs` (например, `Program.cs`).
2.  Убедитесь, что у вас установлен .NET SDK.
3.  Откройте командную строку или терминал.
4.  Перейдите в каталог, где находится файл `Program.cs`.
5.  Выполните команду `dotnet run`.

