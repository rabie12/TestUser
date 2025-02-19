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
    public LdapContextSource ldapContext