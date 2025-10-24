# C# + XAML + VS Code - คู่มือโครงสร้างโปรเจ็คอย่างละเอียด

ฉันจะอธิบายโครงสร้างโปรเจ็คที่สมบูรณ์พร้อมตำแหน่งไฟล์ รายละเอียด และตัวอย่างโปรเจ็คซับซ้อน

## 📁 โครงสร้างโปรเจ็คพื้นฐาน

```
MyWpfApp/
├── .vscode/
│   ├── launch.json          # การตั้งค่า Debug
│   ├── settings.json        # การตั้งค่า VS Code
│   └── extensions.json      # Extensions ที่แนะนำ
├── .github/
│   └── workflows/          # CI/CD Pipelines
├── src/
│   ├── App.xaml            # Application Root
│   ├── App.xaml.cs         # Application Code-behind
│   ├── MainWindow.xaml     # Main UI
│   ├── MainWindow.xaml.cs  # Main UI Logic
│   ├── ViewModels/         # MVVM ViewModels
│   ├── Models/             # Data Models
│   ├── Views/              # User Controls
│   ├── Services/           # Business Logic
│   ├── Resources/          # Styles, Templates
│   └── Utils/              # Helper Classes
├── Tests/
│   ├── UnitTests/
│   └── UITests/
├── Docs/
│   └── Architecture.md
├── MyWpfApp.csproj         # Project File
├── .gitignore
├── README.md
└── LICENSE
```

---

## 📋 รายละเอียดโฟลเดอร์แต่ละอัน

### 1. `.vscode/` - การตั้งค่า VS Code

```json name=.vscode/settings.json
{
  "[csharp]": {
    "editor.defaultFormatter": "ms-dotnettools.csharp",
    "editor.formatOnSave": true,
    "editor.codeActionsOnSave": {
      "source.fixAll.eslint": true
    }
  },
  "[xaml]": {
    "editor.defaultFormatter": "ms-dotnettools.csharp"
  },
  "omnisharp.enableEditorConfigSupport": true,
  "omnisharp.enableRoslynAnalyzers": true,
  "files.exclude": {
    "**/bin": true,
    "**/obj": true
  }
}
```

```json name=.vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": ".NET Core Launch (console)",
      "type": "coreclr",
      "request": "launch",
      "preLaunchTask": "build",
      "program": "${workspaceFolder}/bin/Debug/net6.0/MyWpfApp.dll",
      "args": [],
      "cwd": "${workspaceFolder}",
      "stopAtEntry": false,
      "console": "internalConsole"
    }
  ]
}
```

```json name=.vscode/extensions.json
{
  "recommendations": [
    "ms-dotnettools.csharp",
    "ms-dotnettools.vscode-dotnet-runtime",
    "jchannon.csharpextensions",
    "kreativ-software.csharp-to-ts",
    "ms-vscode.makefile-tools"
  ]
}
```

### 2. `src/` - Source Code

#### **App.xaml**
```xml name=src/App.xaml
<Application 
    x:Class="MyWpfApp.App"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="clr-namespace:MyWpfApp">
    
    <Application.Resources>
        <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <ResourceDictionary Source="Resources/Styles.xaml"/>
                <ResourceDictionary Source="Resources/Colors.xaml"/>
            </ResourceDictionary.MergedDictionaries>
        </ResourceDictionary>
    </Application.Resources>
</Application>
```

#### **App.xaml.cs**
```csharp name=src/App.xaml.cs
using System.Windows;

namespace MyWpfApp
{
    public partial class App : Application
    {
        protected override void OnStartup(StartupEventArgs e)
        {
            base.OnStartup(e);
            
            // Initialize Services
            ServiceLocator.Initialize();
            
            MainWindow = new MainWindow();
            MainWindow.Show();
        }
    }
}
```

#### **MainWindow.xaml**
```xml name=src/MainWindow.xaml
<Window 
    x:Class="MyWpfApp.MainWindow"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="clr-namespace:MyWpfApp"
    Title="My WPF Application"
    Height="600"
    Width="800"
    WindowStartupLocation="CenterScreen">
    
    <Grid>
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="250"/>
            <ColumnDefinition Width="*"/>
        </Grid.ColumnDefinitions>
        
        <!-- Left Sidebar -->
        <StackPanel Grid.Column="0" Background="#F0F0F0" Padding="10">
            <TextBlock Text="Navigation" FontSize="16" FontWeight="Bold"/>
            <Button Content="Home" Margin="0,10,0,0" Height="40"/>
            <Button Content="Settings" Margin="0,10,0,0" Height="40"/>
            <Button Content="About" Margin="0,10,0,0" Height="40"/>
        </StackPanel>
        
        <!-- Main Content -->
        <ContentControl Grid.Column="1" Content="{Binding CurrentView}" Margin="10"/>
    </Grid>
</Window>
```

#### **MainWindow.xaml.cs**
```csharp name=src/MainWindow.xaml.cs
using System.Windows;
using MyWpfApp.ViewModels;

namespace MyWpfApp
{
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent();
            DataContext = new MainViewModel();
        }
    }
}
```

### 3. `ViewModels/` - MVVM Pattern

```csharp name=src/ViewModels/BaseViewModel.cs
using System.ComponentModel;
using System.Runtime.CompilerServices;

namespace MyWpfApp.ViewModels
{
    public abstract class BaseViewModel : INotifyPropertyChanged
    {
        public event PropertyChangedEventHandler? PropertyChanged;

        protected void OnPropertyChanged([CallerMemberName] string propertyName = "")
        {
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
        }

        protected void SetProperty<T>(ref T field, T value, [CallerMemberName] string propertyName = "")
        {
            if (!Equals(field, value))
            {
                field = value;
                OnPropertyChanged(propertyName);
            }
        }
    }
}
```

```csharp name=src/ViewModels/MainViewModel.cs
using System.Collections.ObjectModel;
using MyWpfApp.Models;
using MyWpfApp.Services;

namespace MyWpfApp.ViewModels
{
    public class MainViewModel : BaseViewModel
    {
        private readonly IUserService _userService;
        private ObservableCollection<User> _users;
        private User _selectedUser;

        public ObservableCollection<User> Users
        {
            get => _users;
            set => SetProperty(ref _users, value);
        }

        public User SelectedUser
        {
            get => _selectedUser;
            set => SetProperty(ref _selectedUser, value);
        }

        public MainViewModel()
        {
            _userService = ServiceLocator.GetService<IUserService>();
            LoadUsers();
        }

        private async void LoadUsers()
        {
            var users = await _userService.GetAllUsersAsync();
            Users = new ObservableCollection<User>(users);
        }
    }
}
```

### 4. `Models/` - Data Models

```csharp name=src/Models/User.cs
namespace MyWpfApp.Models
{
    public class User
    {
        public int Id { get; set; }
        public string FirstName { get; set; }
        public string LastName { get; set; }
        public string Email { get; set; }
        public DateTime CreatedDate { get; set; }

        public string FullName => $"{FirstName} {LastName}";
    }
}
```

### 5. `Services/` - Business Logic

```csharp name=src/Services/IUserService.cs
using MyWpfApp.Models;

namespace MyWpfApp.Services
{
    public interface IUserService
    {
        Task<IEnumerable<User>> GetAllUsersAsync();
        Task<User> GetUserByIdAsync(int id);
        Task<bool> CreateUserAsync(User user);
        Task<bool> UpdateUserAsync(User user);
        Task<bool> DeleteUserAsync(int id);
    }
}
```

```csharp name=src/Services/UserService.cs
using MyWpfApp.Models;
using System.Net.Http.Json;

namespace MyWpfApp.Services
{
    public class UserService : IUserService
    {
        private readonly HttpClient _httpClient;
        private const string ApiBaseUrl = "https://api.example.com";

        public UserService(HttpClient httpClient)
        {
            _httpClient = httpClient;
        }

        public async Task<IEnumerable<User>> GetAllUsersAsync()
        {
            try
            {
                var response = await _httpClient.GetAsync($"{ApiBaseUrl}/users");
                return await response.Content.ReadAsAsync<IEnumerable<User>>();
            }
            catch (Exception ex)
            {
                Console.WriteLine($"Error: {ex.Message}");
                return Enumerable.Empty<User>();
            }
        }

        public async Task<User> GetUserByIdAsync(int id)
        {
            var response = await _httpClient.GetAsync($"{ApiBaseUrl}/users/{id}");
            return await response.Content.ReadAsAsync<User>();
        }

        public async Task<bool> CreateUserAsync(User user)
        {
            var response = await _httpClient.PostAsJsonAsync($"{ApiBaseUrl}/users", user);
            return response.IsSuccessStatusCode;
        }

        public async Task<bool> UpdateUserAsync(User user)
        {
            var response = await _httpClient.PutAsJsonAsync($"{ApiBaseUrl}/users/{user.Id}", user);
            return response.IsSuccessStatusCode;
        }

        public async Task<bool> DeleteUserAsync(int id)
        {
            var response = await _httpClient.DeleteAsync($"{ApiBaseUrl}/users/{id}");
            return response.IsSuccessStatusCode;
        }
    }
}
```

### 6. `Views/` - User Controls

```xml name=src/Views/UserListView.xaml
<UserControl
    x:Class="MyWpfApp.Views.UserListView"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    
    <Grid>
        <DataGrid 
            ItemsSource="{Binding Users}"
            SelectedItem="{Binding SelectedUser}"
            AutoGenerateColumns="False">
            <DataGrid.Columns>
                <DataGridTextColumn Header="ID" Binding="{Binding Id}" Width="50"/>
                <DataGridTextColumn Header="Name" Binding="{Binding FullName}" Width="150"/>
                <DataGridTextColumn Header="Email" Binding="{Binding Email}" Width="200"/>
                <DataGridTextColumn Header="Created" Binding="{Binding CreatedDate}" Width="150"/>
            </DataGrid.Columns>
        </DataGrid>
    </Grid>
</UserControl>
```

### 7. `Resources/` - Styles and Colors

```xml name=src/Resources/Colors.xaml
<ResourceDictionary 
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    
    <!-- Primary Colors -->
    <Color x:Key="PrimaryColor">#0078D4</Color>
    <Color x:Key="SecondaryColor">#50E6FF</Color>
    <Color x:Key="SuccessColor">#107C10</Color>
    <Color x:Key="ErrorColor">#E81B23</Color>
    
    <!-- Brushes -->
    <SolidColorBrush x:Key="PrimaryBrush" Color="{StaticResource PrimaryColor}"/>
    <SolidColorBrush x:Key="SecondaryBrush" Color="{StaticResource SecondaryColor}"/>
</ResourceDictionary>
```

```xml name=src/Resources/Styles.xaml
<ResourceDictionary
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    
    <!-- Button Style -->
    <Style TargetType="Button" x:Key="PrimaryButtonStyle">
        <Setter Property="Background" Value="{StaticResource PrimaryBrush}"/>
        <Setter Property="Foreground" Value="White"/>
        <Setter Property="Padding" Value="15,10"/>
        <Setter Property="FontSize" Value="14"/>
        <Setter Property="BorderThickness" Value="0"/>
        <Setter Property="Cursor" Value="Hand"/>
    </Style>
    
    <!-- TextBlock Style -->
    <Style TargetType="TextBlock" x:Key="HeaderStyle">
        <Setter Property="FontSize" Value="24"/>
        <Setter Property="FontWeight" Value="Bold"/>
        <Setter Property="Foreground" Value="#333333"/>
    </Style>
</ResourceDictionary>
```

### 8. `Utils/` - Helper Classes

```csharp name=src/Utils/ServiceLocator.cs
using MyWpfApp.Services;

namespace MyWpfApp.Utils
{
    public static class ServiceLocator
    {
        private static Dictionary<Type, object> _services = new();

        public static void Initialize()
        {
            // Register Services
            var httpClient = new HttpClient();
            Register<HttpClient>(httpClient);
            Register<IUserService>(new UserService(httpClient));
        }

        public static void Register<T>(object service)
        {
            _services[typeof(T)] = service;
        }

        public static T GetService<T>()
        {
            if (_services.TryGetValue(typeof(T), out var service))
                return (T)service;
            throw new Exception($"Service {typeof(T).Name} not found");
        }
    }
}
```

```csharp name=src/Utils/RelayCommand.cs
using System.Windows.Input;

namespace MyWpfApp.Utils
{
    public class RelayCommand : ICommand
    {
        private readonly Action<object?> _execute;
        private readonly Predicate<object?>? _canExecute;

        public event EventHandler? CanExecuteChanged
        {
            add { CommandManager.RequerySuggested += value; }
            remove { CommandManager.RequerySuggested -= value; }
        }

        public RelayCommand(Action<object?> execute, Predicate<object?>? canExecute = null)
        {
            _execute = execute;
            _canExecute = canExecute;
        }

        public bool CanExecute(object? parameter) => _canExecute?.Invoke(parameter) ?? true;
        public void Execute(object? parameter) => _execute(parameter);
    }
}
```

---

## 🛠️ Project File (.csproj)

```xml name=MyWpfApp.csproj
<Project Sdk="Microsoft.NET.Sdk.WindowsDesktop">

  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net6.0-windows</TargetFramework>
    <UseWPF>true</UseWPF>
    <Nullable>enable</Nullable>
    <LangVersion>latest</LangVersion>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="CommunityToolkit.Mvvm" Version="8.2.0" />
    <PackageReference Include="Newtonsoft.Json" Version="13.0.3" />
  </ItemGroup>

</Project>
```

---

## 📊 โปรเจ็คซับซ้อนตัวอย่าง: Employee Management System

### โครงสร้าง:
```
EmployeeManagementSystem/
├── src/
│   ├── ViewModels/
│   │   ├── MainViewModel.cs
│   │   ├── EmployeeListViewModel.cs
│   │   ├── EmployeeDetailViewModel.cs
│   │   └── DepartmentViewModel.cs
│   ├── Models/
│   │   ├── Employee.cs
│   │   ├── Department.cs
│   │   └── Attendance.cs
│   ├── Services/
│   │   ├── IEmployeeService.cs
│   │   ├── EmployeeService.cs
│   │   ├── IDepartmentService.cs
│   │   ├── DepartmentService.cs
│   │   └── IReportService.cs
│   ├── Views/
│   │   ├── EmployeeListView.xaml
│   │   ├── EmployeeDetailView.xaml
│   │   ├── DepartmentView.xaml
│   │   └── ReportView.xaml
│   └── Resources/
│       ├── Styles.xaml
│       ├── Colors.xaml
│       └── Templates.xaml
└── Tests/
    ├── EmployeeServiceTests.cs
    └── ViewModelTests.cs
```

### ตัวอย่าง: Employee Model
```csharp name=src/Models/Employee.cs
namespace EmployeeManagementSystem.Models
{
    public class Employee
    {
        public int EmployeeId { get; set; }
        public string FirstName { get; set; }
        public string LastName { get; set; }
        public string Email { get; set; }
        public string PhoneNumber { get; set; }
        public int DepartmentId { get; set; }
        public decimal Salary { get; set; }
        public DateTime HireDate { get; set; }
        public Department Department { get; set; }
        public List<Attendance> AttendanceRecords { get; set; }

        public string FullName => $"{FirstName} {LastName}";
    }
}
```

### ตัวอย่าง: EmployeeDetailViewModel
```csharp name=src/ViewModels/EmployeeDetailViewModel.cs
using System.Windows.Input;
using EmployeeManagementSystem.Models;
using EmployeeManagementSystem.Services;
using EmployeeManagementSystem.Utils;

namespace EmployeeManagementSystem.ViewModels
{
    public class EmployeeDetailViewModel : BaseViewModel
    {
        private readonly IEmployeeService _employeeService;
        private Employee _employee;
        private ICommand _saveCommand;
        private bool _isLoading;

        public Employee Employee
        {
            get => _employee;
            set => SetProperty(ref _employee, value);
        }

        public bool IsLoading
        {
            get => _isLoading;
            set => SetProperty(ref _isLoading, value);
        }

        public ICommand SaveCommand => _saveCommand ??= new RelayCommand(ExecuteSave);

        public EmployeeDetailViewModel(IEmployeeService employeeService)
        {
            _employeeService = employeeService;
        }

        public async void LoadEmployee(int employeeId)
        {
            IsLoading = true;
            Employee = await _employeeService.GetEmployeeByIdAsync(employeeId);
            IsLoading = false;
        }

        private async void ExecuteSave(object? parameter)
        {
            IsLoading = true;
            await _employeeService.UpdateEmployeeAsync(Employee);
            IsLoading = false;
        }
    }
}
```

### ตัวอย่าง: EmployeeDetailView.xaml
```xml name=src/Views/EmployeeDetailView.xaml
<UserControl
    x:Class="EmployeeManagementSystem.Views.EmployeeDetailView"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    
    <Grid IsEnabled="{Binding IsLoading, Converter={StaticResource InverseBoolConverter}}">
        <StackPanel Margin="20">
            <TextBlock Text="Employee Details" Style="{StaticResource HeaderStyle}"/>
            
            <Grid Margin="0,20,0,0">
                <Grid.ColumnDefinitions>
                    <ColumnDefinition Width="Auto"/>
                    <ColumnDefinition Width="*"/>
                </Grid.ColumnDefinitions>
                <Grid.RowDefinitions>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                </Grid.RowDefinitions>
                
                <!-- First Name -->
                <TextBlock Text="First Name:" Grid.Row="0" Grid.Column="0" Margin="0,0,10,0" VerticalAlignment="Center"/>
                <TextBox Grid.Row="0" Grid.Column="1" Text="{Binding Employee.FirstName}" Height="35" Padding="10"/>
                
                <!-- Last Name -->
                <TextBlock Text="Last Name:" Grid.Row="1" Grid.Column="0" Margin="0,10,10,0" VerticalAlignment="Center"/>
                <TextBox Grid.Row="1" Grid.Column="1" Text="{Binding Employee.LastName}" Height="35" Padding="10" Margin="0,10,0,0"/>
                
                <!-- Email -->
                <TextBlock Text="Email:" Grid.Row="2" Grid.Column="0" Margin="0,10,10,0" VerticalAlignment="Center"/>
                <TextBox Grid.Row="2" Grid.Column="1" Text="{Binding Employee.Email}" Height="35" Padding="10" Margin="0,10,0,0"/>
                
                <!-- Salary -->
                <TextBlock Text="Salary:" Grid.Row="3" Grid.Column="0" Margin="0,10,10,0" VerticalAlignment="Center"/>
                <TextBox Grid.Row="3" Grid.Column="1" Text="{Binding Employee.Salary}" Height="35" Padding="10" Margin="0,10,0,0"/>
                
                <!-- Save Button -->
                <Button 
                    Grid.Row="4" 
                    Grid.Column="1" 
                    Content="Save" 
                    Command="{Binding SaveCommand}"
                    Style="{StaticResource PrimaryButtonStyle}"
                    Height="40" 
                    Margin="0,20,0,0"
                    HorizontalAlignment="Right" 
                    Width="100"/>
            </Grid>
        </StackPanel>
    </Grid>
</UserControl>
```

---

## 🚀 File Naming Conventions

| Type | Convention | Example |
|------|-----------|---------|
| ViewModel | `*ViewModel.cs` | `UserViewModel.cs` |
| View | `*View.xaml` | `UserView.xaml` |
| Model | `*.cs` | `User.cs` |
| Service Interface | `I*Service.cs` | `IUserService.cs` |
| Service Implementation | `*Service.cs` | `UserService.cs` |
| Resources | `*.xaml` | `Styles.xaml` |

---

## 📦 Dependencies (.gitignore)

```
bin/
obj/
.vs/
.vscode/
*.user
*.suo
*.sln.docstates
packages/
```

---

นี่คือโครงสร้างที่เสถียร สมดุล และสามารถขยายได้สำหรับโปรเจ็ค C# + XAML + VS Code แบบมืออาชีพ! 🎉
