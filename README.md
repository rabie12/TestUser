Below is a complete example of how to configure your Spring Boot application so that Swagger (using springdoc-openapi) only shows your REST endpoints and not all your entity models. You can:
	1.	Configure your Maven (or Gradle) dependencies,
	2.	Create an OpenAPI configuration class to limit scanning to your controller endpoints, and
	3.	Optionally hide entity model details if you don’t want them to appear in the generated documentation.

1. Maven Dependencies

Make sure you have the necessary dependencies in your pom.xml (adjust versions as needed):

<dependencies>
    <!-- Spring Boot Starter Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- Springdoc OpenAPI UI -->
    <dependency>
        <groupId>org.springdoc</groupId>
        <artifactId>springdoc-openapi-ui</artifactId>
        <version>1.6.14</version>
    </dependency>

    <!-- Optionally, if you use validation -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>
    
    <!-- Your other dependencies -->
</dependencies>

2. OpenAPI Configuration

Create a configuration class (for example, OpenApiConfig.java) to limit what gets exposed. In this example, we group only endpoints under /api/**:

package com.example.demo.config;

import org.springdoc.core.GroupedOpenApi;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class OpenApiConfig {

    /**
     * Exposes only endpoints that match the path pattern /api/**
     * Adjust pathsToMatch as needed.
     */
    @Bean
    public GroupedOpenApi publicApi() {
        return GroupedOpenApi.builder()
                .group("public")
                .pathsToMatch("/api/**")
                .build();
    }
}

This configuration tells springdoc to only include endpoints that match /api/** in your Swagger UI. In many cases, this prevents internal classes (or endpoints not meant for external use) from being exposed.

3. Application Properties

You can also adjust some properties in your application.properties (or application.yml) file if desired:

# Set the path for the API docs and Swagger UI
springdoc.api-docs.path=/v3/api-docs
springdoc.swagger-ui.path=/swagger-ui.html

For YAML, it would be:

springdoc:
  api-docs:
    path: /v3/api-docs
  swagger-ui:
    path: /swagger-ui.html

4. Hiding Entities (Optional)

If you notice that Swagger is picking up your JPA entity classes as models and you’d prefer not to show them, you can hide them by annotating the class with @Schema(hidden = true). For example:

package com.example.demo.model;

import io.swagger.v3.oas.annotations.media.Schema;
import jakarta.persistence.Entity;
import jakarta.persistence.Id;

@Entity
@Schema(hidden = true) // This hides the entity from the swagger model definitions
public class UserEntity {

    @Id
    private Long id;
    
    private String email;
    private String name;

    // Getters and setters...
}

If you want to use a separate DTO for your REST API (which is often recommended), you can let your entity be hidden from Swagger and expose only the DTO. For instance, you may have a User DTO class that is documented:

package com.example.demo.dto;

import io.swagger.v3.oas.annotations.media.Schema;

@Schema(description = "User Data Transfer Object")
public class User {

    @Schema(description = "Unique identifier", example = "1")
    private Long id;

    @Schema(description = "User email", example = "alice@example.com")
    private String email;

    @Schema(description = "User name", example = "Alice")
    private String name;

    // Constructors, getters, setters...
}

Then, in your controller, use the DTO instead of the entity.

5. Example User Controller

Here’s the complete controller (using the DTO) with Swagger annotations:

package com.example.demo.controller;

import com.example.demo.dto.User;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.media.ArraySchema;
import io.swagger.v3.oas.annotations.media.Content;
import io.swagger.v3.oas.annotations.media.Schema;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import io.swagger.v3.oas.annotations.responses.ApiResponses;
import io.swagger.v3.oas.annotations.tags.Tag;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;
import java.util.Arrays;

@RestController
@RequestMapping("/api/users")
@Tag(name = "User Controller", description = "Operations related to users")
public class UserController {

    @Operation(summary = "Get all users", description = "Retrieves a list of all users")
    @ApiResponses(value = {
        @ApiResponse(responseCode = "200", description = "Successful retrieval",
            content = @Content(array = @ArraySchema(schema = @Schema(implementation = User.class)))),
        @ApiResponse(responseCode = "401", description = "Unauthorized", content = @Content)
    })
    @GetMapping
    public ResponseEntity<List<User>> getAllUsers() {
        // Replace with service call in production
        List<User> users = Arrays.asList(
                new User(1L, "alice@example.com", "Alice"),
                new User(2L, "bob@example.com", "Bob")
        );
        return ResponseEntity.ok(users);
    }

    @Operation(summary = "Get user by ID", description = "Retrieves a user by its identifier")
    @ApiResponses(value = {
        @ApiResponse(responseCode = "200", description = "User found",
            content = @Content(schema = @Schema(implementation = User.class))),
        @ApiResponse(responseCode = "404", description = "User not found", content = @Content)
    })
    @GetMapping("/{id}")
    public ResponseEntity<User> getUserById(@PathVariable Long id) {
        // Replace with service call in production
        User user = new User(id, "user@example.com", "Example User");
        return ResponseEntity.ok(user);
    }

    @Operation(summary = "Create a user", description = "Creates a new user")
    @ApiResponses(value = {
        @ApiResponse(responseCode = "201", description = "User created",
            content = @Content(schema = @Schema(implementation = User.class))),
        @ApiResponse(responseCode = "400", description = "Invalid data", content = @Content)
    })
    @PostMapping
    public ResponseEntity<User> createUser(@RequestBody User user) {
        // Replace with service call in production
        user.setId(123L);
        return ResponseEntity.status(HttpStatus.CREATED).body(user);
    }

    @Operation(summary = "Delete a user", description = "Deletes a user by its identifier")
    @ApiResponses(value = {
        @ApiResponse(responseCode = "204", description = "User deleted", content = @Content),
        @ApiResponse(responseCode = "404", description = "User not found", content = @Content)
    })
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        // Replace with service call in production
        return ResponseEntity.noContent().build();
    }
}

Final Notes
	•	Grouping endpoints: The GroupedOpenApi configuration ensures that only paths matching /api/** are documented.
	•	DTO vs. Entity: It’s a good practice to separate your REST API models (DTOs) from your persistence entities. This also prevents internal entity details from leaking into your API documentation.
	•	Customizing the UI: You can further customize your Swagger UI by adding additional OpenAPI configuration or using springdoc properties.

With these configurations in place, you should see only your REST endpoints (and their DTO models) in the Swagger UI while keeping internal entity details hidden.
 com.example.demo.controller;

import com.example.demo.model.User;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;
import java.util.Arrays;

import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.media.ArraySchema;
import io.swagger.v3.oas.annotations.media.Content;
import io.swagger.v3.oas.annotations.media.Schema;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import io.swagger.v3.oas.annotations.responses.ApiResponses;
import io.swagger.v3.oas.annotations.tags.Tag;

@RestController
@RequestMapping("/api/users")
@Tag(name = "User Controller", description = "Gestion des utilisateurs")
public class UserController {

    @Operation(summary = "Récupérer la liste des utilisateurs", description = "Retourne la liste complète des utilisateurs.")
    @ApiResponses(value = {
        @ApiResponse(responseCode = "200", description = "Opération réussie",
                content = @Content(array = @ArraySchema(schema = @Schema(implementation = User.class)))),
        @ApiResponse(responseCode = "401", description = "Non autorisé", content = @Content)
    })
    @GetMapping
    public ResponseEntity<List<User>> getAllUsers() {
        // Exemple de données en dur, à remplacer par un appel à votre service
        List<User> users = Arrays.asList(
                new User(1L, "alice@example.com", "Alice"),
                new User(2L, "bob@example.com", "Bob")
        );
        return ResponseEntity.ok(users);
    }

    @Operation(summary = "Récupérer un utilisateur par son ID", description = "Retourne un utilisateur correspondant à l'ID fourni.")
    @ApiResponses(value = {
        @ApiResponse(responseCode = "200", description = "Opération réussie",
                content = @Content(schema = @Schema(implementation = User.class))),
        @ApiResponse(responseCode = "404", description = "Utilisateur non trouvé", content = @Content)
    })
    @GetMapping("/{id}")
    public ResponseEntity<User> getUserById(@PathVariable Long id) {
        // Exemple simplifié, à remplacer par un appel à votre service
        User user = new User(id, "user@example.com", "Exemple");
        return ResponseEntity.ok(user);
    }

    @Operation(summary = "Créer un nouvel utilisateur", description = "Permet de créer un nouvel utilisateur dans le système.")
    @ApiResponses(value = {
        @ApiResponse(responseCode = "201", description = "Utilisateur créé avec succès",
                content = @Content(schema = @Schema(implementation = User.class))),
        @ApiResponse(responseCode = "400", description = "Données invalides", content = @Content)
    })
    @PostMapping
    public ResponseEntity<User> createUser(@RequestBody User user) {
        // Traitement de la création de l'utilisateur, par exemple via un service
        // Ici on simule l'attribution d'un ID et le retour de l'objet créé.
        user.setId(123L);
        return ResponseEntity.status(HttpStatus.CREATED).body(user);
    }
import jakarta.persistence.Column;
import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.Id;
import org.hibernate.annotations.GenericGenerator;

import java.util.UUID;

@Entity
public class MyEntity {

    @Id
    @GeneratedValue(generator = "UUID")
    @GenericGenerator(
        name = "UUID",
        strategy = "org.hibernate.id.UUIDGenerator"
    )
    @Column(name = "id", updatable = false, nullable = false)
    private UUID id;

    // other fields, getters, setters, etc.
}
    @Operation(summary = "Supprimer un utilisateur", description = "Supprime l'utilisateur correspondant à l'ID fourni.")
    @ApiResponses(value = {
        @ApiResponse(responseCode = "204", description = "Utilisateur supprimé avec succès", content = @Content), le
        @ApiResponse(responseCode = "404", description = "Utilisateur non trouvé", content = @Content)
    })
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        // Appel au service pour supprimer l'utilisateur
        return ResponseEntity.noContent().build();
    }
}
