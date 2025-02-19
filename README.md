
If the AuthenticationSuccessEvent is not recognized or not triggering, you can manually log the LDAP roles after authentication using a filter instead of relying on event listeners.

✅ 1. Use a Custom Filter to Log LDAP Roles

Instead of AuthenticationSuccessEvent, use a Spring Security filter that logs the user’s roles after authentication.

Create LdapRoleLoggingFilter.java

import jakarta.servlet.Filter;
import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.ServletRequest;
import jakarta.servlet.ServletResponse;
import jakarta.servlet.http.HttpServletRequest;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.stereotype.Component;

import java.io.IOException;

@Component
public class LdapRoleLoggingFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {

        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();

        if (authentication != null && authentication.isAuthenticated()) {
            System.out.println("✅ Authenticated User: " + authentication.getName());
            System.out.println("🔹 Roles from LDAP:");
            for (GrantedAuthority authority : authentication.getAuthorities()) {
                System.out.println(" - " + authority.getAuthority());
            }
        }

        chain.doFilter(request, response);
    }
}

✅ 2. Register the Filter in SecurityConfig

Modify your SecurityConfig.java to register this filter after authentication.

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

@Configuration
public class SecurityConfig {

    private final LdapRoleLoggingFilter ldapRoleLoggingFilter;

    public SecurityConfig(LdapRoleLoggingFilter ldapRoleLoggingFilter) {
        this.ldapRoleLoggingFilter = ldapRoleLoggingFilter;
    }

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth.anyRequest().authenticated())
            .formLogin()
            .and()
            .logout().permitAll();

        // Add LDAP role logging filter after authentication
        http.addFilterAfter(ldapRoleLoggingFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }
}

✅ 3. Test Authentication & Check Logs

Start Your Application & Log In
	•	Start your Spring Boot application.
	•	Log in via browser or Postman/cURL.
	•	Check the console output.

Example Login Request Using cURL

curl -u john.doe:password http://localhost:8080

Expected Console Output

✅ Authenticated User: john.doe
🔹 Roles from LDAP:
 - ROLE_ADMIN
 - ROLE_USER

✅ Summary