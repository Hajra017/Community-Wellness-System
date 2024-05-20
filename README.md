                                                Community Wellness System

The Community Wellness System is a Java-based application designed to facilitate wellness challenges within a community. It allows users to register, create challenges, and track their progress over time. This project serves as an end-semester project for Object-Oriented Programming (OOP) in Java.

Features

User Management: Users can register and log in securely.
Challenge Creation: Admins can create wellness challenges with names and descriptions.
Progress Tracking: Users can track their progress in various challenges.
Technologies Used
Java

JavaFX for the front-end
SQLite for database integration
Project Structure
The project is structured as follows:

src/: Contains the source code files.
Main.java: Entry point of the application.
database/: Contains the database utility class.
user management/: Contains classes for user registration and login.
challenge creation/: Contains classes for challenge creation.
progress tracking/: Contains classes for progress tracking.
UI/: Contains JavaFX UI files and controllers.
resources/: Contains FXML files for UI design.
wellness.db: SQLite database file.

CODE:
#Challenge.java

package Project;
public class Challenge {
    private int id;
    private String name;
    private String description;

    // Constructor
    public Challenge(int id, String name, String description) {
        this.id = id;
        this.name = name;
        this.description = description;
    }

    // Getter for id
    public int getId() {
        return id;
    }

    // Setter for id
    public void setId(int id) {
        this.id = id;
    }

    // Getter for name
    public String getName() {
        return name;
    }

    // Setter for name
    public void setName(String name) {
        this.name = name;
    }

    // Getter for description
    public String getDescription() {
        return description;
    }

    // Setter for description
    public void setDescription(String description) {
        this.description = description;
    }
}

#ChallengeDAO.java

package Project;

import java.sql.*;
import java.util.ArrayList;
import java.util.List;

public class ChallengeDAO {

    private DatabaseUtil databaseUtil;

    public ChallengeDAO() {
        this.databaseUtil = databaseUtil;
    }

    public boolean createChallenge(String name, String description) {
        String sql = "INSERT INTO challenges(name, description) VALUES(?, ?)";

        try (Connection conn = databaseUtil.connect();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            pstmt.setString(1, name);
            pstmt.setString(2, description);
            pstmt.executeUpdate();
            return true;
        } catch (SQLException e) {
            System.out.println(e.getMessage());
            return false;
        }
    }

    public List<Challenge> getAllChallenges() {
        List<Challenge> challenges = new ArrayList<>();
        String sql = "SELECT * FROM challenges";

        try (Connection conn = databaseUtil.connect();
             Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery(sql)) {

            while (rs.next()) {
                Challenge challenge = new Challenge(
                        rs.getInt("id"),
                        rs.getString("name"),
                        rs.getString("description")
                );
                challenges.add(challenge);
            }
        } catch (SQLException e) {
            System.out.println(e.getMessage());
        }
        return challenges;
    }
}

#CommunityWellnessApp.java

package Project;
import javafx.application.Application;
import javafx.fxml.FXMLLoader;
import javafx.scene.Scene;
import javafx.stage.Stage;

public class CommunityWellnessApp extends Application {

    @Override
    public void start(Stage primaryStage) throws Exception {
        DatabaseUtil.createTables();

        FXMLLoader loader = new FXMLLoader(getClass().getResource("Login.fxml"));
        Scene scene = new Scene(loader.load());

        primaryStage.setTitle("Community Wellness System");
        primaryStage.setScene(scene);
        primaryStage.show();
    }

    public static void main(String[] args) {
        launch(args);
    }
}

#CreateChallengeController

package Project;
import javafx.fxml.FXML;
import javafx.scene.control.TextArea;
import javafx.scene.control.TextField;
import javafx.scene.control.Alert;
import javafx.scene.control.Alert.AlertType;

public class CreateChallengeController {
    @FXML private TextField nameField;
    @FXML private TextArea descriptionField;

    private ChallengeDAO challengeDAO = new ChallengeDAO();

    @FXML
    private void handleCreate() {
        String name = nameField.getText();
        String description = descriptionField.getText();
        boolean success = challengeDAO.createChallenge(name, description);

        if (success) {
            showAlert("Challenge Created", "Challenge created successfully!");
        } else {
            showAlert("Creation Failed", "Could not create challenge");
        }
    }


    private void showAlert(String title, String content) {
        Alert alert = new Alert(AlertType.INFORMATION);
        alert.setTitle(title);
        alert.setContentText(content);
        alert.showAndWait();
    }
}

#DatabaseUtil

package Project;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.sql.Statement;

public class DatabaseUtil {

    private static final String URL = "jdbc:mysql://localhost:3306/community_wellness";
    private static final String USER = "hajra_user";
    private static final String PASSWORD = "Hajra123@";

    public static Connection connect() {
        Connection conn = null;
        try {
            conn = DriverManager.getConnection(URL, USER, PASSWORD);
        } catch (SQLException e) {
            System.out.println(e.getMessage());
        }
        return conn;
    }

    public static void createTables() {
        String createUserTable = "CREATE TABLE IF NOT EXISTS users (" +
                "id INT AUTO_INCREMENT PRIMARY KEY, " +
                "username VARCHAR(50) NOT NULL, " +
                "email VARCHAR(100) NOT NULL, " +
                "password VARCHAR(100) NOT NULL, " +
                "location VARCHAR(100));";

        String createChallengeTable = "CREATE TABLE IF NOT EXISTS challenges (" +
                "id INT AUTO_INCREMENT PRIMARY KEY, " +
                "name VARCHAR(100) NOT NULL, " +
                "duration INT NOT NULL, " +
                "goal VARCHAR(255));";

        try (Connection conn = connect();
             Statement stmt = conn.createStatement()) {
            stmt.execute(createUserTable);
            stmt.execute(createChallengeTable);
        } catch (SQLException e) {
            System.out.println(e.getMessage());
        }
    }
}

#LoginController.java

package Project;
import javafx.fxml.FXML;
import javafx.scene.control.PasswordField;
import javafx.scene.control.TextField;
import javafx.scene.control.Alert;
import javafx.scene.control.Alert.AlertType;
import javafx.stage.Stage;
import javafx.scene.Scene;
import javafx.fxml.FXMLLoader;

public class LoginController {
    @FXML private TextField usernameField;
    @FXML private PasswordField passwordField;

    private UserDAO userDAO = new UserDAO();

    @FXML
    private void handleLogin() {
        String username = usernameField.getText();
        String password = passwordField.getText();
        User user = userDAO.loginUser(username, password);

        if (user != null) {
            loadMainUI(user);
        } else {
            showAlert("Login Failed", "Invalid username or password");
        }
    }

    @FXML
    private void handleRegister() {
        String username = usernameField.getText();
        String password = passwordField.getText();
        boolean success = userDAO.registerUser(username, password);

        if (success) {
            showAlert("Registration Successful", "User registered successfully!");
        } else {
            showAlert("Registration Failed", "Could not register user");
        }
    }

    private void showAlert(String title, String content) {
        Alert alert = new Alert(AlertType.INFORMATION);
        alert.setTitle(title);
        alert.setContentText(content);
        alert.showAndWait();
    }

    private void loadMainUI(User user) {
        try {
            FXMLLoader loader = new FXMLLoader(getClass().getResource("MainUI.fxml"));
            Stage stage = (Stage) usernameField.getScene().getWindow();
            stage.setScene(new Scene(loader.load()));
            MainUIController controller = loader.getController();
            controller.initData(user);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

#Main.java

package Project;
import javafx.application.Application;

public class Main {
    public static void main(String[] args) {
        Application.launch(CommunityWellnessApp.class, args);
    }
}

#MainUIController.java

package Project;
import javafx.fxml.FXML;
import javafx.scene.control.Label;

public class MainUIController {
    @FXML private Label welcomeLabel;

    private User user;

    public void initData(User user) {
        this.user = user;
        welcomeLabel.setText("Welcome, " + user.getUsername());
    }

    @FXML
    private void handleCreateChallenge() {
        // Implement challenge creation logic
    }

    @FXML
    private void handleViewChallenges() {
        // Implement view challenges logic
    }

    @FXML
    private void handleTrackProgress() {
        // Implement progress tracking logic
    }
}

#Progress.java

package Project;

public class Progress {
    private int id;
    private int userId;
    private int challengeId;
    private int progress;

    public Progress(int id, int userId, int challengeId, int progress) {
        this.id = id;
        this.userId = userId;
        this.challengeId = challengeId;
        this.progress = progress;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public int getUserId() {
        return userId;
    }

    public void setUserId(int userId) {
        this.userId = userId;
    }

    public int getChallengeId() {
        return challengeId;
    }

    public void setChallengeId(int challengeId) {
        this.challengeId = challengeId;
    }

    public int getProgress() {
        return progress;
    }

    public void setProgress(int progress) {
        this.progress = progress;
    }
}

#ProgressDAO.java

package Project;
import java.sql.*;

public class ProgressDAO {

    public boolean updateProgress(int userId, int challengeId, int progress) {
        String sql = "INSERT INTO progress(user_id, challenge_id, progress) VALUES(?, ?, ?) "
                + "ON CONFLICT(user_id, challenge_id) DO UPDATE SET progress = ?";

        try (Connection conn = DatabaseUtil.connect();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            pstmt.setInt(1, userId);
            pstmt.setInt(2, challengeId);
            pstmt.setInt(3, progress);
            pstmt.setInt(4, progress);
            pstmt.executeUpdate();
            return true;
        } catch (SQLException e) {
            System.out.println(e.getMessage());
            return false;
        }
    }

    public Progress getProgress(int userId, int challengeId) {
        String sql = "SELECT * FROM progress WHERE user_id = ? AND challenge_id = ?";

        try (Connection conn = DatabaseUtil.connect();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            pstmt.setInt(1, userId);
            pstmt.setInt(2, challengeId);
            ResultSet rs = pstmt.executeQuery();

            if (rs.next()) {
                return new Progress(
                        rs.getInt("id"),
                        rs.getInt("user_id"),
                        rs.getInt("challenge_id"),
                        rs.getInt("progress")
                );
            }
        } catch (SQLException e) {
            System.out.println(e.getMessage());
        }
        return null;
    }
}

#TrackProgressController

package Project;
import javafx.fxml.FXML;
import javafx.scene.control.TextField;
import javafx.scene.control.Alert;
import javafx.scene.control.Alert.AlertType;

public class TrackProgressController {
    @FXML private TextField challengeIdField;
    @FXML private TextField progressField;

    private ProgressDAO progressDAO = new ProgressDAO();
    private User user;

    public void initData(User user) {
        this.user = user;
    }

    @FXML
    private void handleUpdate() {
        int challengeId = Integer.parseInt(challengeIdField.getText());
        int progress = Integer.parseInt(progressField.getText());
        boolean success = progressDAO.updateProgress(user.getId(), challengeId, progress);

        if (success) {
            showAlert("Progress Updated", "Progress updated successfully!");
        } else {
            showAlert("Update Failed", "Could not update progress");
        }
    }

    private void showAlert(String title, String content) {
        Alert alert = new Alert(AlertType.INFORMATION);
        alert.setTitle(title);
        alert.setContentText(content);
        alert.showAndWait();
    }
}

#User.java

package Project;
public class User {
    private int id;
    private String username;
    private String password;

    // Constructor
    public User(int id, String username, String password) {
        this.id = id;
        this.username = username;
        this.password = password;
    }

    // Getter for id
    public int getId() {
        return id;
    }

    // Setter for id
    public void setId(int id) {
        this.id = id;
    }

    // Getter for username
    public String getUsername() {
        return username;
    }

    // Setter for username
    public void setUsername(String username) {
        this.username = username;
    }

    // Getter for password
    public String getPassword() {
        return password;
    }

    // Setter for password
    public void setPassword(String password) {
        this.password = password;
    }
}

#UserDAO.java

package Project;
import java.sql.*;

public class UserDAO {

    public boolean registerUser(String username, String password) {
        String sql = "INSERT INTO users(username, password) VALUES(?, ?)";

        try (Connection conn = DatabaseUtil.connect();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            pstmt.setString(1, username);
            pstmt.setString(2, password);
            pstmt.executeUpdate();
            return true;
        } catch (SQLException e) {
            System.out.println(e.getMessage());
            return false;
        }
    }

    public User loginUser(String username, String password) {
        String sql = "SELECT * FROM users WHERE username = ? AND password = ?";

        try (Connection conn = DatabaseUtil.connect();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            pstmt.setString(1, username);
            pstmt.setString(2, password);
            ResultSet rs = pstmt.executeQuery();

            if (rs.next()) {
                return new User(rs.getInt("id"), rs.getString("username"), rs.getString("password"));
            }
        } catch (SQLException e) {
            System.out.println(e.getMessage());
        }
        return null;
    }
}

#ViewChallengesController

package Project;
import javafx.fxml.FXML;
import javafx.scene.control.ListView;
import javafx.collections.FXCollections;
import javafx.collections.ObservableList;

import java.util.List;

public class ViewChallengesController {
    @FXML private ListView<String> challengesListView;

    private ChallengeDAO challengeDAO = new ChallengeDAO();

    @FXML
    private void handleRefresh() {
        List<Challenge> challenges = challengeDAO.getAllChallenges();
        ObservableList<String> challengeNames = FXCollections.observableArrayList();
        for (Challenge challenge : challenges) {
            challengeNames.add(challenge.getName());
        }
        challengesListView.setItems(challengeNames);
    }
}

#import Project.DatabaseUtil;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;

public class Main {
    public static void main(String[] args) {
        // Example usage of DatabaseUtil to connect to the database
        try (Connection conn = DatabaseUtil.connect()) {
            if (conn != null) {
                System.out.println("Connected to the database!");

                // Example query
                String query = "SELECT * FROM users";
                try (PreparedStatement pstmt = conn.prepareStatement(query)) {
                    // Execute your query here
                }
            } else {
                System.out.println("Failed to connect to the database!");
            }
        } catch (SQLException e) {
            System.out.println("SQL Exception: " + e.getMessage());
        }
    }
}

#CreateChallenge.fxml

<?xml version="1.0" encoding="UTF-8"?>

<?import javafx.scene.control.Button?>
<?import javafx.scene.control.TextArea?>
<?import javafx.scene.control.TextField?>
<?import javafx.scene.layout.VBox?>

<VBox alignment="CENTER" spacing="10.0" xmlns="http://javafx.com/javafx/8" xmlns:fx="http://javafx.com/fxml/1" fx:controller="Project.CreateChallengeController">
    <TextField fx:id="nameField" promptText="Challenge Name"/>
    <TextArea fx:id="descriptionField" promptText="Challenge Description"/>
    <Button text="Create" onAction="#handleCreate"/>
</VBox>

#Login.fxml

<?xml version="1.0" encoding="UTF-8"?>

<?import javafx.geometry.Insets?>
<?import javafx.scene.control.Button?>
<?import javafx.scene.control.PasswordField?>
<?import javafx.scene.control.TextField?>
<?import javafx.scene.layout.VBox?>
<?import Project.LoginController?> <!-- Importing LoginController -->

<VBox alignment="CENTER" spacing="10.0" xmlns="http://javafx.com/javafx/8" xmlns:fx="http://javafx.com/fxml/1" fx:controller="Project.LoginController"> <!-- Specifying the fully qualified name -->
    <TextField fx:id="usernameField" promptText="Username"/>
    <PasswordField fx:id="passwordField" promptText="Password"/>
    <Button text="Login" onAction="#handleLogin"/>
    <Button text="Register" onAction="#handleRegister"/>
</VBox>

#LoginController.fxml

<?xml version="1.0" encoding="UTF-8"?>

<?import java.lang.*?>
<?import java.util.*?>
<?import javafx.scene.*?>
<?import javafx.scene.control.*?>
<?import javafx.scene.layout.*?>

<?import Project.*?> <!-- Importing the Project package -->

<AnchorPane xmlns="http://javafx.com/javafx"
            xmlns:fx="http://javafx.com/fxml"
            fx:controller="LoginController"
            prefHeight="400.0" prefWidth="600.0">

    <!-- Your FXML content here -->

</AnchorPane>

#MainUI.fxml

<?xml version="1.0" encoding="UTF-8"?>

<?import javafx.scene.control.Button?>
<?import javafx.scene.control.Label?>
<?import javafx.scene.layout.VBox?>

<VBox alignment="CENTER" spacing="10.0" xmlns="http://javafx.com/javafx/8" xmlns:fx="http://javafx.com/fxml/1" fx:controller="Project.MainUIController">
    <Label fx:id="welcomeLabel"/>
    <Button text="Create Challenge" onAction="#handleCreateChallenge"/>
    <Button text="View Challenges" onAction="#handleViewChallenges"/>
    <Button text="Track Progress" onAction="#handleTrackProgress"/>
</VBox>

