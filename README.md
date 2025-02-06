package com.example.demo.controller;

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

    @Operation(summary = "Supprimer un utilisateur", description = "Supprime l'utilisateur correspondant à l'ID fourni.")
    @ApiResponses(value = {
        @ApiResponse(responseCode = "204", description = "Utilisateur supprimé avec succès", content = @Content),
        @ApiResponse(responseCode = "404", description = "Utilisateur non trouvé", content = @Content)
    })
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        // Appel au service pour supprimer l'utilisateur
        return ResponseEntity.noContent().build();
    }
}
