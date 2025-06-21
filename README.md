# Tugas Elearning Pemrograman Berorientasi Obyek 2
<ul>
  <li>Mata Kuliah: Pemrograman Berorientasi Obyek 2</li>
  <li>Dosen Pengampu: <a href="https://github.com/Muhammad-Ikhwan-Fathulloh">Muhammad Ikhwan Fathulloh</a></li>
</ul>

## Profil
<ul>
  <li>Nama: Fadillah Fajri</li>
  <li>NIM: 19552011285</li>
</ul>

##  JavaFX Dashboard Boilerplate
<p>
  Proyek ini adalah boilerplate (template dasar) untuk membangun aplikasi dashboard menggunakan JavaFX. 
  Dirancang agar pengembang dapat langsung fokus pada fitur bisnis tanpa harus membangun struktur UI dan manajemen tampilan dari awal.
  
  Berikut adalah langkah-langkah untuk menggunakan Boilerplate JavaFX Dashboard.
</p>

##  Database
<p>
  Sistem ini menggunakan MySQL untuk menyimpan data. 
</p>
<p>
  Buat Database dan Tabel di MySQL
</p>

``` sql
CREATE DATABASE todo_system;

USE todo_system;

CREATE TABLE IF NOT EXISTS users (
  id INT AUTO_INCREMENT PRIMARY KEY,
  username VARCHAR(255) NOT NULL UNIQUE,
  password VARCHAR(255) NOT NULL,
  role VARCHAR(255) NOT NULL
);

CREATE TABLE IF NOT EXISTS todos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    is_completed BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```
---
DatabaseConnection.java

``` java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class DatabaseConnection {

    private static final String URL = "jdbc:mysql://localhost:3306/your_database_name"; // Ganti 'your_database_name' dengan nama database Anda
    private static final String USER = "your_username"; // Ganti 'your_username' dengan username database Anda
    private static final String PASSWORD = "your_password"; // Ganti 'your_password' dengan password database Anda

    private static Connection connection;

    static {
        try {
            // Memuat driver JDBC
            Class.forName("com.mysql.cj.jdbc.Driver");

            // Membuat koneksi ke database
            connection = DriverManager.getConnection(URL, USER, PASSWORD);

            System.out.println("Database connection established successfully.");
        } catch (ClassNotFoundException e) {
            System.err.println("JDBC Driver not found.");
            e.printStackTrace();
        } catch (SQLException e) {
            System.err.println("Failed to establish database connection.");
            e.printStackTrace();
        }
    }

    public static Connection getConnection() {
        return connection;
    }

    public static void closeConnection() {
        if (connection != null) {
            try {
                connection.close();
                System.out.println("Database connection closed.");
            } catch (SQLException e) {
                System.err.println("Failed to close database connection.");
                e.printStackTrace();
            }
        }
    }
}
```
##  Tambahkan Library JDBC untuk MySQL
<p>
  Pastikan Anda memiliki driver JDBC untuk database yang Anda gunakan. Untuk MySQL, Anda bisa mengunduh driver dari <a href="https://dev.mysql.com/downloads/connector/j/">MySQL Connector/J</a> atau menggunakan Maven dengan menambahkan dependensi berikut di pom.xml :
</p>

``` xml
<dependencies>
    <dependency>
      <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.32</version>
    </dependency>
</dependencies>
```
DashboardView.java

``` java
package com.mycompany.boilerplate;

import java.sql.SQLException;
import java.util.logging.Level;
import java.util.logging.Logger;
import javafx.geometry.Insets;
import javafx.scene.Scene;
import javafx.scene.control.*;
import javafx.scene.layout.BorderPane;
import javafx.scene.layout.VBox;
import javafx.stage.Stage;

public class DashboardView {
     private Stage primaryStage;
    private UserOperations userOperations;
    private String username;

    public DashboardView(Stage primaryStage, String username) {
        this.primaryStage = primaryStage;
        this.username = username;
    }

    public BorderPane getView() {
        BorderPane root = new BorderPane();
        root.setPadding(new Insets(20)); // Add padding to the root

        // Menu Navigasi
        VBox menu = new VBox(10);
        menu.setPadding(new Insets(10));
        menu.setStyle("-fx-background-color: #f0f0f0; -fx-padding: 10px;"); // Add background color to menu

        Button todoButton = new Button("Todos");
        todoButton.setStyle("-fx-padding: 10px 20px; -fx-font-size: 14px;");
        todoButton.setOnAction(e -> {
            // Tampilkan TodoView
            System.out.println("Fitur Todo");
            TodoView todoView = new TodoView(primaryStage, username); 
            primaryStage.setScene(new Scene(todoView.getView(), 800, 600));
        });

        Button logoutButton = new Button("Logout");
        logoutButton.setStyle("-fx-padding: 10px 20px; -fx-font-size: 14px;");
        logoutButton.setOnAction(e -> {
            // Arahkan ke LoginView
            LoginView loginView = new LoginView(primaryStage);
            primaryStage.setScene(new Scene(loginView.getView(), 800, 600));
        });

        menu.getChildren().addAll(new Label("Todo Apps"), todoButton, logoutButton);
        root.setLeft(menu);

        // Tampilan Utama
        VBox content = new VBox(10);
        content.setPadding(new Insets(20));
        
        Label welcomeLabel = new Label("Selamat datang "+username+" di Dashboard!");
        welcomeLabel.setStyle("-fx-font-size: 20px; -fx-font-weight: bold;");

        // Add more content here (e.g., recent todos, statistics, etc.)
        try {
            userOperations = new UserOperations();
        } catch (SQLException ex) {
            Logger.getLogger(DashboardView.class.getName()).log(Level.SEVERE, null, ex);
        }
        
        User user = userOperations.getProfile(username);

        content.getChildren().add(welcomeLabel);
        if (user != null) {
            // Display user profile information (e.g., in a Label)
            Label profileLabel = new Label("Username: " + user.getUsername());
            Label roleLabel = new Label("Role: " + user.getRole());
            content.getChildren().addAll(profileLabel, roleLabel);
        } 
        root.setCenter(content);

        return root;
    }
}
```
TodoView.java

``` java

package com.mycompany.boilerplate;

import javafx.collections.FXCollections;
import javafx.collections.ObservableList;
import javafx.geometry.Insets;
import javafx.scene.Scene;
import javafx.scene.control.*;
import javafx.stage.Modality;
import javafx.scene.layout.VBox;
import javafx.stage.Stage;

import java.sql.SQLException;
import javafx.scene.layout.BorderPane;
import javafx.scene.layout.HBox;

public class TodoView {
  private TodoOperations todoOperations;
    private TableView<Todo> tableView;
    private ObservableList<Todo> todoList;
    private Stage primaryStage;
    private String username;

    public TodoView(Stage primaryStage, String username) {
        this.primaryStage = primaryStage;
        this.username = username;
        try {
            todoOperations = new TodoOperations();
            todoList = FXCollections.observableArrayList(todoOperations.getTodos());
        } catch (SQLException e) {
            e.printStackTrace();
            showError("Error loading todos: " + e.getMessage());
        }
    }

    public BorderPane getView() {
        BorderPane root = new BorderPane();
        root.setPadding(new Insets(20));

        // Menu Navigasi
        VBox menu = new VBox(10);
        menu.setPadding(new Insets(10));
        menu.setStyle("-fx-background-color: #f0f0f0; -fx-padding: 10px;"); 

        Button dashboardButton = new Button("Dashboard");
        dashboardButton.setStyle("-fx-padding: 10px 20px; -fx-font-size: 14px;");
        dashboardButton.setOnAction(e -> {
            // Tampilkan TodoView (already on TodoView)
            System.out.println("Dashboard");
             DashboardView dashboardView = new DashboardView(primaryStage, username); 
             primaryStage.setScene(new Scene(dashboardView.getView(), 800, 600)); 
        });

        Button logoutButton = new Button("Logout");
        logoutButton.setStyle("-fx-padding: 10px 20px; -fx-font-size: 14px;");
        logoutButton.setOnAction(e -> {
            // Arahkan ke LoginView
            LoginView loginView = new LoginView(primaryStage);
            primaryStage.setScene(new Scene(loginView.getView(), 800, 600));
        });

        menu.getChildren().addAll(new Label("Todo Apps"), dashboardButton, logoutButton);
        root.setLeft(menu);

        // Tampilan Utama
        VBox content = new VBox(10);
        content.setPadding(new Insets(20));

        // TableView
        tableView = new TableView<>();
        tableView.setItems(todoList); 

        // Kolom
        TableColumn<Todo, Integer> idColumn = new TableColumn<>("ID");
        idColumn.setCellValueFactory(data -> data.getValue().idProperty().asObject());

        TableColumn<Todo, String> titleColumn = new TableColumn<>("Title");
        titleColumn.setCellValueFactory(data -> data.getValue().titleProperty());

        TableColumn<Todo, String> descriptionColumn = new TableColumn<>("Description");
        descriptionColumn.setCellValueFactory(data -> data.getValue().descriptionProperty());

        TableColumn<Todo, Boolean> isCompletedColumn = new TableColumn<>("Completed");
        isCompletedColumn.setCellValueFactory(data -> data.getValue().isCompletedProperty());

        TableColumn<Todo, Void> actionColumn = new TableColumn<>("Actions");
        actionColumn.setCellFactory(param -> new TableCell<>() {
            private Button editButton = new Button("Edit");
            private Button deleteButton = new Button("Delete");
            private Button markCompletedButton = new Button("Mark as Completed");

            @Override
            protected void updateItem(Void item, boolean empty) {
                super.updateItem(item, empty);
                if (empty) {
                    setGraphic(null);
                } else {
                    editButton.setOnAction(event -> {
                        Todo selected = getTableView().getItems().get(getIndex());
                        showTodoModal(selected);
                    });

                    deleteButton.setOnAction(event -> {
                        Todo selected = getTableView().getItems().get(getIndex());
                        todoOperations.deleteTodo(selected.getId());
                        refreshTable();
                        showSuccess("Todo deleted successfully!");
                    });

                    markCompletedButton.setOnAction(event -> {
                        Todo selected = getTableView().getItems().get(getIndex());
                        todoOperations.markAsCompleted(selected.getId());
                        refreshTable();
                        showSuccess("Todo marked as completed!");
                    });

                    HBox buttonContainer = new HBox(5);
                    buttonContainer.getChildren().addAll(editButton, deleteButton, markCompletedButton);
                    setGraphic(buttonContainer);
                }
            }
        });

        tableView.getColumns().addAll(idColumn, titleColumn, descriptionColumn, isCompletedColumn, actionColumn);
        tableView.setStyle("-fx-border-color: #ccc; -fx-border-width: 1px;"); 
        actionColumn.setMinWidth(350); // Atur lebar minimum kolom
        
        // Add Button
        Button addButton = new Button("Add");
        addButton.setOnAction(e -> {
            showTodoModal(null);
        });
        
        // Add TableView to content
        content.getChildren().addAll(addButton, tableView); 

        root.setCenter(content);

        return root;
    }

    private void showTodoModal(Todo todo) {
        Stage modalStage = new Stage();
        modalStage.initModality(Modality.APPLICATION_MODAL);
        modalStage.setTitle(todo == null ? "Add Todo" : "Edit Todo");

        TextField titleField = new TextField(todo == null ? "" : todo.getTitle());
        TextField descriptionField = new TextField(todo == null ? "" : todo.getDescription());

        Button saveButton = new Button(todo == null ? "Add" : "Save");
        saveButton.setOnAction(e -> {
            if (todo == null) {
                todoOperations.addTodo(new Todo(0, titleField.getText(), descriptionField.getText(), false, null));
                showSuccess("Todo added successfully!");
            } else {
                todoOperations.updateTodo(todo.getId(), titleField.getText(), descriptionField.getText());
                showSuccess("Todo updated successfully!");
            }
            refreshTable();
            modalStage.close();
        });

        VBox modalContent = new VBox(10);
        modalContent.setPadding(new Insets(10));
        modalContent.getChildren().addAll(new Label("Title:"), titleField, new Label("Description:"), descriptionField, saveButton);

        Scene modalScene = new Scene(modalContent);
        modalStage.setScene(modalScene);
        modalStage.showAndWait();
    }

    private void refreshTable() {
        todoList.setAll(todoOperations.getTodos());
    }

    private void showSuccess(String message) {
        Alert alert = new Alert(Alert.AlertType.INFORMATION);
        alert.setTitle("Success");
        alert.setHeaderText(null);
        alert.setContentText(message);
        alert.showAndWait();
    }

    private void showError(String message) {
        Alert alert = new Alert(Alert.AlertType.ERROR);
        alert.setTitle("Error");
        alert.setHeaderText(null);
        alert.setContentText(message);
        alert.showAndWait();
    }
}
```
TodoOperations.java
``` java
package com.mycompany.boilerplate;

import java.sql.*;
import java.util.ArrayList;
import java.util.List;

public class TodoOperations {
     private Connection connection;

    public TodoOperations() throws SQLException {
        connection = DatabaseConnection.getConnection();
    }

    // Create
    public void addTodo(Todo todo) {
        String query = "INSERT INTO todos (title, description) VALUES (?, ?)";
        try (PreparedStatement stmt = connection.prepareStatement(query)) {
            stmt.setString(1, todo.getTitle());
            stmt.setString(2, todo.getDescription());
            stmt.executeUpdate();
            System.out.println("To-Do added successfully!");
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    // Read
    public List<Todo> getTodos() {
        List<Todo> todos = new ArrayList<>();
        String query = "SELECT * FROM todos";
        try (Statement stmt = connection.createStatement();
             ResultSet rs = stmt.executeQuery(query)) {

            while (rs.next()) {
                todos.add(new Todo(
                        rs.getInt("id"),
                        rs.getString("title"),
                        rs.getString("description"),
                        rs.getBoolean("is_completed"),
                        rs.getString("created_at")
                ));
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return todos;
    }

    // Update
    public void updateTodo(int id, String newTitle, String newDescription) {
        String query = "UPDATE todos SET title = ?, description = ? WHERE id = ?";
        try (PreparedStatement stmt = connection.prepareStatement(query)) {
            stmt.setString(1, newTitle);
            stmt.setString(2, newDescription);
            stmt.setInt(3, id);
            stmt.executeUpdate();
            System.out.println("To-Do updated successfully!");
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    // Delete
    public void deleteTodo(int id) {
        String query = "DELETE FROM todos WHERE id = ?";
        try (PreparedStatement stmt = connection.prepareStatement(query)) {
            stmt.setInt(1, id);
            stmt.executeUpdate();
            System.out.println("To-Do deleted successfully!");
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    // Mark as Completed
    public void markAsCompleted(int id) {
        String query = "UPDATE todos SET is_completed = TRUE WHERE id = ?";
        try (PreparedStatement stmt = connection.prepareStatement(query)) {
            stmt.setInt(1, id);
            stmt.executeUpdate();
            System.out.println("To-Do marked as completed!");
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```
Todo.java
``` java
package com.mycompany.boilerplate;

import javafx.beans.property.*;

public class Todo {
    private IntegerProperty id;
    private StringProperty title;
    private StringProperty description;
    private BooleanProperty isCompleted;
    private StringProperty createdAt;

    public Todo(int id, String title, String description, boolean isCompleted, String createdAt) {
        this.id = new SimpleIntegerProperty(id);
        this.title = new SimpleStringProperty(title);
        this.description = new SimpleStringProperty(description);
        this.isCompleted = new SimpleBooleanProperty(isCompleted);
        this.createdAt = new SimpleStringProperty(createdAt);
    }

    public int getId() {
        return id.get();
    }

    public IntegerProperty idProperty() {
        return id;
    }

    public String getTitle() {
        return title.get();
    }

    public StringProperty titleProperty() {
        return title;
    }

    public String getDescription() {
        return description.get();
    }

    public StringProperty descriptionProperty() {
        return description;
    }

    public boolean isIsCompleted() {
        return isCompleted.get();
    }

    public BooleanProperty isCompletedProperty() {
        return isCompleted;
    }

    public String getCreatedAt() {
        return createdAt.get();
    }

    public StringProperty createdAtProperty() {
        return createdAt;
    }
}
```
LoginView.java
``` java
package com.mycompany.boilerplate;

import java.sql.SQLException;
import java.util.logging.Level;
import java.util.logging.Logger;
import javafx.geometry.Insets;
import javafx.scene.Scene;
import javafx.scene.control.*;
import javafx.scene.layout.VBox;
import javafx.stage.Stage;

public class LoginView {
     private Stage primaryStage;
    private UserOperations userOperations;

    public LoginView(Stage primaryStage) {
        this.primaryStage = primaryStage;
    }

    public VBox getView() {
        VBox root = new VBox(10);
        root.setPadding(new Insets(20));
        root.getStyleClass().add("root");

        // Elemen Login
        Label titleLabel = new Label("Login");
        titleLabel.setStyle("-fx-font-size: 24px; -fx-font-weight: bold;");

        TextField usernameField = new TextField();
        usernameField.setPromptText("Username");
        usernameField.setStyle("-fx-pref-width: 300px; -fx-padding: 10px;");

        PasswordField passwordField = new PasswordField();
        passwordField.setPromptText("Password");
        passwordField.setStyle("-fx-pref-width: 300px; -fx-padding: 10px;");
        Button loginButton = new Button("Login");
        loginButton.setStyle("-fx-background-color: #4CAF50; -fx-text-fill: white; -fx-padding: 10px 20px;");

        // Elemen Register
        Label registerLink = new Label("Belum punya akun? Register di sini.");
        registerLink.setStyle("-fx-text-fill: blue; -fx-underline: true;");
        registerLink.setOnMouseClicked(event -> {
            // Arahkan ke RegisterView
            RegisterView registerView = new RegisterView(primaryStage);
            Scene registerScene = new Scene(registerView.getView(), 800, 600);
            primaryStage.setScene(registerScene);
        });

        loginButton.setOnAction(e -> {
            String username = usernameField.getText();
            String password = passwordField.getText();

            try {
                userOperations = new UserOperations();
            } catch (SQLException ex) {
                Logger.getLogger(LoginView.class.getName()).log(Level.SEVERE, null, ex);
            }
            if (userOperations.loginUser(username, password)) {
                // Login sukses, arahkan ke Dashboard
                DashboardView dashboardView = new DashboardView(primaryStage, username);
                Scene dashboardScene = new Scene(dashboardView.getView(), 800, 600);
                primaryStage.setScene(dashboardScene);
            } else {
                showError("Login gagal! Periksa username dan password Anda.");
            }
        });
        
        root.getChildren().addAll(titleLabel, usernameField, passwordField, loginButton, registerLink);
        return root;
    }

    private void showError(String message) {
        Alert alert = new Alert(Alert.AlertType.ERROR);
        alert.setTitle("Error");
        alert.setHeaderText(null);
        alert.setContentText(message);
        alert.showAndWait();
    }
}
```
RegisterView.java
``` java
package com.mycompany.boilerplate;

import java.sql.SQLException;
import java.util.logging.Level;
import java.util.logging.Logger;
import javafx.collections.FXCollections;
import javafx.collections.ObservableList;
import javafx.geometry.Insets;
import javafx.scene.Scene;
import javafx.scene.control.*;
import javafx.scene.layout.VBox;
import javafx.stage.Stage;

public class RegisterView {
     private Stage primaryStage;
    private UserOperations userOperations;
    
    public RegisterView(Stage primaryStage) {
        this.primaryStage = primaryStage;
    }

    public VBox getView() {
        VBox root = new VBox(10);
        root.setPadding(new Insets(20));

        // Elemen Register
        Label titleLabel = new Label("Register");
        titleLabel.setStyle("-fx-font-size: 24px; -fx-font-weight: bold;");
        
        TextField usernameField = new TextField();
        usernameField.setPromptText("Username");
        usernameField.setStyle("-fx-pref-width: 300px; -fx-padding: 10px;");

        PasswordField passwordField = new PasswordField();
        passwordField.setPromptText("Password");
        passwordField.setStyle("-fx-pref-width: 300px; -fx-padding: 10px;");

        ComboBox<String> roleComboBox = new ComboBox<>();
        ObservableList<String> roles = FXCollections.observableArrayList("User", "Admin");
        roleComboBox.setItems(roles);
        roleComboBox.setValue("User"); // Set default value to "User"
        roleComboBox.setStyle("-fx-pref-width: 300px; -fx-padding: 10px;"); 
        Button registerButton = new Button("Register");
        registerButton.setStyle("-fx-background-color: #4CAF50; -fx-text-fill: white; -fx-padding: 10px 20px;");
        
        Label loginLink = new Label("Sudah punya akun? Login di sini.");
        loginLink.setStyle("-fx-text-fill: blue; -fx-underline: true;");
        loginLink.setOnMouseClicked(event -> {
            // Arahkan ke RegisterView
            LoginView loginView = new LoginView(primaryStage);
            Scene loginScene = new Scene(loginView.getView(), 800, 600);
            primaryStage.setScene(loginScene);
        });

        registerButton.setOnAction(e -> {
            String username = usernameField.getText();
            String password = passwordField.getText();
            String role = roleComboBox.getValue();

            try {
                userOperations = new UserOperations();
            } catch (SQLException ex) {
                Logger.getLogger(RegisterView.class.getName()).log(Level.SEVERE, null, ex);
            }
            if (userOperations.registerUser(username, password, role)) {
                showSuccess("Register berhasil! Silakan login.");
                // Arahkan ke LoginView
                LoginView loginView = new LoginView(primaryStage);
                primaryStage.setScene(new Scene(loginView.getView(), 800, 600));
            } else {
                showError("Register gagal! Username mungkin sudah digunakan.");
            }
        });

        root.getChildren().addAll(titleLabel, usernameField, passwordField, roleComboBox, registerButton, loginLink);
        return root;
    }

    private void showSuccess(String message) {
        Alert alert = new Alert(Alert.AlertType.INFORMATION);
        alert.setTitle("Success");
        alert.setHeaderText(null);
        alert.setContentText(message);
        alert.showAndWait();
    }

    private void showError(String message) {
        Alert alert = new Alert(Alert.AlertType.ERROR);
        alert.setTitle("Error");
        alert.setHeaderText(null);
        alert.setContentText(message);
        alert.showAndWait();
    }
}
```
User.java
``` java
package com.mycompany.boilerplate;

import javafx.beans.property.*;
/**
 *
 * @author Fadil
 */
public class User {
     private IntegerProperty id;
    private StringProperty username;
    private StringProperty password;
    private StringProperty role;

    public User(int id, String username, String password, String role) {
        this.id = new SimpleIntegerProperty(id);
        this.username = new SimpleStringProperty(username);
        this.password = new SimpleStringProperty(password);
        this.role = new SimpleStringProperty(role);
    }

    // Getter and Property for 'id'
    public int getId() {
        return id.get();
    }

    public IntegerProperty idProperty() {
        return id;
    }

    public void setId(int id) {
        this.id.set(id);
    }

    // Getter and Property for 'username'
    public String getUsername() {
        return username.get();
    }

    public StringProperty usernameProperty() {
        return username;
    }

    public void setUsername(String username) {
        this.username.set(username);
    }

    // Getter and Property for 'password'
    public String getPassword() {
        return password.get();
    }

    public StringProperty passwordProperty() {
        return password;
    }

    public void setPassword(String password) {
        this.password.set(password);
    }
    
    public StringProperty roleProperty() {
        return role;
    }

    public void setRole(String role) {
        this.role.set(role);
    }

    public String getRole() {
        return role.get();
    }
}
```
UserOperations.java
``` java
package com.mycompany.boilerplate;

import java.sql.*;

public class UserOperations {
    private Connection connection;

    public UserOperations() throws SQLException {
        connection = DatabaseConnection.getConnection();
    }

    // Register
    public void registerUser(String username, String password) {
        String query = "INSERT INTO users (username, password) VALUES (?, ?)";
        try (PreparedStatement stmt = connection.prepareStatement(query)) {
            stmt.setString(1, username);
            stmt.setString(2, password); // Make sure to hash the password before storing
            stmt.executeUpdate();
            System.out.println("User registered successfully!");
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    // Login
    public boolean loginUser(String username, String password) {
        String query = "SELECT * FROM users WHERE username = ? AND password = ?";
        try (PreparedStatement stmt = connection.prepareStatement(query)) {
            stmt.setString(1, username);
            stmt.setString(2, password); // Compare the hashed password
            try (ResultSet rs = stmt.executeQuery()) {
                if (rs.next()) {
                    System.out.println("Login successful!");
                    return true;
                } else {
                    System.out.println("Invalid username or password.");
                    return false;
                }
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return false;
    }

    // Logout (example implementation using a session system)
    public void logoutUser() {
        // Assuming you have a session system in place
        System.out.println("User logged out successfully!");
    }

    // Get Profile
    public User getProfile(String username) {
        String query = "SELECT * FROM users WHERE username = ?";
        try (PreparedStatement stmt = connection.prepareStatement(query)) {
            stmt.setString(1, username);
            try (ResultSet rs = stmt.executeQuery()) {
                if (rs.next()) {
                    return new User(
                            rs.getInt("id"),
                            rs.getString("username"),
                            rs.getString("password"), // For security, avoid returning the password
                            rs.getString("role")
                    );
                }
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return null;
    }

    // Update Password
    public void updatePassword(String username, String newPassword) {
        String query = "UPDATE users SET password = ? WHERE username = ?";
        try (PreparedStatement stmt = connection.prepareStatement(query)) {
            stmt.setString(1, newPassword); // Make sure to hash the password before storing
            stmt.setString(2, username);
            int rowsAffected = stmt.executeUpdate();
            if (rowsAffected > 0) {
                System.out.println("Password updated successfully!");
            } else {
                System.out.println("Failed to update password. User not found.");
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    public boolean registerUser(String username, String password, String role) {
    String query = "INSERT INTO users (username, password, role) VALUES (?, ?, ?)";
    try (PreparedStatement stmt = connection.prepareStatement(query)) {
        stmt.setString(1, username);
        stmt.setString(2, password); 
        stmt.setString(3, role);
        stmt.executeUpdate();
        System.out.println("User registered successfully!");
        return true;
    } catch (SQLException e) {
        e.printStackTrace();
        return false;
    }
   }
}
```
Main.java
``` java
package com.mycompany.boilerplate;

import javafx.application.Application;
import javafx.scene.Scene;
import javafx.scene.layout.BorderPane;
import javafx.stage.Stage;

public class Main extends Application {

    @Override
    public void start(Stage primaryStage) {
        try {
            // Inisialisasi Root Layout
            BorderPane root = new BorderPane();

            // Tampilkan Halaman LoginView terlebih dahulu
            LoginView loginView = new LoginView(primaryStage);
            root.setCenter(loginView.getView());

            // Buat Scene
            Scene scene = new Scene(root, 800, 600);

            // Atur Stage
            primaryStage.setTitle("Todo App");
            primaryStage.setScene(scene);
            primaryStage.show();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        launch(args);
    }
}
```
## Demo Proyek
<ul>
  <li>Youtube: <a href="https://youtu.be/tOwJjqK9ebk">Youtube</a></li>
</ul>
