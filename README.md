import jakarta.persistence.*;
import java.util.Set;

@Entity
@Table(name = "roles")
public class Role {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(unique = true, nullable = false)
    private String name;

    @ManyToMany(fetch = FetchType.EAGER)
    @JoinTable(
        name = "role_permissions", // Correct join table name
        joinColumns = @JoinColumn(name = "role_id"), // Correct column for role
        inverseJoinColumns = @JoinColumn(name = "permission_id") // Correct column for permission
    )
    private Set<Permission> permissions;

    public Role() {}

    public Role(String name) {
        this.name = name;
    }

    public void setPermissions(Set<Permission> permissions) {
        this.permissions = permissions;
    }

    public Set<Permission> getPermissions() {
        return permissions;
    }
}