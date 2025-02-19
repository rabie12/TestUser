You’re right! In the latest versions of Spring Security LDAP, LdapBindAuthenticator has been removed. Instead, you can use PasswordComparisonAuthenticator or configure authentication directly in SecurityFilterChain.

How to Implement LDAP Authentication in Spring Security (Latest Version)

Since LdapBindAuthenticator is no longer available, the recommended approach is using Spring Security’s built-in LDAP authentication with PasswordComparisonAuthenticator or SecurityFilterChain.

1. Add Dependencies

Make sure your pom.xml includes:

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.ldap</groupId>
    <artifactId>spring-ldap-core</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-ldap</artifactId>
</dependency>

2. Configure LDAP Authentication in Security Filter

Instead of LdapBindAuthenticator, configure Spring Security directly in SecurityConfig.java:

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.configuration.AuthenticationConfiguration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.ldap.authentication.LdapAuthenticationProvider;
import org.springframework.security.ldap.authentication.PasswordComparisonAuthenticator;
import org.springframework.security.ldap.authentication.ad.ActiveDirectoryLdapAuthenticationProvider;
import org.springframework.security.ldap.userdetails.DefaultLdapAuthoritiesPopulator;
import org.springframework.security.ldap.userdetails.LdapAuthoritiesPopulator;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .requestMatchers("/user/**").hasRole("USER")
                .anyRequest().authenticated()
            )
            .formLogin()
            .and()
            .logout().permitAll();

        return http.build();
    }

    @Bean
    public LdapAuthenticationProvider ldapAuthenticationProvider() {
        String ldapUrl = "ldap://localhost:389";  // Change to your LDAP server
        String userDnPattern = "uid={0},ou=users,dc=example,dc=com";
        String groupSearchBase = "ou=groups,dc=example,dc=com";

        PasswordComparisonAuthenticator authenticator =
            new PasswordComparisonAuthenticator(ldapContextSource());
        authenticator.setUserDnPatterns(new String[]{userDnPattern});
        authenticator.setPasswordAttribute("userPassword");

        LdapAuthoritiesPopulator authoritiesPopulator =
            new DefaultLdapAuthoritiesPopulator(ldapContextSource(), groupSearchBase);

        return new LdapAuthenticationProvider(authenticator, authoritiesPopulator);
    }

    @Bean
    public org.springframework.ldap.core.support.LdapContextSource ldapContextSource() {
        org.springframework.ldap.core.support.LdapContextSource contextSource =
            new org.springframework.ldap.core.support.LdapContextSource();
        contextSource.setUrl("ldap://localhost:389");  // Change this to your LDAP server
        contextSource.setBase("dc=example,dc=com");
        contextSource.setUserDn("cn=admin,dc=example,dc=com");
        contextSource.setPassword("adminpassword");
        return contextSource;
    }
}

3. Explanation
	•	LDAP URL → Your LDAP server (e.g., ldap://localhost:389).
	•	User DN Pattern → Defines how users are stored (uid={0},ou=users,dc=example,dc=com).
	•	Group Search Base → Where to search for groups (ou=groups,dc=example,dc=com).
	•	PasswordComparisonAuthenticator → New recommended alternative to LdapBindAuthenticator.
	•	Spring Security Filters → Protects routes like /admin/** and /user/**.

4. Test LDAP Authentication
	•	Start your LDAP server (e.g., OpenLDAP).
	•	Try logging in with an LDAP user.
	•	If authentication fails, check:

journalctl -u slapd.service  # Logs for OpenLDAP

Alternative: Active Directory LDAP (For Windows)

If you’re integrating with Active Directory, use ActiveDirectoryLdapAuthenticationProvider:

@Bean
public LdapAuthenticationProvider ldapAuthenticationProvider() {
    ActiveDirectoryLdapAuthenticationProvider provider =
        new ActiveDirectoryLdapAuthenticationProvider("example.com", "ldap://ad.example.com:389");
    provider.setConvertSubErrorCodesToExceptions(true);
    return provider;
}

Conclusion
	•	LdapBindAuthenticator is removed → Use PasswordComparisonAuthenticator instead.
	•	Use SecurityFilterChain to protect routes.
	•	Works with OpenLDAP or Active Directory.

Let me know if you need modifications!