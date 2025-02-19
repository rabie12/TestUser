Since roles are stored in LDAP, you need to retrieve them after authentication and log them. Here’s how you can do it:

✅ 1. Modify Security Configuration to Get Roles from LDAP

Since Spring Security LDAP uses DefaultLdapAuthoritiesPopulator to fetch roles, we will log them after authentication.

Security Configuration (SecurityConfig.java)

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.ldap.authentication.LdapAuthenticationProvider;
import org.springframework.security.ldap.authentication.PasswordComparisonAuthenticator;
import org.springframework.security.ldap.userdetails.DefaultLdapAuthoritiesPopulator;
import org.springframework.security.ldap.userdetails.LdapAuthoritiesPopulator;
import org.springframework.ldap.core.support.LdapContextSource;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class SecurityConfig {

    @Value("${spring.ldap.url}")
    private String ldapUrl;

    @Value("${spring.ldap.base}")
    private String ldapBase;

    @Value("${spring.ldap.user-dn-pattern}")
    private String userDnPattern;

    @Value("${spring.ldap.group-search-base}")
    private String groupSearchBase;

    @Value("${spring.ldap.admin-user}")
    private String adminUser;

    @Value("${spring.ldap.admin-password}")
    private String adminPassword;

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth.anyRequest().authenticated())
            .formLogin()
            .and()
            .logout().permitAll();

        return http.build();
    }

    @Bean
    public LdapAuthenticationProvider ldapAuthenticationProvider() {
        PasswordComparisonAuthenticator authenticator =
            new PasswordComparisonAuthenticator(ldapContextSource());
        authenticator.setUserDnPatterns(new String[]{userDnPattern});
        authenticator.setPasswordAttribute("userPassword");

        LdapAuthoritiesPopulator authoritiesPopulator =
            new DefaultLdapAuthoritiesPopulator(ldapContextSource(), groupSearchBase);

        return new LdapAuthenticationProvider(authenticator, authoritiesPopulator);
    }

    @Bean
    public LdapContextSource ldapContextSource() {
        LdapContextSource contextSource = new LdapContextSource();
        contextSource.setUrl(ldapUrl);
        contextSource.setBase(ldapBase);
        contextSource.setUserDn(adminUser);
        contextSource.setPassword(adminPassword);
        return contextSource;
    }
}

✅ 2. Log LDAP Roles After Authentication

Use an event listener to log user roles after a successful LDAP authentication.

Logging Listener (AuthenticationSuccessListener.java)

import org.springframework.context.ApplicationListener;
import org.springframework.security.authentication.event.AuthenticationSuccessEvent;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.stereotype.Component;

@Component
public class AuthenticationSuccessListener implements ApplicationListener<AuthenticationSuccessEvent> {

    @Override
    public void onApplicationEvent(AuthenticationSuccessEvent event) {
        Object principal = event.getAuthentication().getPrincipal();

        if (principal instanceof UserDetails userDetails) {
            System.out.println("✅ Authenticated User: " + userDetails.getUsername());
            System.out.println("🔹 Roles from LDAP:");
            for (GrantedAuthority authority : userDetails.getAuthorities()) {
                System.out.println(" - " + authority.getAuthority());
            }
        }
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
	1.	Spring Security fetches user roles from LDAP.
	2.	Event Listener (AuthenticationSuccessListener) logs roles after authentication.
	3.	You can check roles in the console immediately after login.

Now, every time a user logs in via LDAP, their roles will be printed in the console. Let me know if you need any refinements! 🚀