<h1>Avalonia application / Dapper</h1>

# Содержание:

- [1. Установка Avalonia](#1-установка-avalonia)
- [2. Установка Dapper](#2-dapper)
- [3. Интерфейс INotifyPropertyChanged](#3-интерфейс-inotifypropertychanged)
- [4. MessageBox для Avalonia](#4-messagebox-для-avalonia)
- [5. База List](#5-база-list)
- [6. Data Grid](#6-data-grid)
- [7. Открытие нового окна](#7-открытие-нового-окна)
    - [7.1 Запуск окна по центру экрана](#71-запуск-окна-по-центру-экрана)
- [8. Авторизация](#8-авторизация)
- [9. Регистрация](#9-регистрация)

<br><br>

# 1. Установка Avalonia

Прописать в командную строку (win + r => cmd)
- dotnet new install Avalonia.Templates

[↑ Содержание ↑](#содержание)

<br><br>

# 2. Dapper

Команды импорта:
- dotnet add package MySqlConnector
- dotnet add package Dapper

### Получение списка:
```c#
using (MySqlConnection database = new MySqlConnection(Globals.connectionString))
        {
            userList = database.Query<User>(
                "SELECT userid, login, password, userroleid " +
                "FROM user").ToList();
        }
```



Глобальный класс в папке <b>Classes</b>

```c#
public class Globals
{
    public static string connectionString = "Server=kolei.ru; User ID=свой; Password=свой; Database=свой";
}
```
[Доп инфа](https://github.com/kolei/PiRIS/blob/master/articles/sprint_lab.md)


[↑ Содержание ↑](#содержание)

<br><br>

# 3. Интерфейс INotifyPropertyChanged

```c#
public partial class MainWindow : Window, INotifyPropertyChanged
{
    public event PropertyChangedEventHandler? PropertyChanged;
    private void Invalidate()
    {
        if (PropertyChanged != null)
            PropertyChanged(this, new PropertyChangedEventArgs("свой список"));
    }
    ...
}
```

[↑ Содержание ↑](#содержание)

<br><br>

# 4. MessageBox для Avalonia

Команда импорта:
- dotnet add package MessageBox.Avalonia

```c#
var box = MessageBoxManager
        .GetMessageBoxStandard("Caption", "Are you sure you would like to delete appender_replace_page_1?",
            ButtonEnum.YesNo);

    var result = await box.ShowAsync();
```

[Инструкция как пользоваться](https://github.com/AvaloniaCommunity/MessageBox.Avalonia)

[↑ Содержание ↑](#содержание)

<br><br>

# 5. База List

```c#
private List<User> _userList = null;
public List<User> userList
{
    get
    {
        var res = _userList;
        ...
        return res;
    }
    set
    {
        _userList = value;
        Invalidate();
    }
}
```
[↑ Содержание ↑](#содержание)

<br><br>

# <b>6. Data Grid</b>

Команда импорта:
- dotnet add package Avalonia.Controls.DataGrid



Include Data Grid Styles

You must reference the data grid themes to include the additional styles that the data grid uses. You can do this by adding a <StyleInclude> element to the application (App.axaml file).

```c#
<Application.Styles>
    <FluentTheme />
    <StyleInclude Source="avares://Avalonia.Controls.DataGrid/Themes/Fluent.xaml"/>
</Application.Styles>
```

Пример с автогенерацией:
```c#
<DataGrid
        ItemsSource="{Binding #root.orderList}"
        AutoGenerateColumns="True"
        BorderThickness="1" BorderBrush="Black"
        GridLinesVisibility="All"
        IsReadOnly="True"
        >
        
    </DataGrid>
```

Пример с ручной разметкой:
```c#
<DataGrid Margin="20" ItemsSource="{Binding People}"
          //CanUserReorderColumns="True"
          //CanUserResizeColumns="True"
          //CanUserSortColumns="False"
          BorderThickness="1" BorderBrush="Gray"
          GridLinesVisibility="All"
          IsReadOnly="True"
          >
  <DataGrid.Columns>
     <DataGridTextColumn Header="First Name"  Binding="{Binding FirstName}"/>
     <DataGridTextColumn Header="Last Name" Binding="{Binding LastName}" />
  </DataGrid.Columns>
</DataGrid>
```

[↑ Содержание ↑](#содержание)

<br><br>

# 7. Открытие нового окна

```c#
    var window = new OrganizerWindow();
    window.Show();
    // закрытие текущего
    Close();
```

[↑ Содержание ↑](#содержание)

<br><br>

# 7.1 Запуск окна по центру экрана
```c#
// окно по центру
private void OnLoaded(object? sender, EventArgs e)
{
    // центрируем окно
    var screen = Screens.All[0];
    var screenWidth = screen.WorkingArea.Width;
    var screenHeight = screen.WorkingArea.Height;
    var windowWiWidth = Width;
    var windowHeight = Height;

    // устанавливаем положение окна
    Position = new PixelPoint((int)((screenWidth - windowWiWidth) / 2), (int)((screenHeight - windowHeight) / 2));
}
```

```c#
public MainWindow()
{
    InitializeComponent();
    Loaded += OnLoaded;
}
```

[↑ Содержание ↑](#содержание)

<br><br>

# <b>8. Авторизация

## Более старая версия
```c#
// кнопка авторизации
private User currentUser = null;
private string res = "";
private async void Authorization_Button_OnClick(object? sender, RoutedEventArgs e)
{
    foreach (var user in userList)
    {
        if (user.login == LoginTextBox.Text)
        {
            if (user.password == PasswordTextBox.Text)
            {
                currentUser = user;
                res = currentUser.userroleid.ToString();
                break;
            }
        }
    }
    if (res == "1")
    {
        var window = new HeadsOfTheDepartmentWindow();
        window.Show();
        Close();
    }
    else if(res == "2")
    {
        var window = new OrganizerWindow();
        window.Show();
        Close();
    }
    else if (res == "3")
    {
        var window = new TechnicianWindow();
        window.Show();
        Close();
    }
    else
    {
        var box = MessageBoxManager.GetMessageBoxStandard(
            "Ошибка", "Такого пользователя не существует",
            ButtonEnum.Ok);
        var result = await box.ShowAsync();
    }
    
}
```

## Версия для демоэкзамена
```c#
private User currentUser;
private int LocalBlockCount;
// кнопка авторизации
private async void AuthorizationButton_OnClick(object? sender, RoutedEventArgs e)
{
    try
    {
        // проверка на пустые поля
        if (LoginTextBox.Text.Length == 0 || PasswordTextBox.Text.Length == 0)
        {
            var box = MessageBoxManager.GetMessageBoxStandard("Ошибка",
                "Поля Логин и Пароль обязательны для заполнения", ButtonEnum.Ok, MsBox.Avalonia.Enums.Icon.Error);
            await box.ShowAsync();
        }
        else
        {
            // цикл проверки каждого пользователя из списка
            foreach (var user in userList)
            {
                if (user.Login == LoginTextBox.Text)
                {
                    if (user.Password == PasswordTextBox.Text)
                    {
                        // присваивание по логину и паролю текущего пользователя
                        currentUser = user;
                        // проверка статуса текущего пользователя
                        if (currentUser.Active == 1)
                        {
                            // сброс количества блокировок у текущего пользователя
                            using (MySqlConnection database = new MySqlConnection(Globals.connectionString))
                            {
                                database.Execute("UPDATE User SET BlockCount = 0 WHERE Id = @Id", currentUser);
                            }
                            if (currentUser.Role_id == 1)
                            {
                                var box = MessageBoxManager.GetMessageBoxStandard("Уведомление",
                                    $"Добро пожаловать. Вы успешно авторизовались как пользователь {currentUser.LastName}",
                                    ButtonEnum.Ok, MsBox.Avalonia.Enums.Icon.Info);
                                await box.ShowAsync();
                                var window = new UserWindow();
                                window.Show();
                                Close();
                            }
                            else if (currentUser.Role_id == 2)
                            {
                                var box = MessageBoxManager.GetMessageBoxStandard("Уведомление",
                                    $"Добро пожаловать. Вы успешно авторизовались как администратор {currentUser.LastName}",
                                    ButtonEnum.Ok, MsBox.Avalonia.Enums.Icon.Info);
                                await box.ShowAsync();
                                var window = new AdminWindow();
                                window.Show();
                                Close();
                            }
                        }
                        else
                        {
                            // уведомление при заблокированном статусе
                            var box = MessageBoxManager.GetMessageBoxStandard("Уведомление",
                                "Вы заблокированы. Обратитесь к администратору системы", ButtonEnum.Ok,
                                MsBox.Avalonia.Enums.Icon.Warning);
                            await box.ShowAsync();
                        }
                        break;
                    }
                    else
                    {
                        // присваивание по логину текущего пользователя
                        currentUser = user;
                        // увеличение счетчика попыток у текущего пользователя
                        LocalBlockCount = currentUser.BlockCount;
                        LocalBlockCount++;
                        currentUser.BlockCount = LocalBlockCount;
                        // обновление в бд количества попыток
                        using (MySqlConnection database = new MySqlConnection(Globals.connectionString))
                        {
                            database.Execute("UPDATE User SET BlockCount = @BlockCount WHERE Id = @Id",
                                currentUser);
                        }
                        // блокировка пользователя при 3х неверных попытках
                        if (currentUser.BlockCount > 2)
                        {
                            currentUser.Active = 0;
                            using (MySqlConnection database = new MySqlConnection(Globals.connectionString))
                            {
                                database.Execute("UPDATE User SET Active = 0 WHERE Id = @Id", currentUser);
                            }
                        }
                        var box = MessageBoxManager.GetMessageBoxStandard("Вы ввели неверный логин или пароль",
                            $@"Пожалуйста проверьте ещё раз введенные данные. Неверных попыток: {currentUser.BlockCount}", ButtonEnum.Ok,
                            MsBox.Avalonia.Enums.Icon.Error);
                        await box.ShowAsync();
                        break;
                    }
                }
            }
        }
    }
    catch (Exception exception)
    {
        var box = MessageBoxManager.GetMessageBoxStandard("Ошибка системы", $"{exception}", ButtonEnum.Ok,
            MsBox.Avalonia.Enums.Icon.Error);
        await box.ShowAsync();
    }
}
```

[↑ Содержание ↑](#содержание)

<br><br>

# 9. Регистрация
```c#
private User currentUser;
    // кнопка регистраци
    private async void RegistrationButton_OnClick(object? sender, RoutedEventArgs e)
    {
        try
        {
            if (RegLoginTextBox.Text.Length == 0 || RegPasswordTextBox.Text.Length == 0 || RegFirstNameTextBox.Text.Length == 0 || RegPhoneTextBox.Text.Length == 0)
            {
                var box = MessageBoxManager.GetMessageBoxStandard("Ошибка",
                    "Поля обязательны для заполнения", ButtonEnum.Ok, MsBox.Avalonia.Enums.Icon.Error);
                await box.ShowAsync();
            }
            else
            {
                currentUser = new User();
                currentUser.Login = RegLoginTextBox.Text;
                currentUser.Password = RegPasswordTextBox.Text;
                currentUser.Role_Id = 1;
                currentUser.FirstName = RegFirstNameTextBox.Text;
                currentUser.Phone = RegPhoneTextBox.Text;
                using (MySqlConnection database = new MySqlConnection(Globals.connectionString))
                {
                    database.Execute(
                        "INSERT INTO User (Login, Password, Role_Id, FirstName, Phone) VALUES (@Login, @Password, @Role_Id, @FirstName, @Phone)",
                        currentUser);
                }
                var box = MessageBoxManager.GetMessageBoxStandard("Уведомление",
                    $"Добро пожаловать! Вы успешно зарегистрировались как {currentUser.FirstName}.", ButtonEnum.Ok, MsBox.Avalonia.Enums.Icon.Info);
                await box.ShowAsync();
                Close();
            }
        }
        catch (Exception exception)
        {
            var box = MessageBoxManager.GetMessageBoxStandard("Ошибка",
                $"{exception}", ButtonEnum.Ok, MsBox.Avalonia.Enums.Icon.Error);
            await box.ShowAsync();
        }
    }
```
[↑ Содержание ↑](#содержание)

<br><br>

# 10.

