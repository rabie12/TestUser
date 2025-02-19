If you want to load LDAP configuration using @Value, you can directly inject properties from application.yml into your Spring Security configuration.

1. Define LDAP Configuration in application.yml

spring:
  ldap:
    url: "ldap://localhost:389"
    base: "dc=example,dc=com"
    user-dn-pattern: "uid={0},ou=users,dc=example,dc=com"
    group-search-base: "ou=groups,dc=example,dc=com"
    admin-user: "cn=admin,dc=example,dc=com"
    admin-password: "adminpassword"

2. Update SecurityConfig Using @Value

Instead of using @ConfigurationProperties, use @Value to inject properties directly into SecurityConfig.java:

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
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

3. Explanation
	•	@Value("${spring.ldap.url}") → Reads values from application.yml.
	•	No need for extra configuration classes like LdapProperties.
	•	LDAP authentication provider is configured dynamically using injected values.

4. Testing

Start your Spring Boot application and test authentication:

curl -u username:password http://localhost:8080/user/profile

or use the Spring Security login form.

✅ Why This Approach?

✔ Simple & Lightweight
✔ Directly Uses @Value Without Extra Classes
✔ Works With Environment Variables (application.yml or .properties)

Let me know if you need any refinements!