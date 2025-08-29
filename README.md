# Simple-Wchat-Moment
The final project in my freshman year

package socialMedia;

import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

public class Comment {
    private String author;
    private String content;
    private LocalDateTime timestamp;

// Parameter-containing structure
    public Comment(String author, String content) {
        this.author = author;
        this.content = content;
        this.timestamp = LocalDateTime.now();
    }

    public String getAuthor() {
        return author;
    }

    public String getContent() {
        return content;
    }

    public String getTimestamp() {
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
        return timestamp.format(formatter);
    }

    @Override
    public String toString() {
        return author + " [" + getTimestamp() + "]: " + content;
    }
}


package socialMedia;

// to import javaFX
import javafx.application.Application; // basic class
import javafx.geometry.Insets; // edge distance
import javafx.scene.Scene; // UI
import javafx.scene.control.*; // the element of UI, such as button
import javafx.scene.layout.*;
import javafx.stage.Stage; // main window

public class FriendCircleApp extends Application {

    private PostManager postManager = new PostManager();
    // vertical
    private VBox postListView = new VBox(10);
    //box
    private ScrollPane scrollPane;

    @Override
    public void start(Stage primaryStage) {     // entrance
        primaryStage.setTitle("moment");

        TextField authorInput = new TextField();
        authorInput.setPromptText("your name: ");
        TextArea postInput = new TextArea();
        postInput.setPromptText("write something: ");

        postInput.setWrapText(true);
        postInput.setPrefRowCount(3);
        postInput.setTextFormatter(new TextFormatter<String>(change ->
                change.getControlNewText().length() <= 200 ? change : null));

        Button postButton = new Button("submit");

        postButton.setOnAction(e -> {
            String author = authorInput.getText().trim();
            String content = postInput.getText().trim();
            if (!author.isEmpty() && !content.isEmpty()) {
                postManager.addPost(author, content);
                updatePostList();
                postInput.clear();
                authorInput.clear();
            }
        });

        scrollPane = new ScrollPane(postListView);
        scrollPane.setFitToWidth(true);

        VBox layout = new VBox(10, authorInput, postInput, postButton, scrollPane);
        layout.setPadding(new Insets(10));

        //pixel
        primaryStage.setScene(new Scene(layout, 500, 600));
        primaryStage.show();
    }

    private void updatePostList() {
        postListView.getChildren().clear();

        for (Post post : postManager.getAllPosts()) {
            VBox postBox = new VBox(10);
            postBox.setPadding(new Insets(10));
            postBox.setStyle("-fx-border-color: #ccc; -fx-background-color: #f9f9f9; -fx-background-radius: 8;");

            Label authorLabel = new Label(post.getAuthor() + " published at: " + post.getTimestamp());
            Label contentLabel = new Label(post.getContent());
            contentLabel.setWrapText(true);

            Label likeLabel = new Label("👍 " + post.getLikes());
            Button likeButton = new Button("like");
            Button deleteButton = new Button("delete");
//button
            likeButton.setOnAction(e -> {
                post.like();
                updatePostList();
            });

            deleteButton.setOnAction(e -> {
                postManager.removePost(post);
                updatePostList();
            });

            HBox actionBox = new HBox(10, likeButton, deleteButton, likeLabel);

// creat a vbox to show the moment in column
            VBox commentBox = new VBox(5);
            for (Comment comment : post.getComments()) {
                Label commentLabel = new Label(comment.toString());
                commentLabel.setWrapText(true);
                commentBox.getChildren().add(commentLabel);
            }

// put the name and comment
            TextField commentAuthorInput = new TextField();
            commentAuthorInput.setPromptText("your name: ");
            TextField commentContentInput = new TextField();
            commentContentInput.setPromptText("write comments");

            Button commentButton = new Button("comments");
            commentButton.setOnAction(e -> {
                String commentAuthor = commentAuthorInput.getText().trim();
                String commentContent = commentContentInput.getText().trim();
                if (!commentAuthor.isEmpty() && !commentContent.isEmpty() && commentContent.length() >= 3) {
                    post.addComment(new Comment(commentAuthor, commentContent));
                    updatePostList();
                }
            });
// put the button
            HBox commentInputBox = new HBox(5, commentAuthorInput, commentContentInput, commentButton);
            commentInputBox.setPadding(new Insets(5, 0, 0, 0));
// add to UI
            postBox.getChildren().addAll(authorLabel, contentLabel, actionBox, commentBox, commentInputBox);
            postListView.getChildren().add(postBox);
        }

        scrollPane.layout();
        scrollPane.setVvalue(1.0);
    }

    public static void main(String[] args) {
        launch(args);
    }
}


package socialMedia;

import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.ArrayList;
import java.util.List;

//constructor
public class Post {
    private String author;
    private String content;
    private LocalDateTime timestamp;
    private int likes;
    private List<Comment> comments = new ArrayList<>();

// Parameter-containing structure
    public Post(String author, String content) {
        this.author = author;
        this.content = content;
        this.timestamp = LocalDateTime.now();
    }

    public String getAuthor() {
        return author;
    }

    public String getContent() {
        return content;
    }

    public String getTimestamp() {
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
        return timestamp.format(formatter);
    }

    public int getLikes() {
        return likes;
    }

    public void like() {
        likes++;
    }

    public List<Comment> getComments() {
        return comments;
    }

    public void addComment(Comment comment) {
        comments.add(comment);
    }

    @Override
    public String toString() {
        return author + " [" + getTimestamp() + "]: " + content;
    }
}



package socialMedia;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

//to create, delete, storage, get the whole post
public class PostManager {
    private List<Post> posts = new ArrayList<>();

    public void addPost(String author, String content) {
        posts.add(new Post(author, content));
    }

    public void removePost(Post post) {
        posts.remove(post);
    }

    public List<Post> getAllPosts() {
        List<Post> reversed = new ArrayList<>(posts);
        Collections.reverse(reversed);
        return reversed;
    }
}


   
// Java module descriptor
module com.example.javafx {
    requires javafx.controls;
    requires javafx.fxml;

    requires org.controlsfx.controls;
    requires org.kordamp.bootstrapfx.core;

    opens com.example.javafx to javafx.fxml;
    exports com.example.javafx;
    opens socialMedia to javafx.graphics;
}
